# 第3章 画像分類で学ぶプリトレインドモデル

この章では、ml5.jsが提供する代表的な機能「ImageClassifier」を使って画像を判定する方法を学びます。静止画とWebカメラ映像を扱い、実際のアプリに組み込む際のポイントを理解しましょう。

## 3.1 画像分類の基本フロー
1. モデルのロード (`ml5.imageClassifier`)
2. 推論対象の画像データを取得（`loadImage` や `video` 要素）
3. `classify()` メソッドにデータを渡して推論
4. 結果をUIに反映

ml5.jsのImageClassifierはMobileNetやDarknetなど複数のモデルをサポートしています。本章では精度とサイズのバランスが良いMobileNetを使用します。

## 3.2 静止画像を分類するサンプル

```javascript
// public/sketch.js
let classifier; // 画像分類モデルを格納する変数
let img; // 推論対象の画像
let label = '読み込み中...'; // 表示するラベル
let confidence = 0; // 推論の確信度

function preload() {
  // 1. MobileNetをベースにした事前学習済みモデルをロード
  classifier = ml5.imageClassifier('MobileNet', () => {
    console.log('モデルのロードが完了しました');
  });
  // 2. ローカルの画像を読み込む（CDNの画像URLでも可）
  img = loadImage('assets/images/dog.jpg');
}

function setup() {
  createCanvas(640, 480);
  // 3. 画像の分類を実行。結果はhandleResultで処理
  classifier.classify(img, handleResult);
}

function draw() {
  background(255);
  image(img, 0, 0, width, height); // 読み込んだ画像をキャンバス全体に表示

  fill(0, 170);
  rect(0, height - 60, width, 60); // 結果表示用の半透明パネル

  fill(255);
  textSize(20);
  textAlign(LEFT, CENTER);
  text(`推論結果: ${label}`, 16, height - 30);
  text(`確信度: ${(confidence * 100).toFixed(2)}%`, 16, height - 6);
}

function handleResult(error, results) {
  if (error) {
    console.error('推論中にエラー', error);
    label = 'エラーが発生しました';
    confidence = 0;
    return;
  }
  // 4. 最も確信度の高い結果を保存
  label = results[0].label;
  confidence = results[0].confidence;
}
```

### コードの要点
- `loadImage()` は `preload()` 内で呼び出すと非同期読み込みを安全に扱えます。
- `results` は確信度の高い順に並んだ配列です。`results[0]` に最有力候補が入っています。

## 3.3 Webカメラからリアルタイム分類
静止画像だけでなく、Webカメラの映像を逐次解析して結果を更新できます。

```javascript
// public/sketch.js
let classifier;
let video;
let currentResult = '準備中...';

function setup() {
  createCanvas(640, 480);

  // 1. ビデオ入力を作成し、キャンバスとは別にDOM上へ配置
  video = createCapture(VIDEO, () => {
    console.log('Webカメラのストリームを取得しました');
  });
  video.size(640, 480);
  video.hide(); // p5.jsのキャンバスに手動で描画するため非表示にする

  // 2. MobileNetをロードし、完了したら classifyVideo を呼び出す
  classifier = ml5.imageClassifier('MobileNet', () => {
    console.log('モデル準備完了');
    classifyVideo();
  });
}

function draw() {
  background(0);
  image(video, 0, 0); // キャンバスに最新のフレームを描画

  fill(0, 150);
  rect(0, height - 60, width, 60); // 下部にステータスバーを描画

  fill(255);
  textSize(22);
  textAlign(CENTER, CENTER);
  text(currentResult, width / 2, height - 30);
}

function classifyVideo() {
  // 3. 現在の動画フレームを分類し、結果が出たら再度呼び出す
  classifier.classify(video, (error, results) => {
    if (error) {
      console.error(error);
      currentResult = 'エラーが発生しました';
      return;
    }
    const { label, confidence } = results[0];
    currentResult = `${label} (${(confidence * 100).toFixed(1)}%)`;
    classifyVideo(); // 再帰的に呼んで常に最新の結果を表示
  });
}
```

### 安定動作のコツ
- WebカメラへのアクセスにはHTTPSが必要です。ローカル環境では `localhost` であれば例外的に許可されます。
- 処理負荷が高い場合、`classifyVideo()` を `setTimeout` で数百ミリ秒遅延させるとCPU使用率を抑えられます。

## 3.4 結果の可視化とインタラクション
- **ラベルごとの色分け**: 推論結果に応じて背景色を変えることで、直感的なフィードバックが得られます。
- **履歴の保持**: `results` を配列に保存して折れ線グラフ化すると、分類の変化を追跡できます。
- **音声と連携**: 手を写したら音が鳴るなど、他のml5機能と組み合わせると表現の幅が広がります。

## 3.5 ありがちなエラーと対処法
- **CORSエラー**: 画像を外部URLから読み込む場合はCORSが許可されたサーバを利用するか、自分でホスティングする。
- **モデル未ロード**: `classify()` を呼ぶタイミングが早いとエラーになる。ロード完了のコールバック内で呼ぶ。
- **推論結果が不安定**: Webカメラのフレームが暗い場合は照明を整える、`video.size()` で解像度を下げて速度を上げる。

## 練習問題
1. 静止画像サンプルで、結果をキャンバス上ではなくHTMLの `<div>` に表示するにはどのような変更が必要ですか？
2. Webカメラ分類で推論の頻度を0.5秒ごとにしたい場合、どのように `classifyVideo()` を書き換えるとよいでしょうか？
3. 複数のモデル(MobileNetとDarknet)を切り替えて使いたいとき、どのようなUIとコード構成が考えられますか？

### 解答例
1. HTML側に `<div id="result"></div>` を用意し、`handleResult()` 内で `document.getElementById('result').textContent = ...` を設定する。p5.js `draw()` 内でのテキスト描画は不要になる。
2. `classifyVideo()` 内で `setTimeout(classifyVideo, 500);` を使って再帰呼び出しの間隔を空ける。または `setInterval` を利用して一定周期で分類する。
3. セレクトボックスやボタンでモデル名を選択し、選択が変わったら `ml5.imageClassifier(selectedModel, callback)` で新しいインスタンスを生成してから推論を再開する。モデルごとにインスタンスをキャッシュすると切り替えが速くなる。
