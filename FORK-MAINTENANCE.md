# FORK-MAINTENANCE — mimikun フォーク運用手順

mimikun/mastodon フォークを upstream（mastodon/mastodon）の**タグ付きリリース**へ追従させるための手順書。

## 大原則

> **`mimikun` ブランチには upstream をマージしない。**
> `mimikun` は常に「**upstream のリリースタグ + 独自コミットのみ**」の状態に保つ。

こうしておくと、新リリースへの追従は `git rebase --onto` 一発で済む。
独自コミットは設定・ツール系（`CLAUDE.md` / `mise.toml` / `Taskfile.yml` / `.github/ISSUE_TEMPLATE` 削除 など）が中心で、
upstream のコードとほぼ衝突しない。

逆に upstream の `stable-x.y` ブランチを取り込む（マージする）と、
独自コミットと upstream コミットが混在し、バージョン bump や Crowdin 翻訳の自動コミットで
**毎回大量の衝突**が発生する（2026-06-21 の整理前がこの状態だった）。

## リモート設定（確認用）

```
origin    git@github.com:mimikun/mastodon.git    # 自分のフォーク
upstream  git@github.com:mastodon/mastodon.git   # 本家
```

未設定なら:

```bash
git remote add upstream git@github.com:mastodon/mastodon.git
```

## 通常の追従手順（v4.6.1 / v4.7.0 など、次のリリースへ）

例として「現在 `mimikun` は `v4.6.0` ベース」→「`v4.7.0` へ上げる」場合。
パッチリリース（`v4.6.1`）でもマイナー/メジャー（`v4.7.0`）でも手順は同じ。

```bash
# 1) upstream の最新タグを取得
git fetch upstream --tags

# 2) 独自コミットだけを新タグの上に載せ替える
#    書式: git rebase --onto <新タグ> <前回ベースにしたタグ> mimikun
git rebase --onto v4.7.0 v4.6.0 mimikun
#    ↑ 第2引数「前回ベースにしたタグ」を間違えると upstream コミットが混入するので注意

# 3) 衝突が出たら解決して継続（独自ファイル中心なので基本は出ない）
#    git add -A && git rebase --continue
#    どうにもならなければ中断: git rebase --abort

# 4) 検証（下記セクション参照）

# 5) リモート反映（履歴が変わるので force-with-lease）
git push --force-with-lease origin mimikun
```

### `--onto` の引数の意味

```
git rebase --onto <新ベース> <旧ベース> <対象ブランチ>
```

- `<新ベース>`: 移動先。新しいリリースタグ（例 `v4.7.0`）。
- `<旧ベース>`: 「ここより後ろのコミットだけを運ぶ」という起点。前回ベースにしたタグ（例 `v4.6.0`）。
- `<対象ブランチ>`: `mimikun`。

→ 「`v4.6.0..mimikun` の独自コミットだけ」を `v4.7.0` の上に置き直す、という意味になる。

### `<旧ベース>` が分からなくなったら（必ず確認できる）

第2引数は「mimikun が**今載っているベースタグ**」。これを間違える（古いタグを渡す）と、
その間の upstream コミットまで独自コミット扱いで運ぼうとして衝突する。
迷ったら毎回これで確認してから `--onto` に渡せば間違えない:

```bash
git fetch upstream --tags
git describe --tags --abbrev=0 mimikun    # → 例: v4.6.1 と出れば、それが <旧ベース>
```

例: v4.6.1 → v4.7.0 に上げるなら `git rebase --onto v4.7.0 v4.6.1 mimikun`。
（前回 v4.6.1 に上げていれば、ベースは v4.6.0 ではなく **v4.6.1** になっている点に注意）

## 検証（push 前に必ず）

```bash
# a) バージョンが目的のものか
sed -n '5,20p' lib/mastodon/version.rb        # major/minor/patch を目視

# b) 独自ファイルが存在するか
ls -la CLAUDE.md mise.toml Taskfile.yml FORK-MAINTENANCE.md

# c) ISSUE_TEMPLATE が消えているか（独自方針）
ls .github/ISSUE_TEMPLATE/ 2>&1               # → No such file or directory が期待値

# d) 新タグとの差分が「独自コミットだけ」か
git log --oneline <新タグ>..mimikun           # 例: git log --oneline v4.7.0..mimikun
git diff <新タグ>..mimikun --stat             # 触れるのは独自ファイルのみが期待値
```

`git log <新タグ>..mimikun` に upstream の version bump / Crowdin 翻訳 / バグ修正が混ざっていたら手順ミス。
`git rebase --abort` でやり直す。

## 安全策

```bash
# 作業前にバックアップブランチを切っておく
git branch backup/mimikun-pre-<新タグ> mimikun
# 例: git branch backup/mimikun-pre-4.7.0 mimikun

# 問題なければ後で削除
git branch -D backup/mimikun-pre-<新タグ>
```

## 現在の独自コミットを確認する

**ここに一覧を書かない。** 手で維持すると git が持っている情報の劣化コピーになり、
コミットを積むたびに古くなる（実際に v4.6.0 時点の 7 件のまま放置されていた）。

```bash
git log --oneline (git describe --tags --abbrev=0 mimikun)..mimikun   # fish
git log --oneline $(git describe --tags --abbrev=0 mimikun)..mimikun  # bash/zsh
```

独自コミットを追加・変更したら、次回の rebase でそのまま新タグへ運ばれる。

## 独自変更を追加・更新する

`mimikun` は「ベースタグ + 独自コミット」の構成なので、独自変更は **`mimikun` の上にコミットを積むだけ**でよい。
追加した分は `<ベースタグ>..mimikun` の範囲に入るので、次回の `git rebase --onto` で**自動的に新タグへ運ばれる**。
特別な操作は不要。

### 新しい独自変更を入れる（例: 設定ファイル追加など）

```bash
git switch mimikun
# ファイルを編集・追加
git add -A
git commit -m "feat: 〇〇を追加"
git push origin mimikun        # 積むだけなので force 不要
```

### 既存の独自変更を更新する（例: mise の yarn バージョン固定を上げる）

最も簡単で安全なのは、**上書きの新コミットを積む**こと。履歴は伸びるが、フォーク運用では十分。

```bash
git switch mimikun
# mise.toml の yarn バージョンを書き換え
git add mise.toml
git commit -m "chore(mise): bump yarn to x.y.z"
git push origin mimikun
```

> 補足: 元のコミット自体を書き換えてきれいにしたい場合は `git rebase -i <ベースタグ>` で
> 該当コミットを `edit` / `squash` できる（ローカル端末の対話シェルで実行）。
> ただし履歴の書き換え + `git push --force-with-lease` が必要になり、衝突解決の手間も増えるため、
> 通常は上の「新コミットを積む」方式で十分。

### 注意点

- 独自変更は **upstream が触らないファイル**（`CLAUDE.md` / `mise.toml` / `Taskfile.yml` / 本ファイル等）に
  留めるほど、将来の rebase で衝突しない。
- upstream のコードファイル（`app/`, `config/`, `lib/` 等）を独自に改変すると、
  リリース追従のたびにそのファイルで衝突しやすくなる。改変する場合は最小限・局所的にする。

## デプロイ（実サーバの更新）

本ファイルの主題は**コードの追従**だが、サーバ側で必ず踏む罠が3つあるのでここに置く。
リリースごとの [Mastodon リリースノート](https://github.com/mastodon/mastodon/releases)
の確認、`bundle install` / `yarn install` / `rails db:migrate` /
アセットプリコンパイル / 各種再起動が要るのは従来どおり。

### 1. サーバ側は `git pull` ではなく `git reset --hard`

**`mimikun` ブランチは毎リリース `git rebase --onto` で載せ替えられ、SHA が総取り替えになる。**
サーバの作業ツリーは古い SHA を持ったままなので、`git pull` は
**「独自コミット × 2」を突き合わせるマージ**になる。

```bash
git status --short                    # 空であることを先に確認
git fetch origin
git reset --hard origin/mimikun
```

実例（2026-08-23、4.6.4 → 4.7.0）: サーバの `git status -sb` が
`## mimikun...origin/mimikun [ahead 118, behind 498]` を表示していた。
`ahead 118` は**サーバが独自コミットを持っている**のではなく、
**rebase 前の SHA を指しているだけ**。ここで `pull` すると大量に衝突するか、
通っても古いコードが混ざったマージコミットができる。

### 2. `git clean` を打たない

`public/system/`（アップロード済みメディア）と `.env.production` は
**gitignore 対象**なので、`git clean -fd` は**それらを削除する**。
`git reset --hard` は追跡対象しか触らないので安全。

### 3. systemctl の glob は `start` では効かない

```bash
# 停止: glob が効く（起動中＝ロード済みユニットに展開されるため）
sudo systemctl stop mastodon-sidekiq.service mastodon-web.service 'mastodon-streaming*'

# 起動: glob は何にも展開されない。明示名で書く
sudo systemctl start mastodon-web.service mastodon-streaming.service mastodon-sidekiq.service
```

停止中のユニットは systemd から見えなくなるため、`start` にパターンを渡すと
`Warning: systemctl start called with a glob pattern.` が出て**何も起動しない**。

`mastodon-streaming.service` は `mastodon-streaming@<port>.service` を引っ張る
ラッパーで、自身は `active (exited)` になる。**これは正常**（実体は `@<port>` の
`running` のほう）。

### 補足: マイグレーションを分割するかどうか

アプリ層（web / sidekiq / streaming）を**全部止めてから**流すなら、
`SKIP_POST_DEPLOYMENT_MIGRATIONS` による分割は**不要**。
分割はダウンタイムを短くするための手法であって、総所要時間は変わらない。
止めずに流す場合のみ、本体 → 再起動 → post-deployment の順に割る。

なお PostgreSQL / Redis / nginx は別ユニットなので、上記の `stop` では止まらない。
`db:migrate` はそのまま通る。
