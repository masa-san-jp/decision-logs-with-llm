# 添付資料01: 初回テスト用アプリ開発プロンプト

**概要:** Xcode環境でのビルドテスト用「深呼吸アプリ（Focus Breath）」を作成するためのLLMへの指示書。

## Prompt Content

# Role
あなたは世界トップクラスのiOSエンジニアであり、UI/UXデザイナーです。
SwiftUIを用いたモダンで美しいiPhoneアプリの開発を得意としています。

# Task
以下の要件に基づき、完全に動作するiOSアプリのソースコードを作成してください。

# App Idea: "Focus Breath" (深呼吸・集中タイマー)
ユーザーがリラックスしたり集中したりするための、シンプルな深呼吸サポートアプリ。
画面のアニメーションに合わせて呼吸を整える機能。

# Requirements
1. **Framework**: SwiftUI (iOS 15.0+)
2. **Architecture**: MVVM（ただし今回は単一ファイルに収めるため、構造体等で整理する）
3. **Design**:
   - AppleのHuman Interface Guidelinesに準拠したミニマルなデザイン。
   - アニメーションを効果的に使い、円（サークル）が呼吸に合わせて拡大・縮小する動きを入れる。
   - 色使いは目に優しいパステルカラーやグラデーションを使用。
4. **Features**:
   - **Start/Stopボタン**: タイマーの開始と停止。
   - **Text Guide**: "吸って (Inhale)" / "止めて (Hold)" / "吐いて (Exhale)" のテキストをアニメーションに合わせて切り替える。
   - **Haptic Feedback**: フェーズが変わるタイミングでiPhoneが振動する（UIImpactFeedbackGeneratorを使用）。

# Constraints (重要)
- 初心者がXcodeで試しやすいように、**すべてのコードを `ContentView.swift` という1つのファイルにまとめて記述してください。**
- `import` 文から書き始め、プレビュー用の `struct ContentView_Previews` まで含めてください。
- 外部ライブラリは使用せず、標準のSwiftUIのみを使用してください。
- エラー処理を含め、そのままコピペしてビルドが通るコードにしてください。

# Output Format
\`\`\`swift
// ここにSwiftのコードを出力
\`\`\`
