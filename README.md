# 项目说明
本文主要讲述如何使用 qbittorrentEE 这个软件, 来实现自动的定时定量下载资源. 仅供参考

ps: 假设我有个虚拟机: 172.18.157.205 我需要再上面配置下载程序


# 要求
- 具备定时启停功能
- 具备限速功能.
- 具备统计功能.

# 安装软件
## 找到安装包
- 找到安装包
```
https://github.com/c0re100/qBittorrent-Enhanced-Edition/releases
```
根据你的平台来选择对应的安装包, 比如是linux的正常看架构来选. 比如大部分都是x86/64架构的. 就可以选择如下的安装包
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/683a4430-f9ba-4057-83e9-326d238b7bfa)


## 下载安装
- 确定环境
  
我的虚拟机设备是: 172.18.157.205
我希望安装在: /srv/qbee 下
```bash
# 进入 /srv
cd /srv
# 下载安装包, 如果访问不了, 则通过rz/或者sz, 或者ftp将配置包传递过来也可以
wget  https://github.com/c0re100/qBittorrent-Enhanced-Edition/releases/download/release-4.6.3.10/qbittorrent-enhanced-nox_x86_64-linux-musl_static.zip
# 解压
unzip qbittorrent-enhanced-nox_x86_64-linux-musl_static.zip
```
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/eae90f68-f5c1-4383-8d5e-099fa36eed32)

- 设定自己的目录
```bash
mkdir -p qbee
mv qbittorrent-nox qbee
```

## 配置并启动
主要是不希望自己程序跑着跑着挂了, 这里选supervisor 进行管理.
没有安装过使用如下安装和使用
- 安装supervisor
```bash
# 如果没有安装过supervisor使用如下进行安装
sudo yum install supervisor 
# 设置开机自启
sudo systemctl enable supervisord
```
- 配置qbee进程
配置supervisor来管理qbee(如下我选择8090端口, 也可以自己选择别的端口)
```
vi /etc/supervisor/conf.d/qbee.conf
```
写入如下数据
```
[program:qbee]
command=/srv/qbee/qbittorrent-nox --webui-port=8090
autostart = true
startsecs = 1
autorestart = true
user=root
stdout_logfile = /srv/qbee/qbee.log
stdout_logfile_maxbytes=5MB
stdout_logfile_backups=5
stderr_logfile = /srv/qbee/qbee.err
stderr_logfile_maxbytes=5MB
stderr_logfile_backups=5
```
- 启动
```bash
# 加载到进程中
supervisorctl update qbee
# 查看是否启动ok
supervisorctl status qbee
#关闭用 supervisorctl stop qbee
```
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/fb3fd895-306d-484f-b0d0-e46b00ce9a44)


- 查看日志获取临时密码
cat /srv/qbee/qbee.log
  ![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/58ca9cb6-3aaa-46f2-afdd-2b04c8b2412c)
临时密码: Sj64J5y6f
默认账号是admin


## 进入web程序
http://172.18.157.205:8090
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/e49c6fd7-7dc9-40a0-8ca3-73d526c7f924)

## 配置中文(方便管理)
Tools->options->behavior->Language->简体中文->save

![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/502bfc18-bf52-4d32-bb19-84fbe4361825)

## 修改密码
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/d7e82f20-9c85-4af1-9671-6b9b1b5749a4)


## 修改tracker提升下载速度
正常到这个步骤之后就可以开始下载了, 但是这样下载效率很低, 为了更快的下载, 我们需要配置tracker以提升下载速度.
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/955906ec-1cc4-4c06-a2be-2c3d05f48db3)
保存并重新打开可看到

![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/4f05b1b6-4dbe-4d64-b006-141424a049cf)

## 配置上下行,防止吸血
为了防止被吸血, 我们需要修改上传速度.同时我们不希望白天影响自己上网, 希望在夜间下载东西, 那就按照下放进行修改
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/9017e062-4248-431d-9c5d-4419a852f0a3)

## (选做) 配置监听磁力链接
有时候我们希望我们远程的把某个磁力链接放到对应目录下时,就自动的开启下载.
mkdir -p /srv/qbee/auto_download
配置如下监控目录, 并保存
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/f404a263-e65b-4279-8553-1520d4b1a2b8)

## 进行下载测试
上述都ok之后我们便可以开启下载
首先我们需要找到磁力资源(磁力网站比较多, 自己上网搜一搜)
比如我们找到电影加勒比海盗的链接
把磁力链接上传到目录
/srv/qbee/auto_download
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/9ec18ddd-6268-43e9-bfd8-1d273b30a7f4)
我们会看到自动消失
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/98ba8bab-66be-4c21-9a73-2e23940d0d1d)
然后我们查看我们的下载(目前下载太慢主要是因为上面限制了10k)
 ![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/fb1f9fe0-4123-4ddc-afff-36f64e6ba0eb)
如果想要立即下载 开放10k限制 -> 1024K
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/9e75a010-0844-4591-bcc4-9664c2f655cd)
下载速度很不错
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/184d4434-55de-4de4-9e3c-23bdc13dcb60)


## 查看统计消耗
有时候需要统计自己下载了多少. 以防用超流量
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/2ac8a11c-56c4-4784-8d30-5c6618702de7)


## 查看下载后文件
由于配置自动下载, 则会直接下载在配置的目录
/srv/qbee/auto_download
![image](https://github.com/linkellymcwq/usage_qbEE/assets/167314308/5480261d-e0b8-4cad-a37d-8e2ac44267b9)































