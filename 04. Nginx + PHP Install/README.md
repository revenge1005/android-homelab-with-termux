# Nginx + PHP Install

이 문서는 Termux의 PRoot-Distro 환경(Ubuntu)에서 Nginx + PHP를 설치하고 초기 설정하는 방법을 설명합니다.

## 01. 선행 작업
- [03. Ubuntu(proot 환경) 설치](https://github.com/revenge1005/android-homelab-with-termux/tree/main/03.%20ubuntu-install)를 완료하여 PRoot-Distro에 Ubuntu 환경이 설정되어 있어야 합니다.
- Termux에서 `proot-distro login ubuntu` 명령어로 Ubuntu 환경에 접속한 상태로 진행합니다.

## 02. 설치 및 설정 단계

### 1. Nginx + PHP 설치

```bash
apt update

apt -y install nginx php8.3 php8.3-mbstring php-pear php8.3-fpm vim
```

```bash
# 버전 확인
php -v

# phpinfo test 페이지 생성
echo '<?php phpinfo(); ?>' > /var/www/html/info.php
```

### 2. 설정 및 실행

```bash
# Termux/PRoot-Distro 환경에서는 80 포트를 사용할 수 없다. 8080 포트로 변경.

vim sites-available/default

server {
        # 8080 포트로 변경
        listen 8080 default_server;
        listen [::]:8080 default_server;

        ...
        ...

        # 추가
        location ~ \.php$ {
              include snippets/fastcgi-php.conf;
              fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        }


# nginx, php-fpm 실행
nginx

/usr/sbin/php-fpm8.3 -D
```

![04-1](https://github.com/revenge1005/android-homelab-with-termux/blob/main/04.%20Nginx%20%2B%20PHP%20Install/04-1.png)