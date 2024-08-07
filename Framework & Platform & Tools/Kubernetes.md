### Cloud Native

云原生是在云计算环境中构建、部署和管理现代应用程序的软件方法。

# Kubernetes Architecture

Kubernetes is a system for running and coordinating containerized applications across a cluster of machines.

### Cluster

Kubernetes brings together individual physical or virtual machines into a cluster using a shared network to communicate between each server. This cluster is the physical platform where all Kubernetes components, capabilities, and workloads are configured.

### Master Node (control-plane)

The control plane's components make global decisions about the cluster, exposing an API for users and clients, health checking other servers, deciding how best to split up and assign work (scheduling), and orchestrating communication between other components.

Control plane components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all control plane components on the same machine, and do not run user containers on this machine.

Components:

- etcd

  Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.

- kube-apiserver

  Exposes the Kubernetes API, which is the front end for the Kubernetes control plane. It is used cluster-internally by the master components, the worker nodes, and your Kubernetes-native apps, as well as externally by clients such as *kubectl*.

- kube-controller-manager

  In Kubernetes, controllers are control loops that watch the state of your cluster, then make or request changes where needed. Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process. Typical controllers are:

  - Node controller: Responsible for noticing and responding when nodes go down.
  - Job controller: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
  - Storage controller
  - Deployment controller
  - **Workflow controller**
  - ...

- kube-scheduler

  watches for newly created Pods with no assigned node, and selects a node for them to run on.

- cloud-controller-manager (optional)

### Node Components

Nodes are responsible for accepting and running workloads using local and external resources. The node receives work instructions from the master server and creates or destroys containers accordingly, adjusting networking rules to route and forward traffic appropriately.

Components:

- container runtime

- kubelet

  The kubelet is the primary "node agent" that runs on each node. It makes sure that containers are running in a Pod. It can register the node with the apiserver using one of: the hostname; a flag to override the hostname; or specific logic for a cloud provider.

- kube-proxy

  kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes **Service**.

### Context

A kubernetes context is just a set of access **parameters** that contains a Kubernetes cluster, a user, and a namespace. kubernetes Context is essentially the **configuration** that you use to access a particular cluster & namespace with a user account.

### Operator

A Kubernetes operator is an application-specific **controller** that extends the functionality of the Kubernetes API to create, configure, and **manage instances** of complex applications on behalf of a Kubernetes user.

### API

The Kubenetes API exposes an HTTP API that lets end users, different parts of your cluster, and external components communicate with one another, and lets you query and manipulate the state of objects in Kubernetes.

Most operations can be performed through the command-line interface (e.g. kubectl), which in turn use the API. However, you can also access the API directly using REST calls.

### Resources

A resource is an endpoint in the API that stores a collection of objects of a certain kind.

A costom resource allows users to create and define their own API objects.

A **Custom Resource Definition (CRD)** is what you use to define a Custom Resource.





# Kubenetes Objects

Kubernetes objects are persistent entities representing the state of the cluster. To work with Kubernetes objects--whether to create, modify, or delete them--you'll need to use the Kubernetes API.

Almost every Kubernetes object includes two nested object fields that govern the object's configuration: the object *`spec`* and the object *`status`*.

When you create an object in Kubernetes, you must provide some information about the object. Kubernetes API require to include that information as JSON in the API request body. Most often, you **provide the information to `kubectl` in a .yaml file**, and `kubectl` converts the information to JSON when making the API request (by `kubectl apply`). In the `.yaml` file for the Kubernetes object you want to create, you'll need to set values for the following fields:

- `apiVersion` - Which version of the Kubernetes API you're using to create this object
- `kind` - What kind of object you want to create
- `metadata` - Data that helps uniquely identify the object, including a `name` string, `UID`, and optional `namespace`
- `spec` - What state you desire for the object, different for every Kubernetes object.

### Pods

A pod is the most basic unit that Kubernetes deals with, which **generally represents one or more containers** that should be controlled as a single application. Pods consist of containers that operate closely together, share a life cycle, and should always be scheduled on **the same node**. They are managed entirely as a unit and share their environment, volumes, and IP space. One container act as the **main** container, and other containers are **helper** containers.

> **Static pods:**
>
> *Static Pods* are managed directly by the kubelet daemon on a specific node. Unlike Pods that are managed by the control plane (for example, a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)); instead, the kubelet watches each static Pod (and restarts it if it fails). The running kubelet periodically scans the configured directory (`/etc/kubernetes/manifests` in our example) for changes and adds/removes Pods as files appear/disappear in this directory.

### Service

Service is an abstract way to expose an application running on a set of Pods as a network service.

Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them. In Kubernetes, a Service is an abstraction which defines a logical set of Pods and a policy by which to access them (sometimes this pattern is called a micro-service). The set of Pods targeted by a Service is usually determined by a selector. A service groups together logical collections of pods that perform the same function to present them as a single entity—frontends do not care which backend they use.

A Service in Kubernetes is a REST object, similar to a Pod. Like all of the REST objects, you can `POST` a Service definition to the API server to create a new instance. 服务类型：

- ClusterIP: 默认模式，Service 只能够在集群内部访问。可以使用 Ingress 或者 Gateway API 向公共互联网公开服务。
- NodePort: 通过每个节点上的 IP 和静态端口（`NodePort`）公开 Service，使 Service 可通过节点端口访问。
- LoadBalancer：使用负载均衡器向外部公开 Service。Kubernetes 不直接提供负载均衡组件，用户须提供，或者将 Kubernetes 集群与某个云平台集成。
- ExternalName：将服务映射到 externalName 字段的内容（例如，映射到主机名 api.foo.bar.example）。 该映射将集群的 DNS 服务器配置为返回具有该外部主机名值的 CNAME 记录。 集群不会为之创建任何类型代理。

### Ingress

An API object that manages **external** access to the services in a cluster, typically HTTP.

### Volumn

Volumes are a abstraction that allows data to be shared by all containers within a pod and remain available until the pod is terminated.

**Persistent volumes** are a mechanism for abstracting more robust storage that is not tied to the pod life cycle. Once a pod is done with a persistent volume, the volume’s reclamation policy determines whether the volume is kept around until manually deleted or removed along with the data immediately.

Persistent Volume(PV), Persistent Volume Claims(PVC), HostPath, EmptyDri, ...

> VolumeAttachment 有时候会报错找不到 `/dev/disk/by-id/csci-3xxxxxxxxx`, 此时可以手动 `ln ../../sda csci-3xxxxxxx` 即可正确 attach PV
>
> 为避免操作系统底层文件，可直接卸载grafana，删除所有相关volumeattachment，不用删除pvc，再部署即可

### Namespace

Namespaces are a way to divide cluster resources between multiple users. Namespaces provide a scope for names. Names of resources need to be unique within a namespace, but not across namespaces.

### Workload

To make life considerably easier, you don't need to manage each Pod directly. Instead, you can use workload resources that manage a set of pods on your behalf.

- ReplicaSet

  A ReplicaSet ensures that a stable set of replica Pods running at any given time.

  Workload objects such as Deployment make use of ReplicaSets to ensure that the configured number of Pods are running in your cluster, based on the spec of that ReplicaSet.

- Deployment

  An API object that manages a replicated application, typically by running Pods with **no local state**.

- StatefulSet

  StatefulSet is the workload API object used to manage stateful applications. It manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.

- Job

  A finite or batch task that runs to completion. (one time)

- CronJob

  Manages a Job that runs on a periodic schedule.

- DaemonSet

  Ensures a copy of a Pod is running across a set of nodes in a cluster. Used to deploy system daemons such as log collectors and monitoring agents that typically must run on every Node.

### Resource, Affinity and Label

A label in Kubernetes is a semantic tag that can be **attached to Kubernetes objects** to mark them as a part of a group.

Resource 包括默认资源如 CPU，Memory，用户也可自定义资源 (Custom Resource Definition, CRD)

Label 是在各级组件如 node，pod 上打的标签，用户也可自定义标签，注意自定义标签的 value 必须被引号 quote！

Affinity 可根据标签进行亲和调度（e.g.数据亲和）或反亲和调度（e.g.每个节点只需要一个proxy）

### GVK vs GVR

GVK: Group Version Kind 

- Each Kind in K8S has Group and Version. i.e. Kind "Pod" is in Group "core" , Version "v1"
- Each GVK map to a given root Go type in the package

GVR: Group Version Resource

- GVR is a "use" or "instance" of GVK in the K8S API
- The command "kubectl api-resources" gives us a list of GVR in the K8S cluster

Scheme:

- The scheme is defined to keep track of a given GO type mapping to a given GVK. 





# Custom Resource Definition (CRD)

CRD:

- CRD stands for Custom Resource Definition
- Each CRD is like Kind in K8S, so it also has Group, Version
- CRD is the extension of the K8S API
- Once it is defined, it acts like GVK in K8S API.

CR:

- CR stands for Custom Resource
- CR is a "use" or "instance" of CRD in the K8S API
- Once it is instantiated, it acts like GVR in K8S API.
- The command "kubectl api-resources" gives us a list including both GVR and CR in the cluster.





# Kubernetes Extensions

### Container Runtime Interface (CRI)

Container Runtime Interface or CRI plugins are here to allow for new CR API to be fully utilized. Runtimes like [Docker ](https://www.docker.com/)can be made more flexible with the right plugin. Naturally, CRI plugins offer one major benefit: they allow you to run different container runtimes without having to recompile.

### Container Network Interface (CNI)

CNI (*Container Network Interface*), a [Cloud Native Computing Foundation](https://cncf.io/) project, consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers, along with a number of supported plugins.

[Calico](https://www.projectcalico.org/) is a popular CNI plugin used by many cluster administrators, offers scalable networking functions using the standard L3 approach. It automatically enables compartmentalized networking in environments like AWS. It also enables seamless networking in on-premise deployments.

### Container Storage Interface (CSI)

CSI offers CSI volumes and dynamic provisioning of storage blocks as functions.

### Scheduler extension





# Kubernetes client-go

### Client

- RESTClient
- **Clientset**: manage kubernetes built-in objects
- DynamicClient
- DiscoveryClient

### Reflector



### Informer



### WorkQueue



### Operator pattern

- CRD
- Controller

### Code Generator

- client-gen
- lister-gen
- controller-gen
- informer-gen
- deep copy-gen





# Cluster API

Cluster API is meant to simplify provisioning, upgrading, and operating multiple Kubernetes clusters.

### Cluster Types

- Bootstrap Cluster

  A bootstrap cluster is used to create the management cluster. It is a temporary cluster typically created locally using kind, and is destroyed once the management cluster is created.

- Management Cluster (control-plane cluster)

  A management cluster is used to create and manage the lifecycle of workload clusters. This is a long-lived cluster. It contains the Custom Resource Definitions (CRD’s) and hosts the CAPI controllers to manage the resources.

  The management cluster hosts three distinct types of components:

  - Cluster API Core Manager

    This controller manager is responsible for managing the lifecycle of the cluster. It understands the Cluster, Machine, MachineDeployment and MachineSet resources which are used to declare a cluster without any specific infrastructure details.

  - Bootstrap Provider

    The purpose of this provider is to generate a cloud-init script that can be used by the infrastructure providers when creating the machines for the clusters. It converts a Machine into a Kubernetes Node. There can be multiple implementations of this provider and each implementation will have its own CRD.

  - Infrastructure Provider

- Workload Clusters

  Workload clusters are provisioned and managed by the management cluster using CAPI resources defined on the management cluster. The workload clusters are not CAPI-enabled and are not aware of the CAPI CRD’s or controllers. Typically, you would end up building multiple workload clusters. The workload clusters are used to host the application workloads.

### Custom Resource Definitions (CRD’s)

- Cluster (like Pod)

- MachineDeployment (like Deployment)

- Machine

  A Machine represents a K8s node. It represents an instance at a provider, which can be any kind of server, like an Azure VM, an AWS EC2 instance or a Raspberry Pi. It is a declarative spec for a platform or infrastructure component that hosts a Kubernetes node such as a bare metal server or a VM. A machine resource is immutable. When a machine spec is updated, the controller deletes the machine and creates a new one.

- MachineSet (like Replicaset)

  The MachineSet controller will create machines based on the defined *replicas* and the machine template.





# Kubenetes Tools

### Kubectl

Kubectl is Kubernetes command line interface used to interact with the cluster.

- `kubectl create`
- `kubectl get xxx` or `kubectl describe xxx` or `kubectl logs xxx`: 查看组件状况
- `kubectl apply -f [FILE_NAME] `: apply a config to a resource by file name (JSON/YAML)
- `kubectl expose`: expose a resource as a new Kubernetes service
- `kubectl run`: create a **pod** and run an image in it
- `kubectl exec`: execute a command on a container in a pod
  - 可用于登陆：`kubectl exec -it my-pod --container main-app -- /bin/bash`

- `kubectl delete`: delete
  - pod 删除卡在 terminating 可使用 `kubectl delete pod <PODNAME> --grace-period=0 --force --namespace <NAMESPACE>`
- `kubectl config`: 
  - view: view context configuration 
  - set-context: create new context
  - get-contexts: list all kubernetes contexts
  - current-context: view the current context
  - use-context: switch to different context
  - `KUBECONFIG=./scheduling-dev-mgmt.kubeconfig:scheduling-dev-wkld.kubeconfig kubectl config view --flatten > scheduling-dev.kubeconfig `: merge config files
- `kubectl top pod -n namespace`: 查看所有pod，nodes中内存，CPU使用情况 
- `kubectl cp <ns>/<podname>:/path/to/src /path/to/dst`: 复制文件
- `kubectl label node <node_name> key1=val1 key2=val2`: 给 node/pod/... 打标签

### Kubeadm

doesn't support MacOS

### kOps

We like to think of it as `kubectl` for **clusters**.

- `kops create`: create cluster, not deployed

- `kops update`: deploy cluster

- `kops validate`: check if the cluseter is ready

- `kops delete`: remove the cluster from the cloud

- ` kops export kubecfg --admin`: export admin login infos to kubeconfig

- `kops create secre `: add login keys for nodes

  > First generate ssh key by:
  >
  > `cd ~/.ssh` and `ssh-keygen -t rsa -b 4096 -f cloud-computing`

- xxx

### Commandl Line Tools

- kubectx: switch Kubernetes context
  - `kubectx`: list available context
  - `kubectx [CONTEXT_NAME]`: switch to context
- kubens: switch Kubenetes namespace
  - `kubens`: list available namespace
  - `kubens [CONTEXT_NAME]`: switch to namespace
- kube-ps1: display the Kubernetes context and namespace
  - `kubeoff`: turn off tube-ps1 status
  - `kubeon`: turn on tube-ps1 status

### K3D vs Minikube vs Kind

##### K3D

- `k3d cluster create [CLUSTER_NAME]`

- `k3d cluster delete [CLUSTER_NAME]`

- `k3d registry create [REGISTRY_NAME] --port [PORT_NUM]`: create a local private registry

- `k3d kubeconfig merge [CLUSTER_NAME] --kubeconfig-merge-default`: Write/Merge kubeconfig(s) from cluster(s) into new or existing kubeconfig/file.

- `k3d node create [NODE_NAME] -c [CLUSTER_NAME]`: 

  Add a node to a cluster

- xxx

##### Minikube

On M1 machines, run the following command to start Minikube:

`minikube start --driver=docker --alsologtostderr`

##### Kind

### Helm vs Kustomize

Helm is a package manager that helps you to find, share, and use software that is built for Kubernetes. Helm streamlines the installation and management of Kubernetes applications, and is the equivalent of the apt, yum, or homebrew utilities for Kubernetes. Helm uses a packaging format called **Chart**.

- add chart repo: `helm repo add [NAME] [URL] [flags]`
- install chart: `helm isntall [NAME] [CHART] [flags]`
- uninstall a release: `helm uninstall [RELEASE_NAME]`
- list all releases: `helm list`

Kustomize is a **built-in** configuration management tool for the Kubernetes ecosystem. It manage the kubernetes objects in a declarative way. Kustomize lets you customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched and usable as is.

- Generate customized YAML: `kustomize build`

### Telepresence

Telepresence is an open source tool that lets you run a **single** service **locally**, while connecting that service to a **remote** Kubernetes cluster. This lets developers working on multi-service applications to: Do fast local development of a single service, even if that service depends on other services in your cluster.

### Kubebuilder







# Cloud Platforms

### Google Cloud Platform

Login:

`gcloud auth login`

`gcloud auth application-default login`

Multiple account: 

`gcloud config configurations`

If you do need to add a new account just invoke the following:

`gcloud auth login`

Delete subnet:

`gcloud compute networks subnets delete`

### Amason Web Service

### Azure



# ETCD

**etcd 是一种开源的分布式统一键值存储**，用于分布式系统或计算机集群的共享配置、服务发现和的调度协调。 etcd 是 Kubernetes 的首要数据存储，也是容器编排的实际标准系统。

- list values: `kubectl exec -it ds-core-etcd-0 -- etcdctl get --prefix /`
- delete values: `kubectl exec -it ds-core-etcd-0 -- etcdctl del --prefix /`





# 加载GPU资源

- 安装 nvidia-container-toolkit 和 nvidia-docker2
  - 下载并安装 gpg key
    - `wget https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-keyring_1.0-1_all.deb`
    - `sudo dpkg -i cuda-keyring_1.0-1_all.deb`
  - 正确配置 `/etc/apt/sources.list.d/nvidia-container-toolkit.list`
  - `apt-get update` + `apt-get install nvidia-container-toolkit nvidia-docker2`
- 安装 nvidia-docker2 后，有选项可以自动修改 `/etc/docker/daemon.json` 并在 `runtimes` 中添加 `nvidia` 的路径等，但并未将其设置为默认，需要在文件中新增 `"default-runtime": "nvidia"`
- 更新 systemd 配置并重启 docker daemon: `systemctl daemon-reload` + `systemctl restart docker`
- 在集群中每个节点部署 `nvidia-device-plugin`: `$ kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/1.0.0-beta4/nvidia-device-plugin.yml `
- 创建 pod 时可指定资源类型为`nvidia.com/gpu`
