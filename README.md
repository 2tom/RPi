RaspberryPi2
===

## 作成日
2016/01/12

### 更新日
2016/04/18


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
(mac)$ ansible-playbook -i hosts site.yml -k
```

### SORACOMセットアップ

#### FS01BUの認識

```
$ sudo apt-get update
$ sudo apt-get install wvdial usb-modeswitch
- FS01BU を挿入
$ lsusb
Bus 001 Device 007: ID 1c9e:6801 OMEGA TECHNOLOGY     ← 1c9e:6801であることを確認
Bus 001 Device 004: ID 2019:ab2a PLANEX GW-USNano2 802.11n Wireless Adapter [Realtek RTL8188CUS]
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

$ sudo reboot
$ lsusb
Bus 001 Device 008: ID 1c9e:6801 OMEGA TECHNOLOGY <- 6801になっていることを確認
```

#### wvdialファイル作成+接続確認

```
$ curl -O http://soracom-files.s3.amazonaws.com/connect_air.sh
$ chmod 755 ./connect_air.sh
$ sudo ./connect_air.sh
Bus 001 Device 007: ID 1c9e:6801 OMEGA TECHNOLOGY
Look for target devices ...
   product ID matched
 Found devices in target mode or class (1)
Look for default devices ...
 No devices in default mode found. Nothing to do. Bye!

waiting for modem device
--> WvDial: Internet dialer version 1.61
--> Initializing modem.
--> Sending: ATZ
ATZ
OK
--> Sending: ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
OK
--> Sending: AT+CGDCONT=1,"IP","soracom.io"
AT+CGDCONT=1,"IP","soracom.io"
OK
--> Modem initialized.
--> Sending: ATD*99***1#
--> Waiting for carrier.
ATD*99***1#
CONNECT 14400000
--> Carrier detected.  Starting PPP immediately.
--> Starting pppd at Thu Apr 28 17:50:34 2016
--> Pid of pppd: 2345
--> Using interface ppp0
--> pppd: ���v�z�[01]�z�[01]
--> pppd: ���v�z�[01]�z�[01]
--> pppd: ���v�z�[01]�z�[01]
--> pppd: ���v�z�[01]�z�[01]
--> pppd: ���v�z�[01]�z�[01]
--> pppd: ���v�z�[01]�z�[01]
--> local  IP address 10.245.7.163
--> pppd: ���v�z�[01]�z�[01]
--> remote IP address 10.64.64.64
--> pppd: ���v�z�[01]�z�[01]
--> primary   DNS address 100.127.0.53
--> pppd: ���v�z�[01]�z�[01]
--> secondary DNS address 100.127.1.53
--> pppd: ���v�z�[01]�z�[01]

出力がとまるので、　CTRL+C　x2　で終了する。

$ sudo cat /etc/wvdial.conf
[Dialer Defaults] 
Init1 = ATZ 
Init2 = ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0 
Init3 = AT+CGDCONT=1,"IP","soracom.io" 
Dial Attempts = 3 
Stupid Mode = 1 
Modem Type = Analog Modem 
Dial Command = ATD 
Stupid Mode = yes 
Baud = 460800 
New PPPD = yes 
Modem = /dev/ttyUSB2 
ISDN = 0 
APN = soracom.io 
Phone = *99***1# 
Username = sora 
Password = sora 
Carrier Check = no 
Auto DNS = 1 
Check Def Route = 1 
```

#### ifupで接続できるようにする

```
$ sudo vi /etc/network/interfaces
allow-hotplug soracom
iface soracom inet wvdial

$ sudo ifup soracom
$ ifconfig ppp0
ppp0      Link encap:Point-to-Pointプロトコル
          inetアドレス:10.139.204.216 P-t-P:10.64.64.64  マスク:255.255.255.255
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  メトリック:1
          RXパケット:116 エラー:0 損失:0 オーバラン:0 フレーム:0
          TXパケット:120 エラー:0 損失:0 オーバラン:0 キャリア:0
      衝突(Collisions):0 TXキュー長:3
          RXバイト:8633 (8.4 KiB)  TXバイト:8577 (8.3 KiB)
$ sudo ifdown soracom
```

#### 接続が切れても自動再接続をする

```
$ sudo vi /etc/ppp/peers/wvdial
noauth
name wvdial
usepeerdns
replacedefaultroute
persist
holdoff 10
```

#### デバイス名を固定する

```
$ sudo vi /etc/udev/rules.d/10-soracom.rules
KERNEL=="ttyUSB*", ATTRS{../idVendor}=="1c9e", ATTRS{../idProduct}=="6801", ATTRS{bNumEndpoints}=="02", ATTRS{bInterfaceNumber}=="02", SYMLINK+="modem"

$ sudo reboot

$ ls -l /dev/ttyUSB* /dev/modem
lrwxrwxrwx 1 root root         7  4月 18 00:35 /dev/modem -> ttyUSB2
crw-rw---- 1 root dialout 188, 0  4月 18 00:35 /dev/ttyUSB0
crw-rw---- 1 root dialout 188, 1  4月 18 00:35 /dev/ttyUSB1
crw-rw---- 1 root dialout 188, 2  4月 18 00:35 /dev/ttyUSB2

$ sudo ifdown soracom
$ sudo vi /etc/wvdial.conf
Modem = /dev/ttyUSB2 を削除

$ sudo ifup soracom
```

#### デバイス挿入時に接続する
```
$ sudo vi /etc/udev/rules.d/10-soracom.rules
KERNEL=="ttyUSB*", ATTRS{../idVendor}=="1c9e", ATTRS{../idProduct}=="6801", ATTRS{bNumEndpoints}=="02", ATTRS{bInterfaceNumber}=="02", SYMLINK+="modem", ENV{SYSTEMD_WANTS}="ifup@soracom"

$ sudo reboot
```
