
```shell
# 初始化
kubeadm init --image-repository registry.aliyuncs.com/google_containers  \
--apiserver-advertise-address=192.168.66.19 \
--pod-network-cidr=10.244.0.0/16 \
--service-cidr=10.96.0.0/12
kubectl apply -f kube-flannel.yml
# 将master节点作为node节点使用
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

```shell
# 删除控制层面
kubectl drain master --delete-emptydir-data --force --ignore-daemonsets
kubectl delete daemonset --all -n kube-system
kubectl delete node master
```

```shell
# 创建Nginx service
kubectl expose deployment nginx --port=80 --target-port=32380 --type=NodePort

# 创建Nginx的pod controller
kubectl create deployment nginx --image=nginx:1.14 --port=80 --replicas=2 
```







