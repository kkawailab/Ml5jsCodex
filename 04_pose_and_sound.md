# 第4章 ポーズ推定と音声認識でつくるインタラクション

ml5.jsは視覚だけでなく、身体の動きや音声もリアルタイムに解析できます。この章では PoseNet を使った骨格推定と、SoundClassifier を使った音声分類を学び、インタラクティブな作品に応用する方法を紹介します。

## 4.1 PoseNet の仕組み
PoseNet は画像から人体の主要な関節位置（キーポイント）を検出するモデルです。ml5.jsを通じて簡単に利用でき、ダンス、ジェスチャー操作、フィットネスなどに応用されています。

### PoseNetの基本ステップ
1. Webカメラから動画を取得
2. `ml5.poseNet()` でモデルをロード
3. `pose` イベントで推定結果を受け取る
4. キーポイントと骨格ラインを描画

## 4.2 PoseNetのサンプルコード

```javascript
// public/sketch.js
let video;
let poseNet;
let poses = []; // 推定結果を格納

function setup() {
  createCanvas(640, 480);

  video = createCapture(VIDEO);
  video.size(width, height);
  video.hide(); // キャンバスに自分で描画するため非表示にする

  // PoseNetモデルをロード。シングル検出モードで精度を上げる
  poseNet = ml5.poseNet(video, { flipHorizontal: true }, () => {
    console.log('PoseNet準備完了');
  });
  // 新しいポーズが推定されるたびにコールバックが呼ばれる
  poseNet.on('pose', (results) => {
    poses = results;
  });
}

function draw() {
  image(video, 0, 0, width, height); // カメラ映像を表示
  drawKeypoints();
  drawSkeleton();
}

function drawKeypoints() {
  // 取得した全人物のキーポイントを描画
  for (const pose of poses) {
    for (const keypoint of pose.pose.keypoints) {
      if (keypoint.score > 0.5) { // 信頼度が十分高い点のみ描画
        fill(0, 255, 0);
        noStroke();
        ellipse(keypoint.position.x, keypoint.position.y, 12, 12);
      }
    }
  }
}

function drawSkeleton() {
  stroke(0, 150, 255);
  strokeWeight(3);
  for (const pose of poses) {
    for (const skeleton of pose.skeleton) {
      const [partA, partB] = skeleton;
      line(partA.position.x, partA.position.y, partB.position.x, partB.position.y);
    }
  }
}
```

### コードのポイント
- `flipHorizontal: true` を指定すると鏡のように左右反転し、ユーザが直感的に動きを把握できます。
- `poseNet.on('pose', ...)` で得られる `poses` 配列には、各人物のキーポイント座標とスコアが含まれます。

## 4.3 SoundClassifier で音声を認識
SoundClassifier は短い音声サンプルをリアルタイムでラベル分けします。既存の「SpeechCommands」モデルを使えば、「up」「down」など簡単な単語を識別できます。

```javascript
// public/sketch.js
let soundClassifier;
let label = 'Listening...';

function setup() {
  noCanvas(); // 今回はビジュアルを使わないためキャンバスは省略

  const options = {
    probabilityThreshold: 0.7, // この値以上の信頼度でラベルを返す
    includeSpectrogram: false,
  };

  // 事前学習済みモデル SpeechCommands18 をロード
  soundClassifier = ml5.soundClassifier('SpeechCommands18w', options, () => {
    console.log('SoundClassifier ready');
  });

  soundClassifier.classify((error, results) => {
    if (error) {
      console.error(error);
      return;
    }
    label = `${results[0].label} (${(results[0].confidence * 100).toFixed(1)}%)`;
    document.body.innerHTML = `<h1>${label}</h1>`; // 結果を画面に表示
  });
}
```

### 音声分類の注意点
- 初回利用時、ブラウザがマイク使用許可を求めるので許可する必要があります。
- `probabilityThreshold` を適切に調整することで誤認識を減らせます。
- ノイズが多い環境では外部マイクを使う、ヘッドセットを利用するなどの工夫が必要です。

## 4.4 ポーズと音声を組み合わせるアイデア
- 手を上げたら音声コマンドでメニューを決定する「ハンズフリーUI」
- 体育の授業で、正しいフォームのときだけ効果音を鳴らすトレーニングアプリ
- 踊りのステップをポーズで認識し、音声で次の技を案内するデモ

## 4.5 性能を引き上げるTips
- PoseNetでは `imageScaleFactor` や `minConfidence` を調整して速度と精度のバランスを取る。
- SoundClassifierでは結果の移動平均を取って、単発ノイズによる誤判定を抑える。
- イベントループが重いときは、`requestAnimationFrame` 内での処理を極力軽くする。

## 練習問題
1. PoseNetサンプルで両手の位置を判定し、左右の手が頭より上にあるときだけ背景を緑にするにはどうすればよいですか？ロジックの流れを答えてください。
2. SoundClassifierで特定の単語（例: "stop"）が検出されたときに音楽を停止するような機能を追加するには、どんなAPIやHTML要素を利用するとよいでしょうか？
3. ポーズ推定と音声分類を同時に動かすと処理が重くなりました。パフォーマンスを改善する戦略を2つ挙げてください。

### 解答例
1. `poses` の最初の要素から `pose.leftWrist`, `pose.rightWrist`, `pose.nose` などの `y` 座標を取得し、手の `y` が頭(鼻)より小さいときに `background(0, 255, 0)` を描画する。条件が満たされない場合は通常の `image(video, ...)` を描画する。
2. `<audio>` 要素をHTMLに配置し、`document.getElementById('player').pause()` のようにJavaScriptから制御する。SoundClassifierのコールバック内でラベルが"stop"になったときに `pause()` を呼ぶ。
3. (a) PoseNetの推論頻度を `setInterval` で落とす、または `singlePose` モードを使用する。(b) SoundClassifierの `probabilityThreshold` を上げて無駄なDOM更新を減らす。(c) Web Workerで重い処理を分離する。（いずれか2つ）
