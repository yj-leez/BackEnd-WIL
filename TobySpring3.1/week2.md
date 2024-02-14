# 토비의 스프링 2주차

## 의존관계 주입

### 제어의 역전(IoC)와 의존관계 주입

- 제어의 역전(IoC): 객체를 직접 생성하는게 아니라 외부에서 생성한 후 주입시켜주는 방식, 스프링은 의존관계 주입뿐만 아니라, 의존관계 검색(dependency lookup)도 존재한다
- 스프링에서  애플리케이션 컨텍스트는 의존관계 검색을 사용하기위해 getBean()메서드를 제공한다. 검색하는 오브젝트(아래의 예시에서는 UserDao)는 자신이 스프링 빈일 필요가 없다.

    ```java
    /* 의존관계 검색을 이용하는 UserDao 생성자 */
    public UserDao(){
    	AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
    	this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
    }
    ```

- 의존관계 주입(DI): 스프링이 다른 프레임워크와 차별화되어 제공하는 IoC 방식으로 DI는 IoC를 실행한 개념이라 볼 수 있다

### 메서드를 이용한 의존관계 주입

- 생성자 메서드를 이용한 주입: 생성자 호출 시점에 딱 1번만 호출되는 것이 보장되며, 불변, 필수 의존관계에 사용된다.

    ```java
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy
      discountPolicy) {
              this.memberRepository = memberRepository;
              this.discountPolicy = discountPolicy;
    }
    ```

    - 생성자가 딱 1개만 존재할 시, @Autowired를 생략해도 자동주입된다.
- 수정자 메서드를 이용한 주입: 외부에서 오브젝트 내부의 애트리뷰트 값을 변경하려할 때 주로 사용된다. 파라미터로 전달된 값을 내부의 인스턴스 변수에 저장한다. 외부로부터 제공받은 오브젝트 레퍼런스를 저장해뒀다가 내부의 메서드에서 사용하게 하는 DI 방식에서 사용하기 적합하다.

    ```java
    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
                this.memberRepository = memberRepository;
    }
    @Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
                this.discountPolicy = discountPolicy;
    }
    ```

- 필드 주입: 간결하지만 외부에서 변경이 불가능해 테스트코드나 Configuration에서만 주로 사용한다

    ```java
    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private DiscountPolicy discountPolicy;
    ```

- 일반 메서드 주입: 수정자 메서드와 다르게 한 번에 여러 파라미터를 받을 수 있다.

    ```java
    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;
    @Autowired
    public void init(MemberRepository memberRepository, DiscountPolicy
        discountPolicy) {
                this.memberRepository = memberRepository;
                this.discountPolicy = discountPolicy;
    }
    ```

- 대부분의 의존관계 주입은 한번 일어나면 애플리케이션 종료시점까지 의존관계를 변경할 일이 없어서 final을 붙이고, 생성자 주입을 사용한다. 따라서 롬복의 @RequiredArgsConstructor을 이용하여 간단하게 최적화하여 사용하자

## 빈 스코프

### 싱글톤 스코프

- 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
- 하나의 빈을 여러 개의 빈에서 DI하더라도 매번 동일한 오브젝트가 주입

### 프로토타입 스코프

- 스프링이 관리하는 빈은 의존관계 주입, 초기화, DI와 DL을 통한 사용, 제거까지 모든 오브젝트의 생명주기를 컨테이너가 관리하지만

    프로토타입 빈은 독특하게 이 IoC의 기본 원칙을 따르지 않는다. 생성, 초기화, DI(의존관계 주입)까지만 제공한다.

- 빈 오브젝트의 관리는 전적으로 DI 받은 오브젝트에 의존한다. 종료 메서드에 대한 호출도 클라이언트가 직접 해야한다.
- 사용자의 요청마다 독립적으로 오브젝트를 생성하려는 경우 도메인 오브젝트나 DTO를 new 키워드로 생성하여 파라미터로 전달하여 사용할 수 있다.

    하지만 이러한 방식을 통해서는 DI를 사용할 수 없다. 요청에 따라 독립적인 오브젝트를 코드가 아닌 컨테이너를 통해 오브젝트를 생성하고 초기하여 DI를 하기 위해 프로토타입 빈을 사용한다.


예시) 콜센터 고객의 A/S 신청 접수된 내용을 DB에 저장하고 신청한 고객에게 이메일을 발송하는 로직

dao)

```java
public class ServiceRequest {
		String customerNo;
		String productNo;
		String description;
		...
}
```

컨트롤러 계층)

```java
public void serviceRequestFormSubmit(HttpServletRequest request) {
		ServiceRequest serviceRequest = new ServiceRequest();  //매번 ServiceRequest객체를 생성
		serviceRequest.setCustomerNo(request.getParameter("custno"));
		...
		this.serviceRequestService.addNewServiceRequest(serviceRequest);
		...
}
```

서비스 계층)

```java
public void addNewServiceRequest(ServiceRequest serviceRequest) {
		Customer customer = this.customerDao.findCustomerByNo(
				serviceRequest.getCustomerNo());
		...
		this.serviceRequestDao.add(serviceRequest, customer);

		this.emailService.sendEmail(customer.getEmail(),
				"A/S 접수가 정상적으로 처리되었습니다.");
}
```

- 단점: ServiceRequest가 모든 계층의 코드와 강하게 결합되어 확장이나 유지보수 시 어려움이 있고, 테스트 시 용이하지 않다.

↓ 오브젝트 중심의 구조로 변경하기

dao)

```java
@Component
@Scope("prototype")
public class ServiceRequest{
    Customer customer;
    String productNo;
    String description;
    @Autowired CutomerDao cutomerDao; //DB

    public void setCustomerByCustomerNo(String customerNo){
        this.customer = customerDao.findCustomerByNo(customerNo);
    }

		public void notifyServiceRequestRegistration() {
				if(this.customer.serviceNotificationMethod == NotificationMethod.EMAIL) {
						this.emailService.sendEmail(customer.getEmail(),
								"A/S 접수가 정상적으로 처리되었습니다.");
		}
}
```

컨트롤러 계층)

```java
@Autowired ApplicationContext context;

public void serviceRequestFormSubmit(HttpServletReqeust request){
    ServiceRequest serviceRequest = this.context.getBean(ServiceRequest.class);
    serviceRequest.setCustomerByCustomerNo(request.getParameter("custno"));
		...
		this.serviceRequestService.addNewServiceRequest(serviceRequest);
		...
}
```

서비스 계층)

```java
public void addNewServiceRequest(ServiceRequest serviceRequest) {
		this.serviceRequestDao.add(serviceRequest);
		serviceRequest.notifyServiceRequestRegistration();
}
```

- 이와 같이 매번 새롭게 오브젝트를 생성하면서도 DI를 적용할 수 있다.

    고급 AOP 기능을 사용하면 프로토타입 빈을 이용하지 않고 new 키워드로 생성하더라도 DI가 되도록 할 수 있다.


*참고: ApplicationContext를 이용해 getBean()을 호출하고 있으므로 DL 방식 사용하고 있으며, new 키워드를 대신하기 위해 프로토타입의 용도로 쓴다면 DI보단 DL방식을 사용하길 권장한다.
