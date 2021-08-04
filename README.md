# k3s_study

# 在Ubuntu上安装K3S

我们简单使用k3s来学习k8s

### 准备工作:

1. install docker  (我们在这里使用docker作为容器 而不是默认的ContainerD)
2. 准备mysql  (我们在这里使用mysql作为注册中心 而不是默认的Sqlite)

### 开始安装主节点:

``` 
curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -s - server --datastore-endpoint="mysql://root:root@tcp(192.168.88.99:3305)/k3s" --docker
```

### 基础配置当前普通用户可使用`kubectl`

``` 
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
$ vim ~/.bashrc
export KUBECONFIG=~/.kube/config
$ source ~/.bashrc
```

### 添加一个子节点

``` 
将kubeconfig文件写入到/etc/rancher/k3s/k3s.yaml，由 K3s 安装的 kubectl 将自动使用该文件

设置K3S_URL参数会使 K3s 以 worker 模式运行。K3s agent 将在所提供的 URL 上向监听的 K3s 服务器注册。K3S_TOKEN使用的值存储在你的服务器节点上的/var/lib/rancher/k3s/server/node-token路径下。

curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn K3S_URL=https://192.168.88.99:6443 K3S_TOKEN=K10b00d58e153c20552749a3e3d6d03d2ff1d974a1d869d96ccecd85a0e777a93b6::server:1984e3457e28d7e1869537fc1e725dc6 sh -s -agent --docker
```

### 获取服务状态

``` 
systemctl status k3s.service  // 服务端
systemctl status k3s-client.service // 客户端
```

### 卸载 K3s

``` 
/usr/local/bin/k3s-uninstall.sh   // 服务端
/usr/local/bin/k3s-agent-uninstall.sh // 客户端
```

# 基础命令

> 获取nodes

``` 
wangy@mv-ubuntu-006:~$ kubectl get nodes
NAME            STATUS   ROLES                  AGE    VERSION
ubuntu          Ready    <none>                 72m    v1.21.3+k3s1
mv-ubuntu-006   Ready    control-plane,master   164m   v1.21.3+k3s1
```

> 获取namespace

``` 
kubectl get namespace
```

> 创建一个namespace

``` 
    kubectl create namespace new_namespace
```

> 获取所有pod

``` 
    sudo k3s kubectl get pods --all-namespaces
```

> 获取特定namespaces下的pod

``` 
    kubectl get pods -n kube-system
```

> 删除namespaces 并删除 旗下所有pod

``` 
    kubectl delete ns namespaces_name
```

> 查看某个pod状态

``` 
    kubectl describe pods -n namespace_name  pod_id
```

> 查看某个pod的log

``` 
    kubectl logs -n namespace_name  pod_id
```

> 小试牛刀

``` 
        kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.16.0/cert-manager.yaml
        
        export KUBECONFIG=/etc/rancher/k3s/k3s.yaml # helm 会用到

        # 为cert-manager 创建命名空间
        kubectl create namespace cert-manager
      
        # 添加 Ketstack Helm 仓库
        helm repo add jetstack https://charts.jetstack.io
      
        # 本地更新 Helm Chart 窗口缓存
        helm repo update
      
        # 安装 cert-manager Helm chart
        helm install \
         cert-manager jetstack/cert-manager \
         --namespace cert-manager \
         --version v0.16.0

        # 查看状态
        kubectl get pods --namespace cert-manager
        # 可能拉取不下来images 替换Helm镜像  Azure镜像
        helm repo add stable https://mirror.azure.cn/kubernetes/charts/
        helm repo add incubator https://mirror.azure.cn/kubernetes/charts-incubator/
        # 查看helm源
        helm repo list
        # 删除默认源
        helm repo remove stable

        # 删除POD
        kubectl delete pod $POD_NAME --force --grace-period=0
        # 强制删除NAMESPACE
        kubectl delete namespace $NAMESPACENAME_NAME --force --grace-period=0

        # 查看pod详情
        kubectl describe pod --namespace $NAMESPACENAME_NAME $POD_NAME
        # 查看日志
        kubectl logs $POD_NAME
        # 在Pod中执行命令
        kubectl exec $POD_NAME env
        # 启动容器中的bash
        kubectl exec -ti $POD_NAME bash
```

# 部署 kubernetes-dashboard

dashboard.admin-user.yml

``` 
apiVersion: v1
kind: ServiceAccount
metadata:
name: admin-user
namespace: kubernetes-dashboard
```

dashboard.admin-user-role.yml

``` 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
name: admin-user
roleRef:
apiGroup: rbac.authorization.k8s.io
kind: ClusterRole
name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

部署`admin-user` 配置：

`kubectl create -f dashboard.admin-user.yml -f dashboard.admin-user-role.yml`

获得 `Bearer Token`

`kubectl -n kubernetes-dashboard describe secret admin-user-token | grep '^token'`

本地访问仪表板

`kubectl proxy`

``` 
现在可以通过以下网址访问仪表盘：

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
使用admin-user Bearer Token Sign In

衍生阅读 端口转发: https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/
```

部署仪表盘

``` 
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml -f dashboard.admin-user.yml -f dashboard.admin-user-role.yml
```

删除仪表板和 admin-user 配置

``` 
sudo k3s kubectl delete ns kubernetes-dashboard
```
