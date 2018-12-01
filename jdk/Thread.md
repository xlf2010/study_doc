本文讨论jvm线程在特定的操作系统Linux的实现方式，本地实现代码为openjdk8。


// Class hierarchy
// - Thread
//   - NamedThread
//     - VMThread
//     - ConcurrentGCThread
//     - WorkerThread
//       - GangWorker
//       - GCTaskThread
//   - JavaThread
//   - WatcherThread


##线程状态说明
线程标志位枚举，int类型标识，openjdk8\jdk\src\share\javavm\export\jvmti.h
```C++
enum {
	//线程存活状态
    JVMTI_THREAD_STATE_ALIVE = 0x0001,

	//线程终结状态
    JVMTI_THREAD_STATE_TERMINATED = 0x0002,

	//线程可运行状态
    JVMTI_THREAD_STATE_RUNNABLE = 0x0004,

	//线程等待获取锁状态，阻塞在monitorenter指令中，即synchronized语句
    JVMTI_THREAD_STATE_BLOCKED_ON_MONITOR_ENTER = 0x0400,

	//线程等待状态，可能调用object.wait或Unsafe.park使线程阻塞
    JVMTI_THREAD_STATE_WAITING = 0x0080,

	//无限期等待，没有设置超时，可能调用不带参数的object.wait()或Unsafe.park()使线程阻塞
    JVMTI_THREAD_STATE_WAITING_INDEFINITELY = 0x0010,

	//带超时的等待，超时后返回，可能调用带参数的object.wait(long)或Unsafe.park(long)使线程阻塞
    JVMTI_THREAD_STATE_WAITING_WITH_TIMEOUT = 0x0020,

	//线程睡眠状态，一般调用Thread.sleep(long);
    JVMTI_THREAD_STATE_SLEEPING = 0x0040,

	//对象等待状态，一般调用Object.wait方法
    JVMTI_THREAD_STATE_IN_OBJECT_WAIT = 0x0100,

	//线程阻塞状态，一般调用Unsafe.park方法
    JVMTI_THREAD_STATE_PARKED = 0x0200,

	//线程暂停，Thread.suspend()调用或以其他方式调用了该对象的suspend方法
    JVMTI_THREAD_STATE_SUSPENDED = 0x100000,
	
	//线程中断标志，Thread.interrupt()调用
    JVMTI_THREAD_STATE_INTERRUPTED = 0x200000,

	//JNI本地方法被调用标志，当执行Java字节码时这标识不会被设置，执行VM代码，JNI代码是会被设置
    JVMTI_THREAD_STATE_IN_NATIVE = 0x400000,
	
	//预留，由JVM实现者定义使用
    JVMTI_THREAD_STATE_VENDOR_1 = 0x10000000,
    JVMTI_THREAD_STATE_VENDOR_2 = 0x20000000,
    JVMTI_THREAD_STATE_VENDOR_3 = 0x40000000
};
```

park与wait区别：park不需任何前置条件即可让线程进入等待状态；wait需要获得锁才能执行，并执行成功后会释放所得的锁，其他线程可竞争锁，唤醒时需再获取相应的锁。

线程状态组合枚举值位于 openjdk8\hotspot\src\share\vm\classfile\javaClasses.hpp
这个值可以Java代码调用 Thread.getState() 获取。

```C++
// Java Thread Status for JVMTI and M&M use.
  // This thread status info is saved in threadStatus field of
  // java.lang.Thread java class.
  enum ThreadStatus {
    NEW                      = 0,

	//可运行状态ALIVE+RUNNABLE
    RUNNABLE                 = JVMTI_THREAD_STATE_ALIVE +
                               JVMTI_THREAD_STATE_RUNNABLE,

    //睡眠状态Thread.sleep()，ALIVE+WAITING+WAITING_WITH_TIMEOUT
	SLEEPING                 = JVMTI_THREAD_STATE_ALIVE +          
                               JVMTI_THREAD_STATE_WAITING +
                               JVMTI_THREAD_STATE_WAITING_WITH_TIMEOUT +
                               JVMTI_THREAD_STATE_SLEEPING,

	//调用Object.wait()等待状态，ALIVE+WAITING+WAITING_INDEFINITELY+IN_OBJECT_WAIT
    IN_OBJECT_WAIT           = JVMTI_THREAD_STATE_ALIVE +          
                               JVMTI_THREAD_STATE_WAITING +
                               JVMTI_THREAD_STATE_WAITING_INDEFINITELY +
                               JVMTI_THREAD_STATE_IN_OBJECT_WAIT,

	// 调用Object.wait(long)等待，ALIVE+WAITING+WAITING_WITH_TIMEOUT+IN_OBJECT_WAIT
    IN_OBJECT_WAIT_TIMED     = JVMTI_THREAD_STATE_ALIVE +          
                               JVMTI_THREAD_STATE_WAITING +
                               JVMTI_THREAD_STATE_WAITING_WITH_TIMEOUT +
                               JVMTI_THREAD_STATE_IN_OBJECT_WAIT,

	// 不带超时线程阻塞		ALIVE+WAITING+WAITING_INDEFINITELY+PARKED
    PARKED                   = JVMTI_THREAD_STATE_ALIVE +          
                               JVMTI_THREAD_STATE_WAITING +
                               JVMTI_THREAD_STATE_WAITING_INDEFINITELY +
                               JVMTI_THREAD_STATE_PARKED,

	// 带超时线程阻塞		ALIVE+WAITING+WAITING_WITH_TIMEOUT+PARKED
    PARKED_TIMED             = JVMTI_THREAD_STATE_ALIVE +
                               JVMTI_THREAD_STATE_WAITING +
                               JVMTI_THREAD_STATE_WAITING_WITH_TIMEOUT +
                               JVMTI_THREAD_STATE_PARKED,
	
	//线程阻塞在重入synchronized块，等待其他线程退出
    BLOCKED_ON_MONITOR_ENTER = JVMTI_THREAD_STATE_ALIVE +          
                               JVMTI_THREAD_STATE_BLOCKED_ON_MONITOR_ENTER,

	//线程终结
    TERMINATED               = JVMTI_THREAD_STATE_TERMINATED
  };
```

##java.lang.Thread.start();
当一个线程对象Thread的start方法被调用时，先将线程加入到当前线程组，随后执行一个本地方法 

private native void start0();

该方法对应的实际执行C++代码在
openjdk8\hotspot\src\share\vm\prims\jvm.cpp

```C++
//JVM_ENTRY为一个宏，用来定义一个JVM_StartThread函数，宏的两个参数一个为返回值，一个为函数名，在预处理时会展开，JNIEnv是JNI的执行环境指针，jthread当前调用的start方法的对象。
JVM_ENTRY(void, JVM_StartThread(JNIEnv* env, jobject jthread))
  JVMWrapper("JVM_StartThread");
  JavaThread *native_thread = NULL;

  bool throw_illegal_thread_state = false;

  {
    //构造一个互斥锁，避免并发调用重复启动线程，释放锁在MutexLocker的析构函数中，函数结束后会释放锁
    MutexLocker mu(Threads_lock);

	//判断线程是否创建成功，创建成功则抛出illegal_thread_state
	//JavaThread创建成功与更新状态threadStatus会有一小段时间差，也就是Java代码start方法判断：
	//if (threadStatus != 0)
    //        throw new IllegalThreadStateException();
	//此判断不一定能准确，因此最好判断线程对象有没有重复创建
    if (java_lang_Thread::thread(JNIHandles::resolve_non_null(jthread)) != NULL) {
      throw_illegal_thread_state = true;
    } else {
		//获取jthread对象中栈大小
      jlong size =java_lang_Thread::stackSize(JNIHandles::resolve_non_null(jthread));

	  //size 在java是带符号的整数，但size_t是无符号
      size_t sz = size > 0 ? (size_t) size : 0;
	
	  //创建一个本地Java线程，在Linux平台下使用pthread创建,thread_entry见后面解释
      native_thread = new JavaThread(&thread_entry, sz);

      // At this point it may be possible that no osthread was created for the
      // JavaThread due to lack of memory. Check for this situation and throw
      // an exception if necessary. Eventually we may want to change this so
      // that we only grab the lock if the thread was created successfully -
      // then we can also do this check and throw the exception in the
      // JavaThread constructor.
      if (native_thread->osthread() != NULL) {
        // Note: the current thread is not being used within "prepare".
        native_thread->prepare(jthread);
      }
    }
  }

  if (throw_illegal_thread_state) {
    THROW(vmSymbols::java_lang_IllegalThreadStateException());
  }

  assert(native_thread != NULL, "Starting null thread?");

  if (native_thread->osthread() == NULL) {
    // No one should hold a reference to the 'native_thread'.
    delete native_thread;
    if (JvmtiExport::should_post_resource_exhausted()) {
      JvmtiExport::post_resource_exhausted(
        JVMTI_RESOURCE_EXHAUSTED_OOM_ERROR | JVMTI_RESOURCE_EXHAUSTED_THREADS,
        "unable to create new native thread");
    }
    THROW_MSG(vmSymbols::java_lang_OutOfMemoryError(),
              "unable to create new native thread");
  }

  Thread::start(native_thread);

JVM_END
```
thread_entry定义如下
```C++
#define THREAD __the_thread__
#define TRAPS  Thread* THREAD
//TRAPS为宏定义，指向Thread指针，最终会展开成Thread* __the_thread__
//THREAD展开成__the_thread__
static void thread_entry(JavaThread* thread, TRAPS) {
  HandleMark hm(THREAD);
  Handle obj(THREAD, thread->threadObj());
  JavaValue result(T_VOID);
  JavaCalls::call_virtual(&result,
                          obj,
                          KlassHandle(THREAD, SystemDictionary::Thread_klass()),
                          vmSymbols::run_method_name(),
                          vmSymbols::void_method_signature(),
                          THREAD);
}
```



##java.lang.Thread.stop();
jdk一个线程A调用另一个线程B的stop方法是通过发送ThreadDeath给线程B，ThreadDeath extends Error，如果线程B没有catch到该Error，则线程B会被终止掉，线程B持有的锁会被释放掉，线程B在锁的临界资源可能导致不一致问题，因此jdk不建议使用stop来终止一个线程。
但并不是线程B收到ThreadDeath后会立刻终止。有两种情况并不会终止线程B：1.线程B catch了ThreadDeath后消化了这个Error，线程不会被终止；2.线程B在执行一些阻塞IO操作，比如说socket.accept,read或write操作，等待结果返回后才会抛ThreadDeath Error。