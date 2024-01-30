---
title: "Notion API・Lambdaで自動入力食べログ専用ブックマークマネージャ作った"
emoji: "🍕"
type: "tech"
topics:
  - "python"
  - "lambda"
  - "api"
  - "notion"
  - "bookmark"
published: true
published_at: "2022-01-31 08:55"
---

## いい店データベースを作りたい
自分専用/友人たちといい店情報をため込んでおくといろいろと楽しそう。

やってみたらできたので備忘録を兼ねてアウトプットします。

今回はすべてpython3.9

## 概要
### 完成図
ユーザーがやることは以下二つのみ
- iPhoneからブラウザで食べログの店ページを開く。共有からショートカットを選択する。だけでブックマーク追加
![](https://storage.googleapis.com/zenn-user-upload/57ab57666945-20220131.jpg)

- いろいろ追加されてできたNotionのページを見れる
![](https://storage.googleapis.com/zenn-user-upload/bac3226fc79f-20220131.png)
![](https://storage.googleapis.com/zenn-user-upload/aa4eb8642b97-20220131.png)
![](https://storage.googleapis.com/zenn-user-upload/fd791640accd-20220131.png)
Notionのデータベースすごくて、表示も何種類かあって、ソートしたり検索したりもできる。

以下リンクでNotionテンプレートとして公開したので見てみてね。
https://onyx-fruit-c1a.notion.site/MyBookMark-for-TabeLog-TEMPLATE-b6a9ec1b742f47209fda544827323f32

### 実現する方法としては以下
- iPhoneから食べログのurlをlambda上の自作APIにPOST
- Lambdaでurlからスクレイピングして必要情報を取り出す
- いい感じに作ってあるデータベースにNotion APIたたいて追加
- notionアプリから見る
非常に適当ですが以下図の感じ
![](https://storage.googleapis.com/zenn-user-upload/53d17a58078f-20220131.jpg)

---
## やったこと
- 1, Notionでいい感じにデータベースを作る
- 2, LambdaにAPIを置く
	- 2.1, スクレイピング
	- 2.2, Notion API
	- 2.3, Lambda
- 3, iPhoneからパパっとAPIをたたけるショートカットを設定 
- 4, 友人たちに共有

### 1, Notionでいい感じにデータベースを作る
上記画像/テンプレートみたいなページをNotionで作った

impressionとtypeは自分でどんな感じか/どんな時に使いたい店なのかとかを入力する仕様にした。

点数評価はあえて拾ってない。自分や友達の声のほうがずっと信用できるので

### 2, LambdaにAPIを置く
ここはNotion APIを触ってみながら、スクレイピングできそうな項目探しながら中身の関数を考えて、lambdaのREST APIはpostmanでたたいてみながらやった。

今回は以下三点について参考文献とハマった点を書くことにする。
- スクレイピング
- Notion API
- Lambda

#### 2.1, スクレイピング
スクレイピングについては以下urlを参照しながらやってみた。beautifulsoup4使った。
https://qiita.com/Chanmoro/items/db51658b073acddea4ac

https://developer.mozilla.org/ja/docs/Web/API/Document_Object_Model/Introduction

#### 2.2, Notion API
Notion APIについては公式のドキュメントが非常にわかりやすかった。notion-sdk-pyが非常に便利。Yusuke Sato氏のZenn記事も非常に参考になった。

各オブジェクトにアクセスする際に親要素のidが必要になったりする点に注意。
https://developers.notion.com/

https://github.com/ramnes/notion-sdk-py

https://zenn.dev/ysksatoo/articles/66fd26893a6cdd

#### 2.3, Lambda
Lambdaに上記二つを満たす関数をおいて、API GatewayをトリガーとしてREST APIとした。
以下に注意点的なことを数点書く
- lambda_handler関数の中で、リクエストボディは辞書dict型で変数eventに入ってる
- APIキーはリクエストヘッダーのx-api-keyとして入れて送る
- requests等moduleはzipファイルにしてレイヤーとして設定しないとimportできない
- 環境変数はawsのページからGUIで設定できる

https://qiita.com/miyuki_samitani/items/f01f1bd49334f97fe84c

https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/api-gateway-setup-api-key-with-console.html

https://aws.amazon.com/jp/premiumsupport/knowledge-center/lambda-import-module-error-python/

恥ずかしいけどLmabda関数をgithubにざっくり乗せといた。参考にはならない気がするけど置いておきます。これどういうこと？みたいなことがあったらガシガシ聞いてほしい。
https://github.com/xianglishan/bs4_notion-client_test

### 3, iPhoneからパパっとAPIをたたけるショートカットを設定
上記で作ったAPIをたたく用のクライアントの話。

test環境みたいにpostmanでたたいてもいいけど、iPhoneから手軽に一瞬でやりたいのでiPhnoeのショートカットを使う。iPhoneからAPIたたくベストプラクティスではないかもしれない。これいいよって方法があったら教えてください。

こちらです。
https://apps.apple.com/jp/app/%E3%82%B7%E3%83%A7%E3%83%BC%E3%83%88%E3%82%AB%E3%83%83%E3%83%88/id915249334

これいろいろできるらしい。今回使うのは
- 共有シートからURLの入力を受け取る
- URLの内容を取得
![](https://storage.googleapis.com/zenn-user-upload/b9c7e7731456-20220131.jpg)

こんな感じに設定する。これはメソッド/ヘッダー/ボディを入力してurlにhttpリクエストしてる。

つまりどうやらlinuxのコマンドでいう`curl`してる。

curlした後は'ok!posted!!'てテキスト表示させたり、Notionアプリ開かせたりするように設定した。

今回作成したショートカットを、「共有」から「アクションの編集」で「よく使う項目」として設定すれば上のほうに出てきて便利

### 4, 友人たちに共有
Notionはコラボレーションワークスペースなので個人プランでも共有が簡単。右上の共有ボタンからユーザー名とかメールアドレスで共有

ショートカットもiPhoneユーザー間なら共有可能

あとはみんなでいい店を寄り合っていい店データベースを作り上げる

---
## 完成!!!
おおよそやりたいことはできたが以下懸念点が残る
- Notion APIはオープンベータなので仕様変更がある場合要修正
- 営業時間変更、お店がなくなる等の情報をとってこれない
- 同じ店を何回も追加できてしまう
月一とかでurlをクロールして情報の最新化とかをしていきたい

感想
- postmanめちゃくちゃ便利
- Notion APIめちゃくちゃ便利


以上です。どこかの誰かに刺されば嬉しいです。