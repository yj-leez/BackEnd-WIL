# 7장. 람다와 스트림

## 아이템 42. 익명 클래스보다는 람다를 사용하라

- 람다식: 함수형 프로그래밍을 지원하며, 코드를 간결하고 간편하게 정의할 수 있음
    - (`parameters`) -> `expression` : `parameters`는 매개변수 목록이고, `expression`은 람다식이 반환하는 결과
    
    ```java
    new Thread(new Runnable() {
       @Override
       public void run() { 
          System.out.println("Welcome Heejin blog"); 
       }
    }).start();
    
    /**
    * 람다식 사용
    */
    new Thread(()->{
          System.out.println("Welcome Heejin blog");
    }).start();
    ```
    
- 이전에는 익명 클래스를 함수 객체로 사용했음 → 자바는 함수형 프로그래밍에 적합하지 않았음
    
    → 람다식을 함수 객체로 사용하여 익명 클래스를 대체
    
    ```java
    Collections.sort(words, 
    					(s1,s2) -> Integer.compare(s1.length(), s2.length()));
    ```
    
    - 람다 대신 비교자 생성 메서드를 사용하면 코드를 더 간결하게 할 수 있음
        
        ```java
        Collections.sort(words, comparingInt(String::length));
        ```
        
- 상수별 클래스 몸체와 데이터를 사용한 열거타입(아이템 34)은 *람다를 인스턴스 필드에 저장한* 열거타입으로 간단히 구현 가능
    
    ```java
    public enum Operation {
        PLUS  ("+", (x, y) -> x + y),
        MINUS ("-", (x, y) -> x - y),
        TIMES ("*", (x, y) -> x * y),
        DIVIDE("/", (x, y) -> x / y);
    
        private final String symbol;
        private final DoubleBinaryOperator op;
    
        Operation(String symbol, DoubleBinaryOperator op) {
            this.symbol = symbol;
            this.op = op;
        }
    		@Override public String toString() { return symbol; }
    
        public double apply(double x, double y) {
            return op.applyAsDouble(x, y);
        }
    }
    ```
    
- 하지만 람다는 이름도 없고 문서화도 못하므로 코드 줄 수가 길어지거나 코드 자체가 명확하지 않으면 사용하지 않는 것이 좋음
- 또한 람다는 함수형 인터페이스(딱 하나의 추상 메서드만을 가지는 인터페이스)에서만 쓰일 수 있고, 자기자신을 참조할 수 없다는 제약을 가짐
    
    ```java
    @FunctionalInterface // 함수형 인터페이스임을 알리는 애너테이션
    interface MyOperation {
        int operate(int a, int b);
    }
    
    public class Main {
        public static void main(String[] args) {
            // 람다 표현식을 사용하여 함수형 인터페이스 구현
            MyOperation addition = (a, b) -> a + b;
            MyOperation subtraction = (a, b) -> a - b;
    
            // 람다 표현식을 통한 메서드 호출
            System.out.println("Addition: " + addition.operate(5, 3));
            System.out.println("Subtraction: " + subtraction.operate(5, 3));
        }
    }
    ```
    

## 아이템 43. 람다보다는 메서드 참조를 사용하라

- 람다로 할 수 있는 일 중 메서드 참조로 해결할 수 있는 일이 있음

| 메서드 참조 유형 | 예 | 같은 기능을 하는 람다 |
| --- | --- | --- |
| 정적
클래스::정적메소드 | Integer::parseInt
Math::sqrt | str → Integer.parseInt(str)
number -> Math.sqrt(number) |
| 한정적(인스턴스)
인스턴스::인스턴스메소드 | Instant.now()::isAfter

System.out::println | Instant then = Instant.now();
t → then.isAfter(t)

number → System.out.println(number) |
| 비한정적(인스턴스)
클래스::인스턴스메소드 | String::toLowerCase
Integer::compareTo | str → str.toLowerCase()
(a, b) -> a.compareTo(b) |
| 클래스 생성자 | TreeMap<K,V>::new | () → new TreeMap<K,V>() |
| 배열 생성자 | int[]::new | len → new int[len] |

### 매서드 참조는 람다의 간단한 대안이 될 수 있음

**매서드 참조가 짧고 명확하다면 메서드 참조를, 그렇지 않은 경우만 람다를 사용하라**

## 아이템 44. 표준 함수형 인터페이스를 사용하라

????? 다시 보자

## 아이템 45.  스트림은 주의해서 사용하라

- 스트림: 컬렉션에 저장되어 있는 엘리먼트들을 하나씩 순회하면서 처리 할 수 있는 코드패턴
    
    람다식과 함께 사용되어 컬렉션에 들어있는 데이터에 대한 처리를 간결한 표현으로 작성할 수 있음
    

```java
ArrayList<String> list = new ArrayList<String>(Arrays.asList("a", "b", "c"));
list.stream()
    .filter("b"::equals)    
    .forEach(System.out::println);
```

- 하지만 과하게 쓰면 오히려 읽기 힘들고 유지보수하기 어려워짐

```java
try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
}

```

- 스트림을 사용하면 좋은 상황
    1. 원소들의 시퀀스를 일관되게 변환
    2. 원소들의 시퀀스를 필터링
    3. 원소들의 시퀀스를 하나의 연산을 사용해 결합
    4. 원소들의 시퀀스를 컬렉션에 모으는 경우
    5. 특정 조건을 만족하는 원소를 찾는 경우
    6. 원소들의 시퀀스를 병렬로 처리하는 경우
- 스트림을 사용하기 어려운 상황
    1. 스트림 파이프라인이 여러 연산단계로 구성될 때, 각 단계의 값들을 동시에 접근하고자 할 때
        - 스트림 파이프라인은 연산이 지나가면 원래값을 잃는 구조이기 때문
- 스트림을 리턴하는 메서드는 복수 명사로 쓰기를 추천

## 아이템 46. 스트림에서는 부작용 없는 함수를 사용하라

- 스트림 패러다임의 핵심: 계산을 일련의 변환으로 재구성하는 부분
    
    각 변환 단계는 입력만이 결과에 영향을 주는 순수함수여야 함
    
    → 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야 함
    

```java
/**
* 스트림을 제대로 활용하지 못한 예 - 스트림을 가장한 반복적 코드
*/
Map<String, Long> freq = new HashMap<>();
        try (Stream<String> words = new Scanner(file).tokens()) {
            words.forEach(word -> {
                freq.merge(word.toLowerCase(), 1L, Long::sum);
            });
        }

/**
* 스트림을 제대로 활용한 예
*/
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}

```

- Collectoors 클래스를 사용하면 스트림의 원소를 컬렉션으로 쉽게 모을 수 있음
    - `toList()`, `toSet()`, `toCollection()`: 스트림의 원소를 컬렉션으로 모으는 수집기
        
        ```java
        List<String> topTen = freq.keySet().stream()
                        .sorted(comparing(freq::get).reversed())
                        .limit(10)
                        .collect(toList());  // Collectors의 멤버를 정적 임포트하면 가독성이 좋아짐
        ```
        
    - `groupingBy`: SQL의 groupby와 유사
        
        ```java
        Stream<String> s = Stream.of("apple", "banana", "orange");
        Map<Integer, Long> map = s.collect(Collectors.groupingBy(String::length, Collectors.counting()));
        ```
        
    - `joining`: 문자열(CharSequence)에만 사용하며 문자열들을 연결함

## 아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

- 스트림은 Iterable을 지원하지 않고, Iterable 역시 스트림을 지원하지 않은 것 등을 고려했을 때 반환타입은 Collection 인터페이스로 하는 것을 추천
    - 컬렉션은 Iterable과 스트림 둘 다 지원하기 때문

## 아이템 48. **스트림 병렬화는 주의해서 적용하라**

- 병렬 스트림은 .parallel() 로 제공되지만 데이터 소스가 Stream.iterate 거나 중간 연산으로 limit을 쓸 때는 사용하지 말아라
- 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap, 배열, int, long 일 때가 병렬화의 효과가 가장 좋음

**그냥 계산히 확실히 올바르게 수행되고 성능도 확실히 빨라질 거라는 확신 없이는 스트림 파이프라인 병렬화는 시도도 말아라**