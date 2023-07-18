---
title: "プログラムの流れ"
free: true
---
次のプログラムが[p5.js Web Editor](https://editor.p5js.org/)を開いたときに表示されます。

```js
function setup() {
  createCanvas(400, 400);
}

function draw() {
  background(220);
}
```

プログラムは「基本的には」上から下へと実行されます。[^1]

drawの中の`background(220);`の下に、いくつか行を追加してみましょう。

```js
function draw() {
  triangle(240, 180, 130, 360, 350, 360);
  background(220);
  square(170, 150, 170, 10);
  rect(120, 40, 160, 150);
  circle(120, 180, 180);
}
```

:::details 何が表示されるでしょうか？
![](/images/books/p5js-for-kids/function-figs.png)
:::

draw中の各行の順番を変えて、表示がどう変わるか試してみましょう。

`triangle`, `square`, `rect`, `circle`はそれぞれ三角、丸四角、四角、円を書くための関数です。こうした関数はメニューのヘルプ > [リファレンス（英語）](https://p5js.org/reference/)で調べることができます。

関数は`function`というキーワードを使って定義することができます。つまり、setupとdrawも関数です。
setupやdrawは定義する関数につける名前です。そして、中括弧`{`, `}`の間に関数が呼ばれたときに実行される文の列を書きます。`background(220)`のように関数名に`()`とその関数に渡す値を書くことで関数を呼び出すことができます。[^2]

squareとrectの呼び出しを一つの関数にしてみましょう。

```js
function sikakuKyoudai() {
    rect(120, 40, 160, 150);
    square(170, 150, 170, 10);
}

function draw() {
    background(220);
    circle(120, 180, 180);
    sikakuKyoudai();
    triangle(240, 180, 130, 360, 350, 360);
}
```

このように関数は自由に作成することができます。

JavaScriptでは、関数を定義する順序には決まりがありません。上のプログラムでsikakuKyoudaiをdrawの後で定義しても表示は変わりません。プログラムは上から下に読み込まれ実行されていくのですが、`function`は関数を定義するだけで、関数を呼び出すわけではありません。`{`の`}`の中は関数が呼び出されるまで実行されません。

### まとめ

- プログラムは上から下に実行されることを確かめました。
- `function`を使ってオリジナルの関数を定義し、それを呼び出しました。

### チャレンジ問題

次の絵を描けるかな？

![](/images/books/p5js-for-kids/function-challenge.png)

:::details ヒント
円の一部だけを描く関数があるよ。[arc](https://p5js.org/reference/#/p5/arc)
:::


### さらに気になる人向け

僕たちの書いたプログラムがsetupとdrawの関数定義しかしていないのに、画面に図形が表示されるのはどうしてか気になって夜も眠れないという人向けに追加の説明です。

p5.jsがブラウザの上で動いているので、調べようにもウェブサーバー上のどこか知らないところで起きているというのであればしかたがないのですが、君が書いたp5.jsのプログラムは君が使っているコンピュータの中で処理されています。それを確かめましょう。

実行ボタンの下に > というボタンがあります。

![](/images/books/p5js-for-kids/function-navigation.png)

これをクリックすると、スケッチファイルが表示されます。

![](/images/books/p5js-for-kids/function-sketch-files.png)

いままで見ていたのは`sketch.js`というファイルでした。`index.html`を選んでみましょう。（`1.7.0`という数字は違う数字になっているかもしれません。）

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/addons/p5.sound.min.js"></script>
    <link rel="stylesheet" type="text/css" href="style.css">
    <meta charset="utf-8" />

  </head>
  <body>
    <main>
    </main>
    <script src="sketch.js"></script>
  </body>
</html>
```

これは[HTML](https://developer.mozilla.org/ja/docs/Web/HTML)という形式で書かれたファイルです。HTMLはウェブページを記述するためのもので、いつも見ているウェブページはウェブブラウザがこういったHTMLファイルを読み込んで表示しています。p5.js Web Editorでは、まさにこのHTMLファイルを読み込んでプレビューのところに表示しているのです。

試しに`<main>`と`</main>`の間に`こんにちは！`と書いてみましょう。

```html
  ...
  <body>
    <main>
      こんにちは！
    </main>
    <script src="sketch.js"></script>
  </body>
</html>
```

プレビューに「こんにちは！」と表示されましたか？（実行してくださいね）

ブラウザは確かにこのファイルを読み込んでいます。では、setupとdrawを呼び出しているのはどこでしょうか？

`<script src="sketch.js"></script>`という行があります。これはさっきまで編集していたsketch.js を読み込んでいるという意味です。そういえば、上の方にも`<script ...`という行がありました。ここで指定されている [https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.js](https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.js) をブラウザーで新しいタブを開いてみてみましょう。

ものすごく長いプログラムが表示されました。こんな長いプログラムは読めないと思いますよね。安心してください。このプログラムは人間が書いたもっと分かりやすいプログラムをもとに機械的に一つのファイルにまとめたものです。では、その元のプログラムはどこにあるのでしょうか？

p5.jsの[ホームページ](https://p5js.org/)をよく見ると、ページの下の方にGitHubというリンクがあります。p5.jsは[オープンソース](https://ja.wikipedia.org/wiki/%E3%82%AA%E3%83%BC%E3%83%97%E3%83%B3%E3%82%BD%E3%83%BC%E3%82%B9)でコードは公開されています。では、そのGitHubのp5.jsのページを開いてみましょう。

https://github.com/processing/p5.js

先程の長い長いコードはここのコードから生成されたものです。srcディレクトリーをみてみましょう。

[app.js](https://github.com/processing/p5.js/blob/552cdad6faa82b04f219c0f5f63ebea3fd81da15/src/app.js)というファイルがトップにあります。その中は大量に`import`というキーワードが出てきます。これは他のファイルを読み込んでいるという意味です。最後のimportは`core/init`です。プログラムは上から下に実行されるんでしたね。すべての準備が終わって最後に実行されるのが`core/init`ということです。これかなという気がしますね。開いてみましょう。

[core/init.js](https://github.com/processing/p5.js/blob/552cdad6faa82b04f219c0f5f63ebea3fd81da15/src/core/init.js)

`const _globalInit = () => {`は`function _globalInit() {`とほぼ同じです。つまり、これも関数定義であって誰かが呼び出さないと実行されません。しかし、最後までみていくと関数定義ではない行があります。

```js
Promise.all([waitForDocumentReady(), waitingForTranslator]).then(_globalInit);
```

この行をブラウザが読み込んだとき、当然、ブラウザはこの行を実行します。しかし、僕たちの書いたsketch.jsはHTMLファイルでずっと下に書いてありました。つまり、まだ読み込まれていません。僕たちの書いたsetupもdrawもまだ定義されていませんので、p5.jsのプログラムを動かし始めても何も表示されません。そこで、ここでは他のスクリプトなどが全部読み込み終わるのを待ってから_globalInitを呼び出すというテクニックが使われています。

_globalInitは先程、定義されていました。定義をみてみると最後にこう書かれています。

```js
    if (
      ((window.setup && typeof window.setup === 'function') ||
        (window.draw && typeof window.draw === 'function')) &&
      !p5.instance
    ) {
      new p5();
    }
  }
```

おお！setupとdrawのことが書かれています。しかし、これは呼び出しているわけではありません。呼び出すなら`window.setup()`と`()`がついているはずです。これは、setupあるいはdrawが定義されていたなら（`!p5.instance`は無視します）、`new p5();`を実行しています。p5はcore/init.jsの一番上で、`import p5 from '../core/main';`という行がありますので、core/main.jsを見てみましょう。

[core/main.js](https://github.com/processing/p5.js/blob/552cdad6faa82b04f219c0f5f63ebea3fd81da15/src/core/main.js#L36)で`class p5 {`という行がありました。さきほどの`new p5()`はこの次の`constructor(...)`を呼び出します。setupとdrawを探しましょう。

このconstructorはとても長くて591行目まで続きます。constructorの最後で`this._start()`と_startという関数が呼ばれているようです。

```js
    if (document.readyState === 'complete') {
      this._start();
    } else {
      window.addEventListener('load', this._start.bind(this), false);
    }
```

this._startは234行目にあります。細かいことはおいておいてsetupとdrawを探すと、275行目に

```js
        this._setup();
        if (!this._recording) {
          this._draw();
        }
```

とあります。あと292行目にも似たような行があります。setupとdrawはそれぞれ_setupと_drawの中で呼ばれていそうです。

_setupは324行目にあります。ずっとみていくと、、、ありました。setupが呼び出されています。

```js
      if (typeof context.setup === 'function') {
        context.setup();
      }
```

_drawは374行目から定義されています。これもざっくりと見ていくと、drawは見つかりませんが、396行目でredrawという関数が呼ばれています。怪しいですね。また、_drawの最後で、次のような行があります。これは、ブラウザに_drawを再描画の際に呼び出すことを依頼しています。_drawの最後で次も呼んでもらえるようにブラウザに依頼するので、_drawはブラウザがこのページを表示している限り繰り返し呼ばれます。[^3]

```js
        this._requestAnimId = window.requestAnimationFrame(this._draw);
```

redrawをクリックすると、GitHubが[core/structure.js](https://github.com/processing/p5.js/blob/main/src/core/structure.js#L459)で定義されていると教えてくれます。開いてみてみましょう。ありました。489行目でdrawが呼び出されています。

```js
      try {
        context.draw();
      } finally {
```

setupとdrawが呼び出されていることが確認できました。以上の流れを整理してみましょう。

1. ブラウザがHTMLと必要なスクリプトを読み込む。
2. _globalInitが呼び出される。
3. _globalInitはsetupとdrawが定義されていたら`new p5()`を実行する。
4. p5のconstructorが呼び出される。
5. constructorは_startを呼び出す。
6. _startは_setupを呼び、次に_drawを呼び出す。
7. _setupの中で僕たちのsetupが呼び出される。
8. _drawの中でredrawが呼び出される。
9. redrawの中で僕たちのdrawが呼び出される。
10. _drawは最後にブラウザに_drawを呼び出すよう依頼する。

これで、setupが最初に1回だけ、drawが何度も呼び出されていることが確認できました。

---
[^1]: [命令形プログラミング](https://ja.wikipedia.org/wiki/%E5%91%BD%E4%BB%A4%E5%9E%8B%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0)といいます。それに対して[宣言型プログラミング](https://ja.wikipedia.org/wiki/%E5%AE%A3%E8%A8%80%E5%9E%8B%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0)というパラダイムもあります。[ビスケット](https://www.viscuit.com/)による[解説](https://www.viscuit.com/whatisviscuit/)。
[^2]: パラメータを受け取る関数の定義は別の機会に。
[^3]: こうすることで、ブラウザが最小化されたり、違うタブが開かれたときに_drawを呼び出すことを防ぎます。
