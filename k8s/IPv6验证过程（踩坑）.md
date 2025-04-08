数据库

```sql
clu_info				# 集群基本信息
clu_detail				# 集群关联的ip
RES_RESOURCE			# 主机资源
SYS_TENANT				# 租户，包含镜像仓库账号密码

CLU_OPERATE_APPLY		# 执行中的任务
    
EE_EXEC_JOB				# 作业任务基本信息，看到作业任务类型
	EE_EXEC_JOB_DETL		# 作业任务详细任务步骤信息
EE_EXEC_CMD				# 根据作业任务类型名称，找到类型ID
EE_EXEC_CMD_SCRIPT		# 根据类型ID找到任务需要执行的脚本ID
EE_EXEC_SCRIPT			# 作业所需要执行的脚本
-- 对应作业任务类型所需要执行的脚本
select ecs.EXEC_SCRIPT_ID, es.EXEC_SCRIPT_NAME, es.SCRIPT_FILE, es.EXEC_CMD
from EE_EXEC_CMD_SCRIPT ecs Left Join EE_EXEC_SCRIPT es
    on ecs.EXEC_SCRIPT_ID = es.EXEC_SCRIPT_ID 
    where ecs.EXEC_CMD_ID = 'EB8F93D8926EE8911ECD2F0BDB54C187';
    
    
```





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







#### 部署集群间的问题

##### 清空环境

- master不小心删除了，但是程序正常运行的原因是：`项目是依赖calico之间运行的，k8s只是负责容器的调度`（去问问这个是什么原理）

  **Master** 节点不可用，但程序正常运行的原因是：项目依赖于 Calico 提供的容器间网络通信，而 **K8s** 主要负责集群的管理和调度，容器的实际运行依赖于 **worker** 节点。

- ssh-copy-id 10.1.12.172 直接执行免密操作

  ```shell
  # 删除本地的旧私钥和公钥文件
  rm -f ~/.ssh/id_rsa ~/.ssh/id_rsa.pub
  # 生成密钥
  ssh-keygen -t rsa -b 2048
  ```

  

- 如果没有 sealos，直接从其他有的复制过来即可`scp -r /usr/bin/sealos 10.1.12.172:/usr/bin/sealos`
  - 执行 sealos 前要把关于 k8s 的文件尽量清楚干净，删除对应占用端口的进程。（git上有对应的命令）

    新的 sealos 版本需要把 containerd 一同清除掉。

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
    sudo rm -rf /var/lib/etcd 
    
    #{pf_step_msg}:判断kubelet、kube-proxy进程是否存活"
    ps -ef | grep kube-proxy | grep -v grep | awk '{print $2}' && ps -ef | grep kubelet | grep -v grep | awk '{print $2}'
    
    #{pf_step_msg}:清理进程(kubelet、kube-proxy)
    ps aux|grep kube|grep -v grep |awk '{print $2}'|xargs kill -9 
    
    sudo systemctl stop containerd
    
    sudo rm -f /usr/bin/containerd
    sudo rm -f /usr/bin/containerd-shim
    sudo rm -f /usr/bin/containerd-shim-runc-v1
    sudo rm -f /usr/bin/containerd-shim-runc-v2
    sudo rm -f /usr/bin/containerd-stress
    sudo rm -f /usr/bin/ctr
    sudo rm -f /usr/bin/crictl
    sudo rm -rf /etc/containerd
    sudo rm -rf /var/lib/containerd
    sudo rm -rf /var/log/containerd
    sudo rm -f /etc/systemd/system/containerd.service
    sudo rm -rf /opt/containerd
    sudo rm -rf /etc/ld.so.conf.d/containerd.conf
    # 或者
    sudo rm -f /lib/systemd/system/containerd.service
    
    ```
    
    重复执行需要清空 
    
    - `sealos delete --nodes 10.1.12.236 --cluster wu9a3duzktct`（旧版本）
    
    - `sealos reset --masters 10.1.12.172 --nodes 10.1.12.234 --cluster fortest1111 --force`（新版本）
      
      - 执行结果如果直接几行，那再部署就会失败
      - 执行结果如果有很多行，再部署就能成功
      
    - sealos reset --masters='10.1.12.172' --nodes='10.1.12.234' --cluster fortest1111 --force
      
    - 看情况执行
    
      ```shell
      # 清空sealos创建的hostsName
      sed -i "/"apiserver.cluster.local"/d" /etc/hosts
      sed -i "/"lvscare.node.ip"/d" /etc/hosts
      
      ```
    
      
    
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

  如错误提示，其实问题就是免密登入问题。对122的免密登入（**对自身的免密登入**）。

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

  12.20 用5.0基础Clusterfile跑也会出现这个问题。(乌龙，用了含有v6的配置)但是为什么有v6就会失败呢？成功了，是新版部署过程问题。

  

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

     ​	开始怀疑是不是kubelet配置的问题。

     发现sealos官方有双栈的安装方式，1.24需要4.0版本以上的才行。了解到我们版本是4.1左右的，并且做了改动（root权限）。
  
     - 官方提供的版本 podSubnet 和 serviceSubnet 是需要两个值的
       
       ```yaml
       networking:
         podSubnet: 172.20.0.0/22,fd00:2222::/112                 # IPv6 和 IPv4 双栈 Pod 子网
         serviceSubnet: 10.86.0.0/22,fd00:1111::/116              # IPv6 和 IPv4 双栈服务子网
       ```
       
       但是我自己执行会出现：
       
       `[EROR] Applied to cluster error: failed to init generator config init kubeadm config error: invalid CIDR address: 172.20.0.0/22,fd00:2222::/112`
       
       意思就是格式错了，不能放两个值，但是官网的就这样，所以开始怀疑是不是sealos的版本问题。
       
       https://sealos.run/docs/5.0.0/developer-guide/lifecycle-management/advanced-guide/dual-stack-cluster/
       
       一查，确实是版本问题，想要双栈版本需要 >= 4.3，最后选择将sealos升级到了5.0，然后官方步骤生成Clusterfile
       
       ```shell
       sealos gen harbor.paas.nl:10081/paas_public/kube-system/kubernetes:v1.24.2 \
         --masters 10.1.12.172 --nodes 10.1.12.236 \
         --env criData=/paas/containerd,registryDomain=harbor.paas.nl,registryUsername=admin,registryPassword=Nlpaas123 \
         --pk /root/.ssh/id_rsa --user=root --cluster wu9a3duzktct  > /root/cluster-file/Clusterfile
       ```
       
       然后再通过相同的命令执行Clusterfile，加上`--debug`可以看到更多的详细信息，有出错也好看些
       
       `sealos apply -f Clusterfile --debug`
       
       然后就遇到了新的问题
       
       ```sheel
       Error: failed to init masters: master pull image failed, error: run command `kubeadm config images pull --cri-socket unix://  --kubernetes-version v1.24.2  -v 6` on 10.1.8.11:22, output: I1219 14:00:44.430914  419414 interface.go:432] Looking for default routes with IPv4 addresses
       
       I1219 14:00:44.431229  419414 kubelet.go:214] the value of KubeletConfiguration.cgroupDriver is empty; setting it to "systemd"
       exit status 1
       output: time="2024-12-19T14:03:15+08:00" level=fatal msg="pulling image: rpc error: code = Unknown desc = 
       failed to pull and unpack image \"k8s.gcr.io/kube-apiserver:v1.24.2\": failed to resolve reference \"k8s.gcr.io/kube-apiserver:v1.24.2\": failed to do request: Head \"https://k8s.gcr.io/v2/kube-apiserver/manifests/v1.24.2\": dial tcp 173.194.203.82:443: i/o timeout"
       ```
       
       这里面的问题主要是因为，无法从`k8s.gcr.io`仓库下载组建，因为k8s要初始化要这些组件，但我们是内网，无法访问外网。
       
       通过用户交流群后发现，**sealos自己是有提供专门的k8s镜像**。解释说是`本质上是使用rootfs特性 做了一个压缩包，内部一堆初始化脚本分发到各个节点`。总而言之就是要使用sealos提供的，否则后续部署就是会出错（这里的出错应该就指的是初始化的错误）。
       
       - 那就下载镜像推到本地仓库吧，然后再重试之前的步骤，发现还是相同的问题，依旧是这个问题`failed to pull and unpack image \"k8s.gcr.io/kube-apiserver:v1.24.2\":`
       
       - 得知要跟换镜像后，第一时间跑去跟礽武交流，发现我们自己的1.24.2是有改版过的，改的是：初始化的时候需要到k8s的仓库（k8s.gcr.io）下载组建的这个步骤改成到自己的harbor仓库。听到的时候感觉天塌了，难道我们要再改造一个k8s镜像，虽然听起来改的东西不会很多，但是不会想动这块代码。将风险暴露上级。
       
       最后群里另外一个工作人员，额外解释了两嘴：`1.跟换了sealos版本的镜像还是出现相同的错误，那就删掉本地Clusterfile 2.默认会走sealos内部仓库缓存的镜像的 无需外网环境`，也就是说他们sealos提供的k8s镜像是可行的。第一个有点不太明白，但是想了想应该是`.sealos/xx`内对应集群名称内的文件，那我将这整个删除掉
       
       最后总结，就是Clusterfile缓存的问题，删除掉.sealos内对应的集群后，再次部署，成功将集群跑起来了（当然基础版本的，无helm\calico，不是ipv6，因为两个都换了版本，先试验是否可行）
       
       补上IPV6后再试试看。
       
       
     
     
  
  
  
  
  
  除此之外，还有个小问题： 2. serviceSubnet 内ipv6的前缀长度要 >= 108
  
  查询问题得知，serviceSubnet 与 podSubnet 的私有网段是不能相同的，要不ip会冲突
  
  

### 试验记录：	

12.16

周一 页面部署测试，执行引擎部署过程慢，研究改成脚本部署

周二 脚本部署测试，ipv6部分，逗号出错，研究sealos要升级

周三 sealos版本升级，执行的脚本需要改造，命令有变动

周四 测试新脚本，出错，询问群，镜像要换成sealos提供的

周五 单独增加ipv6的配置出现老问题关于kubelet相关的，现在尝试用默认gen的配置，额外增加helm和calico测试；不行，出现找不到172问题，现在再尝试昨晚成功的简单的案例是否可行；不行，出现新的错误:

- ```shell
   Error: failed to init masters: init master0 failed, error: 
   run command `kubeadm init --config=/var/lib/sealos/data/wu9a3duzktct/etc/kubeadm-init.yaml \
   	--skip-certificate-key-print --skip-token-print -v 0 --ignore-preflight-errors=SystemVerification` 
   on 10.1.12.172:22, output: W1220 10:12:35.410191   51457 initconfiguration.go:306] 
   error unmarshaling configuration schema.GroupVersionKind
   {Group:"kubelet.config.k8s.io", Version:"v1beta1", Kind:"KubeletConfiguration"}: strict decoding error: 
   unknown field "containerRuntimeEndpoint", unknown field "imageServiceEndpoint", unknown field "localStorageCapacityIsolation"
  ```

  就是这几个字段无法解析（再此之前，额外在当前执行的机子上清空了环境，kubelet状态dead，待确认是不是这个问题，执行`sudo systemctl daemon-reload` 后，这个dead的kubelet就不见了，想`systemctl restart kubelet`重启都不行，一直 no found。）；

- 最后解决：问东问西的一圈后发现，群里也没有具体方案。最后自己跟换机子部署的时候发现（122 换成 172，用122当master（认为应该是配置被搞乱了，122没动过比较干净，就想着试试看）），执行新的Clusterfile的时候一直跟我说我用了重复的hostName，我打开host后发现其实并么有，反倒是有sealos额外添加的几个

  ```shell
  10.103.97.2 apiserver.cluster.local
  10.1.12.236 lvscare.node.ip
  ---
  10.1.12.122 apiserver.cluster.local
  ```

  我将这个移除后，再执行就没有这个错误了，然后再执行又是另外的错误（就是那些历史的Clusterfile配置被执行了），我就很疑惑，我都换了机器怎么还是会这样？就这样又发现了，每台机子上都有个.sealos，然后将这个内对应的集群删除后，就能成功的再次apply起来。

  

额外小问题：新版本，按照旧的方式rm，似乎不能卸干净，再次安装会失败，得再卸载再安装才能成功

- 部署成功后，无法通过 sealos reset 的方式清空

- 旧方式清空环境后再apply，会失败，但这时再reset，就能执行成功，但，对应的虚机的harbor的hostName会被清掉（不影响）

- 这时候再执行apply就能成功部署

  由此新问题：

  部署成功后虚机的harbor的hostName的ip会被改成master的ip。
  
- **最新研究结果（12.26）**，若是没有改动什么，无需清空掉hostname和缓存，只要reset执行成功，便能再次部署成功。



12.23

周一： 成功将一整个双栈搭建起来，其中一个小问题就是calico的tigera没能成功将calico其他的pod拉起来，进入describe查看后发现是ipv4的blockSize设置小了。

```yaml
calicoNetwork:
  # Note: The ipPools section cannot be modified post-install.
  ipPools:
    - blockSize: 22
      cidr: 172.20.0.0/24    # 这里是24，size却是22，size调整一致或大于前缀大小即可解决问题
      encapsulation: VXLAN
      natOutgoing: Enabled
      nodeSelector: all()
```

周二：修改执行引擎，测试页面化部署，都是些小问题，只是重新部署一次执行引擎有点久

周三：修改执行引擎

周四：容器云执行报错后，在master上出现rm、sudo、systemctl命令消失；再次出现组件要在外网仓库下载的情况，可能是缓存问题，因为失败后我再次执行reset后就能成功部署，另外值得一提的事reset的命令参数--cluster似乎必须要加，要不就等于没reset。执行到后半部分：
需要执行`sed -i 's/10.1.12.122 harbor.paas.nl/10.1.12.220 harbor.paas.nl/' /etc/hosts`

周五：helm一直出错，一开始是calico的，这个还好，版本错误; 另一个是prometheus，这个是端口区间问题。



12.30日

周一：页面部署能成功执行，然后查看pod后发现大部分镜像拉取失败，排查后发现，containerd的配置`/etc/containerd/config.toml`的内容没有对应的80和10081端口配置，补上就都能成功拉起启动，但是，在install执行引擎脚本里没有对应的修改方法，起初以为在.sealos内的Clusterfile是关键点，最后才发现，这是sealos将执行完后补充数据的文件放在了此处，而不是由这个执行的，所以改错了位置。研究来研究去，最后决定直接进行复制。

周二：通过昨天的配置发现还是有些问题，calico和prometheus跑不起来，想着是不是`/etc/containerd/cert.d`这个文件夹的内容没有复制，这个文件内包含着`config.toml`内配置的harbor地址的文件夹，有几个端口就对应几个文件夹，比如：`harbor.paas.nl:80`、`harbor.paas.nl:10081`，里面有个`hosts.toml`。

了解到这些后，并不期望手动创建这些文件，然后研究半天无果，最后还是通过执行引擎脚本生成文件夹和内部内容（成功挖坑）。即使这样子还是发现无法将着两个类型的pod启动起来。就又开始一个一个研究对比。然后再发现，原先版本的sealos和新版本的sealos所创建的`config.toml`是不相同的，然后做了替换。但是还是不行，继续排查。遇到174的v6升级，阻塞。

周三元旦

周四：继续周二的排查，一次发现两个问题，一个是hosts对应的文件权限不对（改成了755，但是这个问题应该没有太大影响），另外一个就是导致这所有问题关键点，所创建的`hosts.toml`配置，==里面的 http 多了个 s==，将其修正后，IPv4就都正常了。

接下来开始研究如何上级ipv6，修改了脚本，增加上对应内容，开始测试，发现原本的serviceIp这个配置没有改成功过，做调整废了些时间。能够成功部署后发现calico的配置没有改成v6。万事俱备后，再次部署都成功后，发现122上的calico和prometheus没办法跑起来，其他的都正常，logs和describe后发现有ipv6的ip冲突，那就将冲突的移除掉。成功跑起来。

```shell
# 移除冲突ip命令
sudo ip -6 addr del 2001:db8::100/32 dev ens192
sudo ip -6 addr del 1000:db8::100/32 dev ens192
```



2025.1.6

**周一：**查找扩容脚本、生成参数的具体位置。找到生成位置`CluV1PrepareScript`。计划扩容改会成sealos方式。试验执行命令是否可行。

```sql
select ecs.EXEC_SCRIPT_ID, es.EXEC_SCRIPT_NAME
from EE_EXEC_CMD_SCRIPT ecs Left Join EE_EXEC_SCRIPT  es
    on ecs.EXEC_SCRIPT_ID = es.EXEC_SCRIPT_ID where ecs.EXEC_CMD_ID = 'EB8F93D8926EE8911ECD2F0BDB54C187';
-- EB8F93D8926EE8911ECD2F0BDB54C187 在 EE_EXEC_SCRIPT 对应的是 pf_sealos_scale_out，代码ClusterOpeateServiceImpl中选择了这个类型
-- 结果是当前对应的模板 - 将v2改成v1
```

修改脚本命令后，带讨论sealos扩容是否可行。

**周二：**确认后，放弃使用sealos方式扩容，因为生产就是经常出现问题无法排查，没办法换回来的。只好开始调整原来的模式脚本。

遇到失败任务，想再次扩容说有任务在运行，直接清零`CLU_OPERATE_APPLY`表中的数据。中间发现pmvabs这个文件在进行拷贝的时候找不到文件，经过排查后发现挂载在218里面的`/home/enginepackagebin`内并没有这个pmvabs文件夹。补充后正常脚本进行。后续断续出现拷贝失败问题同理。问题解决后，便可正常扩容成功。无需修改脚本。

**周三：**开始修改剧本中的变量，其中需要额外注意的就是条件判断。经过研究学习后，发现任务中有可进行条件判断的参数

```yaml
- name: 判断为双栈可执行内容
  shell: "echo '10.1.12.220 harbor.paas.nl' >> /etc/hosts"
  become: true							# 用于提升权限的参数，类似于在命令行中使用 sudo。
  connection: local						# 连接类型，用于指定任务的执行方式。local 表示在本地主机上运行，而不是通过 SSH 连接到远程主机。
  when: ip_protocol == "dualstack"		# 会自动解析变量，所以无需{{ip_protocol}}
```

这个叫任务，一整个yaml文件叫剧本（因为有角色、权限等）或自动化脚本；

修改任务脚本的过程就是不断的重试，没有什么卡点。

**周四：**继续脚本的试验，将镜像仓库ip、ipv6、pod的ip范围等改成变量试验。ipv4执行成功，但是calico的tigera跑起来但是logs有错误，一看是calico的yaml.j2有些问题，因为里面包含了ipv6的配置，做调整

```yaml
    ipPools:
      - blockSize: 24
        cidr: 172.20.0.0/24
        encapsulation: VXLAN
        natOutgoing: Enabled
        nodeSelector: all()
{% if IP_PROTOCOL != 'IPV4' %}
      - blockSize: 122
        cidr: fd00:2222::/112
        encapsulation: VXLAN
        natOutgoing: Enabled
        nodeSelector: all()
    nodeAddressAutodetectionV6:
      interface: "eth.*|en.*|em.*"
{% endif %}
    nodeAddressAutodetectionV4:
      interface: "eth.*|en.*|em.*"
{% if CLUSTER_MTU is defined %}
    mtu: {{CLUSTER_MTU}}
{% endif %}
```

.yaml.j2 文件是一个使用 Jinja2 模板引擎编写的 YAML 模板文件。在文件中，通常会包含 Jinja2 模板语法，例如 {{ ... }} 或 {% ... %}，用于动态插值或逻辑处理。

出现多次问题，判断没有效果，后来才发现我的 IP_PROTOCOL != 'IPV4' 都用成了大写，没识别成功。

**周五：**ipv4成功。试验双栈的时候发现错误，是ipv6的参数没有传递下来，导致在部署的时候，master_ipv6这个变量为空，修改Clusterfile任务报错。修改集群保存接口，在入参部分增加ipv6字段，在service内保存cluDetail部分给detail对象增加ipv6参数。





1.13

**周一：**调查试验非root权限sealos命令情况；了解rootless是否可行；下载sealos源码以及golangIDE

创建账户角色`bevanpaas`测试过程。

```shell
# 创建用户
sudo useradd -m -s /bin/bash -c "for test sealos apply" bevanpaas
# 重置密码
sudo passwd bevanpaas
# 检查 /etc/passwd 文件
cat /etc/passwd | grep bevanpaas
# 结果：bevanpaas:x:1002:1002:for test sealos apply:/home/bevanpaas:/bin/bash
# 检查用户主目录是否创建
ls /home

# 为用户分配权限 ： 
sudo usermod -aG sudo bevanpaas 或者是： sudo usermod -aG wheel bevanpaas
# 前提查看系统中是否存在超级用户相关的组：
cat /etc/group | grep -E "sudo|wheel"

# 允许用户在无需输入密码的情况下运行 sudo sealos
# 编辑 sudoers 文件
sudo visudo
# 添加一条规则，允许新用户执行 sealos 时不输入密码。找到类似以下内容的位置：
# User privilege specification
root    ALL=(ALL:ALL) ALL
# 添加新用户的规则：  （确认命令路径：which sealos） 
bevanpaas ALL=(ALL) NOPASSWD: /usr/bin/sealos
```

另外帮忙排查部署失败问题，最后想起来是早上说的执行引擎做测试改arm版本。

**周二：**

1. 今天下载完sealos源码开始查看。
2. 另一件事是，昨天测试的执行引擎代码回退供测试同学测试，回退将我的install脚本也回退了，导致后面测试出了问题，排查看到部署脚本漏了配置，调整了两次。
3. 扩容选机器的时候碰到选不到想要的空闲机子，原因是因为空闲的236，在资源表中是被使用状态，将它改成free即可。
4. 扩容脚本内的k8s选的镜像非sealos版本，所以，测试人员在测试扩进236，缩出172后，calico的apiservcer崩了。kdp后看到拉取的是`https://k8s.gcr.io/v2...`，修改installV2内all.yaml内k8s对应的版本，带测试中。

**周三：**

本想着继续修改sealos源码的，后面测试频繁出问题，就配合着修改：

1. 页面 扩容 缩容 增加字段，检查接口是否有包含IPV6字段
2. 排查为何更换了节点就会一直报错（其中的node一直拉镜像失败，`k8s.gcr.io`问题），排查了好一会儿想起来，是因为hosts，跟之前更换节点尝试部署失败的情况一样。清空sealos在上面配置的hosts，再次部署就全部能ready了。
3. 扩容脚本虽然能成功，但是新加进来的节点还是会出现这个问题（`k8s.gcr.io`），主要是proxy。 ----问题未解决

**周四：**

1. 继续昨天那个问题，昨晚扩容了下236，还是失败，早上想着缩出236再试试看，却一直是资源紧张不让缩，然后手动的删了detail的236的数据，导致后面一直扩容都失败（以172为master的集群都是），后面删掉重新部署以122为master的集群，就能正常扩容了。

2. 下午开始查看具体扩容为什么还是会去`k8s.gcr.io`下载，在项目脚本中哪里都找不到调整的位置，想法很简单，因为想着是独立先把要加进来的节点环境配好（k8s以及组件），中间修改了calico.j2的配置，依旧没有用

3. 问题解决：后面沟通了下，发现，其实这个proxy是个DS，那他的配置其实并不在本地项目，而是在master内。这时候焕然大悟，就是因为是最开始部署的集群我在Clusterfile中明确的指出了harbor地址，但是里面的配置依旧是没有变化，所以当有新的节点扩建来后，读的还是那个错误的配置，导致一直尝试从`k8s.gcr.io`下载镜像。那么我是用命令

   ```shell
   kubectl edit daemonset kube-proxy -n kube-system 
   # 将镜像改成
   image: harbor.paas.nl:80/paas_public/kube-proxy:v1.23.1
   ```

   保存后自动执行，再次查看proxy，就成功Run起来了。然后其余的错误，比如calico也打差不差，也是DS内的镜像不对，直接修改即可正常启动。

4. 补上了卸载的清除多余hosts命令。后续：无用，卸载的剧本会清掉hosts内的配置，但是额外扩进来的却无法一同清除。

**周五：**

1. 剧本中补充修改任务 - 修改sealos部署后的ds内容（proxy，calico-node），这个后期争取在sealos中将其修改掉。然后开始不断的调试位置，一开始还出现任务尽然优先执行问题（有个子task问题，平铺开就没事了）。中间遇到.8机子内存满了，导致无法编译的情况，多次手动清理，仅能多空出4%的内存，完全不够用，后面开始排查内存的问题（存储有着100G的空间，但实际并没用到这么多，df -h 却显示已经用了100%），人工智能建议清除缓存后，直接空出65%的内存。
2. 扩容时长发生`systemctl enable kubelet.service`问题。





1.20

**周一**：修改4.4版本的sealos源码，下午的时候发现有5.0的源码

**周二**：帮忙一同看看出口网关tunl0的问题，研究了路由相关问题，排查tunl0从哪里来的，得知由calico分配。继续调整sealos源码

**周三**：搭建流水线生成二进制文件，提交源码，移除源码中无用部分（frontend等）。在编译过程一直报错。

**周四**：寻找sealos编译所需要的依赖并down下来，多次编译依旧失败。下午和院长他们一起看出口网关的事情，再次细致研究了下路由表的规则是如何出入的（学习细节已经记录到零食知识点中），最后确认tunl0内calico分配的ip是ipip的只支持IPv4，然后想要支持v6的需要使用vxlan或者ip6tnl内的ip。然后重新将122，236部署了双栈供出口网关测试。发现新部署的双栈集群的tunl0并没有被分配Id（原因是calico的配置指定的就是vxlan）

**周五**：

- 早上询问了下宋迪原先是怎么变成称二进制的，还需要下载些什么。后来得知，不需要，只需要将项目fork到自己的github上，然后执行一同fork过来的actions就行。直接打开新世界，虽然以前有看到actions，但是一直没在意这是什么。然后花了些时间开始摸索。
- 下午继续研究，后续被叫去看出口网关怎么回事，因为vxlan.calico内的ip配置在路由表中的规则无法做跳转。一番试验后，发现原来一配置上去关于vxlan.calico的规则，没几分钟就会被清掉。目前不清楚只为什么会被清掉（有可能是因为出错了？）



2.7-8：配合做出口网关脚本修改，部署环境。

2.10

**周一**：找到正确的编译actions，获取到两种版本的sealos（amd、arm），并从自己的github中下载源码做修改准备

**周二**：提交修改部分再次编译，然后见编译后的sealos放进环境测试，发现报错

```shell
[root@k8s-host122 bin]# sealos1
sealos1: /usr/lib64/libc.so.6: version `GLIBC_2.38' not found (required by sealos1)
sealos1: /usr/lib64/libc.so.6: version `GLIBC_2.33' not found (required by sealos1)
sealos1: /usr/lib64/libc.so.6: version `GLIBC_2.34' not found (required by sealos1)
sealos1: /usr/lib64/libc.so.6: version `GLIBC_2.32' not found (required by sealos1)
```

查了下是说少了依赖的包，一开始以为是我更换了的仓库地址导致的，修改回来后还是同样的错误，这样应该就是我选错了action吧，明天再选一个其他能编译成功的下载下来看看。 

**周三**：再尝试自己github内其他的action多次后，发现都是同样的问题。然后跑到官网下载同样构建后的sealos，尽然报同样的错误，这时候的我无敌的疑惑。（周四的解释了下，大概率就是最新版本的sealos，用的东西版本都高了，所以无论哪边下载都是一样的。）

**周四**：使用Relase的action来尝试构建，因为我看到了对应的版本tag，所以认为只要编译完成后，就能生成最终的功能，然后开始试图编译，但是发现缺少了些PAT（其实就是Token），好在之前学习的时候对这个配置过程有印象（先账户生成权限token，然后到仓库里面在配置上对应名称的环境变量）。配置完成后，看构建过程是能走了，但是到后面docker还是因为我的github的账户名称是大写的导致报错，修改后，就都完成了，可是我却找不到文件在何处。

这时候群里里面的大佬回复了说，这个缺包的错误其实就是因为版本问题，要么就是系统太低，要么就是golang版本过高，与之调整就行，我就讲1.20改成1.16，这样之前位子下载的sealos确实就能运行了，不过我看了下版本有些不对，是4.17的，就很疑惑，现在怀疑的情况是源码的问题，要弄下调整

**周五**：使用下载下来的5.0版本的sealos，然后，再试试action是后的sealos是否可行。



2.17

周一：还是停留在action上，升版本降下来的来回尝试，还是不可行

周二：早上通过dp询问后了解到，源码内有Makefile文件可以提供编译，先是在mac的idea上直接编译，但是出问题，而且默认版本还是arm的。所以现在挪到了自己的虚拟机上开始尝试。

```shell
make -f /root/cluster-file/sealos-5.0.0/Makefile -C /root/cluster-file/sealos-5.0.0 build
make -f /root/cluster-file/sealos-5.0.0/Makefile -C /root/cluster-file/sealos-5.0.0 clean
make -f /root/cluster-file/sealos-5.0.0/Makefile -dC /root/cluster-file/sealos-5.0.0 build.multiarch PLATFORMS="linux_amd64"

make -f /root/cluster-file/sealos-5.0.0/Makefile -C /root/cluster-file/sealos-5.0.0 tools
make -f /root/cluster-file/sealos-5.0.0/Makefile -C /root/cluster-file/sealos-5.0.0 gen

make build BINS="sealos" PLATFORMS="linux_arm64 linux_amd64"

make -f /root/cluster-file/sealos-5.0.0/Makefile -C /root/cluster-file/sealos-5.0.0 gen V=1
```

周三：换成自己的虚拟机测试，发现还是老毛病，他不愿意变动，一直出现的是DeepCode-gen错误，就有点像版本没对上的感觉，但是却一直解决不了。

周四：友龙哥介入，先fork了5.0的代码，但是最后的action，问题一样，

周五：这次fork的时候把仅fork main取消了，然后action构建的二进制文件可以执行。提交自己的代码做测试，可行。



2.24

周一：每次上传调试，测试修改仓库地址后，使用以往的k8s版本是否能成功部署起来，后来发现那个地址实质是在k8s内部自己调用的，修改无果，只能继续使用sealos提供的k8s

```shell
rm -f /root/cluster-file/sealos \
rm -f /bin/sealos1

sudo cp /root/cluster-file/sealos /bin/sealos1 \
cd /bin \
chmod 755 sealos1

```

周二：linux创建不同账号，配置过程，需要对sudo进行全配置，否者每次执行sudo命令需要密码

####   用户创建

```shell
sudo useradd -m bevanan  				# 创建用户及家目录
echo "bevanan ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/bevanan  # sudo ALL 权限
sudo chmod 440 /etc/sudoers.d/bevanan  	# 设置正确权限

ssh-keygen -t rsa -b 4096  				# 按提示生成密钥，默认保存在~/.ssh/id_rsa
ssh-keygen -t rsa -b 2048
sudo passwd bevanan  					# 设置临时密码 newlanD181.
ssh-copy-id bevanan@10.1.12.218			# 按提示输入临时密码(其他机子的用户免密到本机的用户)
sudo passwd -d bevanan  				# 可选：删除密码，仅依赖密钥登录

sudo -l  								# 查看bevanan的sudo权限
sudo command  							# 执行命令无需密码
```

用新用户开始开始创建clusterfile，陆续遇到的问题

```shell
> --pk /root/.ssh/id_rsa --user=root --cluster fortest123  > /home/bevanan/cluster-file/Clusterfile
Error: error parsing image name "//harbor.paas.nl:10081/paas_public/kube-system/kubernetes-sealos:v1.24.2": pinging container registry harbor.paas.nl:10081: Get "http://harbor.paas.nl:10081/v2/": dial tcp: lookup harbor.paas.nl on 192.168.30.2:53: no such host
[bevanan@host826 ~]$ vi /etc/hosts
```

后续在使用3台新增的设备后设置用户bevanan，然后之间免密，在sealos gen 的时候使用bevanan用户，后续能正常部署，这让我中间研究actions和makefile就有点尴尬了，如果能走通，他就是一开始给我们了错误的方向。

周三： 

- 申请 等待局方的3台机子做最后的sealos在普通用户下是否能正常执行的验证。

- 研究单独IPV6部署是否可行，sealos在gen的时候填写v6出错，不可行。开始改用江苏的部署方式。研究修改难度

周四：

- 开始动手调整脚本内容，以及数据库内的任务的执行流程。

周五：将部署方式修改成二进制模式



3.3

本周：修改数据库流程、等待局方机子、测试部署过程、将江苏IPv6部分挪过来

```sql
INSERT INTO EE_EXEC_CMD (EXEC_CMD_ID, CMD_CODE, CMD_CATG, CMD_CNNAME, CMD_VERSION, CMD_DESC) 
VALUES ('77C86F2E9C1DBF12E0530F08010AB99D', 'pf_ansible_deploy', 'k8sClu', 'k8s集群部署', '1', '参数：对象key=对象value')

INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609CA457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '84FBb04C1AC5400AA139BC12460EC89B', 0, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609CB457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '6BA3bE691989354AA42ABE39B2CC87CE', 10, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609CC457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', 'F908b7EE80B73E23AFC70CCB621B3AD0', 20, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609CD457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '3CC3b947A35B337DAEF458E46ECDEAFD', 30, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609CE457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '7B04b85FE472393082E60E7C8E7057A9', 30, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609CF457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', 'A6FFb7E108B036519BF1A008BBD95901', 30, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609D0457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '7CCEbABD84BA4E6EAC1EEC795E2E1BF9', 40, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609D1457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '765Ab26CE49934479B345F3B445BBDE0', 50, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609D2457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '1ECFb6F94225350BAE38515FBD841C3Z', 60, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609D3457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', 'E9A0bCEA35DB35809282536B6DB8710Z', 70, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609D4457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', 'A144bDFC44DF498F83FE2CAA2D7B4AAZ', 80, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609D5457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '3982b5FA9AFD4E6C9C62CA542923FF7Z', 80, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609D6457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '9E50b63D90C64A0F8084AB32632F5C4Z', 80, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609D7457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '399CbDD01C8F4AA59B7EEF05419256CZ', 81, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609D8457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '2FF4b0FDFB2B4E339F096B3A0E1C957Z', 80, 0, 'alone', 0);
INSERT INTO EE_EXEC_CMD_SCRIPT(EXEC_CMD_SCRIPT_ID, EXEC_CMD_ID, EXEC_SCRIPT_ID, EXEC_ORDER, IS_ROLLBACK, EXEC_WAY, IS_FIXED) VALUES ('C9B8DAC609D9457FE055020C2941013w', '77C86F2E9C1DBF12E0530F08010AB99D', '12b4b8d21b3c46629e6jnde4e324805Z', 90, 0, 'alone', 0);

INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('84FBb04C1AC5400AA139BC12460EC89B', '脚本准备', '/script/clu/installv3', null, 'com.newland.paas.paasservice.executeenginesvr.script.cluk8s.CluV3PrepareScript', 'com.newland.paas.paasservice.executeenginesvr.script.cluk8s.CluV3PrepareScript', 'javaClass', 0, 1000, 600000, 'PREFIX_ASC');
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('6BA3bE691989354AA42ABE39B2CC87CE', 'k8s集群部署-卸载', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/99.clean.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('F908b7EE80B73E23AFC70CCB621B3AD0', 'k8s集群部署-准备', '/script/clu/installv3', '/script/clu/install', 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/00.prepare.yml ', 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/99.clean.yml', 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('3CC3b947A35B337DAEF458E46ECDEAFD', 'k8s集群部署-keepalived', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/01.keepalived.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('7B04b85FE472393082E60E7C8E7057A9', 'k8s集群部署-etcd', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/02.etcd.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('A6FFb7E108B036519BF1A008BBD95901', 'k8s集群部署-containerd', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/91.containerd.yml', null, 'ansible', 0, 1000, 1200000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('7CCEbABD84BA4E6EAC1EEC795E2E1BF9', 'k8s集群部署-k8s-master', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/04.kube-master.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('765Ab26CE49934479B345F3B445BBDE0', 'k8s集群部署-k8s-node', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/05.kube-node.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('1ECFb6F94225350BAE38515FBD841C3Z', 'k8s集群部署-calico', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/06.calico-rr.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('E9A0bCEA35DB35809282536B6DB8710Z', 'k8s集群部署-kubedns', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/07.kubedns.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('A144bDFC44DF498F83FE2CAA2D7B4AAZ', 'k8s集群部署-更新kube-dns配置', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/11.kubedns.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('3982b5FA9AFD4E6C9C62CA542923FF7Z', 'k8s集群部署-prometheus', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/08.prometheus.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('9E50b63D90C64A0F8084AB32632F5C4Z', 'k8s集群部署-fluentd', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/10.fjfluentd.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('399CbDD01C8F4AA59B7EEF05419256CZ', 'k8s集群部署-eventrouter', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/13.eventrouter.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('2FF4b0FDFB2B4E339F096B3A0E1C957Z', 'k8s集群部署-安装插件', '/script/clu/installv3', null, 'ansible-playbook -i {localPath}/jobtmp/{jobId}/hosts -e @{localPath}/jobtmp/{jobId}/param.json {localPath}/jobtmp/{jobId}/12.plugins.yml', null, 'ansible', 0, 1000, 600000, null);
INSERT INTO EE_EXEC_SCRIPT(EXEC_SCRIPT_ID, EXEC_SCRIPT_NAME, SCRIPT_FILE, SCRIPT_UNDOFILE, EXEC_CMD, EXEC_UNDOCMD, SCRIPT_TYPE, RETRY_COUNT, RETRY_INVL_TIME, TIMEOUT, ROLLBACK_TYPE) VALUES ('12b4b8d21b3c46629e6jnde4e324805Z', 'k8s集群部署-证书返回', null, null, 'com.newland.paas.paasservice.executeenginesvr.script.cluk8s.CluInstallEndScript', null, 'javaClass', 0, 1000, 600000, null);

```





3.10

周一：验证二进制脚本，想debug执行引擎步骤，但是出现了，观测就会失败的问题。等待镜像上传完毕（arm和x86一同上传），尝试登入局方提供的3台机器，发现本地网络不可用。

周二：

- 上午尝试在玻璃房内登入对应的机器，不行。询问是否能配置什么隧道或者跳转机-2.17事故后，设为违规操作。后续小明哥那边得知可使用`10.47.202.140   bomc/As1!_BmWy2046(`机子做跳转，在工具上不知道怎么配置都失败了，但是最后还是登入在这台机子上然后通过`ssh paasopt@10.45.103.128`的方式跳转成功登入。

- 开始通过这个方式对这些机子配置对应的sealos环境，以及免密操作。局方测试很快就将129 130 搭建了起来。问题也出现了

  - 其中corndns启动失败，似乎是对应的镜像没有找到的问题

  - 另一个是kubectl 命令虽然能执行，但是在paasopt用户下无法查看到集群。

    ```shell
    [paasopt@k8s-host21175 root]$ kubectl get nodes
    The connection to the server localhost:8080 was refused - did you specify the right host or port?
    ```

    要加上sudo才能正常看到集群节点。开始猜测是什么原因

    1. 集群给部署到了root上。然后paasopt没办法有效的查看到对应的节点

周三：

- 任务为：在本地环境（174）上通过install来部署（用paasopt用户），然后看看是否还会再出现这个问题。以及还是需要sudo权限的话，那就要排查会有什么影响。

- 下午提供两台机子供我测试（21段的 175、176），开始修改paasopt的密码，将与root独立开。

  ```shell
  # 重制密码
  [paasopt@k8s-host21175 root] sudo passwd paasopt
  ```

  然后在这个用户上在执行 kubectl get nodes，发现问题竟然一样，那就说明确实部署是部在了root上。

  `kubectl` 在 `paasopt` 用户下无法连接集群，但通过 `sudo` 能正常工作，根本原因是 **`kubectl` 的配置文件（`kubeconfig`）的权限或路径问题**。

  原因分析

  1. 默认 kubeconfig 路径不同：
     - 当以 `paasopt` 用户执行 `kubectl` 时，会尝试读取 ~/.kube/config（即 /home/paasopt/.kube/config）。
     - 当以 `sudo` 执行时，会读取 `root` 用户的配置（/root/.kube/config）。
  2. 配置文件权限问题：
     - 如果集群部署时以 `root` 用户操作，生成的 `kubeconfig` 文件默认保存在 /root/.kube/config，且权限为 `600`（仅 `root` 可读）。
     - `paasopt` 用户没有权限访问 /root/.kube/config，且其本地的 ~/.kube/config 可能不存在或配置错误。
  3. 错误提示的含义：
     - 表示 `kubectl` 没有找到有效的集群配置，回退到默认的本地连接（但 Kubernetes API 服务端未在本地运行）。

  问题解决：

  将 `root` 的配置复制到 `paasopt` 用户目录：

  ```bash
  sudo mkdir -p /home/paasopt/.kube
  sudo cp /root/.kube/config /home/paasopt/.kube/
  sudo chown -R paasopt:paasopt /home/paasopt/.kube
  sudo chmod 600 /home/paasopt/.kube/config
  ```



周四：

- 早上部署完执行引擎后发现174不断404，后面排查发现是250没空间了，通过crictl images 和 crictl rmi 清除了执行引擎的镜像后即可。 

- 但是就是在这占满的期间，对应250节点里面的pod开始被驱逐（Evicted）了，例如片段：

  ```shell
  manage       resmgr-64696bf65b-z6c5r         0/1        Evicted         0        14m
  manage       resmgr-64696bf65b-z6jwd         0/1        Evicted         0        7m28s
  manage       resmgr-64696bf65b-z6r9m         0/1        Evicted         0        7m53s
  manage       resmgr-64696bf65b-z98zg         0/1        Evicted         0        11m
  manage       resmgr-64696bf65b-zcbkh         0/1        Evicted         0        14m
  manage       resmgr-64696bf65b-zdblq         0/1        Evicted         0        22m
  manage       resmgr-64696bf65b-zdf5g         0/1        Evicted         0        13m
  manage       resmgr-64696bf65b-zfl9b         0/1        Evicted         0        6m28s
  manage       resmgr-64696bf65b-znwsl         0/1        Evicted         0        6m5s
  manage       resmgr-64696bf65b-zr5sf         0/1        Evicted         0        12m
  manage       resmgr-64696bf65b-zrfnd         0/1        Evicted         0        18m
  manage       resmgr-64696bf65b-zs4bk         0/1        Evicted         0        10m
  manage       resmgr-64696bf65b-zzd8n         0/1        Evicted         0        13m
  manage       resmgr-64696bf65b-zznth         0/1        Evicted         0        14m
  ```

  密密麻麻的特别多重复的，估计就是等的越久，他重试的pod越多。那么需要将其delete就可以（执行过程等了很久，估计有个几百条）

  ```shell
  kubectl get pods --all-namespaces | grep Evicted | awk '{print $1 " " $2}' | xargs -L1 kubectl delete pod -n
  ```

- 前几天的问题找到原因了，一直在脚本准备这个步骤出问题，是因为虽然我讲`pf_ansible_deploy`改回了`pf_seal_deploy`，但是我没重新编译clu镜像。。。。。，排查过程是看了下对应JOB的DETAIL那张表。发现脚本准备步骤走的是V3，才焕然大悟。况且，如果是sealos的话，DETAIL也不应该有那么多的步骤，sealos的只有4条。



周五：

- 原先install内对应的变量

  ```yaml
  --pk {{home.stdout}}/.ssh/id_rsa --user={{ ansible_user }} --clus
  ```

- 遇到问题：审批通过后的部署任务，一直卡住在运行中没有进展 --- 原因，执行引擎卡死，需要删除对应任务重来

- 解决问题：在普通用户下，需要sudo kubectl的问题

  ```yaml
          - name: "{pf_step_msg}:kubernetes config"
            shell: |
              sudo mkdir -p {{home.stdout}}/.kube
              sudo cp /root/.kube/config {{home.stdout}}/.kube/
              sudo chown -R {{ansible_user}}:{{ansible_user}} {{home.stdout}}/.kube
              sudo chmod 600 {{home.stdout}}/.kube/config
            when: ansible_user != "root"
  ```

  

3.17

本周任务主要负责 将部署改成ansible脚本

周一：

周二：成功执行完成脚本准备环节，下一步卸载出问题，排查到找不到文件。

周三：

- 排查有两个结果，一个是找不到param.json，另一个是找不到99.clean.yaml。在执行引擎看了下，是有成功复制了待执行的yaml，看来是路径不同，就看了数据库中script配的，确实少了一截。这下不得不换成与项目一致的方法了（JOB任务，在准备阶段通过generateJobDetl录入）。
- 方法生成成功执行，但是后面的执行出了问题，就是页面是正常的执行，但是只是表面，内部的yaml一点都没执行。排查了下，看起来似乎没找到对应的hosts，这个hosts是带个编码的，询问得知后面根据跳板机再生成的文件。

周四：





4.7

周一：

- 发现问题：kubelet中的service配置文件的要与containerd的一致

  ```
  containerd
  [plugins."io.containerd.grpc.v1.cri"]
      sandbox_image = "{{containerd_harbor_domain}}:{{containerd_harbor_port}}/paas_public/kube-system/pause-{{ 'arm64' if machine == 'arm' else 'amd64' }}:3.7"
  
  kubelet
  {% if machine != "arm" %}
    --pod-infra-container-image={{kubelet_image_repository_path}}/pause-amd64:3.7 \
  {% endif %}
  ```

  最修改的话，就需要修改原harbor路径，做一个新的提交

  ```docker
  docker login http://harbor.paas.nl:10081  -u admin -p Nlpaas123
  docker pull 10.1.12.220/paas_public/kube-system/pause:3.7
  docker tag harbor.paas.nl:10081/paas_public/kube-system/pause:3.7 harbor.paas.nl:10081/paas_public/kube-system/pause-amd:3.7
  docker push harbor.paas.nl:10081/paas_public/kube-system/pause-amd:3.7
  ```
  
  重新部署，问题解决

周二：

- 执行主kube- master的main剧本时，遇到innerCommProt参数未找到情况，不解，明明在prepare里面已经赋值了。





### 最新部分



