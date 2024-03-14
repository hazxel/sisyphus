# namespece & nsenter

https://asphaltt.github.io/post/linux-how-nsenter-works/

在 Linux 系统里，`nsenter` 是一个命令行工具，用于进入到另一个 `namespace`。譬如，`nsenter -n -t 1 bash` 就是进入到 `pid` 为 1 的进程所在的网络 `namespace` 里。

可以通过`docker inspect`得到进程号，从而切换到 kubernetes 中 pod 容器的 namespace：

```c++
#!/usr/bin/env bash

function e_net() {
  set -eu
  pod=`kubectl get pod ${pod_name} -n ${namespace} -o template --template='{{range .status.containerStatuses}}{{.containerID}}{{end}}' | sed 's/docker:\/\/\(.*\)$/\1/'`
  pid=`docker inspect -f {{.State.Pid}} $pod`
  echo -e "\033[32m Entering pod netns for ${namespace}/${pod_name} \033[0m\n"
  cmd="nsenter -n -t ${pid}"
  echo -e "\033[32m Execute the command: ${cmd} \033[0m"
  ${cmd}
}

# 运行函数
pod_name=$1
namespace=${2-"default"}
e_net
```





# iptables

在一个高层 iptables 上可能包含多个表。表可能包含多个链。链可以是内置的或用户定义的。链可能包含多个规则。规则是为数据包定义的。

### Table

1. filter: iptables 的默认表，有内置链 INPUT, OUTPUT, FORWARD
2. nat: 有内置链 PREROUTING, POSTROUTING, OUTPUT
3. mangle: 有内置链 INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING
4. raw: 有内置链 PREROUTING, OUTPUT

### Chain

链名必须大写

### Rule

### Commands

- 查看某个 table：`iptables -t filter --list`
- 查看某个chain



# Commands

- iperf: a tool for active measurements of the maximum achievable bandwidth on IP networks
- ifconfig: displays information about all network interfaces currently in operation
- IP/port usage: `netstat -tulpn | grep LISTEN`
- 端口进程占用: `lsof -i:10002`
- Check ssh service: `ps -ef | grep ssh`
- capture tcp packet: `tcpdump -i any 'dst port 31220'`