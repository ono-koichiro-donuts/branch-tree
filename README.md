# branch-tree

ブランチの親子関係を GitHub PR 情報をもとにツリー表示する CLI ツールです。

## 動作条件

| 依存 | バージョン | インストール方法 |
|------|-----------|----------------|
| Python 3 | 3.6+ | macOS 標準搭載 |
| git | - | 通常インストール済み |
| GitHub CLI (`gh`) | 2.0+ | `brew install gh` |

### gh の認証

```bash
gh auth login
```

## インストール

pathの通ったディレクトリにコピーして実行権限を付与して下さい。

```bash
# ~/bin に配置する場合
cp branch-tree ~/bin/branch-tree
chmod +x ~/bin/branch-tree
```

## 使い方

```bash
# カレントブランチのツリーを表示
branch-tree

# ブランチを指定
branch-tree <branch>

# PR 情報（issue番号・PR番号・タイトル）を表示
branch-tree --pr [branch]

# HTML 形式で stdout に出力
branch-tree --html [branch]

# HTML を生成してブラウザで開く
branch-tree --open [branch]

# ルートブランチを指定（デフォルト: develop）
branch-tree --root main [branch]
```

### オプション一覧

| オプション | 説明 |
|-----------|------|
| `--pr` | PR 番号・タイトル、紐づく issue を表示 |
| `--html` | HTML 形式で stdout に出力 |
| `--open` | HTML を生成してブラウザで開く（`--html` を兼ねる） |
| `--root <branch>` | ルートブランチを指定。指定するとリポジトリルートの `.branch-tree` に保存され、以降のデフォルトになる |

## 表示例

```
develop
└── #101 親PRのタイトル
      feature/parent-branch
    └── #123 カレントブランチのPRタイトル
          feature/current-branch   ← カレントブランチ
        ├── #130 子ブランチA
              feature/child-a
        └── #131 子ブランチB
              feature/child-b
```

`--pr` 指定時、PR に紐づく issue が存在する場合は issue 番号も先頭に表示されます。

```
└── #42 issue タイトル
    #101 PR タイトル
      feature/parent-branch
```

## リポジトリごとのルートブランチ設定

初回に `--root` を指定すると、リポジトリルートに `.branch-tree` が作成されます。

```bash
branch-tree --root main
```

以降そのリポジトリでは `main` がルートブランチとして使われます。`.branch-tree` をコミットすればチームで共有できます。

## 仕組み

- ブランチの親は `gh pr list --head <branch>` の `baseRefName` を辿って取得
- 子ブランチは `gh pr list --base <branch>` で取得
- 紐づく issue は GitHub GraphQL API (`closingIssuesReferences`) で取得
- ルートブランチ（デフォルト: `develop`）に到達するまで再帰的に親を辿る
