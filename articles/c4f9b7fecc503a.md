---
title: "WSL2+Docker+VSCodeの開発環境構築とPythonでWebアプリを試すまで"
emoji: "🧰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [wsl, wsl2, docker, vscode, fastapi]
published: false
---

## 概要
Windowsには割と簡単にLinux環境を構築できます。コンテナと組み合わせることで本番と同じ環境で開発ができるようになります。
また、この環境をセットアップすることで次の3つの独立した開発/実行環境を状況に応じて使い分けられます。
1. Windows環境
2. WSL環境
3. コンテナ内環境

:::message alert
細かい導入の手順は公式ドキュメントや他の方の記事を参照してください。
ざっくりとした全体の流れのメモになります
:::

## 環境
- Windows10 Home 21H1

# WSL2の導入

[Microsoftの公式ドキュメント(日本語)](https://docs.microsoft.com/ja-jp/windows/wsl/install-win10)が分かりやすいです。

<!-- ## 0x80370102エラーへの対処 -->
<!-- 私の環境では途中で[エラーが発生](https://docs.microsoft.com/ja-jp/windows/wsl/install-win10#:~:text=%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%8C%E3%82%A8%E3%83%A9%E3%83%BC%200x80070003%20%E3%81%BE%E3%81%9F%E3%81%AF%E3%82%A8%E3%83%A9%E3%83%BC%200x80370102%20%E3%81%A7%E5%A4%B1%E6%95%97%E3%81%97%E3%81%9F)しました。どうやらCPU側で仮想化がサポートされていない場合、BIOS設定をいじる必要があるみたいです。 -->

### どのLinuxディストリビューションを選択するか

### 開始ディレクトリをLinuxのホームディレクトリ`~`に変更
- Windows TerminalでUbuntuを開くと、WindowsのCドライブ内のユーザディレクトリが初期位置になります。
- 詳しくは後述しますが、マウントされたWindowsのディレクトリをいじくり回すのは推奨されていません。[公式の手順](https://docs.microsoft.com/ja-jp/windows/terminal/troubleshooting#set-your-wsl-distribution-to-start-in-the-home--directory-when-launched)にならい、設定を変更おきます。
- 現在はGUIで設定変更できるようです。(プロファイル > Ubuntu-20.04 > 全般 >ディレクトリの開始)
  - この値を`//wsl$/Ubuntu-20.04/home/<Your Ubuntu Username>`に置き換えます。
  - ちなみにセパレータはスラッシュ`/`でもバックスラッシュ`\`どちらでも大丈夫でした。

### パッケージのアップデート
```bash
$ sudo apt update
```

## ソースコードはWSL側に格納する
>特殊な理由がない限り、複数のオペレーティング システム間でファイルを操作しないことをお勧めします。 Linux コマンド ライン (Ubuntu、OpenSUSE など) で作業している場合、最速のパフォーマンス速度を実現するには、ファイルを WSL ファイル システムに格納します。 Windows コマンド ライン (PowerShell、コマンド プロンプト) で作業している場合、ファイルを Windows ファイル システムに格納してください。
> [Microsoft公式 - OS ファイル システム間でのパフォーマンス](https://docs.microsoft.com/ja-jp/windows/wsl/compare-versions#performance-across-os-file-systems) 


# VSCodeからWSL2に接続してソースファイルを編集する(まずはDockerなしで)
WSLへリモート接続するためのVSCode拡張機能をインストールします。2つのうちどちらかを選びます。
- [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)
- [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack): Remote - WSLを含んだ拡張機能のパッケージです。このあと使う[Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)も含まれているので、私はこちらをインストールしました。

### リモートに接続するとはどういうことか
[公式](https://code.visualstudio.com/docs/remote/remote-overview)の説明画像を拝借しました。
![](https://storage.googleapis.com/zenn-user-upload/a5bccfe849a6bbd8d1ead92d.png)
*WSL2に初めて接続した際にVSCode ServerがWSL2にインストールされます*

#### Remote - WSLの特徴
- ローカル(Windows)とは完全に隔離された環境でありながら、ローカルとほとんど変わらない操作感で開発ができる
- Linux環境で開発するため、改行コードもLFになる
- VSCodeの組み込みターミナルでbashが使える
- VSCodeの拡張機能もローカルと独立(一部見た目を変えるテーマなどは共有)
  - たとえばリモート接続中にVSCodeのPython拡張機能を追加すると、WSLのホームディレクトリの`.vscode-server`内にインストールされます。

#### Remote - WSLなしじゃだめなの？
WSLのLinuxファイルシステム上にあるファイルは例えば以下のようなパスでWindowsからアクセスできます。
- `\\wsl$\Ubuntu-20.04\home\kcabo\my-app\hello.py`

このファイルをRemote - WSLなしでWindows上のファイルとして開くことも可能です。しかしその場合pythonインタプリタはWindowsにインストールされている中から選ぶことになります。Linux上のPythonランタイムは使えません。

既存のWindows環境はあくまでソースコードを書くためのインターフェースとし、実際にコードが走るWSLとは隔離させる、これがRemote - WSLの役割と私は解釈してます。

#### ローカルとリモートの違い


# Dockerの導入

### Remote - Containerでコンテナに接続してソースファイルを編集する


# FastAPIでAPIサーバーを立ててブラウザで叩いてみる