# 別添資料B：GX10 + gpt-oss:20b 環境最適化ガイド

**対応フェーズ**: 第3フェーズ（ハードウェア環境の特定と最適化）

---

## 環境構成

| マシン | 用途 | スペック |
|--------|------|---------|
| M4 MacBook Pro 128GB | 開発・テスト | Apple Silicon, 128GB統合メモリ |
| ASUS Ascent GX10 | 本番実行 | GB10 Blackwell GPU, 128GB LPDDR5x, 1000 TOPS, DGX OS |
| メインモデル | gpt-oss:20b | 21Bパラメータ (3.6B active), MoE, MXFP4量子化 |

## gpt-oss:20bの優位点

- ネイティブfunction calling対応（エージェントのツール呼び出しに必須）
- chain-of-thought推論（reasoning effort: low/medium/high で制御可能）
- MoEで活性パラメータ3.6B → 推論が速い
- メモリ使用量16GB以下 → 128GBマシンで複数並列実行可能
- Apache 2.0ライセンス → 業務利用に制約なし

## GX10の優位点

- Blackwell GPUがMXFP4をハードウェアネイティブ処理（gpt-oss:20bとの相性極良）
- OllamaはNVIDIA AI Software Stackに同梱
- `OLLAMA_NUM_PARALLEL=4` で4エージェント並列可能
- ConnectX-7搭載 → 2台スタックで256GB共有メモリ（120bモデルも視野）

## 2台リモート連携アーキテクチャ

```
MacBook（開発・テスト）          GX10（本番実行）
┌──────────────────┐           ┌──────────────────┐
│ コード編集         │  SSH/API  │ Ollama Server    │
│ プロンプト調整      │ ────────▶ │ gpt-oss:20b      │
│ テスト実行         │           │ 並列実行          │
└──────────────────┘           └──────────────────┘
```

### 接続方法

```bash
# 方法1: 直接接続
# GX10側: OLLAMA_HOST=0.0.0.0 ollama serve
# MacBook側:
export OLLAMA_HOST=http://gx10.local:11434

# 方法2: SSHトンネル
ssh -L 11434:localhost:11434 user@gx10.local

# 方法3: GX10上で直接実行
ssh user@gx10.local && cd ~/agent-teams && python run_team.py
```

## GX10セットアップ手順

```bash
ollama pull gpt-oss:20b
export OLLAMA_NUM_PARALLEL=4
export OLLAMA_HOST=0.0.0.0
# systemdで永続化
sudo systemctl edit ollama
# Environment="OLLAMA_NUM_PARALLEL=4"
# Environment="OLLAMA_HOST=0.0.0.0"
sudo systemctl restart ollama
```

## 性能見込み

| シナリオ | エージェント数 | GX10推定時間 |
|---------|-------------|------------|
| コードレビュー | 4 | 5-15分 |
| バグ仮説調査 | 4 | 10-20分 |
| 機能開発 | 4 | 15-40分 |
| 技術リサーチ | 4 | 10-25分 |

## 発展：gpt-oss:120bへの昇格

GX10 2台をConnectX-7でスタックすれば256GB共有メモリ。リードに120b、ワーカーに20bのハイブリッド構成が可能。
