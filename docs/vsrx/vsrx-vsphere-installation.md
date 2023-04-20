# vSRX インストール for vSphere

## ダウンロード
Juniper のサポートサイトからイメージファイルをダウンロードします。
<a href="https://support.juniper.net/support/downloads/" target="_blank" rel="noopener noreferrer">https://support.juniper.net/support/downloads/</a>

![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-download.png)

バージョンやプラットフォームを選択します。
※ 特に理由がなければ vSRX3.0 をお勧めします。トライアルライセンスが付与されています。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-download2.png)

EULA に同意するとダウンロード用の URL が表示されます。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-download3.png)

## インストール
ここでは vSphere 7.0 を使用しています。

OVF テンプレートのデプロイを選択します。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy1.png)

ローカルファイルのアップロードで、先の手順で入手したファイルをアップロードします。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy2.png)

仮想マシン名を設定します。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy3.png)

コンピューティングリソースを選択します。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy4.png)

「次へ」を選択します。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy5.png)

使用許諾契約書を確認し、チェックを入れます。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy6.png)

使用するストレージを選択します。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy7.png)

ネットワークを選択します。ここで設定したネットワークが管理用の fxp0 になります。
デプロイが完了すると、デフォルトで 3 NIC が設定されます。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy8.png)

設定を確認し、「完了」を選択します。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy9.png)

デプロイが完了すると、起動前に仮想マシン設定の編集を行います。
デフォルトで 3 つのネットワークアダプタが設定されます。必要に応じて追加します。
ネットワークアダプタのタイプは統一する必要があります。SR-IOV など特殊な場合を除いて、VMXNET3 がおすすめです。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy10.png)

また、CPU はハードウェア仮想化の設定を行うのが推奨となっています。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy11.png)

起動するとログインプロンプトになります。root、パスワードなしでログインできます。
![](https://publicmediastore.blob.core.windows.net/photo/srx/vsrx-deploy12.png)

以上で vSRX のデプロイは完了です。

## 参考資料
<a href="https://www.juniper.net/documentation/us/en/software/vsrx/vsrx-consolidated-deployment-guide/vsrx-vmware/topics/concept/security-vsrx-vmware-overview.html" target="_blank" rel="noopener noreferrer">https://www.juniper.net/documentation/us/en/software/vsrx/vsrx-consolidated-deployment-guide/vsrx-vmware/topics/concept/security-vsrx-vmware-overview.html</a>