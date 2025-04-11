# Ubuntu 설치

이 문서는 Termux의 PRoot-Distro 환경(Ubuntu)에서 MariaDB를 설치하고 초기 설정하는 방법을 설명합니다.

## 선행 작업
- [03. Ubuntu 설치](https://github.com/revenge1005/android-homelab-with-termux/tree/main/03.%20ubuntu-install)를 완료하여 PRoot-Distro에 Ubuntu 환경이 설정되어 있어야 합니다.
- Termux에서 `proot-distro login ubuntu` 명령어로 Ubuntu 환경에 접속한 상태로 진행합니다.

## proot, proot-distro 설치

```bash
# 루팅없이 리눅스 배포판을 실행할 수 있게 해주는 proot proot-distro 설치
apt install proot proot-distro
```


## 리눅스 배포판 확인/설치

```bash
# 리눅스 배포판 확인
proot-distro list

# Ubuntu 설치
proot-distro install ubuntu
```

![03-1](https://github.com/revenge1005/android-homelab-with-termux/blob/main/03.%20ubuntu-install/03-1.png)

## Ubuntu 로그인

```bash
# root 계정으로 ubuntu에 로그인 됨
proot-distro login ubuntu
```

![03-2](https://github.com/revenge1005/android-homelab-with-termux/blob/main/03.%20ubuntu-install/03-2.PNG)

## Acquire wakelock 설정

앱이 절정모드로 진입하는 것을 막기 위해 상태바에서 'Acquire wakelock'를 설정

![03-3](https://github.com/revenge1005/android-homelab-with-termux/blob/main/03.%20ubuntu-install/03-3.png)
