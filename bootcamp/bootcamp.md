# bootcamp

1. 启动我们的服务

``` 
kubectl apply -f bootcamp-deployment.yaml
```

Tips:

``` 
kubectl create -f xxx.yaml （不建议使用，无法更新，必须先delete）
kubectl apply -f xxx.yaml （创建+更新，可以重复使用）

kubectl delete -f xxx.yaml  // 通过yaml 删除服务
```

2. 获取pod

``` 
$ kubectl get pods  
NAME                                      READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-v1-7946548f4f-rfk5t   1/1     Running   0          3m24s
kubernetes-bootcamp-v1-7946548f4f-w5lm8   1/1     Running   0          3m24s
```

3. 当前pods运行在default namespaces 下 我们先删除这个pod

``` 
kubectl delete -f xxx.yaml  // 通过yaml 删除服务
```

4. 创建一个namespace 来运行我们的服务

``` 
kubectl create namespace bootcamp
```

5. 指定namespaces部署服务

``` 
kubectl apply -n bootcamp -f bootcamp-deployment.yaml
```

6. 指定namespaces 查询服务

``` 
$ kubectl get pods -n bootcamp
NAME                                      READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-v1-7946548f4f-f95hd   1/1     Running   0          41s
kubernetes-bootcamp-v1-7946548f4f-wzt4m   1/1     Running   0          41s
```

7. 启动服务 

``` 
kubectl apply -n bootcamp -f bootcamp-service.yaml
```

8. 获取服务

``` 
kubectl get service
```
