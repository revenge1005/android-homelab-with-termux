# Mariadb 설치

이 문서는 Termux의 PRoot-Distro 환경(Ubuntu)에서 MariaDB를 설치하고 초기 설정하는 방법을 설명합니다.

## 선행 작업
- [03. Ubuntu 설치](https://github.com/revenge1005/android-homelab-with-termux/tree/main/03.%20ubuntu-install)를 완료하여 PRoot-Distro에 Ubuntu 환경이 설정되어 있어야 합니다.
- Termux에서 `proot-distro login ubuntu` 명령어로 Ubuntu 환경에 접속한 상태로 진행합니다.

## 설치 및 설정 단계

### 1. MariaDB 설치

```bash
apt update
apt -y install mariadb-server
```

### 2. MariaDB root 설정

```bash
mysqld_safe --skip-grant-tables --skip-networking &

mysql -u root
```

```bash
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY '새_비밀번호';
# 외부 접속 허용
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '패스워드';
EXIT;
```

### 3. MariaDB 프로세스 종료

```bash
# MariaDB 프로세스 ID 확인
ps aux | grep mysqld

# 프로세스 종료
kill -9 <PID>
```

### 4. 재실행

```bash
mysqld -u -root&

mysql -u root -p
```

### 5. Termux 실행 시 자동으로 MariaDB 실행되도록 하기

```bash
vim .bashrc

if ! ps -e |grep "mysql" > /dev/null;
then
	echo "starting mariadb"
	mysqld -u -root&
fi
```

![05-1](https://github.com/revenge1005/android-homelab-with-termux/blob/main/05.%20mariadb%20install/05-1.PNG)