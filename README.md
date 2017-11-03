# Echo Web
Go web framework Echo example. 
> Echo middleware [echo-mw](https://github.com/hb-go/echo-mw)

> Echo中文文档 [go-echo.org](http://go-echo.org/)

> Requires
- go1.8+
- Echo V3

#### 目录
- [环境配置](#环境配置)
- [运行](#运行)
- [打包](#打包)
- [目录结构](#目录结构)
- [框架功能](#框架功能)
- [框架功能](#框架功能)
- [Confd管理配置](#Confd管理配置)
- [Docker部署](#Docker部署)

## 环境配置

##### 1.源码下载
```shell
$ cd $GOPATH/src
$ git clone git@github.com:hb-go/echo-web.git
```

##### 2.依赖安装
> [dep工具安装](https://github.com/golang/dep#usage)
```shell
$ cd echo_web/
$ dep ensure
```

##### 3.MySQL配置
```shell
# ./conf/conf.toml
[database]
name = "goweb_db"
user_name = "goweb_dba"
pwd  = "123456"
host = "127.0.0.1"
port = "3306"

# 测试数据库SQL脚本
./echo-web/res/db_structure.sql
```

##### 4.Redis、Memcached配置，可选

> 可选需修改session、cache的store配置
- session_store = "FILE"或"COOKIE"
- cache_store = "IN_MEMORY"


```shell
# ./conf/conf.toml
[redis]
server = "127.0.0.1:6379"
pwd = "123456"

[memcached]
server = "localhost:11211"
```

##### 5.子域名
```shell
# ./conf/conf.toml
[server]
addr = ":8080"
domain_api = "echo.api.localhost.com"
domain_web = "echo.www.localhost.com"

# 改host
$ vi /etc/hosts
127.0.0.1       echo.api.localhost.com
127.0.0.1       echo.www.localhost.com

# Nginx配置，可选
server{
    listen       80;
    server_name  echo.www.localhost.com echo.api.localhost.com;

    charset utf-8;

    location / {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host            $http_host;

        proxy_pass http://127.0.0.1:8080;
    }
}
```

##### 6.Bindata打包工具，可选(运行可选，打包必选)
> [Bindata安装](https://github.com/jteeuwen/go-bindata#installation)

## 运行
```shell
# 首次运行必须带 -a -t，更新assets.go、template.go的资源路径为本地
$ ./run.sh [-a] [-t]        # -a -t 可选(须安装Bindata)，以debug方式更新assets、template的Bindata资源包

# 浏览器访问
http://echo.www.localhost.com      # Nginx代理
http://echo.www.localhost.com:8080 # 无代理

# OpenTracing
http://localhost:8700/traces
```

## 打包
> 打包静态资源及模板文件须[安装Bindata](https://github.com/jteeuwen/go-bindata#installation)

```shell
# 打包
$ ./build.sh 		    # 默认本机
$ ./build.sh -l		    # 打包Linux平台

# 运行
$ ./echo-web -c conf/conf.toml      # -c指定配置文件，默认conf/conf.toml
```
## 目录结构
```sh
assets          Web服务静态资源
conf            项目配置
middleware      中间件
model           模型，数据库连接&ORM
  └ orm         ORM扩展
module          模块封装
  ├ auth        Auth授权
  ├ cache       缓存
  ├ log         日志
  ├ render      渲染
  ├ session     Session
  └ tmpl        Web模板
res             项目资源
  └ db          数据
router          路由
  └ api         接口路由
    ├context    自定义Context，便于扩展API层扩展
    └router     路由
  ├ socket      socket示范
  └ web         Web路由
    ├context    自定义Context，便于扩展Webb层扩展
    └router     路由        
template        模板
  └ pongo2      pongo2模板
util            公共工具
  ├ conv        类型转换
  ├ crypt       加/解密
  └ sql         SQL
```

## 框架功能

功能 | 描述
:--- | :---
[配置](https://github.com/hb-go/echo-web/tree/master/conf) | [toml](http://github.com/BurntSushi/toml)配置文件
[子域名部署](https://github.com/hb-go/echo-web/blob/master/router/router.go) | 子域名区分模块
[缓存](https://github.com/hb-go/echo-web/blob/master/module/cache) | Redis、Memcached、Memory
[Session](https://github.com/hb-go/echo-web/blob/master/module/session) | Redis、File、Cookie，支持Flash
[ORM](https://github.com/hb-go/echo-web/tree/master/model) | Fork [gorm](http://github.com/jinzhu/gorm)，`FirstSQL`、`LastSQL`、`FindSQL`、`CountSQL`支持构造查询SQL
[查询缓存](https://github.com/hb-go/echo-web/tree/master/model/orm) | 支持`First`、`Last`、`Find`、`Count`的查询缓存
[模板](https://github.com/hb-go/echo-web/tree/master/module/render) | 支持html/template、[pongo2](http://github.com/flosch/pongo2)，模板支持打包[bindata](https://github.com/jteeuwen/go-bindata#installation)
静态 | 静态资源，支持打包[bindata](https://github.com/jteeuwen/go-bindata#installation)
安全 | CORS、CSRF、XSS、HSTS、验证码等
[监控](https://github.com/hb-go/echo-web/blob/master/middleware/opentracing/opentracing.go) | [OpenTracing](http://opentracing.io/)，如何在项目中更方便的使用还需要研究，如[ORM层](https://github.com/hb-go/echo-web/blob/master/router/web/home.go#L37)
其他 | JWT、Socket演示

目标功能 | 描述
:--- | :---
安全 | SQL注入等
日志 | 分级
多语言 | i18n

## Confd管理配置
```bash
# 安装confd
$ go get github.com/kelseyhightower/confd
```
```bash
# 将配置写入etcd，统一前缀echo-web
$ etcdctl ls --recursive --sort /echo-web
/echo-web/app
/echo-web/app/name
......
/echo-web/tmpl/suffix
/echo-web/tmpl/type

$ cd {pwd}/echo-web
$ confd -onetime -confdir conf  -backend etcd -node http://127.0.0.1:4001 -prefix echo-web
```

## Docker部署
> Dockerfile含两种方式，源码方式镜像包偏大
```bash
# 创建镜像，注意修改配置
$ docker build -t hbchen/echo-web:v0.0.1 .

# 运行容器
$ docker run  \
     -p 8080:8080 \
     --name=echo-web \
     hbchen/echo-web:v0.0.1
```

#### *MySQL、Redis、Memcached等服务配置问题
```bash
# 如果是服务在宿主机需要配置服务IP为主机IP，127.0.0.1/localhost网络不通

# hbchen/echo-web使用的配置，在宿主机host上做个映射(192.168.1.8为主机IP)
# 192.168.1.8 mysql.localhost.com
# 192.168.1.8 redis.localhost.com
# 192.168.1.8 memcached.localhost.com
[database]
host = "mysql.localhost.com"

[redis]
server = "redis.localhost.com:6379"

[memcached]
server = "memcached.localhost.com:11211"
```

##### 自动修改/etc/hosts，映射自定义域名到主机IP
```bash
$ vi ~/.profile

# 添加
grep -v "etcd.localhost.com\|consul.localhost.com\|mysql.localhost.com\|redis.localhost.com\|memcached.localhost.com" /etc/hosts > ~/hosts_temp
cat ~/hosts_temp > /etc/hosts
LC_ALL=C ifconfig en0 | grep 'inet ' | cut -d ' ' -f2 | awk '{print $1 " etcd.localhost.com\n" $1 " consul.localhost.com\n" $1 " mysql.localhost.com\n"$1     " redis.localhost.com\n" $1 " memcached.localhost.com"}' >> /etc/hosts
```

