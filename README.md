# docker-ubuntu-pandalearning
A ubuntu image for aarch64 deviecepandalearning

本镜像基于DockerHub提供的官方ubuntu创建，本人只在aarch64架构的斐讯N1设备上运行通过，理论上适用于aarch64架构的其他设备。

测试设备：斐讯 N1

测试环境：荒野无灯提供的小钢炮0327系统：https://www.right.com.cn/forum/thread-324404-1-1.html

实现功能：
通过运行Panda-Learning源码实现XueXiQiangGuo每日自动化刷满25分，通过crontab也可实现多用户运行;

Panda-Learning项目主页：https://github.com/Alivon/Panda-Learning

使用指南：

1. 小钢炮系统启用docker功能;

2. 在Docker里创建一个虚拟网段：

    ip link set eth0 promisc on 
    
    docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 macnet    
    （其中subnet填写网关所在的网段，gateway填写网关地址） 

3. 下载并启动容器 

    docker run -it --name ubuntu-pandalearning --restart always --network macnet --ip 192.168.1.2 lostinfever/aarch64-ubuntu-pandalearning:latest
    
    （--ip可以指定容器的IP，使用macvlan虚拟网段，容器可以获得一个和宿主机同级的IP，局域网内可以直接访问所有端口，省去端口映射的麻烦）

4. SSH登录（root/admin）刚刚创建的ubuntu系统，测试运行Panda-Learning源码是否正常：

    cd /media/pdlearning && Python3 -u pandalearning.py

    输入你的强国用户名密码并保存，能显示自己的分数即为正常运行

5. 输入 crontab -e 编辑crontab内容：

    0 5 * * * cd /media/download/pdlearning/ && python3 -u pandalearning.py user1>> user1.log 2>&1
    
    #每天5:00开启user1的学习，并输出日志到 /media/download/pdlearning/user1.log

    25 5 * * * cd /media/download/pdlearning/ && python3 -u pandalearning.py user2 >> user2.log 2>&1
    
    #每天5:25开启user2的学习，并输出日志到 /media/download/pdlearning/user2.log
    
    修改user1或user2为你自己标记的用户名，“Ctrl + O”写入修改内容，“Ctrl + X”退出

    重启cron任务： service cron reload && service cron restart

备注：
 一般每个任务20分钟左右可以学习完成，由于开启多线程，所以耗费内存较大，单个任务内存消耗在500mb左右；
有多个用户时，需要每个用户先手动运行保存用户信息，或复制其他平台上的user目录到/media/download/pdlearning/导入以前的用户信息
多用户最好设置25分钟以上的间隔，避免占用过多内存影响宿主机的运行

免责声明：
本镜像基于Github上的开源非盈利项目Panda Learning，仅作为程序员之间相互学习交流之用，使用需严格遵守开源许可协议，禁止使用本镜像进行任何盈利活动。对一切非法使用所产生的后果，本人概不负责。
