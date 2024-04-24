# Ray

Ray 的出发点加速机器学习的调优和训练的速度。支持分布式计算和 Python Pandas/Numpy。Ray 除了基础的计算平台，还包括 Tune (超参数调节) 和 RLlib (增强学习)。

### Ray编程模型

- 任务(Task)，即 worker 执行一个无状态函数
- 行动器(Actor)，即 worker 管理一个一个有状态 class

### Apache Arrow 

Ray 的底层内存数据结构基于 Apache Arrow，一种列式内存数据结构，已成为数据处理领域最通用的数据结构。

### Plasma

Plasma 是 Ray 团队基于 Arrow 开发的一个内存数据服务。Plasma 在 Linux 共享内存创建了 Arrow 封装的对象，单独作为一个进程运行。其他进程可以通过 Plasma Client Library 来访问这块共享内存里的 Arrow 存储。

### 部署

开源社区有完整的解决方案 Kuberay 项目。每个 Ray Cluster 由 Head 节点和 Worker 节点组成，每个节点是一份计算资源，可以是物理机、Docker，K8s Pod 等。

### Ray Data

基于 Ray，Ray Data 负责数据的读取与存储，数据转换，机器学习特征预处理等。Ray Data is a scalable data processing library for ML workloads. It provides flexible and performant APIs for scaling Offline batch inference and Data preprocessing and ingest for ML training. Ray Data uses streaming execution to efficiently process large datasets.



# Dask

Dask 的目标是为了弥补在数据科学使用 Python 时性能上的不足。同样支持分布式计算和 Pandas/Numpy。

Dask 的底层数据结构是 xarray/xray，是 NumFocus 赞助的开源类似 numpy 的数据结构。

