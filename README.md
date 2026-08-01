# zenn-content

[Zenn](https://zenn.dev/) の記事をGitHubで管理するためのリポジトリ。

## 書く

```bash
npx zenn new:article        # 記事を新規作成（articles/ に .md ができる）
npx zenn preview            # http://localhost:8000 で公開後の見た目を確認
```

## 出す

記事の frontmatter を `published: true` にして push すると公開される。

```bash
git add . && git commit -m "記事を公開" && git push
```

## 決めごと

- **画像は `images/<記事のスラッグ>/` に置く。** 記事からは `/images/<スラッグ>/foo.gif` と絶対パスで参照する。外部URL（GitHubのraw等）は直接指さない
- **ファイル名がそのままURLのスラッグになる。** 半角英小文字・数字・ハイフン・アンダースコアで12〜50字。あとから変えるとURLが変わるので最初に決める
- `topics` は5個まで。`type` は `tech`（コードあり）か `idea`（読み物）

## 記事

| スラッグ | 題 | 状態 |
|---|---|---|
| `claude-code-clone-from-scratch` | Claude Codeを自作したら、特別なことは何もしていなかった | 下書き |

- [📘 Zenn CLIの公式ガイド](https://zenn.dev/zenn/articles/zenn-cli-guide)
