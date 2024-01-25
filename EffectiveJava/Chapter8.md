# 8장. 메서드

## 아이템 49. 매개변수가 유효한지 검사하라

- 매개변수는 메서드 몸체가 실행되기 전에 확인해야 함
    - 매개변수 검사를 제대로 하지 못한다면 수행 도중 모호한 예외를 던지며 실패하는 등의 문제가 생김
- public protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야함
    - 아래 메서드는 m이 null이면 NullPointerException을 제공함
        
        아래 코드에선 찾을 수 없는데, Biginteger 클래스 단에서 기술했기 때문
        
        → 클래스 수준 주석은 그 클래스의 모든 public 메서드에 적용되므로 각 메셔드에 일일이 기술하는 것보다 훨씬 깔끔한 방법
        
    
    ```java
    public BigInteger mod(BigInteger m) {
        if(m.signum() <= 0)
            throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
    }
    ```
    
    - java.until.Objects.requireNonNull 메서드로 null 검사를 용이하게 할 수 있음
- private 메서드는 단언문(assert)을 이용하여 매개변수 유효성을 검증할 수 있음
    
    ```java
    private static void sort(long a[], int offset, int length) {
        assert a != null;
        assert offeset >= 0 && offset <= a.length;
    }
    ```
    

**메서드나 생성자를 작성할 때면 매개변수들에 어떤 제약이 있을지 생각해야한다 그 제약들을 문서화하고 메서드 코드 시작 부분에서 명시적으로 검사해야한다**

## 아이템 50. 적시에 방어적 본사본을 만들라

```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    if( this.start.compareTo(this.end) > 0) 
        throw new IllegalArgumentException(this.start + " after " + this.end);
}

public Date start() {
    return new Date(start.getTime());    
}

public Date end() {
    return new Date(end.getTime()):
}
```

- 방어적 복사를 활용하여 객체의 허락 없이 외부에서 내부를 수정하는 일이 불가능하게끔 하여 완벽한 불변으로 설계하라
    - 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사
    - 접근자가 단순히 가변 필드의 방어적 복사본을 반환하도록 설계

## 아이템 51. 메서드 시그니처를 신중히 설계하라

- 메서 이름을 신중히 짓자
- 편의 메서드를 너무 많이 만들지 말자
- 매개변수 목록은 짧게 유지하자
- 매개변수의 타입으로는 클래스보다는 인터페이스가 더 낫다
    
    eg) `HashMap`을 직접 넘기기 보다는 `Map`을 넘기자
    
- boolean보다는 원소 2개짜리 열거 타입이 낫다

## 아이템 52. 다중정의는 신중히 사용하라

- 다중정의된 메서드들 중 어느 메서드를 호출할지는 `컴파일` 타임에 정해짐
- 메서드를 재정의했다면 해당 객체의 `런타임` 타입이 어떤 메서드를 호출할 지의 기준이 됨
- 다중정의가 혼동을 일으키는 상황을 피해야하므로 매개변수 수가 같은 다중정의는 만들지 않는 것이 좋음
    - 다중정의대신 메서드 이름을 다르게 지어주는 방법도 존재
- 생성자도 이름을 다르게 지을 수 없으니 두 번째 생성자부터는 무조건 다중정의가 됨
    
    → 정적 팩터리라는 대안을 사용할 수 있음
    

## 아이템 53. 가변인수는 신중히 사용하라

- 가변인수 메서드를 `static int sum(int… args)`와 같이 사용한다면 인수를 0개만 넣었을 때 런타임에서 실패함
    
    → 이를 방지하기 위해 `static int min(int firstArg, int… remainingArgs)`와 같이 사용할 수 있음
    
    → 하지만 이 방법도 메서드를 호출할 때마다 배열을 새로 하나 할당하고 초기화함
    
    → 인자를 4개까지 받을 때는 매개변수의 개수가 다른 다중정의 메서드를 사용하도록 함으로써 가변인수 메서드 호출을 줄일 수 있음
    

## 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

- 추가적인 오류 처리 코드가 필요하므로 컬렉션이 비어있을 때 null을 반환하기보다는 빈 불변 컬렉션을 반환해라

```java
public List<Cheese> getCheese(){
	return cheesesInStock.isEmpty() ? Collectors.emptyList()
		: new ArrayList<>(cheeseInStock);
}
```

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheese(){
	return cheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

## 아이템 55. 옵셔널 반환은 신중히 하라

- Optional<T> : null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있음
- 빈 컬렉션을 반환하기보다는 Optional<E>를 반환하는 편이 나음
    
    Optional.empty() 와 Optional.of(result) 두 정적 팩터리를 이용하여 반환
    
    *주의 of() 메서드에 null을 넣으면 NullPointerException을 던지니 주의
    
- 옵셔널을 반환하면 클라이언트는 값을 받지 못했을 때 취할 행동을 정의할 수 있음
    1. 기본값 설정
        
        ```java
        String lastWordInLexicon = max(words).orElse("단어 없음");
        ```
        
    2. 원하는 예외 던지기
        
        ```java
        Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
        ```
        
        *예외 팩터리를 건넴 → 예외가 실제로 발생하지 않는 한 예외 생성 비용이 들지 않음
        
        람다 표현식으로 예외를 생성하는 경우, 예외 객체가 생성되기 전에 람다 표현식이 실행
        
    3. 항상 값이 채워져 있다고 가정
        
        .get()으로 바로 꺼내 사용하면 되지만 잘못 판단한 경우 NoSuchElementException이 발생
        
- 스트림을 사용한다면 채워진 옵셔널들에서 값을 뽑아 Stream<T>에 건네 담아 처리하는 경우가 드물지 않음

```java
streamOfOptionals
	.filter(Optional::isPresent)
	.map(Optional::get)

streamOfOptionals
	.flatMap(Optional::stream)
```

- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안됨
    - Optional<List<T>>를 반환하기보다는 빈 List<T>를 반환하자

**값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야하는 메서드라면 옵셔널을 반환해야 할 상황일 수 있음**

— 적용 예시 → 훨씬 더 깔끔해졌음

```java
boolean existsByKorName(String korName);
Family findByKorName(String korName);

// 존재하는지 확인 후 조회
if(!familyRepository.existsByKorName(family)) throw new CustomException(ErrorCode.FAMILY_NOT_FOUND);
Family find = familyRepository.findByKorName(family);
```

↓ 수정 후

```java
Optional<Family> findByKorName(String korName);

Family find = familyRepository.findByKorName(family)
		.orElseThrow(() -> new CustomException(ErrorCode.FAMILY_NOT_FOUND));
//		.orElseThrow(CustomException(ErrorCode.FAMILY_NOT_FOUND)::new);  //매개변수 있을 때는 이렇게 생성자 메서드 참조 사용 못함
```

## 아이템 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라