# k8s

[微博内部对 k8s 的应用]（./weibo_k8s.md）
[k8s 网络之Calico](./Calico.md)

## 带着问题去探索

* Kubernetes包含几个组件，各个组件的功能是什么，组件之间是如何交互的？
* [kubernetes的网络是如何实现的?](k8s_net.md)
* Kubernetes的Pause容器有什么用，是否可以去掉？
* Kubernetes中的Pod内几个容器之间的关系是什么？
* 一个经典Pod的完整生命周期是怎么样的？
* Kubernetes的Service和ep是如何关联和相互影响的？
* 详述kube-proxy的工作原理，一个请求是如何经过层层转发落到某个Pod上的？注意请求可能来自Pod也可能来自外部。
* rc/rs功能是怎么实现的？请详述从API接收到一个创建rc/rs的请求，到最终在节点上创建Pod的全过程，尽可能详细。另外，当一个Pod失效时，Kubernetes是如何发现并重启另一个Pod的？
* deployment/rs有什么区别，其使用方式、使用条件和原理是什么？
* cgroup中的CPU有哪几种限制方式。Kubernetes是如何使用实现request和limit的？
* 设想一个一千台物理机，上万规模容器的Kubernetes集群，请详述使用Kubernetes时需要注意哪些问题？应该怎样解决？（提示可以从高可用，高性能等方向，覆盖从镜像中心到Kubernetes各个组件等）
* 设想Kubernetes集群管理从一千台节点到五千台节点，可能会遇到什么样的瓶颈，应该如何解决？
* Kubernetes的运营中有哪些注意的要点？
* 集群发生雪崩的条件，以及预防手段。
* 设计一种可以替代kube-proxy的实现。
* Sidecar的设计模式如何在Kubernetes中进行应用，有什么意义？
* 灰度发布是什么，如何使用Kubernetes现有的资源实现灰度发布？
* 介绍Kubernetes实践中踩过的比较大的一个坑和解决方式。
