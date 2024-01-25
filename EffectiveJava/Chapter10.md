# 10장. 예외

## 아이템 69. 예외는 진짜 예외 상황에만 사용하라

- 예외는 오직 예외 상황에서만 써야한다. 절대 일상적인 제어 흐름용으로 쓰지 말 것

## 아이템 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

### 오류(Error)

- 시스템에 비정상적인 상황이 생겼을 때 발생
- 시스템 레벨에서 발생하기 때문에 심각한 수준의 문제상황

### 예외(Exception)

- 개발자가 구현한 로직에서 발생
- 예외는 발생할 상황을 미리 예측하여 처리할 수 있음
- 예외는 개발자가 처리를 할 수 있기 때문에 예외를 구분하고, 그에 따라 처리 방법을 명확히 알고 사용하는 것이 중요함

### 검사 예외(Checked Exception) vs 런타임 예외(Unchecked Exception)

| 구분 | Checked Exception | Unchecked Exception |
| --- | --- | --- |
| 확인 시점 | 컴파일 시점 | 런타임 시점 |
| 처리 여부 | 반드시 예외 처리해야함 | 명시적으로 하지 않아도 됨 |
| 트랜잭션 처리 | 예외 발생 시 롤백하지 않음 | 예외 발생 시 롤백함 |
| 종류 | IOException, ClassNotFoundException | NullPointerException,ClassCastException |
- RuntimeException 클래스들은 주로 프로그래머의 실수에 의해 발생될 수 있는 예외들임
    
    → 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하고 비검사 throwable을 구현할 때는 RuntimeException의 하위 클래스여야함
    
- 배열의 범위를 벗어난다던가(ArrayOutOfBoundsException), 값이 null인 참조변수의 멤버를 호출하려 했다던가(NullPointerException), 클래스간의 형변환을 잘못했다던가(ClassCastException), 정수를 0으로 나누려고(ArithmeticException)하는 경우에 발생

![Untitled](https://github.com/yj-leez/Java-WIL/assets/77960090/3bb05f53-4993-44c1-af82-6a8a705b9679)

- 복구 가능하다고 믿는다면 검사 예외를, 그렇지 않다면 런타임 예외를 사용하자
- 확신하기 어렵다면 비검사 예외를 선택하는 편이 나을 것이다(아이템 71)
- 검사 예외라면 복구에 필요한 정보를 알려주는 메서드도 제공하자

## 아이템 71. 필요 없는 검사 예외 사용은 피하라

- 꼭 필요한 곳에만 사용한다면, 검사 예외는 프로그램의 안정성을 높여줌 하지만 남용하면 쓰기 어려운 API 를 낳을 수 있음
- 검사 예외를 피하는 방법으로 빈 옵셔널을 반환하는 방법이 있음
- 예시
    
    ```java
    Long userId = 1L;
    try {
       User user = userService.findUserById(userId);
    } catch (Exception e) {
        log.error("fail to find user, userId: {}", userId)
        user = new User(userId);
    }
    ```
    
    ↓ 수정 후
    
    ```java
    Long userId = 1L;
    
    User user = repository.findUserById(userId)
                    .orElseGet(() -> new User(userId));
    ```
    
- 옵셔널만으로는 상황을 처리하기에 충분한 정보를 제공할 수 없을 때만 검사 예외를 던지자
- 예외 상황에서 복구할 방법이 없다면 비검사 예외를 던지자

## 아이템 72. 표준 예외를 사용하라

| 예외 | 주요 쓰임 |
| --- | --- |
| IllegalArgumentException | 허용하지 않는 값이 인수로 건네졌을 때 |
| IllegalStateException | 객체가 메서드를 수행하기에 적절하지 않은 상태일 때 |
| NullPointerException | null을 허용하지 않는 메서드에 null을 건넸을 때 |
| IndexOutOfBoundsException | 인덱스가 범위를 넘어섰을 때 |
| ConcurrentModificationException | 허용하지 않는 동시 수정이 발견됐을 때 |
| UnsupportedOperationException | 호출한 메서드를 지원하지 않을 때 |
- 상황이 부합한다면 항 상 표준 예외를 재사용하자 또한 더많은 정보를 제공하길 원한다면 표준예외를 확장해도 좋음
- 하지만 예외는 직렬화할 수 있다는 사실을 기억하자

## 아이템 74. 메서드가 던지는 모든 예외를 문서화하라

- 공통 상위 클래스 하나로(eg. Exception) 뭉뚱그려 선언하지 말자
- 메서드가 던질 가능성이 있는 모든 예외를 @throws 태그로 문서화하되 비검사 예외는 메서드 선언의 throws 목록에 넣지 말자

```java
 /**
     * @throws SQLException SQL 이 잘못된 경우
     * @throws ClassNotFoundException 지정한 경로에 클래스파일이 존재하지 않는경우.
     * @throws NullPointerException 지정한 요소에 null 이 들어오는 경우
     */
    public void examMethod() throws SQLException,ClassNotFoundException{

    }
```

## 아이템 75. 예외의 상세 메시지에 실패 관련 정보를 담으라

- 발생한 예외와 관련된 모든 매개변수와 필드의 값을 실패 메세지에 담아야 함
- 필요한 정보들을 예외 생성자에서 모두 받아서 상세 메시지까지 미리 생성해놓는 방법도 괜찮음

```java
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index){

	// 실패를 포착하는 상세 메세지를 생성한다
	super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));

	// 프로그램에서 이용할 수 있도록 실패 정보를 저장해둔다
	this.lowerBound = lowerBound;
	this.upperBound = upperBound;
	this.index = index;

}
```

## 아이템 76. 가능한 한 실패 원자적으로 만들라

- 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야함: 실패 원자적이다
- 메서드를 실패 원자적으로 만들기
    1. 불변 객체로 설계
    2. 작업 수행에 앞서 매개변수의 유효성을 검사
    3. 작업 도중 발생하는 실패를 가로채는 복구 코드를 작성하여 작업 전 상태로 되돌리는 방법
