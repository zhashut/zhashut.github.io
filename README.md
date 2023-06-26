# SQL 生成器项目部署

谨记：如果容器启动成功了，但是访问不到大部分是防火墙没放行端口，记得要用的端口都放行
一开始先新建两个文件夹：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687593062450-13302c6f-1ebb-43b5-b9f1-851c0709bf5e.png#averageHue=%230a0805&clientId=u95cb1498-cb95-4&from=paste&height=89&id=uf15dc25a&originHeight=111&originWidth=290&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=6531&status=done&style=none&taskId=u076d5ea4-fc21-4b13-8ddf-f650c4425da&title=&width=232)

#  1、安装 Docker
## 1. 安装
```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
## 2. 设置开机启动 docker
```shell
systemctl enable docker
```
##  3. 配置阿里云镜像
```shell
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://e2wlsmur.mirror.aliyuncs.com"],
  "iptables": true
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker
```
## 4. 启动 docker
```shell
sudo systemctl start docker
```
# 2、防火墙
**如果 FirewallD 没有运行，你首先需要启动并启用它。这是在 CentOS 系统中使用 firewalld 配置防火墙规则的方法。执行以下命令：**
## 安装 firewalld（如果尚未安装）
```
sudo yum install firewalld -y
```
## 启动并启用 firewalld
让它在系统启动时自动运行：
```
sudo systemctl start firewalld
sudo systemctl enable firewalld
```
# 3、安装 nodejs
## 下载二进制包
```shell
wget https://npmmirror.com/mirrors/node/v16.18.0/node-v16.18.0-linux-x64.tar.xz
```
## 解压
```shell
tar -xvf node-v16.18.0-linux-x64.tar.xz
```
##  建立软连接
```shell
ln -s /root/envrionment/node-v16.18.0-linux-x64/bin/node /usr/bin/node
ln -s /root/envrionment/node-v16.18.0-linux-x64/bin/npm /usr/bin/npm
```
## 测试
```shell
node -v
npm -v
```
## 安装 cnpm
```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org

建立软连接
ln -s /root/node-v16.18.0-linux-x64/bin/cnpm /usr/bin/cnpm

cnpm -v
```
## 防火墙放行端口
```
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --reload
```
# 4、安装 go
##  安装
```shell
1. 下载
wget https://dl.google.com/go/go1.19.6.linux-amd64.tar.gz

2. 解压
tar -xvf go1.19.6.linux-amd64.tar.gz

3. 配置环境变量
cd go
vim ~/.bashrc

4. 进入文件后，在末尾加
export GOROOT=/root/envrionment/go
export GOPATH=/root/projects/go
export PATH=$PATH:$GOROOT/bin:$GPPATH/bin

5.按Esc 然后 :wq - 保存退出

6. source ~/.bashrc
```
## 设置代理加速
```shell
export GOPROXY="https://goproxy.cn,https://mirrors.aliyun.com/goproxy/,https://goproxy.io,direct"

go env -w GO111MODULE=on
```
补充目录结构：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687593010499-fd1459cf-9c1d-40cf-9c3c-94fde9c4659f.png#averageHue=%230a0403&clientId=u95cb1498-cb95-4&from=paste&height=66&id=u940d54a5&originHeight=82&originWidth=814&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=8503&status=done&style=none&taskId=u784ac478-d908-496f-bf93-beee64cfae7&title=&width=651.2)
# 5、安装 MySQL
## 拉取Mysql镜像
```bash
docker pull mysql:8.0.26 ==> docker pull 镜像名称:镜像版本
```

2. docker images
- 查看所有镜像我们发现mysql已经拉取成功了！

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25801081/1652705233271-341b67ff-e68b-4dae-bfbd-463b72642671.png#averageHue=%23120a08&clientId=u715b8b57-3809-4&from=paste&id=lCvtl&originHeight=147&originWidth=698&originalType=url&ratio=1&rotation=0&showTitle=false&size=55652&status=done&style=none&taskId=ua1e9d2fe-0c97-4962-ae47-8915b3292ae&title=)
## 启动并挂载镜像
```bash
-d: 后台运行容器，也可以使用镜像id
-p 将容器的端口映射到本机的端口
-v 将主机目录挂载到容器的目录
-e 设置参数  MYSQL_ROOT_PASSWORD 指定登录密码

 docker run -d --name mysql -p 3306:3306 -v /opt/containerd/mysql/conf:/etc/mysql/conf.d -v /opt/containerd/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=<密码> mysql:8.0.26

#重启容器后自动启动
docker container update --restart=always 容器id
```
## 建数据库表
```bash
# 进入容器
docker exec -it 容器id /bin/bash

# 进入myslq
mysql -uroot -p

# 创建数据库表
-- 创建库
create database if not exists sqlfather;

-- 切换库
use sqlfather;

-- 用户表
create table if not exists user
(
    id           bigint auto_increment comment 'id' primary key,
    userName     varchar(256)                           null comment '用户昵称',
    userAccount  varchar(256)                           not null comment '账号',
    userAvatar   varchar(1024)                          null comment '用户头像',
    gender       tinyint                                null comment '性别',
    userRole     varchar(256) default 'user'            not null comment '用户角色：user/ admin',
    userPassword varchar(512)                           not null comment '密码',
    createTime   datetime     default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime   datetime     default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    isDelete     tinyint      default 0                 not null comment '是否删除',
    constraint uni_userAccount
        unique (userAccount)
)
    comment '用户';

-- 词库表
create table if not exists dict
(
    id            bigint auto_increment comment 'id' primary key,
    name          varchar(512)                       null comment '词库名称',
    content       text                               null comment '词库内容（json 数组）',
    reviewStatus  int      default 0                 not null comment '状态（0-待审核, 1-通过, 2-拒绝）',
    reviewMessage varchar(512)                       null comment '审核信息',
    userId        bigint                             not null comment '创建用户 id',
    createTime    datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime    datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    isDelete      tinyint  default 0                 not null comment '是否删除'
) comment '词库';

create index idx_name
    on dict (name);

-- 表信息表
create table if not exists table_info
(
    id            bigint auto_increment comment 'id' primary key,
    name          varchar(512)                       null comment '名称',
    content       text                               null comment '表信息（json）',
    reviewStatus  int      default 0                 not null comment '状态（0-待审核, 1-通过, 2-拒绝）',
    reviewMessage varchar(512)                       null comment '审核信息',
    userId        bigint                             not null comment '创建用户 id',
    createTime    datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime    datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    isDelete      tinyint  default 0                 not null comment '是否删除'
) comment '表信息';

create index idx_name
    on table_info (name);

-- 字段信息表
create table if not exists field_info
(
    id            bigint auto_increment comment 'id' primary key,
    name          varchar(512)                       null comment '名称',
    fieldName     varchar(512)                       null comment '字段名称',
    content       text                               null comment '字段信息（json）',
    reviewStatus  int      default 0                 not null comment '状态（0-待审核, 1-通过, 2-拒绝）',
    reviewMessage varchar(512)                       null comment '审核信息',
    userId        bigint                             not null comment '创建用户 id',
    createTime    datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime    datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    isDelete      tinyint  default 0                 not null comment '是否删除'
) comment '字段信息';

create index idx_fieldName
    on field_info (fieldName);

create index idx_name
    on field_info (name);

-- 举报表
create table if not exists report
(
    id             bigint auto_increment comment 'id' primary key,
    content        text                               not null comment '内容',
    type           int                                not null comment '举报实体类型（0-词库）',
    reportedId     bigint                             not null comment '被举报对象 id',
    reportedUserId bigint                             not null comment '被举报用户 id',
    status         int      default 0                 not null comment '状态（0-未处理, 1-已处理）',
    userId         bigint                             not null comment '创建用户 id',
    createTime     datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime     datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    isDelete       tinyint  default 0                 not null comment '是否删除'
) comment '举报';
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687590000651-451d7c15-b79e-4ede-9f5b-179965941669.png#averageHue=%23f9f7f6&clientId=u59561d62-6d44-4&from=paste&height=922&id=ue0ac5b8a&originHeight=1152&originWidth=650&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=53096&status=done&style=none&taskId=u5d19f2f1-5c78-4cc3-aa03-858c2edd975&title=&width=520)
##  建立用户并授权
```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'127.0.0.1' IDENTIFIED BY 'root' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY 'root' WITH GRANT OPTION;

-- 8.0以后的只用执行下面这条语句
grant all privileges on *.* to 'root'@'localhost';

FLUSH PRIVILEGES;
```
## 防火墙放行端口
```
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
```
## 使用 navicat 远程连接测试
如果没开放外部端口连接这里是连不上的，只给服务器内部连接，开放有被攻击的威胁
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25801081/1652705578509-4e46ca5f-6223-4433-984c-a6fb3ccf89bb.png#averageHue=%23faf9f9&clientId=u715b8b57-3809-4&from=paste&height=567&id=OeUGG&originHeight=821&originWidth=691&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24260&status=done&style=none&taskId=u57934632-e7d8-48e3-b7e8-3f3d9294ecd&title=&width=476.79998779296875)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687590073418-37301343-691c-47cb-8aa3-5caea233ac2f.png#averageHue=%232f2f2e&clientId=u59561d62-6d44-4&from=paste&height=311&id=u45a6c548&originHeight=389&originWidth=318&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=13862&status=done&style=none&taskId=u85355417-f856-4346-a7f3-03ec46ed4f0&title=&width=254.4)
# 6、安装 Redis
## 拉取 Redis 镜像
```bash
docker pull redis
```

2. 查看是否拉取成功了，以下表示拉取成功
```bash
docker images
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25801081/1652706050755-49ee3e5e-8c45-4f71-aeb3-05579e8eb124.png#averageHue=%23f0eeec&clientId=u715b8b57-3809-4&from=paste&height=106&id=IcK9M&originHeight=133&originWidth=695&originalType=binary&ratio=1&rotation=0&showTitle=false&size=13673&status=done&style=none&taskId=u403e2e34-8a0e-41da-af8f-d2c537fd973&title=&width=556)
## 运行 Redis 镜像
```bash
docker run --name redis -p 6379:6379 -d redis redis-server --appendonly yes --requirepass "自己的密码"

#  -p:端口映射
#  -d:后台运行
#  redis-server --appendonly yes:开启缓存模式，可以存储至硬盘中

docker container update --restart=always 容器id
```
## 防火墙放行端口
```
sudo firewall-cmd --zone=public --add-port=6379/tcp --permanent
sudo firewall-cmd --reload
```
## 测试连接 Reids
可以使用 **Reids Desktop Manger **图形化界面连接
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25801081/1652706210226-21cac4c7-f1e3-4345-ae8a-f69ac6c61e04.png#averageHue=%23f9f9f9&clientId=u715b8b57-3809-4&from=paste&height=603&id=BZmzN&originHeight=772&originWidth=702&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34907&status=done&style=none&taskId=u4b254c63-1fb5-4d37-a4c5-298a9266987&title=&width=548.6000366210938)
# 7、安装 git
## 更新软件包
```shell
sudo yum update
```
## 添加 EPEL
（Extra Packages for Enterprise Linux）存储库，此存储库提供额外的软件包。运行以下命令：
```shell
sudo yum install epel-release
```
## 通过 EPEL 存储库安装Git
```shell
sudo yum install git
```
## 验证Git

1. 安装完成后，查看是否已正确安装。运行以下命令：
```shell
git --version
```
如果命令返回Git的版本号，那么安装就成功了。
## 配置全局设置
```shell
1. 配置用户名
git config --global user.name "zhashut"
2. 配置邮箱
git config --global user.email "2105798885@qq.com"
3. 查看配置
git config --global --list
```
# 8、部署前端
## 准备两个配置文件
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687784561140-0fae601a-3fc1-4f2e-9168-af23d6633236.png#averageHue=%232a2e32&clientId=u6b7b656e-e649-4&from=paste&height=508&id=u33c96ceb&originHeight=635&originWidth=231&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=26963&status=done&style=none&taskId=u44fb078d-1883-4a71-b34b-bb0e3ec52a7&title=&width=184.8)
```bash
server {
    listen 80;
    server_name <服务器地址/域名>;
    # gzip config
    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 9;
    gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";

    root /usr/share/nginx/html;
    include /etc/nginx/mime.types;

    location / {
        try_files $uri /index.html;
    }

}
```
```dockerfile
FROM nginx

WORKDIR /usr/share/nginx/html/
USER root

COPY ./docker/nginx.conf /etc/nginx/conf.d/default.conf

COPY ./dist  /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```
## bulid 项目
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25801081/1652701089724-fcdadc71-dd8c-4778-b016-693f07124a17.png#averageHue=%23292d35&clientId=u715b8b57-3809-4&from=paste&height=390&id=ud6d2a0f3&originHeight=769&originWidth=1373&originalType=binary&ratio=1&rotation=0&showTitle=false&size=118206&status=done&style=none&taskId=u0fc16244-d807-4e3b-b698-72c36cff92e&title=&width=697)
## 将前端代码 copy / pull 到服务器中
### 可以使用 github 的方式

1. 将代码 push 上 gitee/github 仓库，在服务器的指定文件夹下把代码 `clone` 下来

![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687784667659-1512c874-3468-43c6-b9f5-932e3cc1d5b5.png#averageHue=%230b0806&clientId=u6b7b656e-e649-4&from=paste&height=54&id=u79aadd3a&originHeight=67&originWidth=378&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=5480&status=done&style=none&taskId=u12758d89-fcf8-41ee-b268-45bb5acba2d&title=&width=302.4)

2. 进入该文件中

![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687784681530-87551c85-d55d-48b6-9920-dd0de65f14af.png#averageHue=%230d0a07&clientId=u6b7b656e-e649-4&from=paste&height=67&id=uf9fd6c10&originHeight=84&originWidth=722&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=9800&status=done&style=none&taskId=u9fb843e7-839a-4342-a409-313abb029fb&title=&width=577.6)
### 不可以使用 github  的方式
直接在本地 copy 下面选中的代码，不需要 node_modules
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687591730970-9f550ace-207f-47ba-bad2-f8f724e153c1.png#averageHue=%23cbdec2&clientId=u9cd7c754-5a66-4&from=paste&height=600&id=u2d6e86b4&originHeight=750&originWidth=852&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=75637&status=done&style=none&taskId=u651fac4b-e071-40a4-87f7-a02ee240b49&title=&width=681.6)
在 **xftp **中进入到项目文件夹，新建一个前端的文件夹然后把刚刚复制的都 copy 进去就好了
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687591836028-b44c1f10-bbf8-486d-801f-c910050fc7be.png#averageHue=%23f3f2f1&clientId=u9cd7c754-5a66-4&from=paste&height=654&id=uced9a62b&originHeight=818&originWidth=1150&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=102020&status=done&style=none&taskId=u1141d838-b6cc-4c6e-8f21-638fd4ee8e2&title=&width=920)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687591845947-f8341108-de39-4b89-a81e-9d28d94792ab.png#averageHue=%23faf9f7&clientId=u9cd7c754-5a66-4&from=paste&height=454&id=u1800516f&originHeight=567&originWidth=575&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=37601&status=done&style=none&taskId=ub24a94f4-5c70-41e9-a8b8-bcaaceebbd9&title=&width=460)
## 制作镜像
```bash
1.根据 Dockerfile 构建镜像：
docker build -t sql-generate-frontend:v0.1 .

2.docker run 启动 -d: 在后台启动
docker run -p 8000:80 -d sql-generate-frontend:v0.1

docker container update --restart=always 容器id
```
## 防火墙放行端口
```
sudo firewall-cmd --zone=public --add-port=8000/tcp --permanent
sudo firewall-cmd --reload
```
## 访问页面
到这里如果没有问题的话，访问自己绑定域名就可以看见前端页面了
```bash
http://<服务器地址>:8000/
```
# 9、部署后端
## 1、编写 Dockerfile
```dockerfile
# 启动编译环境
FROM golang:1.19-alpine AS builder

# 配置编译环境
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64 \
    GOPROXY=https://goproxy.cn,direct \
    GOBIN=/go/bin

# 创建编译目录
WORKDIR /go/src/sql-generate

# 将代码复制到容器中
COPY go.mod .
COPY go.sum .

# 安装依赖包
RUN go mod tidy

# 将所有源文件复制到容器中
COPY . .

# 编译代码并输出到 /go/bin
RUN go build -o /go/bin/sql-generate .

# 使用 alpine 镜像
FROM alpine:3.18

# 复制构建好的可执行文件到镜像中
COPY --from=builder /go/bin/sql-generate /bin/sql-generate

# 从编译器容器中复制 config.yaml 文件到容器中
COPY --from=builder /go/src/sql-generate/config.yaml /config.yaml

# 申明暴露的端口
EXPOSE 8102

# 设置服务入口
ENTRYPOINT [ "/bin/sql-generate" ]
```
## 2、制作镜像
这里忽略把代码 copy 至服务器中了，跟前端一样的，后端是所有代码都要 copy
```dockerfile
docker build -t sql-generate -f ./Dockerfile .

docker run -p 8102:8102 -d sql-generate:latest

docker container update --restart=always 容器id
```
## 3、防火墙放行端口
```
sudo firewall-cmd --zone=public --add-port=8102/tcp --permanent
sudo firewall-cmd --reload
```
## 4、无法读取 yaml 文件需要调整的go代码
如果上面运行没什么问题这下面就不用看了
![](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687594256833-744d4afb-b513-4070-81d8-a6f598884cf7.png#averageHue=%23daac6c&from=url&height=136&id=LYBvN&originHeight=272&originWidth=742&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=371)
### 1、Cache 配置
```go
package initialize

import (
	"context"
	"flag"
	"fmt"
	"github.com/redis/go-redis/v9"
	"sql_generate/global"
)

/**
 * Created with GoLand 2022.2.3.
 * @author: 炸薯条
 * Date: 2023/6/22
 * Time: 18:28
 * Description: 初始化缓存配置
 */

var rdb = flag.Int("rdb", 0, "db")
var rPoolSize = flag.Int("pool_size", 10, "pool_size")
var rhost = flag.String("rhost", "127.0.0.1", "host")
var rpassword = flag.String("rpassword", "写自己的密码", "password")
var rport = flag.Int("rport", 6379, "port")

func InitCache() {
	flag.Parse()
	global.CaChe = redis.NewClient(&redis.Options{
		Addr:     fmt.Sprintf("%s:%d", *rhost, *rport),
		Password: *rpassword, // 密码
		DB:       *rdb,       // 数据库
		PoolSize: *rPoolSize, // 连接池大小
	})

	_, err := global.CaChe.Ping(context.Background()).Result()
	if err != nil {
		panic(err)
	}
}

```
### 2、DB 配置
```go
package initialize

import (
	"flag"
	"fmt"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
	"gorm.io/gorm/schema"
	"log"
	"os"
	"sql_generate/global"
	"time"
)

/**
 * Created with GoLand 2022.2.3.
 * @author: 炸薯条
 * Date: 2023/6/9
 * Time: 20:18
 * Description: db 全局初始化
 */

var db = flag.String("db", "sqlfather", "database name")
var host = flag.String("host", "127.0.0.1", "host")
var user = flag.String("user", "root", "user")
var password = flag.String("password", "写上自己的密码", "password")
var port = flag.Int("port", 3306, "port")

func InitDB() {
	flag.Parse()
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=utf8mb4&parseTime=True&loc=Local",
		*user, *password, *host, *port, *db)
	newLogger := logger.New(
		log.New(os.Stdout, "\r\n", log.LstdFlags),
		logger.Config{
			SlowThreshold: time.Second,
			Colorful:      true,
			LogLevel:      logger.Info,
		},
	)
	var err error
	global.DB, err = gorm.Open(mysql.Open(dsn), &gorm.Config{
		Logger: newLogger,
		NamingStrategy: schema.NamingStrategy{
			SingularTable: true, // 自动生成数据库表的时候表名不带s后缀
		},
	})
	if err != nil {
		panic(err)
	}
}

```
### 3、main 启动代码
```go
package main

import (
	"flag"
	"fmt"
	"go.uber.org/zap"
	"os"
	"os/signal"
	"sql_generate/initialize"
	"syscall"
)

/**
 * Created with GoLand 2022.2.3.
 * @author: 炸薯条
 * Date: 2023/5/14
 * Time: 22:10
 * Description: No Description
 */

var addr = flag.Int("addr", 8102, "address to listen")

func main() {
	flag.Parse()
	// 初始化配置
	initialize.InitLogger()
	//initialize.InitConfig()
	initialize.InitDB()
	initialize.InitCache()
	r := initialize.Router()

	// 启动监听端口
	zap.S().Debugf("启动服务器，端口：%d", *addr)
	go func() {
		if err := r.Run(fmt.Sprintf(":%d", *addr)); err != nil {
			zap.S().Fatal("启动失败", err.Error())
		}
	}()

	// 优雅退出
	quit := make(chan os.Signal)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit
	zap.S().Infof("服务退出成功")
}

```
# 10、打包镜像
```dockerfile
# 先登录
docker login 
```
## 1、后端镜像
```dockerfile
docker tag sql-generate:latest zhashut/sql-generate:backend

docker push zhashut/sql-generate:backend
```
## 2、前端镜像
```dockerfile
docker tag sql-generate-frontend:v0.7 zhashut/sql-generate:frontend

docker push zhashut/sql-generate:frontend
```
## 3、MySQL 镜像
```dockerfile
docker tag mysql:8.0.26 zhashut/sql-generate:mysql

docker push zhashut/sql-generate:mysql
```
## 4、Redis 镜像
```dockerfile
docker tag redis:latest zhashut/sql-generate:redis

docker push zhashut/sql-generate:redis
```
## 5、查看仓库
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25801081/1687604246524-b7bc77f2-7ab2-490b-ba00-0c076569bbda.png#averageHue=%23e1c69e&clientId=uf0f1e5c4-6d3f-4&from=paste&height=728&id=u7862da3a&originHeight=910&originWidth=1279&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=102900&status=done&style=none&taskId=u77dcc6f2-867d-44e0-867e-7778e7d0807&title=&width=1023.2)
