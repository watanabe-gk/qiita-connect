---
title: Haskellで学んだ関数の等価性問題がReact/TypeScriptで引き起こす実装上の落とし穴
tags:
  - TypeScript
  - React
  - Haskell
  - 関数型プログラミング
  - パフォーマンス
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

Haskellを学んでいると、「関数はEqクラスのインスタンスになれない」という概念に出会います。

```haskell
-- Haskellではこれがエラーになる
ghci> (\x -> x + 1) == (\x -> x + 1)
Error: No instance for (Eq (Int -> Int))
```

これは単なる理論上の制約ではなく、**TypeScript/JavaScriptでも同じ問題が存在し、実装上の様々なバグの原因**となります。

本記事では、Haskellで学んだ関数の等価性問題が、React/TypeScriptの実務でどのような不利益を引き起こすのか、具体例とともに解説します。

## TL;DR（結論）

関数の等価性判定ができないことで発生する主な問題:

- ❌ useEffectの無限ループ → アプリケーションクラッシュ
- ❌ イベントリスナーの削除失敗 → メモリリーク
- ✅ 解決策: useCallbackで関数の同一性を保証

## 関数の等価性問題とは

### Haskellでの制約

Haskellでは、関数型は`Eq`クラスのインスタンスになれません。

```haskell
-- これはコンパイルエラー
ghci> (\x -> x + 1) == (\x -> x + 1)
Error: No instance for (Eq (Int -> Int))

-- 関数のリストも作れるが、比較はできない
let funcs = [(\x -> x + 1), (\x -> x * 2)]
-- funcs !! 0 == funcs !! 1  -- エラー
```

### なぜ関数の等価性は判定できないのか？

#### 1. 数学的な理由：停止性問題

2つの関数が等しいかを判定するには、**すべての可能な入力に対して同じ出力を返すか検証**する必要があります。

```haskell
-- これら2つの関数は数学的には等しい
f1 x = x + 1
f2 x = 1 + x

-- しかし、プログラム的に等価性を判定するには：
-- すべての x ∈ Int に対して f1(x) == f2(x) を確認する必要がある
-- → Int は無限個あるため、検証不可能
```

さらに、**停止性問題**（チューリングの決定不能問題）により、一般的な関数の等価性判定は計算不可能です。

```haskell
-- これら2つが等しいか判定できるか？
g1 x = if someComplexComputation x then 1 else 0
g2 x = 0

-- もし someComplexComputation がすべての入力で False を返すなら等しい
-- しかし、それを判定するには停止性問題を解く必要がある
-- → 原理的に不可能
```

#### 2. メモリ表現の理由：関数は実行時オブジェクト

プログラム実行時、関数は**メモリ上のコードへのポインタ**として表現されます。

```typescript
// TypeScript/JavaScriptでの例
const f1 = (x: number) => x + 1;
const f2 = (x: number) => x + 1;

// メモリ構造（概念図）:
// f1 → [メモリアドレス: 0x1000] → 関数の実装コード
// f2 → [メモリアドレス: 0x2000] → 関数の実装コード（別の場所）

console.log(f1 === f2);  // false
// ↑ メモリアドレスを比較している（0x1000 !== 0x2000）
```

同じ処理を行う関数でも、**メモリ上の異なる位置に配置される**ため、参照が異なります。

#### 3. 実装の観点：参照の等価性 vs 意味的な等価性

プログラミング言語には2種類の等価性があります:

| 等価性の種類 | 意味 | TypeScript/JavaScript | Haskell |
|------------|------|----------------------|---------|
| **参照の等価性** | 同じメモリ位置を指すか | `===` で判定可能 | 実装依存 |
| **意味的な等価性** | 同じ振る舞いをするか | 判定不可能 | 判定不可能 |

```typescript
// 参照の等価性
const f1 = (x: number) => x + 1;
const f2 = f1;  // 同じ参照をコピー
console.log(f1 === f2);  // true（同じメモリアドレス）

// 意味的な等価性
const g1 = (x: number) => x + 1;
const g2 = (x: number) => 1 + x;  // 数学的には同じ
console.log(g1 === g2);  // false（異なるメモリアドレス）

// 意味的には等しいが、参照は異なる
console.log(g1(5) === g2(5));  // true（結果は同じ）
```

### TypeScript/JavaScriptでの具体例

```typescript
const f1 = (x: number) => x + 1;
const f2 = (x: number) => x + 1;

console.log(f1 === f2);  // false（期待はtrueだが実際はfalse）
console.log(f1 == f2);   // false

// 同じ関数を変数に代入した場合のみtrue
const f3 = f1;
console.log(f1 === f3);  // true（同じ参照）
```

これは**参照の等価性**を比較しているためで、**意味的な等価性**（同じ処理を行うかどうか）は判定できません。

### Haskellとの共通点

Haskellで「関数がEqのインスタンスになれない」のと同じ理由で、TypeScript/JavaScriptでも関数の意味的な等価性は判定できません。この制約は**言語の問題ではなく、計算理論の根本的な限界**なのです。

## 実装上の不利益とバグ例

### 1. useEffectの無限ループ

```typescript
// ❌ 悪い例: 関数を依存配列に含める
function DataFetcher() {
  const [data, setData] = useState(null);

  const fetchData = () => {
    return fetch('/api/data').then(res => res.json());
  };

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // fetchDataは毎レンダリングで新規作成 → 無限ループ

  return <div>{JSON.stringify(data)}</div>;
}

// ✅ 良い例: useCallbackで関数の同一性を保証
function DataFetcher() {
  const [data, setData] = useState(null);

  const fetchData = useCallback(() => {
    return fetch('/api/data').then(res => res.json());
  }, []);

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]);

  return <div>{JSON.stringify(data)}</div>;
}
```

### 2. イベントリスナーの削除失敗

```typescript
// ❌ 悪い例: 異なる関数インスタンスなので削除されない
function ScrollTracker() {
  useEffect(() => {
    window.addEventListener('scroll', () => {
      console.log('Scrolled!');
    });

    return () => {
      window.removeEventListener('scroll', () => {
        console.log('Scrolled!');
      });
    };
  }, []);
}

// ✅ 良い例: 同じ関数インスタンスを使う
function ScrollTracker() {
  useEffect(() => {
    const handleScroll = () => {
      console.log('Scrolled!');
    };

    window.addEventListener('scroll', handleScroll);
    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  }, []);
}
```

## まとめ

- 関数の等価性問題は理論だけでなく、実装上の様々なバグの原因になる
- Reactでは特に`useCallback`と`useMemo`が重要
- イベントリスナーの管理では同じ関数インスタンスを使う
- Haskellで学んだ概念がTypeScript/JavaScriptでも役立つ

関数の等価性を意識することで、より効率的でバグの少ないコードが書けるようになります。

## 参考資料

- [React Hooks API Reference - useCallback](https://react.dev/reference/react/useCallback)
- [React Hooks API Reference - useMemo](https://react.dev/reference/react/useMemo)
- [Haskell Eq Class](https://hackage.haskell.org/package/base/docs/Prelude.html#t:Eq)

---

この記事が、関数の等価性問題を理解する助けになれば幸いです。
質問やフィードバックがあれば、コメントでお知らせください！
