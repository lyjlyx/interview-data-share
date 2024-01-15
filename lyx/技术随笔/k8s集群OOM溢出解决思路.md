# k8s集群OOM溢出解决思路

### 问题抛出

生产环境中的容器会因为内存溢出问题而重启，为了解决这种异常重启问题我们需要去排查一下问题原因

![image-20240114105809849](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240114105809849.png) 

看到的这个容器的启动脚本为:

```shell
helm upgrade -i $SERVICE_NAME  lspace-prod/lspace-web --set image.tag=${IMAGE_TAG},serviceName=$SERVICE_NAME,servletContext=message,replicaCount=$REPLICA,spring.profiles.active=$ENVIROMENT --kubeconfig $KUBE_CONFIG -n $KUBE_NAMESPACE
```

可以看到我们没有设置jvm虚拟机的内存大小是多少，所以使用的是默认的



### 排查过程



#### JVM排查

首先我们做的处理先分析grafana：

这边有个小技巧，这边赘述一下，如果是OOM挂了的话这个容器的ip是不会变的，所以我们直接看对应pod的ip地址分析就行了

![image-20240114111245942](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240114111245942.png) 

![image-20240114111300142](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240114111300142.png)  

我们根据k8s详情中的Last State可以知道发生的时间为: 2024-01-14 10:13:15分

![image-20240114111411141](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240114111411141.png) 

看到JVM上有明显的断层

![image-20240114111328215](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240114111328215.png) 

发现当时是使用到了446M，但是没有超过最大的max:1.72G



这边先看看finished时间节点的容器日志有没有异常

```shell
cd /var/log/pods/lspace-prod_lspace-message-service-6c58d88bf8-5fmpl_4c72ef5e-6849-491d-b0b1-41e9d2807251/lspace-message-service/
```

使用命令查看这时候程序里面有没有异常

```sheel
grep '24-01-14 10:' 6.log
```

看了一遍过去发现没有什么异常



**结合上面的各种信息这里可以下个判断，其实并不是JVM导致的OOMKilled，因为我们之前的容器也添加过memory的大小给他添加到了1.5G，但是还是会出现**

附脚本添加过程：

该脚本的容器与上图无关

原先的脚本：

```shell
helm upgrade -i $SERVICE_NAME  lspace-prod/lspace-web --set image.tag=${IMAGE_TAG},serviceName=$SERVICE_NAME,servletContext=diga,replicaCount=$REPLICA,spring.profiles.active=$ENVIROMENT --kubeconfig $KUBE_CONFIG -n $KUBE_NAMESPACE
```

修改过后的脚本

```shell
helm upgrade -i $SERVICE_NAME  lspace-prod/lspace-web --set image.tag=${IMAGE_TAG},serviceName=$SERVICE_NAME,servletContext=diga,replicaCount=$REPLICA,spring.profiles.active=$ENVIROMENT,java_opts="-Xms1024m -Xmx1024m",resources.limits.memory=1500Mi,resources.requests.memory=1500Mi --kubeconfig $KUBE_CONFIG -n $KUBE_NAMESPACE
```

字段解释resources.limits.memory也就是资源我们限制到最大1.5G，资源所能索要的也为1.5G

**资源增加过后发现其实只是降低了点OOM出现的频率，该OOM还是会，所以既没治标也没治本**



#### 容器排查

既然不是JVM的锅，那就往上一级找，排查下容器pod看看是否有异常的事件发生

grafana搜索 pod

![image-20240114122958844](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240114122958844.png) 

筛选一下pod

![image-20240114123053750](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240114123053750.png)





















