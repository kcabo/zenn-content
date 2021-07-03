---
title: "Windows TerminalとUbuntu導入時にやった設定変更まとめ (フォント変更ほか)"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [windowsterminal, ubuntu, wsl, wsl2]
published: true
---

# Windows Terminalの設定

## キーバインドの変更
Chromeショートカットキーに寄せます。`ctrl+t`でタブを開き、`ctrl+w`で閉じられるようにします。
| action    | デフォルト   | 変更先 |
| --------- | ------------ | ------ |
| newTab    | ctrl+shift+t | ctrl+t |
| closePane | ctrl+shift+w | ctrl+w |

執筆時点ではキーバインドの変更はGUIからできません。設定ファイルを直接編集^[次のアップデートでGUIで変更できるようになりそうです: https://devblogs.microsoft.com/commandline/windows-terminal-preview-1-9-release/#settings-ui-updates]する必要があります。設定から「JSONファイルを開く」を押すと`settings.json`が開きますので、適当な位置に設定内容を追加しましょう。

```json:settings.json
{
    "actions": 
    [
        {
            "command": "newTab",
            "keys": "ctrl+t"
        },
        {
            "command": "closePane",
            "keys": "ctrl+w"
        },
    ]
}
```

## フォント設定をHackGen Consoleに
デフォルトのフォントは`Cascadia Code`というフォントなのですが、日本語の文字幅が変になってしまいます。
![](https://storage.googleapis.com/zenn-user-upload/a98cfc7f275d39395de39d2f.png =400x)

日本語用のフォントを入れて綺麗にしましょう。私は[HackGen](https://github.com/yuru7/HackGen/releases)の`HackGen Console`を使わせていただきました。プロファイル > Ubuntu-20.04 > 外観 > テキスト > フォントフェイス から選びましょう。フォントインストール後に再起動をいれないと多分選べません。
![](https://storage.googleapis.com/zenn-user-upload/c4b70fc741d179fd37bcd7bf.png =400x)
*美しい...!*

### 日本語フォントでカーソルがずれる症状
以前`Source Han Code JP`を使っていましたが、これですと**日本語の文字幅が正しく認識されず、カーソルがずれてしまいました。**

`Source Han Code JP`も等幅フォントなのですが、おそらく全角の幅が半角の1.5倍であることが影響しているようです。`HackGen Console`は半角1:全角2なのでずれません。(その代わり日本語が少し大きめに見えます)

Windows TerminalのIssueにも上がっていました。`HackGen35 Console`でもずれるようです。
https://github.com/microsoft/terminal/issues/9940

## 配色はOne Half Darkに
プロファイル > Ubuntu-20.04 > 外観 > テキスト > 配色 から変えられます。お好みで。

## 警告音の代わりにベルマークを表示させる
操作ミスの度に警告音がうるさいので、タブの部分に🔔が表示されるだけにします。
プロファイル > Ubuntu-20.04 > 詳細設定 > ベル通知スタイル を 「ビジュアル(フラッシュタスクバー)」に変更します。
![](https://storage.googleapis.com/zenn-user-upload/1f0ec5e43d0d753f2013d032.png)

## Ubuntuをデフォルトに
スタートアップ > 規定のプロファイル からUbuntuを選びます。これで新しいタブはデフォルトでUbuntuが開くようになりました。

## 開始ディレクトリをLinuxのホームディレクトリ`~`に変更
パフォーマンス上、UbuntuからWindowsのファイルを触るのは推奨されていません。
>特殊な理由がない限り、複数のオペレーティング システム間でファイルを操作しないことをお勧めします。 Linux コマンド ライン (Ubuntu、OpenSUSE など) で作業している場合、最速のパフォーマンス速度を実現するには、ファイルを WSL ファイル システムに格納します。 
> [Microsoft公式 - OS ファイル システム間でのパフォーマンス](https://docs.microsoft.com/ja-jp/windows/wsl/compare-versions#performance-across-os-file-systems) 

ですがなぜかWindows TerminalでUbuntuを開くと、WindowsのCドライブ内のユーザディレクトリが初期位置になります。Ubuntuからは基本Ubuntu内しか操作しないと思うので、初期位置をホームディレクトリ`~`に変更します。

[公式の手順](https://docs.microsoft.com/ja-jp/windows/terminal/troubleshooting#set-your-wsl-distribution-to-start-in-the-home--directory-when-launched)ではJSONファイルを直接編集していますが、**現在はGUIで設定変更できます**。(プロファイル > Ubuntu-20.04 > 全般 >ディレクトリの開始)
この値を`//wsl$/Ubuntu-20.04/home/<Your Ubuntu Username>`に置き換えます。
ちなみにセパレータはスラッシュ`/`でもバックスラッシュ`\`どちらでも大丈夫でした。


# Ubuntuの設定

## 日本語化
``` bash
sudo apt install language-pack-ja
sudo update-locale LANG=ja_JP.UTF8
```

## プロンプトをカスタマイズ
プロンプトを画像のようにカスタマイズしました。
![](https://storage.googleapis.com/zenn-user-upload/e8c06c771df5414348bea8ca.png)

ホームディレクトリにある`.bashrc`の一部を次のように書き換えることで設定できます。
```diff bash:.bashrc
    if [ "$color_prompt" = yes ]; then
-       PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
+       PS1='\e[32m\u@\h \e[35m\t \e[34m\w \e[0m\$ '
    else
        PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
    fi
```

なぜこうなるかなど、詳しくはこちらの記事を御覧ください。
https://zenn.dev/kcabo/articles/555d9cc6dad0c3


