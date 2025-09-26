# 第1章 ml5.jsの世界へようこそ

ml5.jsは、ブラウザ上で使えるp5.jsフレンドリーな機械学習ライブラリです。TensorFlow.jsを基盤にしており、複雑なニューラルネットワークの仕組みを知らなくても、シンプルなAPIでAI機能を扱えます。この章では、ml5.jsが解決できる課題や基本思想を理解し、実際にどのようなコードで動くのかを体験します。

## 1.1 機械学習をブラウザで
- **手軽さ**: ブラウザとJavaScriptだけでAIを扱えるため、インストールの手間やGPU環境が不要です。
- **リアルタイム性**: Webカメラやマイク入力を即座に解析できるため、インタラクティブ作品に向いています。
- **創造性**: アート・教育・プロトタイピングなど幅広い分野で活用が進んでいます。

## 1.2 ml5.jsが提供する代表的な機能
- 画像分類 (ImageClassifier)
- 姿勢推定 (PoseNet, BlazePose)
- 音声認識・分類 (SoundClassifier)
- 物体検出 (ObjectDetector)
- 特徴抽出と転移学習 (FeatureExtractor)

## 1.3 はじめてのml5.jsコード
以下は、静止画像を読み込んで何が写っているかを判定する最小限の例です。p5.jsを利用することで、描画と推論結果の表示を簡潔に書けます。

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <title>First ml5.js App</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.2/p5.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.2/addons/p5.dom.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/ml5/0.12.2/ml5.min.js"></script>
  </head>
  <body>
    <script>
      let classifier;
      let resultText = "モデルを読み込み中...";
      let inputImage;

      function preload() {
        // MobileNetベースの画像分類モデルをロード
        classifier = ml5.imageClassifier('MobileNet', () => {
          resultText = 'モデルの準備ができました！';
        });
        // 分類したい画像を読み込む
        inputImage = loadImage('images/sample-cat.jpg');
      }

      function setup() {
        createCanvas(640, 480);
        // モデル読み込みが終わったら、画像の分類を実行
        classifier.classify(inputImage, handleResult);
      }

      function draw() {
        background(240);
        image(inputImage, 0, 0, width, height);
        fill(0);
        textSize(20);
        text(resultText, 10, height - 20);
      }

      function handleResult(error, results) {
        if (error) {
          console.error(error);
          resultText = 'エラーが発生しました';
          return;
        }
        // 最も可能性が高いラベルと確信度を表示
        const { label, confidence } = results[0];
        resultText = `${label} (${(confidence * 100).toFixed(2)}%)`;
      }
    </script>
  </body>
</html>
```

## 1.4 コードのポイント
- `ml5.imageClassifier('MobileNet')` で、事前学習済みモデル MobileNet をロードしています。
- `classifier.classify()` は非同期処理で、結果はコールバックで受け取ります。
- p5.jsの `preload()` と `setup()` を利用すると、アセットの読み込みとキャンバス初期化を簡潔に分けられます。

## 1.5 次章への準備
次章では、サンプルを動かすためのディレクトリ構成、ローカルサーバの立て方、エディタ設定などを詳しく紹介します。事前に `images/sample-cat.jpg` のようなテスト画像を用意しておくとスムーズです。

## 練習問題
1. 上記サンプルで別の画像を分類するには、どのような修正が必要ですか？短い手順で説明してください。
2. 結果表示を日本語の文章にしたい場合、どのように `resultText` を加工するとよいでしょうか？
3. モデル読み込み中はキャンバスにローディングメッセージを表示するよう改造してください。必要なp5.jsの機能も併せて答えてください。

### 解答例
1. `loadImage()` に渡すファイルパスを変更し、同じ場所に新しい画像を配置する。必要なら `images` フォルダを更新する。
2. たとえば ``resultText = `これは${label}に見えます (確信度 ${(confidence * 100).toFixed(1)}%)`;`` のようにテンプレート文字列で文章を組み立てる。
3. `draw()` 内で `resultText` が初期値の間だけ `text("読み込み中...", 10, 30);` のように描画する。p5.jsの `text()` と条件分岐（`if` 文）を利用する。
