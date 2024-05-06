# docker swarm

## 核心概念

| 概念    | 解释                                                         |      |
| ------- | ------------------------------------------------------------ | ---- |
| Swarm   | 集群的管理和编排是使用嵌入docker引擎的SwarmKit，可以在docker初始化时启动Swarm模式或者加入已存在的Swarm |      |
| Node    |                                                              |      |
| Task    |                                                              |      |
| Service | 一组任务的集合，服务定义了任务的属性。服务有两种模式：<br />replicated services：按照一定规则在各个工作节点上运行指定个数的任务<br />global services：每个工作节点上运行一个任务<br />两种模式通过docker service create 的 --mode参数指定。 |      |

## 集群管理

```bash
docker swarm init 
```

