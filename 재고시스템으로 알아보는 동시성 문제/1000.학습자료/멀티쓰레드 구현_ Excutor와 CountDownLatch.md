#CS 



# Executor

Executor는 자바에서 비동기처리를 하기 위한 인터페이스이다. 멀티쓰레드를 구성할 때 직접 쓰레드를 생성하고, 종료하는 등의 복잡한 로직들을 대신 해주며 ThreadPool을 관리해줌으로써 조금 더 편하고 안정적으로 비동기 작업을 처리할 수 있도록 도와준다.  
  

## 1. Executor

```java
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```

`Executor`는 Runnable task를 실행하는 함수를 가지고 있는 인터페이스이다. 문서에 있는 내용을 요약해보면

> _작업을 제출하는 것과 작업이 실행되는 방식(쓰레드 사용, 스케줄링 등)을 분리하는 방법을 제공한다.  
> 일반적으로 쓰레드를 명시적으로 생성하는 대신 사용된다.  
> 일반적으로 작업은 호출자의 쓰레드와 다른 쓰레드에서 실행된다._

한 줄로 요약을 하자면  
`Executor` 사용을 통해 어떤 작업을 실행할 때 쓰레드를 만들고 스케줄링을 설정하는 것을 매번하지 않고, 내부에서 쓰레드를 생성 및 관리하게 하여 작업을 실행에서 분리한다.  
이를 통해 작업자가 어떻게 실행되는 것인지에 대한 관심을 줄이고 작업(Task)에 집중할 수 있게 해준다.

  
  

## 2. ExecutorService

```java
/**
 * An executor that provides methods to manage termination and
 * methods that can produce a Future for tracking progress of
 * one or more asynchronous tasks.
 *
 * An ExecutorService can be shut down, which will cause
 * it to reject new tasks. Two different methods are provided for
 * shutting down an ExecutorService. The shutdown
 * method will allow previously submitted tasks to execute before
 * terminating, while the shutdownNow method prevents waiting
 * tasks from starting and attempts to stop currently executing tasks.
 * Upon termination, an executor has no tasks actively executing, no
 * tasks awaiting execution, and no new tasks can be submitted.  An
 * unused ExecutorService should be shut down to allow
 * reclamation of its resources.
 */
 
public interface ExecutorService extends Executor {

    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    
    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

`ExecutorService`는 `Executor`에서 더 확장된 기능을 제공하는 인터페이스이다.  
종료를 관리하는 함수와 하나 또는 여러개의 비동기 작업들의 진행도를 트래킹하는 `Future`를 생성하는 함수를 제공한다.  

## 3. Future

```java
/**
 * A Future represents the result of an asynchronous
 * computation. Methods are provided to check if the computation is
 * complete, to wait for its completion, and to retrieve the result of
 * the computation. The result can only be retrieved using method
 * get when the computation has completed, blocking if
 * necessary until it is ready. Cancellation is performed by the
 * cancel method. Additional methods are provided to
 * determine if the task completed normally or was cancelled. Once a
 * computation has completed, the computation cannot be cancelled.
 * If you would like to use a Future for the sake
 * of cancellability but not provide a usable result, you can
 * declare types of the form Future<?> and
 * return null as a result of the underlying task.
 */
 
public interface Future<V> {

    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

앞서 살펴본 `ExecutorService`의 submit() 함수의 결과로 반환되는 `Future`는 비동기 연산의 결과를 나타내는 인터페이스다.  
결과는 연산이 완료되면 get() 함수를 통해서 가져올 수 있고, 연산이 끝나지 않았으면 blocking된다.  
이 과정에서 작업이 완료되기 전까지 다른 작업을 수행할 수 있다.

  
  

## 4. ThreadPoolExecutor

`ThreadPoolExecutor`는 `ExecutorService` 구현체의 하나로 Thread pool에서 사용 가능한 Thread를 사용해 각각의 작업을 실행한다. 대부분 직접 사용하기보다는 `Executors` 라는 클래스를 통해 사용하게 된다.

먼저 생성자를 통해 어떠한 값들이 쓰이게 되는지 확인해보자.

```java
	public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

`corePoolSize` : Thread pool에서 유지하는 최소 thread 개수  
`maximumPoolSize` : Thread pool의 최대 thread 개수  
`keepAliveTime` : Thread의 개수가 corePoolSize보다 큰 경우 이 시간만큼 idle 상태로 대기하다가 사용되지 않으면 제거된다.  
`unit` : keepAliveTime의 시간 단위  
`workQueue` : 작업들이 실행되기 전에 유지되는 queue  
`threadFactory` : executor가 새로운 쓰레드를 만들 때 사용하는 factory  
`handler` : thread 개수와 queue개수가 제한된 용량을 넘었을 경우 실행되는 핸들러이다.

  

```java
	private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
	private static final intCOUNT_BITS= Integer.SIZE- 3;
	private static final intCOUNT_MASK= (1 <<COUNT_BITS) - 1;

	// runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;

    // Packing and unpacking ctl
    private static int runStateOf(int c)     { return c & ~COUNT_MASK; }
    private static int workerCountOf(int c)  { return c & COUNT_MASK; }
    private static int ctlOf(int rs, int wc) { return rs | wc; }
```

`ThreadPoolExecutor`는 `AtomicInteger`를 사용하여 bit 연산을 통해 `runState`와 `workerCount`를 packing하여 관리한다. Integer 32 bit 중에서 최상위 3개의 bit로 `runState`를 나타내며 나머지 29 bit로 `worker`의 개수를 표현한다.

  

```java
	public void execute(Runnable command) {
	    if (command == null)
	        throw new NullPointerException();
	    int c = ctl.get();
	    if (workerCountOf(c) < corePoolSize) {
	        if (addWorker(command, true))
	            return;
	        c = ctl.get();
	    }
	    if (isRunning(c) && workQueue.offer(command)) {
	        int recheck = ctl.get();
	        if (!isRunning(recheck) && remove(command))
	            reject(command);
	        else if (workerCountOf(recheck) == 0)
	            addWorker(null, false);
	    }
	    else if (!addWorker(command, false))
	        reject(command);
	}
```

가장 중요한 `execute()`함수이다. Runnable command를 받아 thread에게 일을 실행하는 역할을 한다.  
함수는 크게 3가지 부분으로 나눌 수 있다.

**첫번째**로 위에서 살펴본 AtomicInteger를 통해 worker의 개수를 확인한 다음 core pool size보다 작은 경우 worker를 추가함과 동시에 commnad를 실행한다.

**두번째**로 runState를 확인한다음 workQueue에 command를 밀어넣는다.  
그리고 executor의 runState를 다시 한 번 확인을 하고 running상태이면 worker를 추가하고, running 상태가 아니면 queue에서 command를 빼내고 reject(command)를 실행한다. 이 함수는 처음에 보았던 handler를 호출한다.

**마지막**으로 task를 queue에 넣을 수 없는 경우 새로운 쓰레드를 생성하려고 시도하고 실패할 경우는 reject(command)를 실행한다.

# CountDownLatch 

멀티쓰레드 프로그래밍에서 쓰레드들이 모든 작업을 마친 후에 특정한 작업을 해야하는 경우가 있다. 이를 위해 다른 쓰레드들에서 일련의 작업이 완료 될 때까지 대기하도록 Sync를 맞춰주는 기능을 자바는 java.util.concurrent.CountDownLatch(int count)을 제공한다.

CountDownLatch를 인스턴스화할 때 주어진 카운트로 초기화된다. await 메서드를 실행하면 해당 쓰레드는 다른 쓰레드에서 countDown 메서드 호출로 인해 현재 카운트가 0이 될 때까지 대기한다. 그 후에는 대기중인 스레드가 해제된다.

1. Main 쓰레드는 latch.await() 메소드에 의해 대기상태로 진입한다.
2. 이 후 Worker 쓰레드들이 자신의 작업을 마친 후, latch.countDown() 메소드를 통해 카운트를 줄여간다.
3. 카운트가 0이 되면 Main 쓰레드를 대기상태에서 해제된다.

```java
public class CountDownLatchExample {
	static final int max = 3;

	public static void testCountDownLatch() throws Exception {
		final CountDownLatch latch = new CountDownLatch(max);

		for (long i = 1; i <= max; i++) {
			new Thread(new Worker(latch, i * 100)).start();
		}

		latch.await();

		System.out.println("########### CountDownLatch End ###########");
	}

	static class Worker implements Runnable {
		private CountDownLatch latch;
		private long n;

		public Worker(CountDownLatch latch, long n) {
			this.latch = latch;
			this.n = n;
		}

		@Override
		public void run() {
			try {
				int cnt = 0;
				for (int i = 0; i < n; i++) {
					cnt++;
				}

				System.out.println(cnt);
			} catch (Exception ex) {
				ex.printStackTrace();
			} finally {
				if (this.latch == null) {
					return;
				}

				latch.countDown();
			}
		}
	}

	public static void main(String[] args) throws Exception {
		System.out.println("start");
		testCountDownLatch();
		System.out.println("end");
	}

	/**
	 * start
	 * 200
	 * 100
	 * 300
	 * ########### CountDownLatch End ###########
	 * end
	 */
}
```

