---
layout: post
title: "使用Docker搭建PHP站点（Apache）"
categories: Docker
---

# 使用Docker搭建PHP站点（Apache）

1：编写docker-compose.yml文件
```
version: '3.8'
services:
  php74:
    container_name: 'joxq-php74'
    hostname: 'joxq-php74'
    image: 'php:7.4-apache'
    restart: unless-stopped
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - ./root:/root
      - ./root/www:/var/www
    networks:
      - net1
    environment:
      - APACHE_LOG_DIR=/root/www

networks:
  net1:
    external: true
```
2：启动站点
```
docker compose up -d
```

3：执行初始化脚本

3.1：安装sudo(让你的apache拥有root权限)
```
#func
append_line_if_not_exists() {
  line_to_add="$1"
  file_path="$2"
  grep -qxF "$line_to_add" "$file_path" || echo "$line_to_add" | tee -a "$file_path"
}

#common
apt update

#sudo(让你的apache拥有root权限)
apt install -y sudo
append_line_if_not_exists 'www-data ALL=(ALL:ALL) NOPASSWD: ALL' '/etc/sudoers'
```

3.2：安装PHP扩展
```
#php-ext-pdo_mysql
docker-php-ext-install pdo_mysql

#php-ext-zip
apt install -y libzip-dev
docker-php-ext-install zip

#reload
service apache2 reload
```

3.3：启用站点扩展
```
cd /etc/apache2/mods-enabled
ln -s ../mods-available/rewrite.load .
ln -s ../mods-available/ssl.load .
service apache2 reload
```

3.4：更新站点配置(配置SSL)
```
<VirtualHost *:80>
    DocumentRoot /root/firadio-yun-php/appbase/api/fapi
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <Directory />
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:443>
    DocumentRoot /root/firadio-yun-php/appbase/api/fapi
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    SSLEngine on
    SSLCertificateFile "/root/ssl/joxqyd.feieryun.cn_bundle.crt"
    SSLCertificateKeyFile "/root/ssl/joxqyd.feieryun.cn.key"
    <Directory />
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
使其生效
```
cp 000-default.conf /etc/apache2/sites-enabled
service apache2 reload
```
