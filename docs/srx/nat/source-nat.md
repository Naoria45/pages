# Source NAT

Source NAT はルーターを通過するパケットの送信元アドレスの変換を行います。
SRX では 1 対 1 でアドレスを変換する NAT、1 対多、多対多のアドレス変換を行う pool NAT が利用可能です。

![](https://publicmediastore.blob.core.windows.net/photo/srx/source-nat_v2.png)

## エグレスインターフェース NAT
エグレス（egress）インターフェース NAT では、ルーターを通過したパケットの送信元アドレスを出力インターフェースのアドレスに変換します。
エグレスインターフェース NAT ではポートのアドレスも変換 (NAPT) されます。
変換に使用されるポートは 1024-65535 です。

### 設定サンプル
```
set security nat source rule-set SrcNat-RS1 from zone trust
set security nat source rule-set SrcNat-RS1 to zone untrust
set security nat source rule-set SrcNat-RS1 rule SrcNatR1 match source-address 0.0.0.0/0
set security nat source rule-set SrcNat-RS1 rule SrcNatR1 match destination-address 0.0.0.0/0
set security nat source rule-set SrcNat-RS1 rule SrcNatR1 then source-nat interface
set security policies from-zone trust to-zone untrust policy internet-access match source-address any
set security policies from-zone trust to-zone untrust policy internet-access match destination-address any
set security policies from-zone trust to-zone untrust policy internet-access match application any
set security policies from-zone trust to-zone untrust policy internet-access then permit
set security zones security-zone trust host-inbound-traffic system-services ping
set security zones security-zone trust interfaces ge-0/0/1.0
set security zones security-zone untrust host-inbound-traffic system-services ping
set security zones security-zone untrust interfaces ge-0/0/0.0
set interfaces ge-0/0/0 unit 0 family inet address 10.0.0.1/24
set interfaces ge-0/0/1 unit 0 family inet address 172.17.42.1/24
```

### 説明
インターフェースの IP アドレスやゾーンの設定は省略しています。

送信元ルールセット `SrcNat-RS1` を定義し、どのゾーンからどのゾーンの通信へ NAT を適用するか定義します。

```
set security nat source rule-set SrcNat-RS1 from zone trust
set security nat source rule-set SrcNat-RS1 to zone untrust
```

NAT の対象とするパケットの送信元、宛先を指定し、インターフェースのアドレスに変換するルール `SrcNatR1` を作成します。
```
set security nat source rule-set SrcNat-RS1 rule SrcNatR1 match source-address 0.0.0.0/0
set security nat source rule-set SrcNat-RS1 rule SrcNatR1 match destination-address 0.0.0.0/0
set security nat source rule-set SrcNat-RS1 rule SrcNatR1 then source-nat interface
```

trust から untrust ゾーンへの許可ポリシーを作成します。
```
set security policies from-zone trust to-zone untrust policy internet-access match source-address any
set security policies from-zone trust to-zone untrust policy internet-access match destination-address any
set security policies from-zone trust to-zone untrust policy internet-access match application any
set security policies from-zone trust to-zone untrust policy internet-access then permit
```

## 1対1 Pool NAT
インターフェース NAT 以外の NAT ではプールの定義が必要です。
プールを使用する Source NAT では、ポートアドレスの変換がデフォルトですが、行わない設定が可能です。
また、変換した送信元 IP アドレスに応答するため、プロキシ APR の設定が必要になります。

![](https://publicmediastore.blob.core.windows.net/photo/srx/source-nat-pool.png)

### 設定サンプル

```
set security nat source pool src-nat-pool-1 address 10.0.0.12/32
set security nat source rule-set RS1 from zone trust
set security nat source rule-set RS1 to zone untrust
set security nat source rule-set RS1 rule SNATR1 match source-address 0.0.0.0/0
set security nat source rule-set RS1 rule SNATR1 match destination-address 0.0.0.0/0
set security nat source rule-set RS1 rule SNATR1 then source-nat pool src-nat-pool-1
set security nat proxy-arp interface ge-0/0/0.0 address 10.0.0.12/32
set security policies from-zone trust to-zone untrust policy nat match source-address any
set security policies from-zone trust to-zone untrust policy nat match destination-address any
set security policies from-zone trust to-zone untrust policy nat match application any
set security policies from-zone trust to-zone untrust policy nat then permit
set security zones security-zone trust host-inbound-traffic system-services ping
set security zones security-zone trust interfaces ge-0/0/1.0
set security zones security-zone untrust host-inbound-traffic system-services ping
set security zones security-zone untrust interfaces ge-0/0/0.0
set interfaces ge-0/0/0 unit 0 family inet address 10.0.0.1/24
set interfaces ge-0/0/1 unit 0 family inet address 172.17.42.1/24
```
### 説明
インターフェース NAT との違いのみ記載します。

変換するアドレスのプール `src-nat-pool-1` を作成しています。
```
set security nat source pool src-nat-pool-1 address 10.0.0.12/32
```

NAT ルールにプールを使用する指定をします。
```
set security nat source rule-set RS1 rule SNATR1 then source-nat pool src-nat-pool-1
```

プロキシ ARP を指定します。
```
set security nat proxy-arp interface ge-0/0/0.0 address 10.0.0.12/32
```

(ポート変換を行わない場合)
利用できるアドレスが極端に少なくなるため通常は使用しませんが、ポート変換をしない場合にはプール設定に port no-tranlation を設定します。
```
set security nat source pool src-nat-pool-1 port no-translation
```

## Source NAT の確認

セッションにポリシーが適用されているかの確認には `show security flow session nat` を使用します。

```
root@vSRX> show security flow session nat    
Session ID: 2122, Policy name: nat/4, Timeout: 298, Session State: Valid
  In: 172.17.42.2/47612 --> 10.0.0.2/80;tcp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 7, Bytes: 1188, 
  Out: 10.0.0.2/80 --> 10.0.0.1/22745;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 6, Bytes: 3881, 
Total sessions: 1
```

In はルーターが受信した NAT 前のパケット、Out は戻りのパケットを表します。

ポリシーの確認は、`show security nat source rule all` を使用します。

```
root> show security nat source rule all
Total rules: 1
Total referenced IPv4/IPv6 ip-prefixes: 2/0
source NAT rule: SrcNatR1                                           # ルール名
  Rule set                   : SrcNat-RS1                           # ルールセット名
  Rule Id                    : 1
  Rule position              : 1
  From zone                  : trust                                # 送信元ゾーン 
  To zone                    : untrust                              # 宛先ゾーン
  Match
    Source addresses         : 0.0.0.0         - 255.255.255.255
    Destination addresses    : 0.0.0.0         - 255.255.255.255
  Action                        : interface
    Persistent NAT type         : N/A
    Persistent NAT mapping type : address-port-mapping              # アドレスポート変換
    Inactivity timeout          : 0
    Max session number          : 0
    Persistent NAT block session: disabled
  Translation hits           : 10                                   # ヒット数
    Successful sessions      : 10
  Number of sessions         : 0                                    # セッション数
```


## 参考資料
<a href="https://www.juniper.net/documentation/jp/ja/software/junos/nat/topics/topic-map/nat-security-source-and-source-pool.html" target="_blank" rel="noopener noreferrer">https://www.juniper.net/documentation/jp/ja/software/junos/nat/topics/topic-map/nat-security-source-and-source-pool.html</a>