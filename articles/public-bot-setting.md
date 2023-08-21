---
title: "Discord Bot の招待リンクは自作できてしまう"
emoji: "🔒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["discord", "bot"]
published: true
---

## TL;DR

**招待リンクは自分で作成できるので, 公開しない Bot は Public Bot をOFFにしよう.**

## 招待リンクの仕組み

Discord Bot を招待するリンクは, OAuth2 URL と呼ばれるもので以下のような構成になっています. (あくまで一例. SCOPE によって異なる.)

```md
https://discord.com/api/oauth2/authorize?client_id=<client_id>&permissions=<permission_bit>&scope=bot
```

- `<client_id>`: Bot の ユーザーID と同一で、開発者モードが有効であれば誰でも入手可能.
- `<permission_bit>`: [Discord API の権限値](https://discord.com/developers/docs/topics/permissions#permissions-bitwise-permission-flags). 指定することで導入時のデフォルト権限を指定できる. `0` で権限指定なしを表す.

仕組みを見ていただければわかるが, どれも自分で指定することができるため基本的に **招待リンクは自作可能** であることがわかります.

これがどういうことかというと, **開発者は公開を想定していないのに知らぬ間に無関係のギルドに導入されてしまう** ということです.

## 回避方法

この問題を回避するため Discord には **Public Bot** という設定があります.

OFF にすると **自分以外** のユーザーが特定の Bot の招待リンクを自作しても導入することができなくなります.

![Discord Developer Portal の "Bot" メニュー](/images/public-bot-setting/image-1.jpg)

----

この設定を知らない人をよく見かけるので、もし **特定のギルド向けに Bot を開発している** のであれば確認しておくことをおすすめします.
