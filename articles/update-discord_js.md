---
title: "discord.js v14へのアップデート"
emoji: "✊"
type: "tech"
topics:
  - "discord"
  - "discordjs"
published: true
---

::: message

Qiitaで投稿した記事を再編集したものです.

:::

----

2022/07/18, Discord.js v14がリリースされました。
いつも通り破壊的変更がいっぱいなので、そのまま `yarn upgrade` しても動きません。注意しましょう。

:::message alert

公式の [Updating from v13 to v14](https://discordjs.guide/additional-info/changes-in-v14.html) を翻訳したものになります。

翻訳が異なっている場合があるのでもしありましたら、編集リクエストを送っていただけると助かります。
:::

## 変更内容

変更内容は公式リポジトリの [CHANGELOG](https://github.com/discordjs/discord.js/blob/main/packages/discord.js/CHANGELOG.md#1400---2022-07-17) を確認してください。(メジャーアップデートなので膨大です)

## 始める前に

Discord.js は Node.js v16.9以上を要求します。

ターミナルにて `node -v` を実行して今自分がどのNodeを使用しているかを確認し、 v16.9以下だったらアップデートを行いましょう。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/666481/b0a3f0b8-1f10-23ee-91d0-02557e923134.png)

----

## Builderがdiscord.js v14と統合された

スラッシュコマンドなどの登録に使用した `@discordjs/builder` はdiscord.js v14に統合されました。もし、単体の `@discordjs/builder` がプロジェクトにある場合はパッケージ名の衝突を回避するため、パッケージをアンインストールしましょう。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/666481/49ee4f6a-4dd9-3294-3fc6-136a872a85ff.png)


```shell
## npm
npm uninstall @discordjs/builders

## yarn
yarn remove @discordjs/builders

## pnpm
pnpm remove @discordjs/builders
```

## 破壊的変更

### API バージョン

discord.js v14はDiscord API v10に切り替わっています。

### 列挙型値

enumパラメータに文字列または数値型を受け入れていた全ての領域は排他的に数値を受け入れるようになっています。

discord.js v13以下でエクスポートされるenumは discord-api-types の新しいenumに置き換わります。

#### 新しいenumの違い

discord.jsのenumとdiscord-api-typesの違いは次の通り

1. 列挙型は単数形で表現されるようになります。つまり `ApplicationCommandOptionTypes` は`ApplicationCommandOptionType` となります。
2. `Message` の接頭辞を持つ列挙型はMessageの列挙型を持たなくなりました。つまり `MessageButtonStyles` は `ButtonStyle` となります。
3. Enumの値は `SCREAMING_SNAKE_CASE` ではなく `PascalCase` になっています。つまり `CHAT_INPUT` は `ChatInput` となります。

:::message alert

enum値の代わりに生の数値([マジックナンバー](https://en.wikipedia.org/wiki/Magic_number_(programming)))を使いたくなるかもしれませんが、あまり望まれません。

enumはより読みやすく、APIの変更に強いため、コードの可読性が落ちやすいマジックナンバーを使わずなるべくenumを使うようにしましょう。

:::

### 一般的な列挙型の破壊

クライアント初期化、JSONスラッシュコマンド、JSONメッセージコンポーネントなどの領域はこれらの変更に対応するため、修正が必要になる可能性があります。

#### クライアントの初期化

```diff
- const { Client, Intents } = require('discord.js');
+ const { Client, GatewayIntentBits, Partials } = require('discord.js');

- const client = new Client({ intents: [Intents.FLAGS.GUILDS], partials: ['CHANNEL'] });
+ const client = new Client({ intents: [GatewayIntentBits.Guilds], partials: [Partials.Channel] });
```


(Intentの指定に新たに `GatewayIntentBits` が参加しています。)

#### アプリケーションコマンドのデータ変更

```diff
+ const { ApplicationCommandType, ApplicationCommandOptionType } = require('discord.js');

const command = {
  name: 'ping',
- type: 'CHAT_INPUT',
+ type: ApplicationCommandType.ChatInput,
  options: [
    name: 'option',
    description: 'A sample option',
-   type: 'STRING',
+   type: ApplicationCommandOptionType.String,
  ],
};
```

#### メッセージボタンのデータ変更

```diff
+ const { ButtonStyle } = require('discord.js');

const button = {
  label: 'test',
- style: 'PRIMARY',
+ style: ButtonStyle.Primary,
  customId: '1234'
}
```

### メソッドベースの型保護の削除

#### Channel

いくつかのチャンネルタイプガードメソッドは一つのチャンネルタイプを絞り込むように削除されました。

かわりにチャンネルを絞り込むためにタイププロパティーを `ChannelType` と比較します。

```diff
-channel.isText()
+channel.type === ChannelType.GuildText

-channel.isVoice()
+channel.type === ChannelType.GuildVoice

-channel.isDM()
+channel.type === ChannelType.DM
```

#### Interaction

チャンネルと同様にいくつかインタラクションタイプガードが削除されました。

```diff
-interaction.isCommand();
+interaction.type === InteractionType.ApplicationCommand;

-interaction.isAutocomplete();
+interaction.type === InteractionType.ApplicationCommandAutocomplete;

-interaction.isMessageComponent();
+interaction.type === InteractionType.MessageComponent;

-interaction.isModalSubmit();
+interaction.type === InteractionType.ModalSubmit;
```

#### Builder

ビルダーは以前のようにDiscord APIから返されなくなりました。

例えばDiscord APIに `EmbedBuilder` を送るとDiscord APIから同じデータのEmbedが帰ってきます。これは受け取ったコンポーネントなどの構造をコードで処理する方法に影響を与えます。

#### `create()`, `edit()` パラメータの統合

マネージャーやオブジェクトの様々な `create()`, `edit()` メソッドのパラメータが統合されました。

- `Guild#edit()` のデータ・パラメーターに `reason` が追加されました。
- `GuildChannel#edit()` のデータ・パラメータに `reason` が追加されました。
- `GuildEmoji#edit()` のデータパラメータに `reason` が追加されました。
- `Role#edit()` がデータパラメータに `reason` が追加されました。
- `Sticker#edit()` がデータパラメータに `reason` が追加されました。
- `ThreadChannel#edit()` がデータパラメータ `reason` が追加されました。
- `GuildChannelManager#create()` でオプションパラメータに `name` を指定するように変更
- `GuildChannelManager#createWebhook()` (およびその他のテキストベースのチャネル) はオプションパラメータにチャネルと名前を取るようになった。
- `GuildChannelManager#edit()` がデータの一部として reason を受け取るようになった。
- `GuildEmojiManager#edit()` はデータの一部として `reason` を受け取るようになりました。
- `GuildManager#create()` でオプションの一部として `name` を受け取るように変更
- `GuildMemberManager#edit()` が `reason` をデータの一部として受け取るように変更
- `GuildMember#edit()` が `reason` をデータの一部として受け取るように。
- `GuildStickerManager#edit()` がデータの一部として `reason` を取るように。
- `RoleManager#edit()` がオプションの一部として `reason` を取るようになった。
- `Webhook#edit()` がオプションの一部として `reason` を取るようになった。
- `GuildEmojiManager#create()` がオプションの一部として添付ファイルと名前を取るようになった。
- `GuildStickerManager#create()` がオプションの一部としてファイル、名前、タグを取るようになりました。

#### Activity

以下のプロパティーはDiscord Developer Documentationに記載されていないため、削除されました。

- `Activity#id`
- `Activity#platform`
- `Activity#sessionId`
- `Activity#syncId`

#### Application
`Application#fetchAssets()` は、Discord APIでサポートされなくなったため、削除されました。

### BitField

BitField 構成要素に BitField サフィックスが追加され、enumとの命名衝突を回避できるようになりました。

```diff
- new Permissions()
+ new PermissionsBitField()

- new MessageFlags()
+ new MessageFlagsBitField()

- new ThreadMemberFlags()
+ new ThreadMemberFlagsBitField()

- new UserFlags()
+ new UserFlagsBitField()

- new SystemChannelFlags()
+ new SystemChannelFlagsBitField()

- new ApplicationFlags()
+ new ApplicationFlagsBitField()

- new Intents()
+ new IntentsBitField()

- new ActivityFlags()
+ new ActivityFlagsBitField()
```

- `#FLAGS` は `#Flags` に名称変更されました。

#### CDN

CDNのURLを返すメソッドは動的な画像のURLを返すようになりました。(利用可能な場合のみ)

この動作は `ImageURLOptions` パラメータで `forceStatic` を `true` に設定することでオーバーライドできます。

#### CategoryChannel

`CategoryChannel#children` はカテゴリーが含むチャンネルのCollectionではなくなりました。

v14からはManagerとなります。(`CategoryChannelChildManager`)

これは、 `CategoryChannel#createChannel()` が `CategoryChannelChildManager` に移動したことも意味します。

#### Channel

以下のタイプガードが削除されました。

- `Channel#isText()`
- `Channel#isVoice()`
- `Channel#isDirectory()`
- `Channel#isDM()`
- `Channel#isGroupDM()`
- `Channel#isCategory()`
- `Channel#isNews()`

#### CommandInteractionOptionResolver

`CommandInteractionOptionResolver#getMember()` は `required` (必須)のパラメータを持たなくなりました。

詳細は以下のプルリクエストを確認してください。

[discordjs/discord.js - refactor: remove required from getMember #7188](https://github.com/discordjs/discord.js/pull/7188)

#### Constants

- 多くの定数オブジェクトやキー配列がトップレベルのエクスポートになりました。

```diff
- const { Constants } = require('discord.js');
- const { Colors } = Constants;
+ const { Colors } = require('discord.js');
```

- リファクタリングされた定数構造体は、`SCREAMING_SNAKE_CASE` のメンバ名ではなく、`PascalCase` のメンバ名を持っています。
- エクスポートされた定数構造体の多くが置換され、名前が変更されました。

```diff
- Opcodes
+ GatewayOpcodes

- WSEvents
+ GatewayDispatchEvents

- WSCodes
+ GatewayCloseCodes

- InviteScopes
+ OAuth2Scopes
```

#### Event

(not GuildEvents.)

- `message`, `interaction` イベントは削除されました。今後は `messageCreate`, `interactionCreate` イベントを使用しましょう。
- `applicationCommandCreate`, `applicationCommandDelete`, `applicationCommandUpdate` は、すべて削除されました。
    - 詳細については以下のプルリクエストを参照してください。
    - [discordjs/discord.js - refactor(Client): remove applicationCommand events #6492](https://github.com/discordjs/discord.js/pull/6492)

#### GuildBanManager

ユーザーをBANする際の日数オプションは、Discord APIに合わせ、 `deleteMessageDays` に名称が変更されました。

#### Guild

- `Guild#setRolePositions()` と `Guild#setChannelPositions()` が削除されました。
    - `RoleManager#setPositions()` と `GuildChannelManager#setPositions()` をそれぞれ使用してください。
- `Guild#maximumPresences` のデフォルト値が `25,000` でなくなりました。
- `Guild#me` は `GuildMemberManager#me` に移動されました。
    - 詳しくは以下のプルリクエストを参照してください。
    - [discordjs/discord.js - feat: move me to GuildMemberManager manager](https://github.com/discordjs/discord.js/pull/7669)

#### GuildAuditLogs & GuildAuditLogsEntry

- `GuildAuditLogs.build()` は廃止されたと判断されたため、削除されました。
- 以下のプロパティとメソッドは、`GuildAuditLogsEntry` クラスに移動されました。
    - `GuildAuditLogs.Targets`
    - `GuildAuditLogs.actionType()`
    - `GuildAuditLogs.targetType()`

#### GuildMember

`GuildMember#pending` が `null` に指定できるようになり、ギルドメンバーが部分的であることを考慮するようになりました。

詳しくは以下のIssueを確認してください。

[discordjs/discord.js - Misleading GuildMember#pending boolean on partial guild members #6546](https://github.com/discordjs/discord.js/issues/6546)

#### IntegrationApplication

`IntegrationApplication#summary` は、APIでサポートされなくなったため削除されました。

#### Interaction

以下のタイプガードは削除されました。

```diff
- interaction.isCommand()
- interaction.isContextMenu()
- interaction.isAutocomplete()
- interaction.isModalSubmit()
- interaction.isMessageComponent()
```

また、インタラクションに応答する際、ギルドがキャッシュされていないとAPI Messageが返されることがありましたが、インタラクションの返信は常にdiscord.jsのMessageオブジェクトが返るようになりました。

#### Invite

`Invite#channel` と `Invite#inviter` がゲッターになり、キャッシュから構造体を解決するようになった

#### MessageComponent

- `MessageComponents` の名前も変更されました。`Message` という接頭辞はなくなり、`Builder` という接尾辞がつくようになりました。

```diff
- const button = new MessageButton();
+ const button = new ButtonBuilder();

- const selectMenu = new MessageSelectMenu();
+ const selectMenu = new SelectMenuBuilder();

- const actionRow = new MessageActionRow();
+ const actionRow = new ActionRowBuilder();

- const textInput = new TextInputComponent();
+ const textInput = new TextInputBuilder();
```

- APIから受け取ったコンポーネントは直接改変させることは出来ません。APIからコンポーネントを改変させたい場合は `ComponentBuilder#from` を使用します。例えばボタンを `Mutable` にしたい場合は以下のような変更を行います。

```diff
- const editedButton = receivedButton
-   .setDisabled(true);

+ const { ButtonBuilder } = require('discord.js');
+ const editedButton = ButtonBuilder.from(receivedButton)
+   .setDisabled(true);
```

#### MesssageManager

`MessageManager#fetch()` の第2パラメータが削除されました。

第2パラメータであった `BaseFetchOptions` は最初のパラメーターに統合されています。

```diff
- messageManager.fetch('1234567890', { cache: false, force: true });
+ messageManager.fetch({ message: '1234567890', cache: false, force: true });
```

#### MessageSelectMenu

- `MessageSelectMenu` は `SelectMenuBuilder` に名称変更されました。
- `SelectMenuBuilder#addOption()` が削除されました。
    - 代替として `SelectMenuBuilder#addOptions()` を使います。

#### MessageEmbed

- `MessageEmbed` は `EmbedBuilder` に名称変更されました。
- `EmbedBuilder#setAuthor()` は、単独の `EmbedAuthorOptions` オブジェクトを受け入れるようになりました。
- `EmbedBuilder#setFooter()` は、新しい唯一の `FooterOptions` を受け入れるようになりました。
- `EmbedBuilder#addField()` が削除されました。
    - 代替として `EmbedBuilder#addFields()` を使います。

```diff
- new MessageEmbed().addField('Inline field title', 'Some value here', true);

+ new EmbedBuilder().addFields([
+  { name: 'one', value: 'one' },
+  { name: 'two', value: 'two' },
+]);
```

##### Example

```js
const { Client, Intents, EmbedBuilder } = require("discord.js");

const client = new Client({ intents: [ Intents.Flags.GUILDS ] });

const embed = new EmbedBuilder()
    .setTitle('Test')
    .setDescription('Test')
    .setColor('#0099ff')

client.on('messageCreate', (message) => {
    message.reply({ embeds: [embed] })
})

client.login('token');
```

#### PartialTypes

`PartialTypes` 文字列配列は削除されました。代わりに `Partials` 列挙型を使用します。

これに加えて、新しいパーシャル `Partials.ThreadMember` が追加されました。

#### PermissionOverwritesManager

上書きは `SCREAMING_SNAKE_CASE` パーミッションキーではなく、`PascalCase` パーミッションキーで行われるようになりました。

#### REST Events

以下のイベントは **`Client` から**削除されました。

- `apiRequest`
- `apiResponse`
- `invalidRequestWarning`
- `rateLimit`

これらのイベントには，`Client#rest` からアクセスする必要があります。

また、これらの `apiRequest`, `apiResponse`, `rateLimit` のイベント名が変更されています。

```diff
- client.on('apiRequest', ...);
+ client.rest.on('request', ...);

- client.on('apiResponse', ...);
+ client.rest.on('response', ...);

- client.on('rateLimit', ...);
+ client.rest.on('rateLimited', ...);
```

#### RoleManager

`Role.comparePositions()` が削除されました。代わりに 
 `RoleManager#comparePositions()` を使ってください。

#### Sticker

`Sticker#tags` は null 可能な文字列 `(string | null)` になりました。

[discordjs/discord.js - refact: Sticker's tags property #8010](https://github.com/discordjs/discord.js/pull/8010)

#### ThreadChannel

`ThreadAutoArchiveDuration` で使われていた `MAX` ヘルパーは削除されました。

#### ThreadMemberManager

- `ThreadMemberManager#fetch()` の第二パラメータが削除された。かつて第2パラメータであった `BaseFetchOptions` は、現在、第1パラメータに統合されています。
- キャッシュを指定するための `boolean` ヘルパーが削除されました。

```diff
// The second parameter is merged into the first parameter.
- threadMemberManager.fetch('1234567890', { cache: false, force: true });
+ threadMemberManager.fetch({ member: '1234567890', cache: false, force: true });

// The lone boolean has been removed. One must be explicit here.
- threadMemberManager.fetch(false);
+ threadMemberManager.fetch({ cache: false });
```

#### Util

- `Util.removeMentions()` は削除されました。メンションを制御するには、代わりに `MessageOptions` の `allowedMentions` を使う必要があります。
- `Util.splitMessage()` は削除されました。
- `Util.resolveAutoArchiveMaxLimit()` は削除されました。

#### `.deleted` Field(s)

`.deleted Field(s)` が削除されました。

deleted プロパティは、構造体が削除されたかどうかを確認するために使用できなくなります。

詳細については、このIssueを参照してください。

[discordjs/discord.js - Removal of .deleted in structures #7091](https://github.com/discordjs/discord.js/issues/7091)

#### VoiceChannel

`VoiceChannel#editable` は削除されました。代わりに `GuildChannel#manageable` を使ってください。

類似の列挙型の多くは [discord-api-types のドキュメントで見ることができます](https://discord-api-types.dev/api/discord-api-types-v10/enum/ActivityFlags)。

#### VoiceRegion

`VoiceRegion#vip` は削除されました。(Discord APIから削除されたため)

#### Webhook

`Webhook#fetchMessage()` は、`WebhookFetchMessageOptions` タイプの唯一のオブジェクトを受け取るようになりました。

### Features

#### AutocompleteInteraction

呼び出されたアプリケーションコマンドが登録されているギルドのIDである `AutocompleteInteraction#commandGuildId` が追加されました。

#### Channel

- ストアチャンネルは削除されました。(Discord APIから削除されたため)
- `Channel#url` が追加され、クライアントと同じようにチャンネルへのリンクになりました。
- 新しいタイプガードも追加されました。
    - `Channel#isDMBased()`
    - `Channel#isTextBased()`
    - `Channel#isVoiceBased()`

#### Collection

- `Collection#merge()`, `Collection#combineEntries()` を追加されました。。
- イミュータブルコレクションを示すReadonlyCollectionが追加されました。

#### Collector

コレクターによって収集されない要素があるときに発行される新しい `ignore` イベントが追加されました。

#### CommandInteraction

呼び出されたアプリケーションコマンドが登録されているギルドの ID である `CommandInteraction#commandGuildId` が追加されました。

#### Guild

ギルドの[MFAレベル(管理レベル)](https://discord.com/developers/docs/resources/guild#guild-object-mfa-level)を設定する `Guild#setMFALevel()` が追加されました。

```js
client.on('guildCreate', (guild) => {
    guild.setMFALevel(0, 'not request 2fa')
    guild.setMFALevel(1, 'request 2fa')
})
```

#### GuildChannelManager

チャンネル作成時に `videoQualityMode` を使用して、カメラのビデオ品質モードを初期設定することができます。

#### GuildMemberManager

`GuildMemberManager#fetchMe()` が追加され、ギルド内のクライアントユーザーを取得するようになりました。

#### GuildEmojiManager

既存のギルド絵文字を管理するために `GuildEmojiManager#delete()` と `GuildEmojiManager#edit()` が追加されました。

#### Interaction

与えられたインタラクションに返信できるかどうかを確認できる `Interaction#isRepliable()` が追加されました。

```js
client.on('interactionCreate', async (interaction) => {
    interaction.isRepliable()
})
```

#### MessageReaction

クラスが属するリアクションでクライアントユーザーを反応できるように `MessageReaction#react()` を追加されました、

#### Unsafe Builders

Unsafe ビルダーは、入力のバリデーションを行わないことを除いて、通常のビルダーと全く同じように動作します。Unsafe ビルダーは、通常のビルダーに `Unsafe` というプレフィックスを付けることで命名されます。

#### Webhook

`Webhook#applicationId` を追加しました。


