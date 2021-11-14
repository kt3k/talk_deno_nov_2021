class: middle, center

<img src="./assets/logo.svg" align="center" width="200" />

# Deno の話

---
class: center middle

Deno 使ってますか? 🙋‍♀️🙋‍♂️

---

# 話す人

<img src="./assets/hinosawa.jpg" align="right" width="300" />

日野澤歓也 twitter @kt3k

- GREE (2012 - 2013)
- Recruit (2015 - 2019)
- Deno Land (2021 -)

<small>2018年から Deno にコントリビュートを開始。2020年作者に誘われ Deno Land に転職。現在はフルタイムで Deno と Deno Deploy を開発中。</small>

---
class: inverse middle center

Deno のロードマップ

---
Deno のこれまでのロードマップ
- Web 互換性
- TypeScript サポート
- ESM サポート
- V8 サンドボックスセキュリティ
- Node を上回るHTTPパフォーマンス

---
Deno のこれまでのロードマップ
- Web 互換性 ▲
- TypeScript サポート ✅
- ESM サポート ✅
- V8 サンドボックスセキュリティ ✅
- Node を上回るHTTPパフォーマンス ❌

v1.0 時点

---
Deno のこれまでのロードマップ
- Web 互換性 ✅
- TypeScript サポート ✅
- ESM サポート ✅
- V8 サンドボックスセキュリティ ✅
- Node を上回るHTTPパフォーマンス ▲

v1.16 (最新) では大部分達成

---
Deno のこれからのロードマップ

---
Deno のこれからのロードマップ

- さらなるHTTPパフォーマンス向上
- Node.js 互換性 (後述)

---
class: inverse middle center

Q. Node製のライブラリはDenoで使えるか

---
Q. Node製のライブラリはDenoで使えるか

A. 主に2つの方法がある (どちらも完璧ではない)

- npm モジュールを Deno 用に変換する CDN を介して使う方法 (非公式)
- Deno の Node.js 互換モードを使う(2022 Q2 予定)方法 (公式)

---

1) npm モジュールを Deno 用に変換する CDN を介して使う (非公式)

- esm.sh <br />`https://esm.sh/<package名>`
- skypack <br />`https://cdn.skypack.dev/<package名>`

React などはこの手法で動く

```
import React from "https://esm.sh/react";
import ReactDOM from "https://esm.sh/react-dom";
```

---
2) Deno の Node.js 互換モードを使う

- 現在、Deno コアチームにて鋭意開発中
- 2022 Q2 の v2 リリースである程度使える状態にする事が目標

```
npm install express
deno run --compat --unstable -A server.js
```

```js
const express = require('express')
const app = express()
app.get('/', (req, res) => {
  res.send('Hello World!')
})
console.log('Listening on localhost:3000');
app.listen(3000)
```

---
Node.js 互換モードについては [v1.5 リリース記事](https://deno.com/blog/v1.15#improving-node-compatibility) に概要の記述があります。


---
class: inverse middle center

Q. 権限管理のベストプラクティスは?

---
Q. 権限管理のベストプラクティスは?

A. 必要最小限のものを渡す

例. リンター
```
deno run --allow-read linter.ts
```

例. フォーマッター
```
deno run --allow-read --allow-write formatter.ts
```

---
Q. 権限管理のベストプラクティスは?

A. ネットワークパーミッションを渡す場合はホスト名を指定する

例. ポート8080で動く、Postgres と通信するサーバー
```
deno run \
  --allow-net=localhost:8080,host-to.postgres:5432 \
  server.ts
```

---
Q. 権限管理のベストプラクティスは?

A. 環境変数、プロセス実行は許可範囲を出来るだけ絞る

```
deno run \
  --allow-env=AWS_ACCESS_KEY_ID,AWS_SECRET_ACCESS_KEY \
  do_something_in_aws.ts
```

```
deno run --allow-run=git do_something_with_git.ts
```

---
Q. 権限管理のベストプラクティスは?

Note: `--allow-run` (引数なし) `--allow-run=deno` は `--allow-all` (すべて許可) と同じとなるため注意

以下のコードで権限の昇格が可能
```
Deno.run({
  cmd: ["deno", "run", "--allow-all", import.meta.url]
});
```

---
class: inverse middle center
Webサーバ用のライブラリ比較

---
Webサーバ用のライブラリ比較

- Webフレームワークが20個ぐらいある。
- 以下おすすめを紹介

---
1) oak

- 一番人気
- koa にインスパイアされたデザイン

```ts
import { Application, Router }
  from "https://deno.land/x/oak/mod.ts";

const router = new Router()
  .get("/", (context) => {
    context.response.body = "Hello world!";
  })
  .get("/book/:id", (context) => {
    context.response.body = books.get(context.params.id);
  });

const app = new Application();
app.use(router.routes());
await app.listen({ port: 8000 });
```

---
2) `std/http` + [URLPattern](https://developer.mozilla.org/en-US/docs/Web/API/URLPattern) API

- std/http は Deno 標準モジュールが提供する http サーバー機能

```ts
import { serve }
  from "https://deno.land/std@0.114.0/http/server.ts";

const topPage = new URLPattern({ pathname: "/" });
const myPage = new URLPattern({ pathname: "/me" });

console.log("Listening on http://localhost:8000");
await serve((req) => {
  if (topPage.test(req.url))
    return new Response("Top page");
  if (myPage.test(req.url))
    return new Response("My page");
  return new Response("Not Found", { status: 404 });
});
```

---
3) sift

- JSX 対応などが便利、ミニマルな API

```js
/** @jsx h */
import { h, jsx, serve }
  from "https://deno.land/x/sift@0.4.2/mod.ts";

const App =
  () => <div><h1>Hello world!</h1></div>;

const NotFound =
  () => <div><h1>Page not found</h1></div>;

serve({
  "/": () => jsx(<App />),
  404: () => jsx(<NotFound />, { status: 404 }),
});
```

---
class: inverse middle center

Q. When will deno be production ready

---
Q. When will deno be production ready

- It depends on what you need in production.
- ランタイムの安定性という意味では既に production ready と言って良い段階
- API も 1.0 以降は semver を遵守しているため、バージョンアップで急に壊れたりはしない
- npm モジュールが使えなければ困るという事であれば 2022 Q2 (目標) の Node.js 互換モードを一旦待ってください

---
class: inverse middle center

Q. deno が TS を捨てたときの裏話? があれば聞きたい

https://docs.google.com/document/d/1_WvwHl7BXUPmoiSeD8G83JmS8ypsTPqed4Btkqkn_-4/preview?pru=AAABcrrKL5k*nQ4LS569NsRRAce2BVanXw

---
deno が TS を捨てた話

- 内部 API 実装で Deno は最初は TS を使っていたが、それを辞めて素の JS を使うようになったという話

---
deno が TS を捨てた話

- 内部 API の TS は Deno のユーザーが使う TS とは全く別物で独自の専用プログラムでバンドルしていた
  - そもそもメンテできる人が限られていた
- dts 定義なども独自ツールで生成していたが、そのツールのバグで特定の型の API が定義出来ない不都合などがあった
  - メンテできる人もほぼ1人だった

=> 辛さが限界に達し、TS をやめた方が良いという判断になった

---
class: inverse middle center

Q. ORM比較

---
ORM 比較

- 現状だと denodb しか選択肢がなさそう。<br />参考: [Deno で試すデータベースアクセス](https://ccbaxy.xyz/blog/2021/07/01/js23/)

- [prisma は未対応](https://github.com/prisma/prisma/issues/2452)

- マイグレーションは [Nessie](https://github.com/halvardssm/deno-nessie) がそれなりに動いてる模様

(Deno コアチームは ORM に対する意識がおそらくかなり低い)

---
class: middle center

<img src="./assets/logo.svg" align="center" width="200" />

End
