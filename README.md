RaspberryPi2
===

## 作成日
2016/01/12


## 購入物品
|品名|内容|
|---|---|
|Raspberry Pi|Raspberry Pi 2 ModelB|
|microSDカード|RASPBIANを利用する場合は4GB程度が必要。余裕みて8GB程度以上ならなんとか|
|microUSB給電ケーブル|RaspberryPiはmicroUSB端子経由で給電されるのでA-MicroBタイプを選択する|
|USB充電アダプタ|おおよそ1.2A程度が望ましいとされる。iphoneの充電ケーブルで大丈夫|
|RaspberryPi2 ケース|基盤ショート等防ぐために、あった方がよい|
|HDMIケーブル|初期セットアップ時に必要|
|USBキーボード|初期セットアップ時に必要|
|作業用PC|OSイメージをmircoSDに書き込むためのマシン|
|GrovePi+|Groveシステム用のベースシールド, Grove使わない人はいらない|
|WiFiドングル|Buffalo WLI-UC-GNM2を買ったがPlanexの方が発熱量少なくていいらしい|


## OSイメージダウンロード+解凍

### 初期ユーザ
|項目|内容|
|---|---|
|user|pi|
|pass|raspberry|

### OSイメージダウンロード
まず、下記サイトへアクセスし、OSイメージを落とす
最初は**NOOBS**を利用するとセットアップが容易

[Raspberry Pi サイト](https://www.raspberrypi.org/downloads/)

### OSイメージ作成

```
- zipファイルを解凍する
- 出力ファイルは以下の通り(NOOBS v1_5_0)
total 49008
drwxrwxr-x@ 3 tera  staff       102  2 17 12:45 defaults
drwxrwxr-x@ 4 tera  staff       136  2 17 14:14 os
-rw-rw-r--@ 1 tera  staff      9728  2 18 16:23 riscos-boot.bin
-rw-rw-r--@ 1 tera  staff      2250  2 18 16:23 INSTRUCTIONS-README.txt
-rwxrwxr-x@ 1 tera  staff   2148120  2 18 16:24 recovery7.img
-rw-rw-r--@ 1 tera  staff  20238331  2 18 16:26 recovery.rfs
-rwxrwxr-x@ 1 tera  staff   2100464  2 18 16:26 recovery.img
-rw-r--r--@ 1 tera  staff    554520  2 18 16:26 recovery.elf
-rw-r--r--@ 1 tera  staff        54  2 18 16:26 recovery.cmdline
-rw-r--r--@ 1 tera  staff     17856  2 18 16:26 bootcode.bin
-rw-rw-r--@ 1 tera  staff         0  2 18 16:26 RECOVERY_FILES_DO_NOT_EDIT
-rw-rw-r--@ 1 tera  staff       356  5 11 15:53 BUILD-DATA

- 上記ファイルをMicroSDカードへコピー
- MicroSDカードを抜き取り
```

### Raspberry Pi ケーブル結線

- 周辺機器は電源投入前に接続した方が安全
- HDMIケーブルとモニタ接続
- USBキーボード接続(マウスがあればそれも)
- MicroSDを側面から挿入
- MicroUSBケーブルと電源アダプタ結線（）
- 起動が始まる

### OSイメージインストール
- 起動直後にNOOBSの選択画面がでるため Raspbianを選択
- 自動的にOSセットアップが開始される
- 言語、キーボードを日本語に選択しておく（初回起動時に自動的に選択される）
- OSインストール完了まで待つ(5分程度)
- OSインストール完了ダイアログがでたらOKを選択
- 自動的に再起動開始

### 初回起動後
- raspi-config をFINISHを選択する


### WiFi設定

- WiFiドングルを検索

```
$ lsusb
```

- 認識可能なssid検索

```
$ sudo iwlist wlan0 scan | grep ESSID
```


```
$ wpa_passphrase [ESSID] [password] | sudo tee -a /etc/wpa_supplicant/wpa_supplicant.conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
	ssid="ESSID"
	#psk="********"
	psk=**********************************
}
```

```
$ sudo ifdown wlan0
$ sudo ifup wlan0
$ ifconfig -a
```

### SSHログイン

```
(mac)$ ssh pi@*.*.*.*
* ssh-keygen -R *.*.*.*が必要な場合もある
$ exit
```

### 初期セットアップ
- ansibleが利用可能な環境から実施

```
(mac)$ git clone https://github.com/2tom/RPi.git
(mac)$ cd RPi/playbooks
(mac)$ vi rasp_inventory roles/common/vars/main.yml
- 自身の環境にあった変数を設定
(mac)$ ansible-playbooks -i rasp_inventory site.yml -k
```
