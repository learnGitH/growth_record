1、LockSupport

LockSupport.park():让当前线程阻塞到当前位置

LockSupport.unpark(thread):唤醒指定的线程thread,让对应执行了LockSupport.park()阻塞的线程继续执行

```
@Slf4j
public class Juc01_Thread_LockSupport {

    public static void main(String[] args) {

        Thread t0 = new Thread(new Runnable() {

            @Override
            public void run() {
                Thread current = Thread.currentThread();
                log.info("{},开始执行!",current.getName());
                for(;;){//spin 自旋
                    log.info("准备park住当前线程：{}....",current.getName());
                    LockSupport.park();
                    log.info("当前线程{}已经被唤醒....",current.getName());
                }
            }

        },"t0");

        t0.start();

        try {
            Thread.sleep(5000);
            log.info("准备唤醒{}线程!",t0.getName());
            LockSupport.unpark(t0);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

2、CAS

CAS比较并交换，例如现在主内存有一个值a=0,此时有两个线程T0,T1需要对其进行修改，则把该值读到工作内存，然后设置expect=0,和对应修改的值，然后写回主内存的时候将expect和主内存的值进行比较，如果相等，则把修改的值写到主内存，如果不相等，则重信读取主内存的值再继续修改判断（这个过程是原子性的）

```
@Slf4j
public class Juc04_Thread_Cas {
    /**
     * 当前加锁状态,记录加锁的次数
     */
    private volatile int state = 0;
    private static CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
    private static Juc04_Thread_Cas cas = new Juc04_Thread_Cas();

    public static void main(String[] args) {
        new Thread(new Worker(),"t-0").start();
        new Thread(new Worker(),"t-1").start();
        new Thread(new Worker(),"t-2").start();
        new Thread(new Worker(),"t-3").start();
        new Thread(new Worker(),"t-4").start();
    }

    static class Worker implements Runnable{

        @Override
        public void run() {
            log.info("请求:{}到达预定点,准备开始抢state:)",Thread.currentThread().getName());
            try {
                cyclicBarrier.await();
                if(cas.compareAndSwapState(0,1)){
                    log.info("当前请求:{},抢到锁!",Thread.currentThread().getName());
                }else{
                    log.info("当前请求:{},抢锁失败!",Thread.currentThread().getName());
                }
            } catch (InterruptedException|BrokenBarrierException e) {
                e.printStackTrace();
            }
        }

    }

    /**
     * 原子操作
     * @param oldValue
     *        oldvalue:线程工作内存当中的值
     * @param
     *        newValue:要替换的新值
     * @return
     */
    public final boolean compareAndSwapState(int oldValue,int newValue){
        return unsafe.compareAndSwapInt(this,stateOffset,oldValue,newValue);
    }

    private static final Unsafe unsafe = UnsafeInstance.reflectGetUnsafe();

    private static final long stateOffset;

    static {
        try {
            stateOffset = unsafe.objectFieldOffset(Juc04_Thread_Cas.class.getDeclaredField("state"));
        } catch (Exception e) {
            throw new Error();
        }
    }
}
```

3、CyclicBarrier



在当前类，执行ctrl+shift+alt+u查看该类的继承关系，



对于ReentrantLock的自我分析：

```
//t0,t1,t2
lock.lock();
	while(true) {
		if（获得锁则跳出循环）{  //获取锁的逻辑
			break;
		}
		//保存当前线程到队列，以便被获取
		queue.set(thread);
		LockSupport.park();	//让当前线程阻塞，执行unlock()的时候再来唤醒我
	}

//t0获取锁成功执行业务逻辑
//业务逻辑
lock.unlock();
//获取需要唤醒的线程
Thread thread = queue.get();
LockSupport.unpart(唤醒的线程);
```

总结：

1）、获取锁的逻辑需要用到CAS，AbstractQueuedSynchronizer里面有一个state被volatile修饰的变量，通过该变量执行CAS操作，来确保只有一个线程获取到锁。

2）、队列，因为没有获取到锁的线程需要保存起来，AbstractQueuedSynchronizer里面有一个静态的内部类Node,通过他来实现双向列表

3）、exclusiveOwnerThread该变量用来保存当前获取到锁的线程

4）、LockSupport：用来阻塞线程和唤起线程

5）、自旋



4、lock之AQS分析

此时有三个线程T0,T1,T2同一时间类抢锁，只有T0能够抢到

T0获取到锁的逻辑

```
public static ReentrantLock lock = new ReentrantLock(true);
//主要有公平锁和非公平锁，FairSync和NonfairSync都继承这Sync,Sync是ReentrantLock静态内部类，并且继承这AbstractQueuedSynchronizer（AQS）
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
}
//以下以公平锁来进行分析
final void lock() {
	acquire(1);
}

public final void acquire(int arg) {
	if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
       selfInterrupt();
}

//首先执行tryAcquire(int arg)尝试获取锁的逻辑（如果返回true说明获取到锁，false说明没有，需要执行后
//续的逻辑）
protected final boolean tryAcquire(int acquires) {
			//获取到当前的线程
            final Thread current = Thread.currentThread();
            //获取状态，因为是否已有线程获取锁是通过该状态来进行控制的，如果是1说明已有线程拥有锁的使用				//权，如果0说明还没有线程拥有
            int c = getState();
            if (c == 0) {
            	//这个hasQueuedPredecessors是关键，主要判断当前线程之前是否有队列线程，如果有则返回				 //true,不再执行后续获取锁的逻辑，而是需要入队，如果当前线程位于队列的前端或者队列为空，				//则返回false，说明可以获取锁（这里需要结合后面的逻辑来一起理解为什么要这样设计）
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            //这里不会有并发问题，永远只有一个线程可以进入，因为这里判断了是否拥有锁是否同一个线程（为了				//实现可重入锁）
            else if (current == getExclusiveOwnerThread()) {
            	//当前状态加1
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                //设置当前状态
                setState(nextc);
                return true;
            }
            return false;
}

//方法hasQueuedPredecessors理解
Returns:
	true if there is a queued thread preceding the current thread, and false if the 		current thread is at the head of the queue or the queue is empty
Since:1.7
public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
}
如果是第一次进来那么tail和head都为空，所以t和h也就是为空,所有h != t也就是为false,由于是&&的关系只要有一个为false则返回false,不再执行后面的逻辑
如果已经有了队列，但是里面只有一个空节点，那么说明队列已经初始化过，并且里面的线程都执行完了，没有等待的线程，这时h.next是等于null，由于或的关系，则一个true后就返回true了，返回ture说明了需要入队，当前线程不能执行获取锁，为什么会是这样呢？明明h.next等于null,说明队列已经没有等待的线程了，之所以会执行h.next这句说明h != t，说明队列有在排队的线程，而刚好执行到h.next的时候，刚好有线程获取到锁了弹出节点，所以h.next就等于null,说有线程已经获取到锁了，这时就应该执行入队操作。
最后s.thread != Thread.currentThread()，如果为true说明不相等，说明不是同一个线程，所以需要入队，让队头线程先获取锁，如果为false说明相等，则可以操作获取锁。

tryAquire(int acquires)方法总结：如果没有线程拥有锁，则执行获取锁，并把状态state设置为1，并设置exclusiveOwnerThread等于当前线程，并返回true.如果current == getExclusiveOwnerThread()说明是重入，则把state再加1并返回true,否则返回false.

//addWaiter(Node.EXCLUSIVE)逻辑分析（之所以会执行该方法，是因为tryAcquire(arg)返回false,没有获取到
//锁T1,T2线程，需要执行入队操作）
private Node addWaiter(Node mode) {
		//创建一个新的节点
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        //这里第一次进来的时候tail为空所以不会执行，什么时候会执行呢？当队列已经建立了，然后将节点放入队尾，然后返回该节点，跟enq(node)里面的逻辑时一致的，只是enq(node)里面多了一步初始化队列的操作
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        //插入节点到队列，如果第一次没有初始化，则先进行初始化
        enq(node);
        return node;
}

//enq(node)
    /**
     * Inserts node into queue, initializing if necessary. See picture above.
     * @param node the node to insert
     * @return node's predecessor
     */
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            //第一次为null时进行初始化
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
            	//将节点放在队尾，然后进行返回该节点
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
    
addWaiter总结：主要新建一个节点，插入到队尾，然后返回该节点，如果有必要先初始化队列（当队列为空的时候）

//acquireQueued（addWaiter(Node.EXCLUSIVE), arg)）逻辑分析
addWaiter(Node.EXCLUSIVE)上面已经分析了，插入队列并返回该节点
    /**
       以独占不间断模式获取队列中已存在的线程。由条件等待使用
     * Acquires in exclusive uninterruptible mode for thread already in
     * queue. Used by condition wait methods as well as acquire.
     *
     * @param node the node
     * @param arg the acquire argument
     * @return {@code true} if interrupted while waiting
     */
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
            	//返回该节点的前一个节点prev
                final Node p = node.predecessor();
                //如果等于head说明该节点时队列的第一个节点，可以尝试获取锁的逻辑
                //tryAcquire(arg)获取锁的逻辑，前面已经分析过了
                //之所以要这样设计是因为是第一节点，说明可以获取锁，如果获取到则执行，获取不到再阻塞
                //因为线程的阻塞和唤醒需要时间，这样做可以节省一些时间，因为放入队列的时候有可能别的线程				 //已经释放锁了
                if (p == head && tryAcquire(arg)) {
                	//设置头节点为node
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                //如果不是第一个节点，或者没有获取到锁，则执行阻塞的逻辑，第一轮先执行
                //shouldParkAfterFailedAcquire(p, node)主要设置waitStatus的状态为Node.SIGNAL
                //因为只有该状态才能被唤醒，默认该状态为0，执行完返回false,然后进行第二轮循环，就返回					//true了，在执行parkAndCheckInterrupt()进行LockSupport.park(this)阻塞，等待
                //中断或者LockSupport.unpark进行唤醒
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    
//设置头节点的逻辑，又变会空节点，head和tail都指向它
    /**
     * Sets head of queue to be node, thus dequeuing. Called only by
     * acquire methods.  Also nulls out unused fields for sake of GC
     * and to suppress unnecessary signals and traversals.
     *
     * @param node the node
     */
    private void setHead(Node node) {
        head = node;
        node.thread = null;
        node.prev = null;
    }

```



5、unlock分析

```
Attempts to release this lock.
If the current thread is the holder of this lock then the hold count is decremented. If the hold count is now zero then the lock is released. If the current thread is not the holder of this lock then IllegalMonitorStateException is thrown.
Throws:IllegalMonitorStateException – if the current thread does not hold this lock
public void unlock() {
    sync.release(1);
}

//release(1)
    /**
     * Releases in exclusive mode.  Implemented by unblocking one or
     * more threads if {@link #tryRelease} returns true.
     * This method can be used to implement method {@link Lock#unlock}.
     *
     * @param arg the release argument.  This value is conveyed to
     *        {@link #tryRelease} but is otherwise uninterpreted and
     *        can represent anything you like.
     * @return the value returned from {@link #tryRelease}
     */
    public final boolean release(int arg) {
    	//执行tryRelease(arg)进行释放锁，主要进行把state减1，如果减1后state为0，则满足释放的条件
    	//free设置为true,然后把exclusiveOwnerThread设置为空，并返回free为true,说明已成功释放锁，
    	//可以唤醒其他线程来抢锁
        if (tryRelease(arg)) {
            Node h = head;
            //判断h.waitStatus != 0则符合唤醒的条件，因为放入的时候已经设置了waitStatus为SIGNAL了
            //这里为什么要判断h的waitStaus呢？因为放入节点的时候就是这样设计的，放在该节点的前一个节点
            //以便判断
            if (h != null && h.waitStatus != 0)
                //唤醒
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    
//unparkSuccessor唤醒队头的节点
    /**
     * Wakes up node's successor, if one exists.
     *
     * @param node the node
     */
    private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        //由于放入队列是SIGNAL=-1,所以满足条件
        if (ws < 0)
        	//将节点由-1设置为0
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
         //获取该节点的下一个节点，也就是要唤醒的节点（因为头节点是空节点，设置是为了方便判断）
        Node s = node.next;
        //如果节点为空，或者waitStatus大于0说明不满足唤醒的条件，不进行唤醒
        //waitStatus大于0表示该线程已被取消
        if (s == null || s.waitStatus > 0) {
        	//设置为空，以便GC
            s = null;
            //当需要唤醒的节点的waitStatus>0时，说明该节点已取消，则把它置为null,并且再判断接下去的节			  //点是否满足被唤醒的条件
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
        	//唤醒，此时会执行原来阻塞该线程的自旋的逻辑，进行获取锁，并把该节点去点，重新设置头节点
            LockSupport.unpark(s.thread);
    }

```



6、阻塞队列BlockingQueue

常见的4种阻塞队列：

ArrayBlockingQueue:由数组支持的有界队列

LinkedBlockingQueue:由链表节点支持的可选有界队列

PriorityBlockingQueue:由优先级堆支持的无界优先级队列

DelayQueue:由优先级堆支持的、基于时间的调度队列



7、Semaphore

Semaphore 字面意思是信号量的意思，它的作用是控制访问特定资源的线程数目，底层依 

赖AQS的状态State，是在生产当中比较常用的一个工具类。

```
public class SemaphoreRunner {

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(2);
        for (int i = 0; i < 5; i++) {
            new Thread(new Task(semaphore,"shenhaibin+" + i)).start();
        }
    }

    static class Task extends Thread{
        Semaphore semaphore;
        public Task(Semaphore semaphore,String tname) {
            this.semaphore= semaphore;
            this.setName(tname);
        }

        public void run() {
            try{
                semaphore.acquire();
                System.out.println(Thread.currentThread().getName()+":aquire() at time:"+System.currentTimeMillis());
                Thread.sleep(1000);
                semaphore.release();
                System.out.println(Thread.currentThread().getName()+":aquire() at time:"+System.currentTimeMillis());
            }catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

}
```

一次只有两个线程执行 acquire()，只有线程进行 release() 方法后 ,才会有别的线程执行 acquire()。 



8、CountDownLatch

CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。 

```
/**
 2 * 看大夫任务
 3 */
public class SeeDoctorTask implements Runnable {
    private CountDownLatch countDownLatch;

        public SeeDoctorTask(CountDownLatch countDownLatch){
            this.countDownLatch = countDownLatch;
        }

        public void run() {
            try {
                System.out.println("开始看医生");
                Thread.sleep(3000);
                System.out.println("看医生结束，准备离开病房");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
            if (countDownLatch != null)
                countDownLatch.countDown();
            }
        }

}

/**
 27 * 排队的任务
 28 */
public class QueueTask implements Runnable {
    private CountDownLatch countDownLatch;

    public QueueTask(CountDownLatch countDownLatch) {
        this.countDownLatch = countDownLatch;
    }

    public void run() {
        try {
            System.out.println("开始在医院药房排队买药....");
            Thread.sleep(5000);
            System.out.println("排队成功，可以开始缴费买药");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            if (countDownLatch != null)
                countDownLatch.countDown();
        }
    }
}

public class CountDownLaunchRunner {

    public static void main(String[] args) throws InterruptedException {
        long now = System.currentTimeMillis();
        CountDownLatch countDownLatch = new CountDownLatch(2);

        new Thread(new SeeDoctorTask(countDownLatch)).start();
        new Thread(new QueueTask(countDownLatch)).start();
        //等待线程池中的2个任务执行完毕，否则一直
        countDownLatch.await();
        System.out.println("over，回家 cost:"+ (System.currentTimeMillis()-now));
    }

}
```



9、CyclicBarrier

栅栏屏障，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。 CyclicBarrier默认的构造方法是CyclicBarrier（int parties），其参数表示屏障拦截的线 程数量，每个线程调用await方法告CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。

```
public class CyclicBarrierRunner implements Runnable {
    private CyclicBarrier cyclicBarrier;
    private int index ;

    public CyclicBarrierRunner(CyclicBarrier cyclicBarrier, int index) {
        this.cyclicBarrier = cyclicBarrier;
        this.index = index;
    }
    public void run() {
        try {
            System.out.println("index: " + index);
            index--;
            cyclicBarrier.await();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args) throws Exception {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(11, new Runnable() {
            public void run() {
                System.out.println("所有特工到达屏障，准备开始执行秘密任务");
            }
        });
        for (int i = 0; i < 10; i++) {
            new Thread(new CyclicBarrierRunner(cyclicBarrier, i)).start();
        }
        cyclicBarrier.await();
        System.out.println("全部到达屏障....");
    }
}
```



10、Atomic

在Atomic包里一共有12个类，四种原子更新方式，分别是原子更新基本类型，原子更新数组，原子更新引用和原子更新字段。Atomic包里的类基本都是使用Unsafe实现的包装类。

**基本类：**AtomicInteger、AtomicLong、AtomicBoolean；

```
public class T1_AtomicInteger {

    public static int total = 0;
    static AtomicInteger atomic = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(10);

        for(int i=0;i<10;i++){
            new Thread(()->{
                for(int j=0;j<1000;j++){
                    /*synchronized () {
                        total++;//cas
                    }*/
                    atomic.getAndIncrement();
                }
                countDownLatch.countDown();
            }).start();
        }

        countDownLatch.await();
        System.out.println(atomic.get());
    }
}
```

atomic.getAndIncrement()原理分析：

```java
/**
 * Atomically increments by one the current value.
 *
 * @return the previous value
 */
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

//Unsafe类
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            //通过对象和该对象的偏移量获取该值
            var5 = this.getIntVolatile(var1, var2);
         	//比较并交换成新的值var5+var4
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
}
```



**引用类型：**AtomicReference、AtomicReference的ABA实例、 AtomicStampedRerence、AtomicMarkableReference； 

**数组类型：**AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray 

```java
public class AtomicIntegerArrayRunner {

    static int[] value = new int[]{1,2};
    //副本拷贝，不会修改原数组
    static AtomicIntegerArray aiArray = new AtomicIntegerArray(value);

    public static void main(String[] args) {
        //todo 原子修改数组下标0的数值
        aiArray.getAndSet(0,3);
        System.out.println(aiArray.get(0));
        //下面输出还是1，因为上面是一个副本的拷贝，不会改远数组
        System.out.println(value[0]);
	}
}

    /**
     * Creates a new AtomicIntegerArray with the same length as, and
     * all elements copied from, the given array.
     *
     * @param array the array to copy elements from
     * @throws NullPointerException if array is null
     */
    public AtomicIntegerArray(int[] array) {
        // Visibility guaranteed by final field guarantees
        this.array = array.clone();
    }


@Data
public class Tuling {

    private Integer sequence;

    public Tuling(Integer seq){
        sequence = seq;
    }
}

public class AtomicReferenceArrayRunner {

    static Tuling[] ovalue = new Tuling[]{new Tuling(1),new Tuling(2)};
    static AtomicReferenceArray<Tuling> objarray = new AtomicReferenceArray(ovalue);

    public static void main(String[] args) {
        System.out.println(objarray.get(0).getSequence());

        objarray.set(0,new Tuling(3));

        System.out.println(objarray.get(0).getSequence());

    }

}
```

**属性原子修改器（Updater）**：AtomicIntegerFieldUpdater、 AtomicLongFieldUpdater、AtomicReferenceFieldUpdater

```java
public class AtomicIntegerFieldUpdateRunner {

    static AtomicIntegerFieldUpdater aifu = AtomicIntegerFieldUpdater.newUpdater(Student.class,"old");

    public static void main(String[] args) {
        Student stu = new Student("杨过",18);
        System.out.println(aifu.getAndIncrement(stu));
        System.out.println(aifu.getAndIncrement(stu));
        System.out.println(aifu.get(stu));
    }

    static class Student{
        private String name;
        public volatile int old;

        public Student(String name ,int old){
            this.name = name;
            this.old = old;
        }

        public String getName() {
            return name;
        }

        public int getOld() {
            return old;
        }
    }

}
```

```
public class AtomicReferenceFieldUpdaterRunner {

    static AtomicReferenceFieldUpdater atomic = AtomicReferenceFieldUpdater.newUpdater(Document.class,String.class,"name");

    public static void main(String[] args) {
        Document document = new Document("杨过",1);

        System.out.println(atomic.get(document));

        atomic.getAndSet(document,"xiaolongnv");

        System.out.println(atomic.get(document));

        //另一种方式修改
        UnaryOperator<String> uo = s->{
            System.out.println("UnaryOperator:-->"+s);
            return "小龙女";
        };
        System.out.println(atomic.getAndUpdate(document, uo));
        System.out.println(atomic.get(document));

    }

    @Data
    static class Document{
        public volatile String name;
        private int version;

        Document(String obj,int v){
            name = obj;
            version = v;
        }

    }
}
```



11、ABA问题

```
@Slf4j
public class AtomicAbaProblemRunner {
    static AtomicInteger atomicInteger = new AtomicInteger(1);

    public static void main(String[] args) {
        Thread main = new Thread(new Runnable() {
            @Override
            public void run() {
                int a = atomicInteger.get();
                log.info("操作线程"+Thread.currentThread().getName()+"--修改前操作数值:"+a);
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                boolean isCasSuccess = atomicInteger.compareAndSet(a,2);
                if(isCasSuccess){
                    log.info("操作线程"+Thread.currentThread().getName()+"--Cas修改后操作数值:"+atomicInteger.get());
                }else{
                    log.info("CAS修改失败");
                }
            }
        },"主线程");

        Thread other = new Thread(new Runnable() {
            @Override
            public void run() {
                atomicInteger.incrementAndGet();// 1+1 = 2;
                log.info("操作线程"+Thread.currentThread().getName()+"--increase后值:"+atomicInteger.get());
                atomicInteger.decrementAndGet();// atomic-1 = 2-1;
                log.info("操作线程"+Thread.currentThread().getName()+"--decrease后值:"+atomicInteger.get());
            }
        },"干扰线程");

        main.start();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        other.start();

    }
}
```

解决ABA问题：通过版本号

```java
public class AtomicStampedRerenceRunner {

    private static AtomicStampedReference<Integer> atomicStampedRef =
            new AtomicStampedReference<>(1, 0);

    public static void main(String[] args){
        Thread main = new Thread(() -> {
            int stamp = atomicStampedRef.getStamp(); //获取当前标识别
            System.out.println("操作线程" + Thread.currentThread()+ "stamp="+stamp + ",初始值 a = " + atomicStampedRef.getReference());
            try {
                Thread.sleep(3000); //等待1秒 ，以便让干扰线程执行
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean isCASSuccess = atomicStampedRef.compareAndSet(1,2,stamp,stamp +1);  //此时expectedReference未发生改变，但是stamp已经被修改了,所以CAS失败
            System.out.println("操作线程" + Thread.currentThread() + "stamp="+stamp + ",CAS操作结果: " + isCASSuccess);
        },"主操作线程");

        Thread other = new Thread(() -> {
            int stamp = atomicStampedRef.getStamp();
            atomicStampedRef.compareAndSet(1,2,stamp,stamp+1);
            System.out.println("操作线程" + Thread.currentThread() + "stamp="+atomicStampedRef.getStamp() +",【increment】 ,值 a= "+ atomicStampedRef.getReference());
            stamp = atomicStampedRef.getStamp();
            atomicStampedRef.compareAndSet(2,1,stamp,stamp+1);
            System.out.println("操作线程" + Thread.currentThread() + "stamp="+atomicStampedRef.getStamp() +",【decrement】 ,值 a= "+ atomicStampedRef.getReference());
        },"干扰线程");

        main.start();
        LockSupport.parkNanos(1000000);
        other.start();
    }
}
```



12、Unsafe



13、HashMap

红黑树的特征：

1）、每个节点要么是黑色，要么是红色

2）、根节点是黑色

3）、**每个叶子节点都是黑色的空节点（NIL节点）**

4）、如果一个节点是红色的，则它的子节点必须是黑色的

5）、从任一节点到其每个叶子节点的所有路径都包含相同数目的黑色节点

6）、新插入节点默认为红色，插入后需要校验红黑树是否符合规则，不符合则需要进行平衡

二叉树的访问次序：前序遍历、中序遍历、后序遍历、层序遍历





