# 损失函数



# Optimizer 优化器

Pytorch中优化器的功能为：**管理**和**更新**模型中可学习参数的值，使得模型输出更接近真实标签。

在Pytorch中，自动求导模块可以根据损失函数对模型的参数进行求梯度运算。优化器会获取得到的梯度，然后利用一些策略去更新模型的参数，最终使得损失函数的值下降。

### torch.optim

`torch.optim` 是一个实现了各种优化算法的库，支持大部分常用的方法如 SGD，Adam，RMSprop 等。

```python
# 构造一个 optimizer 对象，可保存当前的参数状态
optimizer = torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
optimizer = torch.optim.Adam([var1, var2], lr=0.0001)

optimizer.zero_grad()  # 清空梯度信息
loss = loss_function(output, target)  # 基于前向传播的结果(output)和目标(target)计算误差
loss.backward()  # 计算梯度信息
optimizer.step()  # 更新参数
```

另外，如果要使用GPU进行训练，需要先使用`.cuda()`方法将模型和优化器移动到GPU上。此时模型中的参数将会变成与CPU上不同的对象。



# Dataloader

Dataloader是PyTorch提供的一个数据加载器，用于对数据进行批量加载和处理，主要功能有：

1. 批量加载：可将数据分成小批次进行加载，使得每次迭代都能处理多个数据样本，以提高计算效率，帮助模型更好地泛化。
2. 数据洗牌：支持对数据进行随机洗牌操作，这有助于模型在训练过程中看到不同的数据排列，避免过拟合。
3. 并发预取：在数据加载过程中进行并发预取，即在模型处理当前批次数据的同时，预先加载下一批次数据，减少数据加载的时间，提高训练效率。

```python
from torch.utils.data import DataLoader
# 创建一个自定义数据集类，继承 Dataset，实现 len 和 getitem 方法
class MyDataset(torch.utils.data.Dataset):
    def __init__(self, data, target):
        self.data = data
        self.target = target
    def __len__(self):
        return len(self.data)
    def __getitem__(self, idx):
        return self.data[idx], self.target[idx]
# 创建 dataloader 实例   
dataset = MyDataset(data, target)
dataloader = DataLoader(dataset, batch_size=32, shuffle=True)
# 训练
for batch_idx, (data, target) in enumerate(dataloader):
    # 在这里进行模型的训练操作
    pass
```

