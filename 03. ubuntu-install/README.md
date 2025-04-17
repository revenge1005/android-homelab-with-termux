# Ubuntu(proot) install

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

## 02. proot, proot-distro 설치

- Proot는 사용자 공간에서 chroot, mount --bind와 같은 기능을 에뮬레이션하는 도구입니다.
 
  - 루트 권한 없이도 임의의 디렉토리를 새로운 루트 파일 시스템처럼 사용할 수 있게 해주며, 시스템 콜을 가로채어 루트 권한이 필요한 작업을 흉내낼 수 있습니다.

- proot-distro는 다양한 리눅스 배포판(예: Ubuntu, Debian, Alpine, Arch 등)을 Termux 내에서 손쉽게 설치, 삭제, 백업, 복원, 로그인할 수 있게 해주는 관리 도구입니다.

  - proot-distro를 사용하면 복잡한 수동 설정 없이도 다양한 리눅스 환경을 Termux 안에서 바로 사용할 수 있습니다.

```bash
# 루팅없이 리눅스 배포판을 실행할 수 있게 해주는 proot proot-distro 설치
~ $ pkg install proot proot-distro -y
```

![03-1](https://github.com/revenge1005/android-homelab-with-termux/blob/main/03.%20ubuntu-install/03-1.png)

## 03. 리눅스 배포판 확인/설치

```bash
# 리눅스 배포판 확인
~ $ proot-distro list

# Ubuntu 설치
~ $ proot-distro install ubuntu
```

![03-2](https://github.com/revenge1005/android-homelab-with-termux/blob/main/03.%20ubuntu-install/03-2.PNG)

## 04. Ubuntu 로그인

```bash
# root 계정으로 ubuntu에 로그인 됨
~ $ proot-distro login ubuntu
```

![03-3](https://github.com/revenge1005/android-homelab-with-termux/blob/main/03.%20ubuntu-install/03-3.png)

## 05. Acquire wakelock 설정

앱이 절정모드로 진입하는 것을 막기 위해 상태바에서 'Acquire wakelock'를 설정

![03-4](https://github.com/revenge1005/android-homelab-with-termux/blob/main/03.%20ubuntu-install/03-4.png)
