---
title: "p5.jsをはじめる"
free: true
---
p5.jsをはじめるには何もインストールする必要はありません。

ブラウザを立ち上げて、[p5.js Web Editor](https://editor.p5js.org/) を開いてみましょう。
次のような画面が開きましたか？

![p5.jsエディター](/images/books/p5js-for-kids/first-p5js_editor.png)

右上の English ▼ のところをクリックして、日本語を選びましょう。

![日本語](/images/books/p5js-for-kids/first-japanese.png)

メニューが日本語になりました。

すでに何やらプログラムのようなものが書いてあるので実行してみましょう。
実行するには、下のボタンを押してください。

![実行](/images/books/p5js-for-kids/first-play.png)

実行すると、何が変わったか分かりますか？
上の実行する前の画面と見比べてみましょう。

![](/images/books/p5js-for-kids/first-background.png)

:::details 答え
- プレビューの下が灰色になりました。
- ▶のボタンが赤くなって、■のボタンが灰色になりました。（プログラムは動き続けています。■を押すと止まります。）
:::

`createCanvas`や`background`の中の数字を変えてみましょう。

数字を変えたら▶を押して実行してみましょう。何が変わりましたか？

自動更新のチェックをオンにすれば▶を押さなくてもプログラムが変更されるたびに自動的に再実行されるようになります。

:::details createCanvas
`createCanvas`は[キャンバス](https://ja.wikipedia.org/wiki/%E3%82%AD%E3%83%A3%E3%83%B3%E3%83%90%E3%82%B9)を作り出します。
数字はその大きさを与えています。最初の数字が横幅で、2つ目の数字が縦幅です。
:::

:::details background
`background`はキャンバスの背景色を決めます。
0が黒で、255が白です。それより小さかったり大きかったりしても黒か白になります。[^1]

`background`に`background(10, 150, 200)`のように3つの数字を与えると背景に色がつきます。さまざまな3つの数字の組み合わせを試して好きな色を作ってみましょう。[^2]
:::

ここで残念なお知らせがあります。ブラウザで新しいウィンドウかタブで[p5.js Web Editor](https://editor.p5js.org/)を新しく開いてみましょう。さっきのプログラムではなく、最初に戻ってしまいます。日本語に設定したことも戻ってしまってます。

そんな君にいいお知らせが、右上のアカウント作成からアカウントを作成しましょう。そうすれば、作ったプログラム（スケッチといいます）を保存できるようになります。
アカウント作成にはメールアドレスなどが必要なのでお父さん、お母さんに手伝ってもらってください。

![img.png](/images/books/p5js-for-kids/first-account.png)

アカウントを作成したら、「ファイル▼ 保存」でスケッチを保存できるようになります。
保存したスケッチは「ファイル▼ 開く」で読み込めます。

### まとめ

- [p5.js Web Editor](https://editor.p5js.org/)を使って、プログラムを実行してみました。
- アカウントを作成して、スケッチを保存できるようになりました。

---
[^1]: p5.jsに与えた数字がおかしくてコンピュータが壊れてしまったりすることはないので安心してください。
[^2]: ここで使っている色の指定方法は [RGB](https://ja.wikipedia.org/wiki/RGB) といいます。
