# Model Asset Exchange のモデルを使用してヨガのポーズを認識する Web アプリを作成する
### Model Asset Exchange のモデルを使用してヨガのポーズを認識する Web アプリを作成する

English version: https://developer.ibm.com/patterns/build-a-web-application-that-recognizes-yoga-poses-using-a-model-from-the-model-asset-exchange

ソースコード: https://github.com/IBM/yogait

##### 最新の英語版コンテンツは上記URLを参照してください。
last_updated: "2019-10-03"

## 概要

この IBM Developer コード・パターンでは、Model Asset eXchange に用意されている人体姿勢推定器モデルを使用して、所定の画像から各個人のポーズを検出する方法を説明します。座標を使用しながら、モデルによって作成されたポーズ・ラインを、画像から検出されたそれぞれの人の全身ポーズに組み立てていきます。

## 説明

[人体姿勢推定器](https://developer.ibm.com/jp/exchanges/models/all/max-human-pose-estimator/)モデルは、所定の画像から各個人とそのポーズを検出します。このモデルはまず、入力された画像から人を検出し、その個人の身体のパーツ (鼻、首、目、肩、ひじ、手首、腰、ひざ、足首など) を識別します。次に、「ポーズ・ライン」によって関連する身体のパーツの各ペアを結び付けます。以下の画像に示されているように、ある線で左目を鼻に結び付け、別の線で鼻を首に結び付けるといった具合です。

![ポーズ・ラインの例](./images/pose-lines.png)

ポーズ・ラインはそれぞれ [x1, y1, x2, y2] のリストで表されます。最初の座標ペア (x1, y1) が身体の 1 つのパーツの開始点であり、2 番目の座標ペア (x2, y2) が終了点、かつ、そのパーツに関連する他のパーツの開始点となります。このように複数のポーズ・ラインが結び付けられて、画像内で検出された各個人の全身ポーズが組み立てられます。

このモデルのベースとなっているのは、[OpenPose モデル](https://github.com/ildoonet/tf-pose-estimation)の TF 実装です。このリポジトリー内のコードが、モデルを Web サービスとして Docker コンテナー内にデプロイします。このリポジトリーは、[IBM Developer Model Asset eXchange](https://developer.ibm.com/jp/exchanges/models/) の一部として開発されたものです。

Yogait というヨガ・アシスタントは、人体姿勢推定器 MAX モデルを使用して、ユーザーがとっているヨガのポーズを推定します。ポーズの分類には事前トレーニング済みの SVM を使用していますが、Yogait は MAX モデルが返すデカルト線ではなく、極座標表現を使って分類を行います。これによって、ポーズの分類が大幅に容易になります。x-y 座標系で SVM をトレーニングすると、データを増補する際に並進と回転が必要になる一方で、極座標表現は、推定されたモデルの中心を基準とした関節の相対位置に依存するだけだからです。

各関節の [x,y] 座標が [phi, rho] に変換されます。

![ポーズの変換](./images/pose-conversion.jpg)

SVM は、極性ベクトルをフラット化したバージョンに対して分類を行います。デカルト表現と比べ、この極座標表現はわずかしかデータを使用しないため、キャプチャーされたフレームのどの部分でも人間を分類できます。デカルト表現を使用する場合、ポーズ全体をカメラフレームの中央に位置させなければなりません。

このコード・パターンを完了すると、以下の方法がわかるようになります。

* 人体姿勢推定器 MAX モデルの Docker イメージをビルドする
* REST エンドポイントが設定された深層学習モデルをデプロイする
* MAX モデルの REST API を使用して、動画のフレーム内の人間のポーズを推定する
* モデルの REST API を使用する Web アプリケーションを実行する

## フロー

![Yogait のフロー・アーキテクチャーを示す図](./images/flow-diagram-yogait.png)

1. サーバーが Web カメラからキャプチャーした動画を 1 フレームずつモデル API に送信します。
1. Web UI がサーバーに対し、フレームに対して推定されたポーズ・ラインをリクエストします。
1. サーバーがモデル API からデータを受信し、その結果を反映して Web UI を更新します。

## 手順

このパターンの詳細な手順については、[README](https://github.com/IBM/yogait/blob/master/README.md) を参照してください。手順の概要は以下のとおりです。

1. MAX モデルをセットアップします。
1. Web アプリを起動します。
1. Python スクリプトを使用してローカルで実行します。
1. Jupyter Notebook を使用します。