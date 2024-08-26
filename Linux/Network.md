# socket

### address struct

- sockaddr: generic descriptor for any socket operation (in `sys/socket.h`)
- sockaddr_in: specific descriptor for IPv4 communication (in `netinet/in.h`)

This is a kind of "polymorphism" : the `bind()` function pretends to take a `struct sockaddr *`, but in fact, it will assume that the appropriate type of structure is passed in; i. e. one that corresponds to the type of socket you give it as `bind()`'s first argument.

###  POSIX socket functions

- `socket`: creates an endpoint for communication.
  
  `(int domain, int type, int protocol) -> int`: returns a socket descriptor.
  
  - `domain`: specifies a communications domain. This implicitly selects the protocol family (defined in ⟨sys/socket.h⟩) to be used. e.g. `PF_INET` for IPv4
  - `type`: semantics of communication
    - `SOCK_STREAM`: sequenced, reliable, two-way connection based byte streams.
    - `SOCK_DGRAM`: supports datagrams (connectionless, unreliable messages of a fixed (typically small) maximum length)
    - `SOCK_RAW`: provide access to internal network protocols and interfaces
  - `protocal`:  通常设置为 `0`，因为协议一般是根据`domain` 和 `type `自动选择的。
  
- `setsockopt`: set options on sockets

  `(int socket, int level, int opt_name, const void *opt_val, socklen_t opt_len)` -> int

  - `level`: 被设置的选项的生效的级别，给定的 `opt_name` 都属于的特定的级别

    - `SOL_SOCKET`：套接字的通用可选项，选项名以 `SO_` 开头
    - `IPPROTO_TCP`： TCP 协议的相关可选项，选项名以 `TCP_` 开头
    - `IPPROTO_IP`：IP 协议的相关可选项，选项名以 `IP_` 开头

  - `opt_name`: 

    - `SO_REUSEADDR`: 设置 `SO_REUSEADDR` 选项可以让你的程序在 `TIME_WAIT` 状态下重新绑定端口，这在开发和测试时非常有用，但在生产环境中应谨慎使用，确保不会引发端口重用导致的潜在问题。

      > 关闭 TCP 协议套接字后，操作系统会将其置于 `TIME_WAIT` 状态，确保所有传输的数据包在连接关闭后都能被正确处理，这是 TCP 协议的一部份。`TIME_WAIT` 状态的持续时间通常是 2 倍的最大报文段生存时间（约数几分钟），在此期间该端口将被占用，无法立即重新使用。

    - ... TODO

  - `opt_val`: 指向存放选项值的缓冲区的指针。（某些选项值可能是复杂数据结构，而不仅仅是整数）

  - `opt_len`: `opt_val` 缓冲区的长度。

- `bind`: 

- `accept`: 

- `send`: send message to another socket

  `(int socket, const void *buffer, size_t length, int flags) -> ssize_t `

- `recv`: 

  > VS `read` & `write`: `(int __fd, const void *__buf, size_t __nbyte) -> ssize_t `
  >
  > 都可以向套接字发送数据，但 `send` 可以额外通过 `flag` 控制发送行为，提供了更多的灵活性，是更好的选择。（`flag=0` 时他们是等价的 ）

- `connect`

### with epoll

Todo





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
- ifconfig: displays information about all network interfaces currently in operation 网络接口信息
- iwconfig: 查看无线网卡
- ethtool: 查询配置网卡参数等
- IP/port usage: `netstat -tulpn | grep LISTEN`
- 端口进程占用: `lsof -i:10002`
- Check ssh service: `ps -ef | grep ssh`
- capture tcp packet: `tcpdump -i any 'dst port 31220'`