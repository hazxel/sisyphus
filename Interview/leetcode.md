# Hashing

### `hashcode` &`equals`

Use `hashcode` to map the elements, but when collision, use `equals ` to resolve collision.

### Collision

##### Separate Chaining

Use a linked list or RB-tree to store all the collision of a bucket. Array stores the pointers to the data structures. In JDK 1.8, java use RBT instead of linked list if length bigger than 8.

##### Open addressing

Everything is stored in the array. When collision occurs, find a new storage spot in the array. Utilize a flag to indicate whether the spot is in use.

- linear probing: move to the next spot (+1)
- quadratic probing: the $n^{th}$ collision ends up $n^2$ away from the original index

##### Cuckoo hashing / Double hashing

The basic idea of cucbkoo hashing is to resolve collisions by using two hash functions instead of only one. This provides two possible locations in the hash table for each key. In one of the commonly used variants of the algorithm, the hash table is split into two smaller tables of equal size, and each hash function provides an index into one of the tables. It is also possible for both hash functions to provide indexes into a single table.

### Rehashing

Rehash usually happens before number of items goes over half the table size. Need to go through old array and inser teach item to the new one. Array size usually prime, double size by find the next prime above double.



# Heap

### binary heap 二叉堆

是完全二叉树或者是近似完全二叉树，父节点的键值总是大于等于或小于等于任何一个子节点的键值，可用数组实现，插入时放到队尾后上浮（logn），pop时将队尾放到顶端后下沉（logn）

### binominal tree 二项树

递归定义：度为0的二项树只有一个根节点，度为k的二项树的根节点有k个子节点且每个子节点分别都为度为0,1,...,k-1的二项树。

度数为k的二项树可以从两颗度数为k-1的二项树合并得到：把一颗度数为k-1的二项树作为另一颗原度数为k-1的二项树的最左子树，这一性质是二项堆用于堆合并的基础。（度为k的二项树有$2^k$个节点）

### binomial heap 二项堆

二项堆是数个度互不相等的二项树的集合体，由于度为k的二项树有$2^k$个节点，任意元素个数都有其唯一确定的二项堆结构，即用二进制表示元素个数，第k位为0或1就代表是否存在度为k的二项树。二项堆的merge类似于二进制加法，插入和pop都可以转化为merge。decrease key等同于二叉堆的上浮（都为logn）

### fibonacci heap 斐波那契堆

lazyness，只在pop min的时候做子树的合并，其余时候都直接放到链上



# Sorting

### Binary Search

```c++
int binarySearch(int arr[], int low, int high, int x) {
    while (low <= high) {
        int mid = (low + high) / 2;
        if (arr[mid] == x) return mid；
        if (arr[mid] < x) low = mid + 1; else high = mid - 1;
    }
    return -1;
}
```

### Quick Sort

todo

### Hoare's selection algorithm (Top-K / K-th) 

binary search for K-th element, time complexity O(n)





# String

### Substring vs Subsequence

A substring is a contiguous part of a string, i.e., a string inside another string.

A subsequence is a sequence that can be derived from another sequence by removing zero or more elements, without changing the order of the remaining elements.



# Tree Traversal

note that solution will be pretty easy if recursive function is allowed, so we talk about loop solution here.

### Pre-order DFS

The most common DFS, easy to implement with only one stack

```c++
void preorder(Node* root) {
    vector<Node*> s{root};
    while (s.size() != 0) {
        Node* cur = s.back();
        s.pop_back();
        cout << cur->val << endl;
        if (cur->right != nullptr) {
            s.push_back(cur->right);
        }
        if (cur->left != nullptr) {
            s.push_back(cur->left);
        }
    }
}
```

### Inorder DFS

Beside a stack, need an extra flag to tell the node is first time visit or not. Little bit harder than pre-order.

```c++
void inorder(Node* root) {
    vector<Node*> s{root};
    bool isFirstVisit = true;
    while (s.size() != 0) {
        if (isFirstVisit && s.back()->left != nullptr) {
            s.push_back(s.back()->left);
        } else {
            cout << s.back()->val << endl;
            if (s.back()->right != nullptr) {
                s.back() = s.back()->right;
                isFirstVisit = true;
            } else {
                s.pop_back();
                isFirstVisit = false;
            }
        }
    }
}
```

### Postorder DFS

The following solution uses 2 stacks.

```c++
enum class Direction { First, Left, Right };

void postorder(Node* root) {
    vector<Node*> s{root};
    vector<Direction> dir{Direction::First};
    while (!s.empty()) {
        if (dir.back() == Direction::First && s.back()->left != nullptr) {
            s.push_back(s.back()->left);
            dir.back() = Direction::Left;
            dir.push_back(Direction::First);
        } else if (dir.back() != Direction::Right && s.back()->right != nullptr) {
            s.push_back(s.back()->right);
            dir.back() = Direction::Right;
            dir.push_back(Direction::First);
        } else {
            cout << s.back()->val << endl;
            s.pop_back();
            dir.pop_back();
        }
    }
}
```



### BFS

(Todo) use queue, shouldn't be hard

### Serialization

DFS/BFS are both ok, as long as write down NULL child when iterating throught the tree.





# Graph

### Dijkstra

处理非负权值的的单源最短路径算法，复杂度 O((V+E)logV)，选择最近顶点时要对大小为V的队列进行出队（VlogV），根据边更新堆中距离时需要 decrease-key (ElogV)

维护每个其他节点距源节点的距离，每次从中找出距离最近的节点，根据其值更新所有距离后移除，数据结构可采用普通二叉堆，或斐波那契堆

### Bellman-ford

处理单源最短路径算法，复杂度 O(VE)，

进行 V-1 次松弛操作，每次松弛遍历所有边，根据起点距离更新终点距离，若V-1次后还能更新，则图存在负回路

> 引理：若不存在负回路，则到某个点的最短路径不会超过V个点（V-1条边），即不会经过重复的点



# Bitwise

### Shifting

- arithmetic right shifting: fill sign bit in the left
- logical right shifting: do not consider sign bit, fill zero
- Arithmetic left shifting: sign bit unchanged, fill zero
- logical left shifting: fill zero




# Mathmatic Problems

### Prime Number

- Naive Approach: Interate through $2$ to $n$ to check if $n$ is a prime

- Improved Approach: Iterate through 2 to $|\sqrt n| $ to check

- Sieve of Eratosthenes: to find prime number from 1 to n

  - Complexity: $O(n \log \log n)$
  - Algorithm:
    1. Create a list of consecutive integers from 2 to n
    2. Initially, let p equal 2, the smallest prime number.
    3. Enumerate the multiples of p from 2p, 3p to n, and mark them (as not prime) in the list
    4. Find the smallest number in the list greater than p and is not marked. If there was no such number, stop. Otherwise, let p equal to this new number and repeat from step 3.
    5. When the algorithm terminates, the numbers remaining not marked in the list are all the primes below n

- Sieve of Euler:  to find prime number from 1 to n

  - Complexity: $O(n)$

  - Algorithm: 

    For every number, only visit it once if it is not prime. Composite numbers are sieved when encounter it's smallest prime component.

    ```c++
    boolean isprime[];
    int prime[];
    void getprimeoula()// 欧拉筛
    {
            prime = new int[100001];// 记录第几个prime
            int index = 0;
            isprime = new boolean[1000001];
            for (int i = 2; i < 1000001; i++) {
                if (!isprime[i]) {
                    prime[index++] = i;
                }
                for (int j = 0; j < index && i * prime[j] <= 100000; j++){//已知素数范围内枚举
                    isprime[i * prime[j]] = true;// 标记乘积
                    if (i % prime[j] == 0)
                        break;
                }
            }
    }
    ```

### Joseph's Ring

$F(n+1) = (F(n) + m) \% (n+1)  (n > 1)$

