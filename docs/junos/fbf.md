# Fillter-Based Forwarding (FBF) 

フィルターベースフォワーディング (FBF) は通常のルーティングテーブルの経路以外にパケットを転送したい場合に使用されます。ほかのベンダーでいうところの PBR のことです。
フィルタの条件には送信元 IP アドレス、宛先 IP アドレス、送信元ポート番号、宛先ポート番号、IP プロトコル、DSCP 値、TCP フラグなどが利用可能です。

FBF は L3 および L4 での処理となるため、URL や FQDN でのブレークアウトはできません。FQDN での制御が必要な場合、SRX では APBR を使用する必要があります。

### 設定の流れ

- ルーティングインスタンスの作成
- ルーティングインスタンスのインポート
- ファイアウォールフィルタの定義
- インターフェースへの適用

### 設定サンプル
送信元アドレスで next-hop を変えたい場合、以下のような設定になります。

Router1 のデフォルトルートが Router2 になっている場合に、送信元が PC1(10.1.0.2) のトラフィックを Router3 経由で転送する場合を想定します。
各ルータの経路は設定済みで疎通できるものとします。

![Image title](https://publicmediastore.blob.core.windows.net/photo/junos/fbf-source.png)

<pre><code>set interfaces ge-0/0/0 unit 0 family inet address 192.168.1.1/24
set interfaces ge-0/0/1 unit 0 family inet address 192.168.2.1/24
set interfaces ge-0/0/2 unit 0 family inet filter input <span style="color:red">FILTER-FBF</span>
set interfaces ge-0/0/2 unit 0 family inet address 10.1.0.1/24
set interfaces ge-0/0/3 unit 0 family inet address 10.2.0.1/24
set firewall family inet filter <span style="color:red">FILTER-FBF</span> term 1 from source-address 10.1.0.0/24
set firewall family inet filter <span style="color:red">FILTER-FBF</span> term 1 then routing-instance <span style="color:blue">INSTANCE-FBF</span>
set firewall family inet filter <span style="color:red">FILTER-FBF</span> term 2 then accept
set routing-instances <span style="color:blue">INSTANCE-FBF</span> instance-type forwarding
set routing-instances <span style="color:blue">INSTANCE-FBF</span> routing-options static route 0.0.0.0/0 next-hop 192.168.2.2
set routing-options interface-routes rib-group inet <span style="color:lime">FBF</span>
set routing-options static route 0.0.0.0/0 next-hop 192.168.1.2
set routing-options rib-groups <span style="color:lime">FBF</span> import-rib inet.0
set routing-options rib-groups <span style="color:lime">FBF</span> import-rib INSTANCE-FBF.inet.0
</code></pre>

### 説明
対象の next-hop 専用のルーティングインスタンスを作成し、経路をインポート、ファイアウォールフィルターで対象のパケットを指定し、インターフェースに適用する流れとなります。

forwarding 専用のルーティングインスタンスを作成し、next-hop を指定します。
```
set routing-instances INSTANCE-FBF instance-type forwarding
set routing-instances INSTANCE-FBF routing-options static route 0.0.0.0/0 next-hop 192.168.2.2
```

デフォルトのルーティングインスタンス (inet.0) に INSTANCE-FBF から RIB をインポートします。
```
set routing-options interface-routes rib-group inet FBF
set routing-options rib-groups FBF import-rib inet.0
set routing-options rib-groups FBF import-rib INSTANCE-FBF.inet.0
```

ファイアウォールフィルタを定義します。
ここでは送信元アドレスが 10.1.0.0/24 の場合、INSTANCE-FBF に転送するフィルタを作成しています。その他のパケットが暗黙の Deny にならないように term 2 で accept しています。
```
set firewall family inet filter FILTER-FBF term 1 from source-address 10.1.0.0/24
set firewall family inet filter FILTER-FBF term 1 then routing-instance INSTANCE-FBF
set firewall family inet filter FILTER-FBF term 2 then accept
```

パケットを受信するインターフェースに対してファイアウォールフィルタを適用します。
```
set interfaces ge-0/0/2 unit 0 family inet filter input FILTER-FBF
set interfaces ge-0/0/2 unit 0 family inet address 10.1.0.1/24
```

### 確認

PC から traceroute で確認します。

![Image title](https://publicmediastore.blob.core.windows.net/photo/junos/fbf-pc1-traceroute.png)

![Image title](https://publicmediastore.blob.core.windows.net/photo/junos/fbf-pc2-traceroute.png)


## 参考文献
<a href="https://www.juniper.net/documentation/jp/ja/software/junos/routing-policy/topics/example/filter-based-forwarding-example.html" target="_blank" rel="noopener noreferrer">https://www.juniper.net/documentation/jp/ja/software/junos/routing-policy/topics/example/filter-based-forwarding-example.html</a>


