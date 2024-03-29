---
title: "Alacrittyがすごい"
emoji: "🚀"
type: "tech"
topics:
  - "rust"
  - "cli"
  - "terminal"
published: true
publication_name: "approvers"
published_at: "2023-03-19 14:29"
---

::: message

2023/03/19 に前ユーザーで投稿した記事を再編集したものです.

:::

----

:::message

###### 追記・修正履歴

2023/03/30 5:15: [macOS 版特有の署名問題](#homebrew-の-cask-を使用する) の bypass (回避方法) を修正

:::

今年1月に MacBook を購入してからずっと iTerm2 という macOS 向けに開発されたターミナルを使っていました (オススメしてる人が多かったからね.)

https://iterm2.com/

ただ、私は定期的に初期化したくなる初期化癖を持っているので iTerm2 で設定していた内容が初期化するたびに吹っ飛んでしまい、dotfiles の**インストーラーを実行すればセットアップが完了する** という完璧な状態を実現しても、iTerm2 が吐き出す設定ファイルの形式が JSON なのでどうしても dotfiles での管理が厳しいです.

そんな中、Rustで出来たターミナル **Alacritty** に出会いました.最初は試しに使おうとしただけだったのに結構快適でやめられなくなってしまったので、この記事で Alacritty ユーザーを増やそうと思います.

https://github.com/alacritty/alacritty

![](https://user-images.githubusercontent.com/127779256/226105371-f8eb9b1e-9e15-44d6-8cc8-801bebb75be1.png)

## Alacritty って何？

Alacritty の概要に関しては先駆者がいろいろ記事を書いているのでここではざっくりとしか紹介しません.

----

Alacritty は **高速**, **クロスプラットフォーム** を売りにしているターミナルエミュレーターです.

Rust で作られているので、**macOS**, **Linux** などのプラットフォームに対応しています.(Windows というOSも対応しています.)

GPU を利用し、高速なレンダリングを実現しています.

## インストール

3種類のインストール方法が用意されています:

- ビルド済みバイナリを使用する
- homebrew の cask を使用する
- cargo を使用する

ソースコードからビルドする ~~茨の道~~ コースもあります.ここでは長たらしくやる人も多分少ないので紹介しません.

### ビルド済みバイナリを使用する

Alacritty の Releases からビルド済みのバイナリを入手できます.

https://github.com/alacritty/alacritty/releases

### homebrew の cask を使用する

```shell
brew install --cask alacritty
```

Brewfileを使用する場合:

```brewfile
tap "homebrew/cask"
tap "homebrew/cask-versions"

cask "alacritty"
```

:::message alert

Alacritty は Apple に対して証明書を提出していないため、 macOS 上で Alacritty を実行しようとするとGatekeeperに妨害されます.(Finder から右クリックで開くと bypass して開くことが出来ますが.... よろしいことではありません.)

ある利用者がこれらの問題を Alacritty 内の Issue に提起していますが、Alacritty のメンテナたちはこの提出や開発者プログラムへの参加を拒んでいます.

彼らが何を危惧して提出を拒んでいるのかは分かりません.気になる人は以下の Issue を確認してみてください.

[Notarize macOS Binaries・Issue #4673](https://github.com/alacritty/alacritty/issues/4673)

[“Alacritty” can’t be opened because Apple cannot check it for malicious software.・Issue #6500](https://github.com/alacritty/alacritty/issues/6500)

:::

### cargo を使用する

Rust諸々をインストールした状態で

```shell
cargo install alacritty
```

## 何がいいの？

先述したように、 Alacritty の概要は他の先駆者に任せてここでは個人的にいいと思ったポイントを書いていきます.

### YAML 形式で設定できる

Alacritty の設定は YAML 形式で記述していきます.

私が Alacritty に惹かれた一つの理由がこれです.dotfiles管理ができるからです.

読み込み順は次の通り:

1. `$XDG_CONFIG_HOME/alacritty/alacritty.yml`
2. `$XDG_CONFIG_HOME/alacritty.yml`
3. `$HOME/.config/alacritty/alacritty.yml`
4. `$HOME/.alacritty.yml`

[^1]

[^1]: Windows は `%APPDATA%\alacritty\alacritty.yml`

iTerm2 は JSON でしか吐き出せなかったのでこのような YAML 形式で完結するのは有り難い限りです.

設定はこんな感じ.公式で設定例をコメントありで作ってくれているので設定しやすいです.

```yaml:alacritty.yml
font:
  size: 14
  normal:
    family: 'JetBrainsMonoNL Nerd Font Mono'

window:
  dimensions:
    columns: 184
    lines: 46
  decorations: full
```

https://github.com/alacritty/alacritty/blob/master/alacritty.yml

https://github.com/lis2a/dotfiles/blob/main/.config/alacritty/alacritty.yml

### tmux との連携が楽

tmux を使っていると **ターミナル起動に自動でセッションにアタッチしたいな〜** という感情になることがあります.

iTerm2 を使っている頃は次のような可読性がいろいろ終わっているスクリプトを `.zshrc` に組んでいました.

```sh
count=`ps aux | grep tmux | grep -v grep | wc -l`
if test $count -eq 0; then
    echo `tmux`
else test $count -eq 1; then
    echo `tmux a`
fi
```

Alacritty には `shell` というキーでシェルへのパスを指定することが出来ます.

Alacritty 起動時のシェルを zsh にした状態で `tmux` のコマンドを実行するように可読性を維持したまま設定できます.

```yaml:alacritty.yml
shell:
  program: /bin/zsh
  args:
    - -l
    - -c
    - "tmux a -t 0 || tmux"
```

どっちもやってることは同じです.セッションがすでに存在していればそのセッションにアタッチしセッションが存在していない場合は新しいセッションを作成します.

### マルチプラットフォーム

先述した通り Alacritty は YAML で設定ができ dotfiles での管理が可能でなおかつマルチプラットフォームのため macOS 以外でも Linux などで同様の環境を構築できちゃいます.

iTerm2 は macOS 専用だったのでこのような運用は難しかったです. Alacritty じゃなくてもできるけどこれは嬉しい.

(Windows は [^1] で書いてる通り `%APPDATA%` 配下からしか読み込めないので難しいかも？まあ Windows で開発することなんて多分一生無いので大丈夫です.)

## まとめ

まだ数ヶ月しか Alacritty を使っていません.が、ここまで書いていて多分しばらくは他のターミナルに浮気とかは出来ないと思います.

[homebrew の cask を使用する](#homebrew-の-cask-を使用する) の辺りで出した Alacritty のメンテナの意欲を見ていると、何かあったときに [colors.js みたいなこと](https://qiita.com/SnykSec/items/23bcd8dc873239d2bece)しそうでちょっと**うーんという顔をしています**.

(比較対象がそもそも npm コミュニティ だし、colors.js やそれこそ前に[ウクライナ侵攻の抗議目的でサプライチェーン攻撃](https://qiita.com/SnykSec/items/6e75bd78b4deb3371550)を仕掛けた peacenotwar たちが異常すぎるというのも事実なので重く考えすぎというのもありますが...)
