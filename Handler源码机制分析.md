#### Handler的作用：线程切换和延迟执行
1. 角色
    * Message:Handler要处理的事件信息都保存在这个对象里面，一个message就是一个待处理的事件。内部自带大小50消息池，通过单链表头插法进行回收。
    * MessageQueue:事件队列，保存需要处理的事件。
    * Looper:消息轮循检测。通过looper.loop()进入死循环进行消息队列的轮循检测。
    * Handler:事件的发送和处理。创建handler的时候会获取当前线程的looper对象给自己的成员变量，如果为空会报异常。主线程可以直接创建是因为应用启动的时候，主线程就已经创建了自己的looper对象，如果在子线程创建handler对象，必须先调用looper.prepare()方法创建looper对象。
    * ThreadLocal:looper的静态成员变量，用来保存looper对象。在不同的线程通过Thread.currentThread()方法获取到当前线程，然后根据传进来的looper创建threadLocalMap对象并赋值给前线程成员变量threadLocalMap。获取的时候也是拿到当前线程然后取线程的成员变量。

2. 线程间通信过程
    * 在需要处理事件的线程创建handler对象
        1. 如果当前线程是主线程，可以直接创建handler，后面发送的事件将会在主线程得到处理
        2. 如果当前线程是子线程，创建handler之前必须先调用looper.prepare()创建一个绑定到当前线程的looper对象，通过loop方法进入轮循状态。这个looper对象将保存在looper的静态成员变量sThreadLocal对象中。looper的保存方式是线程隔离的-> 通过Thread.currentThread()获取到当前线程，然后拿到当前线程的成员变量threadLocalMap保存looper对象。
    * 创建message对象。message对象自带消息池，以单链表的方式保存消息。通过message.obtain()方法从链表头部获取之前创建的message对象，初始化成员变量进行赋值。用完之后使用recycle方法以头插法的方式进行回收。
    * message发送。handler通过sendMessage系列方法把消息发送到线程的消息队列里面进行保存。
    * looper对象轮循消息队列，获取消息时间戳，跟当前时间进行比较，如果当前时间大于或等于时间戳，looper对象通过dispatchMessage方法将消息分发出来。
    * handler获取到需要处理的message之后，根据优先级message.callBack成员变量 > handler.callBack成员变量 > handler.handleMessage()成员方法的优先级来处理消息。