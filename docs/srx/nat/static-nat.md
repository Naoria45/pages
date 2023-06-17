# Static NAT

Static NAT はグローバル IP アドレスをローカル IP アドレスに 1:1 でマッピングします。
Static NAT ではネットワークのどちら側からも接続を開始することが可能です。
主にインターネットからローカルアドレスのサーバーへのアクセスや、サブネットを変換する際に使用されます。

![](https://publicmediastore.blob.core.windows.net/photo/srx/static-nat.png)

Junos では複数ルールにマッチする設定がされている場合、Static NAT は Destination NAT よりも優先され、戻りのパケットについても Source NAT より優先されます。

## 1:1 のアドレスの変換の場合
上図の PC からの `172.17.42.1` 宛ての通信を `10.0.0.2` に変換する場合の NAT です。
双方向に変換されるため、サーバー発の通信の場合、送信元 IP アドレスが変換されます。

### 設定サンプル
```
set security address-book global address SERVER1 10.0.0.2/32
set security nat static rule-set StaticNat-RS1 from zone untrust
set security nat static rule-set StaticNat-RS1 rule StaticNatR1 match destination-address 172.17.42.1/32
set security nat static rule-set StaticNat-RS1 rule StaticNatR1 then static-nat prefix 10.0.0.2/32
set security policies from-zone untrust to-zone trust policy server-1 match source-address any
set security policies from-zone untrust to-zone trust policy server-1 match destination-address SERVER1
set security policies from-zone untrust to-zone trust policy server-1 match application any
set security policies from-zone untrust to-zone trust policy server-1 then permit
set security policies from-zone trust to-zone untrust policy default-permit match source-address any
set security policies from-zone trust to-zone untrust policy default-permit match destination-address any
set security policies from-zone trust to-zone untrust policy default-permit match application any
set security policies from-zone trust to-zone untrust policy default-permit then permit
set security zones security-zone trust host-inbound-traffic system-services all
set security zones security-zone trust interfaces ge-0/0/1.0
set security zones security-zone untrust host-inbound-traffic system-services all
set security zones security-zone untrust interfaces ge-0/0/0.0
set interfaces ge-0/0/0 unit 0 family inet address 172.17.42.1/24
set interfaces ge-0/0/1 unit 0 family inet address 10.0.0.1/24
```

### 説明

インターフェースの IP アドレスやゾーンの設定は省略しています。

宛先ルールセットを作成します。
```
set security nat static rule-set StaticNat-RS1 from zone untrust
```

どのアドレス宛てのパケットに NAT を適用するかのルールを指定します。
```
set security nat static rule-set StaticNat-RS1 rule StaticNatR1 match destination-address 172.17.42.1/32
set security nat static rule-set StaticNat-RS1 rule StaticNatR1 then static-nat prefix 10.0.0.2/32
```

グローバルアドレスブックを作成し、10.0.0.2/32 のアドレスを SERVER1 という名前で登録します。
```
set security address-book global address SERVER1 10.0.0.2/32
```

セキュリティポリシーで SERVER1 宛ての通信を許可します。
```
set security policies from-zone untrust to-zone trust policy server-1 match source-address any
set security policies from-zone untrust to-zone trust policy server-1 match destination-address SERVER1
set security policies from-zone untrust to-zone trust policy server-1 match application any
set security policies from-zone untrust to-zone trust policy server-1 then permit
```

ここではサーバーから PC 宛ての接続も許可しています。
```
set security policies from-zone trust to-zone untrust policy default-permit match source-address any
set security policies from-zone trust to-zone untrust policy default-permit match destination-address any
set security policies from-zone trust to-zone untrust policy default-permit match application any
set security policies from-zone trust to-zone untrust policy default-permit then permit
```

インターフェースに設定されている IP 以外宛てのパケットの場合、ProxyArp の設定が必要です。
必要に応じて設定してください。
```
set security nat proxy-arp interface ge-0/0/0.0 address 172.17.42.100
``` 

## Static NAT の確認

セッションにポリシーが適用されているかの確認には `show security flow session nat` を使用します。

```
root> show security flow session nat
Session ID: 685, Policy name: server-1/6, Timeout: 2, Session State: Valid
  In: 172.17.42.2/1 --> 172.17.42.1/39;icmp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 1, Bytes: 60,
  Out: 10.0.0.2/39 --> 172.17.42.2/1;icmp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 1, Bytes: 60,
Total sessions: 1
```
In はルーターが受信したパケット、Out は戻りのパケットを表します。

ポリシーの確認は、`show security nat destination rule all` を使用します。
```
root> show security nat static rule all
Total static-nat rules: 1
Total referenced IPv4/IPv6 ip-prefixes: 2/0
Static NAT rule: StaticNatR1
  Rule set                   : StaticNat-RS1
  Rule Id                    : 1
  Rule position              : 1
  From zone                  : untrust
  Destination addresses      : 172.17.42.1
  Host addresses             : 10.0.0.2
  Netmask                    : 32
  Host routing-instance      : N/A
  Translation hits           : 29
    Successful sessions      : 29
  Number of sessions         : 0
```

## 参考資料
<a href="https://www.juniper.net/documentation/jp/ja/software/junos/nat/topics/topic-map/security-nat-static.html" target="_blank" rel="noopener noreferrer">https://www.juniper.net/documentation/jp/ja/software/junos/nat/topics/topic-map/security-nat-static.html</a>