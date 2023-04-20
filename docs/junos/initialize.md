# 初期化方法

Junos 製品の主な初期化の方法は次の3つです。

- 本体のリセットボタン
- load factory-default コマンド
- request system zeroize コマンド

!!! tip
    root パスワードがわからなくなった場合は、パスワードリセットが可能です。
    <a href="https://www.juniper.net/documentation/jp/ja/software/junos/user-access/topics/topic-map/recovering-root-password.html" target="_blank" rel="noopener noreferrer">https://www.juniper.net/documentation/jp/ja/software/junos/user-access/topics/topic-map/recovering-root-password.html</a>


## リセットボタン

SRX300 シリーズ、EX2300 などの機種ではリセットボタン搭載されています。
本体にリセットボタンがある場合、ボタンを押すことで工場出荷時の状態にリセットすることができます。
rescue config や rollback config、ログなどのファイルも削除されます。

リセットボタンは、短く押した場合、rescue config を呼び出して commit が実行されます。
初期化を行う場合、`15秒`以上押し続け、Status LED がアンバーに点灯することを確認します。

## load factory-default コマンド
load factory-default コマンドを使用することで、candidate config を工場出荷時の状態に戻すことが可能です。
このコマンドは Configuration mode で使用し、commit するために新しい root パスワードを設定する必要があります。

1. `load factory-default`
1. `set system root-authentication plain-text-password`
1. `commit`

```
root# load factory-default 
warning: activating factory configuration

root# set system root-authentication plain-text-password 
New password:
Retype new password:

root# commit 
configuration check succeeds
commit complete
```

## request system zeroize コマンド

request system zeroize コマンドを使用することで、デバイスを工場出荷時の状態に戻すことができます。
load factory-default コマンドと異なり、rescue config や rollback config、ログなどのファイルも削除されます。

このコマンドは Operational mode で実行します。

1. `request system zeroize` 

```
root> request system zeroize 
warning: System will be rebooted and may not boot without configuration
Erase all data, including configuration and log files? [yes,no] (no) yes 

warning: ipsec-key-management subsystem not running - not needed by configuration.
warning: zeroizing fpc0

root> 
root> Sep 28 00:44:52 init: multicast-snooping (PID 1312) stopped by signal 17
Sep 28 00:44:52 init: sflow-service (PID 1311) stopped by signal 17 ...
```

再起動後、出荷時の状態になります。


## 参考資料
KB15725 - [SRX] Getting Started - Factory Reset
<a href="https://supportportal.juniper.net/s/article/SRX-Getting-Started-Factory-Reset?language=en_US" target="_blank" rel="noopener noreferrer">https://supportportal.juniper.net/s/article/SRX-Getting-Started-Factory-Reset?language=en_US</a>

[*1] KB23787 - [SRX] Difference between "load factory-default" and "request system zeroize media"
<a href="https://supportportal.juniper.net/s/article/SRX-Difference-between-load-factory-default-and-request-system-zeroize-media?language=en_US" target="_blank" rel="noopener noreferrer">https://supportportal.juniper.net/s/article/SRX-Difference-between-load-factory-default-and-request-system-zeroize-media?language=en_US</a>
