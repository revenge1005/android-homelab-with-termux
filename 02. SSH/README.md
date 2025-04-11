## SSH 설치

```bash
pkg update
pkg install openssh
```

## SSH 구동

```bash
# Termux에서는 기본 ssh 포트가 8022으로 설정되어 있다.
sshd
```

## 네트워크 정보 확인을 위한 net-tools 설치

```bash
pkg install net-tools

ifconfig
```

## SSH 접근하기 위한 ID 확인 및 PW 설정

```bash
# ID 확인
whoami

# PW 설정
passwd
```

![02-1](https://github.com/revenge1005/android-homelab-with-termux/blob/main/02.%20SSH/01.png)
![02-2](https://github.com/revenge1005/android-homelab-with-termux/blob/main/02.%20SSH/02.png)
