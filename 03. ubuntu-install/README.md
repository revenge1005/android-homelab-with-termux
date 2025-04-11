# Ubuntu 설치

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

![03-1]()

## Ubuntu 로그인

```bash
# root 계정으로 ubuntu에 로그인 됨
proot-distro login ubuntu
```

![03-2]()