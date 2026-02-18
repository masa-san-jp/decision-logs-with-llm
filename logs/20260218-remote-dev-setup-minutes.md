# 議事録：スマホ + 自宅Linuxマシンによるリモート開発環境構築

|項目|内容                                             |
|--|-----------------------------------------------|
|日付|2025年（Claude AIとの技術検討チャット）                     |
|目的|スマホから自宅Linuxマシンを操作し、Claude Codeでリモート開発できる環境の構築 |
|環境|自宅マシン: Linux / スマホ: iOS or Android（Terminusアプリ）|

-----

## 1. 検討の背景と目的

発端はRaspberry Pi上でClaude Codeを動かし、スマホのSSHアプリから接続してペアプログラミングを行うという構成の紹介。これに対し「ラズパイでなくても、自宅の既存Linuxマシンでも同じことができるか」という問いが起点となった。

目的は以下の2点に整理される。

- 外出先のスマホから自宅マシンにSSH接続し、Claude Codeをリモート操作できる環境を作ること
- その環境を安全に、かつ設定の手間なく運用できること

-----

## 2. 構成の意思決定

### 2-1. デバイス選定：Raspberry Pi → 自宅Linuxマシンへ

当初の参考情報はRaspberry Piを前提としていたが、自宅にLinuxマシンがある場合はそちらの方がスペックが高く、Claude Codeの動作が快適になるため、自宅Linuxマシンを採用することに決定した。WindowsはWSL2が必要、MacはSSH有効化のみで使える。

### 2-2. リモートアクセス方式：ポート開放 → Tailscale（VPN）へ

外出先から自宅に接続する際、グローバルIPアドレスが変動するという課題が浮上した。選択肢として以下を検討した。

- **ルーターのポート開放 + DDNS**：設定が複雑で、ポートが外部に直接露出するセキュリティリスクあり
- **Tailscale**：VPN不要、アカウント登録だけで仮想固定IPが発行される。ポートを外部に開放しないためセキュリティ面でも優位

→ **Tailscaleを採用**。個人利用の無料枠で十分対応可能。

### 2-3. セッション管理：tmuxを採用

スマホは電波が不安定なため、SSH接続が途切れるリスクがある。接続が切れても作業を継続するために、tmuxでClaude Codeのセッションを常駐させ、再接続後に `tmux attach` で即座に復帰できる構成を採用した。

### 2-4. セキュリティ対策の方針

セキュリティリスクを検討した結果、以下の3点を最低限の対策として実施することに決定した。

- SSH鍵認証への切り替え（パスワード認証を無効化）
- TailscaleアカウントへのMFA（多要素認証）設定
- OSのセキュリティアップデートの自動化（unattended-upgrades）

-----

## 3. 構築手順（完全版）

### 3-1. SSHサーバーの有効化

```bash
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

### 3-2. Tailscaleのインストール

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
# ブラウザでアカウント作成・ログインする
# 完了後、スマホにもTailscaleアプリを入れて同じアカウントでログイン
```

### 3-3. Node.js と Claude Code のインストール

```bash
# nvmでNode.jsをインストール
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install --lts
node -v  # バージョン確認

# Claude Codeをインストール
npm install -g @anthropic-ai/claude-code

# APIキーを設定（永続化）
echo 'export ANTHROPIC_API_KEY="your_api_key_here"' >> ~/.bashrc
source ~/.bashrc
```

### 3-4. tmuxセッションの管理

```bash
# 新しいセッションを作成して Claude Code を起動
tmux new -s claude
claude

# セッションから離脱（接続を切っても作業は残る）
Ctrl+B → D

# セッションに再接続
tmux attach -t claude
```

推奨tmux設定（`~/.tmux.conf`）：

```
# マウス操作を有効化（スマホ操作が楽になる）
set -g mouse on
```

### 3-5. SSH鍵認証の設定

Terminus（スマホアプリ）内でSSH鍵を生成し、公開鍵を自宅マシンに登録する。

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "コピーした公開鍵" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

パスワード認証を無効化：

```bash
sudo nano /etc/ssh/sshd_config
# 以下の行を変更
PasswordAuthentication no
PermitRootLogin no

sudo systemctl restart ssh
# ※必ず鍵認証でログインできることを確認してから無効化すること
```

### 3-6. TailscaleのMFA設定

1. https://login.tailscale.com にアクセス
1. Settings → Security → Two-factor authentication
1. 認証アプリ（Google Authenticator / Authy など）でQRコードをスキャン
1. 設定完了

### 3-7. セキュリティアップデートの自動化

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
# プロンプトで「Yes」を選択するだけで自動更新が有効になる
```

### 3-8. 日常的な接続手順

```bash
# スマホ側
1. Tailscaleアプリをオン
2. Terminusを起動 → 自宅マシン（TailscaleのIP）にSSH接続
3. tmux attach -t claude   # 前回の続きから即再開

# セッションがない場合
tmux new -s claude && claude
```

-----

## 4. 決定事項まとめ

|検討項目         |決定内容                        |
|-------------|----------------------------|
|接続ターゲット      |自宅Linuxマシン（ラズパイ不要）          |
|外部アクセス方法     |Tailscale（固定IP不要、ポート開放不要）   |
|セッション管理      |tmux常駐方式（接続断でも作業を保持）        |
|スマホSSHアプリ    |Termius（iOS/Android対応、鍵認証対応）|
|SSH認証方式      |鍵認証（パスワード認証は無効化）            |
|TailscaleのMFA|設定必須（アカウント乗っ取り防止）           |
|OSアップデート     |unattended-upgradesで自動化     |

-----

## 5. 備考・注意事項

- 鍵認証に切り替える際は、必ず鍵認証でのログインを確認してからパスワード認証を無効化すること。設定ミスで締め出されるリスクがある。
- APIキーは `.bashrc` に平文保存されている。マシンへの不正アクセスがあった場合は Anthropic ダッシュボードからキーを即時無効化すること。
- Tailscaleの無料プランは個人利用に十分な範囲をカバー。
- WindowsはWSL2経由でtmuxが使える。MacはシステムのリモートログインをオンにするだけでSSHが有効になる。