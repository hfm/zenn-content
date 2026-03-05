---
title: "コミットメッセージを自分で書かない"
emoji: "🤖"
type: "idea"
topics: ["ai", "git", "claudecode"]
published: true
---
# git alias を使ってコミットメッセージをAIに書かせる
最近はこれを git alias に入れて、コミットメッセージをゼロから書くことがほぼなくなりました。

```gitconfig
aicommit = "!f() { COMMITMSG=$(claude --no-session-persistence --print 'Generate ONLY a one-line Git commit message in English using imperative mood. The message should summarize what was changed and why, based strictly on the contents of `git diff --cached`. DO NOT add an explanation or a body. Output ONLY the commit summary line.'); git commit -m \"$COMMITMSG\" -e; }; f"
```

これはほぼ KOBA789 さんの記事に書かれている設定を、文言を少しだけ調整して入れたものです。

- コミットメッセージをClaudeに書かせるgit aicommit - Write and Run https://diary.hatenablog.jp/entry/2025/11/23/233725

また aicommit を実行するたびに溜まっていく session が邪魔なので、`--no-session-persistence` をつけて Claude Code の session を残さず使い捨てにしてます。

生成されたコミットメッセージは多少調整してますが、 `Add foobar` みたいな、結局どんなものを追加したんだか分からないコミットメッセージを減らせて満足してます。

## 余談

ワンライナーの中身を軽く解説します。

git alias において、 `!cmd`  と先頭にビックリマークをつけると外部コマンド `cmd` やシェルスクリプト（sh なのか bash なのかあまり分かってないです）を実行できます。
`f() { ... }` は Shell Scripts における関数定義です。[^1]
`f() { ... }; f` の最後の `f` で、定義した `f` 関数を実行してます。

```bash
bash-5.3$ f(){ echo hello;}
bash-5.3$ f
hello
```
```bash
bash-5.3$ f(){ echo hello;}; f
hello
```

つまり、以下のコマンドを1つのシェル関数にして実行してます。

```bash
COMMITMSG=$(ckpd-claude --no-session-persistence --print 'Generate ONLY a one-line Git commit message in English, using imperative mood, summarizing what was changed and why, based strictly on the contents of `git diff --cached`. DO NOT add an explanation or a body. Output ONLY the commit summary line.')
git commit -m "$COMMITMSG" -e
```

Claude Code に "Generate ONLY a one-line Git commit message in English, using imperative mood, summarizing what was changed and why, based strictly on the contents of `git diff --cached`. DO NOT add an explanation or a body. Output ONLY the commit summary line." という指示を出して、その出力を `COMMITMSG` 変数にいれて、git commit につっこんでいる、という流れです。

[^1]:  `fname () compound-command` という仕様なので `a () {}` でも `b () {}` でもいいし、 `{}` じゃなくてもいいです。 https://www.gnu.org/software/bash/manual/html_node/Shell-Functions.html
