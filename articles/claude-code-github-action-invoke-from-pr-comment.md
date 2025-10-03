---
title: "Claude Code GitHub Actions は Forked PR のコメントで呼ぶとコケる"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ai", "claudecode", "github"]
published: true
---

[Claude Code GitHub Actions](https://docs.claude.com/en/docs/claude-code/github-actions) は現在 (2025/10/03)、Fork 先から出された Pull Request の中で、「@claude ほげふが」などコメントによって Claude Code を呼び出そうとしてもうまく動かない。以下の Issues で言及されている既知のもの。

- Comments in (forked) PR trigger claude-code-action with wrong branch https://github.com/anthropics/claude-code-action/issues/46
- Claude Code Action fails with "couldn't find remote ref" error for PRs from forks https://github.com/anthropics/claude-code-action/issues/223

issue_comment イベントで GitHub Actions を呼ぶとデフォルトブランチになるという仕様がある。PR にコメントする行為は issue_comment イベントになる。

- https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#issue_comment
- https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#pull_request_comment-use-issue_comment

Claude Code Action は PR コメントで呼ばれた場合、次のような挙動になる。

1. isPR というフラグが true になり、
   - https://github.com/anthropics/claude-code-action/blob/14ac8aa2/src/github/context.ts#L174
2. isPR == true では、ブランチをチェックアウトするために `git fetch origin ${branchName}` でブランチを取ろうとする
   - https://github.com/anthropics/claude-code-action/blob/14ac8aa2/src/github/operations/branch.ts#L32-L57

デフォルトブランチに Fork 先のブランチは存在しないので git fetch は `couldn't find remote ref` エラーで失敗してしまう。

GitHub Actions の仕様に対して、Claude Code GitHub Action が Forked PR を考慮していない実装になっており、今回の問題が起きているものと思われる。
