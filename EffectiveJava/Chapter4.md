# 4장. 클래스와 인터페이스

## 아이템 15. 클래스와 멤버의 접근 권한을 최소화하라

- 정보 은닉은 중요함 → 자바는 정보 은닉을 위한 다양한 장치를 제공
- 접근 제어 메커니즘: 클래스, 인터페이스, 멤버의 접근성을 명시
    - `private`: 멤버를 선언한 톱레벨 클래스에서만 접근 가능
    - `package-private`: 멤버가 소속된 패키지 안의 모든 클래스에서 접근 가능
        
        접근 제한자를 명시하지 않았을 때 적용되는 패키지 접근 수준
        
        → 외부에서 쓸 이유가 없으면 public보다는 package-private으로 선언하자
        
    - `protected`: package-private의 접근 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근 가능
    - `public`: 모든 곳에서 접근 가능
- 공개 API 설계 후 그 외의 모든 멤버는 private으로 만들고 오직 같은 패키지의 다른 클래스가 접근해야하는 멤버에 한하여 package-private으로 풀어주자
- 일반적으로 스레드 안전을 위해 public 클래스의 인스턴스 필드는 되도록 public이 아니어야 함
- public static final는 기본 타입이나 불변 객체를 참조
    
    → Collections.unmodifiableList(), Map 등으로 불변화해서 참조
    

## 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

- 접근자와 변경자 메서드를 활용해 항상 데이터를 캡슐화해야함

## 아이템 17. 변경 가능성을 최소하라

- 불변 클래스: 인스턴스의 내부 값을 수정할 수 없는 클래스 eg) String, 기본 타입의 박싱된 클래스들
- 클래스를 불변으로 만들기 위한 규칙
    1. 객체의 상태를 변경하는 메서드를 제공하지 않음
    2. 클래스를 확장할 수 없도록 함
        
        : 클래스를 final로 선언하거나 모든 생성자를 private 혹은 package-private으로 만들고 public 정적 팩터리를 제공
        
    3. 모든 필드를 final로 선언
    4. 모든 필드를 private으로 선언
    5. 자신 외에는 가변 컴포넌트에 접근할 수 없도록 함
- 불변 객체는 기본적으로 스레드 안전하여 따로 동기화 할 필요 없음
- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많음
    
    : 아무리 구조가 복잡하더라도 바뀌지 않는 구성요소들로 이뤄져 있다면 불변식을 유지하기 수월함
    
- 단점으로는 값이 다르면 반드시 독립된 객체로 만들어야 함
    
    → 중간 단계에서 만들어지는 객체가 많고 모두 버려진다면 성능 문제가 커질 수 있음
    
    → 예측하여 다단계 연산을 기본 기능으로 제공 eg) `StringBuilder`
    

## 아이템 18. 상속보다는 컴포지션을 사용하라

- 구체 클래스가 다른 구체 클래스를 확장하는 구현 상속은 위험함
    
    : 메서드 호출과 달리 상속은 캡슐화를 깨뜨리기 때문
    
- 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게하자
- 기본 클래스가 새로운 클래스의 구성요소로 쓰인다는 점 → 컴포지션 구성이라 일컬음
    
    ```java
    public class InstrumentedSet<E> extends ForwardingSet<E> {
        private int addCount = 0;
    
        public InstrumentedSet(Set<E> s) {
            super(s); // 생성자는 Set을 매개변수로 받아 부모 클래스의 생성자를 호출
    				// ForwardingSet 클래스의 생성자를 호출하여 내부에서 유지하는 Set 인스턴스를 초기화
        }
    }
    ```
    
- 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 사용하자
- 참고
    - `List.of`: 반환된 리스트는 불변(immutable). 즉, 생성된 후에는 요소를 추가, 제거 또는 변경할 수 없고 `null` 요소를 허용하지 않음
    - `Arrays.asList`: 반환된 리스트는 가변(mutable). 이는 리스트의 크기를 변경할 수 있고, 원소를 변경할 수 있음

## 아이템 19. 상속을 고려해 설계하고 문서화하라. 그러지않았다면 상속을 금지하라

- 상속용 클래스를 설계하는 방법에 대해 - 다시 보기

## 아이템 20. 추상 클래스보다는 인터페이스를 우선하라

- 자바가 제공하는 다중 구현 메커니즘: 인터페이스와 추상 클래스
    - 인터페이스: 모든 메소드가 추상 메소드인 경우
        
        함수의 껍데기만 있는데, 그 이유는 그 함수의 구현을 강제하기 위한 목적
        
        해당 인터페이스를 구현한 객체들에 대해서 동일한 동작을 약속하기 위해 존재
        
    - 추상클래스: 클래스 내 '추상 메소드'가 하나 이상 포함되거나 abstract로 정의된 경우
        
        추상 클래스를 상속 받아서 기능을 이용하고, 확장시키는 데 목적
        
        상속은 슈퍼클래스의 기능을 이용하거나 확장하기 위해서 사용되고, 다중 상속의 모호성 때문에 하나만 상속받을 수 있음
        

추상클래스와 인터페이스 차이점에 대해 이야기

- 인터페이스는 믹스인 정의에 적절함
    
    새로운 인터페이스는 클래스에 쉽게 넣을 수 있지만, 새로운 추상클래스를 끼워넣으려면 계층구조가 복잡(이미 어떤 클래스를 상속받고있는데, 또 상속받아야하니)해지기 때문
    
- 인터페이스 메서드 중 구현 방법이 명백한 것이 있다면 구현을 디폴트 메서드로 제공해 편의성을 높일 수 있음
    
    ```java
    default boolean removeIf(Predicate<? super E> filter({
    	Objects.erquireNonNull(filter);
    	boolean result = false;
    	...
    	return result;
    }
    ```
    
- 인터페이스와 추상 골격 구현 클래스를 함께 제공하여 인터페이스와  추상클래스의 장점을 모두 취할 수 있음

eg) [https://velog.io/@gale4739/Spring-Boot-Interface-골격-구현-클래스-클래스-구조-변경Feat.-Composition](https://velog.io/@gale4739/Spring-Boot-Interface-%EA%B3%A8%EA%B2%A9-%EA%B5%AC%ED%98%84-%ED%81%B4%EB%9E%98%EC%8A%A4-%ED%81%B4%EB%9E%98%EC%8A%A4-%EA%B5%AC%EC%A1%B0-%EB%B3%80%EA%B2%BDFeat.-Composition)

인터페이스 - 골격 구현 클래스 - 클래스 구조

jwt 토큰 필터를 다음의 3단계로 나누어 BaseFilter에서는 필터 클래스에서 공통적으로 사용하는 메소드를 정의하고, AbstractFilter에서는 OncePerRequestFilter를 상속받고 BaseFilter를 구현하는 골격 구현 클래스를 정의

```java
public interface BaseFilter {
    /**
     * 실제 필터 로직 수행하는 메소드
     * @param request
     * @param response
     */
    void doFilterLogic(HttpServletRequest request, HttpServletResponse response);

    /**
     * FilterExceptionHandler getter
     * @return
     */
    FilterExceptionHandler getFilterExceptionHandler();
}
```

```java
public abstract class AbstractFilter extends OncePerRequestFilter implements BaseFilter {
    /**
     * 실제 필터 로직 수행하는 메소드 구현
     * @param request
     * @param response
     * @param filterChain
     * @throws IOException
     */
    @Override
    protected void doFilterInternal(
            @NonNull HttpServletRequest request,
            @NonNull HttpServletResponse response,
            @NonNull FilterChain filterChain
    ) throws IOException {
        try{
            doFilterLogic(request,response);
            filterChain.doFilter(request,response);
        }
        catch (ValidationException e){
            logger.error(e.getMessage());
            getFilterExceptionHandler().setErrorResponse(e.getCode(),e.getMessage(),response);
        }
        catch (Exception e){
            logger.error(e.getMessage());
            getFilterExceptionHandler().sendErrorToSlack(request,response,e);
        }
    }
}
```

```java
@Component
@RequiredArgsConstructor
public final class JwtFilter extends AbstractFilter implements BaseFilter {
    private final FilterExceptionHandler filterExceptionHandler;
    private final UriProvider uriProvider;
    private final TokenProvider tokenProvider;

    @Override
    public void doFilterLogic(HttpServletRequest request, HttpServletResponse response) {
        // 2. 헤더의 토큰이 존재하는지 체크
        logger.info("2. 토큰 유효성 검사");

        // 토큰이 필요 없는 API는 패스
        String uri = uriProvider.getURI(request);
        if(!uriProvider.isValidationPass(uri)){
            String jwt = tokenProvider.resolveToken(request, TokenProvider.HEADER_NAME);

            // 토큰 유효성 검증 후 SecurityContext에 저장
            Claims claims = tokenProvider.getAuthenticationClaims(jwt);
            Authentication authentication = tokenProvider.getAuthentication(jwt, claims);
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
    }

    @Override
    public FilterExceptionHandler getFilterExceptionHandler() {
        return filterExceptionHandler;
    }
}
```

골격 구현 클래스 이야기

## 아이템 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

- 자바 8 이전에는 인터페이스에 새로운 메소드를 추가할 경우 보통 컴파일 오류; 구현 클래스들에서 구현하지 않았기 때문
- 인터페이스에 default 메소드, static 메소드가 등장하면서 새로운 메소드를 모든 구현클래스의 도움없이 추가하는 방법이 생기긴 하였으나, 모든 상황에서 불변식을 해치지 않는 default 메소드를 작성하는 것은 매우 어려움
- 인터페이스를 릴리스한 후라도 결함을 수정하는 게 가능할수도 있지만, 인터페이스를 설계하는 처음부터 세심한 주의를 기울이자

## 아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

- 인터페이스: 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할
    
    → 클라이언트에게 자신의 인스턴스로 무엇을 할 수 있는지 알려주기 위한 용도
    
- 상수 인터페이스: 목적에 맞지 않는 인터페이스 예시 → 사용 금지이다
    
    ```java
    public interface PhysicalConstants {
    
        static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
        static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
        static final double ELECTRON_MASS      = 9.109_383_56e-31;
    }
    ```
    
    - 특정 클래스나 인터페이스에 연관된 상수라면 → 그 자체에 추가해야함
        
        ```java
        private static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
        private static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
        private static final double ELECTRON_MASS      = 9.109_383_56e-31;
        ```
        
        열거 타입으로 나타내기 적합한 상수라면 → 열거 타입으로 만들어 공개
        
        ```java
        public enum PhysicalConstants {
            AVOGADROS_NUMBER(6.022_140_857e23),
            BOLTZMANN_CONSTANT(1.380_648_52e-23),
            ELECTRON_MASS(9.109_383_56e-31);
        
            private final double value;
        
            PhysicalConstants(double value) {
                this.value = value;
            }
        
            public double getValue() {
                return value;
            }
        }
        ```
        
        아니라면 → 유틸리티 클래스에 담아 공개/ 빈번하게 사용 시, 정적 임포트하여 사용
        
        ```java
        public class ConstantsUtil {
        		private ConstantsUtil() {} // 인스턴스화 방지
            public static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
            public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
            public static final double ELECTRON_MASS      = 9.109_383_56e-31;
        }
        ```
        

## 아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

- 태그 달린 클래스: 장황하고, 오류 내기 쉽고, 비효율적
- 클래스 계층 구조: 추상클래스로, 인터페이스로 구조 구현 가능
    
    ```java
    abstract class Figure {
        abstract double area();
    }
    class Circle extends Figure {
        final double radius;
    
        Circle(double radius) { this.radius = radius; }
    
        @Override double area() { return Math.PI * (radius * radius); }
    }
    class Rectangle extends Figure {
        final double length;
        final double width;
    
        Rectangle(double length, double width) {
            this.length = length;
            this.width  = width;
        }
        @Override double area() { return length * width; }
    }
    ```
    

## 아이템 24. 멤버 클래스는 되도록 static으로 만들자

- 중첩 클래스: 다른 클래스 안에 정의된 클래스(정적 멤버 클래스, 비정적 멤버 클래스, 익명 클래스, 지역 클래스)
- 정적 멤버 클래스
    
    ```java
    public class OuterClass {
        static class StaticNestedClass { }
    }
    ```
    
- 비정적 멤버 클래스
    
    ```java
    public class OuterClass {
        class InnerClass { }
    }
    ```
    
    - 바깥 인스턴스 클래스와 암묵적으로 연결되어 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있음
    - 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자

## 아이템 25. 톱레벨 클래스는 한 파일에 하나만 담으라

- 소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 문제 삼지 않음
- 하지만 다른 파일에 똑같은 클래스명으로 클래스 2개가 정의되어 있는 경우 같을 때 컴파일 순서에 따라 문제가 생길 수 있음
- 톱레벨 클래스들은 서로 다른 소스 파일로 분리하자!