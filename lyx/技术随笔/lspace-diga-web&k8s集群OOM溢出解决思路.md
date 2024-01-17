# lspace-diga-web&k8s集群OOM溢出解决思路

根据前面发的message解决思路我们发现diga在加完内存到1.5G之后还是经常OOMKilled，只是频率不会像以前一样那么高

![image-20240116103329775](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240116103329775.png) 



我们以10.1.64.60这个容器的JVM进行分析

![image-20240116103355588](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240116103355588.png) 



我们发现在JVM Memory中有明显断层

 ![image-20240116103436879](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240116103436879.png)

total到了1.17GB的时候也断层了

![image-20240116103516624](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240116103516624.png) 



CPU以及其他资源没有非常高的时候

![image-20240116103634499](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240116103634499.png) 



初步判定还是内存不足的问题。

通过对pod的观察发现还是内存使用非常高

![image-20240116103845783](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240116103845783.png)

看这7天的变化以及重启次数

![image-20240116103934405](https://lyx-study-note-image.oss-cn-shenzhen.aliyuncs.com/img/image-20240116103934405.png) 





























