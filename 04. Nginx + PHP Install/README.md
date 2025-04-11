# Nginx + PHP Install

- 선행 작업 : [03. Ubuntu 설치](https://github.com/revenge1005/android-homelab-with-termux/tree/main/03.%20ubuntu-install)

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

![04-1](https://github.com/revenge1005/android-homelab-with-termux/blob/main/03.%20ubuntu-install/04-1.png)