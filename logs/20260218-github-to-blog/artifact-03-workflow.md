# 成果物03：GitHub Actions ワークフロー

配置場所：`.github/workflows/weekly-blog.yml`

事前準備として、リポジトリの **Settings → Secrets → Actions** に `ANTHROPIC_API_KEY` を登録すること。

-----

```yaml
name: Weekly Blog Generator

on:
  schedule:
    - cron: '0 9 * * 1'  # 毎週月曜 9:00 UTC（日本時間18:00）
  workflow_dispatch:      # 手動実行も可能

jobs:
  generate-blog:
    runs-on: ubuntu-latest

    steps:
      - name: リポジトリをチェックアウト
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: 対象ログファイルを収集
        id: collect
        run: |
          START_DATE=$(date -d '7 days ago' +%Y%m%d)
          END_DATE=$(date +%Y%m%d)
          echo "対象期間: $START_DATE 〜 $END_DATE"

          TARGET_FILES=()

          for item in logs/*; do
            basename=$(basename "$item")
            prefix=$(echo "$basename" | grep -oP '^\d{8}')

            if [[ -z "$prefix" || "$prefix" -lt "$START_DATE" || "$prefix" -gt "$END_DATE" ]]; then
              continue
            fi

            if [[ -f "$item" && "$item" == *.md ]]; then
              TARGET_FILES+=("$item")
            elif [[ -d "$item" ]]; then
              while IFS= read -r mdfile; do
                TARGET_FILES+=("$mdfile")
              done < <(find "$item" -name "*.md" | sort)
            fi
          done

          if [ ${#TARGET_FILES[@]} -eq 0 ]; then
            echo "今週の対象ファイルがありません。終了します。"
            echo "has_files=false" >> $GITHUB_OUTPUT
            exit 0
          fi

          echo "対象ファイル数: ${#TARGET_FILES[@]}"
          echo "has_files=true" >> $GITHUB_OUTPUT
          echo "start_date=$START_DATE" >> $GITHUB_OUTPUT
          echo "end_date=$END_DATE" >> $GITHUB_OUTPUT

          LOGS=""
          for f in "${TARGET_FILES[@]}"; do
            LOGS="${LOGS}\n\n---\n### ファイル: $f\n\n$(cat "$f")"
          done

          echo -e "$LOGS" > /tmp/weekly_logs.txt
          echo "logs_file=/tmp/weekly_logs.txt" >> $GITHUB_OUTPUT

      - name: Claude APIで日本語ブログ記事を生成
        id: generate
        if: steps.collect.outputs.has_files == 'true'
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          START_DATE=${{ steps.collect.outputs.start_date }}
          END_DATE=${{ steps.collect.outputs.end_date }}
          LOGS=$(cat /tmp/weekly_logs.txt)

          PROMPT="あなたは、私（masa）の週次ブログ記事を代わりに書くライターです。

          ## 私について
          - LLMとの議論ログを日常的に残している
          - 思考が好きで、AIとの対話を通じて何かを掴もうとしている
          - ブラックユーモアがあり、自分自身にもAIにも冷静にツッコめる
          - 口癖は「〜だよね」「〜じゃない？」だが、ブログではちょっとだけ丁寧にする

          ## 記事の構成（この順番で、2000文字以上）
          1. **導入** - 今週どんなことを考えていたか、何が気になっていたか
          2. **今週の議論ハイライト** - 特に印象的だった議論を1〜2個ピックアップ
          3. **気づき・学び** - 議論を通じて得たもの。ただし悟ったふりはしない
          4. **各議論の要約とツッコミ** - 各ログを1〜2段落で要約し、末尾にmasaとしてブラックユーモアで冷静にツッコむ。例：「結局AIに丸め込まれた」「この議論、30分かけて最初に戻った」など
          5. **来週やってみたいこと** - 次のアクションや問い

          ## 文体のルール
          - ですます調だが、硬くなりすぎない
          - AIっぽい美文・まとめ感は出さない
          - 迷いや矛盾があればそのまま残す
          - ブラックユーモアを要所に入れる（やりすぎない）
          - 最後に綺麗にまとめない（人生はそういうもの）

          ## 入力データ
          以下は今週（${START_DATE}〜${END_DATE}）のLLMとの議論ログです：

          ${LOGS}

          上記をもとに、noteに投稿するブログ記事を日本語で書いてください。
          タイトルも考えてください（キャッチーだけど煽りすぎないやつ）。
          出力はmarkdown形式で、先頭行をタイトル（# タイトル）にしてください。"

          RESPONSE=$(curl -s https://api.anthropic.com/v1/messages \
            -H "x-api-key: $ANTHROPIC_API_KEY" \
            -H "anthropic-version: 2023-06-01" \
            -H "content-type: application/json" \
            -d "$(jq -n \
              --arg prompt "$PROMPT" \
              '{
                model: "claude-opus-4-6",
                max_tokens: 4096,
                messages: [{"role": "user", "content": $prompt}]
              }'
            )")

          BLOG=$(echo "$RESPONSE" | jq -r '.content[0].text')

          if [[ -z "$BLOG" || "$BLOG" == "null" ]]; then
            echo "APIレスポンスの取得に失敗しました"
            echo "$RESPONSE"
            exit 1
          fi

          FILENAME="blog/$(date +%Y%m%d)-weekly.md"
          mkdir -p blog
          echo "$BLOG" > "$FILENAME"
          echo "blog_file=$FILENAME" >> $GITHUB_OUTPUT
          echo "生成完了: $FILENAME"

      - name: Claude APIで英語ブログ記事を生成
        id: generate_en
        if: steps.collect.outputs.has_files == 'true'
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          START_DATE=${{ steps.collect.outputs.start_date }}
          END_DATE=${{ steps.collect.outputs.end_date }}
          LOGS=$(cat /tmp/weekly_logs.txt)

          PROMPT="You are a writer who creates weekly blog posts on behalf of masa.

          ## About masa
          - Regularly logs discussions with LLMs
          - Loves thinking deeply, and uses AI conversations to try to grasp something
          - Has a dry, black sense of humor — can calmly poke fun at both himself and the AI
          - Casual in conversation, but slightly more polished in writing

          ## Article structure (in this order, 600+ words)
          1. **Intro** - What masa was thinking about this week, what caught his attention
          2. **Highlight of the week** - 1–2 particularly memorable discussions
          3. **Insights & learnings** - What he got out of the discussions. No fake enlightenment.
          4. **Summary + commentary per discussion** - 1–2 paragraphs summarizing each log, ending with a dry, black-humor remark from masa's perspective. e.g. 'Spent 30 minutes just to arrive back at square one.' or 'Turns out I was just getting manipulated by the AI the whole time.'
          5. **What to try next week** - Next actions or questions

          ## Writing style rules
          - Polite but not stiff
          - No AI-sounding polished prose or tidy wrap-ups
          - Leave in doubts and contradictions as-is
          - Sprinkle in black humor (don't overdo it)
          - Don't end neatly — life isn't like that

          ## Input data
          Below are this week's (${START_DATE}–${END_DATE}) LLM discussion logs:

          ${LOGS}

          Based on the above, write a blog post in English for posting on note or dev.to.
          Also come up with a title (catchy but not clickbaity).
          Output in markdown format, with the title as the first line (# Title)."

          RESPONSE=$(curl -s https://api.anthropic.com/v1/messages \
            -H "x-api-key: $ANTHROPIC_API_KEY" \
            -H "anthropic-version: 2023-06-01" \
            -H "content-type: application/json" \
            -d "$(jq -n \
              --arg prompt "$PROMPT" \
              '{
                model: "claude-opus-4-6",
                max_tokens: 4096,
                messages: [{"role": "user", "content": $prompt}]
              }'
            )")

          BLOG_EN=$(echo "$RESPONSE" | jq -r '.content[0].text')

          if [[ -z "$BLOG_EN" || "$BLOG_EN" == "null" ]]; then
            echo "英語APIレスポンスの取得に失敗しました"
            echo "$RESPONSE"
            exit 1
          fi

          FILENAME_EN="blog/$(date +%Y%m%d)-weekly-en.md"
          echo "$BLOG_EN" > "$FILENAME_EN"
          echo "blog_file_en=$FILENAME_EN" >> $GITHUB_OUTPUT
          echo "英語版生成完了: $FILENAME_EN"

      - name: 生成したブログをリポジトリにコミット
        if: steps.collect.outputs.has_files == 'true'
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add blog/
          git commit -m "📝 週次ブログ自動生成 (${{ steps.collect.outputs.start_date }}〜${{ steps.collect.outputs.end_date }})"
          git push
```