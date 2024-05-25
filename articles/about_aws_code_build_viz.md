---
title: "AWSCodeBuildの速度改善をドヤ顔で魅せたい人のために可視化ツールを作った話"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","codebuild","deno","vite"]
published: true
---

## 概要

AWS CodeBuildの実行時間を短縮にチャレンジしていて、その改善効果を、おしゃれに魅せたい！！そんなときはありませんか？（私はあります）

本稿では、よくAWS CodeBuildを利用していて、実行時間の短縮などにもチャレンジしている筆者が、改善の歴史を魅せるのに、ちょうどよいツールがなかったので、AwsCodeBuildVizという可視化ツールを作ったので、紹介します。

見た目と機能とアーキテクチャは、こんな感じです

### 見た目

![AwsCodeBuildViz](/images/about_aws_code_build_viz/AwsCodeBuildViz-min.gif)

### 機能

- AWS CodeBuildの結果を取得する機能
  - 2回目以降の実行では差分取得するので効率的に取得できる
  - AWSにアクセスするための認証は、2種類の方式から選択できる
    - AWS SSO
    - AWS_ACCESS_KEYとAWS_ACCESS_SECRET
- AWS CodeBuildの結果を可視化する機能
  - 以下の切り替えをインタラクティブにできる
    - 集計を日毎/月毎に切り替えできる
    - 表示単位を日毎に指定できる
  - Phase(Build,PreBuild,PostBuildなど)事に可視化ができる
- Docker上で動作
  - ローカル環境を汚さずに実行ができる
- DenoとViteを利用しており、高速に動作します

### アーキテクチャ

![AwsCodeBuildViz-Architecture](/images/about_aws_code_build_viz/architecture.png)

[AwsCodeBuildVizのリポジトリ](https://github.com/yukimura1227/AwsCodeBuildViz)

[使い方は、README.mdを参照ください](https://github.com/yukimura1227/AwsCodeBuildViz/blob/main/README.md)

## 技術的な話

AwsCodeBuildVizは、以下の２つの要素で構成されています。

- AWS CodeBuildの実行履歴を取得するためのcliツール部分
  - Denoベースで、キャッシュ用にDeno KVを利用
- 取得した履歴を可視化するための、可視化ツール部分
  - DenoベースのViteアプリで、React-TypeScriptを利用

### cliツール部分

cliツール部分は、以下のような構成にしています。

Deno1.28頃から、npmをサポートしたという話があり、実用的に利用できるか試してみたかったので、Denoを採用しています。
また、CodeBuildの結果をキャッシュしておくために、Deno KVというデータストアを利用しています。

- Deno1.41.2を利用
  - AWSへのアクセス部分には、@aws-sdk(npm)を利用
  - CodeBuildの結果のキャッシュ用に、Deno KVを利用

#### npm連携はどんな感じ？

今回は、npmで提供されているaws-sdkをDenoから利用してみましたが、今回の範疇では、特段問題なく動かすことができました。

例えば、以下は、CodeBuildの結果のid一覧を取得している部分のコードですが、importする際に、'npm:@aws-sdk/client-codebuild'のようにnpmベースのライブラリであることを示すだけで、サクッと動きました。
(私のローカル開発環境では、型の情報が、イマイチ反映されなかったので、本格的に開発に利用する場合は、LSP周りの設定などを、工夫する必要があるかもしれません。型推論が弱いだけで、明示的に型の情報を指定すれば問題ありません。)

```ts:ListBuilds.ts
import { CodeBuildClient, ListBuildsForProjectCommand, ListBuildsForProjectCommandInput, ListBuildsForProjectCommandOutput } from "npm:@aws-sdk/client-codebuild";

const createClient = (credentials:unknown,region:string) => {
  const client = new CodeBuildClient({
    region: region,
    credentials: credentials,
  });
  return client;
};

const listBuildsOnce = async (client:unknown, buildIdsResult:string[], codeBuildProjectName: string, nextToken?: string) => {
  const input:ListBuildsForProjectCommandInput = {
    projectName: codeBuildProjectName,
    nextToken: nextToken,
  };
  const command = new ListBuildsForProjectCommand(input);

  const response:ListBuildsForProjectCommandOutput = await client.send(command);

  nextToken = response.nextToken;

  for(const buildId of response.ids) {
    buildIdsResult.push(buildId);
  }

  if(nextToken) await listBuildsOnce(client, buildIdsResult, codeBuildProjectName, nextToken);
}

export const ListBuilds = async (credentials:unknown, codeBuildProjectName: string, region: string):Promise<string[]> => {
  const client = await createClient(credentials, region);

  const buildIdsResult:string[] = [];
  await listBuildsOnce(client, buildIdsResult, codeBuildProjectName);
  return buildIdsResult;
};
```

#### Deno KVはどんな感じ？

CodeBuildの情報を毎回取得せず、差分実行を実現するために、何かしらのやり方で、取得結果をキャッシュさせたかったのですが、今回は、Deno KVという機能を使ってキャッシュしてみました。
ローカル環境で実行する場合は、実態はsqliteを利用しているようでした。

例えば、
「CodeBuildの詳細情報がローカルになかった時だけ取得する」という機能を実現したい場合は、以下のように、シンプルに実現することができ、非常に強力な機能でした。

```ts:main.ts
const kv = await Deno.openKv('./denoKVData/kv.sqlite3');
const buildIds = await ListBuilds(credentials, codeBuildProjectName, region);

// CodeBuildの実行都度払い出されるbuildIDは、全件取得してDeno KVに保存
// CodeBuildの実行履歴は、自動的に消えていくので保存しておく
for(const buildId of buildIds) {
  await kv.set( [codeBuildProjectName, buildId], { codeBuildProjectName, buildId });
}

const entries = kv.list({ prefix: [codeBuildProjectName]});
for await (const entry of entries) {
  const buildId:string = (entry.value as { buildId: string }).buildId;

  const buildDetailEntry:BatchGetBuildsCommandOutput = await kv.get([`${codeBuildProjectName}__detail-response`, buildId]);
  // キャッシュがない場合だけ、詳細情報を取得して、キャッシュしておく
  // キャッシュがある場合は、詳細情報を取得しない
  if( buildDetailEntry.value === null || buildDetailEntry.value.response === undefined) {
    const response = await BatchGetBuilds(credentials, buildId, region);
    await kv.set(
      [`${codeBuildProjectName}__detail-response`, buildId],
      { response },
    );
  }
}
```

#### AWS SSOとの連携はどんな感じ？

AWS_ACCESS_KEY_ID,AWS_SECRET_ACCESS_KEYを利用する場合は、特に工夫することはなかったのですが、aws cliのaws sso loginによるログイン機能と連携をしたかったので、以下のような工夫をしました。

まず、AWS_ACCESS_KEY_IDを利用する場合は、必要なトークンを設定した上で、以下のようにdocker compose(compose.yaml)を使って、実行できるようにしてあります。

```bash
docker compose run --rm cli
```

AWS SSOを利用する場合は、dockerのホストマシン側で、aws sso loginを実行した時に生成される、credentialの情報をvolumeマウントすることで、共有するようにしました。

compose.yamlとは別に、compose.sso.yamlを作成することで実現しています。

```yaml:compose.sso.yaml
services:
  cli:
    volumes:
      # 中略
      - ~/.aws:/root/.aws:ro
    environment:
      - AWS_CONFIG_FILE=/root/.aws/config
  # 後略
```

```bash
docker compose -f compose.yaml -f compose.sso.yaml run --rm cli
```

### 可視化ツール部分

可視化ツール部分は、Viteを使って、React-TypeScriptで構築しています。

#### ViteアプリをDenoで動かす

ViteアプリをDeno上で動かせないか調べてみたところ、Denoの公式ドキュメントに記述があり、そのとおりの手順を踏むことで、Deno上でViteを動かすことができました。

[vite#step-1-create-vite-app](https://docs.deno.com/deploy/tutorials/vite#step-1-create-vite-app)

生成されたアプリに同梱されていた、Vite with Denoのアイコンがとってもかわいいです。

![vite-deno](/images/about_aws_code_build_viz/vite-deno.png)

生成されたアプリを眺めてみると、さきほどのcliのところのように、'npm:xxxxxx'のように指定しているわけではなく、NodeJS由来のpackage.jsonを利用しており、node_modulesをそのまま利用しているようでした。

生成されたdeno.jsonでは、--node-modules-dirというオプションがついており、package.json経由でinstallされたnode_modulesを利用しています。

```json:deno.json
{
  "tasks": {
    "dev": "deno run -A --node-modules-dir npm:vite"
  }
}
```

Denoっぽい記述は、以下のようなvite.config.mtsだけでした、
ほかは、NodeJSで作った場合とまったく同じものが動きました。

```ts:vite.config.mts
import { defineConfig } from 'npm:vite@5.1.6';
import react from 'npm:@vitejs/plugin-react@^4.2.1';

import 'npm:react@^18.2.0';
import 'npm:react-dom@^18.2.0';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  server: {
    host: true
  },
});
```

実際のアプリ側は、以下のような感じで、Deno特有の記述が一切ありません。

```ts:main.tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { App } from './App.tsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

既存のNodeJSベースのアプリを、Denoに移植するための敷居が、とても低くなっているので感動しました。

#### フロントエンドの主要ライブラリ

Deno上でViteアプリを動かす上で、node_modulesをそのまま利用できたため、特筆する事項がないので、利用しているnpmライブラリを紹介して終わります。

フロントエンドの主要ライブラリ（カッコ部分はメジャーバージョン）は以下のとおりです。

- TypeScript(5)
- React(18)
- chart.js(4)
- react-chartjs-2(5)

グラフ描画にchart.js、インタラクティブに表示を切り替えたいので、Reactを利用しています。

特にこれといった工夫をすることはなく、集計単位の切り替えや、表示対象期間の切り替えがぬるぬるうごいていい感じです。

![AwsCodeBuildViz](/images/about_aws_code_build_viz/AwsCodeBuildViz-min.gif)

## おわりに

AWS CodeBuildの実行時間を、日毎や月毎に集計して、インタラクティブに表示するAwsCodeBuildVizという可視化ツールを作ったので、中身の技術を紹介しました。

cliツールだけではなく、ブラウザアプリ側もすべてDenoで構築するチャレンジをしたのですが、Denoがnpmだけではなく、node_modules/をディレクトリごとそのまま利用できるようになっていたため、NodeJS -> Denoへの移行の敷居が極めて低くなっていることを実感しました。

Docker上で簡単に動かせるようにしていますので、AWS CodeBuildの実感の推移を、魅せたくなったときは、是非ご活用いただければ幸いです。
