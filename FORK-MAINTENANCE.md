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

## 現在の独自コミット（2026-06-21 時点 / v4.6.0 ベース）

`git log --oneline v4.6.0..mimikun` で確認できる。内訳:

1. 🦻< start my fork（空マーカーコミット）
2. docs: add CLAUDE.md
3. ISSUE_TEMPLATE 削除
4. feat: add mise.toml
5. feat(mise): add yarn
6. feat: add mimikun's taskfile
7. docs: add FORK-MAINTENANCE.md（本ファイル）

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

## デプロイについて（スコープ外メモ）

本手順は**コードの追従のみ**。実サーバ更新時は別途、リリースごとの
[Mastodon リリースノート](https://github.com/mastodon/mastodon/releases) を確認し、
`bundle install` / `yarn install` / `rails db:migrate` / アセットプリコンパイル / 各種再起動などを行うこと。
