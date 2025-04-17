# SSH Server Configuration

## 01. Termux에서 패키지 저장소 서버(미러) 변경 및 추가

```bash
~ $ termux-change-repo
```

![02-1](https://github.com/revenge1005/android-homelab-with-termux/blob/main/02.%20SSH/02-1.png)

```bash
# 루트 권한과 관련된 패키지 저장소 추가
~ $ pkg install root-repo -y

# X11(그래픽 환경) 관련 패키지 저장소 추가
~ $ pkg install x11-repo -y

# 저장소의 최신 패키지 목록을 받아고, 설치된 패키지들을 실제로 최신 버전으로 업그레이드
~ $ pkg update -y && pkg upgrade -y
```

![02-2](https://github.com/revenge1005/android-homelab-with-termux/blob/main/02.%20SSH/02-2.png)

## 02. SSH Server Configuration

- Termux에서는 기본 ssh 포트가 8022으로 설정되어 있다.

### 02-1. 설치

```bash
~ $ pkg install openssh -y
```

### 02-2. SSH 실행

```bash
# 'termux-services'가 없는 경우 설치
~ $ pkg install termux-services -y

# termux-app 재실행
~ $ exit

# SSH Service 등록
~ $ sv-enable sshd

# Service 시작
~ $ sv up sshd

# 또는 아래와 같이 sshd 실행하면 실행됨
~ $ sshd
```

## 03. SSH 접속

```bash
# 스마트폰의 IP 주소를 확인하기 위한 net-tools 설치
~ $ pkg install net-tools
~ $ ifconfig

# ID 확인 및 PW 설정
~ $ whoami
~ $ passwd
```

![02-3](https://github.com/revenge1005/android-homelab-with-termux/blob/main/02.%20SSH/02-3.png)

![02-4](https://github.com/revenge1005/android-homelab-with-termux/blob/main/02.%20SSH/02-4.png)