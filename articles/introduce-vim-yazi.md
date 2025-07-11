---
title: "プラグインの引っ越し地獄から脱却！令和のVimファイラーはこれで決まり！"
emoji: "🐤"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["yazi", "Vim", "Neovim", "filer", "plugin"]
published: true
---

## 忙しい方のための要約

Rust製の超軽量Terminal File Manager `yazi` という素晴らしいファイラーをVim/Neovimから透過的に使うためのプラグイン `vim-yazi` を開発したので、その紹介と、設定方法、使い方、技術的な話をします。
ファイラープラグインの引越し地獄から脱却したい方、Vim/Neovimで使いやすいファイラーを探している方におすすめです。
[vim-yazi](https://github.com/yukimura1227/vim-yazi) で公開しているので、よかったら使ってみてください。（issue報告や要望も大歓迎です）

!["vim-yazi-logo"](/images/introduceVimYazi/vim-yazi-logo.png =256x256)
動作イメージ
![vim-yazi-introduce](/images/introduceVimYazi/vim-yazi-introduce.gif)

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

### ファイルファインダーには決定版がある

ファイラーで、ファイルを探すこともありますが、大域的に探す場合は、fzf.vimが最強です。
`:Files <directory>` で、ファイル名で探したり、 `:Rg <keyword>` で、ファイルの内容で検索したり、筆者も長年愛用しています。
fzf.vimは、Rust製のfzfコマンド、ripgrepコマンド、fdコマンドなど、スタンドアロンで動作するツールを利用しているため、Vim/Neovimのバージョンに依存せず、安定して動作します。また、Pythonのようなランタイムも必要ないため、ローカル環境が汚染されることもありません。
Vimmer以外の方も多く利用しているため、長期的に安定して開発が継続することが期待できます。

### Terminal File Managerのyaziとの出会い

そんな中、Terminal File Managerの `yazi` に出会いました。
`yazi` は、Rust製の軽量で高速なTerminal File Managerで、
以下のような特徴があります。

- 高速なファイル操作
- Vimmer目線で、シンプルで直感的なUI
- デフォルトでおしゃれ(要 Nerd Font)
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

Vimから利用する上で、yazi本体と、ripgrep, fd, fzf, font-symbols-only-nerd-fontは、必須級なので、それらをインストールします。

Macの場合、Homebrewを使うと簡単にインストールできます。

```bash
# vimから利用する上で必須のものだけインストール(ripgrep: ファイル内容の高速な検索, fd: 高速なファイル検索, fzf: 高速なファジーファインダー)
brew install yazi ripgrep fd fzf font-symbols-only-nerd-font
# フルで活用する場合は以下もインストール(ffmpeg: 動画のサムネ表示, 7-Zip: 圧縮ファイルの操作, jq: JSONのPreview, poppler: PDFのPreview, resvg: SVGのPreview, imagemagick: HEIC,jpegのPreview, nerd-font: アイコン表示, zoxide: カレントディレクトリの移動)
brew install yazi ffmpeg sevenzip jq poppler fd ripgrep fzf zoxide resvg imagemagick font-symbols-only-nerd-font
```

他のインストール方法については、[公式ドキュメント](https://yazi-rs.github.io/docs/installation/)を参照してください。

### yaziの基本

Vimから利用するうえで、最低限押さえて起きたいyaziの基本的な操作を紹介します。

(hjklとs,S,zによる検索。あとは、ファイル削除、rename,一括操作を紹介したい)

- tree上での移動
  - `h` : 上のディレクトリへ移動
  - `j` : 下のファイル/ディレクトリへ移動
  - `k` : 上のファイル/ディレクトリへ移動
  - `l` : 下のディレクトリへ移動

![yazi-hjkl](/images/introduceVimYazi/yazi-hjkl.gif)

- ファイル操作
  - `a` : ファイルを追加
  - `r` : ファイルをリネーム
  - `d` : ファイルを削除

![yazi-ard](/images/introduceVimYazi/yazi-ard.gif)

- ファイル検索
  - `s` : ファイル名で検索(fdによるファイル名の高速検索)
  - `S` : ファイル内容で検索(ripgrepによるファイル内容の検索)
  - `z` : ファイル名で検索(fzfによるファイル名のファジーファインド)

![yazi-sSz](/images/introduceVimYazi/yazi-sSz.gif)

その他、一括renameなど、ファイルマネージャーとして、非常に優れた機能を持っています。
詳しくは、[公式ドキュメント:Yazi features](https://yazi-rs.github.io/features)を参照してください。

## vimとyaziを統合するvim-yazi

yaziをCLIツールとして愛用していたのですが、Vimと統合することで、より快適に使えるのではないかと思い、vim-yaziというプラグインを作成しました。(また、冒頭に述べたように、Vimのファイラープラグインの引越地獄の輪廻から脱却したい！！という思いも強くありました。)

絶対世の中の誰かが作っている！！とおもってみてみたのですが、[yazi.nvim](https://github.com/mikavilpas/yazi.nvim) というNeovim専用のものだったので、VimScriptの勉強がてら自分で作ってみよう！！と思い至りました。

### vim-yaziのコンセプト

基本的には、ファイル管理はyaziに任せて、Vim/Neovimから透過的にyaziを利用できるようにすることを目指しています。
具体的には、Vim -> yaziのシームレスな連携と、yazi -> Vimのファイルのオープンを実現します。
また、Vim/Neovimの両方に対応することを目指しました。
denops等を利用せず、なるべくプリミティブな実装(=VimScriptのみ。luaやVim9 scriptを利用しない)

### vim-yaziの機能

- `:Yazi` コマンドでyaziを起動
  - vimの設定で、keymapできるように、`:Yazi [path]` のようにpathを指定して起動できるようにする
  - デフォルトでは、カレントディレクトリを起点にyaziを起動
- yaziで選択したファイルをVimで開く
  - 複数選択された場合も対応する
- オプションで、netrwをハイジャックして、yaziを起動するようにする
- シームレスな画面遷移
  - Vimの場合は、Terminalをフルサイズで起動する
  - Neovimの場合は、Floating Windowで起動する

### 技術トピック

なるべく依存が少ない形で、Vim/Neovim両対応を実現したかったので、VimScriptのみで実装しました。

#### Vim/Neovim両対応のための実装方法

Vim/NeoVim両対応するために実装は、VimScriptのみでやっています。
このあたりは情報が少なかったので、苦労しました。

わかってしまえばなんてことはないのですが、以下のようにhas()関数を使って、Vim/Neovimの機能を分岐させることで、両対応を実現しています。

```vim
if has('nvim')
  " Neovimの実装
elseif has('terminal')
  " :terminal対応可能なVimの実装
else
  " :terminal対応していないVimの実装
endif
```

#### terminal対応

Vim 8 から、`:terminal` というコマンドが追加され、TerminalをVimの中で起動できるようになりました。
これを利用することで、Vimの中でterminalを起動して、その上でyaziを起動することで、シームレスにファイル操作を行うことができます。
また、terminalからVimに戻るときに、選択されたファイルをVimのバッファに読み込むことができます。

ただ、VimScriptからterminalを操作するためのドキュメントが薄く、`term_start()` 関数を利用することで、やりたいことは、簡単に実現できるのですが、表示領域をフルサイズにする方法や、終了時のコールバックのsyntaxがみつからず、他のVimPluginの実装を参考にしながら、試行錯誤しました。

terminalの起動部分は、以下のような形で実現しました。

```vim
    let term_buf = term_start(yazi_cmd, {
      \ 'exit_cb': function('vim_yazi#OnYaziExit'),
      \ 'term_finish': 'close',
      \ 'curwin': 1,
    \ })
〜中略〜
function! vim_yazi#OnYaziExit(job, status)
  call vim_yazi#OpenSelectedFiles()
endfunction
```

curwinを1にすることで、現在のウィンドウでterminalを起動することができます。
また、exit_cbを指定することで、yaziが終了したときに、Vimに戻ることができます。
このとき、選択されたファイルをVimのバッファに読み込むように実装しています。

### カスタマイズ方法

カスタマイズについては、適宜READMEに追記していきますが、初期リリースでは以下のような設定項目を用意しています。

```vim
" yaziの実行ファイルのパス
" デフォルトでは、PATHにあるyaziを実行します。
let g:yazi_executable = 'yazi'

" yaziで複数ファイルを開くことを許可するかどうか
" 0にすると、yaziで選択したファイルを1つだけVimで開くようになります。
" 1にすると、yaziで選択したファイルを複数Vimで開くことができます。
let g:yazi_open_multiple = 0

" netrwをハイジャックして、yaziを起動するかどうか
" 0にすると、netrwはそのまま利用されます。
" 1にすると、netrwを起動したときにyaziが起動します。
let g:yazi_replace_netrw = 1

" yaziのデフォルトキーマッピングを無効にするかどうか
" 0にすると、yaziのデフォルトキーマッピングが有効になります。
" 1にすると、yaziのデフォルトキーマッピングが無効になります
let g:yazi_no_mappings = 1
```

また、キーマップについては、以下のように設定できます。

```vim

" 例えば、<Ctrl-n>でyaziを起動するように設定する場合
nnoremap <silent> <C-n> :Yazi<CR>

" 例えば、<Ctrl-d>で~/Downloadsディレクトリを開くように設定する場合
nnoremap <silent> <C-d> :Yazi ~/Downloads<CR>
```

## おわりに

Vimのファイルプラグインを乗り換え続けて、幾星霜。毎回引っ越し地獄に苦しんできましたが、`yazi` に出会えたことで、Vimmer専用ではない、長期的に使えそうな、Vimファイラーを実現してみました。
数年後、引っ越ししていないことを祈りつつ、`vim-yazi` を作成してみました。


今回作ったものは、[vim-yazi](https://github.com/yukimura1227/vim-yazi) で公開しています。
よかったら、是非使っていただき、不具合や改善要望などあれば、Issueを立てていただけると嬉しいです。