# 基本設定

root（管理者）のパスワード、ホスト名、管理インターフェースの設定について記載します。

各階層で設定可能なコマンドは `?` で確認できます。
```
root@vSRX-01> configure
Entering configuration mode

[edit]
root@vSRX-01# set system ?
Possible completions:
> accounting           System accounting configuration
  allow-6pe-traceroute  Allow IPv4-mapped v6 address in tag icmp6 TTL expired packet
  allow-6vpe-traceroute-src-select  Select best src addr for icmp6 ttl expiry error in case 6vpe
  allow-l3vpn-traceroute-src-select  Select best src addr for icmp ttl expiry error in case l3vpn
  allow-v4mapped-packets  Allow processing for packets with V4 mapped address
+ apply-groups         Groups from which to inherit configuration data
(省略)
```

## root パスワード
root アカウントは Junos であらかじめ用意されているユーザーです。
デフォルトでスーパーユーザーに設定されており、すべてのパーミッションが与えられています。

工場出荷時や初期化後など、root のパスワードがせって視されていない場合、commit する前に root のパスワードを設定する必要があります。
以下のコマンドで設定します。
```
# set system root-authentication plain-text-password
New password:
Retype new password:
```
plain-text-password となっていますが、暗号化されて保存されます。

デフォルトでのパスワードの要件は 6 文字以上です。
条件を満たさない場合以下のようなエラーが表示されます。
```
error: minimum password length is 6
error: require change of case, digits or punctuation
```

デフォルトでは root ログインはコンソールポートからのみ許可されています。
必要に応じて ssh での root ログイン許可の設定を行います。
allow, deny, deny-password を設定できます。
```
# set system services ssh root-login allow
```

公開鍵を用いた SSH 接続を行う場合、以下のコマンドで公開鍵の登録を行います。

```
# set system root-authentication ssh-rsa KEY-STRING
```

ssh-rsa, ssh-ecdsa, ssh-ed25519 が設定可能です。

## ホスト名
デバイスのホスト名は次のコマンドで設定します。

```
# set system host-name HOSTNAME
```

設定されたホスト名は以下のように表示されます。

```
root# set system host-name vSRX-01

[edit]
root# commit
commit complete

[edit]
root@vSRX-01#
```

## 管理インターフェース
管理インターフェースは SSH や Telnet でデバイスにリモートでアクセスするためのインターフェースです。
多くの Juniper 製品では専用の管理ポートを搭載しており、OOBM (Out-of-Band Management) と呼ばれます。

通常のインターフェースの1つを管理インターフェースとして設定できるものもあります。
管理ポートを使用するには、管理インターフェースとして IP アドレスを設定します。

```
# set interface fxp0 unit 0 family inet address 172.16.11.22/24
```

管理インターフェース名はプラットフォームによって異なります。

| プラットフォーム | 管理ポート | 
| --- | --- | 
| EX シリーズ | me0, vme |
| QFX シリーズ | em0, vme |  
| MX シリーズ | fxp0 | 
| SRX シリーズ | fxp0, ge-0/0/0 |
| Junos Evolved </br>プラットフォーム | re0:mgmt-* など |  

詳細は [ドキュメント](https://www.juniper.net/documentation/jp/ja/software/junos/junos-getting-started/topics/concept/interfaces-understanding-management-ethernet-interfaces.html) をご確認ください。


## DNS サーバーの設定
DNS サーバーを設定しておくことでホスト名でほかの機器にアクセスすることができます。
DNS サーバーは複数設定することが可能です。
```
# set system name-server 172.17.1.53
# set system name-server 172.17.1.54
```

デバイス自体が所属するドメイン名を設定が推奨されます。
`example.com` の場合、以下のような設定を行います。

```
# set system domain-name example.com
```
ホスト名を解決する際に、`example.com`, `nw.example.com` の順でドメイン名を検索する設定を行います。
```
# set system domain-search [example.com nw.example.com]
```
ホスト名、IP アドレスが解決できることを確認します。
```
> show host 172.16.10.10
> show host juniper1
```

## 日付、時刻、タイムゾーンの設定

### NTP サーバーの設定
Junos の起動時に時刻を取得するには以下のコマンドを使用して NTP サーバーの IP アドレスを指定します。
```
# set system ntp boot-server 172.17.1.123
```
定期的に時刻同期を行う場合は、以下のコマンドで設定を行います。（複数設定可能）
```
# set system ntp server 172.16.1.123 prefer 
# set system ntp server 172.17.1.123
```
タイムゾーンを設定します。デフォルトは UTC です。
```
# set system time-zone Asia/Tokyo
```
指定できるタイムゾーンの一覧は ? で確認できます。
```
# set system time-zone ?
Possible completions:
  <time-zone>          Time zone name or POSIX-compliant time zone string
  Africa/Abidjan       
  Africa/Accra         
  Africa/Addis_Ababa   
  Africa/Algiers       
  Africa/Asmara
  ...
```
GTM (UTC) からのオフセット (-14 ~ +12) でタイムゾーンを指定することも可能です。
```
# set system time-zone GMT+9 
```

### 設定の確認
システムの時刻を確認するには Operational モードで `show system uptime` コマンドを実行します。
```
> show system uptime 
Current time: 2022-10-17 12:01:07 UTC
Time Source:  LOCAL CLOCK 
System booted: 2022-10-13 01:06:41 UTC (4d 10:54 ago)
Protocols started: 2022-10-13 01:10:39 UTC (4d 10:50 ago)
Last configured: 2022-10-17 10:53:32 UTC (01:07:35 ago) by root
12:01PM  up 4 days, 10:54, 2 users, load averages: 1.30, 1.56, 1.49
```

NTP サーバーの同期状態を確認するには、`show ntp associations` コマンドを実行します。アスタリスクが同期しているサーバを表します。
```
> show ntp associations 
   remote         refid           auth st t when poll reach   delay   offset  jitter
====================================================================================
*ntp-a2.nict.go.jp
                  .NICT.             -  1 -  665 1024  377    6.082   -8.141   0.153
```

### ローカルで時刻を設定する場合
通常は NTP サーバーを設定しますが、アクセスできないような場合は
YYYYMMDDhhmm.ss の形式で、オンボードのクロックを使用して時刻を設定します。
```
set date 202204290550.00
```

## rescue config について

rescue config はデフォルトの状態として残しておきたいコンフィグや緊急時に適用したいコンフィグを保存できます。
rescue config が設定されていない場合、機器の ALERM LED がオレンジ点灯します。
```
> request system configuration rescue save
```

確認コマンド
```
> show system configuration rescue
```

rescue config に戻したい場合は、以下のコマンドを実行します。
```
[edit]
# rollback rescue
```

また、機器に RESET CONFIG ボタンがある場合、ボタンを短く押すことで rescue config に戻すことが可能です。15 秒以上押すと初期化されてしまうため、注意してください。

rescue configを削除する場合、次のコマンドを実行します。

```
> request system configuration rescue delete
```


## インターフェースの設定
### 物理インターフェースの名前
Junos のインターフェースは ge-0/0/1 などで表記されます。
これは "識別子-スロット/PIC/ポート" の形式となっています。

識別子 : ge(gigabit ethernet) や xe(10Gbit ethernet) など。
スロット: 固定インターフェースの場合、通常 0 が割り当てられます。シャーシ型などハイエンドプラットフォームでは FPC (Flexible PIC Concentrator) 基盤のスロット番号が割り当てられます。
PIC : スロット内の PIC(物理インターフェースカード) の位置を示しています。
ポート : ポート番号を示します。

### 論理ユニット
物理インターフェースを論理的に複数に分割することができます。
この論理インターフェースをユニットと呼びます。各物理インターフェースに1つ以上の論理ユニットの設定が必要です。
論理ユニットは `ge-0/0/1.0` のように表記されます。

## IP の設定
IP アドレスの設定は次のようなコマンドになります。
```
# set interfaces ge-0/0/1 unit 0 family inet address 192.168.11.1/24
```
ge-0/0/1 : 物理インターフェースの名前。
unit 0 : 論理ユニット。
family inet : 論理インターフェースで使用するプロトコル。ipv4 は inet 
address : アドレス。

また、論理ユニットを省略表記にして以下のように表すこともできます。
```
# set interfaces ge-0/0/1.0 family inet address 192.168.11.1/24
```

Operational モードでの確認
```
> show configuration interfaces ge-0/0/1 
unit 0 {
    family inet {
        address 192.168.11.1/24;
    }
}
```

表示された結果は複数行で表示されます。set コマンドの形式で表示したい場合、`| display set` を使用します。
```
> show configuration interfaces ge-0/0/1 | display set 
set interfaces ge-0/0/1 unit 0 family inet address 192.168.11.1/24
```

## ライセンスインストール

SRX シリーズは追加のライセンスで機能を追加することができます。
ライセンス投入はファイルを転送してからファイルを指定して読み込む方法と、ライセンスファイル内の文字列をターミナルから入力する方法があります。

```
> request system license add terminal    
[Type ^D at a new line to end input,
 enter blank line between each license key]
```

### ライセンスの確認
現在使用されているライセンスは以下のコマンドで確認できます。
```
> show system license 
License usage: 
                                 Licenses     Licenses    Licenses    Expiry
  Feature name                       used    installed      needed 
  Virtual Appliance                     1            1           0    2022-12-12 01:10:42 UTC
  remote-access-ipsec-vpn-client        0            2           0    permanent
  remote-access-juniper-std             0            2           0    permanent
  VCPU Scale                            2            2           0    2022-12-12 01:10:42 UTC

Licenses installed: 
  License identifier: E20210617001
  License version: 4
  Order Type: trial
  Software Serial Number: 061520210001
  Customer ID: vSRX-JuniperEval
  Features:
    Virtual Appliance - Virtual Appliance
      date-based, 2022-10-13 01:10:42 UTC - 2022-12-12 01:10:42 UTC
    VCPU Scale       - VCPU number scale
      date-based, 2022-10-13 01:10:42 UTC - 2022-12-12 01:10:42 UTC

  License identifier: E20210617002
  License version: 4
  Order Type: trial
  Software Serial Number: 061520210002
  Customer ID: vSRX-JuniperEval
  Features:
    VCPU Scale       - VCPU number scale
      date-based, 2022-10-13 01:10:42 UTC - 2022-12-12 01:10:42 UTC
```

## ユーザー管理
### ログインバナーの設定
ログインバナーを設定すると、ユーザーが SSH などでログインする際にメッセージを表示することができます。
ログインメッセージは接続された際に表示されます。
```
set system login message "Welcome \n to Junos World.\n"
```

ログインアナウンスはログイン後に表示されます。
ホスト名や IP アドレス、警告文などを設定することが多いです。

```
set system login announcement "Login Success."
```

### ログインアカウント
Junos ではローカルユーザーとパスワード、RADIUS または TACACS+ 
プロトコルを使用したリモートサーバによる認証でユーザーの管理を行います。

### ユーザーの作成
```
# set system login user USERNAME class super-user 
```
パスワードの設定
```
# set system login user USERNAME authentication plain-text-password
```
フルネームや UID を指定して作成することも可能です。

### ログインクラス
ユーザーはパーミッションによってログインクラスが定義されています。

- super-user : すべての権限
- operator : パーミッションのクリア、ネットワーク、リセット、トレース、および参照
- read-only : パーミッションの参照
- unauthorized : パーミッションなし

デフォルトでは上記の4クラスが提供されていますが、さらに詳細な権限管理を行いたい場合、
カスタムログインクラスを作成することができます。

### リモート認証の設定

RADIUS サーバーの指定
```
# set system radius-server 172.16.18.12
# set system radius-server 172.16.18.12 port 1812
# set system radius-server 172.16.18.12 seccret Juniper!1
```

TACACS+ サーバーの設定

```
# set system tacplus-server 172.16.27.49
# set system tacplus-server 172.16.27.49 port 49
# set system tacplus-server 172.16.27.49 secret Juniper!1
```

認証を試行する順序を指定します。
```
set system authentication-order [radius tacplus password]
```
この例では RADIUS サーバと TACACS+ の両方が設定されており、
まず RADIUS サーバに認証を実施し、失敗した場合次に TACACS+ サーバー、
最後にローカルに設定されているユーザーアカウントを確認します。

## リモートアクセスの有効化
SSH や Telnet によるデバイスへのアクセスを有効化するには以下のコマンドを使用します。
Telnet や FTP はパスワードがテキストでやり取りされるため、SSH、SCP の使用が推奨されます。
```
# set system services ssh
# set system services ftp
# set system services telnet
```

## SNMP の設定
Junos は SNMPv1, V2c および v3 プロトコルをサポートしています。
SNMP エージェントはデフォルトでは無効です。

### コミュニティ設定

読み込み専用の SNMP コミュニティを作成する
```
set snmp community public
```
```
edit snmp community public
[edit groups common snmp community public]
set authorization read-only
set clients 192.168.1.0/24
set clients 0.0.0.0/0 restrict
```

読み込み/書き込みの SNMP コミュニティを作成する
```
[edit groups common snmp]
edit community private
set authorization read-write
set clients 192.168.1.15/24
set clients 0.0.0.0/0 restrict
```

### SNMP トラップの設定
SNMP トラップは SNMP エージェントからネットワーク管理システムなどに送信される
メッセージです。 

```
edit group common snmp
[edit group common snmp]
set trap-options source-address lo0
set trap-group managers version v2 targets 192.168.1.15
set trap-group managers version v2 targets 192.168.1.15 categories authentication
```

SNMP カテゴリー
| オプション | MIB | 説明 |
|- |- |- |
| authentication | 標準 MIB-II | 絵デバイスでの認証失敗 |
| chassis | Juniper 独自 | ハードウェア状態の通知 |
| configuration | Juniper 独自 | 設定モードの通知 |
| link | Juniper 独自 | インターフェースの変化 |
| rmon-alarm | Juniper 独自 | SNMP リモートモニタリングイベント |
| routing | Juniper 独自 | ルーティングプロトコル通知 | 
| startup | 標準 MIB-II | 再起動 |


## Syslog の設定

| 数値 | レベル | 説明 |
|- |- |- |
| - | なし | 無効 | 
| 0 |emergency | 機能停止を招くシステムパニックなど |
| 1 | alert | データベースの破損など、直ちに対処が必要な状況 |
| 2 | critical | 物理的なエラーなど重大な問題がある状況 |
| 3 | error | 致命的ではない一般的なエラー |
| 4 | warning | モニタリングの必要がある状況 |
| 5 | notice | エラーではないが、対処が必要な可能性がある状況 |
| 6 | info | イベント|
| 7 | any | ファシリティーからの全レベルのメッセージ |

### Syslog サーバーへのメッセージ転送

```
set system syslog host LOGSERVER any notice
```

facility をオーバーライドするには以下のコマンドを使用します。
```
# set facility-override local7
```

### ターミナルへのメッセージ表示
```
set system syslog user * any emergency
set system syslog console any error
```

## CLI の便利機能など

### set コマンド表示
show コマンドでは流し込み出来ない形式で表示されるため、コピペしたい場合などは以下のように `| display set` を使用します。
```
> show configuration interfaces ge-0/0/1 | display set
```

### | no-more 
デフォルトでは more で１ページごとに表示されます。一度に全部を表示したい場合、コマンドの後ろに `| no-more` をつけます。
`show configuration | display set | no-more` のような使い方をします。


### show | compare
Candidate Config と以前のコンフィグを比較する場合などに使用できます。
```
# show | compare rollback 2
```

```
# show | compare rollback 1    
[edit system]
+   login {
+       user naoki {
+           class super-user;
+       }
+   }
+   time-zone GMT+9;
[edit]
```

### | match 
出力から一致する行を抽出します。

```
# show | display set | match interface 
set interfaces ge-0/0/0 unit 0 family inet address 10.1.1.1/24
set interfaces ge-0/0/1 unit 0 family inet address 192.168.1.1/24
set interfaces fxp0 unit 0 family inet dhcp
set interfaces st0 unit 0 family inet address 172.16.1.1/24
```

### rename コマンド
インターフェースや vlan を変更したい場合、rename コマンドで変更が可能です。

```
# rename interfaces ge-0/0/0 to ge-1/0/0
```
### deactivate コマンド


### copy コマンド
```
# copy ge-0/0/1 to ge-0/0/2
```


