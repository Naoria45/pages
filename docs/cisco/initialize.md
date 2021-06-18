# cisco 機器の初期化方法

## ルータの場合

```
# erase startup-config
```


## Catalystスイッチの場合
`write erase`コマンドもしくは`erase startup-config`コマンドを実行します。

```
# erase startup-config
# delete flash:vlan.dat
# reload
```




# パスワードリカバリ
1. コンソールポートに接続します。
2. ルータの電源を投入し、すぐにBreak信号を送信する。
3. rommon のプロンプトになるため、`confreg 0x2142`コマンドでコンフィギュレーションレジスタを



## 参考文献
- [Catalyst スイッチを工場出荷時のデフォルトにリセット](
https://www.cisco.com/c/ja_jp/support/docs/switches/catalyst-2900-xl-series-switches/24328-156.html)
- [概要：Cisco 4000 シリーズサービス統合型ルータのトラブルシューティング](
https://www.cisco.com/c/ja_jp/td/docs/rt/branchrt/4451-xisr/tg/001/isr4400trbl/isr4400trbl02.html)

