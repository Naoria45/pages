# Junos のライフサイクル

Junos のリリースは通常、メジャーリリースが年に1回、マイナーリリースが4半期に1回程度行われています。  
※ Junos 23.2R1 より、半年に1回のリリースとなり、サポート期間が60ヵ月に延長されました。詳細はリリースノートをご確認ください。

Junos の各バージョンのサポート期間は以下に記載されています。
■ Junos のリリース日と EoS マイルストーン
<a href="https://support.juniper.net/support/eol/software/junos/" target="_blank" rel="noopener noreferrer">https://support.juniper.net/support/eol/software/junos/</a>

■ Junos EVO のリリース日と EoS マイルストーン
<a href="https://support.juniper.net/support/eol/software/junosevo/" target="_blank" rel="noopener noreferrer">https://support.juniper.net/support/eol/software/junosevo/</a>

### バージョンの確認方法
`show version` コマンドで確認できます。 ※ バージョンだけを確認したい場合 and haiku は不要です。 
```
naoki@SRX300> show version and haiku
Hostname: SRX300
Model: srx300
Junos: 22.1R1.10
JUNOS Software Release [22.1R1.10]


        Sleet, ice, but no snow
        Take the day off and Enjoy
        Winter in The South
```

### バージョンの表記について
Junos のバージョンは `m.nZb.s` のように表記されます。
ソフトウェアリリース番号 22.3R1.11 を例として示します。

- m は、製品のメジャーリリース番号(例:22)です。
- n は、製品のマイナーリリース番号(例:3)です。
- Z は、ソフトウェアリリースのタイプです。いくつかタイプがありますが、FRS(First Revenue Ship) またはメンテナンスリリースを表す R が一般的です。
- b は、ビルド番号(例:1)で、Z と合わせて R1 は FRS バージョン、R2 以降はメンテナンスリリースを示します。
- s は、製品のスピン番号(例:11)です。

詳細は以下に記載されています。
<a href="https://www.juniper.net/documentation/jp/ja/software/junos/junos-install-upgrade/topics/topic-map/software-install-and-upgrade-overview.html#id-junos-os-installation-package-names" target="_blank" rel="noopener noreferrer">https://www.juniper.net/documentation/jp/ja/software/junos/junos-install-upgrade/topics/topic-map/software-install-and-upgrade-overview.html#id-junos-os-installation-package-names</a>

### 推奨バージョン
最新の推奨バージョンについては以下の KB に記載されています。
<a href="https://supportportal.juniper.net/s/article/Junos-Software-Versions-Suggested-Releases-to-Consider-and-Evaluate" target="_blank" rel="noopener noreferrer">https://supportportal.juniper.net/s/article/Junos-Software-Versions-Suggested-Releases-to-Consider-and-Evaluate</a>

## サポート終了（EOL; End of Life）について
サポートの終了は、Junos のサポート終了と、製品自体のサポート終了があります。

Junos については、上記のリリース日と EoS マイルストーンのページにあるように、リリース時点で OS のサポート終了日が記載されます。
おおむねリリースから2年でソフトウェアの修正や機能追加が行われなくなる End of Engineering (EoE)、EoE から6ヵ月後が QA を含むサポート終了日に設定されています。

製品自体が終息する場合、次のようなアナウンスが行われます。
![](https://publicmediastore.blob.core.windows.net/photo/junos/eol-sample.png)
製品の場合は販売終了のアナウンスが案内された段階で、EOS の期日が記載されます。

製品ごとの EOL 案内ページ
<a href="https://support.juniper.net/support/eol/" target="_blank" rel="noopener noreferrer">https://support.juniper.net/support/eol/</a>

EOL についてのポリシーの詳細は以下の PDF に記載されています。
<a href="https://support.juniper.net/support/pdf/eol/990833.pdf" target="_blank" rel="noopener noreferrer">https://support.juniper.net/support/pdf/eol/990833.pdf</a>

