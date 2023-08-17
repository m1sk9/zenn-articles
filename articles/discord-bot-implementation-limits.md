---
title: "Discord において Bot の同時導入には限界がある...のかもしれない"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["discord", "bot"]
published: true
---

::: message

前ユーザーで投稿した記事を再編集したものです.

:::

----

# はじめに

- 今回の問題は **普通は起こり得ません** .
  - 私たちの Discord の使い方が異常だったために起こった問題です.
- 本件は既に Discord のサポートチームに報告済みです。本記事はあくまで記録のためです.

# TL;DR

Discord において、1つのサーバーにある一定数を超えた Bot が導入されると、 **Application Command をはじめとする Bot のほぼ全ての機能が使用不可/挙動がおかしくなる** ことがありました.

今回は不要・非アクティブの Bot をキックすることで解決しました.

# 背景

当コミュニティ [【限界開発鯖】](https://approvers.dev) では、主に Discord で活動しており、鯖民は自分たちが作った Bot をサーバー内に導入するということをよく行っています.

明確なガイドラインを制定せず3年ほど、ほぼオープンにしていた結果サーバー内には **約70個の Bot** が導入されていました.

![2023年8月. 69個のBotが導入されています。](/images/discord-bot-limit/image-1.jpg)

# 起きた問題

この状況を放置していた結果、次のような問題が発生しました。

## Slash Commands が使用できなくなる

Discord には、2021年に導入された [Slash Commands](https://discord.com/developers/docs/interactions/application-commands#slash-commands) という機能があります.

本来予め用意されたエンドポイントを叩くことで、ギルドまたはBot単位で登録し使用する仕組みのためドキュメントの通りサーバーに対して実行すれば使用できるはずが **`/` のスラッシュコマンドリストにも出てこなく、使用できなくなりました** .

> 前述した該当不具合については #無法地帯 でいくらか話が出たが以下のような不具合が発生している
>
> - Slash CommandをサポートしているBotを導入してもSlash CommandがUI上に出現せず、利用できない (一部のメンバーは利用できる)
> - Message Context Menuを該当ギルドに登録してもUI上に出現しない (登録した本人は利用できる) 正しくは登録した本人とギルドオーナーは使える模様
> - サーバーメニューの Integrations に該当Botが出現しない (本来であれば出現するはず)
>
> この不具合はすでにDiscord 運営の方に報告しているが、３ヶ月程全く話が進行していない
> [スラッシュコマンドへの移行 #560 - approvers/OreOreBot2](https://github.com/approvers/OreOreBot2/issues/560)

これは自分たちが開発した Bot ではなく、公開された Bot でも同じ状況でした.

## Context Menu の挙動がおかしくなる

同じく、 Application Command である [Context Menu (Message)](https://discord.com/developers/docs/interactions/application-commands#message-commands) についても、使用不可...とまでは行きませんが挙動がおかしくなりました.

- デバイス、権限など関係なしに **Context Menu が表示されず利用できない**.
- 一部のメンバーには **Context Menu が表示されるが、選択しても反応しない/正常に動作しない**.

この問題はスラッシュコマンドとは異なり、 一部のメンバーが使えることから **権限設定が不正に動作することから発生している** と考えられます.

:::message

**Application Command の権限設定について:**

2023/02/28 以降、 Application Command の権限設定は**ギルドベース**で動作するようになっています.
(ギルドのモデレーターが詳細に設定できるようになった. ということ)

> Command-level permissions changes now act as an “override” of app-level configurations. This makes them function more like server permissions and channel overwrites.
> [Updates to Command Permissions - Discord ヘルプセンター](https://support.discord.com/hc/en-us/articles/10952896421783) (全文英語)

つまり、ギルドのモデレーターが設定した権限が優先されるようになったということです.

:::

### 権限のオーバーライドとは

先程 **ギルドのモデレーターが設定した権限が優先されるようになった** と言いましたが、例外として Context Menu を登録するエンドポイントに `default_member_permissions` を指定すれば開発者側からオーバーライドできます.

本件では `default_member_permissions` を正しく設定しているにも関わらずコマンド実行にギルド側から勝手に **管理者権限** を要求されるという状況でした. (メンバーが設定したわけではない.)

![管理者権限がギルド側から勝手にオーバーライドされている様子](/images/discord-bot-limit/image-2.png)

:::message

実際には `default_member_permissions` でオーバーライドしなくても実行できるので、この時点で挙動がおかしいことがわかります.

:::

## Integrations (連携サービス) ページに表示されていない Bot がいる

Discord Bot は **連携サービス** という扱いのため、**ギルドに導入する = ギルドに連携する** ということになります.

そのため、本来であれば導入した Bot はすべて **Integrations (連携サービス)** ページに表示されます.

こちらに関しても挙動がおかしく、最低でも2020年8月以降に導入された Bot はすべて表示されませんでした.

**Botの大量キック後のIntegrationsページ:**

![Botの大量キック後のIntegrationsページ。2020年8月以降に導入された Bot が正しく表示されている](/images/discord-bot-limit/image-3.jpg)

# まとめ

Discord Support からの返信がまだなので、本件を **導入している Bot の数が多すぎる** と断言するのは難しいです.

ただし、これが仕様なのであれば **ギルドに導入する Bot の数には限界がある** ということになります.

今回のようなケースはかなり稀であったのか、文献等が少なく解決には1年という時間を費やしてしまいました。またこの間にもスラッシュコマンドが使えないわけなので開発活動にも大きな影響をもたらしました。

仕様とするなら制限を設けるなど同じような問題が発生しないようにしてほしいと考えています.

----

### Off-topic

人のチケットを1年近く放置するのはやめようね、Discord Support

今回の件について、2022年7月に報告したのにそれから約1年ちょっと返信せず放置されていて、少し遺憾です.

70個近くの Bot を導入している私たちが異常という意見には同意します.
