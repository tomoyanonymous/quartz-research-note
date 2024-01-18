#server

1万円で買える2.5Gbe x8、10Gbe(SFP) x1のL2スマートスイッチ。

物の公式ページはこれ

http://visit.seekswan.com:6555/xksupport/SKS3200M-8GPY1XF.htm

ファームと公式説明書はここから型番を選ぶとできる。

http://visit.seekswan.com:6555/app/file/download.html

ファームは買った時v1.3だった。v1.9に更新しないとLACPでのリンクアグリゲーションはできないのでする。サイトから.binを落としておく

中国語のWebマニュアルがここにあった。DeepLで頑張って訳したいならこっち（ただ、ファームのバージョンが古いっぽい）

https://zhuanlan.zhihu.com/p/649139286

日本語マニュアルだとこのAmazonカスタマーレビューがやたら詳しい

https://www.amazon.co.jp/gp/customer-reviews/R2YAU0TN0KBE5J/

## 初期設定

まず適当なPCとスイッチのどのポートでもいいので直結する。

IPを手動設定に変える。自分のPCのIPを192.168.10.1とか、サブネットを255.255.255.0にする。

ブラウザで192.168.10.12にアクセス。Webインターフェースが出てくる。

まずファームの更新をする。Tools- Firmware UpgradeからFirmware Upgrade modeに入る。

![[img/xikestor_manage.png]]

.binをアップロードできるようになるので、さっき落としたv1.9をアップロード。更新すると設定が飛ぶので先にIP設定をしないこと。

アップグレードが完了したらSystemからIPアドレスを設定する。

IPの設定があるので自分のルーターのサブネットに合わせてIPを変える。うちの場合はルーターが192.168.1.1なので、192.168.1.2にした。


## Todo

LACPの設定方法を書きたい