### 4．添付資料3：接続テスト手順書
・-markdown
#添付資料03：IM Studio リモート接続テスト手順書
こすり開発に着手する前に、iPhoneと自宅Macの疎通確認を行うための手順。
##前提条件
- Macに「IM studio」と「railscale」がインストールされていること。
- iPhoneに
「railscale」アプリがインストールされていること。
- 両端末で同じrai1scaleアカウントにログインしていること。
##手順1: Mac側（サーバー）の設定
開く任意のモデル （Liama3）をロードする。
＊*LM Studioを起動＊*し、開発者モード（＜->アイコン）を
**Server Options＊＊を確認・設定する。
“port
**1234＊*（デフォルト）
'Enable Network (Serve on Local Network): **ON**
-CORS：必要に応じてONにする。
4. ＊*［Start Server］**ボタンをクリックする。
'Server listening on http: //0.0.0.0:1234" k
表示されればOK。
＃#手順2:IPアドレスの確認
1. Macまたはiphoneのrailscaleアプリを開く。
2.Macのホスト名横に表示されているIPアドレス
（例：100.x・Y・2）
をコピーする。
＃#手順3：iphone側（クライアント）でのテスト
###A．ブラウザでの生存確認
iPhoneのSafariを開く。
2. アドレスバーに以下を入力してアクセス。
“http://1手順2でコピーしたIP］：1234/v1/models
3.★*判定：**
-JSONテキスト（ ｛"data"：【・・・1｝ ）が表示されれば＊＊成功
**。
###B・
チャットアプリでの動作確認
無料アプリ「Chatbox AI」等をインストール。
2. 設定画面で以下を入力。
- **Provider：**OpenAI API
-**API Key：**1m-studio
-（任意文字列）
- **API HoSt：**
'http://［手順2でコピーしたIP1:1234
3． チャット画面でメッセージを送し、返答があれば＊*疎通完了**