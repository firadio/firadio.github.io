---
layout: post
title: "通过Docker-Compose搭建ScyllaDB"
categories: Database
---

# 通过Docker-Compose搭建ScyllaDB

## 一、创建配置文件
## 1.1：创建项目目录
```
mkdir /root/docker/user/scylla/
mkdir /root/docker/user/scylla/etc/
mkdir /root/docker/user/scylla/etc/cert/
```
其中/root/docker/user/scylla/是项目的根目录，
其中/root/docker/user/scylla/etc/用于保存配置文件和证书文件

## 1.2：创建Docker-Compose.yml
```
version: '3.8'
services:
  jekyll:
    container_name: 'scylla'
    hostname: 'scylla'
    image: scylladb/scylla:latest
    restart: unless-stopped
    ports:
      - '9042:9042'
      - '9142:9142'
      - '7001:7001'
    volumes:
      - ./scylla:/scylla
      - ./scylla/etc/.cassandra:/root/.cassandra
      - ./scylla/data:/var/lib/scylla/data
    entrypoint:
      - bash
      - -c
      - |
        cp /scylla/etc/scylla.yaml /etc/scylla/scylla.yaml
        exec /docker-entrypoint.py \
        --broadcast-address=47.90.101.1 \
        --broadcast-rpc-address=47.90.101.1
```
将上面代码保存到项目的根目录/root/docker/user/scylla/docker-compose.yml
将文件中的49.90.101.1换成您服务器的公网IP地址
可以看到我们已经不开放7000端口了，转而使用SSL的7001端口
