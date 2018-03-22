---
layout: post
title: 已经运行了!
published: true
---

# Elasticsearch

## 安装

```
yum install java-1.8.0-openjdk-headless.x86_64
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
vim /etc/yum.repos.d/elasticsearch.repo
```

```
[elasticsearch-5.x]
name=Elasticsearch repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

```
yum install elasticsearch -y
chkconfig --add elasticsearch
systemctl daemon-reload
systemctl enable elasticsearch.service
```

## es配置

```
mkdir -p  /data/elasticsearch/data
mkdir -p  /data/elasticsearch/logs
chown elasticsearch. /data/elasticsearch/ -R
chmod g+w /data/elasticsearch/ -R
```

/etc/elasticsearch/elasticsearch.yml 
```
cluster.name: graylog
node.name: graylog_data_a
path.data: /data/elasticsearch/data
path.logs: /data/elasticsearch/logs
network.host: 0.0.0.0
http.port: 9200
discovery.zen.ping.unicast.hosts: ["10.9.37.237", "10.9.41.93","10.9.38.76"]
discovery.zen.minimum_master_nodes: 2
gateway.recover_after_nodes: 3
```

/etc/elasticsearch/jvm.options
```
-Xms8g
-Xmx8g
```
> 物理内存的一半,但要小于32G


## 参数优化

/etc/sysctl.conf
```
vm.max_map_count=262144
vm.swappiness = 1
```

```
#完全禁用 swap
sudo swapoff -a
sysctl  -p
```

## 内存锁定

> 后续补充

## 启动

```
systemctl start elasticsearch
```

##  x-pack

```
/usr/share/elasticsearch/bin/elasticsearch-plugin install x-pack
systemctl restart elasticsearch
```

修改密码
```

curl -XPUT 'elastic:changeme@localhost:9200/_xpack/security/user/elastic/_password?pretty' -H 'Content-Type: application/json' -d'
{
  "password": "vuGlr9mnAvtFrwTb"
}'


curl -XPUT 'elastic:vuGlr9mnAvtFrwTb@localhost:9200/_xpack/security/user/kibana/_password?pretty' -H 'Content-Type: application/json' -d'
{
  "password": "jmikmtahNLIPhMIv"
}'

curl -XPUT elastic:vuGlr9mnAvtFrwTb@localhost:9200/_xpack/security/user/graylog -H 'Content-Type: application/json' -d'
{
  "password" : "xWRMr8ksgGiZcNNH", 
  "roles" : [ "superuser"], 
  "enabled": true 
}'
```

license导入
```
curl -XPUT -u elastic 'http://localhost:9200/_xpack/license' -H "Content-Type: application/json" -d @license.json
```