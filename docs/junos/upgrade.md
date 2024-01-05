# アップグレード

## アップグレードパスについて
Junos は一度に 3 つ以上のリリースにまたがるアップグレードとダウングレードはサポートされません。
例えば 20.4 から 21.1, 21.2, 21.3 へのアップグレード、または 20.3, 20.2, 20.1 へのダウングレードのみがサポート対象です。

EEOL(Extended End of Life) の対象となるリリースについては、前後 2 つの EEOL リリースにアップグレード、ダウングレードが可能です。
例えば 20.4 から 21.2, 21.4 へのアップグレード、20.2, 19.4 へのダウングレードがサポートされます。

><a href="https://www.juniper.net/documentation/us/en/software/junos/release-notes/21.4/junos-release-notes-21.4r1/topics/upgrade-downgrade/srx-upgrade-downgrade.html#id_l3t_3w3_44b" target="_blank" rel="noopener noreferrer">https://www.juniper.net/documentation/us/en/software/junos/release-notes/21.4/junos-release-notes-21.4r1/topics/upgrade-downgrade/srx-upgrade-downgrade.html#id_l3t_3w3_44b</a>

## 事前準備

ログの取得や確認項目については以下参照。

><a href="https://www.juniper.net/documentation/jp/ja/software/junos/junos-install-upgrade/topics/topic-map/prepare-sotware-install-and-upgrade.html" target="_blank" rel="noopener noreferrer">https://www.juniper.net/documentation/jp/ja/software/junos/junos-install-upgrade/topics/topic-map/prepare-sotware-install-and-upgrade.html</a>

取得するログの例
```
> show chassis alarms | no-more
> show system core-dumps | no-more
> show pfe statistics traffic | match drop
> show pfe statistics error | no-more
> show system processes extensive | no-more
> show system uptime no-forwarding | no-more
> show chassis fpc detail | no-more
> show chassis environment | no-more
> show chassis routing-engine | no-more

> show interfaces descriptions | match down | no-more
> show interfaces | match “Physical|rate” | no-more
> show bfd session | no-more
> show route summary | no-more
> show bgp summary | no-more | match Establish
```

コンフィグのバックアップを取得しておきます。
```
> show configuration | no-more
```

## ソフトウェアのダウンロード
以下のサポートサイトから必要なソフトウェアをダウンロードします。
<a href="https://support.juniper.net/support/downloads/" target="_blank" rel="noopener noreferrer">https://support.juniper.net/support/downloads/</a>

リリースノートや EoL までの期間などを考慮しバージョンを選定しましょう。

!!! tip
    機種ごとの最新の推奨バージョンは以下の KB に記載されています。
	<a href="https://supportportal.juniper.net/s/article/Junos-Software-Versions-Suggested-Releases-to-Consider-and-Evaluate?language=en_US" target="_blank" rel="noopener noreferrer">https://supportportal.juniper.net/s/article/Junos-Software-Versions-Suggested-Releases-to-Consider-and-Evaluate?language=en_US</a>
	

### ストレージ容量の確保
ファイルコピーに失敗する場合があります。ストレージの空き容量を確認し、不要なファイルを削除します。

ファイルの確認
```
> show system storage detail
```

クリーンアップ  
削除リストが表示されるので、yes を選択します。
```
request system storage cleanup 
```

特定のファイルの削除
```
> file delete /var/tmp/junos-xxx.tgz
```

### デバイスが直接インターネットにアクセスできる場合

file copy コマンドでダウンロードページに表示される URL から直接ダウンロードし保存します。

```
> file copy "URL" /var/tmp/filename

例 
> file copy "https://cdn.juniper.net/software/junos/22.4R2.8/junos-srxsme-22.4R2.8.tgz?SM~(略)~" /var/tmp/junos-22.4r2.tgz
```
ダブルクォーテーションがないとおかしくなります。
ファイル名も指定しないとやたら長くなるので、わかりやすい名前で保存するのをお勧めします。

### SCP や FTP で転送する場合
SCP サーバから転送
```
> scp junos-srxsme-22.4R2.8.tgz user@srx:/var/tmp/junos-srxsme-22.4R2.8.tgz
```

FTP サーバから取得
```
user@srx> ftp <ip address of local ftp server> (and login) 
user@srx> lcd /var/tmp
user@srx> bin 
user@srx> get junos-srxsme-22.4R2.8.tgz
user@srx> bye
```

### USB メモリで転送する場合
USB メモリでファイルの転送を行う場合、Shell でマウント操作が必要です。
```
> start shell
```

USB メモリのデバイス ID を確認
```
% ls /dev/da*
```

マウントポイントの作成
```
% mkdir /tmp/usb 
```

マウント
デバイス ID は通常 da0s1 や da1s1 などです。
```
% mount -t msdosfs /dev/<device id> /tmp/usb
```

ファイルを /var/tmp にコピー
```
% cp /tmp/usb/junos-srxsme-22.4R2.8.tgz /var/tmp
```

shell を抜ける
```
% exit
```

## アップグレード操作

request system software add コマンドでパッケージのインストールを行います。
```
> request system software add /var/tmp/installation-package
```

オプションで no-validate, no-copy, reboot をつけることが可能です。

<table>
	<tbody>
		<tr>
			<td>no-validate</td>
			<td>コンフィグに対する互換性の確認をスキップします。時間は短くなりますが、本番環境ではお勧めしません。</td>
		</tr>
		<tr>
			<td>no-copy</td>
			<td>デフォルトではインストール時にパッケージを /var/sw/pkg/ にコピーされますが、このオプションを指定するとコピーを作成しません。</td>
		</tr>
		<tr>
			<td>reboot</td>
			<td>インストールが完了後、自動で再起動を行います。</td>
		</tr>
	</tbody>
</table>

再起動後、show version にてバージョンを確認します。


## 参考文献
Junos® OS Software Installation and Upgrade Guide  
<a href="https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/index.html" target="_blank" rel="noopener noreferrer">https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/index.html</a>

Juniper SRX 日本語マニュアル 1. Junos OS インストール&アップグレード
<a href="https://www.juniper.net/content/dam/www/assets/additional-resources/jp/ja/junos-installation-upgrade.pdf" target="_blank" rel="noopener noreferrer">https://www.juniper.net/content/dam/www/assets/additional-resources/jp/ja/junos-installation-upgrade.pdf</a>

no-copy option  
<a href="https://supportportal.juniper.net/s/article/Junos-Example-Upgrade-Junos-OS-with-no-copy-option?language=en_US" target="_blank" rel="noopener noreferrer">https://supportportal.juniper.net/s/article/Junos-Example-Upgrade-Junos-OS-with-no-copy-option?language=en_US</a>

