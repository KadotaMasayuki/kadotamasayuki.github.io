
# GitHubを使い始める設定

毎回環境を変えるたびに迷っているので自分メモ。SSHでやります。

2021年ころから、パスワードでの認証はしないことになったらしく、 *Personal Access Token (PAT)* での認証になったようだが、認証が有効な期限があって、期限が切れたときにいつも慌てる、という理由で、SSHを選択。

## 環境

```
$ uname -a
Linux _ 6.12.74+deb13+1-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.12.74-2 (2026-03-08) x86_64 GNU/Linux

$ date
2026年  6月 20日 土曜日 10:11:26 JST
```

## gitコマンドなどを入れる

```
$ sudo apt install curl git
```


## アカウント設定

[GitHub](https://github.com)の登録を済ませておく。

gitコマンドの設定として、ユーザー名とメールアドレスを設定しておく。

[GitHubユーザー名または電子メールの記憶](https://docs.github.com/ja/account-and-profile/how-tos/email-preferences/remembering-your-github-username-or-email)によると、```ユーザー名は https://github.com/ の直後にあるものです。```とのこと。

```
$ git config --global user.name "ユーザー名"
$ git config --global user.email "メールアドレス"
```

※ダブルクオーテーションマークが必要

## SSH - 公開鍵とかを作る

httpsとかトークンとかだと、日数制限があっていつのまにか切れていて毎回戸惑うので、sshでやることにする。

### ローカルに秘密鍵・公開鍵を作る

まずは、ホームディレクトリの *.ssh* ディレクトリがあるか確認。

```
$ cd
$ ls -a
 ...  .ssh  ...
```

なければ作る。

```
$ mkdir .ssh
$ chmod 0700 .ssh
```

んで、中に入る。

```
$ cd .ssh
```

以下のコマンドを実行する。

```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/(username)/.ssh/id_rsa):
```

ファイル名を聞かれるので、デフォルトでなく別の名前を指定する。
ここでは *id_rsa_githubcom* にしてみる。

続けて、以下のようにパスフレーズの入力を聞かれるが、設定せずに空白でENTERする。

```
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

色々メッセージが出て、終了する。

```
Your identification has been saved in id_rsa_githubcom
Your public key has been saved in id_rsa_githubcom.pub
 ;
```

出来上がっているか確認。

```
$ ls -a
 ...  id_rsa_githubcom  id_rsa_githubcom.pub  ...
```

入力したファイル名がついた以下の２つのファイルができていればOK。

- id_rsa_githubcom ... 秘密鍵
- id_rsa_githubcom.pub ... 公開鍵

### GitHubに公開鍵を設定する

GitHubにログインした状態で、[https://github.com/settings/ssh](https://github.com/settings/ssh)をクリックする。

または、プロフィールのSSH設定のリンクを辿っても良い。
( github / settings / SSH and GPG keys / SSH keys / New SSH Key )

*SSH Keys* 欄の *New SSH Key* をクリックする。

*Title* に識別名（何でも良い。わかりやすい名前をつける）
*KeyType* はAuthentication Keyにする。Signing Keyではない。
*Key* に公開鍵(id_rsa_githubcom.pub)の中身を入れる。


vimでクリップボードにコピーしようとすると、設定によっては改行コードを示す文字もコピーされてしまう。なので、以下のコマンドでターミナルに中身を表示させてコピーするのが良いと思う。

```
$ cat id_rsa_githubcom.pub
.....
......
.......
```

ほかにも、LibreOffice Writerなどに頼っても良いと思う。

### .ssh/configファイルでSSH接続に対して使う鍵を設定する

鍵の名前をデフォルトではない値にしたので、このままではSSH接続で使われない。
そこで、 *.ssh/config* ファイルに設定する。

```
$ cat >> config <<EOF
>
> Host github.com
>   HostName github.com
>   User git
>   IdentityFile ~/.ssh/id_rsa_githubcom
>
> EOF
```

### SSH接続確認

```
$ ssh -T git@github.com
```

で

```
Hi （ユーザー名）! You've successfully authenticated, but GitHub does not provide shell access.
```

と出れば良い。
初回接続時は以下のメッセージが出ることが有る。

```
The authenticity of host 'github.com (20.27.177.113)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

*github.comの信ぴょう性が確立できていない。ED25519キーのフィンガープリントは＊＊＊だけど、これは知らない。続けて良いか？*

という内容。初めての接続だからローカルのSSHがこれを知らないのは当然。もちろん yes。

もしうまく行かないときは、*~/.gitconfig*ファイルに以下を設定する。

```
[url "git@github.com:"]
    InsteadOf = https://github.com/
```

それでもうまく行かないときは、-v オプションをつけて実行してログを表示してみると良い。

```
git -vT git@github.com
```

※ログは割愛。あとは自力で。

## リポジトリを操作してみる。

### クローン（ダウンロード）

GitHubのサイトで、自分が管理する適当なリポジトリのアドレスをコピーする。
なければ、適当なリポジトリを作ってからやる。

PC内の適当なディレクトリで、以下のコマンドでクローンする。

```
$ git clone https://github.com/ユーザー名/リポジトリ名.git
Cloning into 'リポジトリ名'...
remote: Enumerating objects: 20, done.
remote: Total 20 (delta 0), reused 0 (delta 0), pack-reused 20 (from 1)
Receiving objects: 100% (20/20), done.
Resolving deltas: 100% (4/4), done.
```

PCの現在のディレクトリに、リポジトリのディレクトリが出来上がる。

このリポジトリのディレクトリを、適当なパスに移動させる。または、このままここで作業しても良い。

または、最初から、リポジトリを置きたいディレクトリでクローンコマンドを入力しても良い。

次に、リポジトリディレクトリ内に移動する。

```
$ cd リポジトリ名
```

以下、すべての操作はリポジトリのディレクトリ内で行う。

### ファイルやディレクトリを追加、または削除（管理対象リストの更新）

現在の状態を見てみる

```
$ ls
README.md  a.py

$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

なんも変更がない、ということがわかる。

次に、バージョン管理したいファイルを1個入れる。

```
$ cat > test.txt <<EOF
> it is test
> EOF

$ ls
README.md  a.py  test.txt

```

この状態で、どのように管理されているか見てみる。

```
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	test.txt

nothing added to commit but untracked files present (use "git add" to track)
```

管理されたファイルに変更はない。が、未管理のファイルがあるので追加しろ、と示される。

次に、gitで管理するよう指定する。

```
$ git add test.txt
```

または、以下のコマンドを使うと、現在のディレクトリにある未登録のファイルやディレクトリを全部追加する

```
$ git add .
```

ステータスを見てみると、

```
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   test.txt
```

管理対象として登録できたことがわかる。

### コミット（ローカルでの変更の確定）

コメントを -m で指定することで、コマンドライン内でコメントできる。

```
$ git commit -m "add test.txt"
[master 1b2b702] add test.txt
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
```

ステータスを見てみると、

```
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

他に更新は無いことがわかる。

### プッシュ（サーバーに送る）

```
$ git push origin HEAD

Enumerating objects: xx, done.
Counting objects: 100% (xx/xx), done.
Delta compression using up to xx threads
Compressing objects: 100% (xx/xx), done.
Writing objects: 100% (xx/xx), 92.70 KiB | 551.00 KiB/s, done.
Total xx (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To github.com:ユーザー名/リポジトリ名.git
   xxxxxxx..xxxxxxx  HEAD -> main
```

リポジトリのページで確認すると、更新されていることがわかる。


## その他

- clone, branch, pull, ... push, pull-request といったような開発手順はここでは述べない。
- 他者のリポジトリを使わせてもらう場合は、クローンだと他者のリポジトリへ変更が送られるので、フォークが良い。ただし、他者のリポジトリをメンテするならクローンで良いのかも。


以上
