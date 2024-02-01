# 11장. 동시성

## 아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라

- `synchronized` 키워드는 해당 메서드나 블록을 한 번에 한 스레드씩 수행하도록 보장한다.
- 동기화를 제대로 사용하면 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막을 수 있고, 한 스레드가 만든 변화를 다른 스레드에서 확인할 수 있게 해준다.
- 잘못된 코드 → 동기화를 하지 않아 메인 스레드가 수정한 값은 백그라운드 스레드가 언제 보게될 지 보증할 수 없어 영원히 수행된다.

```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

- synchronized 키워드를 사용하여 동기화를 해보자

```java
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

- 쓰기와 읽기 모두가 동기화되지 않으면 동작을 보장하지 않으므로 주의하자
- 또한 `stopRequested` 필드를 volatile로 선언하면 동기화를 생략해도 된다. 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다. 다만, 가변 데이터의 배타적 수행을 보장해야하는 경우에는 사용하지 못한다.
- 혹은 java.util.concurrent.atomic 패키지에는 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다. 이 패키지에서 `AtomicLong`은 동기화 효과 중 통신 쪽만 지원하는 volatile과 다르게 원자성(배타적 실행)까지 보장한다.
- 사실 제일 좋은 방법은 불변 데이터만  공유하도록 하고 가변 데이터는 단일 스레드에서만 쓰도록 하는 것이다.

## 아이템 80.  스레드보다는 실행자, 태스크, 스트림을 애용하라

- 실행자 프레임워크: Executor Framework
    
    Executor Framework는 Java 5에 도입된 java.util.concurrent 패키지의 일부이다. **다중 스레드 환경에서 작업을 비동기적으로 관리하고 실행하는 방법**을 제공한다.
    
- `ThreadPoolExecutor`: 스레드 풀 동작을 결정하는 거의 모든 속성을 설정할 수 있다.
- `CachedThreadPool`: 가벼운 서버를 동작한다면, 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임되는 CachedThreadPool을 사용하는 것도 괜찮다.
- 실행자 프레임워크는 또한 포크-조인 태스크를 지원한다.
- 포크-조인 태스크는 포크-조인 풀이라는 특별한 실행자 서비스가 실행해준다. 포크-조인 태스크의 인스턴스는 작은 하위 태스크로 나뉠 수 있고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드는 남은 태스크를 가져와 대신 처리 가능하여 높은 처리량을 달성한다.

## 아이템 81. wait와 notify보다는 동시성 유틸리티를 애용하라

- java.util.concurrent는 실행자 프레임워크, 동시성 컬렉션(concurrent collection), 동기화 장치(synchronizer)로 나눌 수 있다.
- 동시성 컬렉션은 List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션으로, 이전에는 wait와 notify로 하드코딩했던 일들을 동시성 유틸리티가 쉽게 처리해준다.

```java
public static String intern(String s) {
        String result = map.get(s);
        if (result == null) {
            result = map.putIfAbsent(s, s);
            if (result == null)
                result = s;
        }
        return result;
}
```

- 스레드가 다른 스레드를 기다릴 수 있게하는 동기화 장치로는 CountDownLatch와 Semaphore, CyclicBarrier와 Exchanger, Phaser가 있다.
- CountDownLatch는 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
    
    아래는 향수 프로젝트에서 동시성 테스트를 위해 작성했던 코드이다.
    

```java
 		@DisplayName("동시에 같은 닉네임으로 변경 요청")
    @Test
    void updateNickname() throws InterruptedException {
        //given
        // 동시성 테스트 준비
        int numberOfThreads = 2;

        ExecutorService service = Executors.newFixedThreadPool(numberOfThreads);
        CountDownLatch latch = new CountDownLatch(numberOfThreads);

        //when
        service.execute(() -> {
            userService.synchronizedUpdateNickname("same", 1L);
            latch.countDown();
        });
        service.execute(() -> {
            try {
                userService.synchronizedUpdateNickname("same", 2L);
            } catch (CustomException e) {

            } finally {
                latch.countDown();
            }
        });

        latch.await(); // 모든 쓰레드의 작업이 완료될 때까지 대기

        //then
        User user1 = userRepository.findById(1L).get();
        User user2 = userRepository.findById(2L).get();
        System.out.println("user1 = " + user1.getNickname());
        System.out.println("user2 = " + user2.getNickname());
        assertThat(user1.getNickname()).isNotEqualTo(user2.getNickname());

    }
```

- CountDownLatch의 유일한 생성자는 int 값을 받으며, 이 값이 래치의 countDown 메서드를 몇 번 호출해야 대기 중인 스레드들을 깨우는지 결정한다.
- 동기화 장치를 통해 동시 실행 시간을 재는 간단한 코드를 아래와 같이 구현할 수 있다.

```java
public class ConcurrentTimer {
    private ConcurrentTimer() { } // 인스턴스 생성 불가

    public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done  = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                ready.countDown(); // 타이머에게 준비를 마쳤음을 알린다.
                try {
                    start.await(); // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();  // 타이머에게 작업을 마쳤음을 알린다.
                }
            });
        }

        ready.await();     // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await();      // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
}
```

1. concurrency만큼의 스레드들이 ready.countDown();을 차례로 호출하고 start.await(); 를 호출하여 start 래치가 열리길 기다린다.
2. ready 래치가 concurrency만큼 countDown이 되면 await 상태였던 ready 래치가 풀리고 start.countDown(); 하여 작업자들을 깨운다.
3. 작업자들은 action.run();을 실행하고 done.countDown();을 호출하여 마지막 스레드가 호출했을 때 done 래치를 깨운다.
4. 타이머 스레드는 done 래치가 열리자마자 종료시각을 기록한다.
- 어쩔 수 없이 레거시 코드를 다루는 경우를 위해, 간단히 기록하자면 `wait` 메서드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다. `notify` 메서드는 대기 중이던 스레드를 깨우는 데 사용된다.