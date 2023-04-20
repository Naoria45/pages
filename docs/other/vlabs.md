# Juniper vLabs の使い方

## Juniper vLabs とは
Juniper が提供するクラウド上でのテスト環境です。
いくつかのトポロジがあらかじめ用意されており、vMX や vSRX、Apstra や Paragon などの製品を直接操作、体験することができます。

<a href="https://jlabs.juniper.net/vlabs/" target="_blank" rel="noopener noreferrer">https://jlabs.juniper.net/vlabs/</a>
<a href="https://jlabs.juniper.net/assets/pdf/vlabs/vlabs-ug.pdf" target="_blank" rel="noopener noreferrer">ユーザーガイド（英語 PDF）</a> 

![](https://publicmediastore.blob.core.windows.net/photo/vlabs/vlab_overview.jpg)

## 登録方法
Juniper vLabs の利用には Juniper アカウントが必要です。
公式の登録の手順は <a href="https://jlabs.juniper.net/vlabs/sign-up.page" target="_blank" rel="noopener noreferrer">こちら（英語）</a> をご参照ください。
<a href="https://jlabs.juniper.net/vlabs/" target="_blank" rel="noopener noreferrer">https://jlabs.juniper.net/vlabs/</a> にアクセスし、Sign in をクリックします。
ログインページが表示されるので、アカウントを持っていない人は「サインインについてヘルプが必要ですか？」をクリックし、「Create a new account」を選択します。

![](https://publicmediastore.blob.core.windows.net/photo/vlabs/account_signup.jpg)

ユーザー登録の画面になるので、必要事項を記入します。
権限の申請では「Access to EngNet, vLabs other JCL Tools」を選択してください。
Guest User Access ではコミュニティの閲覧などは可能ですが、ラボの利用ができません。

![](https://publicmediastore.blob.core.windows.net/photo/vlabs/account_registration.jpg)

次に会社名などを入力し、Submit をクリックします。
コンプライアンスチェックなどのため、有効になるまで 最大で 24 時間程度 かかる場合があります。

!!! warning
    フリーメール（Gmail や Hotmail）などはラボの利用ができません。会社等のメールアドレスでサインアップを行ってください。

### 使用方法
### 予約
ログインすると利用できるラボトポロジの一覧が表示されます。
![](https://publicmediastore.blob.core.windows.net/photo/vlabs/vlab_topolilst.jpg)

利用したいラボを選択し、「Launch」を選択します。
トポロジが表示されます。この時点ではまだ動いていないので、右上の「RESERVE」からラボを予約します。 
左上の「Instractions」にラボの説明があります。ラボの種類によっては解説がある場合もあります。

![](https://publicmediastore.blob.core.windows.net/photo/vlabs/vlab_topology_sample.jpg)

デフォルトで 3 時間利用可能です。最大 6 時間まで予約可能です。
構成やバージョンは固定です。

![](https://publicmediastore.blob.core.windows.net/photo/vlabs/vlab_reserve.jpg)

Reserve を押してから、環境の準備が始まり、約15~20分程度で利用可能な状態になります。
右上の SETUP にマウスをかざすと完了までの時間が表示されます。

![](https://publicmediastore.blob.core.windows.net/photo/vlabs/vlab_provisioning.jpg)

利用可能になると表示が Active になります。

### 画面説明

機器が正常に起動している場合、機器アイコンの右下に緑色の〇が表示されます。
機器をクリックし、▽を押すと接続方法が表示されます。

![](https://publicmediastore.blob.core.windows.net/photo/vlabs/vlab_ssh.jpg)

SSH を選択した場合、新しいタブでコンソール画面が表示されます。

![](https://publicmediastore.blob.core.windows.net/photo/vlabs/vlab_console.jpg)

機器によっては HTTP(S) などでのアクセスも可能です。
準備は以上です。ラボ環境をご自由にお楽しみください。

### ラボの終了

時間が来れば自動的にラボは終了しますが、リソースに限りがあるため、もし時間が余ったらラボを終了しましょう。

![](https://publicmediastore.blob.core.windows.net/photo/vlabs/vlab_end.jpg)

End をクリックし、確認ボタンを押して終了です。

## Tips
### 外部アクセスの許可
ブラウザではなく Teraterm などのローカルから操作したい場合、外部アクセスの許可が必要になります。
[Command] > [Add Allowed Network Prefixes] より接続元のグローバルアドレスを許可してください。

![](https://publicmediastore.blob.core.windows.net/photo/vlabs/vlab_external_access.jpg)

次に、機器の右下の緑の〇にマウスをかざすと、接続用の IP アドレスとポート番号が表示されます。残念ながらコピペできないので、Teraterm などに入力して接続します。
SSH の認証情報は予約時のメールで通知されます。基本的には jcluser / Juniper!1 です。

