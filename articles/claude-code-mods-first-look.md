---
title: "Claude Codeのmodで「上限メーター」と「APIキーの伏せ字」を作った。コマンド3行で入る"
emoji: "🧩"
type: "tech"
topics: ["claudecode", "claude", "ai", "typescript"]
published: true
---

## 本記事について

10月1日、Claude Codeに「mod」という仕組みが入りました。Claude Codeの動きや画面に、自分で機能を足せる仕組みです。

説明を読むだけでは何に使えるのかつかみにくいので、困っていたことを2つ、実際にmodにしました。

1つ目は利用上限です。作業の途中で上限に達すると、Claudeはそこで止まります。あとどれくらい使えるかは、ふだんの画面には出ていません。そこで、入力欄の上に上限のメーターを出すmodを作りました。上限が近いときは、Claudeにも作業を区切らせます。

2つ目はAPIキーです。`.env`は、Claudeに読ませない設定にできます。それでも、PATHを確かめるつもりで`env`を実行させると、環境変数に入れたキーはそのままClaudeに渡ります。そこで、Claudeが読む前に伏せるmodを作りました。

![上限が92%のとき、Claudeが作業の区切り方を先に出した画面](/images/claude-code-mods-first-look/meter-opus.png)
*上限メーターを入れ、5時間枠を試しに92%として表示した画面。大きな作り直しを頼むと、Claudeは書き換える前に区切り方を出した（Claudeの返事は実際のもの）*

どちらもGitHubに置いてあり、Claude Codeでコマンドを3行打てば入ります。有料プランでClaude Codeを使っていて、上限で作業を止められたことがある方や、APIキーがClaudeに渡っていないか気になる方向けです。

https://github.com/shoujiki-panman/claude-code-mods

## 概要

| mod | 困っていたこと | 入れるとどうなるか |
|---|---|---|
| limit-meter | あとどれくらい使えるか分からないまま、上限で止まる | 5時間枠と週の枠の使った割合とリセットの時刻が、入力欄の上に出る。上限が近いと知らせが出て、Claudeも作業を区切る |
| secret-mask | `.env`を読ませない設定にしても、`env`の結果などに混じったキーはAIに送られる | Claudeが読む前に、キーやパスワードを`[MASKED]`に置き換える |

## 構成

1. modとは
2. 上限メーター（limit-meter）
3. APIキーの伏せ字（secret-mask）
4. 入れ方
5. 作るときに引っかかったところ
6. 入れる前に気をつけること

## 1. modとは

![modはClaude Codeの「出来事」に割り込む](/images/claude-code-mods-first-look/how-it-works.png)
*modのしくみ（筆者作成）*

modは、Claude Codeの動きや画面に、自分の機能を足す小さなプログラムです。Claude Codeは、コマンドを実行する、Claudeが答え終わる、画面を描く、といった「出来事」を起こしながら動いています。modはその出来事に割り込み、そのまま通すか、書き換えて通すか、止めて自分で答えるかを選びます。

これまでの「フック」との違いは、出来事を書き換えられることと、画面に表示を足せることです。今回の2つは、この2点を使っています。上限メーターは画面に表示を足し、伏せ字はClaudeに渡る結果を書き換えます。

使えるのはClaude Code 2.1.287以降です。仕組みの詳細は、[modの公式ドキュメント（英語）](https://code.claude.com/docs/en/plugins/mods/overview)にあります。

## 2. 上限メーター（limit-meter）

### 入力欄の上に、上限の残りを出す

![入力欄の上に、5時間枠と週の枠が出ている](/images/claude-code-mods-first-look/meter-real.png)
*実際の画面。5時間枠は0%、週の枠は17%で、それぞれのリセットの時刻も出る*

Claudeの有料プランには、5時間ごとの上限と、1週間ごとの上限があります。メーターは、この2つの使った割合と、リセットされる時刻を出します。数字は、Claude Codeが応答と一緒に受け取っている値そのままです。

使った割合が70%を超えると黄色、90%を超えると赤になります。

### 上限が近づいたら知らせる

![右上の知らせと、黄色になったメーター](/images/claude-code-mods-first-look/meter-toast.png)
*5時間枠を試しに85%として起動した画面。右上に知らせが出る*

5時間枠が80%と95%を超えたとき、週の枠が90%を超えたときに、右上に知らせを出します。同じ枠の同じ段階では、知らせは1回だけです。

### Claudeにも、作業を区切るよう伝える

5時間枠が90%を超えると、頼みごとに次の一文を添えてClaudeに渡します。この一文は画面には出ません。

> [limit-meter]利用上限が近づいています（5時間枠は92%使用、4:00にリセット）。上限に達すると、作業は途中で止まります。この依頼で2つ以上のファイルを書き換えるなら、書き換える前に、区切り方を短く提案して、ユーザーの返事を待ってください。1つのファイルで済む作業なら、そのまま進めてかまいません。

冒頭の画面は、この状態で「ログイン画面を、今風のデザインに全部作り直して」と頼んだときのものです。Opus 5.5は、ログイン画面が3つのファイルでできていて、全部作り直すと2つ以上を書き換えることになる、と説明しました。そのうえで作業を3段階に分け、「まず1から始めてよいですか？それとも、上限のリセット（4:00）を待ってから3つまとめて進めますか？」と聞いてきました。

### ポイント

- 同じ頼みごとをHaiku 4.5にすると、区切り方を出さずに書き換えを始めました。一文の効き方は、モデルによって違います
- 上限の数字が出るのは、Pro、Max、Teamなどのプランで使っているときだけです。APIキーで使っているときは、代わりにそのセッションの料金を出します
- 上限が近いときの見え方は、`LIMIT_METER_DEMO=92,40 claude`のように割合を入れて起動すると確かめられます。リセットの時刻は本物のままです

## 3. APIキーの伏せ字（secret-mask）

### まず、.envは読ませない設定にする

APIキーを守る基本は、`.env`をClaudeに読ませないことです。Claude Codeの設定ファイル（`.claude/settings.json`）に、次のように書きます。

```json
{
  "permissions": {
    "deny": ["Read(./.env)", "Read(./.env.*)"]
  }
}
```

この設定を入れて試すと、Claudeは`.env`を読めませんでした。Readツールで読ませても、`cat`で表示させても断られます。

### それでも、キーは混じる

![.envを読ませない設定でも、envの結果からキーが渡る。secret-maskを入れると[MASKED]になる](/images/claude-code-mods-first-look/mask-env.png)
*`.env`を読ませない設定のまま、`env`を実行させてキーの行を書き写させた画面。modなし（上）とsecret-maskを入れたとき（下）。値は記事のための偽物*

この設定で止まるのは、ファイルを読むことです。PATHを確かめるつもりで`env`を実行させると、環境変数に入れたAPIキーも一緒に出力され、そのままClaudeに渡りました。Claudeが読んだということは、その値がAIに送られたということです。

secret-maskを入れると、Claudeに届く前に、キーが`[MASKED]`に置き換わります。コマンドの結果のほか、読んだファイルや添付も対象です。`MAX_TOKENS=4096`のような、秘密ではない設定はそのまま残ります。

### 伏せるもの

| 種類 | 例 |
|---|---|
| 形で分かるキー | Anthropic、OpenAI、GitHub、AWS、Slack、Google、Stripe、JWT |
| 名前が秘密を表す設定 | `DB_PASSWORD=...`、`"password": "..."` |
| URLに入ったパスワード | `postgres://user:...@host` |
| 認証のヘッダー | `Authorization: Bearer ...` |
| 秘密鍵 | `-----BEGIN ... PRIVATE KEY-----`から`END`まで |

### ポイント

- 伏せ字は安全網です。`.env`を読ませない設定と一緒に使ってください
- キーは`sk-`や`ghp_`のような頭の部分だけ残し、どのキーかは分かるようにしています
- 伏せたときは、結果の最後に「値を読まずに使う方法を選んで」という一文を添えて、Claudeに伝えます
- `password: string`のような型や、`process.env.API_KEY`のようなコードは伏せません。Claudeがコードを読み違えないようにするためです
- 伏せるのは、Claudeに届く分だけです。画面の表示と、手元に保存される会話の記録には、元の値が残ります

## 4. 入れ方

Claude Codeで打つのは、次の3行です。

```
/plugin marketplace add shoujiki-panman/claude-code-mods
/plugin install limit-meter@shoujiki-panman-mods
/plugin install secret-mask@shoujiki-panman-mods
```

Claude Codeを起動し直すと読み込まれます。片方だけ入れてもかまいません。止めるときや外すときも、`/plugin`から操作します。

## 5. 作るときに引っかかったところ

2つとも、Claude Codeに頼んで作ってもらいました。実際に動かしてみて、初めて分かったことが4つあります。自分でmodを作るときの参考になると思います。

| 引っかかったこと | どうしたか |
|---|---|
| modの中の時計はUTCで動き、日本時間にならない | `/etc/localtime`から手元のタイムゾーンを読み、その時刻で出した |
| `$`（modからClaude Codeを操作する入り口）を関数に渡すと、検査で止められる | 渡す先の関数を、ファイルの上のほうで宣言した |
| Claudeに添えた「区切り方を提案して」が、Haiku 4.5には効かなかった | 「2つ以上のファイルを書き換えるなら」と条件をはっきり書いた。それでもHaiku 4.5には効かず、Opus 5.5には効いた |
| テスト用の道具では、会話の記録に残る直前の書き換えを、最後まで再現できない | 書き換えたあとの中身だけを確かめるテストにした |

modの中身を確かめるには、`claude plugin validate`が便利です。割り込む出来事と、呼び出す機能が一覧で出ます。作っている間は、これを何度も実行しました。

## 6. 入れる前に気をつけること

modは隔離されずに、Claude Codeと同じ権限で動きます。ファイルの読み書きも、ネットへの接続もできます。入れるのは、中身を確かめたmodだけにしてください。

今回の2つが使うものは、次のとおりです。どちらもネットには接続しません。

| mod | 使うもの |
|---|---|
| limit-meter | 上限の数字、`/etc/localtime`、`date +%z`の実行（時刻を手元のタイムゾーンで出すため） |
| secret-mask | Claudeに渡る前のコマンドの結果、読んだファイル、添付 |

## 使い分けの目安

| 困っていること | 入れるmod |
|---|---|
| 上限で、作業の途中に止められたことがある | limit-meter |
| 大きな作業を頼む前に、上限の残りを見たい | limit-meter |
| `.env`をClaudeに読ませたくない | modではなく、設定の`permissions.deny`に`Read(./.env)`を入れる |
| `env`やログの結果を、Claudeに見せることがある | secret-mask |

## 参考

- [Customize Claude Code with mods（開発元のブログ、2026年10月1日）](https://claude.com/blog/claude-code-mods)
- [modの公式ドキュメント（英語）](https://code.claude.com/docs/en/plugins/mods/overview)
- [公式の見本のmod（anthropics/claude-code-playground）](https://github.com/anthropics/claude-code-playground/tree/main/claude-code/mods)
- [Claude Codeが、上限に達しても「きりのいいところ」まで続けるようになった（前に書いた記事）](https://zenn.dev/shoujiki_panman/articles/claude-code-wrap-up-allowance)

## 宣伝：上限に達したら、別のAIで続ける

上限メーターで残りが分かっても、待てないときはあります。session-relayは、新しいチャットに「続きから」と打つだけで、前の会話を引き継ぐ道具です。利用の枠はツールごとに別なので、Claudeが止まってもCodexで続けられます。

入れ方は、ターミナルで次の2行です。

```
npm install -g @shoujiki-panman/session-relay
relay install
```

詳しくは[前に書いた記事](https://zenn.dev/shoujiki_panman/articles/session-relay-no-handoff)にまとめています。

https://github.com/shoujiki-panman/session-relay
