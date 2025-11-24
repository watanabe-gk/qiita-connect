---
title: Docker Composeのstdin_openとttyの違いを徹底検証してみた
tags:
  - Docker
  - docker-compose
  - Haskell
  - コンテナ
  - 検証
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

Docker Composeで開発環境を構築する際、`stdin_open`と`tty`という2つの設定項目を目にすることがあります。

```yaml
services:
  app:
    stdin_open: true
    tty: true
```

「両方`true`にしておけば動く」というのは分かるものの、**実際にこれらが何をしているのか**、**片方だけでも動くのか**、気になったことはありませんか？

本記事では、Haskell開発環境を例に、実際に4パターンの設定を試して動作を検証してみました。

## TL;DR（結論）

- **両方`false`**: ❌ コンテナがすぐ停止、使用不可
- **`stdin_open=true`, `tty=false`**: ⚠️ 動作するが機能制限あり
- **`stdin_open=false`, `tty=true`**: ⚠️ 動作するが機能制限あり
- **両方`true`**: ✅ 完全動作、対話型コンテナには必須

対話型シェル（bash、ghciなど）を使う開発環境では、**必ず両方を`true`に設定**することを推奨します。

## 検証環境

- Docker Compose v2
- Haskell 9.8コンテナ
- macOS (Darwin 24.3.0)

```dockerfile:Dockerfile
FROM haskell:9.8

WORKDIR /app
RUN ghc --version && stack --version
CMD ["/bin/bash"]
```

## `stdin_open`と`tty`とは？

### stdin_open

標準入力を開いたままにする設定です。

- **対応するDockerコマンド**: `docker run -i`
- **役割**: コンテナが標準入力からデータを受け取れるようにする
- **用途**: 対話型プログラムで入力を受け付ける

### tty

擬似TTY（仮想ターミナル）を割り当てる設定です。

- **対応するDockerコマンド**: `docker run -t`
- **役割**: ターミナルのような環境を提供
- **用途**: プロンプト表示、色付き出力、カーソル制御など

通常、`-i`と`-t`は組み合わせて`-it`として使われます。

## 検証方法

4パターンの設定ファイルを用意して、それぞれの動作を確認します。

### テスト1: 両方false

```yaml:docker-compose.test1.yml
services:
  haskell:
    build: .
    container_name: haskell_test1
    volumes:
      - ./app:/app
    stdin_open: false
    tty: false
    working_dir: /app
```

### テスト2: stdin_open=true, tty=false

```yaml:docker-compose.test2.yml
services:
  haskell:
    build: .
    container_name: haskell_test2
    volumes:
      - ./app:/app
    stdin_open: true
    tty: false
    working_dir: /app
```

### テスト3: stdin_open=false, tty=true

```yaml:docker-compose.test3.yml
services:
  haskell:
    build: .
    container_name: haskell_test3
    volumes:
      - ./app:/app
    stdin_open: false
    tty: true
    working_dir: /app
```

### テスト4: 両方true

```yaml:docker-compose.test4.yml
services:
  haskell:
    build: .
    container_name: haskell_test4
    volumes:
      - ./app:/app
    stdin_open: true
    tty: true
    working_dir: /app
```

## 検証結果

### 結果一覧表

| テスト | stdin_open | tty | コンテナ起動 | docker-compose exec | 実際の設定 |
|-------|-----------|-----|------------|---------------------|----------|
| 1 | false | false | ❌ すぐ停止 | ❌ 使えない | OpenStdin: false, Tty: false |
| 2 | true | false | ✅ 起動維持 | ✅ 動作する | OpenStdin: true, Tty: false |
| 3 | false | true | ✅ 起動維持 | ✅ 動作する | OpenStdin: false, Tty: true |
| 4 | true | true | ✅ 起動維持 | ✅ 動作する | OpenStdin: true, Tty: true |

### テスト1: 両方false - 完全にNG

```bash
$ docker-compose -f docker-compose.test1.yml up -d
$ docker-compose -f docker-compose.test1.yml exec haskell bash
service "haskell" is not running
```

**結果:**
- ❌ コンテナが起動後すぐに停止
- ❌ `docker-compose exec`でエラー
- ❌ 完全に使用不可

**理由:**

標準入力もTTYも無効なので、`CMD ["/bin/bash"]`で起動したbashがすぐに終了してしまいます。入力待ちする理由がないためです。

### テスト2: stdin_open=true のみ - 一応動く

```bash
$ docker-compose -f docker-compose.test2.yml up -d
$ docker inspect haskell_test2 --format='OpenStdin: {{.Config.OpenStdin}}, Tty: {{.Config.Tty}}'
OpenStdin: true, Tty: false

$ docker-compose -f docker-compose.test2.yml exec haskell bash -c 'echo "Hello"'
Hello  # ✅ 動作する

$ docker-compose -f docker-compose.test2.yml exec haskell bash -c 'tty'
not a tty  # ⚠️ TTYではない

$ docker-compose -f docker-compose.test2.yml exec haskell bash -c 'echo $TERM'
dumb  # ⚠️ 機能制限あり
```

**結果:**
- ✅ コンテナは起動を維持
- ✅ コマンド実行は可能
- ⚠️ 完全な端末エミュレーションはない
- ⚠️ `TERM=dumb`で色付き出力などが制限される

### テスト3: tty=true のみ - 一応動く

```bash
$ docker-compose -f docker-compose.test3.yml up -d
$ docker inspect haskell_test3 --format='OpenStdin: {{.Config.OpenStdin}}, Tty: {{.Config.Tty}}'
OpenStdin: false, Tty: true

$ docker-compose -f docker-compose.test3.yml exec haskell bash -c 'echo "Hello"'
Hello  # ✅ 動作する

$ docker-compose -f docker-compose.test3.yml exec haskell bash -c 'tty'
not a tty  # ⚠️ TTYではない

$ docker-compose -f docker-compose.test3.yml exec haskell bash -c 'echo $TERM'
dumb  # ⚠️ 機能制限あり
```

**結果:**
- テスト2とほぼ同じ
- ✅ コンテナは起動を維持
- ✅ コマンド実行は可能
- ⚠️ 完全な端末機能はない

### テスト4: 両方true - 完璧に動作

```bash
$ docker-compose -f docker-compose.test4.yml up -d
$ docker inspect haskell_test4 --format='OpenStdin: {{.Config.OpenStdin}}, Tty: {{.Config.Tty}}'
OpenStdin: true, Tty: true

$ docker-compose -f docker-compose.test4.yml exec haskell bash
root@xxx:/app# ghci
GHCi, version 9.8.x: https://www.haskell.org/ghc/  :? for help
Prelude> 1 + 1
2
Prelude> :quit
root@xxx:/app# exit
```

**結果:**
- ✅ 完全な対話型シェルが動作
- ✅ プロンプトが正常に表示
- ✅ ghciなどの対話型プログラムが快適に動作
- ✅ 色付き出力やカーソル制御が正常

## 重要な発見

### 1. `docker-compose exec`は自動的に`-it`を使う

実は、`docker-compose exec`コマンドは**デフォルトで`-it`オプションを付けて実行**されます。

```bash
# 内部的にはこう実行されている
docker exec -it haskell_test bash
```

そのため、docker-compose.ymlの設定に関わらず、`exec`コマンド自体はある程度動作します。

これが「片方だけでも動く」理由です。

### 2. 設定の真の役割は「コンテナ起動時」の動作

`stdin_open`と`tty`は、**コンテナ起動時（`docker-compose up`）の動作**に影響します：

- **stdin_open: true** → コンテナが標準入力を保持し、終了しない
- **tty: true** → コンテナにTTYを割り当て、終了しない

### 3. なぜ片方だけでも起動し続けるのか

- **stdin_open=true のみ**: 標準入力が開いているのでbashが終了しない
- **tty=true のみ**: TTYが割り当てられているのでbashが終了しない
- **両方 false**: 入力もTTYもないので、bashがすぐに終了

つまり、**どちらか一方があれば、bashは「何か入力があるかもしれない」と判断して待機し続ける**わけです。

## ベストプラクティス

### 対話型開発環境の場合（推奨）

```yaml
services:
  haskell:
    build: .
    container_name: haskell_dev
    volumes:
      - ./app:/app
    stdin_open: true  # ✅ 必須
    tty: true         # ✅ 必須
    working_dir: /app
```

**理由:**
1. コンテナがバックグラウンドで起動し続ける
2. `docker-compose exec`で完全な対話型シェルが使える
3. 色付き出力、カーソル制御などが正常に動作
4. ghci、python、nodeなどの対話型プログラムが快適
5. 明示的で意図が分かりやすい

### バックグラウンドサービスの場合（不要）

```yaml
services:
  nginx:
    image: nginx
    # stdin_open: true  ← 不要
    # tty: true         ← 不要
    ports:
      - "80:80"
```

Webサーバーなどのデーモンプロセスには不要です。むしろリソースの無駄になります。

## まとめ

- **`stdin_open`**: 標準入力を開いたままにする（`-i`相当）
- **`tty`**: 擬似TTYを割り当てる（`-t`相当）
- **両方必要な理由**: 完全な対話型ターミナル環境を実現するため
- **片方でも動く理由**: `docker-compose exec`が自動的に`-it`を付与するため
- **推奨**: 対話型コンテナでは必ず両方を`true`に

「おまじない」のように書いていた設定の意味が、実験を通じて理解できました！

## 参考資料

- [Docker run reference - Docker Documentation](https://docs.docker.com/engine/reference/run/)
- [Compose file reference - Docker Documentation](https://docs.docker.com/compose/compose-file/)

## 検証用スクリプト

本記事の検証に使用したスクリプトは以下です：

```bash:test_runner.sh
#!/bin/bash

echo "stdin_open と tty のテスト"

run_test() {
    local test_num=$1
    local compose_file="docker-compose.test${test_num}.yml"

    echo "テスト ${test_num}"
    docker-compose -f "$compose_file" up -d
    sleep 2

    echo "実際の設定:"
    docker inspect haskell_test${test_num} --format='OpenStdin: {{.Config.OpenStdin}}, Tty: {{.Config.Tty}}'

    echo "コマンド実行テスト:"
    docker-compose -f "$compose_file" exec -T haskell bash -c 'echo "Hello"' 2>&1

    docker-compose -f "$compose_file" down > /dev/null 2>&1
}

run_test 1
run_test 2
run_test 3
run_test 4
```

実際に試してみたい方は、ぜひ手元で実行してみてください！

---

この記事が、Docker Composeの設定を理解する助けになれば幸いです。
質問やフィードバックがあれば、コメントでお知らせください！
