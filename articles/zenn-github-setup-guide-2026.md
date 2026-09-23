---
title: "Zenn × GitHub 連携セットアップまとめ"
emoji: "🔗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "github", "nodejs", "初心者"]
published: false
---

# Zenn × GitHub 連携セットアップまとめ

Windows環境で、VS CodeとGitHubを使ってZennに記事を投稿できるようにするまでの手順と、つまずいたポイントのまとめです。

---

## 0. 前提条件

この記事の手順を進めるにあたって、以下があることを前提としています。

- **OS**:Windows(Windows 11で確認。Windows 10でも基本的に同様の手順)
- **GitHubアカウント**:事前に取得済みであること([GitHub公式](https://github.com)から無料で作成可能)
- **Zennアカウント**:事前に取得済みであること(GitHubアカウントでもログイン可能)
- **エディタ**:Visual Studio Code(VS Code)がインストール済みであること
- **Git**:Windows環境にGitがインストール済みであること(Git BashやGit for Windowsなど)
  - 未インストールの場合は [Git for Windows](https://gitforwindows.org/) からダウンロード
- **Node.js**:未インストールでもOK(この記事の中でインストール手順を解説)

### 前提知識(なくても読み進められます)
- ターミナル(コマンドライン)の基本操作に不慣れでも、コマンドをそのままコピペすれば進められる内容にしています
- Gitの `add` / `commit` / `push` の意味が分からなくても大丈夫です(「2. 仕組みの解説」で説明しています)

---

## 1. ZennとGitHubの連携について

ZennはGitHubリポジトリと連携し、ローカルで書いたMarkdownを `git push` するだけで記事を公開・更新できる。

### メリット
- 普段使い慣れたエディタ(VS Codeなど)でローカル執筆できる
- Gitで変更履歴が残る
- `npx zenn preview` でプレビューを確認しながら書ける

---

## 2. 仕組みの解説

### 2-1. ZennとGitHubの連携の仕組み

「なぜGitHubにpushするだけでZennに自動反映されるのか」の裏側です。

1. Zenn側で「GitHub連携」の設定をすると、Zennは指定したGitHubリポジトリに対して**Webhook(ウェブフック)**という仕組みを登録する
2. Webhookとは、GitHub上で「pushされた」などの出来事(イベント)が起きた瞬間に、あらかじめ登録しておいたURL(ここではZenn側のサーバー)へ自動的に通知を送る仕組み
3. `git push` を実行すると、GitHubがこのWebhookを検知し、「リポジトリが更新されました」という通知をZenn側に送信する
4. 通知を受け取ったZennは、リポジトリの中身(`articles`フォルダや`books`フォルダ)を取得しにいき、Markdownファイルの内容をZenn上のデータベースに反映する
5. この一連の流れが数秒〜数十秒で自動的に行われるため、手動アップロードなしで記事が反映されたように見える

つまり「pushする」→「GitHubがZennに知らせる」→「Zennがファイルを読み込んで反映する」という3段階が、裏側で自動的に連続して起きているイメージです。

補足:GitHub側からは記事の**削除**はできない仕様になっている(誤操作防止のため)。削除したい場合はZennのダッシュボードから操作する必要がある。

### 2-2. git(add / commit / push)の仕組み

Gitは「今のファイルの状態」をいきなり記録するのではなく、**3つの段階**を踏んで変更を記録する。

```
作業フォルダ  →  ステージング  →  コミット履歴  →  GitHub(リモート)
(編集する場所)   (add)          (commit)         (push)
```

1. **作業フォルダ(Working Directory)**:VS Codeで実際にファイルを編集している場所。ここでの変更はまだGitに「記録する対象」として認識されていない

2. **`git add`(ステージング)**:編集したファイルを「次に記録する対象」として一時的に登録する操作。買い物カゴに商品を入れるイメージに近い。この時点ではまだ確定(保存)されていない

3. **`git commit`(コミット)**:ステージングした内容を、履歴として確定・保存する操作。買い物カゴの中身を「購入確定」するイメージ。コミットごとに一意のID(ハッシュ値)が振られ、後から「いつ・何を変更したか」を辿れるようになる。`-m` オプションで付けるメッセージは、その変更内容のメモ書き

4. **`git push`(プッシュ)**:ローカル(自分のパソコン)に確定した変更履歴を、GitHub(リモートのサーバー)へ送信する操作。ここで初めて、パソコンの外(GitHub)にファイルが反映される

この3段階に分かれているおかげで、「とりあえず変更を試したけどやっぱり戻したい」といった場合に、pushする前ならローカルだけで取り消しができる、というメリットがある。

---

## 3. セットアップ手順

### 3-1. Node.jsのインストール(Windows)

1. https://nodejs.org/ja/download にアクセス
2. **LTS版**(長期サポート版)を選ぶ。「Current」は最新機能版なので不要
3. Windows用の `.msi` インストーラーをダウンロード
4. ダブルクリックして起動し、基本的に「Next」で進める
5. 「Tools for Native Modules」という画面が出たら、**チェックを入れずに**そのまま「Next」でOK(Zenn CLI利用には不要)
6. インストール完了後、VS Codeを**完全に再起動**する

確認コマンド:
```bash
node -v
npm -v
```
バージョン番号が表示されればOK。

### 3-2. 作業フォルダの準備とVS Codeでの起動

1. デスクトップなどに作業用フォルダを作成(例: `zenn-contents`)
2. VS Codeの「ファイル」→「フォルダーを開く」でそのフォルダを開く
3. 「表示(View)」→「ターミナル(Terminal)」、またはショートカット `Ctrl + @` でターミナルを開く
   - Git Bashがインストール済みなら、ターミナル右上のプルダウンから選択も可能

**確認方法**:ターミナルのプロンプトの先頭に、作成したフォルダのパス(例: `C:\Users\ユーザー名\Desktop\zenn-contents>`)が表示されていればOK。違うフォルダが表示されている場合は `cd フォルダのパス` で移動する。

### 3-3. Zenn CLIのインストールと初期化

```bash
npm init --yes
npm install zenn-cli
npx zenn init
```

- `npm init --yes` → `package.json`(フォルダの設定ファイル)を作成
- `npm install zenn-cli` → Zenn CLI本体をインストール
- `npx zenn init` → `articles`フォルダ、`books`フォルダ、`README.md`などを自動生成

**確認方法**:
```bash
dir
```
を実行し、`articles`・`books`・`node_modules`・`package.json` などのフォルダ/ファイルが表示されていれば成功。`npx zenn init` 実行後に「🎉 Done!」と表示されるのも成功の目印。

### 3-4. Gitでのローカル管理とGitHubへのpush

1. GitHubで空のリポジトリを作成(**Public**を選択。Zenn連携にはPublicが必要)
2. ローカルをGit管理下にする:

```bash
git init
git add .
git commit -m "first commit"
```

3. GitHubリポジトリと接続してpush:

```bash
git branch -M main
git remote add origin https://github.com/ユーザー名/リポジトリ名.git
git push -u origin main
```

#### 各コマンドの意味
| コマンド | 説明 |
|---|---|
| `git branch -M main` | ブランチ名を `main` に統一する |
| `git remote add origin <URL>` | ローカルのGitフォルダとGitHub上のリポジトリを「origin」という名前で紐づける(接続設定のみ、まだファイルは送られない) |
| `git push -u origin main` | 実際にファイルをGitHubへアップロードする。`-u`により以後は `git push` だけで同じ場所に送れるようになる |

**確認方法**:`git push` の実行結果に `* [new branch] main -> main` と表示されればpush成功。念のため、ブラウザでGitHubリポジトリのページ(`https://github.com/ユーザー名/リポジトリ名`)を開き、`articles`フォルダなどのファイルが実際にアップロードされているか目視でも確認する。

### 3-5. 認証エラーのトラブルシューティング

別アカウントの認証情報が残っていて `403 Permission denied` エラーが出た場合の対処法。

1. Windowsの「資格情報マネージャー」→「Windows資格情報」で `git:https://github.com` を探して削除(見当たらない場合は次へ)
2. ターミナルで認証情報を確認・削除:

```bash
cmdkey /list
cmdkey /delete:ターゲット名
```

3. Git Credential Managerから直接削除する場合:

```bash
git credential-manager github logout <アカウント名>
```

4. 上記で該当アカウントが見つからない場合、全ての認証情報を消去:

```bash
git credential-manager erase
```
実行後、入力待ちになるので以下を1行ずつ入力:
```
protocol=https
host=github.com
```
最後に空行(Enterのみ)を送信して完了。

5. 再度pushを実行し、正しいアカウントでブラウザ認証する:

```bash
git push -u origin main
```

---

## 4. Zenn側の連携設定

1. Zenn(https://zenn.dev)にログイン
2. ダッシュボードの「GitHub連携」ページへ移動
3. 「リポジトリを連携する」から、連携するアカウント・リポジトリを選択

**確認方法**:連携完了後、Zennのダッシュボードの「デプロイ」または記事一覧ページに、GitHub上の`articles`フォルダにあるMarkdownファイルが下書き記事として自動的に表示されていればOK。表示に数十秒〜数分かかることもある。

---

## 5. 記事の作成方法

### 新規記事の作成

```bash
npx zenn new:article
```
`articles` フォルダにランダムな名前(例: `7b233825721edc.md`)のMarkdownファイルが生成される。

自分でファイル名(slug)を指定したい場合:
```bash
npx zenn new:article --slug my-first-article
```
※slugは英数字・ハイフン・アンダースコアのみ、12〜50文字。

### ファイルの構成

```markdown
---
title: ""
emoji: "😀"
type: "tech"
topics: []
published: false
---

ここから本文を書く
```

| 項目 | 説明 | 例 |
|---|---|---|
| `title` | 記事タイトル(**必須**。空だとデプロイに失敗する) | `title: "GitHubとZennを連携する方法"` |
| `emoji` | サムネイルに使う絵文字1文字 | `emoji: "🎉"` |
| `type` | `tech`(技術記事)か `idea`(アイデア・ポエム系) | `type: "tech"` |
| `topics` | タグ。最大5個 | `topics: ["github", "zenn", "初心者"]` |
| `published` | `true`で公開、`false`で非公開(下書き) | `published: false` |

### 本文のMarkdown記法例

```markdown
# 大見出し
## 中見出し
### 小見出し

**太字にしたい文章**

- 箇条書き1
- 箇条書き2

1. 番号付きリスト1
2. 番号付きリスト2

`短いコード`

​```javascript
console.log("コードブロックの例");
​```

[リンクの文字](https://example.com)

> 引用文
```

### プレビュー確認

```bash
npx zenn preview
```
`http://localhost:8000` のようなURLが開き、保存するたびに自動更新される。

### GitHubへの反映(公開・更新の流れ)

```bash
git add .
git commit -m "記事タイトルなど"
git push
```

| コマンド | 説明 |
|---|---|
| `git add .` | 変更したファイルを「次にコミットする対象」として登録する |
| `git commit -m "..."` | その時点の変更内容を履歴として記録する |
| `git push` | ローカルの変更をGitHubへ反映する |

**確認方法**:`git push` 後、Zennのダッシュボードの「デプロイ」を見て、対象ファイルの「保存に成功しました」のような表示が出ていればOK。エラーが出た場合は、`title`が空になっていないか、slug(ファイル名)のルールに違反していないかを確認する。

---

## 6. 公開・非公開について

- `published: false` → **下書き状態**。GitHubには反映されるが一般公開はされず、検索にも出ない。Zennのダッシュボードから自分だけ確認できる
- 公開したい時は `published: false` を `published: true` に変更して、再度 `git add` → `commit` → `push`

Zennには特定の人にだけ見せる「限定公開URL」の正式機能はない。非公開にしたい間は `published: false` のまま運用する。

---

## 7. スクラップ(Scraps)について

記事ほどしっかり書く必要のない、**メモや雑談・進行中の考えごとをまとめる機能**。

- Twitterのスレッドのように、短いメッセージを時系列で書き足していける
- 他のユーザーがコメント(返信)することもでき、対話形式で進む
- 「調べながら考えを整理する」「作業ログを残す」「気軽に情報交換する」用途に向く

### 使い分けの目安
- **ちゃんとまとめたい知見** → articles(記事)
- **気軽なメモ・途中経過・雑談** → scraps(スクラップ)

スクラップもZenn CLIで作成可能(例: `npx zenn new:scrap`)、GitHub連携でも管理できる。

---

## おまけ:GitHub PagesでWebサイトを公開する方法(参考)

Zennとは直接関係ないが、GitHubには「GitHub Pages」という無料の静的サイトホスティング機能もある。

1. リポジトリを用意する
   - 個人ページ:リポジトリ名を `ユーザー名.github.io` にする
   - プロジェクト用サイト:通常のリポジトリ名でOK
2. `index.html` などサイトのファイルを用意する
3. `Settings` → `Pages` で公開するブランチとフォルダを選択して保存
4. 数分待てば `https://ユーザー名.github.io/リポジトリ名/` で公開される

注意点:静的サイト(HTML/CSS/JS)限定。PHPやNode.jsのようなサーバーサイド処理は動かない。
