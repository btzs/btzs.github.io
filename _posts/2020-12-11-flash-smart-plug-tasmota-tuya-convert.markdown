---
layout: post
title:  "Flash Smart Plug with Tasmota using Tuya Convert"
date:   2020-12-11 21:03:26 +0800
categories: tasmota home_assistant esp8266
---
When I was looking for ways to control my air conditioners with a low cost solution I came across Home Assistant and Wifi-controlled smart plugs. As I don't want to be dependent on some random vendor I was looking for smart plugs which are based on ESP8266 chip and can be flashed with alternative firmware, prefered over the air (OTA). 

The alernative firmware allows controlling the smart plug locally without cloud access. This means that I can send requests to the plug in my local network directly to the plug using MQTT, API or web interface. 

General steps:
1. Find plugs (or any other type of device with ESP8266 or ESP32, etc.)  which can be flashed with Tuya convert on https://templates.blakadder.com/index.html
2. Set up Tuya convert
3. Flash the device
4. Configure alternative firmware
5. Configure Home Assistant


## 1. Finding the right plug


The vendor has changed the chip a while ago so it is becoming increasingly difficult to find flashable plugs. I personally have used 
Gosund 13A Power Monitoring Plug (https://templates.blakadder.com/gosund_UP111.html)
https://www.amazon.co.uk/Gosund-Wireless-Function-Monitoring-Required/dp/B07ZSDWQQ8

and

 2nice UP111 Power Monitoring Plug 
https://templates.blakadder.com/2nice-UP111.html
https://www.amazon.co.uk/gp/product/B07ZQ34X55/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1

As you can tell they are very similar, down to the model number. I am located in Hong Kong so I need the UK plugs. What I like about them is that they have power monitoring biult in so I can see which device consumes how much electricity.

## 2. Set Up Tuya Convert

I have used a Kali Linux Live system. If I remember correctly the scripts didn't work on my normal Ubuntu system. The reason was that the Wifi adapter must be called wlan0 and not something like wlp3s0 as in Ubuntu. 

Clone the github repo from https://github.com/ct-Open-Source/tuya-convert.git.

    kali@kali:~$ git clone https://github.com/ct-Open-Source/tuya-convert.git
    Cloning into 'tuya-convert'...
    remote: Enumerating objects: 24, done.
    remote: Counting objects: 100% (24/24), done.
    remote: Compressing objects: 100% (24/24), done.
    remote: Total 1357 (delta 9), reused 3 (delta 0), pack-reused 1333
    Receiving objects: 100% (1357/1357), 3.55 MiB | 2.44 MiB/s, done.
    Resolving deltas: 100% (843/843), done.
    
Change directory 

    kali@kali:~$ cd tuya-convert/
    kali@kali:~/tuya-convert$ ls
    config.txt  Dockerfile  install_prereq.sh  README.md  start_flash.sh
    docker      files       LICENSE            scripts

Install the prerequisites

    kali@kali:~/tuya-convert$ sudo ./install_prereq.sh 
    Get:1 http://kali.cs.nctu.edu.tw/kali kali-rolling InRelease [30.5 kB]
    Get:2 http://kali.cs.nctu.edu.tw/kali kali-rolling/main Sources [13.6 MB]
    Get:3 http://kali.cs.nctu.edu.tw/kali kali-rolling/non-free Sources [127 kB]
    Get:4 http://kali.cs.nctu.edu.tw/kali kali-rolling/contrib Sources [63.1 kB]
    Get:5 http://kali.cs.nctu.edu.tw/kali kali-rolling/main amd64 Packages [17.0 MB]
    Get:6 http://kali.cs.nctu.edu.tw/kali kali-rolling/contrib amd64 Packages [105 kB]
    Get:7 http://kali.cs.nctu.edu.tw/kali kali-rolling/non-free amd64 Packages [202 kB]
    Fetched 31.1 MB in 7s (4,516 kB/s)                                        
    Reading package lists... Done
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    build-essential is already the newest version (12.8).
    build-essential set to manually installed.
    Some packages could not be installed. This may mean that you have
    requested an impossible situation or if you are using the unstable
    distribution that some required packages have not yet been created
    or been moved out of Incoming.
    The following information may help to resolve the situation:
    
    The following packages have unmet dependencies:
     python3-tornado : Breaks: mitmproxy (< 5.0~) but 4.0.4-6.1 is to be installed
    E: Error, pkgProblemResolver::Resolve generated breaks, this may be caused by held packages.

I ran into this problem mit with the unmet dependencies. So in removed mitmproxy and tried again:

    kali@kali:~/tuya-convert$ sudo apt remove mitmproxy
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following packages were automatically installed and are no longer required:
      python3-brotli python3-h11 python3-h2 python3-hpack python3-hyperframe
      python3-kaitaistruct python3-pyperclip python3-ruamel.yaml
      python3-sortedcontainers python3-wsproto
    Use 'sudo apt autoremove' to remove them.
    The following packages will be REMOVED:
      kali-linux-default mitmproxy
    0 upgraded, 0 newly installed, 2 to remove and 1309 not upgraded.
    After this operation, 2,849 kB disk space will be freed.
    Do you want to continue? [Y/n] 
    (Reading database ... 293608 files and directories currently installed.)
    Removing kali-linux-default (2020.2.21) ...
    Removing mitmproxy (4.0.4-6.1) ...
    Processing triggers for kali-menu (2020.2.1) ...
    
    kali@kali:~/tuya-convert$ sudo ./install_prereq.sh 
    ...
    ...
    3b96f9b0411c8b19560984a0b2eac3f690d361ba09613e97c3
    Successfully built paho-mqtt sslpsk
    Installing collected packages: paho-mqtt, pycryptodomex, sslpsk
      Attempting uninstall: pycryptodomex
        Found existing installation: pycryptodomex 3.9.7
        Not uninstalling pycryptodomex at /usr/lib/python3/dist-packages, outside environment /usr
        Can't uninstall 'pycryptodomex'. No files were found to uninstall.
    Successfully installed paho-mqtt-1.5.1 pycryptodomex-3.9.9 sslpsk-1.0.0
    Ready to start upgrade


We are now ready to flash the device.

## 3. Flash the device

Enter the directory (in case you're not in there already):

    kali@kali:~$ cd tuya-convert/
    kali@kali:~/tuya-convert$ ls
    config.txt  Dockerfile  install_prereq.sh  README.md  start_flash.sh
    docker      files       LICENSE            scripts
    
Start the procedure:
   
    kali@kali:~/tuya-convert$ sudo ./start_flash.sh 
    tuya-convert v2.4.4
    ======================================================
    TUYA-CONVERT

    https://github.com/ct-Open-Source/tuya-convert
    TUYA-CONVERT was developed by Michael Steigerwald from the IT security company 
    VTRUST (https://www.vtrust.de/) in 
    collaboration with the techjournalists Merlin Schumacher, Pina Merkert, Andrijan 
    Moecker and Jan Mahn at c't Magazine. (https://www.ct.de/)


    ======================================================
    PLEASE READ THIS CAREFULLY!
    ======================================================
    TUYA-CONVERT creates a fake update server environment for ESP8266/85 based tuya 
    devices. It enables you to backup your devices firmware and upload an alternative 
    one (e.g. ESPEasy, Tasmota, Espurna) without the need to open the device and solder 
    a serial connection (OTA, Over-the-air).
    Please make sure that you understand the consequences of flashing an alternative 
    firmware, since you might lose functionality!
    
    Flashing an alternative firmware can cause unexpected device behavior and/or render 
    the device unusable. Be aware that you do use this software at YOUR OWN RISK! Please 
    acknowledge that VTRUST and c't Magazine (or Heise Medien GmbH & Co. KG) CAN NOT be 
    held accountable for ANY DAMAGE or LOSS OF FUNCTIONALITY by typing yes + Enter
    
Confirm by etntering 'yes'.
    
  
    Checking for network interface wlan0... Found.
    Checking UDP port 53... Available.
    Checking UDP port 67... Available.
    Checking TCP port 80... Available.
    Checking TCP port 443... Available.
    Checking UDP port 6666... Available.
    Checking UDP port 6667... Available.
    Checking TCP port 1883... Available.
    Checking TCP port 8886... Available.
    ======================================================
      Starting AP in a screen.
      Starting web server in a screen
      Starting Mosquitto in a screen
      Starting PSK frontend in a screen
      Starting Tuya Discovery in a screen
    
    ======================================================
    
    IMPORTANT
    1. Connect any other device (a smartphone or something) to the WIFI vtrust-flash
       This step is IMPORTANT otherwise the smartconfig may not work!
    2. Put your IoT device in autoconfig/smartconfig/pairing mode (LED will blink fast). 
       This is usually done by pressing and holding the primary button of the device
       Make sure nothing else is plugged into your IoT device while attempting to flash.
    3. Press ENTER to continue
    
Connect for phone, tablet, kindle, etc. to the WiFi network your computer has set up for flashing the device. It is called vtrust-flash. Press 'Enter' once it is connected.


    ======================================================
    Starting smart config pairing procedure
    Waiting for the device to install the intermediate firmware
    Put device in EZ config mode (blinking fast)
    Sending SSID                  vtrust-flash
    Sending wifiPassword          
    Sending token                 00000000
    Sending secret                0101
    ................
    SmartConfig complete.
    Resending SmartConfig Packets
    .........................................................................................
    IoT-device is online with ip 10.42.42.42
    Fetching firmware backup
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
      0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     2 
      1024k    2 26280    0     0   108k      0  0:00:09 --:--:--  0:00:09  1  5 
      1024k    5 58400    0     0  46239      0  0:00:22  0:00:01  0:00:21 46 37 
      1024k   37  384k    0     0   172k      0  0:00:05  0:00:02  0:00:03  1 62 
      1024k   62  643k    0     0   197k      0  0:00:05  0:00:03  0:00:02  1 65 
      1024k   65  674k    0     0   158k      0  0:00:06  0:00:04  0:00:02  1 68 
      1024k   68  705k    0     0   134k      0  0:00:07  0:00:05  0:00:02  1 72 
      1024k   72  738k    0     0   117k      0  0:00:08  0:00:06  0:00:02  1 75 
      1024k   75  769k    0     0   106k      0  0:00:09  0:00:07  0:00:02 78 78 
      1024k   78  801k    0     0  99456      0  0:00:10  0:00:08  0:00:02 32 81 
      1024k   81  832k    0     0  92367      0  0:00:11  0:00:09  0:00:02 32 84 
      1024k   84  865k    0     0  86376      0  0:00:12  0:00:10  0:00:02 32 87 
      1024k   87  896k    0     0  81651      0  0:00:12  0:00:11  0:00:01 32 90 
      1024k   90  928k    0     0  77709      0  0:00:13  0:00:12  0:00:01 32 93 
      1024k   93  959k    0     0  74212      0  0:00:14  0:00:13  0:00:01 32 96 
      1024k   96  990k    0     0  71262      0  0:00:14  0:00:14 --:--:-- 32 99 
      1024k   99 1023k    0     0  68658      0  0:00:15  0:00:15 --:--:-- 32100 
      1024k  100 1024k    0     0  68494      0  0:00:15  0:00:15 --:--:-- 32062
    ======================================================
    Getting Info from IoT-device
    VTRUST-FLASH 1.5
    (c) VTRUST GMBH https://www.vtrust.de/35c3/
    READ FLASH: http://10.42.42.42/backup
    ChipID: d10b89
    MAC: 70:03:9F:D1:0B:89
    BootVersion: 7
    BootMode: normal
    FlashMode: 1M DOUT @ 40MHz
    FlashChipId: 144051
    FlashChipRealSize: 1024K
    Active Userspace: user2 0x81000
    ======================================================
    Ready to flash third party firmware!
    
    For your convenience, the following firmware images are already included in this repository:
      Tasmota v8.1.0.2 (wifiman)
      ESPurna 1.13.5 (base)
    
    You can also provide your own image by placing it in the /files directory
    Please ensure the firmware fits the device and includes the bootloader
    MAXIMUM SIZE IS 512KB
    
    Available options:
      0) return to stock
      1) flash espurna.bin
      2) flash tasmota.bin
      q) quit; do nothing
    Please select 0-2:
    
I go with tasmota here. So I enter '2'.
    
    Are you sure you want to flash tasmota.bin? This is the point of no return [y/N] y
    Attempting to flash tasmota.bin, this may take a few seconds...
    Flashed http://10.42.42.1/files/tasmota.bin successfully in 10752ms, rebooting...
    Look for a tasmota-xxxx SSID to which you can connect and configure
    Be sure to configure your device for proper function!
    
    HAVE FUN!
    ======================================================
    Do you want to flash another device? [y/N] n
    ======================================================
    Cleaning up...
    Closing AP
    Exiting...

Done. That's it. The next step is configuring the smart plugs. 

