# 2장. 객체 생성과 파괴

## 아이템1. 생성자 대신 정적 팩터리 메서드를 고려하라

### 정적 팩터리 메서드란

객체 생성을 생성자가 하지 않고 객체 생성의 역할을 하는 메서드

### 정적 팩터리 메서드의 이점

1. 이름을 가질 수 있다

    ```java
    public class Car {

        private final String name;
        private final int oil;

        public static Car createCar(String name, int oil) {
            return new Car(name, oil);
        }

        public static Car createNoOilCar(String name) {
            return new Car(name, 0);
        }

        private Car(String name, int oil) {
            this.name = name;
            this.oil = oil;
        }
    }
    ```

2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다

    사용되는 값들의 개수가 정해져 있으면 해당 값을 미리 생성해놓고 조회(캐싱) 할 수 있는 구조로 만들 수 있다

    ```java
    public class Day {

        private static final Map<String, Day> days = new HashMap<>();

        static {
            days.put("mon", new Day("Monday"));
            days.put("tue", new Day("Tuesday"));
            days.put("wen", new Day("Wednesday"));
            days.put("thu", new Day("Thursday"));
            days.put("fri", new Day("Friday"));
            days.put("sat", new Day("Saturday"));
            days.put("sun", new Day("Sunday"));
        }

        public static Day from(String day) {
            return days.get(day);
        }
    }
    ```

3. 반환 타입의 하위 타입 객체를 반환할 수 있다

    분기처리를 통해 하위 타입의 객체를 반환할 수 있다

4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다

    반환 타입의 하위 타입이기만 하면 어떤 클래스의 개체를 반환하든 상관없다

5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

    ??????? 이해안됨

6. 객체 생성을 캡슐화 할 수 있다

    정적 팩토리 메서드에 DTO에서 꺼낸 변수가 아닌 DTO를 던지고 안에서 다시 가공을 하면 변수를 캡슐화 할 수 있다.

    ```java
    public class PersonDTO {
        private String firstName;
        private String lastName;
    }

    public class Person {
        private String fullName;

        private Person(String fullName) {
            this.fullName = fullName;
        }

        public static Person createPersonFromDTO(PersonDTO personDTO) {
            // 여기에서 DTO의 데이터를 가공하거나 로직을 수행할 수 있음
            String processedFullName = processFullName(personDTO.getFirstName(), personDTO.getLastName());

            // 생성된 객체 반환
            return new Person(processedFullName);
        }

        private static String processFullName(String firstName, String lastName) {
            // 여기에서 DTO의 데이터를 가공하는 로직을 수행
            return firstName + " " + lastName;
        }
    }
    ```


### 정적 팩터리 메서드의 단점

1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공 시 하위 클래스를 만들 수 없다

    클래스를 상속하려면 하위 클래스가 상위 클래스의 생성자를 호출할 수 있어야 하지만

    정적 팩토리 메서드만을 제공하면 생성자를 직접 호출하는 것이 불가능하므로 상속이 어려워지기 때문

2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다

→ 정적 팩터리 메서드와 public 생성자는 각 장단점을 고려하여 사용해야함. 하지만 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 public 생성자만 제공하던 습관이 있다면 주의.

## 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

- 정적 팩터리 메서드와 public 생성자에는 똑같은 제약이 존재: 선택적 매개변수가 많을 때 적절히 대응하기 어려움
- 해결방안
1. 점층적 생성자 패턴 → 확장하기 어려움
2. 자바빈즈 패턴

    ```java
    public class NutritionFacts {
        // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
        private int servingSize  = -1; // 필수; 기본값 없음
        private int servings     = -1; // 필수; 기본값 없음
        private int calories     = 0;
        private int fat          = 0;
        private int sodium       = 0;
        private int carbohydrate = 0;

        public NutritionFacts() { }
        // Setters
        public void setServingSize(int val)  { servingSize = val; }
        public void setServings(int val)     { servings = val; }
        public void setCalories(int val)     { calories = val; }
        public void setFat(int val)          { fat = val; }
        public void setSodium(int val)       { sodium = val; }
        public void setCarbohydrate(int val) { carbohydrate = val; }

        public static void main(String[] args) {
            NutritionFacts cocaCola = new NutritionFacts();
            cocaCola.setServingSize(240);
            cocaCola.setServings(8);
            cocaCola.setCalories(100);
            cocaCola.setSodium(35);
            cocaCola.setCarbohydrate(27);
        }
    }
    ```

    - 객체 하나를 만들려면 메서드르 여러 개 호출해야 하고 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이는 단점 존재
3. 빌더 패턴 : 점층적 생성자 패턴의 안전성 + 자바빈즈 패턴의 가독성

    ```java
    public class NutritionFacts {
        private final int servingSize;
        private final int servings;
        private final int calories;
        private final int fat;
        private final int sodium;
        private final int carbohydrate;

        public static class Builder {
            // 필수 매개변수
            private final int servingSize;
            private final int servings;

            // 선택 매개변수 - 기본값으로 초기화한다.
            private int calories      = 0;
            private int fat           = 0;
            private int sodium        = 0;
            private int carbohydrate  = 0;

            public Builder(int servingSize, int servings) {
                this.servingSize = servingSize;
                this.servings    = servings;
            }

            public Builder calories(int val)
            { calories = val;      return this; }
            public Builder fat(int val)
            { fat = val;           return this; }
            public Builder sodium(int val)
            { sodium = val;        return this; }
            public Builder carbohydrate(int val)
            { carbohydrate = val;  return this; }

            public NutritionFacts build() {
                return new NutritionFacts(this);
            }
        }

        private NutritionFacts(Builder builder) {
            servingSize  = builder.servingSize;
            servings     = builder.servings;
            calories     = builder.calories;
            fat          = builder.fat;
            sodium       = builder.sodium;
            carbohydrate = builder.carbohydrate;
        }

        public static void main(String[] args) {
            NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                    .calories(100).sodium(35).carbohydrate(27).build();
        }
    }
    ```

    - 필수 매개변수만으로 생성자(정적 팩터리 메소드)를 호출해 빌더 객체를 얻고, 객체가 제공하는 일종의 세터 메서드들로 원하는 매개변수들을 선택 설정
    - 빌더의 생성자와 메서드에서 입력 매개변수를 검사해야함
    - 계층적으로 설계된 클래스와 함께 쓰기에 좋음
    - 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 함

## 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

- 세 가지 방법 존재: pulbic static final 필드 방식, 정적 팩터리 방식, 열거 타입 방식

### public static final필드 방식의 싱글턴

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

}
public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
}
```

- Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장
- 해당 클래스가 싱글턴임이 API에 명확히 확인 가능 → public static final

### 정적 팩터리 방식의 싱글턴

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

}
public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();
}
```

- 싱글턴 방식이 아니게 변경하기 쉬움
- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 점
- 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다는 점

    ```java
    import java.util.function.Supplier;

    public class Elvis {
        private static final Elvis INSTANCE = new Elvis();

        private Elvis() {
            // private constructor
        }

        public static Elvis getInstance() {
            return INSTANCE;
        }

        public static void main(String[] args) {
            Supplier<Elvis> singletonSupplierReference = Singleton::getInstance;
            Elvis elvis2 = singletonSupplierReference.get();
        }
    }
    ```


→ 하지만 두 방법 모두 싱글턴 클래스를 직렬화하려면 단순히 Serializable 구현한다고 선언하는 것만으로는 역직렬화 시 새로운 인스턴스가 만들어지기 때문에 부족함 → `readResovle` 메서드를 구현하여 방지

```java
private Object readResolve(){
		return INSTANCE;
}
```

*직렬화(serialize): 자바 언어에서 사용되는 Object 또는 Data를 다른 컴퓨터의 자바 시스템에서도 사용 할수 있도록 바이트 스트림(stream of bytes) 형태로 연속전인(serial) 데이터로 변환하는 포맷 변환 기술

, 휘발성이 있는 캐싱 데이터를 영구 저장이 필요할 때 사용

### 열거 타입 방식의 싱글턴 - 바람직한 방법

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
}
public static void main(String[] args) {
    Elvis elvis = Elvis.INSTANCE;
    elvis.leaveTheBuilding();
}
```

- public 필드 방식과 유사하지만 더 간결하고, 복잡한 직렬화 상황이나 리플렉션 공격에도 또 다른 인스턴스가 생기는 것을 방지

### Spring에서

- 단, 싱글턴의 단점이 존재하는데 유닛 테스트 시에 Mock 객체(가짜 객체)를 만들 수가 없음

    → 의존성 주입(아이템 5)으로 이를 해결


## 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

- 정적 메서드와 정적 필드만을 담은 클래스
    - 유틸리티 클래스: 관련된 메서드들을 제공

        ```java
        public class MathUtils {
            private MathUtils() {
                // private constructor to prevent instantiation
            }

            public static int add(int a, int b) {
                return a + b;
            }

            public static int multiply(int a, int b) {
                return a * b;
            }

            // ... other static methods
        }
        ```

    - 상수 클래스: 상수를 정의하는 클래스로 사용, 다양한 클래스에서 공통으로 사용되는 값을 저장하는 데에 유용
- static은 상태에 대한 저장이나 다른 메소드의 상태를 의존한다거나 하는 경우가 아닌 경우(Autowired로 의존성 주입이 필요하지 않은 경우)에 보통 사용
- 위와 같은 클래스들은 인스턴스로 만들어 쓰려고 설계한 게 아니지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어줌
- 이를 막기위해 private 생성자를 추가하여 인스턴스화를 막을 수 있음

```java
    private MathUtils() {
        throw new AssertionError();
    }
```

## 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

```java
public class SpellChecker {
    private final Lexicon dictionary;

    /* 의존성 주입 */
    public SpellChecker(Lexicon dictionary){
    	this.dictionary = Objects.requireNotNull(dictionary);
    }

    public static boolean isVaild(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
/* 사용 예시 */
Lexicon dic = new myDictionary();
SpellChecker chk = new SpellChecker(dic);
```

- 의존 객체 주입 방식의 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식이 존재

### Spring에서

- 의존 객체 주입 프레임워크 Spring을 이용하면 의존 객체 주입을 편리하게 사용 가능
    - 생성자 주입, 세터 주입, 필드 주입
- 다른 객체의 상태에 의존하는 경우에는 정적 메소드가 아닌 @Bean이나 @Component를 사용하여 클래스 간의 의존성 해결
    - @Bean: Config 클래스에서 메서드를 통해 명시적으로 빈을 등록
    - @Component: 클래스 자체를 스프링 빈으로 등록하는 방식, @Autowired를 생성자에 사용하여 의존성을 주입

## 아이템 6. 불필요한 객체 생성을 피하라

- 생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있음

    → Boolean(String) 대신 Boolean.valueOf(String) 팩터리 메서드를 사용

- 비싼 객체가 반복해서 필요하다면 캐싱하여 재사용하길 권장

    ```java
    static boolean isRomanNumeral(String s) {
            return s.marches("^(?=.)M*(C[MD] }D?C{0,3})"
                            + "(X[CL]}L?X{0,3})(I[XV]|V?I{0,3})$");
    }
    ↓
    private static final Pattern ROMAN = Pattern.compile(
                "^(?=.)M*(C[MD]|D?C{0,3})"
                        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
            return ROMAN.matcher(s).matches();
    }
    ```

- 반대로 불필요한 객체를 만들어내는 경우

    : 변수를 원시 타입(long)이 아닌 래퍼 클래스(Long)로 선언해서 불필요한 Long 인스턴스가 약 231개나 만들어지는 경우 = 의도치 않은 오토박싱이 숨어들은 경우


## 아이템 7. 다 쓴 객체 참조를 해제하라

- 자기 메모리르 직접 관리하는 클래스라면 null 처리를 하여 메모리 누수현상을 방지하자

    → Stack클래스, 캐시, 리스너 혹은 콜백이 대표적인 케이스


## 아이템 8. finalizer와 cleaner 사용을 피하라

- finalizer와 cleaner는 자바가 제공하는 객체 소멸자이지만, 치명적인 부작용이 존재하므로 사용하지말자

    → 사실 처음 봄


### Spring에서

- 기본적으로 객체는 Spring IoC(Inversion of Control) 컨테이너가 담당

## 아이템 9. try-finally보다는 try-with-resources를 사용하라

- Stream, DB Connection, File 등 외부 자원을 이용할 때는 close 메서드를 호출해 직접 닫아줘야함
- 두 가지 이상의 자원을 닫기위해 `try-finally`문을 사용하는 경우 물리적인 문제가 생겼을 때 제대로 닫히지 않을 가능성이 큼
- `try-with-resources`로 해결 가능

```java
static void copy(String src, String dst) throws IOException {

     try ( InputStream in = new FilInputStream(src);
                OutStream out = new FileOutputStream(dst)) {
             byte[] buf = new byte[BUFFER_SIZE];
             int n;
             while ((n = in.read(buf)) >= 0)
                 out.write(buf, 0, n);
     }
}
```

→ 꼭 회수해야하는 자원을 다룰 때는 `try-with-resources`를 사용하자
