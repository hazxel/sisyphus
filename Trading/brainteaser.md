### 约瑟夫环问题

题目：一共 N 个人围成环，从 1 开始报数到 M，每次报到 M 的出局。

递推分析：

- 首先出局的人是 M 号，则情形可转化为 N-1 人，并从第 M+1 人开始的情况，即偏移了 M 位。 
- 得到递推公式$Ans_{N，M} = (Ans_{N-1,M} - M) \, mod\, N $​ 
- ？？？没想明白，以后有机会接着想

结论：用 M = 2 时， 用二进制表示 $5 = 101$， 答案为第一位移到最后一位，即$011 =3$

### 蒙提霍尔问题（三门问题，山羊问题）

这是一个源自博弈论的数学游戏问题，参赛者会看见三扇门，其中一扇门的里面有一辆汽车，选中里面是汽车的那扇门，就可以赢得该辆汽车，另外两扇门里面则都是一隻山羊。当参赛者选定了一扇门，主持人会开启另一扇是山羊的门；并问：“要不要换一扇门？”换门的话，赢得汽车的机率是$\frac23$，不换的话赢的概率是$\frac13$。这问题亦被叫做蒙提霍尔悖论：因为该问题的答案虽在逻辑上并无矛盾，但十分违反直觉。

### 阶乘末尾多少个零

只要数5的因子就够了因为2够用，$Ans = \sum_{i=1}^{\infty} \frac n {5^i}$​.

### 1000km沙漠500L油箱的车

- 问最少要多少油，所以本着不浪费的原则：
  - 出发时一定满油
  - 到达时一定没油
  - 运油时次数尽量少
- 所以倒数第一个存油点一定是500L油，距离终点 500KM
- 倒数第二个存油点需要如何保证运500L到下一个？
  - 多少油：首先最少需要运两次，然后又必须满油出发，所以需要1000L油存放
  - 多远：运两次正好没油，所以运油路途消耗500，往返一次+单程一次，距离为 $\frac{500}{3}$
- 拓展到倒数第 N 个存油点：存 500N 油，距离上一个 $\frac{500}{2N-1}$​
- 所以算出要七个存油点，就可覆盖到1000: $\frac{500}{1} + ... + \frac{500}{13} > 1000$​
- 最终耗油：$8\times500 - (\frac{500}{1} + ... + \frac{500}{13} - 1000)=3836.5$​

### 打手电筒过桥

手电筒还有17分钟，四个人过桥分别要1，2，5，10 分钟，桥最多两人同时过，怎么安排？

为节省时间必须让最慢两人一起过，并且他们只过不回：12过1回510过2回12过=17

### 木棍随机三段构成三角形概率

条件1: 两个断点在中点两侧，条件2：两个断点距离小于一半，这两个概率都是 0.5 合计 0.25

或用微积分：$2\int_0^{\frac12} x \,dx = \frac14$，这里 x 代表第一个断点离最近末端距离，也代表此时第二个断点随机落下后和第一个断点距离小于二分之一的概率
