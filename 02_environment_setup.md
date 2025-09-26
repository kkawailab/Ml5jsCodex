# 第2章 開発環境の準備とHello ml5.js

ml5.jsはブラウザで動くライブラリですが、効率的に開発するにはフォルダ構成やローカルサーバなどの準備が欠かせません。この章では基本的なディレクトリの作り方から、最初の「Hello ml5.js」を表示するまでを段階的に学びます。

## 2.1 必要なソフトウェア
- 任意のコードエディタ（VS Code、WebStorm、Vim など）
- Node.js (v18 以上を推奨) — ローカルサーバを動かすために使用
- 任意のブラウザ（Chrome, Firefox, Safari など）

## 2.2 プロジェクト構成のテンプレート
以下のようなフォルダ構成にすると、サンプルやアセットを整理しやすくなります。

```
ml5-tutorial/
├── package.json
├── public/
│   ├── index.html
│   ├── sketch.js
│   └── assets/
│       ├── images/
│       └── sounds/
└── README.md
```

- `public/` 配下にブラウザから参照されるファイルをまとめます。
- `assets/` は画像・音声などの素材を種類ごとにフォルダ分けします。

## 2.3 ローカルサーバの起動
`live-server` や `http-server` などを利用すると、ホットリロード付きで手軽に開発できます。

```bash
# 1. プロジェクトフォルダを作成
mkdir ml5-tutorial
cd ml5-tutorial

# 2. パッケージマネージャを初期化
yarn init -y  # npm init -y でも可

# 3. http-server を開発用に導入
yarn add -D http-server

# 4. package.json にスクリプトを追加
# "scripts": {
#   "start": "http-server public -o"
# }

# 5. サーバ起動
yarn start
```

- `-o` オプションを付けるとブラウザが自動で開きます。
- Node.js を使わずに VS Code の「Live Server」拡張や `python -m http.server` を用いる方法もあります。

## 2.4 Hello ml5.js のサンプル
`public/index.html` と `public/sketch.js` の2ファイルで最小構成を実現できます。

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <title>Hello ml5.js</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.2/p5.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/ml5/0.12.2/ml5.min.js"></script>
    <script defer src="sketch.js"></script>
    <style>
      body { margin: 0; font-family: "Helvetica Neue", Arial, sans-serif; }
      #status { padding: 16px; background: #f7f7f7; border-bottom: 1px solid #ccc; }
    </style>
  </head>
  <body>
    <div id="status">ml5.js を読み込み中...</div>
  </body>
</html>
```

```javascript
// public/sketch.js
let mobileNet;

function setup() {
  createCanvas(640, 360); // 640x360 のキャンバスを生成
  background(30, 144, 255); // 背景を空色に設定
  fill(255); // テキストの塗りつぶし色を白に
  textSize(28); // テキストサイズを設定
  textAlign(CENTER, CENTER); // テキストの基準位置を中央に
  text('Hello ml5.js!', width / 2, height / 2);

  // MobileNet モデルをロードし、準備ができたらステータスを更新
  mobileNet = ml5.imageClassifier('MobileNet', () => {
    const status = document.getElementById('status');
    status.textContent = 'ml5.js の読み込みが完了しました！';
  });
}
```

### コード解説
- HTML では CDN から p5.js と ml5.js を読み込み、JavaScript を defer で後から読ませています。
- `createCanvas()` で p5.js のキャンバスを作成し、その上にテキストを描画しています。
- `ml5.imageClassifier()` はモデルを読み込むだけで即座に推論は行いません。読み込み完了のコールバックで任意の処理を記述できます。

## 2.5 より快適な開発のためのTips
- **自動フォーマッタ**: Prettier や ESLint を導入してコードスタイルを保ちましょう。
- **アセット管理**: 画像やモデルファイルは `assets/` 配下にまとめ、ファイル名に日付やバージョンを含めると混乱しにくくなります。
- **バージョン固定**: CDN のURLには明示的にバージョン番号を指定すると、突然の breaking change を避けられます。

## 練習問題
1. `http-server` の代わりに VS Code の Live Server 拡張を使う場合、どのような利点がありますか？2つ答えてください。
2. `public` フォルダ以外をブラウザから見せたくない場合、どんな点に注意すべきでしょうか？
3. Hello ml5.js のサンプルに背景色をクリックで切り替える処理を追加してください。どのp5.jsイベントを利用すると良いかも答えてください。

### 解答例
1. (a) ボタン1つでサーバを起動でき、設定が不要 (b) ファイル保存時に自動でブラウザをリロードできる。
2. Node.js の静的サーバ設定で `public/` だけを公開し、上位フォルダに機密情報を置かない。意図しない `..` パスを許可しない。
3. `mousePressed()` イベントを定義し、クリックされるたびに `background()` の色を変更するようにする。状態管理にはグローバル変数を使う。
