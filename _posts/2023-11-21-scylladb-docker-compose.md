---
layout: post
title: "通过Docker-Compose搭建ScyllaDB"
categories: Database
---

# 通过Docker-Compose搭建ScyllaDB


## 一、创建配置文件


### 1.1：创建项目目录
```
mkdir /root/docker/user/scylla/
mkdir /root/docker/user/scylla/scylla
mkdir /root/docker/user/scylla/scylla/etc/
mkdir /root/docker/user/scylla/scylla/etc/cert/
```
其中/root/docker/user/scylla/是项目的根目录，
其中./scylla/etc/用于保存配置文件和证书文件


### 1.2：创建Docker-Compose.yml
#### 1.2.1：首先打开要编辑的文件
```
nano /root/docker/user/scylla/docker-compose.yml
```
#### 1.2.2：然后将下面代码贴入保存
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
      - ./scylla/data:/var/lib/scylla/data
    environment:
      - SSL_CERTFILE=/scylla/etc/cert/db.crt
    entrypoint:
      - bash
      - -c
      - |
        cp /scylla/etc/scylla.yaml /etc/scylla/scylla.yaml
        cp /scylla/etc/cassandra-rackdc.properties /etc/scylla/cassandra-rackdc.properties
        exec /docker-entrypoint.py \
        --broadcast-address=47.90.101.1 \
        --broadcast-rpc-address=47.90.101.1
```
将文件中的49.90.101.1换成您服务器的公网IP地址，
如果这是第二台服务器需要在exec /docker-entrypoint.py \代码下面添加
```
--seeds=47.90.101.1 \
```

### 1.3：创建配置文件
#### 1.3.1：scylla.yaml
将下面代码保存到项目根目录中的/scylla/etc/scylla.yaml
```
num_tokens: 256
commitlog_sync: periodic
commitlog_sync_period_in_ms: 10000
commitlog_segment_size_in_mb: 32
schema_commitlog_segment_size_in_mb: 32
seed_provider:
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
          - seeds: "127.0.0.1"
listen_address: localhost
native_transport_port: 9042
native_shard_aware_transport_port: 19042
native_transport_port_ssl: 9142
read_request_timeout_in_ms: 5000
write_request_timeout_in_ms: 2000
cas_contention_timeout_in_ms: 1000
endpoint_snitch: GossipingPropertyFileSnitch
rpc_address: localhost
rpc_port: 9160
api_port: 10000
api_address: 127.0.0.1
batch_size_warn_threshold_in_kb: 128
batch_size_fail_threshold_in_kb: 1024
authenticator: PasswordAuthenticator
authorizer: CassandraAuthorizer
partitioner: org.apache.cassandra.dht.Murmur3Partitioner
commitlog_total_space_in_mb: -1
storage_port: 7000
ssl_storage_port: 7001
server_encryption_options:
    internode_encryption: all
    certificate: /scylla/etc/cert/db.crt
    keyfile: /scylla/etc/cert/db.key
    truststore: /scylla/etc/cert/cadb.pem
    require_client_auth: True
client_encryption_options:
    enabled: true
    certificate: /scylla/etc/cert/db.crt
    keyfile: /scylla/etc/cert/db.key
    truststore: /scylla/etc/cert/cadb.pem
    require_client_auth: True
murmur3_partitioner_ignore_msb_bits: 12
force_schema_commit_log: true
consistent_cluster_management: true
api_ui_dir: /opt/scylladb/swagger-ui/dist/
api_doc_dir: /opt/scylladb/api/api-doc/
```
#### 1.3.2：cassandra-rackdc.properties
将下面代码保存到项目根目录中的/scylla/etc/cassandra-rackdc.properties
```
#
# cassandra-rackdc.properties
#
# The lines may include white spaces at the beginning and the end.
# The rack and data center names may also include white spaces.
# All trailing and leading white spaces will be trimmed.
#  
dc=dc1
rack=rack1
# prefer_local=<false | true>
# dc_suffix=<Data Center name suffix, used by EC2SnitchXXX snitches>
#
```
其中dc和rack需要取消注解并填写相应资料


### 1.4：创建证书文件
根据官方提供的方法https://opensource.docs.scylladb.com/stable/operating-scylla/security/generate-certificate.html
在/root/docker/user/scylla/scylla/etc/cert/目录中创建证书
```
cd /root/docker/user/scylla/scylla/etc/cert/
```

#### 1.4.1：创建证书配置文件
首先将下面的代码保存到项目根目录中的/scylla/etc/cert/db.cfg
```
[ req ]
default_bits = 4096
default_keyfile = db.key
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no
[ req_distinguished_name ]
C = SE
ST = Stockholm
L = Stockholm
O = foo.bar
OU = foo.bar
CN= db.foo.bar
emailAddress = postmaster@foo.bar
[v3_ca]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer:always
basicConstraints = CA:true
[v3_req]
# Extensions to add to a certificate request
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
```

#### 1.4.2：生成各种证书文件
```
cd /root/docker/user/scylla/scylla/etc/cert/
openssl genrsa -out cadb.key 4096
openssl req -x509 -new -nodes -key cadb.key -days 3650 -config db.cfg -out cadb.pem
openssl genrsa -out db.key 4096
openssl req -new -key db.key -out db.csr -config db.cfg
openssl x509 -req -in db.csr -CA cadb.pem -CAkey cadb.key -CAcreateserial  -out db.crt -days 365 -sha256
```

### 1.5：加密登录配置文件
#### 1.5.1：首先编辑根目录中的/scylla/etc/cqlshrc
```
nano /root/docker/user/scylla/scylla/etc/cqlshrc
```
#### 1.5.2：然后贴入下面代码进行保存
```
[ssl]
userkey = /scylla/etc/cert/db.key
usercert = /scylla/etc/cert/db.crt
```

## 二、启动Docker并检查运行状态

### 2.1：启动ScyllaDB并登录
#### 2.1.1：启动项目到后台
```
docker compose up -d
```

#### 2.1.2：查看运行状态
```
docker ps -a
```
可以看到我们已经不开放7000端口了，转而使用SSL的7001端口

#### 2.1.3：查看是否有报错
```
docker attach --sig-proxy=false scylla
```

#### 2.1.4：查看节点运行状态
```
docker exec -it scylla nodetool status
```

#### 2.1.5：测试加密登录
```
docker exec -it scylla cp /scylla/etc/cqlshrc /root/.cassandra/cqlshrc
```
```
docker exec -it scylla cqlsh scylla 9142 --ssl -u cassandra -p cassandra
```
出现下面提示代表登录成功
```
Connected to  at scylla:9142.
[cqlsh 5.0.1 | Cassandra 3.0.8 | CQL spec 3.3.1 | Native protocol v4]
Use HELP for help.
cassandra@cqlsh>
```

### 2.2：基础设置
#### 2.2.1：给用于系统认证的键空间设置复制策略
```
ALTER KEYSPACE system_auth WITH REPLICATION = {'class': 'NetworkTopologyStrategy', 'replication_factor': 1};
```
注意replication_factor的数量不能超过数据中心的node数量

#### 2.2.2：管理超级用户
##### 2.2.2.1：创建新的超级用户
```
CREATE ROLE root WITH PASSWORD = '你的密码' AND SUPERUSER = true AND LOGIN = true;
```
##### 2.2.2.2：登录新的超级用户
```
docker exec -it scylla cqlsh -u root
```
##### 2.2.2.3：删除默认超级用户
```
DROP ROLE IF EXISTS 'cassandra';
```

### 2.3：键空间管理和用户权限

#### 2.3.1：键空间管理
##### 2.3.1.1 创建键空间
```
CREATE KEYSPACE IF NOT EXISTS test WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'replication_factor' : 1 };
```

##### 2.3.1.2：删除键空间
```
DROP KEYSPACE test;
```

##### 2.3.1.3：修改键空间
ALTER KEYSPACE test WITH REPLICATION = {'class': 'NetworkTopologyStrategy', 'replication_factor': 1};

##### 2.3.1.4：查看键空间
```
SELECT * FROM system_schema.keyspaces;
```

#### 2.3.2：用户管理
可以参考官方文档
https://opensource.docs.scylladb.com/stable/operating-scylla/security/authorization.html
##### 2.3.2.1：增加用户
```
CREATE USER test WITH PASSWORD 'test';
```

##### 2.3.2.2：删除用户
```
DROP USER test;
```

##### 2.3.2.3：修改用户
修改用户的密码
```
ALTER USER test WITH PASSWORD 'test';
```

##### 2.3.2.4：查询用户
```
LIST USERS;
LIST ROLES of test;
```


#### 2.3.3：权限管理
需要用到Role Based Access Control (RBAC)
https://opensource.docs.scylladb.com/stable/operating-scylla/security/rbac-usecase.html
##### 2.3.3.1：给用户授权
```
grant_permission_statement: GRANT `permissions` ON `resource` TO `user_name`
permissions: ALL [ PERMISSIONS ] | `permission` [ PERMISSION ]
permission: CREATE | ALTER | DROP | SELECT | MODIFY | AUTHORIZE | DESCRIBE
resource: ALL KEYSPACES
        :| KEYSPACE `keyspace_name`
        :| [ TABLE ] `table_name`
        :| ALL USERS
        :| USER `user_name`
```
例子1：给test用户授予对test键空间的读取权限
```
GRANT SELECT ON KEYSPACE test TO test;
```
例子2：给test用户授予对test键空间的全部操作权限（ALTER|AUTHORIZE|CREATE|DROP|MODIFY|SELECT）
```
GRANT ALL ON KEYSPACE test TO test;
```
##### 2.3.3.2：撤销用户权限
撤销test对test键空间的全部权限
```
REVOKE ALL ON KEYSPACE test FROM test;
```

##### 2.3.3.3：查询权限列表
```
LIST ALL PERMISSIONS OF test;
```

### 2.4 表管理

#### 2.4.1：表管理
支持的数据类型可参考官方文档
https://cassandra.apache.org/doc/stable/cassandra/cql/types.html
##### 2.4.1.1：建表
```
CREATE TABLE IF NOT EXISTS test.test (name text, uuid uuid, PRIMARY KEY (name));
```
##### 2.4.1.2：删表
DROP TABLE test.test;

##### 2.4.1.3：改表
加字段
```
ALTER TABLE test ADD(f1 int,f2 float); 
```
删字段
```
ALTER TABLE test DROP(f1,f2);
```

##### 2.4.1.4：查表
```
SELECT * FROM system_schema.columns WHERE keyspace_name='test';
```

#### 2.4.2：表索引
#### 2.4.2.1：创建索引
#### 2.4.2.2：删除索引

### 2.5 数据管理
#### 2.5.1 简单的增删改查
##### 2.5.1.1 插入记录
```
INSERT INTO test.test(name, uuid)VALUES('test', uuid())IF NOT EXISTS;
```
##### 2.5.1.2 删除记录
```
DELETE FROM test.test where name='test';
```
##### 2.5.1.3 修改记录
```
UPDATE test.test SET name='test1' WHERE name='test';
```
##### 2.5.1.4 查询记录
```
SELECT * FROM test.test where name='test1';
```

