# **Github を準備し，コードをアップするまでの手順**

作成日：2019/01/30

更新日：2020/11/02

## **0.目次**

- 最初にやること
- プロダクトを登録するときにやること
- 登録済みのプロダクトを更新したときにやること

## **1. 最初にやること**

- はじめて Github に登録をしたとき．
- 新しい PC で Github にアクセスするとき．

ローカル PC と Github の通信には「ssh 通信」を使用する．
この通信では以下の流れである．

1. 「公開鍵」「秘密鍵」の 2 種類の鍵ファイルを作成する．この 2 つはペアになっている．
2. 「公開鍵」の内容を Github 上に登録する．「秘密鍵」のファイルはローカル PC に保存する．
3. ローカル PC に通信用の設定を記述する．
4. 両者で通信を行う際，「公開鍵」と「秘密鍵」の組み合わせが合っている場合のみ通信が成功する．

### **1.1 作業ディレクトリの確認**

ターミナルを起動するとホームディレクトリにいるはずだが一応下記を実行．

```bash
$ cd ~
```

### **1.2 フォルダの準備**

- ssh-key は適切な場所に配置しないと動かない．
- 以下の手順でフォルダを準備する．

下記コマンドでファイルとフォルダの一覧を表示する．

```bash
$ ls -a
```

一覧が表示されるので，「.ssh」フォルダを探す．

**`「.ssh」`が存在しない場合のみ**下記コマンドでホームディレクトリに`.ssh`フォルダを作成する．

```bash
$ mkdir -p ~/.ssh
```

エラーが出なければ OK．

### **1.3 ssh-key の発行**

- Github にアクセスするには ssh-key が必要となる．
- ssh-key は公開鍵と秘密鍵のペアになっており，「公開鍵を Github に登録」「秘密鍵を PC のローカルに保存」することで通信時に組み合わせがあっているかどうか判断する．
- ターミナル(windows は Git bash)を開いて以下のコマンドを入力する．

まずは以下のコマンドで作業ディレクトリを`.ssh`（前項で準備したフォルダ）に変更する．

```bash
$ cd ~/.ssh
```

続いて，以下のコマンドで ssh キー（公開鍵と秘密鍵のペア）を発行する．

```bash
$ ssh-keygen
```

実行結果

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):
```

上の後で`github_rsa`を入力して Enter．

```bash
Enter passphrase (empty for no passphrase):
```

続き．とりあえず入力せずに Enter．

```bash
Enter same passphrase again:
```

続き．パスワード入力していないので再度そのまま Enter．

```bash
Your identification has been saved in github_rsa.
Your public key has been saved in github_rsa.pub.
The key fingerprint is:
6f:09:00:22:44:55:66:77:95:89:41:7d:a7:58:1b:92 vagrant@localhost.localdomain
The key's randomart image is:
+--[ RSA 2048]----+
|   .o. .         |
|     oEo+ .      |
|    . +=+=       |
|     o.+==       |
|    . . S .      |
|       - + .     |
|      .   +      |
|         .       |
|                 |
+-----------------+
```

これで ssh-key を発行できた．発行した内容を確認する．下記のようになっていれば OK．

```bash
$ ls -la | grep github
```

実行結果（細かな文字列や数値は異なっていて OK）．

```bash
-rw-------  1 vagrant vagrant 1675 11月 20 12:18 2014 github_rsa
-rw-r--r--  1 vagrant vagrant  411 11月 20 12:18 2014 github_rsa.pub
```

引き続き，以下のコマンドで ssh-key を動作するようにする．

```bash
$ eval $(ssh-agent)
```

実行結果（数値は異なっていて OK）

```bash
Agent pid 9899
```

作成したキーペアを ssh-agent に登録する．

```bash
$ ssh-add ~/.ssh/github_rsa
```

実行結果（ディレクトリなどは異なっていて OK）

```bash
Identity added: /home/vagrant/.ssh/github_rsa
```

以上で ssh-key の準備は完了だが，設定ファイルに変更を加える必要がある．

### **1.4 設定ファイルの書き込み**

- 下記のコマンドで設定ファイルに書き込む．
- 2 行目の「vi \~….」ではターミナル内でエディタを起動する．画面が変わりますがパニックにならないよう注意！
- このエディタにはインサートモードとコマンドモードの 2 つがあり，最初はコマンドモードで表示されるが，追記を行うにはインサートモードに変更する必要がある．
- インサートモードにするには「i」キーを押す．画面のどこかに—INSERT—や—挿入—などの文字列が表示される．その状態で「Host github」からの 5 行を追記する．
- 追記が終わったら「esc(コマンドモードに戻る)」→「:wq(保存して終了のコマンド)」→「enter(実行)」で元の画面に戻る．
- ミスったと感じたら「esc」→「:q!」→「enter」でエディタを終了できるので，もう一度「vi \~…..」を入力してやり直しましょう．

ターミナルで以下を 1 行ずつ実行．1 行目がファイルの作成で，2 行目でエディタでファイルを開く．

```bash
$ touch ~/.ssh/config
$ vi ~/.ssh/config
```

`.ssh/config`ファイルを開くと空なので以下を追記する．

```text
Host github.com
	HostName github.com
 	Identityfile ~/.ssh/github_rsa
 	Port 22
 	User git
```

続いて，設定ファイルの権限を変更する．ターミナルで以下を実行．

```bash
$ chmod 700 ~/.ssh/config
```

エラーが出なければ OK．

### **1.5 Github への ssh-key 登録**

Github のサイトにアクセスし，「設定」→「SSH keys」へ進む．「Add SSH key」をクリックして入力画面へ進む．

ターミナルで下記のコマンドを入力し，ssh-key を表示させる．

```bash
$ cat ~/.ssh/github_rsa.pub
```

実行結果（各自異なる文字列が表示される）

```bash
ssh-rsa ...
...
...
...
...
... localhost@0-mac
```

上のように文字列が表示されたら，「ssh-rsa」から全てコピーし，Github サイトの入力欄に貼り付ける．タイトルは PC 名など適当につけて OK．
入力したら「Add key」をクリックして終了．

※公開鍵は PC 毎にペアを作成するため，どの PC で発行した公開鍵なのか判別できるように名前をつけると良い．

### **1.6 初期設定**

Github のユーザ名とメールアドレスを登録する．ターミナルで下記コマンドを入力し，エンターを押す．

**！！！それぞれ自身のアカウントのものを入力すること！！！**

ここで mac ユーザはエラーが出ることがある．エラーが出た場合は 1.8 項を参照しよう．

```bash
$ git config --global user.name "hoge"
$ git config --global user.email "hoge@example.com"
```

続いて以下を入力して内容を確認する．

```bash
$ git config -l
```

実行結果

```bash
user.name=hoge
user.email=hoge@example.com
```

上で入力した内容に間違いなければ OK．エラーが出る場合は 1.8 参照．

### **1.7 Github との接続テスト(ターミナル)**

ターミナルで書きを実行．途中でなにか訊かれたら「yes」で進める．

```bash
$ ssh -i ~/.ssh/github_rsa git@github.com
```

実行結果（`Hi`の後は自分のユーザ名が表示される）

```bash
Hi hoge! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

上記のように表示されれば OK．

### **1.8 「xcrun: error」が発生する場合(mac のみ)**

この場合，git コマンドを実行すると「`Command Line Tools(macOS High Sierra version 10.13) for XCode`」がインストールされていないことが原因．
ウインドウが出てきたら指示に従ってインストールし，完了後に再度コマンドを実行すれば OK．
ウインドウが出てこない場合は以下のコマンドを実行すればインストールできる．

```bash
$ xcode-select --install
```

上記コマンドを実行してもうまく行かない場合，下記の手順を実行する．

- [https://developer.apple.com/download/more/](https://developer.apple.com/download/more/)にアクセスする．自身の AppleID とパスワードを入力する．

- 検索窓に`command line tools`を入力し，`Command Line Tools for Xcode 11.5`を探し，ダウンロード → インストール．

- 再度 1.6 項を進める．

## **2. プロダクトを登録するときにやること**

この手順を行うタイミング

- 新しいプロダクトを作成し，初めて Github にソースコードを共有するとき．

### **2.1 リポジトリを作成する**

- ブラウザで Github にアクセスし，新しいリポジトリを作成する．
- 名前はプロダクト名にしておくとわかりやすくて良い．(janken, mamopad など)
- 作成すると url が表示されるのでそのままにしておく．SSH のタブを選択しておく(後で url を使用するため)

### **2.2 【超重要】ディレクトリを変更する**

まず，ターミナルの作業ディレクトリをプロダクトのディレクトリに移動する必要がある．

**移動しない場合，PC の中身全てを Github にプッシュする羽目になるので必ず行うこと！！**

**この手順を行わなかった場合，PC の今後あらゆるコマンドでエラーが発生する，PC クラッシュするなどの危険が伴う．**

ターミナルで下記コマンドを入力する．(enter は押さないこと)
【重要】cd のあとにスペースを入れること

```bash
$ cd
```

入力したらプロダクトのフォルダをターミナルのウインドウ内にドラッグ&ドロップする．

すると，「cd 」の後にパスが表示されるので，間違いなければ「enter」を押す．

### **2.3 初期化**

下記コマンドを入力する．

```bash
$ git init
```

### **2.4 リポジトリの登録**

コードをアップするリポジトリ(宛先)を登録する．先程ブラウザで作成したリポジトリの url（`git@github.com/***`）を使用する．

ターミナルで以下を実行する．

```bash
$ git remote add origin リポジトリのurl
```

登録されているかどうかは以下のコマンドで確認できる．

```bash
$ git remote -v
```

確認結果

```bash
origin	git@github.com:******/******.git (fetch)
origin	git@github.com:******/******.git (push)
```

間違っている場合は以下で登録し直す．

```bash
$ git remote set-url origin 正しいurl
```

### **2.5 ファイルを add**

Git で管理するファイルを add する．アップするには「add」「commit」「push」の 3 手順が必要．

ターミナルで以下を実行する．add の後にはスペースが入る点に注意！

```bash
$ git add .
```

### **2.6 ファイルを commit**

commit はファイルのバージョンを作成するイメージ．コミットを重ねても，以前のコミットへ状態を戻すことでファイルの内容をもとに戻すことが可能．
以下のコマンドを入力する．

-m のあとの「""」内にコミットメッセージを追加する．(変更内容などが把握できるコメント)

```bash
$ git commit -m"コミットメッセージ"
```

### **2.7 ファイルを push**

push は実際にファイルをアップロードするイメージ．この段階ではじめて Github にコードが追加される．
下記コマンドを入力する．

```bash
$ git push origin master
```

実行結果

```bash
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 385 bytes | 385.00 KiB/s, done.
Total 4 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To github.com:******/******.git
   9ae72be..eb700ec  master -> master
```

ブラウザで Github のページを確認し，ファイルがアップされていれば成功！

## **3. 登録済みのプロダクトを変更したときにやること**

この手順を行うタイミング

- すでに Github リポジトリに登録してあるプロダクトに機能追加，更新などしたとき．

### **3.1 【超重要】ディレクトリを変更する**

- これを忘れると別のファイルを登録することになりものすごく面倒なので注意すること！！
- ターミナルで下記コマンドを入力する．cd のあとには必ずスペース入れる．(enter は押さないこと)

```bash
$ cd
```

入力したらプロダクトのフォルダをドラッグ&ドロップする．
すると，「cd」の後にパスが表示されるので，間違いなければ「enter」を押す．

以降は上記「2.5」「2.6」「2.7」と同様．

### **3.2 ファイルを add**

```bash
$ git add .
```

### **3.3 ファイルを commit**

```bash
$ git commit -m"コミットメッセージ"
```

### **3.4 ファイルを push**

```bash
$ git push origin master
```

実行結果

```bash
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 385 bytes | 385.00 KiB/s, done.
Total 4 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To github.com:******/******.git
   9ae72be..eb700ec  master -> master
```

ブラウザで Github のリポジトリを確認して登録されていれば OK！

以上だ( `･ω･)b
