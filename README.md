```bash
sudo groupadd -g 2000 denny
sudo useradd -u 2001 -m denny -g denny

systemctl disable firewalld
systemctl stop firewalld

yum install -y etcd kubernetes

systemctl start etcd
systemctl start docker.service
systemctl start kube-apiserver.service 
systemctl start kube-controller-manager.service 
systemctl start kube-scheduler.service 
systemctl start kubelet.service 
systemctl start kube-proxy.service 

https://hub.docker.com/u/kubeguide/

##mysql-rc.yaml
apiVersion: v1
kind: ReplicationController # 副本控制器RC
metadata:
  name: mysql # RC 的名称，全局唯一
spec:
  replicas: 1 # 副本期待数量
  selector: 
    app: mysql   # 符合目标的Pod拥有此标签
  template:      # 根据此模板创建Pod的副本(实例)
    metadata:
      labels:
        app: mysql  # Pod 副本拥有的标签，对应的RC的Selector
    spec:
      containers:     # Pod 内容器的定义部分
      - name: mysql   # 容器的名称
        image: mysql:5.6  # 容器对应的Docker Image
        ports:
        - containerPort: 3306   # 容器应用监听的端口号
        env:                    # 注入容器的环境变量
        - name: MYSQL_ROOT_PASSWORD   # 这里第一次写错了 MySQL_ROOT_PASSWORD
          value: "123456"

kubectl delete -f mysql-rc.yaml
kubectl delete -f mysql-svc.yaml
kubectl delete -f myweb-rc.yaml
kubectl delete -f myweb-svc.yaml

kubectl create -f mysql-rc.yaml  
kubectl create -f mysql-svc.yaml 
kubectl create -f myweb-rc.yaml  
kubectl create -f myweb-svc.yaml 

kubectl get rc
kubectl get pods

kubectl describe rc mysql
kubectl get events
###修改/etc/kubernetes/apiserver文件中KUBE_ADMISSION_CONTROL参数,去掉“ServiceAccount”选项。
##重启kube-apiserver服务
systemctl restart kube-apiserver
yum install *rhsm*       
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm
rpm2cpio python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm | cpio -iv --to-stdout ./etc/rhsm/ca/redhat-uep.pem | tee /etc/rhsm/ca/redhat-uep.pem
docker pull registry.access.redhat.com/rhel7/pod-infrastructure:latest

##mysql-svc.yaml
apiVersion: v1
kind: Service         # 表明是Kubernetes Service 
metadata:
  name: mysql         # Service 的全局唯一名称
spec:
	type: NodePort
  ports:
    - port: 3306      # Service 提供服务的端口号，这里写的时候写成了  - port:3306 没有用空格隔开
  selector:           # Service 对应的Pod 拥有这里定义的标签
    app: mysql
    
kubectl create -f mysql-svc.yaml
kubectl get svc

vi myweb-rc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myweb
spec:
  replicas: 3
  selector:
    app: myweb
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
        - name: myweb
          image: kubeguide/tomcat-app:v1
          ports:
          - containerPort: 8080

kubectl create -f myweb-rc.yaml

vi myweb-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: myweb
spec:
  type: NodePort
  ports:
    - port: 8080
      nodePort: 30001
  selector:
    app: myweb

kubectl create -f myweb-svc.yaml
##logs
tail -f /var/log/messages
docker ps
docker logs dockerid
kubectl describe pod/svc name

vi tomcat-deployment.yaml

#与RC不同之处，版本配置不同
apiVersion: extensions/v1beta1
#与RC不同之处，Kind不同
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        app: app-demo
        tier: frontend
    spec:
      containers:
      - name: tomcat-demo
        image: tomcat
# 设置资源限额，CPU通常以千分之一的CPU配额为最小单位，用m来表示。通常一个容器的CPU配额被定义为100~300m，即占用0.1~0.3个CPU；
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080

kubectl create -f tomcat-deployment.yaml 
kubectl get deployment 
kubectl get rs
kubectl describe deployment frontend
kubectl describe rs frontend
docker ps |grep  frontend

cat << 'EOF' >> /etc/systemd/system.conf
DefaultTimeoutStartSec=1800s
DefaultTimeoutStopSec=1800s
EOF

cat << EOF > /k8s/redis-master-rc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  replicas: 1
  selector:
    name: redis-master
  template:
    metadata:
      labels:
        name: redis-master
    spec:
      containers:
      - name: master
        image: kubeguide/redis-master
        ports:
        - containerPort: 6379
EOF

kubectl create -f redis-master-rc.yaml

cat << EOF > /k8s/redis-master-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  selector:
    name: redis-master
  ports:
  - port: 6379
    targetPort: 6379  
EOF
kubectl create -f redis-master-svc.yaml


cat << EOF > /k8s/redis-slave-rc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  replicas: 1
  selector:
    name: redis-slave
  template:
    metadata:
      labels:
        name: redis-slave
    spec:
      containers:
      - name: slave
        image: kubeguide/guestbook-redis-slave
        imagePullPolicy: Never
        ports:
        - containerPort: 6379
EOF

kubectl delete -f redis-slave-rc.yaml
kubectl create -f redis-slave-rc.yaml

cat << EOF > /k8s/redis-slave-svc.yaml     
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  selector:
    name: redis-slave
  ports:
  - port: 6379
EOF
kubectl delete -f redis-slave-svc.yaml  
kubectl create -f redis-slave-svc.yaml  

docker  exec -it d5ca8eced40a /bin/bash
kubectl exec -it redis-slave-v1h3d -- /bin/bash

kubectl delete -f redis-slave-rc.yaml
kubectl delete -f redis-slave-svc.yaml
kubectl delete -f redis-master-rc.yaml
kubectl delete -f redis-master-svc.yaml


kubectl create -f redis-master-rc.yaml
kubectl create -f redis-master-svc.yaml
kubectl create -f redis-slave-rc.yaml
kubectl create -f redis-slave-svc.yaml

kubectl scale rc redis-slave --replicas=3
---git
yum install -y git
ssh-keygen -t rsa -b 4096 -C "123213738@qq.com"
more /root/.ssh/id_rsa.pub
cd ~/.ssh/
vi config
Host github.com  
User 123213738@xx.com  
Hostname ssh.github.com  
PreferredAuthentications publickey  
IdentityFile ~/.ssh/id_rsa  
Port 443
ssh -T git@github.com
git clone git@github.com:123213738/examples.git

```
