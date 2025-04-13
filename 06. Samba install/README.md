# Samba install (루팅 필요)

이 문서는 Termux 환경에서 Samba를 설치하고 초기 설정하여 파일 공유 서비스를 구축하는 방법을 설명합니다.


## 선행 작업
- (필수) 스마트폰이 루팅되어 있어야 합니다.

## 준비물
![06-1](https://github.com/revenge1005/android-homelab-with-termux/blob/main/06.%20Samba%20install/06-1.jpg)

## 설치 및 설정 단계

### 01. Termux 스토리지(내부 및 외부) 설정 및 확인

```bash
# 내부 및 외부 저장소에 접근할 수 있도록 권한을 설정하고 디렉토리를 생성하는 명령어
~ $ termux-setup-storage

# 현재 디렉토리의 파일 및 폴더 목록 확인
~ $ ls
storage

# storage 디렉토리로 이동 및 확인
~ $ cd storage/
~/storage $ ls
dcim  downloads  external-1  movies  music  pictures  shared

# /dev/ 디렉토리 내 목록 확인 시도 (권한 부족으로 실패)
~/storage $ ls /dev/
ls: cannot open directory '/dev/': Permission denied

# 슈퍼유저(root) 권한으로 전환
~/storage $ su

# root 권한으로 /dev/block/by-name 디렉토리 내 목록 확인
:/data/data/com.termux/files/home/storage # ls /dev/block/by-name
ImageFv bk33 bluetooth  dip         logfs      recovery  storsec_b
abl_a   bk41 boot       dsp         logo       sda       switch
abl_b   bk42 cache      dtbo        minidump   sdb       system
aop_a   bk43 cdt        frp         misc       sdc       toolsfv
aop_b   bk44 cmnlib64_a fsc         modem      sdd       tz_a
apdp    bk45 cmnlib64_b fsg         modemst1   sde       tz_b
bk01    bk46 cmnlib_a   hyp_a       modemst2   sdf       userdata
bk02    bk47 cmnlib_b   hyp_b       msadp      sec       vbmeta
bk03    bk48 cust       keymaster_a oops       splash    vendor
bk04    bk49 ddr        keymaster_b persist    spunvm    xbl_a
bk05    bk51 devcfg_a   keystore    persistbak ssd       xbl_b
bk31    bk52 devcfg_b   limits      qupfw_a    sti       xbl_config_a
bk32    bk53 devinfo    logdump     qupfw_b    storsec_a xbl_config_b

# /mnt/ 디렉토리로 이동
:/data/data/com.termux/files/home/storage # cd /mnt/

# /mnt/ 디렉토리 내 파일 및 폴더 목록 확인
:/mnt # ls
appfuse expand   obb     runtime secure vendor
asec    media_rw product sdcard  user

# /mnt/media_rw/ 디렉토리 내 파일 및 폴더 목록 확인 (외부 저장소 관련 디렉토리 표시)
:/mnt # cd media_rw/
:/mnt/media_rw # ls
6338-3138 8C3AF3E33AF3C7EA
:/mnt/media_rw # blkid | tail -n 2
/dev/block/mmcblk0p1: LABEL="disk" UUID="6338-3138" TYPE="exfat"
/dev/block/sdg1: UUID="8C3AF3E33AF3C7EA" TYPE="ntfs"
```

### 02. Termux에 HDD(또는 microSD) 마운트

```bash
# 루트 권한으로 Termux 홈 디렉토리에서 작업 시작 (su 명령으로 루트 전환 가정)
:/data/data/com.termux/files/home # ls
storage  

# HDD 마운트를 위한 디렉토리 생성
:/data/data/com.termux/files/home # mkdir HDD

# microSD 마운트를 위한 디렉토리 생성
:/data/data/com.termux/files/home # mkdir microSD

# HDD 장치(/dev/block/sdg1)를 HDD 디렉토리에 마운트
# -o uid=1000,gid=1000: Termux 사용자의 UID/GID로 소유자 설정
# umask=0000: 모든 사용자에게 읽기/쓰기/실행 권한 부여
:/data/data/com.termux/files/home # mount -o uid=1000,gid=1000,umask=0000 /dev/block/sdg1 HDD/

# microSD 장치(/dev/block/mmcblk0p1)를 microSD 디렉토리에 마운트
# 동일한 옵션으로 Termux 사용자 권한 설정
:/data/data/com.termux/files/home # mount -o uid=1000,gid=1000,umask=0000 /dev/block/mmcblk0p1 microSD/

# 루트 권한 종료
:/data/data/com.termux/files/home # exit

# Termux 기본 디렉토리로 돌아옴 (루트 권한 종료 후)
~/storage $

# 홈 디렉토리로 이동
~/storage $ cd

# 홈 디렉토리의 내용 확인 (새로 생성한 HDD, microSD 디렉토리 확인)
~ $ ls
HDD  microSD  storage
```

### 03. Chroot 환경을 설정하여 SAMBA 서버를 구성

- 참고 문서: Termux에서 Chroot 설치 가이드
- 링크: https://github.com/LinuxDroidMaster/Termux-Desktops/blob/main/Documentation/chroot/box64droid_chroot.md

```bash
# Box64Droid 설치 스크립트 다운로드 및 실행 (chroot 환경 구축, 설치에 시간이 걸림)
~ $ curl -o install https://raw.githubusercontent.com/Ilya114/Box64Droid/main/installers/install.sh && chmod +x install && ./install

# 설치 중 버전 선택 메뉴 출력 (4번 선택)
Hello, this is a Box64Droid installer, please select need version to install:

Actual version:
1) Native (Adreno 610-750, Android 10+)
2) Hangover (All GPU's, WIP (not able to install now))

Irrelevant versions:
3) Non-root (Adreno 610-750, Android 12+)
4) Root (Adreno 610-750, Android 10+)
5) VirGL (Other GPU's, based on non-root, Android 12+)

6) Cancel the Box64Droid installation

4

# 설치 완료 후 안내 메시지 출력, Box64Droid 실행
 ...
 ...

 Installation finished. To start Box64Droid run 'box64droid --start', to see more arguments run 'box64droid --help'

 And if everything goes as planned, Wine and 7-Zip file manager will start.

~ $ box64droid --start

# Box64Droid 실행 메뉴 출력 (8번 터미널 모드 선택 (chroot 환경 진입))
Box64Droid by Ilya114, Glory to Ukraine!

Select, what need run:
1) Wine
2) Wine (debug mode)
3) Change Wine version
4) Change Box64/86 and Wine settings
5) Update Box64, Box86 and system
6) Recreate Wine prefix
7) Winetricks
8) Terminal mode
9) Exit from Box64Droid

8

# chroot 환경에서 네트워크 설정 (시간이 오래 걸릴 수 있음)
root@localhost:~# echo "127.0.0.1 localhost" > /etc/hosts

groupadd -g 3003 aid_inet
groupadd -g 3004 aid_net_raw
groupadd -g 1003 aid_graphics
usermod -g 3003 -G 3003,3004 -a _apt
usermod -G 3003 -a root

apt update
apt upgrade

# Samba 및 관련 패키지 설치
root@localhost:/mnt# apt install samba samba-common-bin

# Samba 설정 파일 편집
root@localhost:/mnt# vim /etc/samba/smb.conf

# smb.conf 마지막에 추가 (HDD와 microSD 공유 설정)
[Android_Share_HDD]
path = /mnt/HDD
writeable = yes
browseable = yes
public = no

[Android_Share_microsd]
path = /mnt/microSD
writeable = yes
browseable = yes
public = no

# Samba 사용자(root) 추가 및 비밀번호 설정
root@localhost:/mnt# smbpasswd -a root
New SMB password:
Retype new SMB password:
Added user root.

# Samba 서비스 시작
root@localhost:/mnt# smbd
```

### 04. Samba 서버 연결

![06-2](https://github.com/revenge1005/android-homelab-with-termux/blob/main/06.%20Samba%20install/06-2.png)

![06-3](https://github.com/revenge1005/android-homelab-with-termux/blob/main/06.%20Samba%20install/06-3.png)

![06-3](https://github.com/revenge1005/android-homelab-with-termux/blob/main/06.%20Samba%20install/06-4.png)