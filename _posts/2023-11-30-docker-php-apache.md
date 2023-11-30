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

#php-ext-pdo_mysql
docker-php-ext-install pdo_mysql

#php-ext-zip
apt install -y libzip-dev
docker-php-ext-install zip

cp 000-default.conf /etc/apache2/sites-enabled

cd /etc/apache2/mods-enabled
ln -s ../mods-available/rewrite.load .
ln -s ../mods-available/ssl.load .

#start
service apache2 reload
```

