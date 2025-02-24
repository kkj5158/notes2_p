Java는 크게 세 가지의 메모리 영역을 가지고 있습니다.

- static 영역
- heap 영역
- stack 영역

**자바 멀티 스레드** 환경에서는 스레드끼리 static 영역과 heap 영역을 공유하므로 "<span style="background:#fff88f">공유자원에 대한 동기화 문제</span>"를 신경써야 합니다. 이때 "원자성 문제"를 해결하기 위한 방법 중 하나인, 자바에서 제공하는 키워드 **synchronized** 를 사용하게 됩니다.

추가적으로 **스레드 동기화**는 멀티 스레드 환경에서 여러 스레드가 하나의 공유자원에 동시에 접근하지 못하도록 막는 것을 말합니다. 공유데이터가 사용되어 동기화가 필요한 부분을 **임계영역**이라고 부르며 자바에서는 <span style="background:#fff88f">이 임계 영억에 synchronized 키워드**를 사용하여 여러 스레드가 동시에 접근하는 것을 금지합니다.</span>

동기화가 필요한 "메소드"나 "코드블럭" 앞에 synchronized 키워드를 사용하여 동기화 할 수 있는데, synchronized로 지정된 임계영역은 한 스레드가 이 영역에 접근하여 사용할때 lock이 걸리게 되어 **다른 스레드가 접근할 수 없게 됩니다.** 이후 해당 스레드가 이 임계영역의 코드를 다 실행한 후 벗어나게 되면 unlock 상태가되고 -> 대기하던 다른 스레드가 이 임계영역에 접근하여 다시 lock을 걸고 사용할 수 있게 되는 것입니다.

lock은 해당 객체당 하나씩 존재하며, synchronized로 설정된 임계영역은 lock 권한을 얻은 하나의 객체만이 독점적으로 사용하게 됩니다.  
  

synchronized는 lock을 이용해 동기화를 수행하며 네 가지의 사용 방법이 존재합니다

- synchronized method
- static synchronized method
- synchronized block
- static synchronized block

---

### - Synchronized method

메소드 이름 앞에 synchronized 키워들를를 사용하면 **해당 메소드 전체를 임계영역으로 설정** 할 수 있습니다.

```java
package CNS.synchronized_test;

import java.time.LocalDateTime;
import java.util.concurrent.TimeUnit;

public class Method {
    public static void main(String[] args) {
        Method sync = new Method();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            sync.syncMethod1("스레드");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 "+ LocalDateTime.now());
            sync.syncMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });

        thread1.start();
        thread2.start();

    }

    private synchronized void syncMethod1(String msg){
        System.out.println(msg + "의 syncMethod1 실행 중 "+LocalDateTime.now());
        try{
            TimeUnit.SECONDS.sleep(5);
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }

    private synchronized void syncMethod2(String msg){
        System.out.println(msg +"의 syncMethod2 실행 중 "+LocalDateTime.now());
        try{
            TimeUnit.SECONDS.sleep(5);
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }
}
```

이에 관한 출력은 아래와 같습니다.

![](https://velog.velcdn.com/images/alswn9938/post/9cd82120-ea4d-41a5-b121-f638c7182500/image.png)

인스턴스 하나(Method)를 생성하고 두 개의 스레드를 만들어 호출하니까 "스레드1이 synchMethod1()을 호출-> 종료한 후 다음 스레드가 syncMethod2()를 호출"하는 것을 확인할 수 있습니다.

### - static synchronized method

static 키워드가 포함된 synchronized 메소드는 인스턴스가 아닌 "클래스" 단위로 lock을 공유합니다.

```java
 public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            syncStaticMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            syncStaticMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });

        thread1.start();
        thread2.start();
    }

    public static synchronized void syncStaticMethod1(String msg) {
        System.out.println(msg + "의 syncStaticMethod1 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static synchronized void syncStaticMethod2(String msg) {
        System.out.println(msg + "의 syncStaticMethod2 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

실행 결과는 다음과 같습니다. 동기화가 잘 지켜지고 있음을 확인할 수 있습니다.  
![](https://velog.velcdn.com/images/alswn9938/post/836072cf-d173-478d-8500-b55795591f0b/image.png)

하지만 아래의 예제와 같은 경우의 고려가 필요합니다.

```java
public class StaticMethod {
    public static void main(String[] args) {
        StaticMethod staticMethod = new StaticMethod();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            syncStaticMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            syncStaticMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });

        Thread thread3 = new Thread(() -> {
            System.out.println("스레드3 시작 " + LocalDateTime.now());
            staticMethod.syncMethod3("스레드3");
            System.out.println("스레드3 종료 " + LocalDateTime.now());
        });

        thread1.start();
        thread2.start();
        thread3.start();
    }

    public static synchronized void syncStaticMethod1(String msg) {
        System.out.println(msg + "의 syncStaticMethod1 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static synchronized void syncStaticMethod2(String msg) {
        System.out.println(msg + "의 syncStaticMethod2 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private synchronized void syncMethod3(String msg) {
        System.out.println(msg + "의 syncMethod3 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

위 코드의 실행결과는 다음과 같습니다. syncMethod3의 동기화가 잘지켜지고 있지 않음을 확인할 수 있습니다.  
![](https://velog.velcdn.com/images/alswn9938/post/ee5de18a-5a15-44fe-b748-041fb1f18e73/image.png)

그 이유는<span style="background:#fff88f"> 클래스 단위에 거는 lock과 인스턴스 단위에 거는 lock은 공유가 되지 않기 때문에 혼용에서 쓴다면 동기화 이슈가 발생하게 되기 때문입니다.</span>

### - synchronized block

인스턴스의 block 단위로 lock을 거는 것입니다. 두 가지의 사용 방법이 있는데 다음과 같습니다.

- synchronized(this)
- synchronized(Object)

**synchronized(this)**

```java
package CNS.synchronized_test;

import java.time.LocalDateTime;
import java.util.concurrent.TimeUnit;

public class Block {
    public static void main(String[] args) {
        Block block = new Block();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            block.syncBlockMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            block.syncBlockMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });
        thread1.start();
        thread2.start();
    }

    private void syncBlockMethod1(String msg) {
        synchronized (this) {
            System.out.println(msg + "의 syncBlockMethod1 실행중" + LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void syncBlockMethod2(String msg) {
        synchronized (this) {
            System.out.println(msg + "의 syncBlockMethod2 실행중" + LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

이와 같이 synchronized 인자값으로 this를 사용하면 synchronized block에 lock이 걸립니다. 여러 스레드가 들어와 각기 다른 synchronized block을 호출해도 같은 객체(this)의 모든 부분에 lock이 걸리기 때문에 동시처리 되지 않고 기다려야합니다.  
쉽게 말해 synchronized(this)의 경우에, 해당 객체 안에 있는 모든 synchronized block에 lock이 걸리는 것입니다.  
![](https://velog.velcdn.com/images/alswn9938/post/c40640ce-947b-4344-9b48-672bf9ca7a1e/image.png)

**synchronized(Object)**  
해당 방식을 사용하면 블록마다 다른 lock이 걸리게 하여 효율적으로 사용할 수 있습니다.

```java
package CNS.synchronized_test;

import java.time.LocalDateTime;
import java.util.concurrent.TimeUnit;

public class Block2 {
    private final Object o1 = new Object();
    private final Object o2 = new Object();

    public static void main(String[] args) {
        Block2 block = new Block2();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            block.syncBlockMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            block.syncBlockMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });
        thread1.start();
        thread2.start();
    }


    private void syncBlockMethod1(String msg) {
        synchronized (o1) {
            System.out.println(msg + "의 syncBlockMethod1 실행중" + LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void syncBlockMethod2(String msg) {
        synchronized (o2) {
            System.out.println(msg + "의 syncBlockMethod2 실행중" + LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

![](https://velog.velcdn.com/images/alswn9938/post/490958f6-8ece-4958-935f-3e3db375dc5b/image.png)

동기화가 지켜지지 않은 것을 확인할 수 있습니다. 이와 같이 지정을 해준다면 동시에 lock이 걸려야하는 부분을 따로 지정해줄 수 있다는 것이 확인됩니다.

### - static synchronized block

static method 안에 synchronized block을 지정할 수 있습니다. static의 특성상 this 같이 현재 객체를 가리키는 표현을 사용할 수 없습니다. static synchronized method 방식과의 차이는 lock 객체를 지정하고, block으로 범위를 한정지을 수 있다는 것입니다. (클래스 단위로 lock을 공유한다는 점은 동일합니다.)
