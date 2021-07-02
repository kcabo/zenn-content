---
title: "Windows Terminalの外観を調整した (フォント・PS)"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# 日本語化は今回は保留
``` bash
sudo apt install language-pack-ja
sudo update-locale LANG=ja_JP.UTF8
```

# Prompt Stringの変更
  - 入力するところの前にカレントディレクトリとかを表示するやつ
  - デフォルトだと少し冗長な気がする
(参考) https://qiita.com/zaburo/items/9194cd9eb841dea897a0


# 起動時のディレクトリの変更
  - Windows TerminalでUbuntuを開くとWindowsのCドライブ内のユーザディレクトリが初期位置になる
  - 後述するけどマウントされたディレクトリをいじくり回すのは推奨されていない
  - Linuxのホームディレクトリ`~`に変更する
  - https://docs.microsoft.com/ja-jp/windows/terminal/troubleshooting#set-your-wsl-distribution-to-start-in-the-home--directory-when-launched
  - 私のWTのバージョン(バージョン: 1.8.1521.0)ではGUIで変更できた
    - プロファイル > Ubuntu-20.04 > 全般 >ディレクトリの開始
    - この値を`//wsl$/Ubuntu-20.04/home/<Your Ubuntu Username>`にすればよい
    - スラッシュ`/`でもバックスラッシュ`\`でもいいみたい？
# カラーテーマとかの外観設定 

# 日本語フォント設定？
  - デフォルトだと日本語の文字幅が変
  - Source Han Code JPを使用する
  - https://github.com/adobe-fonts/source-han-code-jp/releases
  - こちらにある`Fonts version 2.012R (OTF, OTC)`を使用
![デフォルト](https://storage.googleapis.com/zenn-user-upload/b394369f4cf8d83d8b2fa87c.png)
![Source Han Code JP](https://storage.googleapis.com/zenn-user-upload/4971adcc06693db0e1311d3d.png)
  - 見ていて幸せな気持ちになります
