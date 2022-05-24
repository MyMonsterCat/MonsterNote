## Nacos集成Seata1.4.1环境搭建

话不多说，直接开整

## 一、创建seata 数据库

数据库脚本如下

```sql
-- the table to store GlobalSession data
drop table if exists `global_table`;
create table `global_table` (
  `xid` varchar(128)  not null,
  `transaction_id` bigint,
  `status` tinyint not null,
  `application_id` varchar(32),
  `transaction_service_group` varchar(32),
  `transaction_name` varchar(128),
  `timeout` int,
  `begin_time` bigint,
  `application_data` varchar(2000),
  `gmt_create` datetime,
  `gmt_modified` datetime,
  primary key (`xid`),
  key `idx_gmt_modified_status` (`gmt_modified`, `status`),
  key `idx_transaction_id` (`transaction_id`)
);

-- the table to store BranchSession data
drop table if exists `branch_table`;
create table `branch_table` (
  `branch_id` bigint not null,
  `xid` varchar(128) not null,
  `transaction_id` bigint ,
  `resource_group_id` varchar(32),
  `resource_id` varchar(256) ,
  `lock_key` varchar(128) ,
  `branch_type` varchar(8) ,
  `status` tinyint,
  `client_id` varchar(64),
  `application_data` varchar(2000),
  `gmt_create` datetime,
  `gmt_modified` datetime,
  primary key (`branch_id`),
  key `idx_xid` (`xid`)
);

-- the table to store lock data
drop table if exists `lock_table`;
create table `lock_table` (
  `row_key` varchar(128) not null,
  `xid` varchar(96),
  `transaction_id` long ,
  `branch_id` long,
  `resource_id` varchar(256) ,
  `table_name` varchar(32) ,
  `pk` varchar(36) ,
  `gmt_create` datetime ,
  `gmt_modified` datetime,
  primary key(`row_key`)
);
```

## 二、CentOS7安装配置Seata

我使用的版本为1.4.1，[前往下载最新版seata](https://github.com/seata/seata/releases)

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/%E5%BE%AE%E6%9C%8D%E5%8A%A1/seata/%E5%AE%89%E8%A3%85%E5%8C%85.jpg)

上传到服务器中，解压

```clike
tar -zxvf  seata-server-1.4.1.tar.gz
```

修改 seata/conf 目录中的 file.conf

```clike
store {
  ## store mode: file、db、redis
  mode = "file" #填写自己需要的mode
      
  db {
    datasource = "druid"
    dbType = "mysql"
    driverClassName = "com.mysql.cj.jdbc.Driver"
    url = "jdbc:mysql://XXX:3306/seata" #此处填写seata数据库ip
    user = "账户" #填写账户
    password = "密码" #填写密码
    minConn = 5
    maxConn = 100
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }
}
```

修改 seata/conf 目录中的 registry.conf

```clike
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos" #修改为自己需要的
  loadBalance = "RandomLoadBalance"
  loadBalanceVirtualNodes = 10

  nacos {
    application = "seata-server" #注册到nacos的seata服务名称，默认即可
    serverAddr = "xxx:8848" #填写nacos的ip
    group = "SEATA_GROUP" # seata 在 nacos上注册服务的分组
    namespace = "test" # nacos 命名空间ID（如果使用默认的 public 命名空间，可以注掉这行）
    cluster = "default"
    username = "nacos" #nacos验证，没有开启就可以注掉
    password = "SkipCloud" #nacos验证，没有开启就可以注掉
  }

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "nacos"

  nacos {
    serverAddr = "127.0.0.1:8848" #同上
    namespace = "test" #同上
    group = "SEATA_GROUP" #同上
    username = "nacos" #同上
    password = "SkipCloud" #同上
  }
}
```

将下面的脚本保存为 seata/conf/nacos-config.sh 文件（[出处：官方github](https://github.com/seata/seata/blob/develop/script/config-center/nacos/nacos-config.sh)）

```shell
while getopts ":h:p:g:t:u:w:" opt
do
  case $opt in
  h)
    host=$OPTARG
    ;;
  p)
    port=$OPTARG
    ;;
  g)
    group=$OPTARG
    ;;
  t)
    tenant=$OPTARG
    ;;
  u)
    username=$OPTARG
    ;;
  w)
    password=$OPTARG
    ;;
  ?)
    echo " USAGE OPTION: $0 [-h host] [-p port] [-g group] [-t tenant] [-u username] [-w password] "
    exit 1
    ;;
  esac
done

if [[ -z ${host} ]]; then
    host=localhost
fi
if [[ -z ${port} ]]; then
    port=8848
fi
if [[ -z ${group} ]]; then
    group="SEATA_GROUP"
fi
if [[ -z ${tenant} ]]; then
    tenant=""
fi
if [[ -z ${username} ]]; then
    username=""
fi
if [[ -z ${password} ]]; then
    password=""
fi

nacosAddr=$host:$port
contentType="content-type:application/json;charset=UTF-8"

echo "set nacosAddr=$nacosAddr"
echo "set group=$group"

failCount=0
tempLog=$(mktemp -u)
function addConfig() {
  curl -X POST -H "${contentType}" "http://$nacosAddr/nacos/v1/cs/configs?dataId=$1&group=$group&content=$2&tenant=$tenant&username=$username&password=$password" >"${tempLog}" 2>/dev/null
  if [[ -z $(cat "${tempLog}") ]]; then
    echo " Please check the cluster status. "
    exit 1
  fi
  if [[ $(cat "${tempLog}") =~ "true" ]]; then
    echo "Set $1=$2 successfully "
  else
    echo "Set $1=$2 failure "
    (( failCount++ ))
  fi
}

count=0
for line in $(cat $(dirname "$PWD")/config.txt | sed s/[[:space:]]//g); do
  (( count++ ))
    key=${line%%=*}
    value=${line#*=}
    addConfig "${key}" "${value}"
done

echo "========================================================================="
echo " Complete initialization parameters,  total-count:$count ,  failure-count:$failCount "
echo "========================================================================="

if [[ ${failCount} -eq 0 ]]; then
    echo " Init nacos config finished, please start seata-server. "
else
    echo " init nacos config fail. "
fi
```

在 seata 目录下创建 logs 目录及其内部文件

```shell
mkdir logs
cd logs
ouch seata_gc.log
```

将下面的配置保存为 seata/config.txt 文件

```clike
service.vgroupMapping.seata_tx_group=default
```

进入seata/conf目录，执行 nacos-config.sh 脚本将 config.txt 的配置信息推送到nacos上，执行命令如下

```clike
nacos-config.sh -h nacos地址 -p 8848 -u nacos -w SkipCloud -g SEATA_GROUP -t test
```

> 注：
>
> -h: 注册到注册中心的ip
>
> -p: Server rpc 监听端口   
>
> -g: 指定配置的分组
>
> -t: 指定命名空间id
>
> -m: 全局事务会话信息存储模式，file、db、redis，优先读取启动参数 (Seata-Server 1.3及以上版本支持redis)    
>
> -n: Server node，多个Server时，需区分各自节点，用于生成不同区间的transactionId，以免冲突    
>
> -e: 多环境配置参考 http://seata.io/en-us/docs/ops/multi-configuration-isolation.html

执行完成后nacos上会看到如下：

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/%E5%BE%AE%E6%9C%8D%E5%8A%A1/seata/%E9%85%8D%E7%BD%AE%E5%AE%8C%E6%88%90.png)

进入seata/bin目录启动seata服务，启动命令如下

```clike
./seata-server.sh -h 192.168.10.15 -p 8091 -m file > nacos-connect.log 2>&1 &
```

启动成功后，seata服务会注册到nacos上，在nacos的服务列表会出现seata-server

## 三、使用

### 1、添加依赖

```xml
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-all</artifactId>
    <version>$1.4.1</version>
</dependency>
```

### 2、 yml 中增加 Seata 相关的配置

```yaml
seata:
  enabled: true
  enable-auto-data-source-proxy: true
  tx-service-group: my_test_tx_group
  registry:
    type: nacos
    nacos:
      application: seata-server 
      server-addr: 192.168.1.132:8848　　　　　　　　　　# nacos服务地址
      namespace: 2275314e-9569-47f8-b4f7-65317cdf3025　　# 命名空间ID
  #      username: nacos
  #      password: nacos
config:
    type: nacos
    nacos:
      server-addr: 192.168.1.132:8848　　　　　　　　　　# nacos服务地址
      group: SEATA_GROUP
      #      username: nacos
      #      password: nacos
namespace: 2275314e-9569-47f8-b4f7-65317cdf3025　　# 命名空间ID
  service:
    vgroup-mapping:
      my_test_tx_group: default
    disable-global-transaction: false
  client:
    rm:
      report-success-enable: false
```

### 3、业务数据库undo_log建表

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

DROP TABLE IF EXISTS `undo_log`;
CREATE TABLE `undo_log`  (
  `branch_id` bigint(0) NOT NULL COMMENT 'branch transaction id',
  `xid` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT 'global transaction id',
  `context` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT 'undo_log context,such as serialization',
  `rollback_info` longblob NOT NULL COMMENT 'rollback info',
  `log_status` int(0) NOT NULL COMMENT '0:normal status,1:defense status',
  `log_created` datetime(6) NOT NULL COMMENT 'create datetime',
  `log_modified` datetime(6) NOT NULL COMMENT 'modify datetime',
  UNIQUE INDEX `ux_undo_log`(`xid`, `branch_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = 'AT transaction mode undo table' ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;
```

### 4、使用分布式事务

```java
在对应的方法上加上注解 @GlobalTransactional 即可
```

