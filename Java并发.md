#### Thread
1. sleep():休眠当前正在执行的线程。可能会抛出 InterruptedException。因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。没有释放锁。
2. yield():使得线程放弃当前分得的 CPU 时间，但是不使线程阻塞，即线程仍处于可执行状态，随时可能再次分得 CPU 时间。调用 yield() 的效果等价于调度程序认为该线程已执行了足够的时间从而转到另一个线程。
3. join():在线程中调用另一个线程的 join() 方法，会将当前线程挂起，直到目标线程结束。
4. wait() 和 sleep() 的区别
    * wait() 是 Object 类的方法，而 sleep() 是 Thread 的静态方法；
    * wait() 会放弃锁，而 sleep() 不会；
5. synchronized
    * 同步一个方法使多个线程不能同时访问该方法。
    * 保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。
    * 同步一个代码块。
6. 对象锁Lock：相比于关键字synchronized，更加灵活。
7. volatile：普通共享变量被修改之后，什么时候被写入主存是不确定的。volatile 关键字会保证每次修改共享变量之后该值会立即更新到内存中，并且在读取时会从内存中读取值。所以并没有影响到操作的原子性，只是提供了一种内存屏障，保证修改之后被理科写入主存并在主存读取保证其他线程的可见性。除了double和long之外的基本类型数据的读写都是原子的。多线程访问还是可能发生数据修改的情况。
8. 线程池
    * newCachedThreadPool() 无界线程池：可以无限提交任务
        1. 缓存型池子，先查看池中有没有以前建立的线程，如果有，就 reuse 如果没有，就建一个新的线程加入池中
        2. 缓存型池子通常用于执行一些生存期很短的异步型任务 因此在一些面向连接的 daemon 型 SERVER 中用得不多。但对于生存期短的异步任务，它是 Executor 的首选。
        3. 能 reuse 的
    * newFixedThreadPool()：有限制的，进入队列。队列满了之后按照用户设置的策略。
        1. newFixedThreadPool 与 cacheThreadPool 差不多，也是能 reuse 就用，但不能随时建新的线程。
        2. 其独特之处:任意时间点，最多只能有固定数目的活动线程存在，此时如果有新的线程要建立，只能放在另外的队列中等待，直到当前的线程中某个线程终止直接被移出池子。
        3. 和 cacheThreadPool 不
    * SingleThreadExecutor()：有限制的，进入队列。队列满了之后按照用户设置的策略。
        1. 单例线程，任意时间池中只能有一个线程
        2. 用的是和 cache 池和 fixed 池相同的底层池，但线程数目是 1-1,0 秒 IDLE（无 IDLE）
    * 线程等待队列饱和时处理策略
        1. abort: 丢弃任务并并抛出 rejectedExecutionException异常。
        2. CallerRuns: 退还给调用者线程运行新的任务来处理。
        3. Discard: 新提交的任务直接丢弃。
        4. DiscardOldest: 丢弃最老任务，也就是队头的任务，然后将任务提交进来。

#### 消费者，生产者模型
1. 定义：生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。这个阻塞队列就是用来给生产者和消费者解耦的。
2. 生产者消费者队列实现
    ~~~
        public class MyBlockingQueue<T> {

        private int size;
        private final LinkedList<T> mList = new LinkedList<>();
        private final Object lock = new Object();

        public MyBlockingQueue(int size) {
            this.size = size;
        }

        public void put(T task) throws InterruptedException {
            synchronized (lock) {
                while (mList.size() >= size) {
                    lock.wait();
                }
                mList.addLast(task);
                lock.notifyAll();
            }
        }

        public T take() throws InterruptedException {
            synchronized (lock) {
                while (mList.size() <= 0) {
                    lock.wait();
                }
                T task = mList.getFirst();
                lock.notifyAll();
                return task;
            }
        }
    }
    ~~~
#### 公平锁的实现