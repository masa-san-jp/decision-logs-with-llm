# 成果物01：ファイル収集スクリプト

対象リポジトリの `/logs` 配下から、過去7日間のmdファイルを収集するbashスクリプト。

ファイル名先頭・ディレクトリ名先頭の `yyyymmdd` を数値比較で絞り込む。

-----

```bash
START_DATE=$(date -d '7 days ago' +%Y%m%d)
END_DATE=$(date +%Y%m%d)

TARGET_FILES=()

for item in logs/*; do
  basename=$(basename "$item")
  prefix=$(echo "$basename" | grep -oP '^\d{8}')

  if [[ -z "$prefix" || "$prefix" -lt "$START_DATE" || "$prefix" -gt "$END_DATE" ]]; then
    continue
  fi

  if [[ -f "$item" && "$item" == *.md ]]; then
    # 20260218-aaa.md パターン
    TARGET_FILES+=("$item")
  elif [[ -d "$item" ]]; then
    # 20260223/ ディレクトリパターン → 中のmdを全部
    while IFS= read -r mdfile; do
      TARGET_FILES+=("$mdfile")
    done < <(find "$item" -name "*.md" | sort)
  fi
done

echo "対象ファイル数: ${#TARGET_FILES[@]}"
printf '%s\n' "${TARGET_FILES[@]}"
```

-----

## 対応ファイル構造

```
/logs
└ 20260218-aaa.md     ✅ 対象
└ 20260218-bbb.md     ✅ 対象
└ 20260219-ccc.md     ✅ 対象
└ 20260223/           ✅ ディレクトリ → 中のmdを全取得
  └ ddd.md
  └ eee.md
└ 20260101-old.md     ❌ 対象外（7日より前）
```