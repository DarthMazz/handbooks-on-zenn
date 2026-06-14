---
title: "Macの初期設定で入れたものまとめ"
emoji: "🍎"
type: "tech"
topics: ["mac", "homebrew", "copilot", "zed"]
published: true
---

# Macの初期設定で入れたものまとめ

新しい Mac を使い始めるときに、毎回最初に入れている設定とツールをまとめました。

この記事では、最低限の初期設定として以下を入れます。

- `.zshrc` の表示設定
- Homebrew
- GitHub Copilot CLI
- Zed

## `.zshrc` の設定

まずはターミナルの表示を少しだけ見やすくします。

```bash
export PS1="mod728@Mac ~ "
```

プロンプトは毎日目にするものなので、わかりやすい表記にしておくと気分が整います。

## Homebrew のインストール

Mac のセットアップで最初に入れておきたいのが Homebrew です。  
パッケージ管理が楽になり、その後の環境構築がかなりスムーズになります。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

インストール後は、必要に応じて `brew help` などで動作確認しておくと安心です。

## GitHub Copilot CLI のインストール

次に、GitHub Copilot CLI を入れます。  
コマンドラインから Copilot を使えるようになるので、作業の補助として便利です。

```bash
brew install copilot-cli
```

- 参考: [GitHub Copilot CLI のインストール](https://docs.github.com/ja/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli)

## Zed のインストール

エディタは Zed を使っています。  
起動が速く、普段の編集作業を軽快に進めやすいのが気に入っています。

```bash
brew install --cask zed
```

- 参考: [Installing Zed](https://zed.dev/docs/installation)

## まとめ

Mac の初期セットアップでは、まず Homebrew を入れておくと、その後の導入が楽になります。  
そのうえで、Copilot CLI と Zed を追加すると、開発環境の土台がかなり整います。

毎回同じ流れで入れておくものを固定しておくと、新しい Mac に移ったときも迷いません。
