#### 进程内存模型
1. 系统为每一个进程分配的内存空间有用户空间和内核空间两部分。不同进程间的用户空间是相互隔离的，无法共享。但是系统里所有不同进程共享同一个内核空间。进程内用户空间和内核空间进行交互需要通过系统调用函数：
    * copy_from_user:将用户空间的数据拷贝到内核空间
    * copy_to_user  :将内核空间的数据拷贝到用户空间
2. 进程隔离：为了保证安全性和独立性，一个进程不能直接操作或者访问另一个进程，即android的进程是相互独立的，隔离的。所以不同的进程间无法直接进行交互。

#### 内存映射：关联进程中的一个虚拟内存区域和一个磁盘上的对象，使得二者存在映射关系
1. 定义
    * 映射过程：初始化虚拟内存区域。
    * 虚拟内存区域被初始化之后就会在交换空间中换来换去。
    * 被映射的对象称为共享对象。
2. 作用
    * 内存映射完成后，多个进程的虚拟内存区域已和同一个共享对象建立了映射关系，若其中一个进程对该虚拟区域进行写操作，那么其他把这个共享对象映射到自己虚拟内存区域的进程也是可见的。
3. 优点
    * 提高了数据的读写，传输的性能
        1. 减少了数据拷贝次数
        2. 通过内存映射，用户空间和内核空间可以进行高效交互
        3. 用内存读写代替I/O读写
    * 通过虚拟内存和共享对象提高内存的利用率
4. 跨进程通信过程
    * 创建一块共享的接收缓存区
    * 实现地址映射关系：将内核缓存区和接收进程用户空间同时映射到这个共享的接收缓存区。
    * 发送进程通过系统调用copy_from_user发送数据到内核缓存区
    * 由于内核缓存区和接收进程用户空间地址都映射到这个共享的接收缓存区，所以发送数据到内核缓存区也相当于发送到了接收进程用户空间，从而实现跨进程通信

#### 进程间通信过程中的几个角色
1. 客户进程：需要调用远程服务的本地进程，客户端进程。通过defaultServiceManager()方法拿到ServiceManager进程的binder引用，通过这个binder引用查询远程目标service是否存在，存在的话就返回远程service的binder引用。
2. 服务进程：提供服务的远程进程。启动之后通过defaultServiceManager()方法拿到ServiceManager进程的binder引用将自己注册进去，以 服务名 - binder引用 键值对的方式保存。
3. ServiceManager进程：管理服务进程的注册和查询，binder引用(服务进程的代理)的保存。Android系统启动完成之后，就会启动这个SM进程，SM进程启动之后将创建自己的binder对象并保存在binder数组的起始索引0位置。其他进程通过defaultServiceManager()方法获取到这个binder对象。
4. Binder驱动：虚拟设备驱动。通过内存映射完成不同进程间的数据通信，创建线程池管理进程的请求。

#### binder实现进程间通信过程
1. ServiceManager的启动：Android系统启动过程中，在启动init进程之后将会启动ServiceManager进程，ServiceManager进程通过系统调用在BinderDriver中创建一个代表自己的binder实体存放在数组的索引0第一个位置上，并维护一个服务名字到binder引用的映射。
2. 服务注册：启动服务，通过系统调用在BinderDriver中创建一个代表自己的binder实体，然后通过defaultServiceManager()方法获取到ServiceManager的binder引用，使用这个binder引用通过系统调用发送数据给binderDriver，然后这个binderDriver以内存映射的方式把服务名字和相关信息以及远程服务的binder引用保存在ServiceManager创建的映射里面。
3. 客户端连接服务：客户端通过defaultServiceManager()方法获取ServiceManager的binder引用，使用这个binder引用通过系统调用发送数据给binderDriver，然后BinderDriver以内存映射的方式发送数据到ServiceManager进程查询对应服务。查询到之后，返回远程服务的Binder实体的代理对象给客户端进程。
4. 调用服务：通过远程服务的binder引用，以内存映射的方式把数据发送给远程服务从而调用对应的方法。此时客户端进程被挂起，服务进程将结果以内存映射的方式发送给BinderDriver，BinderDriver再通过系统调用将数据返回给客户进程，客户进程恢复，完成调用。

#### activity的启动流程
1. 角色
    * ActivityManagerService：处于system_server进程，负责activity的启动和生命周期。使用ApplicationThreadProxy跟应用进程进行远程通信。
    * ApplicationThread：处于应用进程，负责跟AMS进行交互，运行在binder线程中。通过Handler把AMS传递过来的数据转到ActivityThread进行执行。使用ActivityManagerProxy跟AMS进行远程通信。
    * ActivityThread：应用的UI线程，完成activity的实际创建和生命周期。
2. 启动过程
    1. 应用进程调用startActivity，进而依赖ActivityManagerNative.getDefault获取ActivityManagerProxy将启动新界面的数据通过binder进程间通信发送给AMS。
    2. AMS收到应用进程的请求数据之后，进行数据解析，检测需要启动的activity是否有在清单文件中注册，创建activity的抽象的映像ActivityRecord并保存在栈中。然后通过ApplicationThreadProxy发送数据到应用进程，启动activity。
    3. ApplicationThread收到数据之后，通过ActivityThread类里面的Handler类型对象H将数据从binder线程转入ActivityThread线程。根据接收的数据使用反射创建无参activity对象。
    4. 要实现activity的插件化，就必须绕过AMS的检测，我们通常在清单文件声明注册一个StubActivity，在检测通过之后，Handler将数据转入ActivityThread之后创建activity对象的时候，使用反射替换这个activity对象，并且和返回的binder对象绑定，AMS通过这个binder对象来控制管理activity。
