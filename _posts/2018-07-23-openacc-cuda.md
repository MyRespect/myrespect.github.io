---
layout: post
title: "Notes for openACC and CUDA"
categories: HPC
tags: openAcc CUDA
--- 

* content
{:toc}

openAcc and CUDA are really useful to accelerate the speed of program. In this post, I mainly make some notes about openAcc and CUDA.




#### **openACC**
(1) pgaccelinfo	查看Nvidia GPU和CUDA驱动是否正确安装;

(2) For basic use of openACC, you can refer to [gpuworld](http://bbs.gpuworld.cn/thread-347-1-1.html);

(3) pgcc -acc -ta=tesla:cc70 -Minfo=all a1.c  //the -⁠acc flag to enable OpenACC directives, the -ta=tesla flag to target NVIDIA GPUs and the -Minfo flag to display information while compiling;

(4) export PGI_ACC_TIME=1   程序执行完后会输出执行时间统计;

##### **operation notes**
(1) 对于基本的数据拷贝操作，copy/copyin/copyout,  copyin 拷贝到GPU, copyout 是拷贝回CPU, copy是语句块开始时copyin, 结束时copyout.

(2) 对于简单数据，acc可以自动处理，对于数组或引用，需要手工指派数据拷贝的起始地址，拷贝长度; 例如, int& mgrid (实际上是一个整数, 但是对它做了引用), 对于一个指针而言,告诉GPU拷贝一个就够了, 所以是使用mgrid[0:1];

(3) 块内要进行cout调试输出的话，要update host数据;

(4) 对于Array 等自定义类型, Copy操作执行的是浅拷贝, 采用其他方式提前放到GPU内存中时, 需要申明present;

(5) atomic update 原子更新, 相当于加锁, 在修改之前做记录;

#### **CUDA**
CUDA 是并行计算平台和编程模型, 由NVIDIA的GPU提供支持, 下面是一段CUDA代码示例：
```
__global__
void parallel_pi(long samples, long *in, trng::yarn5s r) {
	long rank=threadIdx.x;
	long size=blockDim.x;
	r.jump(2*(rank*samples/size));// jump ahead
	
	trng::uniform01_dist<float> u;// random number distribution
	in[rank]=0; // local number of points in c i r c l e

	for (long i=rank*samples/size; i<(rank+1)*samples/size; ++i) {
		float x=u(r), y=u(r);
		// choose random x − and y − coordinates
                // std::cout<<x<<" ";
		if (x*x+y*y<=1)
		// i s point in c i r c l e ?
		++in[rank];
		// increase thread − local counter
	}
}

int main(int argc, char *argv[]) {
const long samples=1000000l; // total number of points in square
const int size=128; // number of threads
long *in_device;
cudaMalloc(&in_device, size* sizeof(*in_device));
trng::yarn5s r;

// s t a r t parallel Monte Carlo
parallel_pi<<<1, size>>>(samples, in_device, r);
// gather results
long *in=new long[size];
cudaMemcpy(in, in_device, size* sizeof(*in), cudaMemcpyDeviceToHost);
long sum=0;
for (int rank=0; rank<size; ++rank)
	sum+=in[rank];
// print result
std::cout << " pi = " << 4.0*sum/samples << std::endl;
return EXIT_SUCCESS;
}
```
一块GPU有几千个核心，每个核心能运行10个以上的线程，所有的线程合在一起称为一个网格(grid), 网格再剖分成线程块(block), 线程块包含若干线程，cuda c引入了新的数据类型dim3, 相当于一个结构体，分别是unsigned int x,y,z; 网格尺寸用内置变量gridDim表示，gridDim.x, gridDim.y, gridDim.z 分别表示x,y,z方向上的线程块数量，网格中每个线程块的编号用内置变量blockIdx表示,blockIdx.x, blockId.y, blockId.z；线程块的尺寸用内置变量blockDim 表示，blockDim.x表示当前线程块在x,y,z方向上拥有的线程数量，任意一个线程块内的线程编号用内置变量threadIdx来表示；block用于指定每个线程块的形状，grid用于制定线程网格的形状．

调用流程：数据初始化，开辟内存，将数据复制到设备内存中，使用一对三尖号<<<第一个参数制定线程网格的形状，第2个参数制定线程块的形状>>>调用一个在设备上执行的函数，然后设备执行内核中的并行代码；内核代码执行完成后，控制权交还主机，主机从设备上取回内核的并行计算结果．

编译cuda代码, 这一段编译指令比较特殊，因为使用了TRNG随机数库，所以指定了库的地址：
```
nvcc -o rancuda rancuda.cu -lcuda -lcudart -L/usr/local/lib -ltrng4 -Xcompiler \"-Wl,-rpath,/usr/local/lib\"
```

CUDA C语言对Ｃ语言的扩展之一是加入一些函数前缀:
(1) __host__ int foo(int a){}与C或者C++中的foo(int a){}相同，是由CPU调用，由CPU执行的函数; host前缀修饰的事普通函数，默认缺省，可以调用普通函数;

(2) __global__ void foo(int a){}表示一个内核函数，是一组由GPU执行的并行计算任务，以foo<<>>(a)的形式或者driver API的形式调用; global前缀表示该函数需要在主机上调用并且在设备上执行；不能运行CPU里常见的函数;

(3) __device__ int foo(int a){}则表示一个由GPU中一个线程调用的函数, device前缀定义的函数只能在GPU上执行，所以device修饰的函数里面不能调用一般常见的函数;

如果出现报错如下，则因为将一个普通的函数错误地添加进入了global前缀定义函数，在CUDA文件.cu文件中是不允许的。
```
error : calling a __host__ function from a __global__ function is not allowed.
```

CUDA的cudaMalloc()函数原型:
```
cudaError_t cudaMalloc (void **devPtr, size_t  size );
```
使用实例:
```
float *device_data=NULL;
size_t size=1024*sizeof(float);
cudaMalloc((void**)&device_data,size);
```
之所以取device_data的地址，是为了将cudaMalloc在显存上的数组首地址赋值给device_data;　第一个参数传递的是存储在cpu内存中的指针变量的地址，cudaMalloc执行完成后，向这个地址中写入一个地址值; 前面写(void\*\*)因为有些编译器不支持隐式类型转换．