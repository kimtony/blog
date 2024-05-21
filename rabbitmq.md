## 运行

### 拉取镜像
```
docker pull rabbitmq

docker run -d --hostname my-rabbit --name rabbit -p 15672:15672 -p 5672:5672 rabbitmq
```

### 启动管理界面
```
docker ps 
docker exec -it 镜像ID /bin/bash
rabbitmq-plugins enable rabbitmq_management
```

### 访问地址
ip:15672

guest guest





