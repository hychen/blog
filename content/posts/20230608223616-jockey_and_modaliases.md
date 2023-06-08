+++
title = "Jockey and Modaliases"
publishDate = 2011-12-02
categories = ["觀想"]
draft = false
+++

Jockey 是第三方驅動程式套件管裡程式，使User可以簡單的安裝不被Ubuntu收納的閉源驅動程式 由 Martin Piit 於 UDS-N 時提出，在Onereic版本後都可以使用。

它的設計理念如下

-   使驅動程式的 Package Maintainer 可以利用 modaliases 宣告支援的裝置
-   Jockey 會檢查 package 的Modaliases Section，判斷這個 package 適不適用這台電腦， 使得User 可以輕易找出適合的第三方驅動程式，常見的ATI，Nvidia的顯卡Driver 均是使用這個方式安裝。


## 那麼什麼是 modaliases 呢? {#那麼什麼是-modaliases-呢}

modaliases 其實就是讓 sysfs 把DMI table 及 Kernel 的硬體資訊 Dump 出來到檔名為 modalias 的檔案上，讓 User Space 的軟體可以讀取

```nil
$ cat /sys/devices/pci0000:00/0000:00:1f.1/modalias
      pci:v00008086d000024DBsv0000103Csd0000006Abc01sc01i8A
```

小寫英文字母代表屬性名稱, 所代表的意義每個裝置不一定一樣, v通常代表vendor ID, d通常代表device ID (通常也只用這個來判斷)

在 Arch Linux Wiki上 有篇[文章](https://web.archive.org/web/20120910101845/https://wiki.archlinux.org/index.php/Modalias)，說明 modaliases 在 Kernel Module的用法，算是滿清楚的，有興趣的人可以連過去閱讀。


## 製作 Debian Binary Package 需修改的地方 {#製作-debian-binary-package-需修改的地方}


### 1. 修改 debian/modaliases {#1-dot-修改-debian-modaliases}

寫入這個 pacakge 支援的 modaliases，以下這個範例包含4個MODALIASE Trigger，每一行皆為一個MODALIAS Trigger

\#+end_src
alias usb:v0A5Cp21D3d\*dc\*dsc\*dp\*ic\*isc\*ip\* bcm bt-bcm43142
alias usb:v0A5Cp21D7d\*dc\*dsc\*dp\*ic\*isc\*ip\* bcm bt-bcm43142
alias usb:v0A5Cp21E1d\*dc\*dsc\*dp\*ic\*isc\*ip\* bcm bt-bcm43142
alias usb:v0A5Cp21E3d\*dc\*dsc\*dp\*ic\*isc\*ip\* bcm bt-bcm43142
\#+end_src

MODALIASE Trigger 的語法為

```nil
aliases MODALIAS TAG PACKAGE
```

每一個 Trigger 由三個部份組成，分別是 MODALIASE , TAG , PACKAGE ， 其中MODALIASES前面介紹過了，而 PACKAGE 則為要安裝的套件名稱。

這段句子翻成中文就是”若電腦上有符合 MODALIASE 的裝置，就安裝 PACKAGE “, MODALIASE可以為裝置的硬體資料或是DMI Table


### 2. 修改 debian/control {#2-dot-修改-debian-control}

增加一行 XB-Modaliases: ${modaliases}
再把 dh_modalises 加進 Build-Depends


### 3. 修改 debian/rules {#3-dot-修改-debian-rules}

在dh後面加--with modaliases

```nil
dh $@ --with modaliases"
```


## 如何測試包好的套件？ {#如何測試包好的套件}

要讓Jockey可以判斷你所做的package是不是適合的驅動程式的前提是， 你用apt-cache search可以找到你包好的pacakge。

最簡單的方式就是你自己製作一個Local APT Archive，再把它加進apt source, 這方面資訊網路上很多， 我就不在重複了。 :p
