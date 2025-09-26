# 第5章 特徴抽出と最終プロジェクトで仕上げる

ここまでで事前学習モデルを活用する方法を学びました。この章では、既存モデルの知識を再利用する「転移学習」で自分専用の分類器を作り、ミニプロジェクトにまとめ上げます。

## 5.1 FeatureExtractorとは
ml5.jsの `FeatureExtractor` は、MobileNetなどのモデルが持つ「特徴量」を流用し、少量のデータでも新しい分類器を短時間で学習できるAPIです。

メリット:
- 少ないデータでも高精度が得られる
- ブラウザ上で完結するため、プライバシーを守れる
- モデルの学習・推論をリアルタイムで体験できる

## 5.2 転移学習のワークフロー
1. FeatureExtractorを初期化
2. Webカメラや画像から学習用データを収集
3. 各ラベルに対応するサンプルを登録 (`classifier.addImage`)
4. `train()` で学習を実行
5. `classify()` で推論

## 5.3 サンプル: 表情で背景色を切り替える

```javascript
// public/sketch.js
let featureExtractor;
let classifier;
let video;
let loss;
let label = '未学習';

function setup() {
  createCanvas(640, 480);

  // 1. Webカメラを取得してキャンバスには描画のみ行う
  video = createCapture(VIDEO);
  video.size(width, height);
  video.hide();

  // 2. MobileNetをベースにした特徴抽出器を作成
  featureExtractor = ml5.featureExtractor('MobileNet', () => {
    console.log('FeatureExtractor ready');
  });

  // 3. 画像分類器を初期化。動画ストリームを入力データとして関連付け
  classifier = featureExtractor.classification(video, () => {
    console.log('Classifier ready');
  });

  createButtons();
}

function draw() {
  background(200);

  // 推論結果に応じて背景色を変更
  if (label === 'smile') {
    background(255, 223, 186); // 温かい色
  } else if (label === 'serious') {
    background(186, 225, 255); // 冷静な色
  }

  image(video, 0, 0, width, height);
  drawUI();
}

function createButtons() {
  // 各ラベル用のボタンを動的に作成
  const smileBtn = createButton('Smiling を追加');
  smileBtn.mousePressed(() => {
    // 現在のフレームを "smile" ラベルの学習データとして追加
    classifier.addImage('smile');
  });

  const seriousBtn = createButton('Serious を追加');
  seriousBtn.mousePressed(() => {
    classifier.addImage('serious');
  });

  const trainBtn = createButton('学習を開始');
  trainBtn.mousePressed(() => {
    classifier.train((lossValue) => {
      loss = lossValue;
      if (loss === null) {
        console.log('学習完了');
        classifyVideo();
      }
    });
  });
}

function classifyVideo() {
  classifier.classify((error, result) => {
    if (error) {
      console.error(error);
      return;
    }
    label = result[0].label; // 最も確信度の高いラベルを保存
    classifyVideo(); // 継続的に推論
  });
}

function drawUI() {
  fill(0, 170);
  rect(0, height - 100, width, 100);
  fill(255);
  textSize(20);
  textAlign(LEFT, TOP);
  text(`現在のラベル: ${label}`, 16, height - 90);
  text(`損失: ${loss !== undefined ? loss.toFixed(4) : '未計算'}`, 16, height - 60);
  text(`使い方: 各ボタンで学習画像を追加→学習開始`, 16, height - 30);
}
```

### 実装のポイント
- `classifier.addImage('smile')` を複数回呼び、各ラベルに最低でも20〜30枚のデータを集めると安定します。
- 学習完了後は `classifyVideo()` を繰り返し呼んで常に最新の判定を表示します。
- UIには `p5.dom` の `createButton()` を使用しています。

## 5.4 最終プロジェクト例: 手のジェスチャでBGMをコントロール
### 概要
- PoseNetで手の位置を取得
- FeatureExtractorで表情の状態を判別
- SoundClassifierで音声の「Start」「Stop」を受け付ける

3つの入力を組み合わせ、ジェスチャで音楽を再生/停止し、表情でボリュームを切り替えるインタラクティブアプリを作ります。

### 設計の流れ
1. **UI**: キャンバス、ステータス表示、音量スライダーを用意
2. **データ取得**: PoseNetとFeatureExtractor、SoundClassifierを初期化
3. **ルール定義**:
   - 両手が上がったらBGMを再生
   - SoundClassifierが「stop」を検出したら一時停止
   - 表情がsmileならボリュームを0.8、seriousなら0.3に設定
4. **状態管理**: 各モデルから得たラベルを統合し、1つの状態オブジェクトにまとめる

### 擬似コード
```javascript
const state = {
  handsUp: false,
  expression: 'serious',
  voiceCommand: null,
};

function updateAudio() {
  if (state.voiceCommand === 'stop') {
    bgm.pause();
  } else if (state.handsUp) {
    bgm.play();
  }

  bgm.volume = state.expression === 'smile' ? 0.8 : 0.3;
}
```

このように複数モデルの出力を統合することで、よりリッチな体験を提供できます。

## 5.5 モデルの保存と再利用
学習したモデルは `classifier.save()` でJSONとしてダウンロードできます。次回以降は `ml5.imageClassifier('model.json', video, callback)` のように読み込んで即座に推論を開始できます。

```javascript
// 学習済みモデルを保存するボタン例
const saveBtn = createButton('モデルを保存');
saveBtn.mousePressed(() => {
  classifier.save(); // ブラウザがJSONとbinファイルをダウンロード
});
```

保存したモデルを配布すれば、他のユーザも同じ分類器を共有できます。

## 練習問題
1. FeatureExtractorで3ラベル以上を扱う場合の注意点を2つ挙げてください。
2. 最終プロジェクトで音量制御を滑らかに行いたいとき、どのようなロジックを追加するとよいでしょうか？
3. 学習済みモデルを別ページで再利用する際、どのようなファイル構成とコード変更が必要になりますか？

### 解答例
1. (a) 各ラベルのサンプル数を均等に集める（偏りを避ける） (b) ラベル名にスペースや記号を含めない、または正規化して管理する。
2. ボリュームを直接切り替えるのではなく、`currentVolume = lerp(currentVolume, targetVolume, 0.1);` のように線形補間で徐々に変化させると耳障りになりにくい。
3. `models/` フォルダを作成し、`model.json` とウェイトファイルを配置。新しいページでは `ml5.imageClassifier('models/model.json', video, callback)` のように相対パスを変更し、必要なUIだけを付け替える。
