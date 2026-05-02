---
title: "WinGetを使ってアプリケーションをインストールするには"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [windows,winget]
published: false
---

# WinGet (Windows Package Manager)

https://learn.microsoft.com/ja-jp/windows/package-manager/winget/

## WinGet をインストールするには

管理者権限でコマンドプロンプトまたはPowerShellを開きます。

WinGet をインストールする
```bash
winget install --global --scope user winget-cli
```

## WinGet でアップグレードするには

upgrade コマンドでアップグレード対象のパッケージを表示する

```bash
> winget upgrade
名前                                               ID                                バージョン    利用可能      ソース
-----------------------------------------------------------------------------------------------------------------------
Microsoft Visual C++ 2015-2019 Redistributable (x… Microsoft.VCRedist.2015+.x64      14.29.30133.0 14.40.33816.0 winget
Microsoft Windows Desktop Runtime - 8.0.8 (x64)    Microsoft.DotNet.DesktopRuntime.8 8.0.8         8.0.10        winget
Microsoft Visual C++ 2015 Redistributable (x86) -… Microsoft.VCRedist.2015+.x86      14.0.24212.0  14.40.33816.0 winget
Rye: An Experimental Package Management Solution … Rye.Rye                           0.34.0        0.41.0        winget
Dev Home                                           Microsoft.DevHome                 0.1800.640.0  0.1801.640.0  winget
5 アップグレードを利用できます。
```

対象パッケージをアップグレードする

```bash
> winget upgrade Rye.Rye --version 0.41.0
見つかりました Rye [Rye.Rye] バージョン 0.41.0
このアプリケーションは所有者からライセンス供与されます。
Microsoft はサードパーティのパッケージに対して責任を負わず、ライセンスも付与しません。
このパッケージには次の依存関係が必要です:
  - パッケージ
      Microsoft.VCRedist.2015+.x64
ダウンロード中 https://github.com/astral-sh/rye/releases/download/0.41.0/rye-x86_64-windows.exe
  ██████████████████████████████  13.4 MB / 13.4 MB
インストーラーハッシュが正常に検証されました
パッケージのインストールを開始しています...
インストールが完了しました
```

# Git for Windows

## Git for Windows をインストールするには

```bash
> winget search Git.Git
名前 ID      バージョン ソース
-------------------------------
Git  Git.Git 2.47.0.2   winget

> winget install Git.Git --version 2.47.0.2 
```

## Git for Windows をアップデートするには

```bash
git update-git-for-windows
```

# Windows Terminal

## Windows Terminal をインストールするには

```bash
> winget search Microsoft.WindowsTerminal
名前                     ID                                バージョン  ソース
------------------------------------------------------------------------------
Windows Terminal         Microsoft.WindowsTerminal         1.21.2911.0 winget
Windows Terminal Preview Microsoft.WindowsTerminal.Preview 1.22.2912.0 winget

> winget install Microsoft.WindowsTerminal --version 1.21.2911.0
```

# WSL

## WSL をインストールするには

```bash
> winget search Microsoft.WSL
名前                        ID            バージョン ソース
------------------------------------------------------------
Windows Subsystem for Linux Microsoft.WSL 2.1.5.0    winget 

> winget install Microsoft.WSL --version 2.1.5.0
```
