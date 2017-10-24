How to run a script
===================

## Environment
* OS: Ubuntu 16.04 LTS
* Wi-Fi USB dongle: PLANEX GW-US54Mini2 (rt73usb)
	* Note:
		* We use a default driver in this senario.
		* A default rt73usb driver probaly does not support FT(802.11r)


## Test whther a Wi-Fi USB dongle can work
------------------------------------------------------------
* Plug a Wi-Fi USB dongle into Ubuntu
	* Ubuntu 16.04 LTS can detect the Wi-Fi USB dongle with rt73usb driver.
	* wlx0090cce65504 is the interface name.
```dmesg
[   14.072883] ieee80211 phy0: rt2x00_set_chip: Info - Chipset detected - rt: 2573, rf: 0002, rev: 000a
[   14.146390] ieee80211 phy0: Selected rate control algorithm 'minstrel_ht'
[   14.147086] usbcore: registered new interface driver rt73usb
[   14.197523] rt73usb 1-1.4:1.0 wlx0090cce65504: renamed from wlan0
```

```sh
$ ifconfig
wlx0090cce65504 Link encap:イーサネット  ハードウェアアドレス 00:90:cc:e6:55:04  
          inet6アドレス: fe80::34eb:e7bf:e7bf:e502/64 範囲:リンク
          UP BROADCAST RUNNING MULTICAST  MTU:1500  メトリック:1
          RXパケット:6 エラー:0 損失:0 オーバラン:0 フレーム:0
          TXパケット:22 エラー:0 損失:0 オーバラン:0 キャリア:0
          衝突(Collisions):0 TXキュー長:1000 
          RXバイト:1404 (1.4 KB)  TXバイト:3997 (3.9 KB)
```


## Set up on Ubuntu 16.04
------------------------------------------------------------
* Install dependency tools
```sh
$ sudo apt-get update
$ sudo apt-get install build-essential libgmp3-dev python-dev
$ sudo apt-get install python-pip
$ pip install pycryptodomex
$ sudo apt-get install libnl-3-dev libnl-genl-3-dev pkg-config libssl-dev net-tools git sysfsutils python-scapy
```

* Run disable-hwcrypto.sh
```sh
$ sudo ./disable-hwcrypto.sh
```

* Reboot Ubuntu

* Check if a nohwcrypt parameter is set properly.
```console
$ systool -vm rt73usb
Module = "rt73usb"

  Attributes:
    coresize            = "32768"
    initsize            = "0"
    initstate           = "live"
    refcnt              = "0"
    srcversion          = "FE4ADCE723A8C0B21A28A01"
    taint               = ""
    uevent              = <store method only>
    version             = "2.3.0"

  Parameters:
    nohwcrypt           = "Y"			#### Enable nohwcrypt
```


## Test whther wpa_supplicant can work (without a krack-ft-test.py)
------------------------------------------------------------
* Un-plug and plug a Wi-Fi USB dongle into Ubuntu

* Disable Wi-Fi on System Settings
	* Click top-right setting icon -> [System Settings] -> [Network]
	* Disable Wi-Fi

* Run rfkill
```sh
$ sudo rfkill unblock wifi
```

* Create a wpa_supplicant configuration file as network.conf
	* A basic example below:
```conf
ctrl_interface=/var/run/wpa_supplicant
network={
	ssid="testnet"
	key_mgmt=FT-PSK
	psk="password"
}
```

* Run wpa_supplicant command
```sh
$ sudo wpa_supplicant -D nl80211 -i wlx0090cce65504 -c network.conf
Successfully initialized wpa_supplicant
wlx0090cce65504: CTRL-EVENT-REGDOM-CHANGE init=BEACON_HINT type=UNKNOWN
wlx0090cce65504: CTRL-EVENT-REGDOM-CHANGE init=BEACON_HINT type=UNKNOWN
```

* If fail, see FAQ.


## Run a krack-ft-test.py
------------------------------------------------------------
* Un-plug and plug a Wi-Fi USB dongle into Ubuntu

* Run rfkill
```sh
$ sudo rfkill unblock wifi
```

* Up interface manually
```sh
$ sudo iw wlx0090cce65504 set type monitor
$ sudo ifconfig wlx0090cce65504 up
```

* Modify a script
	* Comment out not to call configure_interfaces()
	```python
			#self.configure_interfaces()
	```

* Run a script
```sh
$ sudo python krack-ft-test.py wpa_supplicant -D nl80211 -i wlx0090cce65504 -c network.conf
Successfully initialized wpa_supplicant
```

* Lauch another terminal and type below
```console
$ sudo wpa_cli -i wlx0090cce65504
> status
wpa_state=SCANNING
address=00:90:cc:e6:55:04
> 
> scan_result
...
> 
> roam 00:90:cc:e6:55:04
FAIL
> 
```


```console
$ sudo wpa_cli -i wlx0090cce65504
> status
bssid=88:57:ee:70:8e:f6
> 
> scan_result
88:57:ee:70:8e:f6	2412	-48	[WPA2-PSK-CCMP][WPS][ESS]	hoge-hoge
...
> 
> roam 88:57:ee:70:8e:f6
OK
```

* As a scan_result, No AP supported FT is found in the around.




