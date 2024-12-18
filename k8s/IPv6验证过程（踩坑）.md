### 部署遇到的问题

1. 在.8.20的环境内部署集群

   1. 12.235为master，12.236为node

   2. 因为之前的机子存在问题，所以每次在部署的时候都会失败（虽然235，236也会失败，原因是calico没能安装，后面回顾描述）

   3. 遇到的问题都有

      1. 脚本准备错误	——对应的机子内存满了，生成不出脚本

      2. 脚本内略过deploy部份的片断脚本

         1. 通过params.json和install.yml内的配置做的判断哪一部分被略过
         2. 后面跟换了其他的机子做测试，就不会出现这个问题了，但是在install culster的时候时间过短

      3. 页面上的字典接口报错，错误为没有权限，问题导致在创建集群的第三部阻塞，选择不到对应的资源名称

      4. 最大的问题主要还是机子的问题，被几台有些问题的机子困住，以为能选中的机子是空闲的，便在此基础上开始部署，但是后来跟勇哥一起弄，才了解到，机子即使被用了，也能被选到。

         1. 导致在执行的过程中就会出现问题，基本就是一些清除的命令会出错。

            

   	4. 最后解决的方案是在其他的集群内，缩容出两台供我们测试



虽然说235、236也不能成功部署，但是根本原因是网络工具没能成功起起来

然后通过文档内提供的calico.yaml执行即可获取，需要注意的是配置中的harbor地址的端口需要由5000换成10081

通过kubectl apply -f calico.yaml，即可获取两个关于calico的pod。

- 但由于calico.yaml内的配置有些问题，对应的正式啥的并不是当前的，所以即使有pod也跑不起来，会一直崩溃

​	

另外在没有calico的时候，master上的状态为notready，安装好后，就变成了ready。



昨晚再对一遍的时候发现，会失败的原因是关于236内的hosts对应的apiserver.cluster.local映射的是其他的机子，所以在后面install的时候它在不断的重试，现在我把对应的地址改成了235，不知道能否成功 ——不行。在最后的阶段他自己会将值改成那个92.2啥的。这个方法也不行。（其实这个最后也无关紧要）



#### 最后的解决：

1. 在最后install的时候，报错的问题是关于想join236的时候遇到问题，但是这个问题很奇怪，因为master上还是NotReady的时候进行Join是不行的，

   （不描述那么多了，直接说怎么解决的）

1. 在报错失败后，两个集群的状态分别是：

   1. master已经是sealos了，node等着join状态

2. 但是master的nodes状态是NotReady，因为calico没有安装好，通过

   `helm install helm-calico -f calico.yaml   paas_public_chart/tigera-operator -n calico  --create-namespace --version v3.23.3`

   对应的calico.yaml文件是通过在本地项目中的`calico-values.yaml.j2`，把变量翻译后，放在master上执行的。

   - 中间遇到了些问题，是没能成功拉取镜像，修改成我们自己harbor的地址就可以了

3. 待上面代码执行完成后，master上的pod会多出几个caclico开头的pod。这时候的master的nodes状态就变成了ready了。然后等这那几个calico的po也变成ready后。master上的所有就正常了

4. 这时候通过在master上生成一串命令

   `kubeadm token create --print-join-command`

   得到类似这样的命令

   `kubeadm join <control-plane-ip>:6443 --token <your-token> --discovery-token-ca-cert-hash sha256:<hash>`

   然后在node上sudo执行，master就成功的扩容了！

   在master，get nodes就能看到这个node了

5. 这时候的node其实还缺少admin.conf的配置，这个配置是从master获取的

   1. 在node上确认目录有没有存在，没有存在就创建

      `ls -ld /root/.kube/config`

      创建：` mkdir -p /root/.kube`，加权限： `chmod 600 /root/.kube/config`

   2. 在master上将对应的配置文件传给node

      ` scp /etc/kubernetes/admin.conf root@10.1.12.236:/root/.kube/config`

      这样就行，/etc/kubernetes/admin.conf的地址是固定的。后面就是刚创建出来的地址





#### **另外的坑：**

在master上创建pod的时候，选择了公司内harbor的nginx，然后找到对应的版本后，手动修改了镜像的路径，如下：

`harbor.paas.nl:10081/paas_public/nginx:v1.23.5`

在以为一切顺利的时候，遇到了问题。镜像一直拉取失败，一直说找不到这个pod。

一下就疑惑了，开始把域名换成ip的尝试，或者修改对应yaml内的名称，或者换成其他的镜像（其他镜像需要其他以来和操作，所以仅换镜像一定会失败），还有docker直接拉取的pull命令，以及后面补全了关于nginx的yaml配置，kind由原来的pod换成了Deployment等。依旧是失败。

最后关注点又回到了harbor身，在查询完nginx的配置后，那个对应的拉取命令其实是错误的，不能用的：
`docker pull 10.1.12.220/paas_public/nginx@sha256:4669f6671aca20a34c3dfcd017e84fb3cae40788ea664866eaea698e3dfe241c`

其实最后发现，再点进对应的版本后，再点击相同的位置的拉取命令得到的是这样的：
`docker pull 10.1.12.220/paas_public/nginx:1.25.3`

这时候才发现nginx这里其实并不需要版本的标识v，然后把对应的删除后，再apply应用，就可以了正常的Running了。

**总结：**需要的镜像，直接使用对应版本内部的拉取命令。不要手动修改路径。







#### 中间遇到的问题

- master不小心删除了，但是程序正常运行的原因是：`项目是依赖calico之间运行的，k8s只是负责容器的调度`（去问问这个是什么原理）

  **Master** 节点不可用，但程序正常运行的原因是：项目依赖于 Calico 提供的容器间网络通信，而 **K8s** 主要负责集群的管理和调度，容器的实际运行依赖于 **worker** 节点。

- ssh-copy-id 10.1.12.172 直接执行免密操作

- sealos如果没有，直接从其他有的复制过来即可`scp -r /usr/bin/sealos 10.1.12.172:/usr/bin/sealos`
  - 执行sealos前要把关于k8s的文件尽量清楚干净，删除对应占用端口的进程。（git上有对应的命令）
  
    ```shell
    sudo rm -rf /etc/containerd/* \
                      sudo rm -rf /var/lib/sealos/data/* \
                      sudo rm -rf  /root/registry \
                      sudo rm -rf  /var/lib/registry \
                      sudo rm -rf /var/lib/registry \
                      sudo rm -rf /etc/registry \
                      sudo rm -rf /etc/registry.yml \
                      systemctl stop kubelet \
                      systemctl daemon-reload \
                      systemctl stop kube-apiserver.service \
                      systemctl stop kube-controller-manager.service \
                      systemctl stop kube-proxy.service \
                      systemctl stop kube-scheduler.service \
                      systemctl disable kube-apiserver.service \
                      systemctl disable kube-controller-manager.service \
                      systemctl disable kube-proxy.service \
                      systemctl disable kube-scheduler.service \
                      sudo rm -rf  /usr/bin/buildah \
                      sudo rm -rf /usr/bin/conntrack \
                      sudo rm -rf /usr/bin/kubelet-pre-start.sh \
                      sudo rm -f sudo /usr/bin/kubelet-post-stop.sh \
                      sudo rm -rf /usr/bin/kubeadm \
                      sudo rm -rf /usr/bin/kubectl \
                      sudo rm -rf /usr/bin/kubelet \
                      sudo rm -rf /etc/sysctl.d/k8s.conf \
                      sudo rm -rf /etc/systemd/system/kubelet.service \
                      sudo rm -rf /etc/systemd/system/kubelet.service.d \
                      systemctl stop image-cri-shim \
                      systemctl disable image-cri-shim \
                      sudo rm -rf /etc/systemd/system/image-cri-shim.service \
                      systemctl daemon-reload \
                      sudo rm -rf /usr/bin/image-cri-shim \
                      sudo rm -rf /etc/crictl.yaml \
                      sudo rm -rf /etc/image-cri-shim.yaml \
                      sudo rm -rf /var/lib/image-cri-shim \
                      sudo rm -rf /etc/kubernetes \
                      sudo rm -rf /var/lib/etcd \
                      sudo rm -rf /usr/bin/containerd \
    				  sudo rm -rf /etc/containerd
    
    #{pf_step_msg}:判断kubelet、kube-proxy进程是否存活"
    ps -ef | grep kube-proxy | grep -v grep | awk '{print $2}' && ps -ef | grep kubelet | grep -v grep | awk '{print $2}'
    
    #{pf_step_msg}:清理进程(kubelet、kube-proxy)
    ps aux|grep kube|grep -v grep |awk '{print $2}'|xargs kill -9 
    ```
  
    `sealos delete --nodes 10.1.12.236 --cluster wu9a3duzktct`
  
  - sealos在执行ClusterFile的时候，里面的配置对应的master地址不能是当前执行的机子的ip `sealos apply -f Clusterfile`
    比如：当前执行的机子是174，那么Clusterfile里master不能是当前的174



- sealos add --nodes 10.1.12.236 --cluster bufeng 

  我是在122上执行的这个命令，但是遇到这个问题

  ```shell
  2024-12-16 09:10:56 [EROR] Applied to cluster error: failed to run checker: checker: failed to get host 10.1.12.172:22 hostname, 
  wait for [10.1.12.122:22] ssh ready timeout:  [ssh 10.1.12.122:22]create ssh session failed, 
  ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain, 
  ensure that the IP address or password is correct
  ```

  如错误提示，其实问题就是免密登入问题。对122的免密登入（对自身的免密登入）。

  就是我看到对自己的免密登入不理解产生的阻塞。



- 在执行修改的ipv6的脚本后，执行到后期，出现了关于kubelet相关的问题，原本以为是新增的脚本除了问题，然后开始挨个修改，最后发现，其实是serviceSubnet 出了问题：

  ```yaml
  apiVersion: kubeadm.k8s.io/v1beta3
  kind: ClusterConfiguration
  kubernetesVersion: v1.24.2
  networking:
    # podSubnet: 172.20.0.0/16
    # serviceSubnet: 10.68.0.0/16
    podSubnet: "fd00::/64"            
    # serviceSubnet: "fd00::/112"     # 出问题的地方
  imageRepository: harbor.paas.nl:10081/paas_public/kube-system
  ```

  注释就没事，开启就出问题。

  ```shell
  10.1.12.172:22: Unfortunately, an error has occurred:
  10.1.12.172:22:         timed out waiting for the condition
  10.1.12.172:22: 
  10.1.12.172:22: This error is likely caused by:
  10.1.12.172:22:         - The kubelet is not running
  10.1.12.172:22:         - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)
  10.1.12.172:22: 
  10.1.12.172:22: If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
  10.1.12.172:22:         - 'systemctl status kubelet'
  10.1.12.172:22:         - 'journalctl -xeu kubelet'
  ```

  

  问题排查过程：

  1. 查询问题得知，serviceSubnet 与 podSubnet 是不能相同的，要不ip会冲突，那么配置就换成了：

     ```yaml
     networking:
       podSubnet: "fd00::/64"            
       serviceSubnet: "fc00::/112"     # 出问题的地方
       # serviceSubnet: 10.68.0.0/16
     ```

     依旧是同样的问题。

     而如果把 serviceSubnet 换成 ipv4 ，就能正常启动。

     开始怀疑是不是kubelet配置的问题。

     发现sealos官方有双栈的安装方式，1.24需要4.0版本以上的才行。了解到我们版本是4.1左右的，并且做了改动（root权限）。

     - 官方提供的版本 podSubnet 和 serviceSubnet 需要两个值
       https://sealos.run/docs/5.0.0/developer-guide/lifecycle-management/advanced-guide/dual-stack-cluster/

       ```yaml
       networking:
         podSubnet: 172.20.0.0/22,fd00:2222::/112                 # IPv6 和 IPv4 双栈 Pod 子网
         serviceSubnet: 10.86.0.0/22,fd00:1111::/116              # IPv6 和 IPv4 双栈服务子网
       ```

       但是会出现：

       `[EROR] Applied to cluster error: failed to init generator config init kubeadm config error: invalid CIDR address: 172.20.0.0/22,fd00:2222::/112`

       意思就是格式错了，不能放两个值，但是官网的就这样，所以开始怀疑是不是sealos的版本问题。确实是版本问题，版本需要 >= 4.3

     

  

  

  除此之外，还有两个小问题：1. podSubnet 和 serviceSubnet 都不能放下两个值。 2. serviceSubnet 内ipv6的前缀长度要 >= 108

  

  查询问题得知，serviceSubnet 与 podSubnet 是不能相同的，要不ip会冲突

  

​	







### 1. **基本命令**

```bash
# 查看 Kubernetes 版本
kubectl version

# 查看集群信息
kubectl cluster-info

# 查看当前上下文（集群配置）
kubectl config current-context

# 查看所有命名空间
kubectl get namespaces

# 查看指定命名空间下的所有资源
kubectl get all -n <namespace>

# 查看集群中所有节点
kubectl get node

# 查看某个节点详细信息
kubectl describe node <node-name>
```



### 2. **Pod 操作**

```shell
# 查看默认命名空间的 Pods
kubectl get pods｜pod|po

# 查看指定命名空间下的 Pods
kubectl get pods -n kube-system

# 查看对应节点下都有哪些pod
kubectl get pod -A -o wide  | grep k8s-host229

# 做实时监控
kubectl get pod -A -o wide -w

# 查看 Pod 的详细信息
kubectl describe pod <pod-name> -n <namespace>

# 创建 Pod（通过 YAML 文件，kind为Pod）
kubectl apply -f xxx.yaml

# 删除 Pod  如果想直接删除相关的pod，用 kubectl delete -f xxx.yaml
kubectl delete pod <pod-name> -n <namespace>

# 进入 Pod 的容器，多个容器的话默认进入一个容器
kubectl exec -it <pod-name> -- /bin/bash
# 如果一个 pod 内有多个容器，想指定进入一个容器 -c
kubectl exec -it <pod-name> -c <c-name> -- /bin/bash

# 查看 Pod 的日志，多个容器的话默认看第一个容器
kubectl logs <pod-name>
# 查看容器内的日志 -c
```



### 3. **Deployment 操作**

- **查看所有 Deployments：**

  ```
  kubectl get deployments
  ```

- **查看指定命名空间下的 Deployments：**

  ```
  kubectl get deployments -n <namespace>
  ```

- **查看 Deployment 详情：**

  ```
  kubectl describe deployment <deployment-name>
  ```

- **创建 Deployment：**

  ```
  kubectl create deployment <deployment-name> --image=<image-name>
  ```

- **更新 Deployment：**

  ```
  kubectl set image deployment/<deployment-name> <container-name>=<new-image>
  ```

- **滚动更新 Deployment：**

  ```
  kubectl rollout restart deployment/<deployment-name>
  ```

- **查看 Deployment 的滚动更新状态：**

  ```
  kubectl rollout status deployment/<deployment-name>
  ```

- **回滚 Deployment：**

  ```
  kubectl rollout undo deployment/<deployment-name>
  ```

### 4. **Service 操作**

- **查看所有 Services：**

  ```
  kubectl get svc
  ```

- **查看指定命名空间下的 Services：**

  ```
  kubectl get svc -n <namespace>
  ```

- **创建 Service：**

  ```
  kubectl expose pod <pod-name> --port=<port> --name=<service-name>
  ```

### 5. **ConfigMap 和 Secret 操作**

- **查看所有 ConfigMaps：**

  ```
  kubectl get configmap
  ```

- **查看指定 ConfigMap 详情：**

  ```
  kubectl describe configmap <configmap-name>
  ```

- **创建 ConfigMap：**

  ```
  kubectl create configmap <configmap-name> --from-literal=<key>=<value>
  ```

  或者是用写一个yaml文件，kind为ConfigMap类型，然后apply就有，例子：

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: calico-config
    namespace: kube-system
  data:
    CALICO_NETWORKING_BACKEND: "bird"
    CALICO_IPV4POOL_CIDR: "192.168.0.0/16"
    CALICO_IPV4POOL_NAT_OUTGOING: "true"
    CALICO_LOG_LEVEL: "info"
    CALICO_ENABLE_NETWORK_POLICY: "true"
    # IPv6 配置
    IP6: "autodetect"
    IP6_AUTODETECTION_METHOD: "can-reach=fe80::ecee:eeff:feee:eeee interface=ens192"
    CALICO_IPV6POOL_CIDR: "2000:100:100:100::/64"
    CALICO_IPV6POOL_IPIP: "CrossSubnet"
    FELIX_IPV6SUPPORT: "true"
  ```

  

- **修改 ConfigMap:**

  ```
  kubectl edit configmap calico-config -n kube-system
  ```

- **查看所有 Secrets：**

  ```
  kubectl get secrets
  ```

- **查看指定 Secret 详情：**

  ```
  kubectl describe secret <secret-name>
  ```

- **创建 Secret：**

  ```
  kubectl create secret generic <secret-name> --from-literal=<key>=<value>
  ```

### 6. **Pod 模拟调度与资源管理**

- **限制资源（CPU、内存）使用：** 在 Pod 的 YAML 文件中添加：

  ```
  yaml复制代码resources:
    requests:
      memory: "64Mi"
      cpu: "250m"
    limits:
      memory: "128Mi"
      cpu: "500m"
  ```

- **查看 Pod 的资源使用情况：**

  ```
  kubectl top pod <pod-name>
  ```

### 7. **其他常用命令**

- **查看 Kubernetes 资源（如 pod、service 等）的 YAML 配置：**

  ```
  kubectl get <resource-type> <resource-name> -o yaml
  ```

- **删除 Kubernetes 资源：**

  ```
  kubectl delete <resource-type> <resource-name>
  ```

- **获取资源的 YAML 配置并修改：**

  ```
  kubectl get <resource-type> <resource-name> -o yaml > resource.yaml
  kubectl apply -f resource.yaml
  ```

- **切换上下文（集群）配置：**

  ```
  kubectl config use-context <context-name>
  ```

- **查看命令帮助信息：**

  ```
  kubectl <command> --help
  ```







## 部署脚本

需要修改的部分

1. ClusterFile
2. kubeadm
3. kubelet
4. 



