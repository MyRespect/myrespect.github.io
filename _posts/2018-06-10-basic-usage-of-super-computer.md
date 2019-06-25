---
layout: post
title:  "Basic Usage of Super Computer"
categories: HPC
tags: Edison Tianhe
--- 

* content
{:toc}

I get an internship in one company, where I obtain the chance to use **Supercomputer** to validate the developed program. Here are some notes to better use Edison in America and Tianhe in China.





### **Edison**
The official website for Edison Supercomputer is [NERSC](http://www.nersc.gov/). Use Edison is not so complicated, just like a remote server. Use ssh to connect to Edison, scp/sftp for external data transfer. Of course, there are other ways to connect Edison like Multi-Factor Authentication(MFA), X windows. NERSC recommends transferring data to and from NERSC using globus online, scp/sftp for smaller files(<1 GB), bbcp or gridftp for large files.

##### **running job**
Interactive Job: 
```
salloc -N 2 -q debug -L SCRATCH -t 00:30:00
# The -N flag specifices the number of nodes, -q specifices the name of the QOS, specify license need for the file systems your job needs.

srun -n 48 -c 4 ./program
# The -n flag specifices the total number of MPI tasks, -c specifices the number of openmp threads, -N specifices the number of MPI tasks per Edison node.
```
Batch Job:
```
#!/bin/bash -l 
#SBATCH -q debug
#SBATCH -N 2
#SBATCH -t 00:10:00
#SBATCH -L SCRATCH 
srun -n 48 ./my_executable
```
Error message: OOM(out of memory) killer terminated this process, it means that code has exhausted the memory available on the node. Solution: use more nodes and fewer cores per node.

##### **module command**
**module list**: list all the modulefiles that currently loaded
**module avail**: list all the modulefiles that are available to be loaded
**module load**: add one or more modulefiles to your current environment
**module display**: to see exactly what a given modulefile will do to your environment
**module show**: show how your environment gets modified and also some other information
**module snapshot -f filename**: captures the currently loaded modules and saves the list into a file for later restore.

##### **others**
Shifter is a open-source software stack that enables users to run custom environment on HPC system, it is compatible with the popular Docker container format so that users can easily run Docker container on NERSC systems.

NERSC system support many kind of compilers, performance and debugging tools like gdb, valgrind. It also has tools for data analytics including statistics, machine learning, imaging etc.


### **Tianhe**

##### **prototype**
Work 分区: MT-2000+, 主频 2.0GHz,一个结点是 32 核,单节点内存 16GB,实际上测试使用的是64核;
Test 分区: FT-2000+, 主频 2.3GHz,一个结点是 64 核,单节点内存 64GB;
相比较天河一号: X5670, 主频 2.93GHz,一个结点是 12 核,单节点内存 24GB;

##### **usage**
加载库, 查看库, 查看库安装路径的命令和Edison相同;
```
yhi #查看可用结点;
yhqueue -u wzhang #查看提交的任务;
yhrun -N 3 -n 32 -p work -w cn[63-48] ./gtc # -N是结点数,-p是指定执行分区,-w指定结点;
yhcancel #取消任务;
yhbatch -N 32 -n 32 -c 32 -p test ./jobscript;
yhbatch -N 32 -n 32 -c 1 -x[41,51] -p test ./jobscript  #-x指定不使用哪些结点;
```
