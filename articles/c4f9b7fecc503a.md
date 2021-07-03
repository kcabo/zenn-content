---
title: "WSL2+Docker+VSCodeの開発環境構築とPythonでWebアプリを試すまで"
emoji: "🧰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vscode, wsl, wsl2, docker, fastapi]
published: true
---

:::message
【追記】記事の構成と意図が分かりにくかったので少し手を加えました🙇🏻
:::

Windowsユーザーの皆さん、手軽にLinux環境で開発したいですよね！そんなときWSL2やコンテナが選択肢に上がるでしょう。VSCodeのRemote Development機能を使い、どちらも試してみました。Windows・WSL・コンテナを使い分けつつ、VSCodeで快適に開発ができるようになります！


## おおまかな流れ
1. 【準備編】WSL2の有効化と拡張機能のインストール
2. 【WSL編】VSCodeからWSL2にリモート接続する
3. 【コンテナ編】Dockerのインストール
4. 【コンテナ編】VSCodeからDockerコンテナにリモート接続する
5. 【コンテナ編】コンテナでWebサーバーを立て、ブラウザからアクセスする

※ 筆者の環境は **Windows10 Home 21H1** です。Windows10 Pro/Enterpriseとは手順が異なるかもしれません。

:::message alert
細かい手順は基本的に公式ドキュメントに頼ってます。Dockerにおける細かい用語の説明などもしません。これはざっくりとした全体の流れや概念の紹介です。
:::

# WSL2の有効化

[Microsoftの公式ドキュメント(日本語)](https://docs.microsoft.com/ja-jp/windows/wsl/install-win10)が分かりやすいです。これに従いましょう。

:::details どのLinuxディストリビューションを選択するか
Ubuntuが最もポピュラーな選択肢でしょう。Microsoft Storeには複数のUbuntuディストリビューションがありますね。
- Ubuntu
- Ubuntu 20.04 LTS
- Ubuntu 18.04 LTS

無印のUbuntuでは最新のバージョンがインストールされます。執筆時点で最新が20.04なので、上2つの中身は同じです。私は2つ目をインストールしました。
参考: [A Guide to Upgrading your Ubuntu App’s Release | Windows Command Line](https://devblogs.microsoft.com/commandline/upgrading-ubuntu/)
:::

:::details 0x80370102エラーが発生したら
私の環境では途中で[エラーが発生](https://docs.microsoft.com/ja-jp/windows/wsl/install-win10#:~:text=%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%8C%E3%82%A8%E3%83%A9%E3%83%BC%200x80070003%20%E3%81%BE%E3%81%9F%E3%81%AF%E3%82%A8%E3%83%A9%E3%83%BC%200x80370102%20%E3%81%A7%E5%A4%B1%E6%95%97%E3%81%97%E3%81%9F)しました。どうやらCPU側で仮想化がサポートされていない場合、BIOS設定をいじる必要があるみたいです。
:::

### パッケージの更新
下記コマンドでUbuntu内のパッケージを最新版に更新できます。導入時は念の為やっておくと安心です。
```bash
$ sudo apt update
$ sudo apt upgrade
```

### ソースコードはWSL側に格納する
パフォーマンス上、プロジェクトフォルダはWSL内に作ったほうがいいようです。
>特殊な理由がない限り、複数のオペレーティング システム間でファイルを操作しないことをお勧めします。 Linux コマンド ライン (Ubuntu、OpenSUSE など) で作業している場合、最速のパフォーマンス速度を実現するには、ファイルを WSL ファイル システムに格納します。 
> [Microsoft公式 - OS ファイル システム間でのパフォーマンス](https://docs.microsoft.com/ja-jp/windows/wsl/compare-versions#performance-across-os-file-systems) 

したがって、Windows Terminalを使用する場合は次の設定をしておくとWSLファイルシステムにアクセスしやすいです。

### 開始ディレクトリをLinuxのホームディレクトリ`~`に変更
Windows TerminalでUbuntuを開くと、WindowsのCドライブのユーザディレクトリが初期位置になります。
Windows Terminalの 設定 > プロファイル > Ubuntu-20.04 > 全般 >ディレクトリの開始 を `%USERPROFILE`から`//wsl$/Ubuntu-20.04/home/<Your Ubuntu Username>`に変更すれば、起動時の初期位置がLinuxのホームディレクトリになります。

その他の設定はこちらの記事にまとめました。
https://zenn.dev/kcabo/articles/9e9ebe1dc8fa75


# VSCode拡張機能のインストール

- [Remote-WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)
  - VSCodeがWSLにリモート接続するために必要な拡張機能です。**コンテナさえ使えればいいよって方は不要**です。
- [Remote-Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
  - VSCodeがコンテナにリモート接続するために必要な拡張機能です。[Docker拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)とは別物です。
- [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)
  - Remote-WSLとRemote-Containersに加え、Remote-SSHがセットになった拡張機能のパッケージです。

私はRemote Developmentをインストールしましたが、次章を飛ばす方はRemote-Containersのみで十分でしょう。


# Remote-WSLでVSCodeからWSL2に接続する(Dockerなしで)

:::message
Dockerを入れずとも手軽にLinuxが使えることを確認する章です。WSL内はクリーンに保ちたい！コンテナしか使わない！って方は飛ばしてください。
:::

### WSL上のPythonでHelloWorldしてみる
それではRemote-WSLを使い、Pythonを実行してみます。Ubuntuを開き、以下を実行します。
```bash
$ pwd # Linuxホームディレクトリにいることを確認
/home/kcabo
$ mkdir my-app # プロジェクトフォルダを作成
$ cd my-app/
$ code . # カレントディレクトリをVSCodeで開く
```
VSCodeが起動するはずです。(起動しない場合はPATHを通してください)
画面左下の接続ボタンに「WSL:～」と表示されていればリモート接続できています。
![](https://storage.googleapis.com/zenn-user-upload/af04f3713d6358465f0f460e.png)

そのままVSCode上で下記ファイルを作成しましょう。
```python:main.py
print('Hello World!')
```
VSCodeの組み込みターミナルを起動すると、PowerShellではなくbash(Ubuntuのデフォルトシェル)が使えることが分かります。Pythonを実行してみましょう。
```bash
$ python3 main.py 
Hello World!

$ which python3 # pythonインタプリタの確認
/usr/bin/python3 # Ubuntuにインストールされたランタイムが使用されている
```


## リモートでWSLに接続するってどういうこと？
ローカルマシンにあるフォルダなのにリモート接続ってどういうことでしょうか。それを説明するのに[公式](https://code.visualstudio.com/docs/remote/remote-overview)の図が分かりやすいです。
![](https://storage.googleapis.com/zenn-user-upload/a5bccfe849a6bbd8d1ead92d.png)

ローカル(=Windows)からWSL上のVSCode Serverにアクセスします。ソースコードの編集はこのサーバーを通じてよしなにやってくれるわけです。VSCode Serverは初接続時にWSL2へインストールされます。

この図を見てわかるようにVSCodeの拡張機能もローカルとは切り離されています。(一部見た目を変えるテーマなどは共有されます)
たとえばリモート接続中にVSCodeのPython拡張機能を追加すると、WSLのホームディレクトリの`.vscode-server`内にインストールされます。


### Remote-WSLでWindowsとは別のランタイムが使える
WSLのLinuxファイルシステムには`\\wsl$\Ubuntu-20.04\home\kcabo\my-app\`といったパスでWindowsからアクセスできます。

つまりWindows上のフォルダとして開くことも可能です。(VSCodeの`Reopen Folder Locally`でできます) しかしその場合pythonインタプリタはWindowsにインストールされている中から選ぶことになります。Linux上のPythonランタイムは使えません。

**既存のWindows環境はあくまでコードを書くためのインターフェースとし、実際にコードが走るWSLとは隔離させる**、これがRemote-WSLの役割と私は解釈してます。

:::details ローカル(Windows)とリモート(WSL)との細かな違い
- ローカルで新規作成したファイルは改行が`CRLF`だが、リモートだと`LF`になる
- エクスプローラーの表示の違い
  ![](https://storage.googleapis.com/zenn-user-upload/7cdff7cc19ab3d5c73792558.png =500x)
- 画像ファイルなどの右クリックメニューが微妙に違う
  ![](https://storage.googleapis.com/zenn-user-upload/ee3d7a0954a9804e8f1e3cac.png =500x)
  - リモート側だけにある[ダウンロード]はローカルのWindowsにファイルを保存します(ファイル保存画面が開きます)
  - ローカルではごみ箱に移動するので[削除]、リモートではごみ箱がないので直接削除=[完全に削除]なのでしょう
:::

# Dockerの導入

公式サイトからDockerをインストールします。
https://hub.docker.com/editions/community/docker-ce-desktop-windows/
インストールダイアログでは`Install required Windows components for WSL 2`に必ずチェックを入れておきましょう。
![インストール時の設定](https://storage.googleapis.com/zenn-user-upload/c2140e90690d403cdc0a96d4.png =400x)

インストール後にDockerDesktopの`Settings`>`General`を確認します。`Use the WSL 2 based engine (Windows Home can only run the WSL 2 backend)`にデフォルトでチェックが入っているはずです。

![](https://storage.googleapis.com/zenn-user-upload/789ac85b337fab0403a711b7.png =600x)

:::message
どこかのタイミングでVSCodeの[Docker拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)を**WSL側に**インストールしておくといいです。
Dockerfileの編集が楽になります。
:::


# Remote-ContainersでVSCodeからコンテナに接続する

### 開発用コンテナを立てる
先ほどの`my-app`をコンテナ化します。`main.py`と同じ階層に`Dockerfile`を作成します。ベースイメージはPythonが動けば何でもいいです。
```Dockerfile:Dockerfile
FROM python:3.9-slim-buster
```
:::message
テンプレートからDockerfileを[自動生成する方法](https://github.com/Microsoft/vscode-remote-try-python)もあります。ですが理解のために今回は手動で作成します。
:::

次に左下の接続ボタンをクリックすると以下の一覧画面が出てきます。下から三番目の`Reopen in Container`を選びましょう。
![](https://storage.googleapis.com/zenn-user-upload/4d32b3e1342ccd47aac784ad.png)

先ほど作成したDockerfileからイメージを作るので、`From 'Dockerfile'`を選択します。
![](https://storage.googleapis.com/zenn-user-upload/9d712044b4a054d4586ab228.png)

すると勝手にイメージがビルドされ、コンテナが起動します。左下の接続ボタンが以下のように変化し、`.devcontainer`フォルダが新しく作られたことに気づくでしょう。
![](https://storage.googleapis.com/zenn-user-upload/4c005e3c6d198644e6cdde6c.png =300x)

### ここはどこ？
今VSCodeで接続しているのはコンテナ内です。VSCodeのターミナルで確認してみます。
```bash
$ pwd
/workspaces/my-app
```

コンテナ内のOSも確認してみます。
```bash
$ cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 10 (buster)"
NAME="Debian GNU/Linux"
VERSION_ID="10"
VERSION="10 (buster)"
VERSION_CODENAME=buster
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```
WSLのUbuntuではなくDebianになっていますね。そうですここはコンテナの中です。不思議な気分です。

### コンテナを消しても変更は維持される
それではVSCode上で先ほどのpythonファイルを編集してみます。
```python:main.py
print('Hello from Container!')
```

実行してみましょう。
```bash
$ python3 main.py 
Hello from Container!
```

問題ないですね。それではここでリモート接続を切り、コンテナを停止させてみます。左下の接続ボタンを押し、`Reopen Folder in WSL`を選びます。
![](https://storage.googleapis.com/zenn-user-upload/c9c697c167f09fd4f33568ef.png)

するとWSLにリモート接続した状態でVSCodeが立ち上がります。(コンテナを抜けるとコンテナも自動的に停止するようになっています)
コンテナ内で編集した`main.py`を確認してみてください。`print('Hello from Container!')`のままになっています！不思議ですね。実はこれ、WSL上のこのフォルダが先ほどのコンテナ内にマウントされていたからなんです。次項で詳述します。


## リモートでコンテナに接続するってどういうこと？
こちらも[公式](https://code.visualstudio.com/docs/remote/containers)の説明画像を拝借しました。
![](https://storage.googleapis.com/zenn-user-upload/0cdbc42498de77bf498bbd3c.png)

WSLへのリモート接続と異なるのは、ローカルのプログラムファイルがコンテナ内にマウントされているということです。

コンテナ内でファイルを編集すると、ローカルのファイルにも変更が反映されます。(それよりも、**マウントされたローカルのファイルをコンテナ内から編集している**、という方が正しいですね)したがってコンテナやそのイメージを破棄しても、編集内容は消えません。

また、WSLと同じく拡張機能も切り離されているので、Linterなどの拡張機能はコンテナ内に別途インストールする必要があります。


# FastAPIでAPIサーバーを立ててブラウザからアクセスする
それでは最後にコンテナでサーバーを立ててWindowsのブラウザから接続してみます。
とりあえず動かすことを前提にちゃっちゃっと作ります。先ほどまでのファイルを以下のように書き換えます。

```python:main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return 'Hello World'
```

```Dockerfile:Dockerfile
FROM python:3.9-slim-buster

RUN pip install fastapi uvicorn[standard]
```

コンテナの設定ファイルを書き換えたので、接続ボタンから`Rebuild Container`を選択します。すると新しく定義されたDockerfileをもとに、コンテナが作り直されます。

**VSCodeのターミナルで**次のコマンドを叩けばサーバーが立ち上がります。
```bash
$ uvicorn main:app --host 0.0.0.0 --port 8000
```

最低限これだけで動きます。ブラウザで http://localhost:8000/ にアクセスしてください。"Hello World"と表示されましたか？



### ポートフォワーディングはVSCodeが勝手にやってくれる
ろくに設定ファイルもいじらずに簡単に接続できました。コンテナの8000番ポートが自動的にローカルの8000番ポートに割り当てられています。
![](https://storage.googleapis.com/zenn-user-upload/bb6f2d2ff2b7d020cc51a8c4.png)
8000番ポートでサーバーが立てられると、VSCodeがそれを認識して適当なローカルアドレスに転送してくれます。便利ですね。


### 実際の開発ではdevcontainer.jsonの設定が必要
簡略化のため、`devcontainer.json`は触らずに実行しました。実際は`devcontainer.json`内に拡張機能などの設定を記述していく必要があります。
https://code.visualstudio.com/docs/remote/devcontainerjson-reference


### (おまけ) VSCodeなしでサーバーを立てる場合
Dockerfileを以下のように書き足します。

```Dockerfile:Dockerfile
FROM python:3.9-slim-buster

RUN pip install fastapi uvicorn[standard]

WORKDIR /app

COPY . .

CMD uvicorn main:app --host 0.0.0.0 --port 8000
```

`my-app`ディレクトリに行き、次を叩けばVSCodeなしでサーバーが立ちます。

```bash
$ docker build -t my-app .
$ docker run -it -p 8000:8000 my-app
```

# まとめ
VSCodeのRemote Developmentを活用すれば環境構築の幅が広がります。具体的には次の3つの独立した開発/実行環境を状況に応じて使い分けられます。
1. Windows環境
2. WSL環境
3. コンテナ内環境

おかげさまでWindowsではAnaconda(python3.7)、WSL(Ubuntu)ではPython3.8、コンテナ(Debian)ではPython3.9が走るという~~気持ち悪い~~カオスな状況になりました。

コンテナさえあればいいじゃん。3つもいらなくね？って感じもしますね笑 ただコンテナが立ち上がるまでの時間が少し気になるので、依存関係の少ない小規模なプロジェクトの場合はWSLにリモート接続して開発しようかと思います。

**Windows環境は極力クリーンに保ちつつ、Node.jsやPythonといったよく使うランタイムはWSLに入れる。中規模以上の開発時など、そのWSLの環境とは隔離して開発したいときには別途コンテナを立てて作業する**、といった感じでいきたいと思います。

[Facebookもこの開発手法を気に入っている](https://developers.facebook.com/blog/post/2019/11/19/facebook-microsoft-partnering-remote-development/?locale=ja_JP)みたいです。今後の進展が楽しみですね。