项目地址：https://github.com/MoonTechLab/LunaTV（新增授权码，配置文件已经更新，授权码从项目官方渠道TG机器人获取：https://t.me/moontv_auth_bot）

1.更新系统软件包

apt update && apt upgrade -y

2.安装必要的工具

apt install -y curl nano

3.安装 Docker 和 Docker Compose

安装Docker：

curl -fsSL https://get.docker.com -o get-docker.sh

sh get-docker.sh

启动Docker并设置开机自启：

systemctl start docker
systemctl enable docker

安装Docker Compose：

curl -L "https://github.com/docker/compose/releases/download/v2.27.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

4.验证安装：

docker --version
docker-compose --version
两行命令都输出了版本号，说明安装成功！

5.创建项目并配置 Docker Compose

创建一个项目目录：

mkdir -p /opt/moontv

进入创建的moontv文件夹：cd /opt/moontv

创建并编辑 Docker Compose 配置文件：

nano docker-compose.yml

配置文件（博客的docker-compose.yml配置文件已经更新，只添加一个参数AUTH_TOKEN，token获取通过他们项目的TG机器人获取，其他不变，按视频开始一步一步操作就可以成功）：
```
# Docker Compose 配置文件
# 定义所有要运行的服务/容器

services:
  # MoonTV 主应用服务 - 您的影视库
  moontv-core:
    # 使用的 Docker 镜像地址 (GitHub Container Registry)
    image: ghcr.io/moontechlab/lunatv:latest
    # 容器名称 (便于管理)
    container_name: moontv-core
    # 重启策略: 除非手动停止，否则总是重启
    restart: unless-stopped
    # 端口映射: 主机端口:容器内部端口
    ports:
      - "3000:3000"  # 将容器内3000端口映射到主机的3000端口，左边是服务器端口，右边是容器内部端口。如果想换端口，比如用8080，就改成 '8080:3000'
    # 环境变量配置
    environment:
      - USERNAME=apepine  # 登录后台管理页面的用户名
      - PASSWORD=1990Apepine  # 登录后台管理页面的密码(必须修改强密码!)
      - NEXT_PUBLIC_STORAGE_TYPE=redis  # 指定使用Redis作为存储后端
      - REDIS_URL=redis://moontv-redis:6379  # Redis连接地址(使用服务名moontv-redis)
      - AUTH_TOKEN=授权码  # 新增授权码 请从自助授权机器人处申请（看官方项目那里有）
    # 网络配置: 连接到自定义网络
    networks:
      - moontv-network
    # 依赖关系: 确保先启动Redis服务
    depends_on:
      - moontv-redis

  # Redis 数据库服务 (用于存储用户数据、收藏等)
  moontv-redis:
    # 使用官方Redis Alpine镜像 (轻量级)
    image: redis:alpine
    # 容器名称
    container_name: moontv-redis
    # 重启策略
    restart: unless-stopped
    # 网络配置
    networks:
      - moontv-network
    # 数据卷映射: 将容器内/data目录映射到宿主机的./data目录
    volumes:
      - "./data:/data"  # 实现数据持久化，防止容器重启后数据丢失

  # Watchtower 自动更新服务
  watchtower:
    # 使用官方Watchtower镜像
    image: containrrr/watchtower
    # 容器名称
    container_name: moontv-watchtower
    # 重启策略
    restart: unless-stopped
    # 卷映射: 赋予Watchtower访问Docker守护进程的权限
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"  # 关键配置: 允许Watchtower控制Docker
    # 环境变量配置
    environment:
      - "WATCHTOWER_CLEANUP=true"  # 更新后自动删除旧镜像，节省磁盘空间
      - "WATCHTOWER_POLL_INTERVAL=3600"  # 检查更新间隔(秒)，3600=1小时

  # Nginx Proxy Manager - 反向代理和SSL证书管理
  nginx-proxy-manager:
    # 使用官方Nginx Proxy Manager镜像
    image: 'docker.io/jc21/nginx-proxy-manager:latest'
    # 容器名称
    container_name: nginx-proxy-manager
    # 重启策略
    restart: unless-stopped
    # 端口映射
    ports:
      - '80:80'    # HTTP 端口
      - '81:81'    # 管理界面端口
      - '443:443'  # HTTPS 端口
    # 数据卷映射
    volumes:
      - ./nginx-data:/data  # 配置数据持久化
      - ./nginx-letsencrypt:/etc/letsencrypt  # SSL证书持久化
    # 网络配置: 连接到自定义网络，以便能够访问MoonTV服务
    networks:
      - moontv-network

# 网络配置部分
networks:
  # 定义自定义网络
  moontv-network:
    driver: bridge  # 使用桥接网络模式，使容器间可以相互通信
```
6.启动 MoonTV 项目

在后台启动所有服务：
```
docker-compose up -d
```
7.查看容器运行状态：

docker-compose ps

容器的状态（STATUS）都是 Up，就说明成功了！

如果哪个容器启动失败请查找容器启动日志解决
查看容器日志（用于排查问题）
```
docker logs moontv-core
docker logs moontv-redis
docker logs moontv-watchtower
```
比如出现moontv-core启动失败，输入命令```docker logs moontv-core```
看到401问题 ，请确认授权码无误的情况下，在 tg 点击解除设备绑定后，然后输入命令：```docker restart moontv-core``` 重启容器

查看 Watchtower 日志：
```
docker-compose logs watchtower
```
打开你的浏览器，在地址栏输入：ip:端口 （ 192.3.253.163:3000 ）

8.设置这个自定义域名：

访问nginx管理面板：ip:81（ 如：192.3.253.163:81 ）

初始账号&密码：
```
Email: admin@example.com
Password: changeme
```
