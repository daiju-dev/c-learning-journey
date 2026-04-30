# git の本質ガイド

## 0. git って何?(超ざっくり)

**git = ファイルの変更履歴を全部記録する装置**

ゲームのセーブデータみたいなもん。
- どんな状態でも「セーブ」して残せる
- 後から「あの時のセーブデータに戻る」ができる
- 誰がいつ何を変えたか全部わかる

これが理解できれば残りは応用。

---

## 1. 3つのエリア(これが最重要)

git は3つのエリアでファイルを管理する。

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ 作業ディレクトリ │  │   ステージング  │  │  リポジトリ     │
│                 │  │                 │  │                 │
│ 編集中のファイル │→│ コミット待機所  │→│ 確定した履歴    │
│                 │  │                 │  │                 │
└─────────────────┘  └─────────────────┘  └─────────────────┘
       ↑                    ↑                    ↑
   vimで編集            git add            git commit
```

### 例えるなら...

| エリア | 例え |
|---|---|
| 作業ディレクトリ | レジに行く前のショッピングカート(いろいろ入れたり出したり) |
| ステージング | レジ台に置いた商品(これから買うと決めたやつ) |
| リポジトリ | 購入完了したレシート(履歴として残る) |

### なぜ2段階(add → commit)なの?

「変更の中で本当にコミットしたいものだけ選ぶ」ため。

例: あなたが `hello.c` と `main.c` を両方編集したとする。
- バグ修正(hello.c)
- 新機能(main.c)

これを別々にコミットしたい時、`git add hello.c` してコミット → `git add main.c` してコミット、と分けられる。
全部まとめてコミットしたければ `git add .` で全部ステージに乗せる。

---

## 2. ローカルとリモート(GitHub)

```
┌─────────────────────┐         ┌─────────────────────┐
│  ローカル(自分のPC) │         │  リモート(GitHub)   │
│                     │         │                     │
│   リポジトリ        │ ──push─→│   リポジトリ        │
│                     │ ←─pull──│                     │
└─────────────────────┘         └─────────────────────┘
```

| コマンド | 意味 |
|---|---|
| `git push` | ローカルの変更を GitHub にアップロード |
| `git pull` | GitHub の変更をローカルに取り込む |
| `git clone URL` | GitHub のリポジトリを丸ごとダウンロード(初回) |

**今あなたがやったこと:**
1. `git remote add origin URL` で「このローカルとGitHubを繋げる」と宣言
2. `git push -u origin main` で初回アップロード

---

## 3. 1日の基本サイクル

毎日これを繰り返す。

```bash
# ① 朝: 最新版を取り込む
git pull

# ② 作業: ファイル編集
vim hello.c

# ③ 状態確認(癖として何度も打つ)
git status

# ④ ステージに乗せる
git add hello.c

# ⑤ 確定する
git commit -m "feat: add hello function"

# ⑥ リモートに送る
git push
```

---

## 4. 必須コマンド一覧

これだけで日常使いの9割カバー。

| コマンド | 何をする | 使う頻度 |
|---|---|---|
| `git status` | 今の状態を見る | ★★★(超頻繁) |
| `git add ファイル` | ステージに追加 | ★★★ |
| `git add .` | 全部ステージに追加 | ★★★ |
| `git commit -m "メッセージ"` | 確定する | ★★★ |
| `git push` | リモートに送る | ★★ |
| `git pull` | リモートから取る | ★★ |
| `git log --oneline` | 履歴を見る | ★★ |
| `git diff` | 変更内容を見る | ★★ |
| `git diff --staged` | ステージ済みの変更を見る | ★ |

---

## 5. `git status` の読み方(これ読めれば強い)

```bash
$ git status
On branch main
Changes not staged for commit:
  modified:   hello.c        ← 編集したけどaddしてない

Changes to be committed:
  new file:   main.c         ← addした、コミット待ち

Untracked files:
  test.txt                   ← gitが知らない新ファイル
```

| カテゴリ | 意味 | 次のアクション |
|---|---|---|
| Changes not staged | 編集済み・未add | `git add` する |
| Changes to be committed | add済み・未commit | `git commit` する |
| Untracked files | gitが認識してない | 必要なら `git add` |

---

## 6. ブランチ(並行作業の仕組み)

ブランチ = **タイムラインの分岐**。

```
main:        A───B───C───D───E
                      \
feature:               F───G   ← 別の作業を並行で進める
```

### なぜ必要か

- `main` には安定版を置いておきたい
- 新機能を試したい → 失敗したら `main` が壊れる
- → 別ブランチ(`feature`)で試す → うまくいったら `main` に統合

### コマンド

```bash
git branch                    # ブランチ一覧
git checkout -b feature-x     # 新ブランチ作成+切替(よく使う)
git checkout main             # mainに戻る
git merge feature-x           # 現在のブランチに feature-x を統合
git branch -d feature-x       # 不要になったブランチを削除
```

学習リポジトリでは当面 `main` のみでOK。Hollw 等の実務では必須。

---

## 7. やらかした時の救済

| 状況 | コマンド |
|---|---|
| ファイル編集を取り消したい(addしてない) | `git checkout -- ファイル名` |
| addを取り消したい | `git restore --staged ファイル名` |
| 直前のコミットメッセージを修正 | `git commit --amend` |
| 直前のコミット自体を取り消し(変更は残す) | `git reset HEAD~1` |
| 全部リモートに合わせたい(核兵器) | `git fetch origin && git reset --hard origin/main` |

**核兵器コマンドはローカルの変更全部消えるので本当に困った時だけ。**

---

## 8. `.gitignore`(コミットしたくないファイル)

リポジトリに含めたくないファイルを書く。
C言語ならよくこう書く:

```
# .gitignore
*.o          # オブジェクトファイル
*.exe        # 実行ファイル
a.out        # gccのデフォルト出力名
hello        # 自分でビルドした実行ファイル名
build/       # ビルドディレクトリ
.vscode/     # エディタ設定
```

学習ディレクトリで作っておくと clean に保てる:

```bash
cd ~/learning/c-basics
cat > .gitignore << 'EOF'
*.o
*.exe
a.out
hello
build/
EOF
git add .gitignore
git commit -m "add .gitignore"
git push
```

---

## 9. コミットメッセージの書き方

学習段階でも癖をつけておく。

### 良い例

```
Day 1: implement hello world with name variation
fix: pointer dereference in malloc test
refactor: extract input validation into function
docs: update README with build instructions
```

### 悪い例

```
更新
aaa
wip
asdf
```

### コツ

- 動詞で始める(`add`, `fix`, `refactor`, `docs`, `feat`)
- 何をしたか1行で書く
- 短く(50文字以内が理想)

---

## 10. Day 1 の git 練習

第3章読み終わったらこれをやる。`git status` を **add前と後で2回打つ** のが体感のポイント。

```bash
cd ~/learning/c-basics
mkdir day01
cd day01
echo "Day 1 学習開始" > note.txt

git status              # ← Untracked files に note.txt が出る
git add note.txt
git status              # ← Changes to be committed に変わる

git commit -m "Day 1: create note"
git log --oneline       # ← 履歴に追加されたか確認
git push
```

3エリアの状態が変わるのが目で見える。これが体感できれば git は卒業。

---

## まとめ

| ポイント | 一言で |
|---|---|
| 3エリア | 編集 → ステージ → リポジトリ |
| 基本サイクル | pull → 編集 → add → commit → push |
| 状態確認 | `git status` を癖にする |
| 履歴確認 | `git log --oneline` |
| ブランチ | 並行作業に使う、当面はmainだけでOK |
| やらかし | 大体は救済できる、慌てない |

これだけ覚えれば日常の9割は戦える。残り1割(rebase, cherry-pick, reflog 等)は必要になった時に学べばいい。
