# Destination NAT

Destination NAT は宛先アドレスの変換を行います。
プライベート IP アドレスを使用しているサーバーをインターネットに公開する際などに使用される NAT です。

![](https://publicmediastore.blob.core.windows.net/photo/srx/destination-nat_v2.png)

## 単一アドレスの変換の場合
ポートの変換を行わない1対1の Destination NAT の設定です。
172.17.42.1 で受信したパケットを 10.0.0.2 のサーバーに転送します。

### 設定サンプル
```
set security address-book global address SERVER1 10.0.0.2/32
set security nat destination pool DstPool1 address 10.0.0.2/32
set security nat destination rule-set DstNat-RS1 from zone untrust
set security nat destination rule-set DstNat-RS1 rule DstNatR1 match destination-address 172.17.42.1/32
set security nat destination rule-set DstNat-RS1 rule DstNatR1 then destination-nat pool DstPool1
set security policies from-zone untrust to-zone trust policy SERVER1-ACCESS match source-address any
set security policies from-zone untrust to-zone trust policy SERVER1-ACCESS match destination-address SERVER1
set security policies from-zone untrust to-zone trust policy SERVER1-ACCESS match application any
set security policies from-zone untrust to-zone trust policy SERVER1-ACCESS then permit
set security zones security-zone trust host-inbound-traffic system-services ping
set security zones security-zone trust interfaces ge-0/0/0.0
set security zones security-zone untrust host-inbound-traffic system-services ping
set security zones security-zone untrust interfaces ge-0/0/1.0
set interfaces ge-0/0/0 unit 0 family inet address 10.0.0.1/24
set interfaces ge-0/0/1 unit 0 family inet address 172.17.42.1/24
```

### 説明
インターフェースの IP アドレスやゾーンの設定は省略しています。

Destination NAT の宛先プールを作成します。
```
set security nat destination pool DstPool1 address 10.0.0.2/32
```

宛先ルールセットを作成します。from には interface, routing-instance も指定できます。
```
set security nat destination rule-set DstNat-RS1 from zone untrust
```

どのアドレス宛てのパケットに NAT を適用するかのルールを指定します。
```
set security nat destination rule-set DstNat-RS1 rule DstNatR1 match destination-address 172.17.42.1/32
set security nat destination rule-set DstNat-RS1 rule DstNatR1 then destination-nat pool DstPool1
```

グローバルアドレスブックを作成し、10.0.0.2/32 のアドレスを SERVER1 という名前で登録します。
```
set security address-book global address SERVER1 10.0.0.2/32
```

セキュリティポリシーで SERVER1 宛ての通信を許可します。
```
set security policies from-zone untrust to-zone trust policy server-access match source-address any
set security policies from-zone untrust to-zone trust policy server-access match destination-address SERVER1
set security policies from-zone untrust to-zone trust policy server-access match application any
set security policies from-zone untrust to-zone trust policy server-access then permit
```

### プロキシ ARP について
自分のインターフェース IP 宛ての通信の場合は必要ありませんが、インターフェースに直接設定されていない IP アドレス宛てのパケットに NAT を行いたい場合、ARP に応答するためにプロキシ ARP を設定する必要があります。
例えば、ge-0/0/1 が 172.17.42.1/24 を持っており、172.17.42.10 宛てのパケットに Destination NAT を行う場合、次の設定が必要です。
```
set proxy-arp interface ge-0/0/1.0 address 172.17.42.10/32
```

## ポート番号に応じてアドレスの変換を行う場合（ポートフォワード）
受信パケットのポート番号に応じて、Destination NAT の宛先を変更する設定。
パブリック IP アドレスが1つしかない場合でも、ssh や web などサービスごとに別々のサーバーに接続を行うことが可能です。

![](https://publicmediastore.blob.core.windows.net/photo/srx/destination-nat-portforward.png)

### 設定サンプル
```
set security address-book global address SERVER1 10.0.0.2/32
set security address-book global address SERVER2 10.0.0.3/32
set security nat destination pool DstPool1 address 10.0.0.2/32
set security nat destination pool DstPool2 address 10.0.0.3/32
set security nat destination rule-set DstNat-RS1 from zone untrust
set security nat destination rule-set DstNat-RS1 rule DstNatR1 match destination-address 172.17.42.1/32
set security nat destination rule-set DstNat-RS1 rule DstNatR1 match destination-port 80
set security nat destination rule-set DstNat-RS1 rule DstNatR1 then destination-nat pool DstPool1
set security nat destination rule-set DstNat-RS1 rule DstNatR2 match destination-address 172.17.42.1/32
set security nat destination rule-set DstNat-RS1 rule DstNatR2 match destination-port 22
set security nat destination rule-set DstNat-RS1 rule DstNatR2 then destination-nat pool DstPool2
set security policies from-zone untrust to-zone trust policy SERVER1-ACCESS match source-address any
set security policies from-zone untrust to-zone trust policy SERVER1-ACCESS match destination-address SERVER1
set security policies from-zone untrust to-zone trust policy SERVER1-ACCESS match application junos-http
set security policies from-zone untrust to-zone trust policy SERVER1-ACCESS then permit
set security policies from-zone untrust to-zone trust policy SERVER2-ACCESS match source-address any
set security policies from-zone untrust to-zone trust policy SERVER2-ACCESS match destination-address SERVER2
set security policies from-zone untrust to-zone trust policy SERVER2-ACCESS match application junos-ssh
set security policies from-zone untrust to-zone trust policy SERVER2-ACCESS then permit
set security zones security-zone trust host-inbound-traffic system-services all
set security zones security-zone trust interfaces ge-0/0/0.0
set security zones security-zone untrust host-inbound-traffic system-services all
set security zones security-zone untrust interfaces ge-0/0/1.0
set interfaces ge-0/0/0 unit 0 family inet address 10.0.0.1/24
set interfaces ge-0/0/1 unit 0 family inet address 172.17.42.1/24
```

## Destination NAT の確認

セッションにポリシーが適用されているかの確認には `show security flow session nat` を使用します。

```
root> show security flow session nat    
Session ID: 172220, Policy name: SERVER1-ACCESS/4, Timeout: 298, Session State: Valid
  In: 172.17.42.2/41954 --> 172.17.42.1/80;tcp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 6, Bytes: 1145, 
  Out: 10.0.0.2/80 --> 172.17.42.2/41954;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 6, Bytes: 3881, 
Total sessions: 1
```

In はルーターが受信したパケット、Out は戻りのパケットを表します。

ポリシーの確認は、`show security nat destination rule all` を使用します。

```
root> show security nat destination rule all       
Total destination-nat rules: 1
Total referenced IPv4/IPv6 ip-prefixes: 1/0
Destination NAT rule: DstNatR1
  Rule set                   : DstNat-RS1
  Rule Id                    : 1
  Rule position              : 1
  From zone                  : untrust
    Destination addresses    : 172.17.42.1     - 172.17.42.1
  Action                     : DstPool1
  Translation hits           : 3
    Successful sessions      : 3
  Number of sessions         : 0
```

プールの設定確認には、`show security nat destination pool all` を使用します。
```
root> show security nat destination pool all       
Total destination-nat pools: 1
Pool name       : DstPool1
Pool id         : 1
Total address   : 1
Translation hits: 3
Address range                        Port 
       10.0.0.2 - 10.0.0.2              0
```


## 参考資料
<a href="https://www.juniper.net/documentation/jp/ja/software/junos/nat/topics/topic-map/security-nat-destination.html" target="_blank" rel="noopener noreferrer">https://www.juniper.net/documentation/jp/ja/software/junos/nat/topics/topic-map/security-nat-destination.html</a>