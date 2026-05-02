---
title: "WinGetでWindowsのアプリをインストールする"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [windows, winget]
published: true
---

# WinGetでWindows環境を整える

Windowsで開発環境や作業環境を整えるとき、毎回ブラウザで配布ページを開いて、インストーラーを1つずつダウンロードしていくのは少し面倒です。

そんなときに便利なのが、Windows向けのパッケージマネージャーである `WinGet` です。

この記事では、自分用のメモも兼ねて、WinGetの基本的な使い方と、Git for Windows / Windows Terminal / WSL をインストールする例をまとめます。

> 2026年5月時点の内容です。  
> 確認に使った `winget` のバージョンは `v1.28.240` です。

公式ドキュメント:

https://learn.microsoft.com/ja-jp/windows/package-manager/winget/

## WinGetとは

`WinGet` は Microsoft が提供する Windows Package Manager です。

コマンドラインからアプリを検索したり、インストールしたり、アップグレードしたりできます。

開発用PCの初期セットアップをするときや、あとで同じ手順をもう一度やりたいときに便利です。

## まずはWinGetが使えるか確認する

最近のWindowsでは、WinGetは `App Installer` とあわせて最初から使えることが多いです。

まずは PowerShell またはコマンドプロンプトで、次のコマンドを実行します。

```bash
winget --version
```

バージョンが表示されれば、そのまま使えます。

手元の環境では、次のように表示されました。

```bash
winget --version
v1.28.240
```

もし `winget` コマンドが見つからない場合は、Microsoft Store から `App Installer` を更新またはインストールすると使えることがあります。

## WinGetの基本操作

WinGetは、まず `search` で探して、次に `install` で入れて、必要に応じて `upgrade` で更新する流れが基本です。

### アプリを検索する

たとえば Git を探すなら、こんな感じです。

```bash
winget search Git
```

候補が複数出ることがあるので、実際にインストールするときは `ID` を使うのが分かりやすいです。

### アプリをインストールする

`ID` を指定してインストールします。

```bash
winget install --id Git.Git
```

バージョンを固定したいときは `--version` を付けます。

```bash
winget install --id Git.Git --version 2.47.0.2
```

### インストール済みアプリをアップグレードする

まず、更新できるアプリ一覧を確認します。

```bash
winget upgrade
```

特定のアプリだけ更新したい場合は、`ID` を指定します。

```bash
winget upgrade --id Git.Git
```

まとめて更新したい場合は次のコマンドです。

```bash
winget upgrade --all
```

## インストール例

ここでは、Windows環境のセットアップでよく使いそうなツールをWinGetでインストールする例を紹介します。

### Git for Windows をインストールする

まずは検索します。

```bash
winget search Git.Git
```

インストールは次のコマンドです。

```bash
winget install --id Git.Git
```

### Windows Terminal をインストールする

こちらもまずは検索します。

```bash
winget search Microsoft.WindowsTerminal
```

インストールは次のコマンドです。

```bash
winget install --id Microsoft.WindowsTerminal
```

### WSL をインストールする

WSL も WinGet からインストールできます。

```bash
winget search Microsoft.WSL
```

```bash
winget install --id Microsoft.WSL
```

インストール後は、環境によって再起動が必要になることがあります。

## よくある注意点

### `search` の結果が複数出る

アプリ名だけで検索すると、似たパッケージがいくつか表示されることがあります。

なので、実際のインストールでは `--id` を付けて対象を明示するのがおすすめです。

### 管理者権限が必要な場合がある

アプリや設定内容によっては、管理者権限の PowerShell / コマンドプロンプトが必要になることがあります。

インストールに失敗したときは、まず権限まわりを確認すると切り分けしやすいです。

### ライセンス確認が表示されることがある

パッケージによっては、インストール時にライセンスや利用規約に関する表示が出ます。

実行ログをざっと見ながら進めると安心です。

## まとめ

WinGetを使うと、Windowsでのアプリ導入やアップグレードをコマンドでまとめて扱えます。

特に、開発環境の初期セットアップをするときや、手順をあとで見返したいときにはかなり便利です。

まずは `winget --version` で使えるかを確認して、`search`、`install`、`upgrade` の3つを試してみるのがよさそうです。

自分としては、Windowsの初期設定をするときに見返すためのメモとしても使っていくつもりです。
