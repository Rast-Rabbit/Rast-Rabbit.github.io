# らびっと あんど らすとだんじょん — サイト制作メモ

TRPGリプレイ・セッションログなどを公開する個人サイト。
GitHub Pages（静的サイトホスティング）+ Jekyll（静的サイトジェネレーター）で構成。

---

## 目次

1. [技術スタックの概要](#技術スタックの概要)
2. [ディレクトリ構造](#ディレクトリ構造)
3. [ローカル開発環境のセットアップ（WSL2）](#ローカル開発環境のセットアップwsl2)
4. [日常の作業フロー](#日常の作業フロー)
5. [セッションログを追加するには](#セッションログを追加するには)
6. [CSS設計の概要](#css設計の概要)
7. [よくある問題](#よくある問題)

---

## 技術スタックの概要

| 技術 | 役割 |
|------|------|
| **GitHub Pages** | 無料の静的サイトホスティング。`main`ブランチにpushすると自動で公開される |
| **Jekyll** | 静的サイトジェネレーター。HTMLテンプレート（`_layouts/`）とパーツ（`_includes/`）を組み合わせて最終的なHTMLを生成する |
| **Liquid** | Jekyllが使うテンプレート言語。`{{ }}` や `{% %}` で書く部分 |
| **WSL2** | WindowsでLinux環境を動かす仕組み。ローカルでJekyllを動かすために使う |

### Jekyllとは？

普通のHTMLサイトでは「ヘッダーを全ページにコピペする」必要がある。
Jekyllを使うと「ヘッダーは`_includes/header.html`に一回だけ書き、各ページは`{% include header.html %}`と書くだけ」になる。
`bundle exec jekyll build`を実行すると、全ページにヘッダーを埋め込んだHTMLが`_site/`フォルダに生成される。

GitHub Pagesは裏側でJekyllを自動実行してくれるので、ソースファイルをpushするだけで公開できる。

---

## ディレクトリ構造

```
trpg-home-page/
│
├── _layouts/
│   └── default.html        # 全ページ共通のHTMLの枠組み（head, ヘッダー含む）
│
├── _includes/
│   └── header.html         # ナビゲーションヘッダーのパーツ
│
├── _config.yml             # Jekyllの設定（サイト名、URLなど）
│
├── css/
│   ├── base.css            # ★ 全ページ共通（背景・ヘッダー・変数など）
│   ├── pages-home.css      # トップページ専用
│   ├── session-logs.css    # セッションログ一覧ページ専用
│   ├── external.css        # Externalページ専用
│   └── log-viewer.css      # セッションログ閲覧ページ（iframeビューワー）専用
│
├── assets/
│   ├── img/                # 背景画像など
│   ├── font/               # Webフォント
│   ├── icons/              # サービスアイコン（SVG/PNG）
│   └── og-image.png        # OGP画像（SNSでシェアしたときに出る画像）
│
├── index.html              # トップページ（layout: defaultを使用）
├── 404.html                # 404エラーページ
│
├── replays/
│   └── index.html          # リプレイ一覧ページ
│
├── session-logs/
│   ├── index.html          # セッションログ一覧ページ
│   ├── tukinashi_maen/
│   │   ├── index.html      # ★ iframeラッパーページ（Jekyllが処理する）
│   │   ├── log_export.html # ログ本体（外部ツール生成・直接編集しない）
│   │   ├── style.css       # ログ専用CSS（外部ツール生成）
│   │   └── images/         # キャラクター画像など（外部ツール生成）
│   └── tukinashi_β/
│       └── ...             # 同上
│
├── external/
│   └── index.html          # 外部リンク一覧ページ
│
├── Gemfile                 # RubyのパッケージリストをJekyll向けに定義
├── Gemfile.lock            # 実際にインストールされたバージョンの記録（自動生成）
├── _config.yml             # Jekyllの設定
└── README.md               # このファイル
```

> **`_site/` フォルダについて**
> Jekyllのビルド結果が入るフォルダ。gitignoreで除外済み（いつでも再生成できるため管理不要）。

---

## ローカル開発環境のセットアップ（WSL2）

> **WSL2とは？**
> Windows上でUbuntu（Linux）を動かす機能。Jekyllの動作確認をWindowsで行うための環境。
> Dockerより起動が速く、トラブルも少ない。

### 1. WSL2のインストール（初回のみ）

PowerShell（管理者権限）で実行：

```powershell
wsl --install
```

再起動後、Ubuntuが起動してユーザー名・パスワードを設定する。

### 2. Rubyのインストール（初回のみ）

WSL2のUbuntu上で実行：

```bash
sudo apt-get update
sudo apt-get install ruby-full build-essential zlib1g-dev
```

Gemのインストール先をホームディレクトリに設定（`~/.bashrc`に追記）：

```bash
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Jekyllをインストール：

```bash
gem install jekyll bundler
```

### 3. プロジェクトの準備（初回のみ）

プロジェクトフォルダに移動（WindowsのEドライブは`/mnt/e/`として見える）：

```bash
cd /mnt/e/trpg-home-page
bundle install
```

### 4. ローカルサーバーを起動する（毎回）

```bash
cd /mnt/e/trpg-home-page
bundle exec jekyll serve --livereload
```

ブラウザで `http://localhost:4000` を開く。
ファイルを保存するとブラウザが自動更新される（ライブリロード）。

### 5. サーバーを止めるには

ターミナルで `Ctrl + C`。

---

## 日常の作業フロー

```
1. WSL2でサーバー起動
   → bundle exec jekyll serve --livereload

2. VSCodeなどでファイルを編集・保存
   → ブラウザが自動更新

3. 見た目を確認

4. gitでコミット＆プッシュ
   → GitHub Pagesに自動反映（数分かかる場合がある）
```

---

## セッションログを追加するには

### ログツールでHTMLを生成した場合

1. `session-logs/` 内に新しいフォルダを作成（例: `session-logs/atarashii_log/`）

2. ログツールが生成したファイルをそのままフォルダに入れる：
   ```
   session-logs/atarashii_log/
   ├── log_export.html   ← ツール生成（触らない）
   ├── style.css         ← ツール生成（触らない）
   └── images/           ← ツール生成（触らない）
   ```

3. **ラッパーページを新規作成**（`index.html`）:
   ```
   session-logs/atarashii_log/index.html
   ```
   内容は既存のものをコピーしてタイトルだけ書き換えるだけでOK：
   ```html
   ---
   layout: default
   title: "ここにログのタイトル"
   nav: session-logs
   page_css:
     - /css/log-viewer.css
   ---
   <div class="log-viewer">
     <iframe id="log-frame" src="log_export.html" title="ここにログのタイトル"></iframe>
   </div>
   <script>
     document.getElementById('log-frame').addEventListener('load', function () {
       var inner = this.contentDocument;
       if (!inner) return;
       var h = inner.querySelector('.header');
       if (h) h.style.display = 'none';
     });
   </script>
   ```

4. **`session-logs/index.html` にカードを追加**:
   既存の `<article class="log-card">` ブロックをコピーして、タイトル・説明・日付・タグを書き換える。
   リンクは `href="./atarashii_log/"` とする。

5. **ログを再生成したときは** `log_export.html`, `style.css`, `images/` を上書きするだけ。
   `index.html`（ラッパー）は変更不要。

### なぜこの構造にしているか？

ログツールが生成するHTMLは触らないことで、「再生成したらpushするだけ」という手順にしている。
サイトのヘッダーや背景はラッパーページ（`index.html`）が提供し、ログ本体はiframeで読み込む。

---

## CSS設計の概要

```
base.css         ← 全ページ必ず読み込む。背景・ヘッダー・フォントなど全体共通。
pages-home.css   ← トップページのみ（タイトルアニメーション・メニューカードなど）
session-logs.css ← セッションログ一覧ページのみ（カード・タグ・フィルターなど）
external.css     ← Externalページのみ（外部リンクカードなど）
log-viewer.css   ← ログ閲覧ページのみ（iframeを全画面にするだけのシンプルなCSS）
```

読み込み順（`_layouts/default.html`で定義）：
1. `base.css`（共通）
2. ページ固有CSS（`page_css`変数で指定）

各ページのfront matter（HTMLファイルの先頭の`---`で囲まれた部分）で指定：
```yaml
page_css:
  - /css/pages-home.css
```

---

## よくある問題

### Q. ローカルで確認するとCSSが当たっていない

- `bundle exec jekyll serve` で起動しているか確認
- ブラウザのキャッシュをクリア（Ctrl+Shift+R）
- CSSファイルのパスが `/css/` で始まっているか確認（相対パスではなく絶対パス）

### Q. ログのiframeにヘッダーが二重に出る

ログツールがヘッダーを自動生成する場合がある。
ラッパーページの`<script>`タグが内側のヘッダーを非表示にするので、通常は自動で解決する。
なお、ログをiframeでなく直接 `log_export.html` のURLで開いた場合は外側のヘッダーは表示されない（正常）。

### Q. `_site/` フォルダが巨大になっている

`_site/` は `.gitignore` で除外済みなので、gitには含まれていない。
ローカルで `bundle exec jekyll clean` を実行して削除できる。

### Q. GitHub Pagesで更新が反映されない

- mainブランチにpushしたか確認
- GitHubのリポジトリ → Actions タブでビルドが成功しているか確認
- ブラウザのキャッシュをクリア
