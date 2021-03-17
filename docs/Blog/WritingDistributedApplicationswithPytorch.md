# Writing Distributed Applications with Pytorch

来自[官方教程](https://pytorch.org/tutorials/intermediate/dist_tuto.html)

## 设置

- 分布式包对应`torch.distributed`；
- `torch.distributed`包和`torch.multiprocessing`包的不同之处在于，前者能够使用不同的通讯协议，以及异构的机器中；
- 集群需要使用合适的工具来访问，比如：[pdsh](https://linux.die.net/man/1/pdsh)或者[clustershell](https://cea-hpc.github.io/clustershell/)。

官方的多进程模板：

```python
"""run.py:"""
#!/usr/bin/env python
import os
import torch
import torch.distributed as dist
from torch.multiprocessing import Process

def run(rank, size):
    """ Distributed function to be implemented later. """
    pass

def init_process(rank, size, fn, backend='gloo'):
    """ Initialize the distributed environment. """
    os.environ['MASTER_ADDR'] = '127.0.0.1'
    os.environ['MASTER_PORT'] = '29500'
    dist.init_process_group(backend, rank=rank, world_size=size)
    fn(rank, size)


if __name__ == "__main__":
    size = 2
    processes = []
    for rank in range(size):
        p = Process(target=init_process, args=(rank, size, run))
        p.start()
        processes.append(p)

    for p in processes:
        p.join()
```

## 点对点通讯

- `send`/`recv`是阻断式通讯；
- `isend`/`irecv`是非阻断式通讯；
- `wait()`等待通讯结束。
  
```
"""Non-blocking point-to-point communication."""

def run(rank, size):
    tensor = torch.zeros(1)
    req = None
    if rank == 0:
        tensor += 1
        # Send the tensor to process 1
        req = dist.isend(tensor=tensor, dst=1)
        print('Rank 0 started sending')
    else:
        # Receive tensor from process 0
        req = dist.irecv(tensor=tensor, src=0)
        print('Rank 1 started receiving')
    req.wait()
    print('Rank ', rank, ' has data ', tensor[0])
```

## 广播

- `scatter`：一到多，多个数据分发；
- `gather`：多到一，多个数据收集到一起；
- `reduce`: 多到一，同时计算reduce函数；
- `allreduce`: 多到多，同时计算reduce函数；
- `broadcase`: 一到多，一个数据复制分发；
- `all-gather`：多到多，多个数据在各个机器被收集。

Reduce函数有
- `dist.reduce_op.SUM`;
- `dist.reduce_op.PRODUCT`；
- `dist.reduce_op.MAX`；
- `dist.reduce_op.MIN`.

- `dist.broadcast(tensor, src, group)`: Copies tensor from src to all other processes.
- `dist.reduce(tensor, dst, op, group)`: Applies op to all tensor and stores the result in dst.
- `dist.all_reduce(tensor, op, group)`: Same as reduce, but the result is stored in all processes.
- `dist.scatter(tensor, src, scatter_list, group)`: Copies the ith tensor scatter_list[i] to the ith process.
- `dist.gather(tensor, dst, gather_list, group)`: Copies tensor from all processes in dst.
- `dist.all_gather(tensor_list, tensor, group)`: Copies tensor from all processes to tensor_list, on all processes.
- `dist.barrier(group)`: block all processes in group until each one has entered this function.

## 分布式训练

官方针对一个监督学习问题来作为示例。第一步需要实现训练集的分解。

```python
""" Dataset partitioning helper """
class Partition(object):

    def __init__(self, data, index):
        self.data = data
        self.index = index

    def __len__(self):
        return len(self.index)

    def __getitem__(self, index):
        data_idx = self.index[index]
        return self.data[data_idx]


class DataPartitioner(object):

    def __init__(self, data, sizes=[0.7, 0.2, 0.1], seed=1234):
        self.data = data
        self.partitions = []
        rng = Random()
        rng.seed(seed)
        data_len = len(data)
        indexes = [x for x in range(0, data_len)]
        rng.shuffle(indexes)

        for frac in sizes:
            part_len = int(frac * data_len)
            self.partitions.append(indexes[0:part_len])
            indexes = indexes[part_len:]

    def use(self, partition):
        return Partition(self.data, self.partitions[partition])
```

接着用在MNIST数据集上

```python
""" Partitioning MNIST """
def partition_dataset():
    dataset = datasets.MNIST('./data', train=True, download=True,
                             transform=transforms.Compose([
                                 transforms.ToTensor(),
                                 transforms.Normalize((0.1307,), (0.3081,))
                             ]))
    size = dist.get_world_size()
    bsz = 128 / float(size)
    partition_sizes = [1.0 / size for _ in range(size)]
    partition = DataPartitioner(dataset, partition_sizes)
    partition = partition.use(dist.get_rank())
    train_set = torch.utils.data.DataLoader(partition,
                                         batch_size=bsz,
                                         shuffle=True)
    return train_set, bsz
```

然后实现`run`部分的函数。

```python
""" Distributed Synchronous SGD Example """
def run(rank, size):
    torch.manual_seed(1234)
    train_set, bsz = partition_dataset()
    model = Net()
    optimizer = optim.SGD(model.parameters(),
                          lr=0.01, momentum=0.5)

    num_batches = ceil(len(train_set.dataset) / float(bsz))
    for epoch in range(10):
        epoch_loss = 0.0
        for data, target in train_set:
            optimizer.zero_grad()
            output = model(data)
            loss = F.nll_loss(output, target)
            epoch_loss += loss.item()
            loss.backward()
            average_gradients(model)
            optimizer.step()
        print('Rank ', dist.get_rank(), ', epoch ',
              epoch, ': ', epoch_loss / num_batches)
```

其中，平均梯度的函数为

```python
""" Gradient averaging. """
def average_gradients(model):
    size = float(dist.get_world_size())
    for param in model.parameters():
        dist.all_reduce(param.grad.data, op=dist.reduce_op.SUM)
        param.grad.data /= size
```

## 初始化

宏变量：

- `MASTER_PORT`: A free port on the machine that will host the process with rank 0.
- `MASTER_ADDR`: IP address of the machine that will host the process with rank 0.
- `WORLD_SIZE`: The total number of processes, so that the master knows how many workers to wait for.
- `RANK`: Rank of each process, so they will know whether it is the master of a worker.

共享文件系统

```python
dist.init_process_group(
    init_method='file:///mnt/nfs/sharedfile',
    rank=args.rank,
    world_size=4)
```

使用TCP

```python
dist.init_process_group(
    init_method='tcp://10.1.1.20:23456',
    rank=args.rank,
    world_size=4)
```