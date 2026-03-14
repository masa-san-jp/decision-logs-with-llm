# decision-logs-with-llm
## about
- いろいろなLLMと会話していると、誰と何を話したのか忘れてしまうので、議事録を残していくことにしました。
- 議事録をプロンプトとして「この話の続きなんだけど…」とやると、あたらしいスレッドを立てたり、他のLLMと議論の続きができるので重宝しています。

## prompt

```
意思決定のプロセスを記録しておくために、このチャットの会話の議事録を作って、マークダウン形式でmdファイルに書き出して。
最終成果物は別のmdファイルで書き出して。
```

---

## Weekly Blog Automation

A GitHub Actions workflow (`weekly-blog.yml`) runs every Friday at 09:00 UTC (18:00 JST) and
generates blog-style posts in both Japanese and English from recent entries in `logs/`.
Generated posts are committed directly to `blog/` as `YYYYMMDD-weekly.md` and `YYYYMMDD-weekly-en.md`.

### How it works

1. The workflow scans `logs/` for directories or files whose names start with a `yyyymmdd` date token
   (e.g. `20260310-grant-agent`).
2. Files whose date falls within the past 7 days are collected.
3. If no dated files are found, the workflow exits gracefully without generating a post.
4. Sensitive values (API keys, tokens, passwords) in the logs are automatically redacted before
   being sent to the LLM.
5. A persona-driven prompt is sent to the configured LLM backend, and two posts are generated:
   a Japanese version and an English version.
6. Both files are committed to the repository automatically.

### Workflow inputs

| Input | Default | Description |
|---|---|---|
| `backend` | `github_models` | LLM backend: `github_models` or `anthropic` |
| `model` | — | Override model name (default: `claude-sonnet-4` / `claude-opus-4-6`) |
| `blog_days` | `7` | Days to look back |
| `blog_date` | today UTC | Override output date (`YYYY-MM-DD`) |

### Secrets

| Secret | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Required when `backend=anthropic` |
| `GITHUB_TOKEN` | Provided automatically by Actions for `github_models` mode |

### Manual workflow dispatch

Go to **Actions → Weekly Blog Generator → Run workflow** in the GitHub UI.
You can override `backend`, `model`, `blog_days`, and `blog_date` inputs before running.

### Scheduling

The workflow is scheduled via cron (`0 9 * * 5` – every Friday 09:00 UTC / 18:00 JST).
To change the schedule, edit `.github/workflows/weekly-blog.yml`.

### Running tests

> Note: The tests cover `scripts/generate_weekly_blog.py`, a standalone Python script
> that is not used by the deployed workflow but may be useful for local experimentation.

```bash
pip install pytest
pytest scripts/tests/
```
