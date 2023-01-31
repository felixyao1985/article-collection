# Kubernetes - Configmap热更新原理客-CSDN博客
[Kubernetes - Configmap 热更新原理\_每日一小知识的博客 - CSDN 博客](https://blog.csdn.net/m0_54861649/article/details/124312039) 

 GitHub 地址： [https://github.com/QingyaFan/container-cloud/issues/2](https://github.com/QingyaFan/container-cloud/issues/2)

Kubernetes 中提供 configmap，用来管理应用的配置，configmap 具备[热更新](https://so.csdn.net/so/search?q=%E7%83%AD%E6%9B%B4%E6%96%B0&spm=1001.2101.3001.7020)的能力，但只有通过目录挂载的 configmap 才具备热更新能力，其余通过环境变量，通过 subPath 挂载的文件都不能动态更新。这篇文章里我们来看看 configmap 热更新的原理，以及为什么只有目录形式挂载才具备热更新能力。

## configmap 热更新原理

我们首先创建一个 configmap（configmap-test.yaml）用于说明，其内容如下。我们初始化好这个 configmap`kubectl apply -f configmap-test.yaml`。

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: marvel-configmap
data:
  marvel: |
    {
      name: "iron man",
      skill: [
        "fight", "fly"
      ]
    }

```

configmap 资源对象会存储在[etcd](https://so.csdn.net/so/search?q=etcd&spm=1001.2101.3001.7020)中，我们看下存储的是什么东东，哦，原来就是明文存储的。

```
[root@bogon ~]# ETCDCTL_API=3 etcdctl get /registry/configmaps/default/marvel-configmap
/registry/configmaps/default/marvel-configmap
k8s

v1	ConfigMap?
?
marvel-configmapdefault"*$02d3b66f-da26-11e9-a8c5-0800275f21132????b?
0kubectl.kubernetes.io/last-applied-configuration?{"apiVersion":"v1","data":{"marval":"{
  name: "iron man",
  skill: [
    "fight", "fly"
  ]
}
"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"marvel-configmap","namespace":"default"}}
zD
marval:{
  name: "iron man",
  skill: [
    "fight", "fly"
  ]
}
"

```

接下来使用一个 redis 的 pod 来挂载这个 configmap：

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name:  redis
  labels:
    name:  redis
spec:
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        name:  redis
    spec:
      containers:
      - image:  redis:5.0.5-alpine
        name:  redis
        resources:
          limits:
            cpu: 1
            memory: "100M"
          requests:
            cpu: "200m"
            memory: "55M"
        ports:
        - containerPort:  6379
          name:  redis
        volumeMounts:
        - mountPath: /data
          name: data
        - mountPath: /etc/marvel
          name: marvel
      volumes:
        - name: data
          emptyDir: {}
        - name: marvel
          configMap:
              name: marval-configmap
              items:
                - key: marvel
                  path: marvel
      restartPolicy: Always
      imagePullPolicy: Always

```

我们启动这个 deploy，然后修改一下 configmap，把用拳头锤别人的浩克加上，看多长时间可以得到更新。

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: marvel-configmap
data:
  marvel: |
    {
      name: "iron man",
      skill: [
        "fight", "fly"
      ]
    },
    {
      name: "hulk",
      skill: [
        "fist"
      ]
    }

```

经过测试，经过了 11s 时间，pod 中的内容得到了更新。再把黑寡妇也加上，耗时 48s 得到了更新。

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: marvel-configmap
data:
  marvel: |
    {
      name: "iron man",
      skill: [
        "fight", "fly"
      ]
    },
    {
      name: "hulk",
      skill: [
        "fist"
      ]
    },
    {
      name: "Black widow",
      skill: [
        "magic"
      ]
    }

```

所以更新延迟不一定，为什么呢？接下来我们看下 configmap 热更新的原理。

## 是 kubelet 在做事

kubelet 是每个节点都会安装的主要代理，负责维护节点上的所有容器，并监控容器的健康状况，同步容器需要的数据，数据可能来自配置文件，也可能来自 etcd。kubelet 有一个启动参数`--sync-frequency`，控制同步配置的时间间隔，它的默认值是 1min，所以更新 configmap 的内容后，真正容器中的挂载内容变化可能在`0~1min`之后。修改一下这个值，修改为 5s，然后更改 configmap 的数据，检查热更新延迟时间，都降低到了 3s 左右，但同时 kubelet 的资源消耗会上升，尤其运行比较多 pod 的 node 上，性能会显著下降。

## 怎么实现的呢

Kubelet 是管理 pod 生命周期的主要组件，同时它也会维护 pod 所需的资源，其中之一就是 configmap，实现定义在`pkg/kubelet/configmap/`中，kubelet 主要是通过 configmap_manager 来管理每个 pod 所使用的 configmap，configmap_manager 有三种：

-   Simple Manager
-   TTL Based Manager
-   Watch Manager

默认使用 `Watch Manager`。其实 Manager 管理的主要是缓存中的 configmap 对象，而 kubelet 同步的是 Pod 和缓存中的 configmap 对象。如下图所示：

![](https://img-blog.csdnimg.cn/20191101010024819.JPG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Fpbmd5YWZhbg==,size_16,color_FFFFFF,t_70)

### Simple Manager

Simple Manager 直接封装了访问 api-server 的逻辑，其更新延迟（图中 delay）为 0。

### TTL Based Manager

当 pod 启动或者更新时，pod 引用的缓存中的 configmap 都会被无效化。获取 configmap 时 (`GetObject()`)，先尝试从 TTL 缓存中获取，如果没有，过期或者无效，将会从 api-server 获取，获取的内容更新到缓存中，替换原来的内容。

`CacheBasedManager` 的定义在 `pkg/kubelet/util/manager/cache_based_manager.go` 中。

```
func NewCacheBasedManager(objectStore Store, getReferencedObjects func(*v1.Pod) sets.String) Manager {
	return &cacheBasedManager{
		objectStore:          objectStore,
		getReferencedObjects: getReferencedObjects,
		registeredPods:       make(map[objectKey]*v1.Pod),
	}
}

```

### Watch Manager

每当 pod 启动或更新时，kubelet 会对该 pod 新引用的所有 configmap 对象启动监控（watches），watch 负责利用新的 configmap 对缓存的 configmap 更新或替换。

`WatchBasedManager` 的定义在 `pkg/kubelet/util/manager/watch_based_manager.go` 中。

```
func NewWatchBasedManager(listObject listObjectFunc, watchObject watchObjectFunc, newObject newObjectFunc, groupResource schema.GroupResource, getReferencedObjects func(*v1.Pod) sets.String) Manager {
	objectStore := NewObjectCache(listObject, watchObject, newObject, groupResource)
	return NewCacheBasedManager(objectStore, getReferencedObjects)
}

```

## 总结

只有当 Pod 使用目录形式挂载 configmap 时才会得到热更新能力，其余两种使用 configmap 的方式是 Pod 环境变量注入和 subPath 形式。

因为 kubelet 是定时（以一定的时间间隔）同步 Pod 和缓存中的 configmap 内容的，且三种 Manager 更新缓存中的 configmap 内容可能会有延迟，所以，当我们更改了 configmap 的内容后，真正反映到 Pod 中可能要经过`syncFrequency + delay`这么长的时间。
