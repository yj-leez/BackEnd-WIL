# Ch9 단위 테스트

### TDD 법칙 세 가지

- 첫째; 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
- 둘째; 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
- 셋째; 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

### 깨끗한 테스트 코드 유지하기

**테스트는 유연성, 유지보수성, 재사용성을 제공한다.**

- 테스트 케이스가 있으면 변경이 쉬워진다. 코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이 단위 테스트이기 때문이다.

### 그래서 깨끗한 테스트 코드를 만드려면?

- **가독성**이 제일 중요하다. 어쩌면 실제 코드보다 테스트 코드에 가독성은 더 중요할 수 있다.

```java
public void testGetPageHierarchyAsWml() throws Exception {
	makePages("PageOne", "PageOne.ChildOne", "PageTwo");

	submitRequest("root", "type:pages");

	assertResponseIsXML();
	assertResponseContains(
		"<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
	);
}
```

- BUILD-OPERATE-CHECK 패턴이 위와 같은 테스트 구조에 적합하다.
    - BUILD : 테스트 자료를 만든다.
    - OPERATE : 테스트 자료를 조작한다.
    - CHECK : 조작한 결과를 확인한다.
    - given-when-then 을 관례적으로 주석으로 사용한다.
- 또한 테스트에 진짜 필요한 자료 유형과 함수만 사용하고 나머지는 모두 뺀다.

### 도메인에 특화된 테스트 언어 DSL

### 이중 표준

- 테스트 코드는 단순하고, 간결하고, 표현력이 풍부해야 하지만, 실제 코드만큼 효율적일 필요는 없다. 실제 환경이 아니라 테스트 환경에서 돌아가는 코드이기 때문이다. `이중 표준`이라는 말은 원래 코드와 테스트 코드의 표준은 즉 구현 방식은 다르다는 것이다. 실제 환경에서는 절대 안되지만 테스트 환경에서는 문제없는 방식이 있다.
- 책의 예에서는, 환경 제어 시스템이 반환하는 상태를 읽기 좋은 테스트 코드로 만든다. 결과를 "hBCHl" 과 같은 형태로 받아서 쓰는 것은 원래 코드에서는 있어서는 안되는 일이다.하지만 테스트 코드에서 이렇게 작성하게 된다면, 가독성이 크게 높아진다.
    
    ```java
    @Test
    public void turnOnLoTempAlarmAtThershold() throws Exceptioin{
    	wayToocold();
    	assertEquals("HBchL", hw.getState());
    }
    ```
    
    ```java
    public String getState(){
    	String state = "";
    	state += heater? "H": "h";
    	state += blower? "B": "b";
    	... 생략 ...
    }
    ```
    
    - Mock 객체를 만들어 getState() 를 정의하고 위와 같은 형태로 비교하는 테스트로 리펙토링 할 수 있다.

### 테스트 당 assert 하나

- assert 문이 단 하나인 함수는 결론이 하나라서 코드를 이해하기 쉽고 빠르다.

```java
public void testGetPageHierarchyAsWml() throws Exception {
	givenPages("PageOne", "PageOne.ChildOne", "PageTwo");

	whenRequestIsIssued("root", "type:pages");

	thenResponseShouldBeXML();
}
```

- 위에선 함수 이름을 바꿔 *given-when-then* 관례를 사용했다. 그러면 테스트 코드를 읽기가 쉬워진다.하지만 테스트를 분리하면 중복되는 코드가 많아진다.
- 단일 assert문이라는 규칙은 훌륭하다. 하지만 때로는 주저 없이 함수 하나에 여러 assert문을 넣어도 된다.

### 테스트 당 개념 하나

- 테스트 함수는 개념 하나만 테스트 하라

### F.I.R.S.T

- **빠르게(Fast)**: 테스트는 빨라야한다. 테스트가 느리면 자주 돌릴 엄두를 못 내고 자주 돌리지 않으면 초반에 문제를 찾아내지 못한다.
- **독립적으로(Independent)**: 각 테스트는 서로 의존하면 안 된다. 각 테스트는 독립적으로 그리고 어떤 순서로 실행해도 괜찮아야 한다.
- **반복가능하게(Repeatable)**: 테스트는 어떤 환경에서도 반복 가능해야 한다. 테스트가 돌아가지 않는 환경이 하나라도 있다면 테스트가 실패한 이유를 둘러댈 변명이 생긴다.
- **자가검증하는(Self-Validating)**: 테스트는 bool값으로 결과를 내야 한다. 통과 여부를 알려고 로그 파일을 읽게 만들어서는 안 된다.
- **적시에(Timely)**: 테스트는 적시에 작성해야 한다. 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다.

### 리팩토링하며 적용하자

- 사실 이미 진행된 프로젝트이기 때문에 지금 당장 모든 로직에 대해 단위 테스트를 작성하기엔 어려움이 있다. 동시성 테스트를 위해 이미 작성된 테스트 로직이 Repeatable 규칙을 어김으로, 규칙에 맞게 테스트를 수정하였다.

### Before

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
        assertThat(user1.getNickname()).isNotEqualTo(user2.getNickname());
}
```

### After

- 맨 처음에 BeforeEach로 데이터를 삽입하고 테스트를 실행하면 데이터가 조회될거라 생각했으나, 새롭게 생성한 스레드에서 조회 시 실패하였다.
- 찾아보니, BeforeEach부터 테스트 코드가 끝날 때까지 삽입한 데이터는 실제 DB에 커밋되지 않으며, 트랜잭션이 다른 상태에서 커밋되지 않은 데이터를 조회하는 게 불가능하기 때문이었다.
- 따라서 스프링 부트를 시작하는 시점에 테스트용 데이터를 넣는 방법으로 수정하였다.

- 하지만 설정 사소한 에러가 많아 많은 삽질을 하게되었다..^^
- 먼저 build.gradle에 테스트 실행 시에만 H2 데이터베이스 라이브러리를 런타임 의존성으로 사용하도록 지정하는 것이다.

```sql
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

runtimeOnly 'com.mysql:mysql-connector-j'

testImplementation 'org.springframework.boot:spring-boot-starter-test'
testRuntimeOnly 'com.h2database:h2'
```

- 그리고 test 프로필이 설정될 때 활성화될 구성 파일을 다음과 같이 작성하였다.

```java
spring:
  config:
    activate:
      on-profile: test
  h2:
    console:
      enabled: true
  jpa:
    database: h2
    generate-ddl: off
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    hibernate:
      ddl-auto: none
    properties:
      hibernate:
        globally_quoted_identifiers: true
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:test;MODE=MySQL;
    username: SA
    password:
    initialization-mode: always
    schema: classpath:schema.sql
    data: classpath:data.sql
```

- `MODE=MySQL;` 과 `org.hibernate.dialect.MySQL5InnoDBDialect` 은 H2 데이터베이스에 MySQL문법을 날려도 변환하여 쿼리 실행이 되도록 한다. MySQL5버전, InnoDB 스토리지 엔진을 사용하여 진행한다는 뜻이다.
- `jdbc:h2:mem:test;` : 어플리케이션이 동작할 때만 존재하는 In-memoryDB임을 알려준다.

- 또한 yml 파일에서 `ddl-auto: none` 으로 hibernate가 table을 자동 생성하는 걸 방지한 후 `schema: classpath:schema.sql` 으로 schema.sql에서 직접 쿼리를 날려 table을 생성하도록 지정하였다.

```sql
create table `user`(
    user_id bigint primary key,
    role varchar(50),
    auth_key varchar(30),
    nickname varchar(30),
    review_cnt integer,
    is_del tinyint default 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    modified_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

- 테스트에 사용하고자 하는 테이블 create 쿼리를 만들어 넣어주었고, user가 H2의 예약어이기 때문에 백틱으로 감싸주었다.

- 블로그마다 말이 다 달라 헷갈렸는데 `@DataJpaTest` 가 아닌 `@SpringBootTest` 를 하였을 때 테스트를 성공할 수 있었다.

```sql
@ActiveProfiles("test")
@SpringBootTest
@Transactional
class UserServiceTest {

		...

}
```

### @DataJpaTest vs @SpringBootTest

- `@DataJpaTest`는 오직 JPA 컴포넌트들만을 테스트하기 위한 어노테이션으로, full-auto config를 해제하고 JPA 테스트와 연관된 config만 적용한다.
- `@DataJpaTest`가 포함하고 있는 어노테이션을 확인해보면 `replace=AutoConfigureTestDatabase.Replace`가 디폴트로 설정되어 있기 때문에 내가 설정해놓은 DB가 아닌 in-memory DB를 활용해서 테스트가 실행된다.
- 위에서 언급했듯, JPA 레포지토리 관련 테스트를 위해 사용하기때문에 UserSevice를 주입받아 사용해야하는 이번 테스트에서는 빈을 찾을 수 없다고 계속해서 에러가 떴었다.

- 반면 `@SpringBootTest`는 full application config을 로드해서 통합 테스트를 진행하기 위한 어노테이션이다.
- DataSource bean을 그대로 사용하기 때문에 in-memory, 로컬, 외부 상관 없이 내가 지정한 DB를 사용해서 테스트가 실행된다.
- 다만, 테스트할 때마다 DB가 롤백되지 않기 때문에 `@Transactional`을 추가로 달아주어야 한다.