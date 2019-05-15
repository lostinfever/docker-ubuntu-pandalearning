## docker-ubuntu-pandalearning
A ubuntu image for aarch64 deviece to autorun pandalearning

本镜像基于DockerHub提供的官方ubuntu创建，本人只在aarch64架构的斐讯N1设备上运行通过，理论上适用于aarch64架构的其他设备。

测试设备：斐讯 N1

测试环境：荒野无灯提供的小钢炮0327系统：https://www.right.com.cn/forum/thread-324404-1-1.html

实现功能：
通过运行Panda-Learning源码实现XueXiQiangGuo每日自动化刷满25分，通过crontab也可实现多用户运行

Panda-Learning项目主页：https://github.com/Alivon/Panda-Learning

DockerHub页面：https://hub.docker.com/r/lostinfever/aarch64-ubuntu-pandalearning

使用指南：

**1、小钢炮系统启用docker功能;**

**2、在Docker里创建一个虚拟网段：**
```
ip link set eth0 promisc on
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 macnet
（其中subnet填写网关所在的网段，gateway填写网关地址） 
```

**3、下载并启动容器**
```
docker run -it --name ubuntu-pandalearning --restart always --network macnet --ip 192.168.1.2 lostinfever/aarch64-ubuntu-pandalearning:latest  
（--ip可以指定容器的IP，使用macvlan虚拟网段，容器可以获得一个和宿主机同级的IP，局域网内可以直接访问所有端口，省去端口映射的麻烦）
```

**4、SSH登录（SSH默认用户名root/admin,端口：22）刚刚创建的ubuntu系统，测试运行Panda-Learning源码是否正常：**
```
cd /media/pdlearning && Python3 -u pandalearning.py
输入你的强国用户名密码并保存，能显示自己的分数即为正常运行
```

**5、创建自动运行shell脚本：**

/media/pdlearn/pdlearn.sh

```
#!/usr/bin/env bash
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

cd /media/pdlearning/ && 

sleep $(($RANDOM%600+600)) &&        
#脚本执行后，随机等待10-20分钟开始执行

./pandalearning-aarch64 user1>> $(date "+%Y%m%d%H%M%S")-user1.log 2>&1 &&

sleep $(($RANDOM%600+120)) &&    
#每个任务运行间隔2-12分钟

./pandalearning-aarch64 user2>> $(date "+%Y%m%d%H%M%S")-user2.log 2>&1 &&

sleep $(($RANDOM%600+120)) &&

./pandalearning-aarch64 user3>> $(date "+%Y%m%d%H%M%S")-user3.log 2>&1 &&
```    

修改user1或user2为你自己标记的用户名，每个任务运行后都会在/media/pdlearning/下生成一个(当前date-名字.log)的日志

设置crontab，让shell脚本每天定时开始：
``` 
crontab -e 

0 6 * * * nohup bash /media/pdlearn/pdlearn.sh &
#每天6:00启动脚本并在后台运行

配置完后，Ctrl+O保存,Ctrl+X退出；

service cron reload && service cron restart
#重载配置并cron服务
``` 

## 备注：
* 加入随机等待，避免被监测到（实际上，用户基数这么大，基本上没可能去监测这个的）

* 有多个用户时，需要每个用户先手动运行保存用户信息，或复制其他平台上的user目录到/media/pdlearning/导入以前的用户信息

**免责声明：
本镜像基于Github上的开源非盈利项目Panda Learning，仅作为程序员之间相互学习交流之用，使用需严格遵守开源许可协议，禁止使用本镜像进行任何盈利活动。对一切非法使用所产生的后果，本人概不负责**
