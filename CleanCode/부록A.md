# 부록 A 동시성II

## 단일 스레드 시스템을 다중 스레드 시스템으로 변환하기

클라이언트/ 서버 예제

```java
// 서버 애플리케이션을 단순화한 버전
ServerSocket serverSocket = new ServerSocket(8009);

while(keepProcessing){
	try{
		Socket socket = serverSocket.accept();
		process(socket);
	} catch(Exception e){
		handle(e);
	}
}
```

```java
// 클라이언트 코드
private void connectSendReceive(int i){
	try{
		Socket socket = new Socket("localhost", PORT);
		MessageUtils.sendMessage(socket, Integer.toString(i));
		MessageUtils.sendMessage(socket);
		socket.close();
	} catch(Exception e){
		e.printStackTrace();
	}
}
```

만약 해당 테스트가 10초 내에 처리가 되는지를 확인했을 때 실패한다면? 애플리케이션이 어디서 시간을 보내는지 확인해보자.

**애플리케이션이 시간을 보낼 가능성**

- I/O - 소켓 사용, 데이터베이스 연결, 가상 메모리 스와핑 기다리기 등등
- 프로세서 - 수치 계산, 정규 표현식 처리, 가비지 컬렉션 등등

만약 프로그램이 주로 **프로세서 연산**에 시간을 보낸다면, **하드웨어를 추가**해 동시에 더 많은 프로세스를 실행할 수 있도록 해야한다. 하지만 주로 **I/O 연산**에 시간을 보낸다면, 다중 스레드를 이용해 CPU를 효과적으로 활용할 수 있다.

→ 일단 서버의 process 함수가 I/O 연산에 시간을 주로 보낸다 가정하고 스레드를 추가해보자.

```java
void process(final Socket socket) {
    if (socket == null) {
        return;
    }
    
    Runnable clientHandler = new Runnable() {
        public void run() {
            try {
                String message = MessageUtils.getMessage(socket);
                MessageUtils.sendMessage(socket, "Processed: " + message);
                closeIgnoringException(socket);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    };

    Thread clientConnection = new Thread(clientHandler);
    clientConnection.start();
}
```

`clientHandler` 라는 Runnable 객체를 생성하고, 이를 이용하여 새로운 스레드를 생성하고 시작하는 로직이다. 이렇게 고치니 테스트를 통과한다 치자. 코드가 부실하지 않고 깨끗하다고 할 수 있는가?

*JVM이 허용하는 수까지만 스레드 생성이 가능할 뿐만 아니라 process 함수가 지는 책임이 너무 많다.*

→ SRP를 고려하여 스레드 관리 책임을 지는 클래스를 분리하자.

```java
public interface ClientScheduler {
   void schedule(ClientRequestProcessor requestProcessor);
}
public class ThreadPerRequestScheduler implements ClientScheduler {
   public void schedule(final ClientRequestProecessor requestprocessor) {
      Runnablel runnable = new Runnable() {
         public void run() {
            requestProcessor.process();
         }
      };
      
      Thread thread = new Thread(runnable);
      thread.start();
   }
}
```

스레드 관리를 한 곳으로 몰게 되면서 스레드를 제어하는 동시성 정책을 바꾸기도 쉬워진다.

## 실행 경로 이해하기

incrementValue 메서드

```java
public class IdGenerator {
   int lastIdUsed;
   
   public int incrementValue() {
      return ++lastIdUsed;
   }
}
```

만약 인스턴스는 하나지만 스레드가 두 개라면 아래와 같은 실행 결과가 가능하다.

- 스레드 1이 94, 2가 95, lastIdUsed가 95가 된다.
- 스레드 1이 95, 2가 94, lastIdUsed가 95가 된다.
- 스레드 1이 94, 2가 94, lastIdUsed가 94가 된다. ***→ ??***

가능한 경로 수를 계산하기 위해 자바 컴파일러가 생성한 바이트 코드를 살펴보자. return ++lastIdUsed 자바 코드 한 줄은 바이크 코드 명령 8개에 해당한다. 

즉, 두 스레드가 8개를 뒤섞어 실행할 가능성이 충분하고 책의 계산을 따르면 단계가 8개, 스레드가 2개일 때 가능한 경로의 수는 (int의 경우에) 12870개나 된다.

### 원자적 계산

*원자적 연산이란?*  **중단이 불가능한 연산**

예를 들어 `lastId = 0;`과 같이 lastId에 0을 할당하는 연산은 원자적이다. 하지만, long으로 바꾼다면 JVM명세에 따르게 되면, 32비트 값을 할당하는 연산 두 개로 나눠지고 스레드의 개입이 가능해지므로 원자적이지 않다.

`++lastId;`과 같은 전처리 증가 연산자 또한 원자적이지 않다.

바이트 코드로 한 번 살펴보자.

*바이트코드란?*  JVM에 의해 실행되는 최종 목적 코드로서 중간 형태의 프로그램

*프레임이란?*  모든 메서드 호출에는 프레임이 필요하다. 반환 주소, 메서드로 넘어온 매개변수, 메서드가 정의하는 지역 변수를 포함한다. 호출 스택을 정의할 때 사용하는 표준 기법이다.

*지역 변수란?*  메서드 범위 내에 정의되는 모든 변수를 가리킨다. 정적 메서드를 제외한 모든 메서드는 this라는 지역 변수를 갖는다. **현재 스레드에서 가장 최근에 메시지를 받아 메서드를 호출한 객체를 가리킨다.** 

*피연산자 스택이란?*  JVM이 지원하는 명령 대다수는 매개변수를 받는다. 피연산자 스택은 이런 매개변수를 저장하는 장소로 표준 LIFO 자료 구조다. 

`lastId = 0;` 만 수행하는 함수의 바이트 코드는 다음과 같다.

- ***ALOAD 0*** → ***ICONST_0*** → ***PUTFIELD lastId***
- 각 명령은 원자적이며 각 명령 사이에 다른 스레드가 끼어든다 하더라도 PUTFIELD를 수행하면  lastId에는 반드시 0이 들어간다.

하지만 `++lastId;` 만 수행하는 함수의 바이트 코드를 보게 되면 중간에 끼어들어 모든 명령을 수행하여 증가된 값을 가져가고 이전 스레드는 갖고 있던 값을 증가함으로써 위와 같은 상황이 발생할 수 있다.

- ***ALOAD 0*** → ***DUP*** → ***GETFIELD lastId*** → ***ICONST_1*** → ***IADD*** → ***DUP_X1*** → ***PUTFIELD lastId*** → ***IRETURN***

→ 모든 바이트 코드를 이해할 필요는 없지만 여러 스레드가 어떻게 동시성을 보장할 수 없게 되는지 그려볼 수 있어야 한다.

## 라이브러리를 이해하라

### Executor 프레임워크

스레드는 생성하나 스레드 풀을 사용하지 않는다면 혹은 직접 생성한 스레드 풀을 사용한다면 Executor 클래스를 고려하라.

Executor 프레임워크는 스레드 풀을 관리하고, 풀 크기를 자동으로 조정하며, 필요하다면 스레드를 재사용한다.

### 스레드를 차단하지 않는(non blocking) 방법

자바 5 VM는 스레드를 차단하지 않고 안정적으로 값을 갱신할 수 있는 `AtomicBoolean`, `AtomicInteger`, `AtomicReference`를 포함한 여러 클래스를 제공한다.

```java
private AtomicInteger value = new AtomicInteger(0);
value.incrementAndGet(); // value++;와 유사
value.get(); // 기본 유형이 아니라 객체 사용
```

`incrementAndGet()` 메서드는 `synchronized`를 이용하여 값을 증가시키는 메서드보다 거의 항상 더 빠르다.

*이게 가능한 이유는?*

**CAS**(Compare and Swap)이라 불리는 연산을 지원하기 때문이다. DB에서 optimistic locking이라는 개념과 유사하다고 한다. 반면 synchronized 키워드는 언제나 락을 거는 pessimistic locking과 유사하다. 

→ 락을 거는 대가는 비싸기 때문에 **락을 거는 것보다 문제를 감지하는 쪽**이 거의 항상 더 효율적이다.

### 다중 스레드 환경에서 안전하지 않은 클래스

본질적으로 다중 스레드 환경에서 안전하지 않는 클래스가 있다.

- SimpleDataFormat
- 데이터베이스 연결
- java.util 컨테이너 클래스
- 서블릿

Concurrent 패키지가 제공하는 클래스를 사용하거나, synchronized를 잘 사용하자.

## 메서드 사이에 존재하는 의존성을 조심하라

```java
// 서버
public class IntegerIterator implements Iterator<Integer>{
	private Integer nextValue = 0;
	
	public synchronized boolean hasNext(){
		return nextValue < 100000;
	}
	
	public synchronized Integer next(){
		if(nextValue == 10000)
			throw new IteratorPastEndException();
		return nextValue++;
	}

	public synchronized Integer getNextValue(){
		return nextValue;
	}
}
```

```java
// 클라이언트
IntegerInterator iterator = new IntegerIterator();
while(iterator.hasNext()){
	int nextValue = iterator.next();
}
```

스레드 하나가 위 코드를 실행한다면 아무 문제가 없지만 스레드 두 개가 인스턴스 하나를 공유하면서 맨 끝에 서로 간섭하게 된다면 예외가 발생할 가능성이 존재한다.

개별 메서드는 동기화되었지만, 클라이언트는 메서드 두 개를 동기화 없이 사용하고 있기 때문에 발생하는 문제이다.

### 실패를 용인한다

말 그대로 실패해도 괜찮도록 프로그램을 조정한다.

### 클라이언트-기반 잠금

다중 스레드 환경에서도 안전하도록 클라이언트를 변경한다. 아래는 예시이다.

```java
IntegetIterator iterator = new IntegerIterator();

while(true) {
   int nextValue;
   synchronized (iterator) {
      if(!iterator.hasNext())
         break;
      nextValue = iterator.next();
   }
   doSometingWith(nextValue);
}
```

서버를 사용하는 모든 프로그래머가 락을 기억해 객체에 걸었다 풀어야 하므로 다소 위험한 전략이다. 저자의 경험으로 보았을 때, 클라이언트-기반 잠금 메커니즘은 진짜 사람이 할 짓이 아니다. 

### 서버- 기반 잠금

서버 IntegerIterator를 다음과 같이 변경하면 클라이언트에 중복해서 락을 걸 필요가 없다.

```java
public class IntegerIteratorServerLocked {
   private Integer nextValue = 0;
   public synchronized Integer getNextOrNull() {
      if (nextValue < 100000)
         return nextValue++;
      else
         return null;
   }
}
```

마치, ConcurrentHashMap의 putIfAbsent() 와 같은 역할이다. 

일반적으로 서버- 기반 잠금이 더 바람직하다.

- 코드 중복이 줄어든다.
- 성능이 좋아진다.
- 오류가 발생할 가능성이 줄어든다. 잠금을 잊어버리는 바람에 오류가 발생할 위험은 프로그래머 한 명으로 제한된다.
- 스레드 정책이 하나다. 서버 한곳에서 정책을 구현한다.
- 공유 변수 범위가 줄어든다. 모두가 서버에 숨겨진다. 문제가 생기면 살펴볼 곳이 적다.

서버 코드에 손대지 못한다면 ADAPTER 패턴을 사용해 API를 변경한 후 잠금을 추가하면 된다.

## 데드락

데드락은 다음 네 조건을 모두 만족해야 발생한다.

- **상호 배제** : 공유 자원을 여러 스레드가 동시에 사용하지 못하고 그 개수가 제한적
- **잠금&대기** : 스레드가 자원을 점유하면 필요한 나머지 자원까지 점유해 작업을 마치지 않는 이상 점유한 자원을 포기하지 않는 것
- **선점 불가** : 한 스레드가 다른 스레드로부터 자원을 빼앗지 못하는 것
- **순환 대기** :
    
    ![Untitled](%E1%84%87%E1%85%AE%E1%84%85%E1%85%A9%E1%86%A8%20A%20%E1%84%83%E1%85%A9%E1%86%BC%E1%84%89%E1%85%B5%E1%84%89%E1%85%A5%E1%86%BCII%2006baa3af1ce94d928716d329bcb31258/Untitled.png)
    

네 조건 중 하나의 조건만 만족하지 않아도 데드락을 발생하지 않는다. *각 조건을 비껴가는 방법을 알아보자.*

- **상호 배제 조건 깨기**
    - 동시에 사용해도 괜찮은 자원(eg. **AtomicInteger**)을 사용한다.
    - 스레드 수 이상으로 자원의 수 늘린다.
    - 자원을 점유하기 전에 필요한 자원이 모두 있는지 확인한다.
- **잠금&대기 조건 깨기** : 만약 실행 중 자원 어느 하나라도 점유하지 못한다면 지금까지 점유한 자원을 다 내놓고 처음부터 시작한다.
    
    하지만 다음과 같은 문제가 있다.
    
    - 기아: 한 스레드가 계속해서 필요한 자원을 점유하지 못한다.
    - 라이브락: 여러 스레드가 한꺼번에 잠금 단계로 진입하는 바람에 계속 자원을 점유했다 내놨다를 반복한다.
- **선점 불가 조건 깨기** : 필요한 자원이 잠겼다면 자원을 소유한 스레드에서 풀어달라 요청하는 메커니즘으로 처리한다.
- **순환 대기 조건 깨기** : 모든 스레드가 일정 순서에 동의하고 그 순서로만 자원을 할당하면 된다. 위의 그림과 같은 예시에서는 T1과 T2가 자원을 똑같은 순서로 할당하게 만들면 된다.