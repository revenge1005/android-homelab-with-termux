# NextCloud Configration

이 문서는 Termux의 PRoot-Distro 환경(Ubuntu)에서 NextCloud를 설치하고 초기 설정하는 방법을 설명합니다.

## 01. 선행 작업
- [03. Ubuntu(proot 환경) 설치](https://github.com/revenge1005/android-homelab-with-termux/tree/main/03.%20ubuntu-install)를 완료하여 PRoot-Distro에 Ubuntu 환경이 설정되어 있어야 합니다.
- Termux에서 `proot-distro login ubuntu` 명령어로 Ubuntu 환경에 접속한 상태로 진행합니다.

## 02. 설치 및 설정 단계

### A. apache2 설치 및 설정

```bash
# apache2 설치
root@localhost:~# apt update
root@localhost:~# apt install apache2

# apache2 설정 변경, Termux 환경에서는 80번 포트를 사용할 수 없으므로 포트를 변경해줘야 합니다.
root@localhost:~# vim /etc/apache2/ports.conf
# 5번째 줄 80 -> 8080
Listen 8080
root@localhost:~# vim /etc/apache2/sites-available/000-default.conf
# 1번째 줄 80 -> 8080
<VirtualHost *:8080>

# apache2를 실행하려고 하면 'failed to create proxy mutex AH00016: Configuration Failed' 에러가 발생하는데, 
# 아래 파일의 내용에서 주석을 해제하면 해결됩니다.
root@localhost:~# vi /etc/apache2/apache2.conf
# 74번째 줄, 주석 해제
Mutex file:${APACHE_LOCK_DIR} default
```

### B. PHP 설치 및 설정

```bash
# PHP, PHP-FPM 설치
root@localhost:~# apt -y install php8.3 php8.3-mbstring php-pear php8.3-fpm

# PHP-FPM을 설정하려면 Virtualhost에 설정을 추가합니다.
root@localhost:~# a2enmod proxy_fcgi setenvif
root@localhost:~# a2enconf php8.3-fpm
```

### C. MariaDB 서버 설치 및 설정

```bash
# MariaDB 서버 설치
root@localhost:~# apt -y install mariadb-server

# 설정
root@localhost:~# mysqld_safe --skip-grant-tables --skip-networking &
root@localhost:~# mysql -u root

...

MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> ALTER USER 'root'@'localhost' IDENTIFIED BY '패스워드';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '패스워드';
MariaDB [(none)]> EXIT;
```

### D. 추가 PHP 모듈을 설치 및 설정

```bash
root@localhost:~# apt -y install php8.3-mbstring php8.3-intl php8.3-gd php8.3-zip php8.3-bz2 \
php8.3-mysql php8.3-bcmath php8.3-gmp php8.3-opcache php8.3-imagick \
php8.3-curl php8.3-soap php8.3-redis php-pear php-json php-apcu \
libmagickcore-6.q16-7-extra redis-server unzip

root@localhost:~# cat <<EOF > /etc/php/8.3/fpm/pool.d/nextcloud.conf
[nextcloud]
user = www-data
group = www-data

listen.owner = www-data
listen.group = www-data
listen = /run/php/nextcloud.sock
listen.allowed_clients = 127.0.0.1

pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35

env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp

php_value[session.save_handler] = files
php_value[session.save_path]    = /var/lib/php/sessions

;; maybe you need to configure parameters below if users want to upload large files
php_value[max_execution_time] = 3600
php_value[memory_limit] = 2G
php_value[post_max_size] = 2G
php_value[upload_max_filesize] = 2G
php_value[max_input_time] = 3600
php_value[max_input_vars] = 2000
php_value[date.timezone] = Asia/Tokyo

php_value[opcache.memory_consumption] = 128
php_value[opcache.interned_strings_buffer] = 32
php_value[opcache.max_accelerated_files] = 10000
php_value[opcache.revalidate_freq] = 1
php_value[opcache.save_comments] = 1
php_value[opcache.jit] = 1255
php_value[opcache.jit_buffer_size] = 128M
EOF
```

### E. MariaDB에서 NextCloud를 위한 사용자 및 데이터베이스를 생성

```bash
root@localhost:~# mysql -u root -p
Enter password:

...

MariaDB [(none)]> create database nextcloud; 

# password 부분 설정
MariaDB [(none)]> grant all privileges on nextcloud.* to nextcloud@'localhost' identified by '패스워드'; 

MariaDB [(none)]> exit 
```

### F. NextCloud 설치 및 설정

```bash
# (주의) 설치가 오래 걸림
# NextCloud 설치, 시간이 오래 걸린다면,
root@localhost:~# wget https://download.nextcloud.com/server/releases/latest.zip -P /var/www/

# axel 명령 사용하면 조금더 빠르게 설치된다. (wget이 2시간 걸리면, axel은 1시간 정도)
root@localhost:~# axel https://download.nextcloud.com/server/releases/latest.zip
root@localhost:~# mv latest.zip /var/www/

root@localhost:~# unzip /var/www/latest.zip -d /var/www/
root@localhost:~# chown -R www-data:www-data /var/www/nextcloud
root@localhost:~# cat <<EOF > /etc/apache2/conf-available/nextcloud.conf
Timeout 3600
ProxyTimeout 3600
DirectoryIndex index.php index.html
Header set Strict-Transport-Security "max-age=15768000; includeSubDomains; preload"

<VirtualHost *:80>
    DocumentRoot /var/www/nextcloud
</VirtualHost>

<Directory "/var/www/nextcloud">
    Options FollowSymLinks MultiViews
    AllowOverride All
    Require all granted

    <FilesMatch \.(php|phar)$>
        SetHandler "proxy:unix:/run/php/nextcloud.sock|fcgi://localhost"
    </FilesMatch>
</Directory>
EOF

root@localhost:~# a2enconf nextcloud
root@localhost:~# a2enmod headers
root@localhost:~# a2enmod rewrite
```

### G. apache2, PHP-FPM 서비스 시작

```bash
root@localhost:~# apache2ctl start && /usr/sbin/php-fpm8.3 -D
```

### H. [http://(스마트폰 IP주소:8080)/] URL에 액세스 및 설정

- 관리자 계정 및 데이터베이스 연결 정보를 구성

![08-1](https://github.com/revenge1005/android-homelab-with-termux/blob/main/08.%20NextCloud%20install/08-1.png)

- NextCloud 추천 애플리케이션을 설치할지 여부를 선택

![08-2](https://github.com/revenge1005/android-homelab-with-termux/blob/main/08.%20NextCloud%20install/08-2.png)

- 시작 페이지가 표시

![08-3](https://github.com/revenge1005/android-homelab-with-termux/blob/main/08.%20NextCloud%20install/08-3.png)

- 설치가 완료된 후에는 URL로 NextCloud에 액세스할 수 있습니다.

![08-4](https://github.com/revenge1005/android-homelab-with-termux/blob/main/08.%20NextCloud%20install/08-4.png)

![08-5](https://github.com/revenge1005/android-homelab-with-termux/blob/main/08.%20NextCloud%20install/08-5.png)

## 03. 번외

- NextCloud의 사용자별 디렉터리는 '/var/www/nextcloud/data/'에 저장되어 있습니다.

![08-6](https://github.com/revenge1005/android-homelab-with-termux/blob/main/08.%20NextCloud%20install/08-6.png)

![08-7](https://github.com/revenge1005/android-homelab-with-termux/blob/main/08.%20NextCloud%20install/08-7.png)

- 서버 생성한 파일을 NextCloud를 통해 옮기려고 할 때 다음과 같이 수행해야 합니다.
- 먼저 옮기려는 파일을 사용자별 디렉터리에 옮깁니다. '/var/www/nextcloud/data/(사용자계정명)/file'

![08-8](https://github.com/revenge1005/android-homelab-with-termux/blob/main/08.%20NextCloud%20install/08-8.png)

- 웹 브라우저에서 해당 파일을 확인하려면 다음 작업 중 하나를 수행해야 합니다.

```bash
php /var/www/nextcloud/occ files:scan --all
```

![08-9](https://github.com/revenge1005/android-homelab-with-termux/blob/main/08.%20NextCloud%20install/08-9.png)

- 웹 브라우저에서 확인.

![08-10](https://github.com/revenge1005/android-homelab-with-termux/blob/main/08.%20NextCloud%20install/08-10.png)