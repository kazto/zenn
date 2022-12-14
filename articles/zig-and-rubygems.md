---
title: "ruby gemをzigで書こう"
emoji: "🪚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zig", "ruby", "rubygems", "mkmf"]
publication_name: thinkingsinc
published: true
---
こんにちは、[kazto](https://twitter.com/kazto_dev)です。

本記事は[Zigアドベントカレンダー](https://qiita.com/advent-calendar/2022/ziglang)14日目の記事です。

Ziglang、とても良い言語ですよね。

今のところまだ認知度が高くないので、アドベントカレンダーを立ててみたんですが、案の定と言いますか参加率はまだまだこれから。

### zigの紹介

2022年12月現在、バージョンは0.10.0、まだまだ破壊的な仕様変更が加わっている状態です。

なぜこんな作り途中の言語を推すのか。

公式サイトには、以下のように特徴を挙げています。

- 隠された制御フローはありません。
- 隠されたメモリ割り当てはありません。
- プリプロセッサ、マクロもありません。

プリプロセッサやマクロが無いことが長所になるのか、については、Zigに関しては長所と言えましょう。

Zigの魅力は、シンプルな言語仕様にあると考えています。

### ビルドシステム

また、Zigには言語自体にビルドシステムが備わっています。C言語に対するMakeコマンド、Makefileの立ち位置のものです。

これが強力で、Makefileの代替として十分役に立つと考えています。例については後述したいと思います。

さらには、Zigのビルドシステムは、C/C++のソースもビルドできます。ZigコンパイラがCコンパイラのラッパとして動作します。

### C/C++資源の活用

上記で、ZigのソースとC/C++のソースをいっしょくたにビルドできることを示しましたが、ビルドしたものが相互に呼び出されないと意味がありません。もちろん、ZigでビルドされたバイナリはC/C++の呼び出し規則に準拠しており、FFIなしに相互に呼び出すことができます。

### 長い前振り

ここまで読んでくださり、ありがとうございます。ええ加減Rubyどこに関係しとんねんとお思いのみなさま。ピースは出揃っております。

- 使いやすいビルドシステム
- C/C++との親和性

ここで問題です。Rubyの開発言語は何でしょうか？

Rubyそのものだったり、近年YJITで採用されたRustは置いておいてください。

そう、C言語ですね。

Ruby gemの大半はRubyそのもので書かれていますがネイティブエクステンションは大半がC/C++で記述されています。

とはいえ、令和にもなってC言語を覚えたくはないですよね？メモリリーク怖いですよね？マクロで魔改造されたコードを追いかけるのは大変ですよね？

### そこでZigですよ

Zigでプログラミングすることは、C言語でプログラミングすることよりも生産性が高いはず、と見込んでおります（個人の感想レベルですが）。

### でも、bundle gemコマンドとか対応してないじゃん

Rubyでネイティブエクステンションをビルドする手順をおさらいしましょう。

- bundle gemでひな形を生成する
- extconf.rbを記述してMakefileを生成する
- Makefileが各コンパイラを呼び出してビルドする
- できたバイナリをパッケージングする

上記のうち、extconf.rbでMakefileを生成するところで、MakefileからZigコンパイラを呼ぶことに対応すれば、Zigで記述されたソースもビルドできそうです。

### 「あるよ…」（って言いたかった）

[extconf.rbでZigを呼び出すためのMakefileを作成するパッケージ](https://github.com/kazto/mkmf-zig)を作成中です。本当はアドベントカレンダーに合わせてリリースしたかったのですが、間に合いませんでした。。。

extconf.rbは、`bundle gem`で生成した直後は以下のコードになります。

```ruby
# frozen_string_literal: true

require "mkmf"

create_makefile("example/example")
```

これを、mkmf-zigパッケージを用いて以下のように変更します。

```ruby
# frozen_string_literal: true

require "mkmf"
require "mkmf/zig"

create_zig_makefile("example/example")
```

そうすると、Zigを呼び出すようなMakefileが作成される、というイメージです。

つぎに、extconf.rbがあるディレクトリで`zig init-lib`を実行します。build.zigとsrc/main.zigが生成されます。

example.zigにおいてruby.hをインクルードし、rubygemsのお作法にしたがって呼び出せる状態にします。

```zig
const ruby = @cImport({
    @cInclude("ruby.h");
});

var rb_mExample: ruby.VALUE = 0;

(省略)
```

この状態で`zig build`を実行すると、`./zig-out/example/libexample.so`がビルドされます。

この.soファイルをextconf.rbと同じディレクトリに`example.so`と移動してやり、`gem build`すれば.gemファイルが作成されるようにしたいと考えています。この辺はMakefileがよしなにやってくれるようにする予定です。

### まとめ

以上のように、ZigとRubyはとっても仲良しになれる可能性を秘めています。Rubyistの皆さん、Zigを使いましょう！！！（なんのアドベントカレンダーだかわかんなくなっちゃった！）
