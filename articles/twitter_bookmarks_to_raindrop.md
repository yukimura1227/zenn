---
title: Twitterのブックマーク整理が辛すぎてブチ切れたので、Raindrop.ioに一括移行したった！！
emoji: 👻
type: idea
topics:
  - twitter
  - raindrop
  - javascript
published: false
---
## この記事の要約

筆者は、Pocketがサービス終了したので、Twitterのブックマーク機能を活用しはじめました。しかし、Twitterのブックマーク機能でカテゴリを分けて管理する運用を1ヶ月以上やってみた結果、とてもつらいことがわかったため、Raindrop.ioを活用することに決めて、無料でデータ移行できるスニペットを作成したので紹介します。

## 対象読者

- Twitterのブックマーク機能に不満がある人
- Pocketサ終で困っている人

## はじめに

Twitterは、非常に素晴らしいツールで、課金もして愛用しています。ただ、ブックマーク機能だけは、ユーザー体験が悪くて辛かった！！
具体的には、Twitter(現X)のブックマーク機能は、 無課金だとフォルダを分けられないし、課金しても、保存時にフォルダを選ぶのはひと手間かかる。。。なにより、**後から整理する際に、フォルダの一括移動できない！！**
など、ブックマークを活用・整理しようと思ったときに、大きな怒りを覚えました。結果的に、ブックマークは、Raindrop.ioに移行することにしました。
TwitterのブックマークをRaindrop.ioに移行するのに、Twitter側にブックマークをエクスポートする機能がなかったので、無料でできる範囲で、一括で移行するjsスニペットを組んで一括移行をしたので、紹介します。

## 申し送り事項

この記事の半分は、Twitterのブックマーク機能がいろいろ足りていないことへの怒りのお気持ちを書いているだけなので、「あるあるぅ！」という共感をしたい人以外は、読み飛ばしていただいて、最後のスニペットを使ったTwitterブックマークのRaindrop.io移行の部分だけ活用いただければと思います。
筆者は、Twitter自体は、素晴らしいサービスと捉えており、今後も課金・愛用するつもりです。唯一、ブックマーク機能だけは、強い不満があったというお話です。

## Twitterのブックマーク機能への怒り

:::details 怒りその１
### 怒りその１

Pocketという、Web記事をブックマークして後から整理するサービスがあり、お世話になっていたのですが、2025年7月にサービス終了してしまいました。
私も愛用していたサービスだったのですが、実際には、多くの記事は、Twitterのポストの保存に利用していたので、Pocketサービス終了に伴い、Twitterのブックマークを利用するようにしました。

私は、Twitter Blueユーザーなので、ブックマークのディレクトリ分割機能も利用できたので、それで十分だと考えていました。

使い方としては、モバイルアプリでTwitterを読んで、気になったものを、適当にブックマークするという運用をしていました。
本当は、ブックマークしたときに、たとえば「AWS」「AI」「フロントエンド技術」「ニュース」「共感」「癒やし系動物」「ネタ」などなどのブックマークディレクトリの仕分けをしたかったのですが、UIがそれを許してくれませんでした。
一応、「ブックマークを押す」->「ディレクトリへ追加を押す」->「ディレクトリを指定する」できるのですが、気軽にブックマークをしたい私としては、この3アクション必要であるという事実は、私にはとても、受け入れることができませんでした。

なので、追加時にディレクトリを使うことはあきらめて、気が向いたときに、ブックマークを整理をする形で運用をしてみようと考えいました。

約1ヶ月ほどブックマーク追加の運用を行い、ついに整理してみるかーと気が向いたのが、2025/08/31のことです。

ブックマークの一覧を開き、まずは、癒やされる動物のポストを一括でディレクトリに移行するかと思ったその時。。。

**一括でディレクトリ移行なんてできなくないかっ！！！！！！！！！**

えっ、このままだと、癒やされる動物のポストを眺めたいだけなのに、小難しい技術の情報が目に入ってきたり、ニュースが入ってきてしまうではないか！！！

これは辛すぎる！！ということで、Pocketの移行先として好評な、Raindrop.ioにブックマークを引っ越すことにしました。

:::

:::details 怒りその２
### 怒りその２

さぁ、ではTwitterのブックマークをエクスポートして、Raindrop.ioにインポートしよう！！
そう思った私は、甘すぎました。

ブックマークをエクスポートするなんて親切な機能はないのです！！
今も課金をしているのですが、さらに追加で課金すれば、APIを使って実現はできるのですが、今回やりたいことに対しては、過剰なコストだと考え、無料でできる範囲で実現することにしました。

:::
## 解決方針

Pocketがサ終してしまっているため、どうようのことができるサービスとして、 [Raindrop.io](https://raindrop.io/) があります。
URLをストックしておいて、複数のURL郡を一括でタグをつけたりディレクトリに移動したりできれば、私がやりたいことは実現できるのですが、Raindrop.ioであれば、無料でやりたいことが実現できるので、Alt Pocketのつもりで活用することにしました。
今後は、Twitterのブックマークは使わず、Raindrop.ioに共有する運用を想定しています。
なので、最初の一度だけ、今登録されている自分のTwitterブックマークに登録されているポストのURL一覧をRaindrop.ioに登録しなおすという方法を取ります。

幸いなことに、Raindrop.ioは、txtやcsvでのインポートが可能なので、

① ブラウザでtwitterのブックマーク一覧ページを開く
② ブラウザの検証ツールを開きコンソールタブで、移植したいURLの一覧を取得するためのコードスニペットを実行する
③ ②で取得したURLの一覧をRaindrop.ioにインポートする

という形で実現しました。

## 具体的なやり方

PCのGoogleChromeを利用していることが前提条件となりますが、他のブラウザでも概ね、同じやり方で実現かのうかと思います。

1. Twitterにログインして、ブックマーク一覧 `https://x.com/i/bookmarks` を開く 
2. `Cmd+Option+J`（Mac）/ `Ctrl+Shift+J`（Win）で **Console** を開く  
3. 下のスニペットをそのまま貼って実行  
   - ページを自動スクロールしながら URL を集め、**`TwitterDookmarkInbox.txt`** をダウンロードします  
   - (クリップボードにも同じ内容がコピーされますが、使いませんでした)
1. Raindrop.ioにログインして、上記ファイルをインポートします。

```javascript
(async () => {
  const sleep = ms => new Promise(r => setTimeout(r, ms));
  const seen = new Set();
  let lastHeight = 0, stuck = 0;

  // URLを収集（/status/<id> を正規化して x.com の永続URLに）
  const harvest = () => {
    document.querySelectorAll('a[href*="/status/"]').forEach(a => {
      const m = a.href.match(/https?:\/\/(?:twitter|x)\.com\/[^/]+\/status\/(\d+)/);
      if (m) seen.add(`https://x.com/i/web/status/${m[1]}`);
    });
  };

  // ページ下端までスクロールしながら収集
  while (true) {
    harvest();
    window.scrollTo(0, document.body.scrollHeight);
    await sleep(5000); // 読み込み待ち。短すぎてうまく動かない場合は10000などにチューニングしてください。
    const newHeight = document.body.scrollHeight;
    stuck = (newHeight === lastHeight) ? stuck + 1 : 0;
    lastHeight = newHeight;
    if (stuck >= 3) break; // 雑実装ですが、高さが変わらなければ終端と判断
  }

  harvest();
  console.log(`Collected ${seen.size} URLs`);

  // クリップボードへコピー（Chromeコンソールで有効）
  try { copy([...seen].join('\n')); } catch (e) {}

  // テキストファイルを自動ダウンロード
  const blob = new Blob([Array.from(seen).join('\n')], {type: 'text/plain'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'TwitterBookmarkInbox.txt';// このファイル名はそのまま、Raindrop側でのディレクトリ名になるので、適宜変更してください。
  a.click();
  URL.revokeObjectURL(url);
})();
```


```javascript
(async () => {
  const sleep = ms => new Promise(r => setTimeout(r, ms));
  const rows = new Map(); // id -> {url,date,text}
  let lastHeight = 0, stuck = 0;

  const sanitize = s => (s || '').replace(/\s+/g, ' ').trim();
  const harvest = () => {
    document.querySelectorAll('a[href*="/status/"]').forEach(a => {
      const m = a.href.match(/https?:\/\/(?:twitter|x)\.com\/[^/]+\/status\/(\d+)/);
      if (!m) return;
      const id = m[1];
      const url = `https://x.com/i/web/status/${id}`;
      if (rows.has(id)) return;

      const article = a.closest('article');
      const date = article?.querySelector('time')?.dateTime || '';
      // 本文らしき部分（UI変更に弱いので fallback で article 全文から先頭200文字）
      let text = '';
      const textNodes = article?.querySelectorAll('[data-testid="tweetText"]');
      if (textNodes && textNodes.length) {
        text = [...textNodes].map(n => n.innerText).join(' ');
      } else if (article) {
        text = article.innerText || '';
      }
      text = sanitize(text).slice(0, 200);

      rows.set(id, { url, date, text });
    });
  };

  while (true) {
    harvest();
    window.scrollTo(0, document.body.scrollHeight);
    await sleep(1200); // 読み込み待ち。短すぎてうまく動かない場合は2000〜2500などにチューニングしてください。
    const newHeight = document.body.scrollHeight;
    stuck = (newHeight === lastHeight) ? stuck + 1 : 0;
    lastHeight = newHeight;
    if (stuck >= 3) break; // 雑実装ですが、高さが変わらなければ終端と判断

  }
  harvest();

  const esc = s => `"${(s || '').replace(/"/g, '""')}"`;
  const csv = ['url,date,text'].concat(
    [...rows.values()].map(r => [esc(r.url), esc(r.date), esc(r.text)].join(','))
  ).join('\n');

  const blob = new Blob([csv], {type: 'text/csv'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'TwitterBookmarkInbox.txt';// このファイル名はそのまま、Raindrop側でのディレクトリ名になるので、適宜変更してください。
  a.click();
  URL.revokeObjectURL(url);

  console.log(`Exported ${rows.size} rows`);
})();
```

以下のように、ブックマークページを開いて、上記スニペットを実行します。

![](/images/twitter_bookmarks_to_raindrop/CleanShot_20250831_130431.png)

ポストのURLを一覧化したものがダウンロードできます。

![](/images/twitter_bookmarks_to_raindrop/Pasted%20image%2020250831130355.png)
![](/images/twitter_bookmarks_to_raindrop/CleanShot_20250831_140156.png)

## おわりに

以上で、TwitterのBookmarkを