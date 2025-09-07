# Gitを触りながら理解する

## Gitの基本

### HEAD
■ HEADとは?
今いるブランチの最新のコミットを指すポインタ
新しくコミットするための目印となる。

■ 疑問
- HEADを過去のコミットに移動させてから新しくコミットするとどうなる?

■ 参考
https://zenn.dev/shgs/articles/c5e2314c649263

### ブランチ
■ ブランチとは?
~~分岐させたルートのことを「ブランチ」と呼ぶことが多そう。~~
コミット履歴を分岐させて、他ブランチ影響を与えずに並行して開発を進められる。
ブランチの実態は、分岐した先の最新のコミットを指すポインタ


■ 疑問
- ブランチは最新のコミットを指すポインタ→どうやってそれまでのコミットがそのブランチのコミットだと記録している?
→ 記録していない。
ブランチは現在のコミットオブジェクトのハッシュのみを持つ。コミットオブジェクトからは、先祖のコミットをたどっていける。現在のブランチが指すコミットの先祖はわかるが、過去のコミットがどのブランチされたものかを断定する情報は持たない。


■ 参考 
https://qiita.com/ymzkjpx/items/00ff664da60c37458aaa
https://qiita.com/matsu_aki/items/58dfff4efa5000947dae
https://qiita.com/kaitoy/items/ce6cd3426a16268389d9


### ローカルリポジトリ
■ ローカルリポジトリとは?
プロジェクトのすべてのファイルとその変更履歴を保存する。

■ 疑問
- ローカルリポジトリ = .gitディレクトリ?

### ステージングエリア


### ワークツリー


### add
■ addでは何をしている?
選択されたファイル内容をblobオブジェクト化してステージング領域に登録する。
- ファイルの内容を圧縮して、`.git/objects/`に保存。
- ファイルの中身のハッシュの先頭2桁がフォルダ名、残りの38桁がファイル名となる。
- ステージング領域`.git/index`に登録。

選択したファイルだけをコミットするための仕組み。

■ addしたときの内部的な動作を追ってみる。
`git ls-files --stage`で`.git/index`の中身を見ることができる。

git add していない状態

```
PS C:\Users\...\git-playground> git ls-files --stage
100644 58bd386d6670c594979b9c330e49a790fa8200a0 0       folder-1/1-a.txt
100644 ec1139d6d0c3a721a63675bd46e56326379157d3 0       folder-1/1-b.txt
100644 0117a9510c1e65bc685aa263fb72e082863e3385 0       folder-1/folder-1-1/1-1-a.txt
100644 4dfe012ea235bd6524fd2d9bdb92ca6165a049f6 0       memo.md
```

folder-1/1-a.txtに変更を加えgit add した状態
```
PS C:\Users\...\git-playground> git ls-files --stage
100644 6ed279136963940437e283a5118a9c65e6d433cd 0       folder-1/1-a.txt
100644 ec1139d6d0c3a721a63675bd46e56326379157d3 0       folder-1/1-b.txt
100644 0117a9510c1e65bc685aa263fb72e082863e3385 0       folder-1/folder-1-1/1-1-a.txt
100644 4dfe012ea235bd6524fd2d9bdb92ca6165a049f6 0       memo.md
```

folder-1/1-a.txtのハッシュが変わったのがわかる。
中身をのぞいてみると
```
変更前
PS C:\Users\...\git-playground> git cat-file -p 58bd386d6670c594979b9c330e49a790fa8200a0
tree・blobオブジェクトのテスト
ここは`folder1/1-a.txt`

変更後
PS C:\Users\...\git-playground> git cat-file -p 6ed279136963940437e283a5118a9c65e6d433cd
tree・blobオブジェクトのテスト
ここは`folder1/1-a.txt`

ここを変更して、ステージングの挙動を見てみる。
```

git add すると、新しいblobオブジェクトが作成され、`.git/index`に登録されるんですね。
indexには今回addしていないファイルのblobオブジェクトも登録されていました。
indexには、差分でなく今回のコミット時のプロジェクト全体のファイル群の状態が登録されているようです。

ある時点でのプロジェクト全体のファイル群の状態のことをスナップショットといいます。
Gitではコミットごとに「プロジェクト全体のファイル状態」を丸ごと保存しています。

### commit
■ commitでは何をしている?
- indexを読み取り新しくtreeオブジェクトを作成する。
  - treeオブジェクトとblobオブジェクトで現在のプロジェクト全体のファイルの中身とディレクトリ構造が記録できる。
  - だからcommitオブジェクトはルートのtreeオブジェクトを持つだけで、その時点でのスナップショットを記録できる。
- commitオブジェクトを作る
- ブランチが指す先を更新
  - HEADが指すブランチがこのコミットを指すようにする。
  - `refs/heads/ブランチ名`がこのコミットのハッシュに書き換わる。


### Gitを構成するオブジェクト
`git cat-file -p ハッシュ`でオブジェクトの中身をみられる。
ファイルの中身のハッシュがファイル名となっている。

1. Blobオブジェクト
   ファイルの中身そのものを保存するオブジェクト。
   ファイル名やディレクトリ構造の情報は持たない単なるバイト列。
   ```
   tree・blobオブジェクトのテスト
   ここは`folder-1/folder-1-1/1-1-a.txt`

   ここに文章を追加してみます。
   blobオブジェクトの中身がどうなるかを確認します。
   ```
2. Treeオブジェクト
   ディレクトリを表すオブジェクト。
   ```
   100644 blob 58bd386d6670c594979b9c330e49a790fa8200a0    1-a.txt
   100644 blob ec1139d6d0c3a721a63675bd46e56326379157d3    1-b.txt
   040000 tree 5830a7729b56f463d9a5b8acd63d62a4777d28d4    folder-1-1
   ```

3. Commitオブジェクト
   ```
   tree a8f0344d14e604428de05493e71698c7e603a172                // treeオブジェクトのハッシュ
   parent 6d84e43f18c0698edad94381eb51ab4831793678              // 親コミットのハッシュ
   author makim0939 <makim0939@gmail.com> 1757143763 +0900 
   committer makim0939 <makim0939@gmail.com> 1757143763 +0900 

   1-1-a.txtに文章を追加    // コミットメッセージ
   ```

4. Tagオブジェクト


BLOB Binary Large Object
画像、音声、動画などの不特定のバイナリ形式の大容量データを格納するデータ型

```
git log
commit 0750e3b710279779a61275c5d31caaf2bdb81b2f (HEAD -> branch-a, origin/branch-a)
Author: makim0939 <makim0939@gmail.com>
Date:   Sat Sep 6 16:29:23 2025 +0900

    1-1-a.txtに文章を追加

commit 6d84e43f18c0698edad94381eb51ab4831793678
Author: makim0939 <makim0939@gmail.com>
Date:   Sat Sep 6 16:27:18 2025 +0900

    tree・blobオブジェクトのテスト用にフォルダとファイルを配置

commit 7ec04bd07c84220a48b3ebc82a3dd9bbe8de48ed (main)
Author: makim0939 <makim0939@gmail.com>
Date:   Sat Sep 6 10:39:51 2025 +0900

    first commit
```