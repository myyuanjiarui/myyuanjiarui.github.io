---
title: Pytorch分布式训练
date: 2025-06-21 21:46 
description: Pytorch官方教程笔记
categories:
- Python
tags:
---
[Pytorch官方教程](https://docs.pytorch.org/tutorials/beginner/ddp_series_theory.html)

<head>
  <meta name="referrer" content="no-referrer" />
</head>


## Distributed Data Parallel(DDP)

- 背景

  当单个GPU在无法训练某个batch_size=B下的模型时，使用DDP可以实现在N卡上训练batch_size=B/N下的模型，获得与单卡batch_size=B类似的效果。

  即DDP可以让相同的数据获得更多的计算量，从而让模型训练的更快。

- `DistributedSampler`

  DistributedSampler可以保证每个设备上获得的batch数据不重叠。

- 运作

  DDP会在每个设备上复制相同的模型，每个复制模型都会计算梯度并且使用[Ring all-reduce](https://tech.preferred.jp/en/blog/technologies-behind-distributed-deep-learning-allreduce/)算法进行梯度的同步（可以简单理解为取平均）

- `DataParallel`

  `DataParallel`是一个废弃的方法，请使用`Distributed Data Parallel`

## Multi GPU training with DDP

- 术语和API接口

  - `world_size`

    `world_size`代表的是分布式运行中的进程总数

  - `rank`

    `rank`代表分配给各进程的唯一标识符（范围：0~world_size-1）

  - `os.environ["MASTER_ADDR"]`

    设置为运行rank0进程所在机器的IP地址。之所以叫做`MASTER`，是因为这台机器负责所有机器的通信工作。

  - `os.environ["MASTER_PORT"]`

    设置为`MASTER_ADDR`机器的任意一个空闲端口。

  - `init_process_group(backend, rank, world_size)`

    初始化默认的分布式进程组。

- 将单GPU训练脚本改为多GPU训练脚本

  基本的更改内容如下：

  - import相关库

    ```py
    import torch.multiprocessing as mp # PyTorch的多进程库
    from torch.utils.data.distributed import DistributedSampler # 导入分布式数据处理包，用于把我们的数据分发到不同的GPU上
    from torch.nn.parallel import DistributedDataParallel as DDP # DDP wrapper
    from torch.distributed import init_process_group, destroy_process_group # 启动和销毁DDP进程
    import os
    ```

  - 构建进程组

    ```py
    os.environ["MASTER_ADDR"] = "localhost" # 设置主机器的IP地址，单节点多GPU设置为localhost，多节点设置为主机器的IP地址
    os.environ["MASTER_PORT"] = "12355" # 选择一个空闲端口
    torch.cuda.set_device(rank) # 为每个进程设置默认GPU，可以防止GPU的挂起或过度内存使用
    init_process_group(backend="nccl", rank=rank, world_size=world_size) # 初始化分布式进程组
    ```

  - 构建DDP模型

    ```py
    self.model = DDP(model, device_ids=[gpu_id])
    ```

  - 分发输入数据

    ```py
    train_data = torch.utils.data.DataLoader(
    		dataset=train_dataset,
      	batch_size=32,
      	shuffle=False, # 无需shuffle，DistributedSampler会帮我们做好shuffle
      	sampler=DistributedSampler(train_dataset) # 将输入数据在所有分布式进程中进行分块
    ) # 每个进程每批次会接受一个包含32个样本的数据，因此总的有效批次样本数为32 * nprocs（例如在4个GPU时为128）
    ```

    ```py
    for epoch in range(max_epochs):
    	...
      self.train_data.sampler.set_epoch(epoch) # 在每个epoch训练开始前调用set_epoch方法可以保证数据在不同的epoch采用不同的顺序，否则每个周期将使用相同的顺序
      ...
    ```

  - 保存模型检查点

    Git diff 与单GPU训练脚本的对比

    ```git
    - ckp = self.model.state_dict()
    + ckp = self.model.module.state_dict() # 当模型被DDP封装后，会多一个module的key
    ...
    ...
    - if epoch % self.save_frequency == 0:
    + if self.gpu_id == 0 and epoch % self.save_frequency == 0: # 需要额外加一个判断GPU的条件，否则每个进程都会保存相同模型的副本，而这是没有必要的
    ```

  - 运行分布式训练任务

    ```git
    - def main(device, total_epochs, save_every):
    + def main(rank, world_size, total_epochs, save_every): # 包含新的参数rank和world_size，替换掉device
    ...
    - trainer = Trainer(model, train_data, optimizer, device, save_every)
    + trainer = Trainer(model, train_data, optimizer, rank, save_every)
    ...
    + destroy_process_group() # 销毁DDP进程
    
    if __name__ == "__main__":
    	...
    - device = 0
    - main(device, total_epochs, save_every)
    + world_size = torch.cuda.device_count() # 获取可用的GPU总数
    + mp.spawn(main, args=(world_size, total_epochs, save_every,), nprocs=world_size) # args参数无需指定rank，spawn函数会自动指定
    ```

  完整的代码multi_gpus.py

  ```py
  import torch
  import torch.nn.functional as F
  from torch.utils.data import Dataset, DataLoader
  from datautils import MyTrainDataset
  import torch.multiprocessing as mp
  from torch.utils.data.distributed import DistributedSampler
  from torch.nn.parallel import DistributedDataParallel as DDP
  from torch.distributed import init_process_group, destroy_process_group
  import os
  # A8000设置NCCL相关环境变量
  os.environ['NCCL_DEBUG'] = 'INFO'
  os.environ['NCCL_SOCKET_IFNAME'] = 'lo'  # 使用回环接口
  os.environ['NCCL_P2P_DISABLE'] = '1'     # 禁用P2P通信
  os.environ['NCCL_IB_DISABLE'] = '1'      # 禁用InfiniBand
  
  class Trainer:
      def __init__(
          self,
          model: torch.nn.Module,
          train_data: DataLoader,
          optimizer: torch.optim.Optimizer,
          gpu_id: int,
          save_every: int, 
      ) -> None:
          self.gpu_id = gpu_id
          self.model = model.to(gpu_id)
          self.train_data = train_data
          self.optimizer = optimizer
          self.save_every = save_every
          self.model = DDP(model, device_ids=[gpu_id])
  
      def _run_batch(self, source, targets):
          self.optimizer.zero_grad()
          output = self.model(source)
          loss = F.cross_entropy(output, targets)
          loss.backward()
          self.optimizer.step()
  
      def _run_epoch(self, epoch):
          b_sz = len(next(iter(self.train_data))[0])
          print(f"[GPU{self.gpu_id}] Epoch {epoch} | Batchsize: {b_sz} | Steps: {len(self.train_data)}")
          for source, targets in self.train_data:
              source = source.to(self.gpu_id)
              targets = targets.to(self.gpu_id)
              self._run_batch(source, targets)
  
      def _save_checkpoint(self, epoch):
          ckp = self.model.module.state_dict()
          PATH = "checkpoint.pt"
          torch.save(ckp, PATH)
          print(f"Epoch {epoch} | Training checkpoint saved at {PATH}")
  
      def train(self, max_epochs: int):
          for epoch in range(max_epochs):
              self.train_data.sampler.set_epoch(epoch)
              self._run_epoch(epoch)
              if self.gpu_id == 0 and epoch % self.save_every == 0:
                  self._save_checkpoint(epoch)
  
  
  def load_train_objs():
      train_set = MyTrainDataset(2048)  # load your dataset
      model = torch.nn.Linear(20, 1)  # load your model
      optimizer = torch.optim.SGD(model.parameters(), lr=1e-3)
      return train_set, model, optimizer
  
  
  def prepare_dataloader(dataset: Dataset, batch_size: int):
      return DataLoader(
          dataset,
          batch_size=batch_size,
          shuffle=False,
          sampler=DistributedSampler(dataset)
      )
  
  
  def main(rank, world_size, total_epochs, save_every, batch_size):
      init_process_group(backend="nccl", rank=rank, world_size=world_size)
      dataset, model, optimizer = load_train_objs()
      train_data = prepare_dataloader(dataset, batch_size)
      trainer = Trainer(model, train_data, optimizer, rank, save_every)
      trainer.train(total_epochs)
      destroy_process_group()
  
  
  if __name__ == "__main__":
      import argparse
      parser = argparse.ArgumentParser(description='simple distributed training job')
      parser.add_argument('total_epochs', type=int, help='Total epochs to train the model')
      parser.add_argument('save_every', type=int, help='How often to save a snapshot')
      parser.add_argument('--batch_size', default=32, type=int, help='Input batch size on each device (default: 32)')
      args = parser.parse_args()
      os.environ["MASTER_ADDR"] = "localhost"
      os.environ["MASTER_PORT"] = "12355"
      world_size = torch.cuda.device_count()
      mp.spawn(main, args=(world_size, args.total_epochs, args.save_every, args.batch_size), nprocs=world_size)
  ```

## 使用`torchrun`的容错分布式训练

- 背景

  分布式训练是非常容易训练失败的情景，例如单个GPU的失效。我们可能还希望训练过程具有弹性，例如GPU资源可以在任务进行过程中动态的加入和离开。

- `torchrun`的优势

  - `torchrun`提供了容错和弹性的分布式训练。
  - 无需显式的设置环境变量如"MASTER_ADDR"和"MASTER_PORT"
    - `self.gpu_id = int(os.environ["LOCAL_RANK"])`，使用环境变量获得当前进程的rank值
  - 无需给main函数显示的传递`rank`和`world_size`变量。
  - 无需调用`mp.spawn`，我们只需要一个通用的main入口（与非分布式的入口相同）

- 断点续连

  我们只需要在代码中定义一个快照保存函数，快照保存模型的state_dict()和最后训练的epoch，下次重新加载即可实现断点续连。

  > 其实这个实现过程都是自定义的，与torchrun本身其实无关

  伪代码如下：

  ```py
  def main():
    load_snapshot(snapshot_path)
    initialize()
    train()
    
  def train():
    for batch in iter(dataset):
      train_step(batch)
      
      if should_checkpoint:
        save_snapshot(snapshot_path)
  ```

  完整代码：

  ```python
  import torch
  import torch.nn.functional as F
  from torch.utils.data import Dataset, DataLoader
  from datautils import MyTrainDataset
  import torch.multiprocessing as mp
  from torch.utils.data.distributed import DistributedSampler
  from torch.nn.parallel import DistributedDataParallel as DDP
  from torch.distributed import init_process_group, destroy_process_group
  import os
  # A8000设置NCCL相关环境变量
  os.environ['NCCL_DEBUG'] = 'INFO'
  os.environ['NCCL_SOCKET_IFNAME'] = 'lo'  # 使用回环接口
  os.environ['NCCL_P2P_DISABLE'] = '1'     # 禁用P2P通信
  os.environ['NCCL_IB_DISABLE'] = '1'      # 禁用InfiniBand
  
  class Trainer:
      def __init__(
          self,
          model: torch.nn.Module,
          train_data: DataLoader,
          optimizer: torch.optim.Optimizer,
          snapshot_path: str,
          save_every: int, 
      ) -> None:
          self.gpu_id = int(os.environ["LOCAL_RANK"]) # 使用环境变量获得当前进程的rank值
          self.model = model.to(self.gpu_id)
          self.train_data = train_data
          self.optimizer = optimizer
          self.save_every = save_every
          self.epochs_run = 0
          if os.path.exists(snapshot_path):
              self._load_snapshot(snapshot_path)
          self.model = DDP(model, device_ids=[self.gpu_id])
  
      def _run_batch(self, source, targets):
          self.optimizer.zero_grad()
          output = self.model(source)
          loss = F.cross_entropy(output, targets)
          loss.backward()
          self.optimizer.step()
  
      def _run_epoch(self, epoch):
          b_sz = len(next(iter(self.train_data))[0])
          print(f"[GPU{self.gpu_id}] Epoch {epoch} | Batchsize: {b_sz} | Steps: {len(self.train_data)}")
          for source, targets in self.train_data:
              source = source.to(self.gpu_id)
              targets = targets.to(self.gpu_id)
              self._run_batch(source, targets)
  
      def _load_snapshot(self, snapshot_path):
          snapshot = torch.load(snapshot_path)
          self.epochs_run = snapshot["EPOCHS_RUN"]
          self.model.load_state_dict(snapshot["MODEL_STATE"])
          print(f"Resuming training from snapshot at Epoch {self.epochs_run}")
  
      def _save_snapshot(self, epoch):
          snapshot = {}
          ckp = self.model.module.state_dict()
          PATH = "snapshot.pt"
          snapshot = {
              "MODEL_STATE": ckp,
              "EPOCHS_RUN": epoch
          }
          torch.save(snapshot, PATH)
          print(f"Epoch {epoch} | Training snapshot saved at {PATH}")
  
      def train(self, max_epochs: int):
          for epoch in range(self.epochs_run, max_epochs):
              self.train_data.sampler.set_epoch(epoch)
              self._run_epoch(epoch)
              if self.gpu_id == 0 and epoch % self.save_every == 0:
                  self._save_snapshot(epoch)
  
  
  def load_train_objs():
      train_set = MyTrainDataset(16)  # load your dataset
      model = torch.nn.Linear(8, 1)  # load your model
      optimizer = torch.optim.SGD(model.parameters(), lr=1e-3)
      return train_set, model, optimizer
  
  
  def prepare_dataloader(dataset: Dataset, batch_size: int):
      return DataLoader(
          dataset,
          batch_size=batch_size,
          shuffle=False,
          sampler=DistributedSampler(dataset)
      )
  
  
  def main(total_epochs, save_every, batch_size, snapshot_path):
      init_process_group(backend="nccl")
      dataset, model, optimizer = load_train_objs()
      train_data = prepare_dataloader(dataset, batch_size)
      trainer = Trainer(model, train_data, optimizer, snapshot_path, save_every)
      trainer.train(total_epochs)
      destroy_process_group()
  
  
  if __name__ == "__main__":
      import argparse
      parser = argparse.ArgumentParser(description='simple distributed training job')
      parser.add_argument('total_epochs', type=int, help='Total epochs to train the model')
      parser.add_argument('save_every', type=int, help='How often to save a snapshot')
      parser.add_argument('--batch_size', default=4, type=int, help='Input batch size on each device (default: 32)')
      args = parser.parse_args()
      snapshot_path = "snapshot.pt"
      main(args.total_epochs, args.save_every, args.batch_size, snapshot_path)
  ```

  运行代码：

  ```sh
  torchrun --standalone --nproc_per_node=2 pytorch_examples/3.torchrun.py 50 10
  ```


## Multinode Training

- 训练效率

  多结点GPU的训练主要瓶颈在GPU之间的TCP通信，所以在一个结点上训练4个GPU比同时在四个结点训练1个GPU更快速。

- `Global Rank`

  在多结点GPU训练中存在`Global Rank`的概念，`Global Rank`是标识不同结点的唯一identifier，需要在运行命令中指定。获得方法如下：

  ```py
  global_rank = int(os.environ["RANK"])
  ```

  `local rank`的概念与单结点多GPU训练相同，用于标识结点中不同GPU，Pytorch会自动分配。

- `Fully-Sharded Data Parallel`

  即FSDP，分片数据并行，每个模型在GPU上不是复制的，而是分片到所有GPU上。

- 示例代码：

  ```py
  import torch
  import torch.nn.functional as F
  from torch.utils.data import Dataset, DataLoader
  from datautils import MyTrainDataset
  import torch.multiprocessing as mp
  from torch.utils.data.distributed import DistributedSampler
  from torch.nn.parallel import DistributedDataParallel as DDP
  from torch.distributed import init_process_group, destroy_process_group
  import os
  # A8000设置NCCL相关环境变量
  os.environ['NCCL_DEBUG'] = 'INFO'
  # os.environ['NCCL_SOCKET_IFNAME'] = 'lo'  # 使用回环接口
  os.environ['NCCL_P2P_DISABLE'] = '1'     # 禁用P2P通信
  os.environ['NCCL_IB_DISABLE'] = '1'      # 禁用InfiniBand
  
  class Trainer:
      def __init__(
          self,
          model: torch.nn.Module,
          train_data: DataLoader,
          optimizer: torch.optim.Optimizer,
          snapshot_path: str,
          save_every: int, 
      ) -> None:
          self.gpu_id = int(os.environ["LOCAL_RANK"]) # 使用环境变量获得当前进程的本地rank值
          self.global_rank = int(os.environ["RANK"]) # 使用环境变力获得当前进程在全局rank值
          self.model = model.to(self.gpu_id)
          self.train_data = train_data
          self.optimizer = optimizer
          self.save_every = save_every
          self.epochs_run = 0
          if os.path.exists(snapshot_path):
              self._load_snapshot(snapshot_path)
          self.model = DDP(model, device_ids=[self.gpu_id])
  
      def _run_batch(self, source, targets):
          self.optimizer.zero_grad()
          output = self.model(source)
          loss = F.cross_entropy(output, targets)
          loss.backward()
          self.optimizer.step()
  
      def _run_epoch(self, epoch):
          b_sz = len(next(iter(self.train_data))[0])
          print(f"[GPU{self.global_rank}] Epoch {epoch} | Batchsize: {b_sz} | Steps: {len(self.train_data)}")
          for source, targets in self.train_data:
              source = source.to(self.gpu_id)
              targets = targets.to(self.gpu_id)
              self._run_batch(source, targets)
  
      def _load_snapshot(self, snapshot_path):
          snapshot = torch.load(snapshot_path)
          self.epochs_run = snapshot["EPOCHS_RUN"]
          self.model.load_state_dict(snapshot["MODEL_STATE"])
          print(f"Resuming training from snapshot at Epoch {self.epochs_run}")
  
      def _save_snapshot(self, epoch):
          snapshot = {}
          ckp = self.model.module.state_dict()
          PATH = "snapshot.pt"
          snapshot = {
              "MODEL_STATE": ckp,
              "EPOCHS_RUN": epoch
          }
          torch.save(snapshot, PATH)
          print(f"Epoch {epoch} | Training snapshot saved at {PATH}")
  
      def train(self, max_epochs: int):
          for epoch in range(self.epochs_run, max_epochs):
              self.train_data.sampler.set_epoch(epoch)
              self._run_epoch(epoch)
              if self.gpu_id == 0 and epoch % self.save_every == 0:
                  self._save_snapshot(epoch)
  
  
  def load_train_objs():
      train_set = MyTrainDataset(16)  # load your dataset
      model = torch.nn.Linear(8, 1)  # load your model
      optimizer = torch.optim.SGD(model.parameters(), lr=1e-3)
      return train_set, model, optimizer
  
  
  def prepare_dataloader(dataset: Dataset, batch_size: int):
      return DataLoader(
          dataset,
          batch_size=batch_size,
          shuffle=False,
          sampler=DistributedSampler(dataset)
      )
  
  
  def main(total_epochs, save_every, batch_size, snapshot_path):
      init_process_group(backend="nccl")
      dataset, model, optimizer = load_train_objs()
      train_data = prepare_dataloader(dataset, batch_size)
      trainer = Trainer(model, train_data, optimizer, snapshot_path, save_every)
      trainer.train(total_epochs)
      destroy_process_group()
  
  
  if __name__ == "__main__":
      import argparse
      parser = argparse.ArgumentParser(description='simple distributed training job')
      parser.add_argument('total_epochs', type=int, help='Total epochs to train the model')
      parser.add_argument('save_every', type=int, help='How often to save a snapshot')
      parser.add_argument('--batch_size', default=4, type=int, help='Input batch size on each device (default: 32)')
      args = parser.parse_args()
      snapshot_path = "snapshot.pt"
      main(args.total_epochs, args.save_every, args.batch_size, snapshot_path)
  ```

  运行命令（假设共有两个结点）：

  - 在结点一：

    ```shell
    torchrun --nproc_per_node=2 --nnode=2 --node_rank=0 multinode.py 50 10
    ```

    

  - 在结点二：

    ```shell
    torchrun --nproc_per_node=4 --nnode=2 --node_rank=1 multinode.py 50 10
    ```

  命令解释：

  - `--nproc_per_node`：代表当前结点参与训练的GPU总数
  - `--nnode`：代表参与训练的结点总数
  - `--node_rank`：代表当前结点的global rank值
