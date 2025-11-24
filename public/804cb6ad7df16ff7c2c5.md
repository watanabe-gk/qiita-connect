---
title: Docker Python環境構築 (言語処理100本ノック対応)
tags:
  - Python
  - Git
  - Docker
  - 言語処理100本ノック
private: false
updated_at: '2024-12-05T15:36:40+09:00'
id: 804cb6ad7df16ff7c2c5
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
PythonをDocker上で動かせるリポジトリを作成しました。

https://github.com/watanabe-gk/docker_python


## 構築済みのもの
numpyやscikit-learn等のオーソドックスなものはもちろん、[言語処理100本ノック 2020 (Rev 2)](https://nlp100.github.io/ja/)でも活用したので、MeCab-Cabochaも構築済みです（これローカルでやろうとしたら結構面倒くさい）。

## 使い方
各勉強項目ごとにscripts配下でディレクトを切ってもらえたら使いやすいのかのと思います。
キカガクさんの勉強したときは `/scripts/kikagaku`
言語処理100本ノック 2020 (Rev 2)の勉強したときは `/scripts/languageProcessing100Knocks`
みたいな感じ。

使用例
```
.
├── README.md
├── docker
│   ├── CRF++-0.58.tar.gz
│   ├── Dockerfile
│   ├── cabocha-0.69.tar.bz2
│   └── requirements.txt
├── docker-compose.yml
└── scripts
    ├── kikagaku
    │   ├── histogram.png
    │   ├── howToUse_Numpy.py
    │   ├── howToUse_Pandas.py
    │   ├── matrix.py
    │   ├── modelRead.py
    │   ├── models
    │   │   └── multipleRegressionAnalysisModel.pkl
    │   ├── multipleRegressionAnalysis.py
    │   ├── pairplot.png
    │   ├── samples
    │   │   ├── housing.csv
    │   │   └── simpleRegressionAnalysisSample.csv
    │   ├── scatter_plot.png
    │   ├── scatter_plot_centered.png
    │   └── utils
    │       ├── __pycache__
    │       │   ├── util.cpython-313.pyc
    │       │   └── util.cpython-39.pyc
    │       └── util.py
    └── languageProcessing100Knocks
        └── 1
            ├── 0.py
            ├── 1.py
            ├── 2.py
            ├── 3.py
            ├── 4.py
            ├── 5.py
            ├── 6.py
            ├── 7.py
            ├── 8.py
            └── 9.py
```
# 自分の勉強用としてgit管理
1. github gitlab 等でリポジトリ作成
1.  リポジトリの初期化
    ```
    rm -rf .git
    ```
1. 新しいリポジトリを初期化
    ```
    git init
    ```
1. すべてのファイルをステージング
    ```
    git add .
    ```
1. コミットを作成
    ```
    git commit -m "Initial commit"
    ```
1. リモートリポジトリの設定
    ```
    git remote add origin <リモートリポジトリのURL>
    ```

1. 最初のプッシュ
    ```
    git push -u origin main
    ```
おしまい！

# 備考
pipでinstallできるものは **requirements.txt** で追記するだけで済みます。
Mecab-Cabochaの環境構築は本来 [CRF++](https://drive.google.com/drive/folders/0B4y35FiV1wh7fngteFhHQUN2Y1B5eUJBNHZUemJYQV9VWlBUb3JlX0xBdWVZTWtSbVBneU0?resourcekey=0-NW5cPRv1Xr2-Vfo_xlDTLQ)と[Cabocha](https://drive.google.com/drive/folders/0B4y35FiV1wh7cGRCUUJHVTNJRnM?resourcekey=0-ym0BJTHMkjw3y1AEgwwaxA)をダウンロードしたものを解凍してモジュールインストール。
とか他にもやることが多いので最新版をダウンロード済みです(/docker配下の **cabocha-0.69.tar.bz2** と **CRF++-0.58.tar.gz**)。それを解凍からモジュールインストールまで行ってるので環境構築で挫折、ということはなるべく省けるようになっていると思います。
