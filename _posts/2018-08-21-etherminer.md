---
layout: post
title: "Coin Mining"
categories: HPC
tags: Bitcoin
--- 

In this post, I mainly make some notes about my test for ethminer.




#### **以太坊挖矿**

1. 基本要求:
ETH 挖矿主要是使用显卡来挖矿。因此你需要一台拥有以下设备的PC（矿机):
显卡、主板、电源、CPU、内存（推荐4G）、硬盘（推荐 60G 的SSD）、PCI-E转接线等。
其中显卡决定挖矿的速度，主板、电源在很大程度上决定了矿机运行的稳定程度。


2. 钱包地址:
挖矿前要准备好自己的钱包地址，可以通过本地钱包软件、交易平台获取钱包地址, 官方图形钱包[地址]( https://github.com/ethereum/mist/releases).交易网站注册帐号，国内推荐云币网https://yunbi.com/
国外推荐https://poloniex.com/网站，按照提示注册生产充值地址

3. 挖矿程序:
开源地址: [ethminer](https://github.com/ethereum-mining/ethminer)

```
Ethminer - GPU Ethereum miner
Usage: ./ethminer [OPTIONS] [pool…]

-L,--dag-load-mode UINT=0   
Set the DAG load mode. 0=parallel, 1=sequential, 2=single

-U,--cuda                   
When mining use the GPU via CUDA

-G,--opencl                 
When mining use the GPU via OpenCL

-P,--pool TEXT ...          
Specify one or more pool URLs. 

-X,--cuda-opencl            
When mining with mixed AMD(OpenCL) and CUDA GPUs

-M,--benchmark UINT=0       
Benchmark mining and exit; Specify block number to benchmark against specific DAG

-Z,--simulation UINT=0      
Mining test. Used to validate kernel optimizations. Specify block number

--tstop UINT=0              
Stop mining on a GPU if temperature exceeds value.

--tstart UINT=40            
Restart mining on a GPU if the temperature drops below, valid: 30..100

CUDA Options:
  --cuda-grid-size UINT=8192  
  Set the grid size

  --cuda-block-size UINT=128  
  Set the block size

  --cuda-devices UINT ...     
  Select list of devices to mine on (default: use all available)

  --cuda-parallel-hash UINT in {1,2,4,8}=4
  Set the number of hashes per kernel

  --cuda-schedule TEXT in {auto,spin,sync,yield}=sync
  Set the scheduler mode

  --cuda-streams UINT=2       
  Set the number of streams
  ```

#### **stratum 协议**

stratum 协议是目前最常用的矿机和矿池之间的TCP通讯协议.
矿工网络分成矿机、矿池、钱包等几个主要部分，有时矿池软件与钱包安装在一起，可合称为矿池。
stratum是JSON为数据格式:
(1). 任务订阅:
矿机启动，首先以mining.subscribe方法向矿池连接，用来订阅工作；

(2). 矿机登录:
矿机以mining.authorize方法，用某个帐号和密码登录到矿池，密码可空，矿池返回true登录成功。该方法必须是在初始化连接之后马上进行，否则矿机得不到矿池任务；

(3). 任务分配:
该命令由矿池定期发给矿机，当矿机以mining.subscribe方法登记后，矿池应该马上以mining.notify返回该任务；

(4). 结果提交:
矿机找到合法share时，就以”mining.submit“方法向矿池提交任务。矿池返回true即提交成功，如果失败则error中有具体原因。