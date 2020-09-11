# Reloader
这个项目叫做 Reloader，它可以监控 Configmap/Secret 的变化，根据 Annotation 选择 Deployment，对相关 Deployment 进行滚动更新。

简单工具的安装还是很简单的：

kubectl apply -f \
https://raw.githubusercontent.com/stakater/Reloader/master/deployments/kubernetes/reloader.yaml
首先创建我们要用到的配置对象，其中包含了一个 Secret 和一个 Configmap：

kubectl create configmap nginx-conf --from-file=configmap/conf/

server {
   listen       8082;         ##端口为8082
   server_name  _;
   root         /html;         ##改数据目录为/html

   location / {
   }
}


接下来部署一个Nginx，来验证 Reloader 的功能：

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-pod
  annotations:
    reloader.stakater.com/auto: "true"
spec:
  selector:
    matchLabels:
      app: nginx-pod
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.9
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
              name: port
              protocol: TCP
          volumeMounts:
            - name: nginx-cm
              mountPath: "/etc/nginx/conf.d"
      volumes:
        - name: nginx-cm
          configMap:
            name: nginx-conf

这里的 Annotation 表示自动监控相关对象。

接下来随意改动一下 Configmap 或者 Secret 的值，就会看到 Pod 重建了。

自动变更有时也需要手工指定的辅助的，例如服务依赖的情况，可以依赖上游服务的 Configmap 变更进行重启；或者是对某些可以自动处理的配置文件进行忽略处理，都可以使用如下两个注解：

secret.reloader.stakater.com/reload: "secret1,secret2"
configmap.reloader.stakater.com/reload: "configmap1, configmap2"
补充
Reloader 的命令行还有两个参数：

--namespaces-to-ignore：忽略部分命名空间的监听
--resources-to-ignore：忽略部分对象的变更

相关链接
https://github.com/stakater/Reloader