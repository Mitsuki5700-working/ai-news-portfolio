# 01. GitHub開発環境構築手順

ローカルPCからGitHubへコード・ドキュメントを公開できる状態にするための手順。

## 前提条件

- Windows PC
- Python導入済み
- Git導入済み（`git --version` で確認）

## 1. GitHub CLI (gh) のインストール

Windowsの場合、`winget` でインストールできる。

```powershell
winget install --id GitHub.cli -e --source winget
```

確認:

```powershell
gh --version
```

## 2. GitHubアカウントの作成

1. https://github.com/signup にアクセス（トップページからの場合は右上の **Sign up** ボタン）
2. メールアドレスを入力 → **Continue**
3. パスワードを設定 → **Continue**
4. `{GitHubユーザー名}` を決めて入力 → **Continue**（公開されるので、実名または一貫したハンドルネームを推奨。後から変更も可能だが、一度外部共有すると変えづらい）
5. 表示される確認コード・パズル認証を完了する
6. 登録したメールアドレス宛に届く確認コードを入力してメール認証を完了する
7. プラン選択画面が出た場合は無料の **Free** プランを選択
8. https://github.com/settings/emails にアクセスし、メールアドレスの横に **Verified** と表示されているか確認する

## 3. Personal Access Token (Classic) の発行

パスワードの代わりにCLIやAPIから使う認証情報を発行する。

### 3-1. 発行画面への移動（UI操作）

1. GitHubにログインした状態で、画面右上の**プロフィールアイコン**（丸いアイコン）をクリック
2. メニュー最下部の **Settings** をクリック
3. 左サイドバーを一番下までスクロールし、**Developer settings** をクリック
4. 左サイドバーの **Personal access tokens** を展開し、**Tokens (classic)** をクリック
5. 右上あたりの **Generate new token** ボタン → **Generate new token (classic)** を選択
6. パスワードの再入力を求められる場合は入力する（本人確認）

### 3-2. トークンの設定

7. **Note**: `{トークン名}`（例: `local-dev-setup`）を入力
8. **Expiration**: 用途に応じて設定（例: 30 days）
9. **Select scopes**: チェックボックス一覧から、目的別に必要な権限のみ選択する（最小権限の原則）
   - `repo` — リポジトリの読み書き（コード・Issue・PR等）。チェックすると配下の項目も自動で選択される
   - `workflow` — GitHub Actionsのワークフローファイル（`.github/workflows/*.yml`）の更新に必要
   - `admin:org` を展開し、そのうち `read:org` のみチェック（`admin:org`本体はチェックしない） — `gh` CLIのログインバリデーションに必要な読み取り専用スコープ
10. 一番下までスクロールし、**Generate token** をクリック

### 3-3. トークンの保存（UI操作）

11. 緑色の枠内に `ghp_` から始まる文字列が**一度だけ**表示される。この画面を離れると二度と表示されないので注意
12. 文字列の右側にある **📋（コピー）アイコン**をクリックしてクリップボードにコピーする（手動でのドラッグ選択は文字の欠落・混入の原因になるため避ける）
13. コピーした値を、パスワードマネージャーなど安全な場所に一時的に控える（リポジトリやドキュメントには絶対に記載しない）

### 3-4. 発行済みトークンのスコープ変更（必要な場合）

トークン文字列を変えずにスコープだけ追加・変更したい場合:

1. https://github.com/settings/tokens にアクセス
2. 対象のトークン名（`{トークン名}`）をクリック
3. スコープのチェックボックスを変更
4. 一番下の **Update token** をクリック（トークン文字列は変わらない）

## 4. gh CLIでの認証

発行したトークンを使ってCLIを認証する。

```bash
echo "{発行したPATトークン}" | gh auth login --hostname github.com --git-protocol https --with-token
```

確認:

```bash
gh auth status
```

`Token scopes: 'read:org', 'repo', 'workflow'` のように、意図したスコープが表示されていればOK。

## 5. ローカルプロジェクトの初期化

```bash
mkdir {プロジェクトフォルダ名}
cd {プロジェクトフォルダ名}
git init
```

## 6. GitHubリポジトリの作成とローカルへの接続

```bash
gh repo create {リポジトリ名} --public --source=. --remote=origin --description "{リポジトリの説明文}"
```

確認:

```bash
git remote -v
# origin  https://github.com/{GitHubユーザー名}/{リポジトリ名}.git (fetch)
# origin  https://github.com/{GitHubユーザー名}/{リポジトリ名}.git (push)
```

## 7. 初回コミット・push

```bash
git add {コミット対象ファイル}
git commit -m "{コミットメッセージ}"
git push -u origin {ブランチ名}
```

## 備考・トラブルシューティング

上記は本来の手順であり、以下は環境固有の問題が起きた場合の対処法（発生しない環境では不要）。

- **`git push`や`gh`のAPI通信がSSL証明書エラーで失敗する場合**: ウイルス対策ソフト（Avast等）のHTTPS通信スキャン機能が通信に割り込んでいる可能性がある。`curl https://api.github.com/rate_limit` で疎通確認し、失敗するようであれば該当ソフトの「Webシールド」「HTTPSスキャン」を一時的に無効化して再試行する。
- **PowerShellでパイプ経由(`Get-Content | gh ...`)でトークンを渡すと`Bad credentials`になる場合**: パイプ処理が文字列を壊すことがあるため、ファイルからのリダイレクト(`<`)やBashのhere-string(`<<<`)を使う方が安定する。
- **`gh auth login`実行時に`missing required scope 'read:org'`等のエラーが出る場合**: GitHubのトークン編集画面で該当スコープを追加（トークン文字列は変わらない）し、再実行する。
