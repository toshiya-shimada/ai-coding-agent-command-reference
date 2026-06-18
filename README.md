# AI Coding Agent Command Reference

Claude Code / Codex CLI / Antigravity CLI / OpenCode / Aider のコマンド早引き対応表です。

このリポジトリは、Claude Code に慣れている開発者が、他の CLI ベース AI coding agent を試すときに迷いやすい操作を横断的に整理するための資料です。特に、`/init`、`claude -c`、`claude -p`、`/undo`、`/model`、`/mcp` のような「手癖になりやすい操作」が、Codex CLI、OpenCode、Aider、Antigravity CLI では何に相当するのか、あるいは相当する機能がないのかを確認できるようにしています。

最初に重要な前提を置きます。この表は「完全互換表」ではありません。Claude Code、Codex CLI、OpenCode、Aider、Antigravity CLI は、どれも AI coding agent の文脈で語られますが、設計思想がかなり違います。Claude Code と Codex CLI は agentic CLI としてかなり近いカテゴリにあります。OpenCode も同じく terminal-first の agent ですが、モデルプロバイダ非依存、agent 設定、OpenCode 独自の TUI/サーバー構成が前面にあります。Aider はより古典的な AI pair programming tool で、明示的にファイルを chat に追加し、diff と git commit を中心に作業する思想です。Antigravity CLI は Google Antigravity 2.0 の CLI として報じられていますが、公開されている詳細な CLI コマンドリファレンスを本稿作成時点では確認できていないため、確認できる範囲に限定して扱います。

## 目次

- [読み方](#読み方)
- [最重要対応表](#最重要対応表)
- [起動と対話セッション](#起動と対話セッション)
- [初期化とプロジェクト指示ファイル](#初期化とプロジェクト指示ファイル)
- [セッション継続と再開](#セッション継続と再開)
- [非対話実行とスクリプト利用](#非対話実行とスクリプト利用)
- [モデル指定とモデル切り替え](#モデル指定とモデル切り替え)
- [Plan / Ask / Read-only 系の操作](#plan--ask--read-only-系の操作)
- [ファイル追加と参照](#ファイル追加と参照)
- [差分確認、取り消し、コミット](#差分確認取り消しコミット)
- [MCP と外部ツール](#mcp-と外部ツール)
- [設定ファイルと永続ルール](#設定ファイルと永続ルール)
- [認証、ログイン、API key](#認証ログインapi-key)
- [更新、診断、アンインストール](#更新診断アンインストール)
- [ツール別の性格](#ツール別の性格)
- [参考リンク](#参考リンク)

## 読み方

表の中では、次の表記を使います。

- **同等**: ほぼ同じ目的で使える操作です。
- **近似**: 目的は近いが、挙動や思想が違う操作です。
- **対応なし**: 同じ概念のコマンドは確認できません。
- **未確認**: 公開リファレンスや手元の検証では確定できない操作です。

特に Aider については、Claude Code のような「セッション一覧から resume」「プロジェクトを `/init` して agent instruction を生成」といった体験とは違います。Aider は chat history、repo map、明示的な file add、auto commit によって作業を進めるツールなので、同じ列に並べても「同じことができる」という意味にはなりません。

Antigravity CLI についてはさらに注意が必要です。Google Antigravity は agent-first development platform として発表され、Antigravity 2.0 で CLI が含まれることも報じられています。しかし、少なくとも本稿作成時点で参照できる公開 CLI command reference は限定的です。そのため、この文書では Antigravity CLI の列に「未確認」を明示し、確認できないコマンドを作りません。

## 最重要対応表

Claude Code ユーザーが最初に知りたい操作だけをまとめると、次のようになります。

| やりたいこと | Claude Code | Codex CLI | OpenCode | Aider | Antigravity CLI |
| --- | --- | --- | --- | --- | --- |
| 対話セッション開始 | `claude` | `codex` | `opencode` | `aider` | 未確認 |
| 初回プロンプト付き起動 | `claude "..."` | `codex "..."` | `opencode --prompt "..."` | `aider -m "..."` または起動後に入力 | 未確認 |
| プロジェクト初期化 | `/init` | `/init` | `/init` | 対応なし。手動で `.aider.conf.yml` 等を作る | 未確認 |
| 指示ファイル | `CLAUDE.md` | `AGENTS.md` | `AGENTS.md` | `.aider.conf.yml`、`.env`、慣習ファイル等 | 未確認 |
| 直近セッション継続 | `claude -c` | `codex resume` | `opencode -c` | 近似: `--restore-chat-history` | 未確認 |
| 特定セッション再開 | `claude -r <session>` | `codex resume <session>` | `opencode -s <sessionID>` | 対応なしに近い | 未確認 |
| 非対話実行 | `claude -p "..."` | `codex exec "..."` | `opencode run "..."` | `aider --message "..."` | 未確認 |
| JSON 出力 | `claude -p --output-format json` | `codex exec --json "..."` | `opencode run --format json "..."` | 対応なしに近い | 未確認 |
| モデル指定 | `claude --model ...` | `codex -m ...` / `codex exec -m ...` | `opencode -m provider/model` | `aider --model ...` | 未確認 |
| 実行中モデル切替 | `/model` | `/model` | TUI/設定でモデル指定、`--model` | `/model` | 未確認 |
| 計画モード | `--permission-mode plan` など | `/plan` | `Tab` で `plan` agent、または `--agent plan` | `/ask`、`/architect` が近い | 未確認 |
| ファイル参照 | パス指定、mention | `/mention` | `@file`、`--file` | `aider file.py`、`/add`、`/read-only` | 未確認 |
| 差分確認 | UI / git diff | `/diff` | `/undo` 前に差分確認、通常は git diff 併用 | `/diff` | 未確認 |
| 取り消し | escape/checkpoint 系、git 復元 | 変更は git / diff 前提、`/archive` 等は会話管理 | `/undo`、`/redo` | `/undo` は aider commit を戻す | 未確認 |
| MCP 管理 | `claude mcp` | `codex mcp`、`/mcp` | `opencode mcp` | 対応なしに近い | 未確認 |
| 更新 | `claude update` | `codex update` | `opencode upgrade` | `aider --upgrade` | 未確認 |
| 診断 | 専用 `doctor` は公式 CLI 表に確認なし | `codex doctor` | `opencode debug` | `aider --help`、`/settings` | 未確認 |

この表で一番実用的なのは、`claude -p` と `codex exec` と `opencode run` と `aider --message` の対応です。CI やシェルスクリプトに組み込む場合、この行が最初の移行ポイントになります。次に重要なのは、`/init` の扱いです。Claude Code、Codex CLI、OpenCode には `/init` がありますが、Aider には同じ意味の `/init` はありません。

## 起動と対話セッション

Claude Code は `claude` で起動します。

```bash
claude
```

Codex CLI は `codex` で起動します。

```bash
codex
```

OpenCode は `opencode` で TUI を起動します。

```bash
opencode
```

Aider は `aider` で起動します。

```bash
aider
```

ただし、Aider は「まずファイルを chat に追加する」思想が強いので、実際には次のように対象ファイルを指定して起動することが多いです。

```bash
aider src/app.py tests/test_app.py
```

Aider 公式ドキュメントでも、編集したい source code files を `aider <file1> <file2> ...` として指定し、それらが chat session に追加されると説明されています。Claude Code や Codex CLI が「リポジトリ全体を agent が探索する」体験に寄っているのに対し、Aider は「編集対象を人間が明示して、必要に応じて repo map で補助する」体験です。

Antigravity CLI は、Antigravity 2.0 に CLI が含まれることは報じられていますが、この記事では起動コマンド名を確定しません。実際に使う場合は、インストール後の `--help` と Google の最新ドキュメントを確認してください。

## 初期化とプロジェクト指示ファイル

Claude Code の `/init` は、プロジェクトを解析して `CLAUDE.md` を作る、または改善するための操作です。Claude Code の世界では、`CLAUDE.md` はプロジェクトの長期的な指示、ビルド手順、テスト手順、設計上の注意、チーム規約を書く場所です。

```text
/init
```

Codex CLI にも `/init` があります。ただし、生成される中心ファイルは `AGENTS.md` です。Codex の slash command reference では、`/init` は current directory に `AGENTS.md` scaffold を生成するコマンドとして説明されています。

```text
/init
```

OpenCode にも `/init` があります。OpenCode でも、プロジェクトを分析して `AGENTS.md` を作成します。

```text
/init
```

Aider には、Claude Code の `/init` と同じ意味の初期化コマンドはありません。Aider の永続設定は `.aider.conf.yml`、環境変数、`.env`、コマンドラインオプションで行います。プロジェクトのコーディング規約を伝えたい場合は、README や専用の conventions file を chat に追加したり、Aider の設定を使ったりします。

```yaml
# .aider.conf.yml
model: anthropic/claude-3-5-sonnet-20241022
auto-commits: true
dark-mode: true
```

この違いは大きいです。Claude Code、Codex CLI、OpenCode は「agent が毎回読む project instruction file」を明確に持つ方向です。Aider は、設定ファイルと chat に追加されたファイルを中心に context を組みます。そのため、「Claude Code の `/init` 相当は Aider では何か」と聞かれたら、答えは「直接対応するコマンドはない。必要なら `.aider.conf.yml` と規約ファイルを手で用意し、必要に応じて `/read-only` や `/add` で読ませる」です。

## セッション継続と再開

Claude Code の直近セッション継続は `-c` / `--continue` です。

```bash
claude -c
claude --continue
```

特定セッションの再開は `-r` / `--resume` です。

```bash
claude -r auth-refactor
claude --resume auth-refactor
```

Codex CLI では、interactive session の再開に `codex resume` を使います。

```bash
codex resume
codex resume <session>
```

また、非対話実行の継続には `codex exec resume` を使えます。

```bash
codex exec "review the change for race conditions"
codex exec resume --last "fix the race conditions you found"
codex exec resume <SESSION_ID> "continue this task"
```

OpenCode では、直近セッション継続が `-c` / `--continue` です。

```bash
opencode -c
opencode --continue
```

特定セッションは `-s` / `--session` で指定します。

```bash
opencode -s <sessionID>
opencode --session <sessionID>
```

セッション一覧は次です。

```bash
opencode session list
opencode session list --max-count 20
opencode session list --format json
```

Aider は Claude Code や Codex CLI のような session picker とは違います。近いものとして chat history の復元があります。

```bash
aider --restore-chat-history
```

ただし、これは「過去の named session を選んで resume する」というより、Aider の chat history を復元する機能です。Aider で長期作業を扱う場合は、git commit、`.aider.chat.history.md`、`/save` などを組み合わせて再現性を作る方が自然です。

## 非対話実行とスクリプト利用

Claude Code の非対話実行は `-p` / `--print` です。

```bash
claude -p "このリポジトリの構成を説明して"
cat logs.txt | claude -p "失敗原因を要約して"
```

Codex CLI の対応は `codex exec` です。

```bash
codex exec "このリポジトリの構成を説明して"
cat logs.txt | codex exec "失敗原因を要約して"
```

Codex は JSON Lines 出力もできます。

```bash
codex exec --json "この差分のリスクをレビューして"
```

OpenCode の対応は `opencode run` です。

```bash
opencode run "このリポジトリの構成を説明して"
opencode run --format json "この差分のリスクをレビューして"
```

Aider の非対話実行は `--message` / `--msg` / `-m` です。

```bash
aider --message "README の改善案を出して"
aider -m "このファイルの docstring を改善して" src/app.py
```

Aider では、`--message` は LLM に 1 回メッセージを送り、返答を処理して終了するモードです。Claude Code の `-p` や Codex の `exec` と似ていますが、Aider はファイル編集と git commit の流れが強いため、スクリプトで使う場合は `--dry-run`、`--auto-commits`、`--yes-always`、`--git` 周辺の挙動を必ず確認してください。

## モデル指定とモデル切り替え

Claude Code では `--model` を使います。

```bash
claude --model sonnet
claude --model claude-sonnet-4-6
```

Codex CLI では `--model` / `-m` を使います。

```bash
codex -m gpt-5.4
codex exec -m gpt-5.4 "この関数をレビューして"
```

OpenCode では `provider/model` 形式が基本です。

```bash
opencode -m anthropic/claude-sonnet-4-20250514
opencode run -m openai/gpt-4.1 "この設計をレビューして"
```

Aider では `--model` を使います。

```bash
aider --model sonnet
aider --model o3-mini --api-key openai=<key>
aider --model anthropic/claude-3-5-sonnet-20241022
```

実行中の切り替えは、Claude Code、Codex CLI、Aider では `/model` が近い操作です。

```text
/model
```

OpenCode では CLI flag の `--model`、config、TUI 上のモデル選択が中心です。OpenCode のモデル名は provider を含むため、Claude Code よりも明示的です。これは OpenCode の強みでもあり、最初のつまずきでもあります。

## Plan / Ask / Read-only 系の操作

Claude Code では plan mode を使うことで、いきなり編集せずに設計や方針を出させられます。

```bash
claude --permission-mode plan
```

Codex CLI では slash command の `/plan` があります。

```text
/plan
/plan この認証方式を移行する計画を立てて
```

OpenCode では `plan` agent に切り替えるのが自然です。TUI では `Tab` で `build` と `plan` を切り替えます。CLI から agent を指定する場合は次のようにします。

```bash
opencode --agent plan
opencode run --agent plan "この変更の影響範囲を調べて"
```

Aider には Claude Code の plan mode と同じものはありませんが、近いものとして `/ask` と `/architect` があります。

```text
/ask このコードの設計上の問題を説明して
/architect この機能追加の実装方針を考えて
```

`/ask` はコードベースについて質問するが編集しないモードです。`/architect` は architect/editor mode に入るためのコマンドで、設計と編集を分ける思想に近いです。ただし、Claude Code の permission mode や OpenCode の read-only agent と同じ安全境界ではありません。

## ファイル追加と参照

Claude Code では、プロンプト内でパスを指定したり、UI 上でファイルを mention したりして対象を伝えます。

```text
src/auth.ts を読んで認証フローを説明して
```

Codex CLI では `/mention` が用意されています。

```text
/mention src/auth.ts
```

また、起動時に画像を添付する場合は `--image` / `-i` があります。

```bash
codex -i screenshot.png "この UI の問題点を説明して"
```

OpenCode は `@` によるファイル fuzzy search と、`opencode run --file` を使えます。

```text
@src/auth.ts の認証フローを説明して
```

```bash
opencode run -f src/auth.ts "このファイルをレビューして"
```

Aider はファイルを chat に追加する概念が非常に重要です。

```bash
aider src/auth.ts tests/test_auth.ts
```

起動後に追加する場合は `/add` です。

```text
/add src/auth.ts
```

参照専用として追加する場合は `/read-only` です。

```text
/read-only docs/auth-design.md
```

Aider では、必要以上に多くのファイルを chat に追加しないことが重要です。公式ドキュメントでも、対象ファイルを絞る方が良い結果につながると説明されています。Claude Code や Codex CLI のように agent に探索させるより、人間が編集対象を明示するスタイルだと理解すると使いやすいです。

## 差分確認、取り消し、コミット

Claude Code は作業結果を差分として確認し、必要なら git で戻す運用が基本です。Claude Code 固有の checkpoint や UI 上の確認もありますが、最終的な安全網は git です。

Codex CLI には `/diff` があります。

```text
/diff
```

会話を分岐したい場合は `/fork`、保存済み会話を再開したい場合は `/resume`、現在の session を片付けたい場合は `/archive` が使えます。

```text
/fork
/resume
/archive
```

OpenCode には `/undo` と `/redo` があります。

```text
/undo
/redo
```

OpenCode の `/undo` は、直前の変更を戻して元のメッセージを再表示し、プロンプトを調整してやり直す用途に向いています。

Aider では `/diff` と `/undo` が重要です。

```text
/diff
/undo
```

Aider の `/undo` は、Aider が作った最後の git commit を戻すためのコマンドです。Aider は変更を git commit する思想が強いので、`/undo` はかなり実用的です。逆に言えば、Aider を使うなら git repository の状態をきれいに保ち、Aider の commit 単位を理解しておく必要があります。

## MCP と外部ツール

Claude Code では `claude mcp` で MCP servers を管理します。

```bash
claude mcp
```

Codex CLI では `codex mcp` と TUI 内の `/mcp` が対応します。

```bash
codex mcp
```

```text
/mcp
```

OpenCode では `opencode mcp` があります。

```bash
opencode mcp add
opencode mcp list
opencode mcp auth <name>
opencode mcp logout <name>
```

Aider は、少なくとも Claude Code / Codex CLI / OpenCode と同じ意味での MCP 管理 CLI を中心機能として持つツールではありません。Aider は多くの LLM provider に接続できますが、MCP を前提とした agent tool platform というより、terminal pair programmer として理解する方が自然です。

Antigravity 2.0 については、Agent Skills、Hooks、Subagents、Extensions / plugins などの機能が報じられていますが、CLI の MCP 管理コマンドは本稿では未確認です。

## 設定ファイルと永続ルール

Claude Code の中心は `CLAUDE.md` です。プロジェクトごとの指示、ビルド手順、テスト手順、規約を置きます。

```text
CLAUDE.md
.claude/
```

Codex CLI は `AGENTS.md` を読みます。Codex manual では、Codex は作業前に `AGENTS.md` を読み、global scope と project scope の指示を組み合わせると説明されています。

```text
~/.codex/AGENTS.md
AGENTS.md
AGENTS.override.md
```

OpenCode も `AGENTS.md` を中心にしつつ、`opencode.jsonc` と `.opencode/` 配下に commands や agents を置きます。

```text
AGENTS.md
opencode.jsonc
.opencode/commands/
.opencode/agents/
```

Aider は `.aider.conf.yml`、環境変数、`.env`、起動時オプションで設定します。

```text
.aider.conf.yml
.env
.aiderignore
```

設定ファイルの対応を単純化すると、次のようになります。

| 目的 | Claude Code | Codex CLI | OpenCode | Aider |
| --- | --- | --- | --- | --- |
| プロジェクト指示 | `CLAUDE.md` | `AGENTS.md` | `AGENTS.md` | 規約ファイルを chat に追加、または README 等 |
| ツール設定 | `.claude/settings...` | `~/.codex/config.toml`、project config | `opencode.jsonc` | `.aider.conf.yml` |
| コマンド拡張 | slash commands / skills | custom prompts / skills / plugins | `.opencode/commands/*.md` | `/load`、`.aider.conf.yml`、外部スクリプト |
| ignore | 設定次第 | 設定次第 | 設定次第 | `.aiderignore` |

## 認証、ログイン、API key

Claude Code は Anthropic / Claude account のログインを使います。

```bash
claude auth login
claude auth status
claude auth logout
```

Codex CLI は `codex login` / `codex logout` を使います。

```bash
codex login
codex logout
```

自動化では `CODEX_API_KEY` を `codex exec` に渡す方式があります。

```bash
CODEX_API_KEY=... codex exec "このリポジトリをレビューして"
```

OpenCode は provider 認証を管理します。現在の CLI では `providers` が表示され、`auth` alias もあります。

```bash
opencode auth login
opencode auth list
opencode auth logout
```

Aider は provider ごとの API key を使います。

```bash
aider --model o3-mini --api-key openai=<key>
aider --model sonnet --api-key anthropic=<key>
```

また、環境変数や `.env` も使えます。

```bash
export OPENAI_API_KEY=...
export ANTHROPIC_API_KEY=...
```

Aider の help では `--openai-api-key`、`--anthropic-api-key`、`--api-key PROVIDER=KEY` などが確認できます。Aider は LiteLLM 経由で多くの provider を扱えるため、モデル名と provider key の指定を把握することが重要です。

## 更新、診断、アンインストール

Claude Code の更新は次です。

```bash
claude update
claude install stable
```

Codex CLI には `codex update` と `codex doctor` があります。

```bash
codex update
codex doctor
codex doctor --json
```

OpenCode には `opencode upgrade`、`opencode debug`、`opencode uninstall` があります。

```bash
opencode upgrade
opencode debug
opencode uninstall
opencode uninstall --keep-config
```

Aider は `--upgrade` を持ちます。

```bash
aider --upgrade
```

設定確認には、Aider の chat 内で `/settings` が使えます。

```text
/settings
```

## ツール別の性格

Claude Code は、Claude を中心にした完成度の高い agentic CLI です。`CLAUDE.md`、MCP、skills、plugins、hooks、subagents、background sessions など、エコシステム全体が Anthropic の設計に沿っています。Claude 系モデルを主力に使い、日々の開発を terminal で回すなら基準点になります。

Codex CLI は、OpenAI の Codex product surface の CLI です。`codex` TUI、`codex exec`、`codex resume`、`AGENTS.md`、sandbox、approval、MCP、plugins、skills など、Claude Code に近い操作体系を持ちながら、OpenAI の Codex app / cloud / IDE / GitHub Action と接続していく位置づけです。非対話実行をスクリプトや CI に入れるなら `codex exec` が軸になります。

OpenCode は、オープンソースでモデルプロバイダ非依存に近い agentic coding tool です。`opencode run`、`opencode session`、`opencode mcp`、`opencode agent`、`opencode serve`、`opencode web` など、CLI と server と TUI を組み合わせた構成が特徴です。Claude Code の代替というより、複数 provider を前提にした agent runtime として見ると理解しやすいです。

Aider は、AI pair programming in your terminal です。Claude Code / Codex CLI / OpenCode のような総合 agent platform ではなく、ファイルを chat に追加し、差分を作り、git commit し、必要なら `/undo` で戻すという作業感が強いです。小さく明確なコード編集、既存ファイルを対象にしたリファクタリング、diff を見ながら進める作業に向いています。

Antigravity CLI は、Google Antigravity 2.0 の一部として扱われる CLI です。ただし、本稿では公開 CLI command reference を確認できていないため、コマンド対応表では未確認を多く残しています。Antigravity 自体は agent-first development platform として、desktop app、SDK、CLI、multi-agent workflow を含む方向で報じられています。実運用で比較するなら、必ずインストール後の `--help` と Google の最新公式ドキュメントを確認してください。

## 参考リンク

- Claude Code CLI reference: <https://code.claude.com/docs/en/cli-reference>
- Claude Code memory / `CLAUDE.md`: <https://code.claude.com/docs/en/memory>
- Codex manual: <https://developers.openai.com/codex/codex-manual.md>
- OpenCode CLI docs: <https://opencode.ai/docs/cli>
- OpenCode intro: <https://opencode.ai/docs>
- OpenCode commands docs: <https://opencode.ai/docs/commands>
- OpenCode agents docs: <https://opencode.ai/docs/agents>
- Aider usage: <https://aider.chat/docs/usage.html>
- Aider in-chat commands: <https://aider.chat/docs/usage/commands.html>
- Aider configuration: <https://aider.chat/docs/config.html>
- Google Antigravity: <https://antigravity.google/>

## 著者

Toshiya Shimada — AI Consulting & Engineering  
[GitHub](https://github.com/toshiya-shimada)

## ライセンス

[![CC BY-ND 4.0](https://licensebuttons.net/l/by-nd/4.0/88x31.png)](https://creativecommons.org/licenses/by-nd/4.0/)

本リポジトリのコンテンツは [CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0/) で公開しています。クレジット表記のもと再配布は自由ですが、改変は許可されていません。
