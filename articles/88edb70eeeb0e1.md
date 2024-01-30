---
title: "AndroidタブレットでJupyterLabする"
emoji: "🐥"
type: "tech"
topics:
  - "android"
  - "jupyternotebook"
  - "termux"
published: true
published_at: "2021-10-04 21:19"
---

## はじめに
最近huaweiのmatepad10.4ていうおもちゃタブレットを買ったのでこれでデータ分析勉強する環境つくればベッドで勉強するだろうから最強になれるって思ってやってみた。備忘録的に書いていく。

## やりたかったこと
- Android(正確にはHarmonyOS2)でも分析or開発環境作れる？
- Dockerさわってみる
- Azureの無料枠初めてしまったからなにかしたい
- データ分析勉強初めたい

## やったこと
- termuxセットアップ
- タブレットのなかでpython動かしたり重いことしたくないのでAzureVM(仮想サーバ)用意
- Docker-composeでJupyter動かす
- sshポートフォワードして仮想的にローカルで動いてる風にする。(セキュアらしい。本番環境用ではないので知らんけど)
- ブラウザからみてみる。
- ついでにcode-server動かしてます。気にしないでね

#### 1,termux
Termux is an Android terminal application and Linux environment.らしい。
以下公式githubから引用。Google playとかにあるアプリ
https://github.com/termux/termux-app

こいつでsshしたりするよ。
これをいれたらとりあえず
```bash:termux
termux-setup-storage
apt update && apt upgrade -y
```
とかしとこう。

#### 2,AzureVM
今回はAzureVMだったけど別に動かせるマシンあればなんでもいい。ubunntuだとして話は進める。
vmでもvpsでもいいのでsshします。
```bash:termux
ssh -i 鍵パス ユーザ@サーバのIPアドレス
```

#### 3,docker-composeでjupyter動かす
dockerとはよくわからんけどdockerイメージからコンテナつくることで環境がめちゃくちゃ楽にできる技術らしい。どういうやり方が本筋なのかはわからないけど、今回はついでにcode-serverも動かすのでdocker-composeでやる。ホームディレクトリにworkディレクトリとdocker-compose.ymlってファイルをつくる。
```bash:
mkdir work
vim docker-compose.yml
```
```bash:docker-compose.yml
version: "3"
services:
  notebook:
    image: jupyter/datascience-notebook
    ports:
      - "8888:8888"
    environment:
      - JUPYTER_ENABLE_LAB=yes
    volumes:
      - ./work:/home/jovyan/work
      - ./.ssh:/home/jovyan/.ssh
      - ./.gitconfig:/home/jovyan/.gitconfig
    command: start-notebook.sh --NotebookApp.token=''

  code:
    image: codercom/code-server
    ports:
      - "8080:8080"
    environment:
      PASSWORD: "pass"
    volumes:
      - ./work:/home/work
      - ./.ssh:/home/coder/.ssh
      - ./.gitconfig:/home/coder/.gitconfig
```
今回は以下２つの公式イメージってやつを使った。
jupyter datascience notebook
https://hub.docker.com/r/jupyter/datascience-notebook/
code-server
https://hub.docker.com/r/codercom/code-server
docker-compose.ymlの書き方解説は以下が分かりやすい。
https://qiita.com/yuta-ushijima/items/d3d98177e1b28f736f04

そしたら以下でdocker-compose起動する。
```bash:
docker-compose up -d --build
```
動いてるかな？てときは
```bash:
docker ps
```
すると動いてるプロセスがみれるみたいな感じだった。

#### 4,ssh port forward
```bash:termux
ssh -i 鍵パス -L 8888:Localhost:8888 ユーザ@サーバのIPアドレス
```
この　termuxをバックグラウンドで動かしといてブラウザでみてみる。

#### 5,ブラウザからみてみる
`http://localhost:8888`
![](https://storage.googleapis.com/zenn-user-upload/cd66b53d9cce28d0610e4955.jpg)
できた。
ついでにcode-serverの方は
`http://サーバのip:8080`
![](https://storage.googleapis.com/zenn-user-upload/4fd7fb2c284ce9a5c7c6a014.jpg)
docker-compose.ymlにかいてあるパスワードのpassってうつとできるよ。
たぶんcode-serverはwebサーバ機能があるうやろな

## おわり
code-serverのパスワード変えたりとかjupyterのtoken変えたりは調べたらすぐわかると思う。
振り返るとかなり簡単で、調べて試しながら前進していたあの感じは何だったんだろうなあ。

たぶんこれ用意するVMが強ければ分析環境強くなる。そのうちすんごい重い計算とかするときは数時間だけEC2とかのGPUコンピューティングできる借りてやったりするんかな。