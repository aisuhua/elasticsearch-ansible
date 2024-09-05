# elasticsearch-ansible

使用 ansible 以简单的方式搭建 elasticsearch 集群。

## 安装

```sh
git clone git@github.com:aisuhua/elasticsearch-ansible.git
cd elasticsearch-ansible

# 按需修改 inventory 、安装目录、用户等信息
vim hosts.yaml

ansible-playbook playbook.yaml
```

## 卸载

```sh
ansible-playbook remove.yaml
```

## 测试

```sh
# 查看集群健康状态
$ curl https://172.31.96.151:19201/_cat/health -k -u elastic -p
Enter host password for user 'elastic':
1725539414 12:30:14 es1 green 3 3 68 34 0 0 0 0 - 100.0%

# 查看集群节点
$ curl https://172.31.96.151:19201/_cat/nodes -k -u elastic -p
Enter host password for user 'elastic':
172.31.96.151 57 94 28 0.31 1.14 3.47 cdfhilmrstw - node3-es1
172.31.96.151 59 94 28 0.31 1.14 3.47 cdfhilmrstw * node1-es1
172.31.96.151 60 94 28 0.31 1.14 3.47 cdfhilmrstw - node2-es1
```

## 特点

- 可离线安装
- 支持在单机上安装多个不同版本的集群
- 卸载简单，方便来回测试
- 自带 100 年 SSL 证书，可简单替换成自签名证书
- 定制简单，该 role 保持了简单
- 支持安装插件、重置内置用户密码（builtin） 、新增和删除 keystore

## 备份

使用快照功能（snapshot）将索引备份到 S3，如已安装 Kibana，则可在 dev tools 执行以下语句

```sh
# 创建 s3 类型的 repo
PUT _snapshot/s3repo
{
  "type" : "s3",
  "settings" : {
    "bucket" : "test",
    "path_style_access" : "true",
    "base_path" : "/es1",
    "endpoint" : "192.168.88.61:19000",
    "protocol" : "http"
  }
}

# 创建快照策略
PUT _slm/policy/daily-snapshots
{
  "name": "<daily-snap-{now/d}>",
  "schedule": "0 30 20 * * ?",
  "repository": "s3repo",
  "config": {
    "feature_states": [],
    "include_global_state": true
  },
  "retention": {
    "expire_after": "7d",
    "max_count": 7
  }
}
```

## 参考

- https://github.com/elastic/ansible-elasticsearch 参考 keystore 设计
- [在Ansible中非交互式修改elastic的密码](https://blog.csdn.net/m0_45985412/article/details/119823902) 修改 elastic 密码
- [ELK非交互修改elastic密码（转载）](https://blog.espnlol.com/?p=582)
- [X-Pack Authentication issue](https://discuss.elastic.co/t/x-pack-authentication-issue/121632) 
- [Change passwords API](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/security-api-change-password.html)
- [elasticsearch-keystore](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-keystore.html)
- [S3 repository](https://www.elastic.co/guide/en/elasticsearch/reference/current/repository-s3.html)
- [Secure settings](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/secure-settings.html)
- [How to add user and group without a password using Ansible?](https://stackoverflow.com/questions/36290485/how-to-add-user-and-group-without-a-password-using-ansible) 创建程序用户
