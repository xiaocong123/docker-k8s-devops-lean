#创建三个节点的swarm集群
    准备三台机器  manager work1 work2
    在manager上执行
    docker swarm init --advertise-addr=192.168.205.10
    执行之后出现一段话  带着token的swarm命令 复制之后在work1 work2上运行
    docker swarm join --token SWMTKN-sdjfkfh 192.168.205.10:2377
    
    manager上查看swarm节点的状况
    docker node ls
    
    
#service的创建和维护
    创建一个service
    docker service create --name demo busybox sh -c "while true;do sleep 3600;done"
    (类似docker run)
    查看
    docker service ls
    docker service ps demo  (查看详细信息)
    docker ps (此时也可以看到正在运行的demo项目)
    
    横向扩展
    docker service scale demo=5   (扩展成5个)
    docker service ls             (查看发现replicas变成了5/5)
    docker service ps demo        (发现demo均匀的部署在了三个节点上)
    
    docker ps 去其他的机器上也能看到容器的运行情况
    
    在一个节点上删除一个容器
    docker rm -f container-id
    
    去manager节点
    docker service ls             (发现replicas变成了4/5)
    稍等一会儿 
    docker service ls             (发现replicas变成了5/5)

    swarm能确保横向的扩展的有效行  稳定性 
    
    删除service
    docker service rm demo        (其他节点的容器也会被清理)
    
#使用swarm部署一个wordpress项目

    实现跨主机通讯  使用overlaywa 网络
    manager节点创建一个overlay
    docker network create -d overlay demo   (work1和work2上也会有demo这个网络)
    
    swarm会同步网络的创建 类似etcd
    docker network ls   查看
    
    创建mysql服务 swarm中代替 docker run
    docker service create --name myslq --env MYSQL_ROOT_PASSWORD=root --env MYSQL_DATABASE=wordpress 
    --network demo --mount type=volume,source=mysql-data,destination=/var/lib/mysql/mysql mysql
    
    创建wordpress服务
    docker service create --name wordpress -p 80:80 --env WORDPRESS_DB_PASSWORD=root 
    --env WORDPRESS_DB_HOST=mysql --network demo wordpress
    
    查看服务
    docker service ps wordpress
    docker service ps mysql
    
    然后外部访问任意一个ip都可以访问到wordpress首页
    
#集群容器间的通信
    问题一
    不同主机的服务名称是可以互相访问  因为底层是有dns机制的 会去overlay网络发现并翻译找到请求的服务
    
    创建服务whoami  访问端口8000 会返回容器的hostname
    docker service create --name whoami -p 8000:8000--network demo -d jwilder/whoami
    
    创建服务busybox
    docker service vreate --name client -d --network doem busybox sh -c "while true;do sleep 3600;done"
    
    docker service ls  (查看服务)
    
    docker service ps whoami   运行在manager
    docker service ps busybox  运行在work1
    
    进入busybox容器ping whoami  发现可以ping通
    docker exec -it busybox-id sh
    ping whoami     10.0.0.7会ping到这个地址 但是这个地址不是容器whoami的ip
    
    将服务whoami扩展两个
    docker service scale whoami=2
    
    docker service ps whoami   容器部署在了work2上
    再次进入busybox容器 ping whoami  结果不变
    
    wget whoami:8000
    more index.html      
    wget whoami:8000
    more index.html     访问两次发现访问的主机是不同的  这是应为底层做的负载均衡
    
    在manager节点 和work1上分别执行
    curl 127.0.0.1:8000  输出了节点信息并且负载均衡
    
    这就是swarm 的ingress负载均衡
    
    查看实现原理
    sudo iptable -nL -t nat   查看本地网络的转发法规则
    (发现8000端口会转发到一个ip)
    
    docker network inspect docker_gwbridge
    发现了跟docker_gwbridge 相连的container有两个  其中一个就是8000端口转发的ip
    
#在swarm集群中 使用docker stack 和 dockercompose部署wordpress项目

    首先改造wordpress 的 docker-compose
    
    docker stack deploy wordpress --compose-file=docker-compose.yml
    docker stack deploy wordpress -c=docker-compose.yml
    
    docker stack ls 
    
    docker stack ps wordpress
    
    查看服务情况
    docker stack services wordpress
    
    访问集群中的任意一个ip就可以了
    
    network在swarm docker-compose中默认是overlay网络
    
    stack 时只能拉取镜像 不能build dockerfile
    
#DockerSecret管理和使用
    创建secret两种方式
    1.
        vim password   123456
        docker secret create my-pw password
    2.
        echo "123456" | docker secret create my-pw2 -
    
        docker secret ls
        
        docker secret rm my-pw2
    
    使用  将密码传入使用的容器
    docker service create --name client --secret my-pw busybox sh -c "while true;do sleep 3600;done"
    
    docker exet -it container-id sh
    
    ls /run/secrets/       就能查看到传入的密码的原文
    
    测试使用传入的密码
    docker service create --name db secret my-pw -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/my-pw mysql
    
#DockerSecret在stack docker-compose中的使用
        
    查看secret-example和wordpress两个文件中的区别
    
    
#service的更新
    第一步 对服务横向扩展
    docker service scale web=2
    
    在其中一台机器上访问返回版本信息的端口
    sh -c "while true;do curl 127.0.0.1:8080 && sleep 1;done"
    --hello docker,version 1.0
    
    更新 会发现1.0和2.0会交替出现 然后都变成2.0  两个节点上的服务会依次更新
    docker service update --image imagename:2.0 web
    
    docker service ps web  查看服务信息
    
    
    //更新端口信息 端口更新会中断服务的
    docker service update --publish-rm 8080:5000 --publish-add 8088:5000 web
     
    
    
    
    
    
    
    
    
    