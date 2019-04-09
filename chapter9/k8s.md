#k8s
    
    
#创建k8s集群
    方便的搭建k8s集群
        使用minikube搭建单节点k8s集群
        使用kubeadm 搭建多节点k8s集群
        使用kops    在云上搭建k8s集群
        
    1.安装minikube
    按照步骤安装好minikube
    再安装好kubectl
    
    查看版本号
    minikube version
    kubectl version      
        
    minikube start
        
    使用kubectl连接刚刚安装好的k8s集群    
        
    kubectl config view         查看安装的配置信息
    kubectl config get-contexts 查看集群
    
#pod k8s中最小的调度单位
    pod   具有相同namespace的container的组合    
        
    创建nginx的pod   注意yml中的kind关键字！！！！
    kubectl create -f pod_nginx.yml
    
    删除
    kubectl delete -f pod_nginx.yml
    kubectl get pods        查看创建的pod
    kubectl get pods -o wide 查看详细信息 ip和node
    
    minikube ssh    进入minibuke这台虚拟机里
    
    docker network inspect bridage
    (可以看到创建的nginx的容器连接到了bridage上面)
    exit        退出
    
    kubectl exec -it nginx sh
    
    kubectl describe nginx 查看nginx容器的详细信息
    
    在monikube这台机器上能访问到nginx的首页  但是外部还是不行  以后讲！！！
    
    清空之前创建的pod
    
#replicationController和replicaSet 对pod进行横向扩展


#deployment    对replication和replicaSet的声明
    kubectl get deployment -o wide
    kubectl set image deployment nginx-deployment nginx=nginx:1.13  对服务nginx升级
    
    回滚
    kubectl rollout history deployment nginx-deployment     查看
    kubectl rollout undo deployment nginx-deployment        回滚
    
#对外部暴漏端口
    kubectl expose pods nginx-pod
    
    1.clusterIp 只能内部访问
    kubectl get svc  查看服务的ClusterIp
    
    2.nodeport类型的服务可以暴漏到公网  端口范围30000-32337
    kubectl expose pods nginx-pod --type=NodePort  创建nodeport类型的services
    
    3.通常前两种都不适用  loadbalance或者externalName
    

    
 
 
#使用tectonic搭建多节点的k8s集群





#使用kops搭建k8s集群 
    首先安装好kops和aws工具
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    