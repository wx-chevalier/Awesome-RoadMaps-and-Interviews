# 虚拟化与编排面试集锦

# 容器与 Docker

- 在容器部署模型中，应该使用多容器模型还是单容器中多进程模型？

# Kubernetes

- 为什么在 Kubernetes 我们不直接使用一个单独的容器（container），而是用 Pod 来封装一个或多个容器呢？为什么我们要运行多个容器呢？我们能将我们所有的应用程序都放到一个容器里面运行么？Kubernetes 的多个容器间应该如何通信？

- Pod，Deployment，Service 与 Ingress 分别在 K8s 集群中起到了怎样的作用？

- Service 定义中的 port，targetPort 与 nodePort 各自的区别是什么？

- 对于使用 nodePort 方式对外暴露的服务，其数据网络流向是什么样子的？
