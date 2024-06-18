## Blockchain

#### Transaction

A transaction is a data structure that describes the transfer of currency from spenders to recipients. It consists of inputs and outputs. 

- **Inputs** are references to outputs of previous transactions. Inputs reference previous output that is being spent by a $(h, i)$-tuple, where $h$ is the hash of the transaction that created the output, and $i$ specifies the index of the output in that transaction.

- **Outputs** are tuples consisting of an amount of bitcoins and a spending condition. Outputs exist in two states: unspent and spent. An output is originally unspent, and can be spent at most once.

- **Spending conditions** are scripts that offer a variety of options. Apart from a single signature, they may include conditions that require multiple signatures, the result of a simple computation, or the solution to a cryptographic puzzle.

- The set of **unspent transaction outputs (UTXOs)** is part of the shared state of Bitcoin. Every node in the Bitcoin network holds a complete replica of that state. Local replicas may temporarily diverge, but consistency is eventually re-established.

- Node Receive Transaction:

  ```pseudocode
  Receive transaction t 
  for (h,i) in t.inputs do
  	if (h, i) is not in local UTXO set or signature invalid 
  		Drop t and stop
  	end if 
  end for
  if sum of values of t.inputs < sum of values of t.outputs 
  	Drop t and stop
  end if
  for each input (h, i) in t.inputs do
  	Remove (h, i) from local UTXO set 
  end for
  for each output o in t.outputs do 
  	Add o to local UTXO set
  end for
  Forward t to neighbors in the Bitcoin network
  ```


> - New transactions refer to old transactions, which refer to even older transactions. Every transaction can publicly followed back to **coinbase transactions** of blocks.
> - A recipient is paid by a transaction whose output’s spending condition locks the payment with recipient's public key. It can be unlocked and spent in the future if the recipient signs a future transaction with the private key.
> - The outputs of a transaction may assign less than the sum of inputs, in which case the difference is called the transaction fee. The fee is used to incentivize other participants (miners) in the system.







#### Proof-of-Work (PoW)

Proof-of-Work is a mechanism that allows a party to prove to another party that a certain amount of computational resources has been utilized for a period of time. A function:
$$
F_d(c,x)\rightarrow \lbrace true,false\rbrace
$$
where difficulty $d$ is a positive number, and challenge $c$ and nonce $x$ are usually bit-strings, is called a Proof-of-Work function if it has following properties:

- $F_d(c,x)$ is fast to compute if $d$, $c$, and $x$ are given.
- Given $d$ and $c$, finding $x$ such that $F_d(c, x) = true$ is computationally difficult but feasible. The difficulty d is used to adjust the time to find such an x.



#### Block

A block is a data structure used to communicate incremental changes to the local state of a node. A block consists of a list of transactions, a timestamp, a reference to a previous block and a nonce. 

- A block lists some transactions the block creator has added to its memory pool since the previous block. Only transactions contained in a block are said to be $confirmed$.

- A node finds and broadcasts a block when it finds a valid nonce for its PoW function.

- The blocks build a tree through references, rooted in the so called **genesis block**. The genesis block’s hash is hard-coded in the Bitcoin source code.

- The current Bitcoin block generation interval is 10 minutes.

- Node Create Block (Mining):

  ```pseudocode
  block bt = {coinbase tx}
  while size(bt) ≤ 1 MB do
  	Choose transaction t in memory pool that consistent with bt and local UTXO set
  	Add t to bt
  end while
  Nonce x = 0, difficulty d, previous block bt*, timestamp = ts
  challenge c=(merkle(bt), hash(bt*), ts, d)
  while Fd(c, x) = false
  	x = x + 1
  end while
  Gossip bt
  Update local UTXO set using bt
  ```

  - Function $merkle(b_t)$ creates a cryptographic representation of the set of transactions in $b_t$. It is compact and has a fixed length no matter how large the set is.
  - Bitcoin sets the difficulty so that globally a block is created about every 10 minutes.



#### Coinbase Transaction

The first transaction in a block is called the coinbase transaction. The block’s miner is rewarded for confirming transactions by allowing it to mine new coins. The coinbase transaction has a dummy input, and the sum of outputs is determined by a fixed subsidy plus the sum of the fees of transactions confirmed in the block.

- A coinbase transaction is the sole exception to the rule that the sum of inputs must be at least the sum of outputs, which allows new bitcoins to enter the system.
- The number of bitcoins that are minted by the coinbase transaction is determined by a subsidy schedule written in the protocol. Initially the subsidy was 50 bitcoins for every block, and is being halved every 210,000 blocks (or 4 years in expectation). Due to the halving of the value of the coinbase transaction, the total amount of bitcoins in circulation never exceeds 21 million bitcoins.



#### Blockchain

The longest path from the genesis block (root of the tree) to a (deepest) leaf is called the blockchain. The blockchain acts as a consistent transaction history on which all nodes eventually agree.

- Only the longest path from the genesis block to a leaf is a valid transaction history, since branches may contradict each other because of doublespends.

- Since only transactions in the longest path are agreed upon, miners have an incentive to append their blocks to the longest chain, thus agreeing on the current state.

- Node Receives Block:

  ```pseudocode
  Receive block b_t
  Node's current head is block b_max at height h_max
  Extract b_t's previous block b_t*
  Find b_t* in the node’s local blockchain replica, at height h_t*
  h_t = h_t* + 1
  if h_t > h_max and is_valid(b_t)
  	Append b_t to local blockchain replica
    h_max = h_t, b_max = b_t
    Update UTXO set using b_t
  end if
  ```

  The $is\_valid$ function represents the consensus rules of Bitcoin. All nodes will converge on the same shared state if and only if all nodes have the same $is\_valid$ function.

#### Fork

Bacically, fork means more than one blocks are found at the same height of a blockchain.

- Temporary Fork: If multiple blocks are found almost concurrently, the blockchain is temporarily forked.

  > Temporary Forks are eventually resolved. In order for the fork to continue to exist, pairs of blocks need to grow almost simultaneously, because only transactions in the longest path are agreed upon. The probability of branches being extended simultaneously decreases exponentially with the length of the fork, hence there will eventually be a time when one branch becomes longer.

- Soft Fork & Hard Fork

  Soft forks and hard forks differ to temporary forks in that they represent a permanent change in the underlying rules of the protocol. 

  - Soft Fork: Happens if miners create the blocks using a more **permissive** consensus rule. That is to say, the set of valid transactions is expanded. Soft Fork is backward compatible, and participants don't need to upgrade the software immediately to keep playing in the network.
  - Hard Fork: Happens if miners create the blocks using a more **restrictive** consensus rule. That is to say, the set of valid transactions is reduced. Every participant need to upgrade their software.



#### Bitcoin

- Bitcoin Network: a randomly connected overlay network of a few tens of thousands of individually controlled nodes.
- Bitcoin Currency: an integer value that is transferred in Bitcoin transactions. This integer value is measured in Satoshi. 100 million Satoshi are 1 Bitcoin.
- The Bitcoin PoW function is given by: $F_d(c,x)\rightarrow $ SHA256$($SHA256$(c\vert x ))<\frac{2^{224}}d$



#### Ethereum





# 公链层级：Layer 0/1/2

区块链是比特币的底层技术，它一共有六层架构:数据层、网络层、共识层、激励层、合约层和应用层。

### Layer 0

又称数据传输层，利用传统网络，使区块链在 Layer 1 顺利运行。使区块链之间可以进行通信，解决孤链问题。例如 Pokladot 和 Cosmos 跨链生态系统。

### Layer 1

定义了区块链的基本规则，共识机制，数据结构等，是区块链最核心的部分。Layer 1 包含大家熟悉的比特币、以太坊之等：

- 数据层：即“区块+链”的结构。从创世区块起构成的链式结构，包含了哈希值、随机数、认证交易的时间戳、交易信息数据、公钥和私钥等
- 网络层：分布式存储。主要是点对点机制、数据传播机制和数据验证机制。分布式算法以及加密签名等都在网络层中实现，区块链上的各个节点通过这种方式来保持联系，共同维护整个区块链账本，比较熟知的有闪电网络、雷电网络等第二层支付协议。
- 共识层：共识机制，主要有 PoW、PoS、DPoS、PoW+PoS、燃烧证明、重要性证明等十几种
- 激励层：一般指挖矿奖励，通过奖励一部分数字资产从而激励矿工去验证交易信息

### Layer 2

在 Layer 1 之上构建协议和解决方案，不改变区块链底层协议和基础规则，通过状态通道、侧链等方案提高交易处理速度，减轻主链的负担（链下扩容）：

- 合约层：主要指智能合约，及其他脚本代码、侧链应用等。将把代码写到合约里，就可以自定义约束条件，不需要第三方信任背书，到时间立即实时操作。
- 应用层：类似于手机上的各种 APP，即区块链的各种应用场景。
