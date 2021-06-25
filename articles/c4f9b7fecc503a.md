---
title: "WSL2+Docker+VSCodeの環境でPythonWebアプリ開発をシミュレート"
emoji: "🧰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [wsl, wsl2, docker, vscode, fastapi]
published: false
---

# 概要
Windowsには割と簡単にLinux環境を構築できます。コンテナと組み合わせることで本番と同じ環境で開発ができるようになります。
:::message alert
細かい導入の手順は公式ドキュメントや他の方の記事を参照してください。
ざっくりとした全体の流れのメモになります
:::

# 環境
- Windows10 Home 21H1

# WSL2の導入

[Microsoftの公式ドキュメント(日本語)](https://docs.microsoft.com/ja-jp/windows/wsl/install-win10)が分かりやすいです。

## 0x80370102エラーへの対処
私の環境では途中で[エラーが発生](https://docs.microsoft.com/ja-jp/windows/wsl/install-win10#:~:text=%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%8C%E3%82%A8%E3%83%A9%E3%83%BC%200x80070003%20%E3%81%BE%E3%81%9F%E3%81%AF%E3%82%A8%E3%83%A9%E3%83%BC%200x80370102%20%E3%81%A7%E5%A4%B1%E6%95%97%E3%81%97%E3%81%9F)しました。どうやらCPU側で仮想化がサポートされていない場合、BIOS設定をいじる必要があるみたいです。

## どのLinuxディストリビューションを選択するか

## bashの調整
- 開始ディレクトリをLinuxのホームディレクトリ`~`に変更します。
  - Windows TerminalでUbuntuを開くと、WindowsのCドライブ内のユーザディレクトリが初期位置になります。
  - 詳しくは後述しますが、マウントされたWindowsのディレクトリをいじくり回すのは推奨されていません。
  - [公式の手順](https://docs.microsoft.com/ja-jp/windows/terminal/troubleshooting#set-your-wsl-distribution-to-start-in-the-home--directory-when-launched)にならい、設定を変更しておくといいです。
  - 現在はGUIで設定変更できるようです。(プロファイル > Ubuntu-20.04 > 全般 >ディレクトリの開始)
    - この値を`//wsl$/Ubuntu-20.04/home/<Your Ubuntu Username>`に置き換えます。
    - ちなみにセパレータはスラッシュ`/`でもバックスラッシュ`\`どちらでも大丈夫でした。
- パッケージのアップデートもしておきました。
    ```bash
    $ sudo apt update
    ```

## ソースコードはWSL側に格納する
>特殊な理由がない限り、複数のオペレーティング システム間でファイルを操作しないことをお勧めします。 Linux コマンド ライン (Ubuntu、OpenSUSE など) で作業している場合、最速のパフォーマンス速度を実現するには、ファイルを WSL ファイル システムに格納します。 Windows コマンド ライン (PowerShell、コマンド プロンプト) で作業している場合、ファイルを Windows ファイル システムに格納してください。
> [Microsoft公式 - OS ファイル システム間でのパフォーマンス](https://docs.microsoft.com/ja-jp/windows/wsl/compare-versions#performance-across-os-file-systems) 


# VSCodeからWSL2に接続してソースファイルを編集する
WSLへリモート接続するためのVSCode拡張機能をインストールします。2つのうちどちらかを選びます。
- [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)
- [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack): Remote - WSLを含んだ拡張機能のパッケージです。このあと使う[Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)も含まれているので、私はこちらをインストールしました。

