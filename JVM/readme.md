# 概述
jvm， java虚拟机

# 种类
## hotspot
目前用的很多
## jrocket
## dalvik

# 内存空间
## 线程共享
### 堆
> ### 可以认为所有的实例对象都是创建于此。
Class级别的对象可能有所出入。
1.7开始，常量池也放在堆中。

### 非堆（方法区）
> ### 方法区只是一种规范，没有规定具体实现方法。
HotSpot虚拟机在1.7和以前版本采用的是永久代的实现方法
1.8开始，就使用metaspace（元空间）实现方法区。


## 线程私有
多线程得情况下，有可能导致内存溢出 OOM Error


### 程序计数器PC
> ### 指向class文件的code段

### 虚拟机栈
>>局部变量表是一组变量值存储空间，用于存放方法参数和方法内部定义的局部变量
其中存放的数据的类型是编译期可知的各种**基本数据类型**、**对象引用**（reference）和**returnAddress**类型（它指向了一条字节码指令的地址）。局部变量表所需的内存空间在**编译期间完成分配**，即在Java程序被编译成Class文件时，就确定了所需分配的最大局部变量表的容量。当进入一个方法时，这个方法需要在栈中分配多大的局部变量空间是完全确定的，**在方法运行期间不会改变局部变量表的大小。**
局部变量表中的第0位索引的Slot默认是用于传递方法所属对象实例的引用，在方法中可以通过关键字“this”来访问这个隐含的参数
> ### 调用java方法的时候会在虚拟机栈中压入一个  **栈帧**
每调用一个方法就会压入一个新的栈帧，调用完毕就会弹出当前栈帧。
> ### 栈帧中有几个重要的区域
>> ### 操作数栈
执行指令的时候，就是针对该栈的栈顶的元素执行。
>> ### pc
程序计数器，指向class文件中的code段
>> ### 局部变量表
存放局部变量，局部引用
局部变量表的第0项用于放置调用这个方法的对象的引用
> ### 递归层数过深，就会导致 StackOutFlowError

### 本地方法栈
>> ### 基本与虚拟机栈相同，不再赘述。


# 方法调用
> 方法的调用不等于方法执行，方法调用阶段唯一的任务是 **确定被调用方法的版本**
> ### 在Java虚拟机里提供了5条调用方法字节码指令，分别如下。
>> ### invokestatic
调用静态方法
>> ### invokespecial
调用实例构造器<init>方法、私用方法和父类方法
>> ### invokevirtual
调用所有的虚方法
>> ### invokeinterface
调用接口时，会在运行再确定一个实现接口的对象
>> ### invokedynamic
现在运行时动态解析出调用点限定符引用的方法，再执行方法
>## 解析调用
**类加载的解析阶段** ，会将其中的一部分符号引用转化为直接引用
只有被 invokestatic 和 invokespecial 指令调用的方法，可以在解析阶段中确定调用的版本
这两个调用被称为解析调用
>## 动态调用
>> ### 静态分派
在编译时期就可以确定函数调用入口的分派
在 **重载** 时通过参数的静态类型而不是实际类型作为判断依据的
在编译阶段Java编译器 **根据参数的静态类型** 决定使用哪个重载版本
>>> **静态分配的典型动作是方法重载**，静态分派发生在编译阶段，虽然编译器能确定方法的重载版本，但是很多情况下这个重载的版本并不是“唯一的”，往往只能确定一个“更加合适的”版本。
>> ### 动态分派
首先这种分派得以成立的前提是
每一个类会有自己的方法表，是常量池中的一部分，存放着符号引用到直接引用的变换。
而在继承的时候，子类会有自己的方法表，如果重写了父类的方法，那么这个符号引用到直接引用的变换会存在于子类的方法表中。
>>> ### 动态分派的过程
1. 在invoke之前，会类似于调用带参函数一样，将对象的引用压到操作数栈顶，然后invoke virtual
2.  1. 虚拟机会根据这个引用找到它得方法表
    2. 如果从中找到了符号引用，那么会分析它得保护级别，private 或者 protect 等
    3. 如果权限验证通过，那么直接转为直接引用然后call即可。
    4. 验证不通过，抛出java.lang.IllegaAccessError异常
3. 如果找不到，那么会到这个类得父类得方法表中查找，找不到就一直往上。最后找不到就抛出 java.lang.AbstractMethodError异常。