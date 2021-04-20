[TOC]









# kubectl概述

kubectl是Kubernetes集群的命令行工具，通过kubectl能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。运行kubectl命令的语法如下所示：

```bash
$ kubectl [command] [TYPE] [NAME] [flags]
```

- **comand**：指定要对资源执行的操作，例如create、get、describe和delete
- **TYPE**：指定资源类型，资源类型是大小学敏感的，开发者能够以单数、复数和缩略的形式。例如：

```
$ kubectl get pod pod1 
$ kubectl get pods pod1 
$ kubectl get po pod1
```

- **NAME**：指定资源的名称，名称也大小写敏感的。如果省略名称，则会显示所有的资源，例如:

```
$ kubectl get pods
```

- **flags**：指定可选的参数。例如，可以使用-s或者–server参数指定Kubernetes API server的地址和端口。

另外，可以通过kubectl help命令获取更多的信息。

```bash
# kubectl help
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create        Create a resource from a file or from stdin.
  expose        Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service
  run           Run a particular image on the cluster
  set           Set specific features on objects

Basic Commands (Intermediate):
  explain       Documentation of resources
  get           Display one or many resources
  edit          Edit a resource on the server
  delete        Delete resources by filenames, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout       Manage the rollout of a resource
  scale         Set a new size for a Deployment, ReplicaSet or Replication Controller
  autoscale     Auto-scale a Deployment, ReplicaSet, or ReplicationController

Cluster Management Commands:
  certificate   Modify certificate resources.
  cluster-info  Display cluster info
  top           Display Resource (CPU/Memory/Storage) usage.
  cordon        Mark node as unschedulable
  uncordon      Mark node as schedulable
  drain         Drain node in preparation for maintenance
  taint         Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe      Show details of a specific resource or group of resources
  logs          Print the logs for a container in a pod
  attach        Attach to a running container
  exec          Execute a command in a container
  port-forward  Forward one or more local ports to a pod
  proxy         Run a proxy to the Kubernetes API server
  cp            Copy files and directories to and from containers.
  auth          Inspect authorization

Advanced Commands:
  diff          Diff live version against would-be applied version
  apply         Apply a configuration to a resource by filename or stdin
  patch         Update field(s) of a resource using strategic merge patch
  replace       Replace a resource by filename or stdin
  wait          Experimental: Wait for a specific condition on one or many resources.
  convert       Convert config files between different API versions
  kustomize     Build a kustomization target from a directory or a remote url.

Settings Commands:
  label         Update the labels on a resource
  annotate      Update the annotations on a resource
  completion    Output shell completion code for the specified shell (bash or zsh)

Other Commands:
  alpha         Commands for features in alpha
  api-resources Print the supported API resources on the server
  api-versions  Print the supported API versions on the server, in the form of "group/version"
  config        Modify kubeconfig files
  plugin        Provides utilities for interacting with plugins.
  version       Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).

```

# 1.1 kubectl 命令分类总结

## 1.1.1 基础命令

| 基础命令 | 解释                                                         |
| -------- | ------------------------------------------------------------ |
| create   | 通过文件名或标准输入创建资源                                 |
| expose   | 使用replication controller, service, deployment或者pod并暴露它作为一个新的Kubernetes Service |
| run      | 在集群中运行一个特定的镜像                                   |
| get      | 显示一个或者多个资源（资源分为pod、instance、service等）     |
| set      | 在对象上设置特定的功能                                       |
| explain  | 文档参考资料                                                 |
| edit     | 使用默认的编辑器编辑一个资源                                 |
| delete   | 通过文件名、标准输入、资源名称，或标签来删除资源             |

示例：

```bash
kubectl create deployment nginx --image=nginx:1.14
kubectl create -f my-nginx.yaml
kubectl run nginx --image=nginx:1.16 --port=80 --replicas=1
kubectl expose deployment/nginx  --type="NodePort" --port=80 --name=nginx
kubectl get cs                          # 查看集群状态
kubectl get nodes                       # 查看集群节点信息
kubectl get ns                          # 查看集群命名空间
kubectl get svc -n kube-system          # 查看指定命名空间的服务
kubectl get pod <pod-name> -o wide      # 查看Pod详细信息
kubectl get pod <pod-name> -o yaml      # 以yaml格式查看Pod详细信息
kubectl get pods                        # 查看资源对象，查看所有Pod列表
kubectl get rc,service                  # 查看资源对象，查看rc和service列表
kubectl get pod,svc,ep --show-labels    # 查看pod,svc,ep能及标签信息
kubectl get all --all-namespaces        # 查看所有的命名空间
```

**kubernetes资源对象类型**

在kubernetes中，提供了很多的资源对象，开发和运维人员可以通过这些对象对容器进行编排。在下表中，是kubectl所支持的资源对象类型，以及它们的缩略别名：

| 资源对象类型               | 缩略别名 |
| -------------------------- | -------- |
| apiservices                |          |
| certificatesigningrequests | csr      |
| clusters                   |          |
| clusterrolebindings        |          |
| clusterroles               |          |
| componentstatuses          | cs       |
| configmaps                 | cm       |
| controllerrevisions        |          |
| cronjobs                   |          |
| customresourcedefinition   | crd      |
| daemonsets                 | ds       |
| deployments                | deploy   |
| endpoints                  | ep       |
| events                     | ev       |
| horizontalpodautoscalers   | hpa      |
| ingresses                  | ing      |
| jobs                       |          |
| limitranges                | limits   |
| namespaces                 | ns       |
| networkpolicies            | netpol   |
| nodes                      | no       |
| persistentvolumeclaims     | pvc      |
| persistentvolumes          | pv       |
| poddisruptionbudget        | pdb      |
| podpreset                  |          |
| pods                       |          |
| podsecuritypolicies        | psp      |
| podtemplates               |          |
| replicasets                |          |
| replicationcontrollers     | rc       |
| resourcequotas             | quota    |
| rolebindings               |          |
| roles                      |          |
| secrets                    |          |
| serviceaccounts            | sa       |
| services                   | svc      |
| statefulsets               |          |
| storageclasses             |          |
|                            |          |



## 1.1.2 部署命令

| 部署命令       | 解释                                                 |
| -------------- | ---------------------------------------------------- |
| rollout        | 管理资源的发布                                       |
| rolling-update | 对给定的复制控制器滚动更新                           |
| scale          | 扩容或者缩绒pod数量、Deployment、ReplicaSet、RC、Job |
| autoscale      | 创建一个自动进行扩容和缩容并设置Pod数量              |

示例：

```bash
kubectl scale deployment nginx --replicas 5    # 扩容
kubectl scale deployment nginx --replicas 3    # 缩容
```



## 1.1.3 集群管理命令

| 集群管理命令 | 解释                             |
| ------------ | -------------------------------- |
| certificate  | 修改证书资源                     |
| cluster-info | 显示集群信息                     |
| top          | 显示资源使用，需要Heapster运行   |
| cordon       | 标记节点不可被调度               |
| uncordon     | 标记几点可被调度                 |
| drain        | 驱逐姐弟啊上的应用，准备下线维护 |
| taint        | 修改节点taint标记                |

示例：

```bash
kubectl cluster-info            # 查看集群状态信息
```



## 1.1.4 故障和调试命令

| 故障和调试命令 | 解释                                                         |
| -------------- | ------------------------------------------------------------ |
| describe       | 显示特定资源胡资源组的详细信息                               |
| logs           | 在一个pod中打印一个容器日志，如果pod只有一个容器，容器名称是可选的 |
| attach         | 附加到一个运行的容器                                         |
| exex           | 执行命令到容器                                               |
| port-forward   | 转发一个多或多个本地端口到pod                                |
| proxy          | 运行一个proxy到kubenetes API Server                          |
| cp             | 拷贝文件或目录到容器中                                       |
| auth           | 检查授权                                                     |

示例：

```bash
kubectl describe nodes <node-name>  # 显示Node的详细信息
kubectl describe pods/<pod-name>    # 显示Pod的详细信息
kubectl logs --tail=200 nginx -n default|less
kubectl logs nginx-deploy -n default          # 查看pod中单个容器日志
kubectl logs nginx-deploy -c nginx -n default # 查看pod中指定容器日志,如果Pod中运行有多个容器，需使用"-c"指定容器名
kubectl exec nginx-deploy -c nginx -n default -- ps aux  #不进入容器执行命令,通过kubectl exec命令指定pod，-c指定容器名称,-n指定名称空间,-- 指定执行的命令来进行操作容器
kubectl exec nginx-deploy-788b9c6b69-tfcrx -c nginx -n default -i -t -- /bin/sh #进入容器,通过kubectl exec命令指定相应的参数 -i -t 来分配一个伪终端和交互式接口，指定 /bin/sh 命令进入容器
kubectl port-forward --address 0.0.0.0 pod/nginx 8888:80 #将Pod的开放的80端口映射到本地8888
```

## 1.1.5 其他命令

| 其他命令     | 解释                                             |
| ------------ | ------------------------------------------------ |
| apply        | 通过文件名或者标准输入对资源应用配置             |
| patch        | 使用补丁修改，更新资源字段                       |
| replace      | 通过文件名或标准输入替换一个资源                 |
| convert      | 不同的API版本之间转换配置文件                    |
| label        | 更新资源上的标签                                 |
| annotate     | 更新资源上的注释                                 |
| completion   | 用于实现kucectl工具自动补全                      |
| api-versions | 打印受支持的API版本                              |
| config       | 修改kubeconfig文件，用于访问API,比如配置认证信息 |
| help         | 所有帮助命令                                     |
| plugin       | 运行一个命令行插件                               |
| version      | 打印客户端和服务版本信息                         |

.





# 场景一：备份恢复docker中mysql

1. 导出

```text
docker exec -i mysql_container mysqldump -uroot -proot --databases database_name --skip-comments > /path/to/my/dump.sql
```

2. 导入

```text
docker cp /path/to/my/dump.sql mycontainer:/dump.sql
docker exec mysql_container /bin/bash -c 'mysql -uroot -proot < /dump.sql'
```

# 场景二：备份会服k8s中的mysql

1. 导出

```text
kubectl exec mysql-58 -n sql -- mysqldump -u root -proot --all-databases > dump_all.sql
```

2. 导入

```text
kubectl exec -it mysql-58 -n sql -- mysql -u root -proot USERS < dump_all.sql
```





