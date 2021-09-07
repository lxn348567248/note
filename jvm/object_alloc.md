# 对象分配置的过程

## 加载类
>如果类没有加载，通过类加载器加载类。
    tip:类在遇到以下指令时，要进行加载。<br/>
    1.new，getStatic（静态字段的读取）,putStatic(静态字段的修改),invokeStatic(调用静态方法）指令<br/>
    2. 通过反射方式操作类时<br/>
    3.类在初始化，父类未初化，初始化你类<br/>
    4.4.jvm启动时，用户需要指定一个执行的主类（包含main的类）虚拟机会先执行这个类

## 分配内存
>类对象的大小，在根据类加载时确定对象的大小，分配内存空间。
### 内存分配置的方式
>1.指针碰撞<br/>
通过指针指分隔已经分配与未分隔的内存，当要分配内存，通过移动此指针进行分配内存。<br/>
>2.空闲列表<br/>
通过一个列表记录已分配和未分配的列表，当要分配内存时，能过这个空闭列表分配内存。

### 分配内存并发问题
>1.CAS(compare and swap)方式<br/>
虚拟机通过CAS方式加失败重试实现内存的分配。<br/>
>2.TLAB方式(Thread Local Alloc Buffer)<br/>
通过在每一个线程中，预分配一块内存区域。jvm默认是开启这个方式分配内存的(-XX:+UseTLAB)

## 设置头信息
>设置对象的头信息，头信息由markworld,kclasspoint组成。markworld主要包含hashCode,age,锁等信息。（可以通过jol这个库可以查看对象头信息）
## 调用\<init\>方法
>这个\<init\>方法，即构造方法。
