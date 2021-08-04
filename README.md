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
