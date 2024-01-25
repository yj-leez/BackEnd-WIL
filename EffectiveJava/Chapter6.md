# 6장. 열거타입과 애너테이션

## 아이템 34. int 상수 대신 열거 타입을 사용하라

### 열거타입

- 자바의 열거 타입 자체는 클래스이며, 상수 하나 당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개
    
    → 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없어 인스턴스들은 딱 하나씩 존재하고 싱글턴을 일반화한 형태로 볼 수 있음
    
- 열거 타입은 컴파일타임 타입 안정성을 제공
    
    ```java
    public enum WeekDay {
        MONDAY(0),
        TUESDAY(1),
        WEDNESDAY(2),
        THURSDAY(3),
        FRIDAY(4),
        SATURDAY(5),
        SUNDAY(6);
    
        private final int value;
    
        WeekDay(int value) {
            this.value = value;
        }
    }
    /**
    * WeekDay 열거타입을 인자로 받으려는 메소드는 반드시 WeekDay의 인스턴스를 건네줘야 함
    */
    enumTypeMethod(MONDAY); // 오류
    enumTypeMethod(WeekDay.MONDAY); // 성공
    ```
    
- 이름이 같은 상수도 평화롭게 공존 → ??
- 열거 타입의 toString 메소드는 출력하기에 적합한 문자열을 내어줌 → 재정의하여 사용 가능
    
    ```java
    assertThat(WeekDay.Monday.toString()).isEqualTo("MONDAY"); // 테스트 통과
    ```
    
- 열거 타입에는 임의의 메소드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수 있음
    
    : Object 메서드들을 높은 품질로 구현해놨고 Comparable과 Serializable을 구현함
    
- 실제로는 클래스이므로 고차원의 추상 개념 하나를 완벽히 표현해낼 수도 있음
    
    : 상수들이 서로 다른 데이터와 연결된 경우
    
    ```java
    // 코드 34-3 데이터와 메서드를 갖는 열거 타입 (211쪽)
    public enum Planet {
        MERCURY(3.302e+23, 2.439e6),
        VENUS  (4.869e+24, 6.052e6),
        EARTH  (5.975e+24, 6.378e6),
        MARS   (6.419e+23, 3.393e6),
        JUPITER(1.899e+27, 7.149e7),
        SATURN (5.685e+26, 6.027e7),
        URANUS (8.683e+25, 2.556e7),
        NEPTUNE(1.024e+26, 2.477e7);
    
        private final double mass;           // 질량(단위: 킬로그램)
        private final double radius;         // 반지름(단위: 미터)
        private final double surfaceGravity; // 표면중력(단위: m / s^2)
    
        // 중력상수(단위: m^3 / kg s^2)
        private static final double G = 6.67300E-11;
    
        // 생성자
        Planet(double mass, double radius) {
            this.mass = mass;
            this.radius = radius;
            surfaceGravity = G * mass / (radius * radius);
        }
    
        public double mass()           { return mass; }
        public double radius()         { return radius; }
        public double surfaceGravity() { return surfaceGravity; }
    
        public double surfaceWeight(double mass) {
            return mass * surfaceGravity;  // F = ma
        }
    }
    ```
    

### **상수별 메소드 구현**

- 상수마다 동작이 달라져야 하는 상황에 적용
- 열거 타입에 추상 메소드를 선언하고 각 상수별로 클래스 몸체를 상수가 자신에 맞게 재정의하는 방법

```java
public enum Operation {
    PLUS("+") { public double apply(double x, double y) { return x + y; } },
    MINUS("-") { public double apply(double x, double y) { return x - y; } },
    TIMES("*") { public double apply(double x, double y) { return x * y; } },
    DIVIDE("/") { public double apply(double x, double y) { return x / y; } };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y); // 추상 메소드
}
```

- toString의 사용 예시

```java
for (Operation op : Operation.values())
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
```

→ `toString()` 메서드를 재정의하려든, `toString()`이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 `fromString()` 메서드도 함께 제공하자

```java
private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, e -> e));

public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
}
```

### **전략 열거 타입 패턴**

- eg) 잔업수당 계산을 private 중첩 열거 타입으로 위임하고 PayrollDay 열거 타입 생성자에서 적절한것을 선택하도록
    
    : 각 날짜에 대한 급여 계산 방법은 `PayType` 열거 타입에 의해 결정
    
    새로운 급여 계산 전략이 추가되거나 변경되어야 한다면, `PayType`에 새로운 상수를 추가하거나 상수의 구현을 변경하면 됨
    

```java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked,payRate);
    }

    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked,payRate);
        }
    }
}
```

### 열거 타입을 언제 쓰면 좋을까?

**필요한 원소를 컴파일 타임에 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자**

## 아이템 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

- 열거타입은 해당 상수가 열거 타입에서 몇 번째 위치인지 반환하는 ordinal 메서드 제공 → 사용하지 말자
    
    대신, 열거 타입 상수에 연결된 값이 필요하다면 인스턴스 필드에 저장하자
    

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

## 아이템 36. 비트 필드 대신 EnumSet을 사용하라

- 열거한 값들이 주로 집합으로 사용될 경우, 비트 필드 열거 상수를 사용하지 말고 EnumSet을 사용하라

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    // 사용 예
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

## 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라

- 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 코드가 있음 → 하지 마라

```java
public class Plant {
    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}

public enum LifeCycle {
    ANNUAL, PERNNIAL, BIENNIAL
}

/** 
* ordinal()을 배열 인덱스로 사용
*/
// Set 배열을 생성해 생애주기별로 관리
Set<Plant>[] plantsByLifeCycleArr =
                (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

// 각 배열을 순회하여 빈 HashSet으로 초기화
for (int i = 0; i < plantsByLifeCycleArr.length; i++)
            plantsByLifeCycleArr[i] = new HashSet<>();

// plant 들을 배열의 Set에 추가
// 이때 plant가 가지고있는 LifeCycle 열거타입의 ordinal 값으로 배열의 인덱스를 결정
// 그 결과 식물의 생애주기 별로 Set에 추가
for (Plant p : garden)
            plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);

// 결과 출력
for (int i = 0; i < plantsByLifeCycleArr.length; i++) {
            System.out.printf("%s: %s%n",
                    Plant.LifeCycle.values()[i], plantsByLifeCycleArr[i]);
}
```

- 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체, EnumMap을 사용하자

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
         new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values())
      plantsByLifeCycle.put(lc, new HashSet<>());

for (Plant p : garden)
      plantsByLifeCycle.get(p.lifeCycle).add(p);

System.out.println(plantsByLifeCycle);
```

→ stream을 사용하여 맵을 관리하면 코드도 더 줄일 수 있음

- 중첩 EnumMap을 사용하여 데이터와 열거타입 쌍을 연결할 수도 있음

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

### 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 좋지 않다

**대신 EnumMap을 사용하고 다차원 관계는 EnumMap< … , EnumMap< …> >으로 표현하라**

## 아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

- 열거 타입은 보통 확장에 적합하지 않으므로 인터페이스를 이용해 확장 가능 열거 타입을 흉내낼 수 있음

```java
public interface Operation {
    double apply(double x, double y);
}
public enum BasicOperation implements Operationn {
    PLUS("+") { public double apply(double x, double y) { return x + y; } },
    MINUS("-") { public double apply(double x, double y) { return x - y; } },
    TIMES("*") { public double apply(double x, double y) { return x * y; } },
    DIVIDE("/") { public double apply(double x, double y) { return x / y; } };
    
    private final String symbol;
    
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override
    public String toString() {
        return symbol;
    }
}
public enum ExtendedOperation implements Operation {
    EXP("^") { public double apply(double x, double y) { return Math.pow(x, y);} },
    REMAINDER("%") { public double apply(double x, double y) { return x % y;} };
    
    private final String symbol;
    ...
}
```

- inteface를 구현하는 여러 enum들에 접근할 수 있음
- 같은 interface를 implements하는 구현체 enum끼리 구현을 상속할 수 없다는 단점 존재
    
    → 공통 부분이 적다면 interface내에서 디폴트 메서드로 구현, 많다면 별도의 도우미 클래스나 정적 도우미 메서드로 분리
    

## 아이템 39. 명명 패턴보다 애너테이션을 사용하라

```java
public class Calculator {
    
    public int addTwoNumbers(int operand1, int operand2) {
        return operand1 + operand2;
    }
}
```

- 명명 패턴은 단점이 많음
    1. 오타를 내면 안됨
    2. 올바른 프로그램 요소에서만 사용된단 보증이 없음
    3. 프로그램 요소를 매개변수로 전달할 방법이 없음

↓ 애너테이션을 사용하자

```java
public class Calculator {
    
    @Addition
    public int addTwoNumbers(@Operand int x, @Operand int y) {
        return x + y;
    }
}

@Retention(RetentionPolicy.RUNTIME) // 런타임에도 유지되어야 함
@Target(ElementType.PARAMETER)  // 파라미터에서만 사용돼야 함
public @interface Operand {
}
```

- 애너테이션 선언에 다는 애너테이션(`@Retention`, `@Target`): 메타애너테이션
- 마커 애너테이션: 아무 매개변수 없이 대상에 마킹하는 애너테이션 eg) `@Operand`
    
    ↔ 매개변수 하나를 받는 애너테이션
    
    ```java
    /**
     * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
     */
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
        Class<? extends Throwable> value(); // 특정 종류의 예외 클래스를 나타내는 Class 객체를 반환
    }
    
    /**
    * 마커 애너테이션과 매개변수 하나짜리 애너태이션을 처리하는 프로그램
    */
    public class RunTests {
        public static void main(String[] args) throws Exception {
            int tests = 0;
            int passed = 0;
            Class<?> testClass = Class.forName(args[0]);
            for (Method m : testClass.getDeclaredMethods()) {
                if (m.isAnnotationPresent(Test.class)) {
                    tests++;
                    try {
                        m.invoke(null);
                        passed++;
                    } catch (InvocationTargetException wrappedExc) {
                        Throwable exc = wrappedExc.getCause();
                        System.out.println(m + " 실패: " + exc);
                    } catch (Exception exc) {
                        System.out.println("잘못 사용한 @Test: " + m);
                    }
                }
    
                if (m.isAnnotationPresent(ExceptionTest.class)) {
                    tests++;
                    try {
                        m.invoke(null);
                        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                    } catch (InvocationTargetException wrappedEx) {
                        Throwable exc = wrappedEx.getCause();
                        Class<? extends Throwable> excType =
                                m.getAnnotation(ExceptionTest.class).value();
                        if (excType.isInstance(exc)) {
                            passed++;
                        } else {
                            System.out.printf(
                                    "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                    m, excType.getName(), exc);
                        }
                    } catch (Exception exc) {
                        System.out.println("잘못 사용한 @ExceptionTest: " + m);
                    }
                }
            }
    
            System.out.printf("성공: %d, 실패: %d%n",
                    passed, tests - passed);
        }
    }
    ```
    
- 배열 매개변수를 받는 애너테이션 타입도 구현할 수 있음

## 아이템 40. @Override 애너테이션을 일관되게 사용하라

- `@Override`: 상위 타입의 메서드를 재정의함
- 애너테이션을 달지 않고 재정의하려할 시 런타임 시 에러가 발생하기 때문에 예외 없이 애너테이션을 달자
    
    하나의 예외: 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때
    

## 아이템 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

- 마커 인터페이스: 아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스 eg) `Serializable`
- 장점
    1. 마커 인터페이스는 마커 애너테이션과 다르게 구현한 클래스의 인스턴스들을 구분하는 타입으로 사용 가능
        
        → 컴파일 타임에 에러 발견 가능
        
    2. 적용 대상을 마커 애너테이션보다 정밀하게 지정할 수 있음
- 단점
    1. 마커 애너테이션은 거대한 애너테이션 시스템의 지원을 받을 수 있음

### 마커 인터페이스와 마커애너테이션은

**각자의 쓰임이 있으므로 단지 타입 정의가 목적이라면 마커 인터페이스를, 클래스나 인터페이스 외의 프로그램 요소에 마킹해야 한다면 마커 애너테이션을 사용하자**