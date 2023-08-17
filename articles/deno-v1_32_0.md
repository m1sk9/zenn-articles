---
title: "Deno v1.32.0 には脆弱性が存在する"
emoji: "🦕"
type: "tech"
topics:
  - "javascript"
  - "typescript"
  - "deno"
  - "security"
  - "cve"
published: true
published_at: "2023-04-08 13:27"
---

::: message

2023/04/08 に前ユーザーで投稿した記事を再編集したものです.

:::

----

すでに Deno 公式からはアナウンスがありましたが、 3/22 にリリースされた v1.32.0 を紹介する各ブログ記事ではこのことに触れられていなかったため、当記事を書いています。

:::message alert

Deno 公式のアナウンスとこの記事で差異があった場合は Deno 公式のアナウンスを優先して確認してください。

:::

## Deno v1.32.0 とは

Deno v1.32.0 は 2023年3月22日 にリリースされたバージョンです。

https://deno.com/blog/v1.32

このバージョンでは主に以下の変更が行われました。

- Node.js との互換性が更に強化
- Web Workers ・ dynamic import に対する `deno compile` のサポート
- `deno run` で拡張子のないファイルを実行できるように
- Deno API ・ Web API にいくつかの変更
- 標準ライブラリにいくつかの破壊的な変更
- TypeScript 5.0 に対応

`package.json` で指定されている npm モジュールを `npm:` 指定なしにインポートできるようにもなってます。

実質 Node.js かそれ以上のランタイムになりましたね、不覚にも笑ってしまいました。

https://twitter.com/deno_land/status/1644370721509097474

https://twitter.com/lis2a_o0/status/1644385860392210432

## 今回の該当脆弱性 `CVE-2023-28445` とは

`ArrayBuffer` において領域圏外へのアクセスが可能になってしまう脆弱性です。

https://github.com/denoland/deno/security/advisories/GHSA-c25x-cm9x-qqgx

Deno Deploy のユーザーはこの影響を受けることはありません。

この脆弱性は v1.32.0 のみに存在しているため、 **v1.31.3 のままにしておく**か、 **`ArrayBuffer` が無効化された v1.32.1 または不具合が修正された v1.32.2 にアップデートする**ことで修正ができます。

## 脆弱性への対処

脆弱性 `CVE-2023-28445` への対処はいずれかの手順を踏みます。

### ダウングレードする

ダウングレードは `deno upgrade` に `--version` フラグを使用して特定のバージョンをインストールする機能を応用することで可能です。

```shell
deno upgrade --version 1.31.3
```

### アップデートする

3/24 にリリースされた v1.32.1 では `ArrayBuffer` 等を無効化する変更 ( v1.32.2 では修正 ) が行われているので、ローカルにある Deno のバージョンが v1.32.0 の場合は `deno upgrade` を使い、アップデートを行います。

https://github.com/denoland/deno/releases/tag/v1.32.1

```shell
# バージョンを確認
deno --version
>> 1.32.0

# アップデート
deno upgrade
```

### 直接無効化

諸事情でアップデートが出来ない場合は `deno run` での実行時に `--v8-flags` フラグを使用することで `ArrayBuffers` を無効化したまま実行することが可能です。

```shell
deno run --v8-flags=--no-harmony-rab-gsab ...
```

----

## まとめ

Deno の公式ブログでの v1.32.0 告知記事では v1.32.1 へのアップデートがアナウンスされていましたが、公式Twitterではそのような文言はなかったため、私も「実質 Node じゃん」のくだりまでは全く気づきませんでした。(幸い、 Homebrew が v1.32.2 を落としていたので何事もなかったですが....)

今回の脆弱性 `CVE-2023-28445` のベーススコアは `9.9` (深刻度は `Critical` ) と認定されているので、そこそこ大きい脆弱性です。

**ご自身のローカルの Deno がもし v1.32.0 のままだったら直ぐにアップデートを行って修正パッチを適用してくださいね。**

### 参考

https://deno.land/manual@v1.32.3/getting_started/installation#updating
