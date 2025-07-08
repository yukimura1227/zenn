---
title: "プラグイン地獄から脱却！令和版Vimファイラーはこれで決まり！"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vim", "yazi", "Neovim", "filer", "plugin"]
published: false
---

## はじめに

Vimを使っていて、標準のファイラーのnetrw(`:e .`と入力すると起動するアレです。)が使いにくくて困ることはありませんか？そういった悩みを解決するために、様々なVim/Neovim向けのファイラープラグインが開発されてきました。
素晴らしいプラグインが数多くありますが、Neovimにしか対応していなかったり、設定が複雑だったり、開発が止まって、別のプラグインに移行する必要があったりと、なかなか一つに絞るのが難しいのが現状です。
筆者も、これまでも多くのファイラープラグインを試してきましたが、どんなにしっくり来たものでも、開発が止まっているなどの理由で、その度に、**プラグインの引っ越し地獄**を味わってきました。
もう、**Vimのファイラープラグインで悩むのはうんざりだ！** と感じている最中に、`yazi` という超軽量で軽快に動作するTerminal File Managerに出会いました（惚れました）。
`yazi`は、Vimmer以外の方も多く利用しているFile Managerなので、少なくともVimmerしか利用していない独自のファイラープラグインよりも、長期的に安定して開発が継続することが期待できます。
そこで、この `yazi` とVim/Neovimを統合するプラグインを作ることで、これまでのVimファイラープラグイン地獄から脱却できるのではないかと考え、`vim-yazi` というプラグインを開発しました。

### 対象読者

- Vim標準のファイラー(netrw)に不満を持っている人
- Vimから使いやすいファイラープラグインを探している人
- 今使っているファイラープラグインが開発が止まっていて不安な人
- Vim/Neovimのファイラーをもう二度と引っ越したくない人

### 記事の概要

Rust製の軽量Terminal File Manager である `yazi` をVim/Neovimから使うためのプラグイン `vim-yazi` を作ったので、その紹介と、設定方法、使い方について解説します。

![](/images/introduceVimYazi/vim-yazi-sample.gif)

## 背景とモチベーション

### 標準のファイラー(netrw)の不満

Vimなどのエディタを使っていると、ファイルを探して開いたり、削除したり、移動したりといった操作をよく行います。
その際、エディタ内でファイルを操作するためのファイラーが必要になります。
Vimには標準で `netrw` というファイラーが付属していますが、操作性や機能面での不満がありました。

例えば、

- Rename操作が、直感的に行えない(RでRenameモードになるが、視覚的にわかりにくい)
- ファイルの作成が直感的に行えない(%で新規バッファが作られるが、touchと異なり、明示的に保存しないとファイルが作成されない)
- 複数ファイルの操作が面倒(マーク機能を使う必要があるが、直感的ではない)
- ディレクトリを辿るのが面倒(ディレクトリを開くためにEnterキーを押す必要がある)

### ファイラープラグインの不安

Vim/Neovimには、様々なファイラープラグインが存在します。
例えば、筆者が愛用していたものに絞っても、[vimfiler](https://github.com/Shougo/vimfiler.vim) や [NERDTree](https://github.com/preservim/nerdtree) [ddu-ui-filer](https://github.com/Shougo/ddu-ui-filer) [Fern](https://github.com/lambdalisue/vim-fern)などがあります。

どれも、秀でた特徴がある、素晴らしいプラグインで、重宝していましたが、以下のような不安がありました。

- いつか、Vimのみ、Neovimのみのサポートになって、引っ越しを余儀なくされるのでは？
- Vim/Neovim両対応のため、DenoやPythonが必要だったり、ローカル環境が汚染されるのでは？
- ターゲットユーザーがVimmerに限られているので、長期的に開発が続くのか不安...

### ファイルファインダーに決定版がある

ファイラーで、ファイルを探すこともありますが、大域的に探す場合は、fzf.vimが最強です。
`:Files <directory>` で、ファイル名で探したり、 `:Rg <keyword>` で、ファイルの内容で検索したり、筆者も長年愛用しています。
fzf.vimは、Rust製のfzfコマンド、ripgrepコマンド、fdコマンドなど、スタンドアロンで動作するツールを利用しているため、Vim/Neovimのバージョンに依存せず、安定して動作します。また、Pythonのようなランタイムも必要ないため、ローカル環境が汚染されることもありません。
vimmer以外の方も多く利用しているため、長期的に安定して開発が継続することが期待できます。

### Terminal File Managerのyaziとの出会い

そんな中、Terminal File Managerの `yazi` に出会いました。
`yazi` は、Rust製の軽量で高速なTerminal File Managerで、
以下のような特徴があります。

- 高速なファイル操作
- Vimmer目線で、シンプルで直感的なUI
- デフォルトでおしゃれ
- プラグイン不要で即座に使える

とても使いやすく、単独のツールとして利用していたのですが、yaziとVimを統合することで、前述の不安や不満を払拭できるのでは？？？ と思い至り、統合のためのVim Plugin を開発しました。

## yaziの紹介

### yaziとは

[yazi](https://github.com/sxyazi/yazi) は、Rust製の超軽量ターミナルファイルマネージャーです。  
- **高速なファイル検索**：`fd` や `ripgrep` による、高速で快適なファイル検索が可能  
- **Vimmer目線で直感的なキーバインド**：`hjkl` での移動や、`s`/`S` でのフィルタリング  
- **デフォルトでおしゃれ**： アイコン表示など、デフォルトでおしゃれなUI
- **プラグイン不要で即座に使える**： 追加のプラグインなしで、すぐに使い始められる

### yaziのセットアップ

Vimから利用する上で、yazi本体と、ripgrep, fd, fzfは、必須級なので、それらをインストールします。

Macの場合、Homebrewを使うと簡単にインストールできます。

```bash
# vimから利用する上で必須のものだけインストール(ripgrep: ファイル内容の高速な検索, fd: 高速なファイル検索, fzf: 高速なファジーファインダー)
brew install yazi ripgrep fd fzf
# フルで活用する場合は以下もインストール(ffmpeg: 動画のサムネ表示, 7-Zip: 圧縮ファイルの操作, jq: JSONのPreview, poppler: PDFのPreview, resvg: SVGのPreview, imagemagick: HEIC,jpegのPreview, nerd-font: アイコン表示, zoxide: カレントディレクトリの移動)
brew install yazi ffmpeg sevenzip jq poppler fd ripgrep fzf zoxide resvg imagemagick font-symbols-only-nerd-font
```

他のインストール方法については、[公式ドキュメント](https://yazi-rs.github.io/docs/installation/)を参照してください。

### yaziの基本

Vimから利用するうえで、最低限押さえて起きたいyaziの基本的な操作を紹介します。

(hjklとs,S,zによる検索。あとは、ファイル削除、rename,一括操作を紹介したい)



## vimとyaziを統合するvim-yazi

### vim-yaziの概要

### vim-yaziのコンセプト

### vim-yaziでできること

## おわりに