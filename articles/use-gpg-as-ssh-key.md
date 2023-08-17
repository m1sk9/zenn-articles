---
title: "GPG鍵をSSH鍵として使う"
emoji: "🔑"
type: "tech"
topics:
  - "ssh"
  - "gpg"
published: true
published_at: "2023-03-19 04:02"
---

GPG鍵をSSH鍵として使おうとした際にいつもコケるので備忘録として

::: message

2023/03/19 に前ユーザーで投稿した記事を再編集したものです.

:::

----

- GnuPG をインストールするときに一緒にインストールされる gpg-agent は ssh-agent として使える
  - なので, GPG鍵をSSH鍵の代わりとして運用できる
- GPG鍵とかSSH鍵とかもう考えるのが面倒くさい！って人は一つにまとめてしまえば精神衛生がよくなる！

## 前提条件

- 私のメインは macOS なので pinentry-mac を使う.
- SSH鍵として使うのは `A` (`authentication`) の副鍵を使う.( `A` は SSH や TLS における認証に使える)
  - `gpg -k` または `gpg -K` したときに `usage: A` になってる鍵
  ```shell
  /Users/lis2a/.gnupg/pubring.kbx
  -------------------------------
  pub   ed25519 2023-03-13 [C]
    CDD5837D99D6BC1D6F5F5CB2002AEC385BEAEBCF
  sub   ed25519 2023-03-13 [S] [expires: 2024-03-12]
  sub   ed25519 2023-03-13 [A] [expires: 2024-03-12] # ←これ
  ```
  - もしないなら `gpg --edit-key` -> `addkey` で副鍵を追加する
    - 副鍵の追加には秘密鍵が必要です.

## gpg-agent の自動起動

GPG関連のコマンドを実行すると基本的 gpg-agent は自動で立ち上がるが, ssh-agent として使用する際は自動起動できない.

シェルの設定ファイルに gpg-agent を起動するよう設定する

```sh:.zshrc
gpg-connect-agent --quiet /bye
```

## pinentry-mac のインストール

pinentry-mac は GnuPG に付属してこないので手動でインストールする必要がある

```shell:shell
# コマンドからインストール
brew install pinentry-mac
```

:::message

Intel Mac と Apple Silicon 搭載の Mac では `pinentry-mac` のパスが変わっているため,明示的に指定しないと起動できず署名に失敗する.

```shell:shell
# Intel Mac
which pinentry-mac
> /usr/local/bin/pinentry-mac

# Apple Silicon
which pinentry-mac
> /opt/homebrew/bin/pinentry-mac
```

`~/.gnupg/gpg-agent.conf` で指定してあるパスを変えてあげると正常に起動する

```diff conf:gpg-agent.conf
- pinentry-program /usr/local/bin/pinentry-mac
+ pinentry-program /opt/homebrew/bin/pinentry-mac
```

:::

## ソケットファイルの場所指定

gpg-agent が参照する SSH用のソケットファイルは `~/.gnupg/S.gpg-agent.ssh` にある.

このソケットファイルへのパスは `gpgconf --list-dirs` で取得できるので `SSH_AUTH_SOCK` という環境変数として `.zprofile` に設定する

```sh:config.fish
set -x SSH_AUTH_SOCK "$(/opt/homebrew/bin/gpgconf --list-dirs agent-ssh-socket)"
```

## SSH鍵として使う鍵の指定

SSH鍵として使いたい鍵の keygrip を `~/.gnupg/sshcontrol` に指定する

keygrip を取得する

```shell:shell
gpg -K --with-keygrip
```

記述する

```shell:shell
echo '<KEYGRUP>' >> ${GNUPGHOME:-$HOME/.gnupg}/sshcontrol
```

:::message

[前提条件](#前提条件) に書いてある通り,SSH鍵には `usage` が `A` (`authentication`) の副鍵を使う.

:::

## SSH鍵を取得する

次のコードで SSH鍵 が吐き出される.

```shell:shell
ssh-add -L
```
