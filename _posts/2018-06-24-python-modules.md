---
layout: post
title: "Python Modules"
categories: Language
tags: Python
--- 

* content
{:toc}

Python has many useful packages, in this post, I make some notes about some python modules: subprocess, paramiko, zipfile, format. 




#### **Anaconda**
```
conda install -c jjhelmus tensorflow-base
conda -n 指定环境名称(name)
source activate basename
source deactivate  basename
conda remove -n 环境名 –all
conda info -e 查看已安装的环境
```


python zip 函数，将对象中的元素打包成一个个元组，返回由这些元组组成的对象, 可用list 转换为列表; 用list和tuple等数据结构表示数组，　list是动态指针数组，保存的是对象的指针，其元素可以使任意类型的元素;

```
re.match(pattern, string, flag=0) # flag有re.I 忽略大小写，re.X为了增加可读性，忽略空格和注释; 正则表达式中^表示行的开头，$表示行的结束;

os.system(cmd) # python中执行linux命令

os.getcwd() # python中查看当前路径

os.path.abspath(‘.’)　# python中查看所在文件路径
```

#### **Timeit**
timeit库用来准确测量小段代码的执行时间．
```
timeit.timeit(stmt='pass', setup='pass', timer=<default timer>, number=1000000)
# 创建一个Timer实例，参数分别是stmt（需要测量的语句或函数），setup（初始化代码或构建环境的导入语句），timer（计时函数），number（每一次测量中语句被执行的次数）;

timeit.repeat(stmt='pass', setup='pass', timer=<default timer>, repeat=3, number=1000000)
# 创建一个Timer实例，指定整个试验的重复次数，返回一个包含了每次试验的执行时间的列表，利用这一函数可以很方便得实现多次试验取平均的方法;

# Example:

def add2(x, c):
    return [xx+c for xx in x]

x = np.random.random(10**5).astype(np.float32).tolist()
t0=timeit.timeit("add2(x,1)", setup="from __main__ import add2, x", number=1000)
print("cost {} seconds ".format(t0))
```

#### **Numba**
Numba translates Python functions to optimized machine code at runtime using the industry-standard LLVM compiler library.　JIT, Just-in-time compilation.

Numba likes loops, numpy functions numpy broadcasting．However, Pandas library is not understood by Numba and as a result Numba would simply run this code via the interpreter but with the added cost of the Numba internal overheads.
```
@jit(nopython=True) # Set "nopython" mode for best performance
```
注意不能盲目迷信jit，不是所有的都可以用jit加速，比如[[[i,j,0] for j in range(y)] for i in range(x)]这样的一条语句使用jit编译后反而会慢了，当然，但是测试的平均时间里面还包含了jit编译语句的时间，这一部分相对来说也是耗时的．

Numpy的Ufunc(universal function)函数能够作用于narray对象的每一个元素上，而不是针对narray对象操作．例如add(), subtract(), multiple(), divide()等．我们可以使用Numba的vectorize实现Numpy的Ufunc功能，从而加速代码的运行；vectorize下的函数所接受的参数都是一个个的数而非整个数组，而我们使用的时候传入的是整个数组；vectorize最炫酷的地方在于，它可以“并行”. 使用 @jit(nogil=True) 实现高效并发，一般来说，数据量越大、并发的效果越明显。反之，数据量小的时候，并发很有可能会降低性能

```
The following Python language features are not supported in Version 0.41:
1. Class definition
2. Exception handling (try .. except, try .. finally)
3. Context management (the with statement)
4. Some comprehensions (list comprehension is supported, but not dict, set or generator comprehensions)
5. Generator delegation (yield from)
```
参考链接:[numpysupported](https://numba.pydata.org/numba-doc/dev/reference/numpysupported.html),[pysupported](http://numba.pydata.org/numba-doc/dev/reference/pysupported.html), [performance-tips](https://numba.pydata.org/numba-doc/dev/user/performance-tips.html)

#### **h5py**
Python库 h5py, h5py文件是存放两类对象的容器，数据集和组，dataset是类似数组类的数据集合，group像文件夹一样的容器，好比python的字典，有key和value，group可以存放dataset或者其他的group.

#### **Subprocess**
One process can fork a subprocess, making the subprocess executes another program, the **subprocess** module in python can achieve that.

subprocess.call([“ls”, “ -l”])
This function is encapsulated based on subprocess.Popen():
class Popen(args, bufsize=0, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=False, shell=False, cwd=None, env=None, universal_newlines=False, startupinfo=None, creationflags=0)

subprocess.check_call() // 父进程等待子进程完成，返回0,　检查退出信息
subprocess.check_output() //父进程等待子进程完成，返回子进程向标准输出的输出结果，检查退出信息
```
child = subprocess.Popen('ping -c4 blog.linuxeye.com',shell=True)
print(child.stdout.read())
child.wait()
child.kill()

child1=subprocess.Popen(["cat", "/etc/passwd"], stdout=subprocess.PIPE)
child2=subprocess.Popen(["grep", "0:0"], stdin=child1.stdout, stdout=subprocess.PIPE)
out.child2.communicate()
```
subprocess.PIPE实际上为文本提供了一个缓存区，直到communicate()方法从PIPE中读取PIPE中的文本. communicate()是Popen对象一个方法，该方法会阻塞父进程，直到子进程完成．

#### **joblib**
保存模型:
```
from sklearn.externals import joblib
joblib.dump(value, filename, compress=0, protocol=None, cache_size=None)
```
持久化任意的python对象为一个文件.并且返回一个字符串列表,表示这些数据分别存放的位置.


#### **copy**
Python共享引用的概念，增强赋值语句(b+=1)比普通赋值语句(b=b+1）高级，因为增强赋值比普通的多了”写回”功能，在条件符合的情况下会以追加的方式进行处理，而普通赋值会新建一个对象，这一特点导致了增强赋值语句中的变量对象始终只有一个，Python 解析器解析该语句时不会额外创建出新的内存对象．

Python中也要注意浅拷贝和深拷贝的问题:
```
from copy import deepcopy
=, copy, deepcopy
```


#### **Paramiko**
paramiko模块，基于SSH用于连接远程服务器并执行相关操作,可以基于用户名密码连接，基于公钥秘钥连接，私钥字符进行连接．
```
import paramiko
ssh=paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect(hostname='172.11.2.109', port=22, username='root', password='founder123')
stdin, stdout, stderr = ssh.exec_command('ls')
result = stdout.read()
```

#### **Zipfile**
gz: 通常只压缩一个文件,与tar结合起来可以实现先打包,再压缩. tar只打包不压缩
zip: 压缩率低于tar

```
import zipfile, os  
z = zipfile.ZipFile(filename, 'w') 
if os.path.isdir(testdir):  
     for d in os.listdir(testdir):  
         z.write(testdir+os.sep+d) 
         z.close()

z = zipfile.ZipFile(filename, 'r')  
for i in z.infolist():  
    print (i.file_size, i.header_offset, i.filename)
```
#### **Format**
通过关键字:
print("{name} and {action}".format(name="zhang", action="play"))

通过位置:
print("{0} and {1}".format("zhang", "play"))

进制转化: 
b, o, d, x分别表示二，八，十，十六进制, 
s : 获取传入对象的__str__方法的返回值，并将其格式化到指定位置, 
r : 获取传入对象的__repr__方法的返回值，并将其格式化到指定位置
```
print('{:o}'.format(250))
print('Request: {0!r}'.format(self.request))
```
字符串格式化：
python 字符串格式化：{<参数序号>:<格式控制标记>}
{:0>5} 0表示填充, 5表示宽度, >表示右对齐,　^表示居中, <表示左对齐

#### **Shutil**
```
shutil.move
shutil.copytree
shutil.rmtree
shutil.copy #copy data and mode bits
```

#### **Warning**
```
warnings.filterwarning("ignore")

python -W ignore file.py
```