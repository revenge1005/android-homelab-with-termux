# Docker Install 

- 이 문서는 Termux 환경에서 QEMU를 사용해 가상 머신을 생성하고, Alpine Linux에 Docker 및 Portainer를 설치하는 과정을 설명합니다.

> "주의: 스마트폰의 성능에 따라 설치 및 실행 시간이 오래 걸릴 수 있습니다."


## 설치 및 설정 단계


### 01. 업데이트 및 x11-repo 추가

```bash
# Termux의 패키지 목록을 업데이트하고, QEMU 패키지를 설치하기 위해 x11-repo를 추가
~ $ pkg update -y && pkg install x11-repo -y
```


### 02. Qemu 관련 패키지 설치 및 설정

```bash
# Docker를 실행하기 위한 가상화 환경을 구성하기 위해 QEMU 관련 패키지를 설치
~ $ pkg install qemu-utils qemu-common qemu-system-x86_64-headless wget

# qemu-utils, qemu-common: QEMU 가상화 도구 및 기본 유틸리티.
# qemu-system-x86_64-headless: 헤드리스(그래픽 인터페이스 없는) x86_64 가상 머신 실행.
# wget: ISO 파일 다운로드에 사용.
```


### 03. 가상 머신에 Alpine Linux OS 설치

```bash
# 작업 디렉토리 생성 및 이동
~ $ mkdir alpine && cd alpine

# Alpine Linux ISO 파일 다운로드
~/alpine $ wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-virt-3.20.3-x86_64.iso

# 가상 디스크 이미지 생성 (10GB)
~/alpine $ qemu-img create -f qcow2 alpine.qcow2 10g

# 가상 머신 실행 및 Alpine Linux 설치
~/alpine $ qemu-system-x86_64 \
  -m 1024 \
  -netdev user,id=n1,hostfwd=tcp::2222-:22,hostfwd=tcp::2375-:2375,hostfwd=tcp::9000-:9000,hostfwd=tcp::8888-:8888 \
  -device virtio-net,netdev=n1 \
  -nographic \
  -hda alpine.qcow2 \
  -cdrom alpine-virt-3.20.3-x86_64.iso
```


### 04. 가상 머신 Alpine Linux OS 설정

- 가상 머신이 부팅되면 Alpine Linux를 설정합니다.

```bash

Welcome to Alpine Linux 3.20
Kernel 6.6.49-0-virt on an x86_64 (/dev/ttyS0)

# root로 로그인 (Password 필요 없음)
localhost login: root
Welcome to Alpine!

The Alpine Wiki contains a large amount of how-to guides and general
information about administrating Alpine systems.
See <https://wiki.alpinelinux.org/>.

You can setup the system with the command: setup-alpine

You may change this message by editing /etc/motd.

# linux 설정
localhost:~# setup-alpine


 ALPINE LINUX INSTALL
----------------------

 Hostname
----------
Enter system hostname (fully qualified form, e.g. 'foo.example.org') [localhost] alpine # <- hostname 설정

 Interface
-----------
Available interfaces are: eth0.
Enter '?' for help on bridges, bonding and vlans.
Which one do you want to initialize? (or '?' or 'done') [eth0] # <- Enter (기본값 사용)
Ip address for eth0? (or 'dhcp', 'none', '?') [dhcp] # <- Enter (기본값 사용)
Do you want to do any manual network configuration? (y/n) [n] # <- Enter (기본값 사용)
udhcpc: started, v1.36.1
udhcpc: broadcasting discover
udhcpc: broadcasting select for 10.0.2.15, server 10.0.2.2
udhcpc: lease of 10.0.2.15 obtained from 10.0.2.2, lease time 86400

# Root 비밀번호 설정
 Root Password
---------------
Changing password for root
New password:
Bad password: too short
Retype password:
passwd: password for root changed by root

 Timezone
----------
Africa/            Egypt              Iran               Poland
America/           Eire               Israel             Portugal
Antarctica/        Etc/               Jamaica            ROC
Arctic/            Europe/            Japan              ROK
Asia/              Factory            Kwajalein          Singapore
Atlantic/          GB                 Libya              Turkey
Australia/         GB-Eire            MET                UCT
Brazil/            GMT                MST                US/
CET                GMT+0              MST7MDT            UTC
CST6CDT            GMT-0              Mexico/            Universal
Canada/            GMT0               NZ                 W-SU
Chile/             Greenwich          NZ-CHAT            WET
Cuba               HST                Navajo             Zulu
EET                Hongkong           PRC                leap-seconds.list
EST                Iceland            PST8PDT            posixrules
EST5EDT            Indian/            Pacific/

Which timezone are you in? [UTC] # <- Enter (기본값 사용)

 * Seeding random number generator ...
 * Saving 256 bits of non-creditable seed for next boot
 [ ok ]
 * Starting busybox acpid ...
 [ ok ]
 * Starting busybox crond ...
 [ ok ]

 Proxy
-------
HTTP/FTP proxy URL? (e.g. 'http://proxy:8080', or 'none') [none] # <- Enter (기본값 사용)

 Network Time Protocol
-----------------------
Wed Apr 16 06:08:14 UTC 2025
Which NTP client to run? ('busybox', 'openntpd', 'chrony' or 'none') [chrony]  # <- Enter (기본값 사용)
 * service chronyd added to runlevel default
 * Caching service dependencies ...
 [ ok ]
 * Starting chronyd ...
 [ ok ]

 APK Mirror
------------
 (f)    Find and use fastest mirror
 (s)    Show mirrorlist
 (r)    Use random mirror
 (e)    Edit /etc/apk/repositories with text editor
 (c)    Community repo enable
 (skip) Skip setting up apk repositories

Enter mirror number or URL: [1] # <- Enter (기본값 사용)

 User
------
Setup a user? (enter a lower-case loginname, or 'no') [no] # <- Enter (기본값 사용)
Which ssh server? ('openssh', 'dropbear' or 'none') [openssh] none  # <- none 입력

 Disk & Install
----------------
Available disks are:
  fd0   (0.0 GB  )
  sda   (10.7 GB ATA      QEMU HARDDISK   )

Which disk(s) would you like to use? (or '?' for help or 'none') [none] sda # <- sda 입력

The following disk is selected:
  sda   (10.7 GB ATA      QEMU HARDDISK   )

How would you like to use it? ('sys', 'data', 'crypt', 'lvm' or '?' for help) [?] sys # <- sys 입력

WARNING: The following disk(s) will be erased:
  sda   (10.7 GB ATA      QEMU HARDDISK   )

WARNING: Erase the above disk(s) and continue? (y/n) [n] y # <- y 입력
Creating file systems...
Installing system on /dev/sda3:
/mnt/boot is device /dev/sda1
100% ████████████████████████████████████████████==> initramfs: creating /boot/initramfs-virt for 6.6.86-0-virt
/boot is device /dev/sda1

Installation is complete. Please reboot.
# 가상머신 재부팅
alpine:~# poweroff
alpine:~# ~/alpine $

# 가상머신 재실행 (오래 걸림)
~/alpine $ qemu-system-x86_64 \
  -m 1024 \
  -netdev user,id=n1,hostfwd=tcp::2222-:22,hostfwd=tcp::2375-:2375,hostfwd=tcp::9000-:9000,hostfwd=tcp::8888-:80 \
  -device virtio-net,netdev=n1 \
  -nographic \
  -hda alpine.qcow2
```


### 05. 가상 머신 - Docker 설치

```bash
# APK 저장소 수정
alpine:~# vi /etc/apk/repositories
http://dl-cdn.alpinelinux.org/alpine/v3.20/community

# Docker 설치 및 서비스 시작
alpine:~# apk update && apk add docker docker-compose && rc-update add docker boot && service docker start

# Portainer 컨테이너 실행
alpine:~# docker run -d \
  -p 8000:8000 \
  -p 9000:9000 \
  --name=portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```


### 06. Portainer 접근 및 컨테이너 생성

- 'http://termux를_실행중인_스마트폰의_IP_주소:9000'로 접근

![07-1](https://github.com/revenge1005/android-homelab-with-termux/blob/main/07.%20Docker%20install/07-1.png)

![07-2](https://github.com/revenge1005/android-homelab-with-termux/blob/main/07.%20Docker%20install/07-2.png)

![07-3](https://github.com/revenge1005/android-homelab-with-termux/blob/main/07.%20Docker%20install/07-3.png)

![07-4](https://github.com/revenge1005/android-homelab-with-termux/blob/main/07.%20Docker%20install/07-4.png)

![07-5](https://github.com/revenge1005/android-homelab-with-termux/blob/main/07.%20Docker%20install/07-5.png)

![07-6](https://github.com/revenge1005/android-homelab-with-termux/blob/main/07.%20Docker%20install/07-6.png)

![07-7](https://github.com/revenge1005/android-homelab-with-termux/blob/main/07.%20Docker%20install/07-7.png)

- nginx_container 접근 : 'http://termux를_실행중인_스마트폰의_IP_주소:8888'

![07-8](https://github.com/revenge1005/android-homelab-with-termux/blob/main/07.%20Docker%20install/07-8.png)

![07-9](https://github.com/revenge1005/android-homelab-with-termux/blob/main/07.%20Docker%20install/07-9.png)