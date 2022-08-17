# NFT数字艺术藏品系统

> 目前市场上最好的数字藏品系统，已有十几家平台使用，用户量超过百万，安全性得到有效验证。联系交流：qq:236056471

## 界面截图
<img src="./screenshot/demo.jpeg">


## NFT数字艺术藏品安装说明

本次操作均在Linux Debian系统下进行，如您使用其他操作系统，请修改相关操作指令。

## 系统配置
- 操作系统: Linux Debian
- 数据库: mysql8
- php版本：8.0
- php框架：laravel9
- redis
- nginx

## 系统功能
- 首发+寄售+转增+铸造+空投+盲盒+合成+发售日历+专辑+邀请注册+排行榜+积分
- 免费接入微信、支付宝、连连支付、杉德支付(可对接其他三方支付)
- 支持私链、文昌链或者可接入其他链
- 支持h5+app
- 支持白名单优先购、注册送空投，或者满足邀请人数送购买资格

## 安装

### 1、PHP设置
- 删除禁用的函数：`putenv` `proc_open` `proc_get_status` `pcntl_alarm` `pcntl_signal` `symlink`
- 安装扩展：redis、fileinfo

### 2、安装步骤

```
1、下载项目
git clone [联系qq 236056471获取代码]

2、使用composer安装
composer install

3、将根目录的.env.example文件复制一份并改名为.env
cp .env.example .env

4、建立好数据库，并在根目录.env文件里配置好数据库账号密码
DB_DATABASE=nft
DB_USERNAME=root
DB_PASSWORD=root

5、创建APP KEY
php artisan key:generate

6、生成 JWT secret key
php artisan jwt:secret

7、执行数据库迁移并创建默认数据
php artisan migrate --seed
```

## 支付配置

### 微信支付配置
修改.env文件里的如下配置项
```
WECHAT_MCH_ID=商户号
WECHAT_MCH_SECRET_KET=商户秘钥
WECHAT_MP_APP_ID=公众号的app_id
```
证书放置在项目根目录`/cert/wechat`文件夹里，文件命名如下:
```
# 商户私钥
apiclient_key.pem

# 商户公钥证书
apiclient_cert.pem
```

### 支付宝配置
修改.env文件里的如下配置项
```
ALIPAY_APP_ID=支付宝分配的 app_id
ALIPAY_APP_SECRET_CERT=应用私钥
```
证书放置在项目根目录`/cert/alipay`文件夹里，文件命名如下:
```
# 应用公钥证书
appCertPublicKey.crt

# 支付宝公钥证书
alipayCertPublicKey_RSA2.crt

# 支付宝根证书
alipayRootCert.crt
```

## 其他配置
以下配置均在后台自行设置

- 储存，支持阿里云OSS和腾讯云COS或者是本地储存（使用本地储存时需要执行 `php artisan storage:link` 创建软连接）
- 短信，支持阿里云和腾讯云的短信服务
- 验证码，可关闭验证码或者使用腾讯滑块验证码
- 实名认证，支持阿里云市场的实名认证接口和悦保平台的认证接口

## 启动任务队列
使用 `php artisan queue:work` 命令来启动队列处理相关业务，可按照下面的「Supervisor 部署」来管理和守护进程。

## Supervisor 部署
[Supervisor](http://www.supervisord.org/) 是 `Linux/Unix` 系统下的一个进程管理工具。可以很方便的监听、启动、停止和重启一个或多个进程。通过 [Supervisor](http://www.supervisord.org/) 管理的进程，当进程意外被 `Kill` 时，[Supervisor](http://www.supervisord.org/) 会自动将它重启，可以很方便地做到进程自动恢复的目的，而无需自己编写 `shell` 脚本来管理进程。

### 安装 Supervisor
```
# Centos
yum install -y supervisor

# Debian
apt-get install supervisor
```

### 创建配置文件
在`/etc/supervisor/conf.d/`目录下创建一个 `supervisord.conf` 文件，内容如下：

```
# 新建一个应用并设置一个名称，这里设置为 laravel
[program:laravel]

# 进程名字
process_name=%(program_name)s_%(process_num)02d

# 进程数量(最大cpu数*4)
numprocs=1

# 设置命令在指定的目录内执行
directory=/您的网站目录

# 这里为您要管理的项目的启动命令
command=php artisan queue:work

# 以哪个用户来运行该进程
user=www

# supervisor 启动时自动该应用
autostart=true

# 进程退出后自动重启进程
autorestart=true

# 进程持续运行多久才认为是启动成功
startsecs=1

# 重试次数
startretries=3

# stderr 日志输出位置
stderr_logfile=/var/log/stderr.log

# stdout 日志输出位置
stdout_logfile=/var/log/stdout.log
```

### 启动 Supervisor
运行下面的命令基于配置文件启动 Supervisor 程序：
```
supervisord -c /etc/supervisor/supervisord.conf
```
如果遇到 `Another program is already listening on a port that one of our HTTP servers is configured to use` 报错，说明已经启动了，先 `kill` 掉进程。

```
# 查找进程
ps aux | grep supervisord

# 结束进程
kill -9 进程ID
```

### 使用 supervisorctl 管理项目
```
# 启动 laravel 应用
supervisorctl start laravel

# 重启 laravel 应用，多进程可在后面加上all重启所有进程
supervisorctl restart laravel [all]

# 停止 laravel 应用
supervisorctl stop laravel  

# 查看所有被管理项目运行状态
supervisorctl status

# 重新加载配置文件(修改过配置文件需执行此命令重载配置，否则不生效)
supervisorctl update

# 重新启动所有程序
supervisorctl reload
```

## 任务调度
使用 `crontab` 来定期执行任务，新增如下一条配置：

```
* * * * * cd /你的项目路径 && php artisan schedule:run >> /dev/null 2>&1
```

如果使用宝塔可以在计划任务里新增一条任务类型为`Shell脚本`，执行周期为 `1分钟`的任务，脚本内容如下:
```
cd /你的项目路径 && php artisan schedule:run
```
### 常用命令
```
# 查看任务列表
crontab -l

# 添加任务
crontab -e

# 查看状态
/etc/init.d/cron status

# 重载配置
/etc/init.d/cron reload

# 重启服务
/etc/init.d/cron restart
```
