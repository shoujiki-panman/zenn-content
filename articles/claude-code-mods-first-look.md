---
title: "Claude Codeに「mod」が来た。危ないコマンドを止める機能も、Claudeに頼めば足せる"
emoji: "🧩"
type: "tech"
topics: ["claudecode", "claude", "ai", "typescript"]
published: true
---

## 本記事について

Claude Codeに作業を任せていて、「そのコマンド、本当に実行して大丈夫？」とヒヤッとしたことはありませんか。

10月1日、Claude Codeに「mod」という仕組みが入りました。Claude Codeに、自分専用の機能をあとから足せる仕組みです。足したい機能は、自分でプログラムを書かなくても、Claudeに頼めば作ってもらえます。

下の動画は、そのmodの見本のひとつです。Claudeが`rm -rf build`でフォルダを丸ごと消そうとしたところで止めて、消えるファイルの一覧と、実行するか取り消すかのボタンを出しています。

![危ないコマンドを止めるmodの動き](/images/claude-code-mods-first-look/blast-radius.gif)
*頼んでから、止められて、取り消すまで（画面はAnthropicが公開している見本「Blast Radius」。日本語の説明は筆者が加えました）*

この記事は、届いたお知らせメールをきっかけに、公式の発表と説明書をClaudeに読んでもらってまとめたものです。Claude Codeを使っていて、「ここがこう動いてくれたら」と思ったことがある方向けです。

## 概要

| modでできること | 例 |
|---|---|
| Claudeの操作を止める・書き換える | 危ないコマンドの前に確認を挟む |
| 画面に表示やボタンを足す | 会話の量を天気予報の形で出す |
| 自分専用の「/コマンド」を足す | 毎回打つ指示を1語にまとめる |
| もとからある機能を置き換える | 差分を見せる /diffを自分の表示にする |

## 構成

1. modとは
2. 公式の見本3つ
3. 作り方は、Claudeに頼むだけ
4. 試し方と配り方
5. 入れる前に気をつけること

## 1. modとは

### ゲームのMODと同じ考え方

ゲームにMOD（改造データ）を入れると、道具や画面が増えたり、遊び方が変わったりします。Claude Codeのmodも同じで、入れるとClaude Codeの動きや画面が変わります。

これまでもClaude Codeには、決まったタイミングで自分のプログラムを動かせる「フック」がありました。ただ、フックでは、Claudeの操作を書き換えることも、画面に新しい表示を描くことも、もとからある機能を置き換えることもできませんでした。modならそれができる、と開発元は説明しています。

### しくみ

![modはClaude Codeの「出来事」に割り込む](/images/claude-code-mods-first-look/how-it-works.png)
*modのしくみ（筆者作成）*

Claude Codeは、コマンドを実行する、ファイルを書き換える、画面を描く、といった「出来事」を起こしながら動いています。modは、この出来事に割り込む小さなプログラムです。割り込み方は、そのまま通す、書き換えてから通す、止めて自分で答える、の3通りです。冒頭の例は3つ目にあたります。

仕組みの詳細は、[Claude Codeのmodの公式ドキュメント（英語）](https://code.claude.com/docs/en/plugins/mods/overview)にあります。

### ポイント

- 書く言葉はJavaScriptかTypeScriptです。ただ、自分で書かなくても作れます（3で説明します）
- 同じ出来事に複数のmodが割り込むときは、読み込んだ順に動きます。別々の人が作ったmodを重ねて使えます
- 操作の許可を求める確認画面だけは、modでも変えられません
- 使えるのはClaude Codeの2.1.287以降です。画面に描くmodは、ターミナル版とデスクトップアプリのCodeタブで表示されます

## 2. 公式の見本3つ

AnthropicがGitHubに置いている見本のmodは、3つです。3つとも、Claude Codeの中でClaudeに作らせたものです。

### Blast Radius：危ないコマンドの前に止める

![消えるファイルの一覧と、実行・取り消しのボタン](/images/claude-code-mods-first-look/blast-radius.png)
*このまま実行すると9個のファイル（1.1MB）が消える、と分かる*

Claudeが、ファイルをまとめて消す`rm -rf`や、変更を捨てる`git reset --hard`、強制プッシュ、データベースの移行のようなコマンドを実行しようとすると、いったん止めます。そして、消えるファイルや失われる変更を一覧で見せる仕組みです。「Proceed」を押すと実行、「Cancel」を押すと取り消しで、取り消したことと理由はClaudeにも伝わります。10分答えなければ、取り消しの扱いです。

見本の説明には「安全網であって、権限の仕組みではない」とあります。`bash -c`のように、別のコマンドで包んだ削除などは見逃します。

### Replay Theater：直したところを1手ずつ見返す

![Replay Theaterの画面](/images/claude-code-mods-first-look/replay-theater.png)
*書き換える前と後を並べて、「次へ」「前へ」で順番に確かめる*

Claudeが直前に行った書き換えを、1か所ずつ見返せます。見本の説明では、関数の名前を変えさせたところ、3つのファイルで5か所が書き換わり、その5か所を順番に確かめられました。

### Token Weather：会話の量を天気予報で見せる

![Token Weatherの表示](/images/claude-code-mods-first-look/token-weather.png)
*使った量が増えるにつれて、晴れ、にわか雨、嵐と変わる*

Claudeが一度に覚えておける会話の量のうち、どれだけ使ったかを入力欄の上に出します。会話が長くなってClaudeが前の話を忘れたように見えるとき、その理由がひと目で分かる、というのが作った狙いです。

3つとも、プログラムを書いたのはClaudeです。なお、見本は公式の製品としてではなく、サポートなしで公開されています。

## 3. 作り方は、Claudeに頼むだけ

### 頼み方

```
> 入力欄の上に、いまのgitのブランチ名を出すmodを作って
```

Claudeは、mod作りの手順書（最初から入っているplugin-authoringというスキル）を読んで、ファイルを書きます。

### ポイント

- 書き始めると「Enable hot reloading for this session?」と聞かれます。「Enable for this session」を選ぶと、そのやり取りが終わった時点で読み込まれ、使っている画面のまま試せます
- 「Not now」を選んだ場合は、次にそのセッションを開いたときに読み込まれます
- Claudeが作ったmodは、作ったセッションの中だけで動きます。ほかでも使いたいときは、フォルダを別の場所に写して`claude --plugin-dir`で起動します
- 開発元は、modに向いているのは「みんなに配るほどではないけれど、自分は欲しい」機能だと説明しています

## 4. 試し方と配り方

### 見本を試す

冒頭のBlast Radiusは、ターミナルで次の3行です。

```
git clone https://github.com/anthropics/claude-code-playground.git
cd claude-code-playground/claude-code/mods
claude --plugin-dir ./blast-radius
```

### 配り方

modは「プラグイン」という入れ物に入れて配ります。

| 配る相手 | 方法 |
|---|---|
| 数人 | フォルダやzipを渡す |
| チーム | 自分たちのプラグイン置き場（マーケットプレイス）を作る |
| 会社全体 | 管理者が一括で入れる |
| 誰にでも | 公開の置き場を作るか、Anthropicのディレクトリに申請する |

入れたプラグインを止めたり外したりするのは、Claude Codeの`/plugin`からです。

## 5. 入れる前に気をつけること

### modは隔離されずに動く

modは、Claude Code自体と同じ権限で、パソコンの上でそのまま動きます。公式の説明書によると、ファイルの読み書きも、ネットへの接続も、パスワードの入った設定の読み取りも可能です。確認が出る前に、Claudeの操作を許可してしまうこともできます。

### ポイント

- 入れるのは、信頼できる作者のmodだけにします
- 入れる前に`claude plugin validate`を実行すると、そのmodが何に割り込み、どんな機能を呼ぶかの一覧が出ます
- TeamやEnterpriseのプランでは、「sec-default」という見張り役のmodが最初に読み込まれ、会社が決めた設定や指示を利用者のmodから守ります

## 使い分けの目安

| こんなとき | modでできること |
|---|---|
| 本番の設定ファイルには、勝手に触ってほしくない | 本番に関わるコマンドの前に、必ず確認を挟む |
| 同じ指示を毎回打っている | 1語の「/コマンド」にまとめる |
| テストやビルドの結果を、いつも見えるところに置きたい | 会話の横の小窓に、結果を出し続ける |
| コマンドの結果に、パスワードが混じることがある | Claudeが読む前に消す |
| 会社で、AIの使い方の決まりを守らせたい | 管理者が、使ってよいmodや操作を制限する |

## 参考

- [Customize Claude Code with mods（開発元のブログ、2026年10月1日）](https://claude.com/blog/claude-code-mods)
- [modの公式ドキュメント（英語）](https://code.claude.com/docs/en/plugins/mods/overview)
- [見本のmod（anthropics/claude-code-playground）](https://github.com/anthropics/claude-code-playground/tree/main/claude-code/mods)

画面写真はanthropics/claude-code-playground（Apache License 2.0）のものに、日本語の説明を加えました。

## 宣伝：フックで作った自動改題の仕掛け

前の記事では、Claude Codeの「フック」を使って、新しいセッションの題名をClaudeに付け直させる仕掛けを作りました。フックは、決まったタイミングで指示を添えたり、プログラムを動かしたりする仕組みで、modより前からある機能です。

[前の記事](https://zenn.dev/shoujiki_panman/articles/claude-code-session-titles-hook)

https://github.com/shoujiki-panman/claude-code-session-titles
