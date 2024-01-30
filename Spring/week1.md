# Spring 다시 정리 1주차

## 스프링과 객체 지향

- 사용자 정보를 DB에 넣고 관리할 수 있는 DAO 클래스를 만들어보자

```java
public class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
				Class.forName("org.postgresql.Driver");

        String user = "postgres";
        String password = "password";

        Connection c = DriverManager.getConnection(
                "jdbc:postgresql://localhost/toby_spring"
                , user
                , password
        );

        // 2. SQL 문장을 담을 Statement를 만들고 실행하기
        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values (?, ?, ?)"
        );
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        // 3. 리소스 반납하기
        ps.close();
        c.close();
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
				Class.forName("org.postgresql.Driver");

        String user = "postgres";
        String password = "password";

        Connection c = DriverManager.getConnection(
                "jdbc:postgresql://localhost/toby_spring"
                , user
                , password
        );

        // 2. SQL 문장을 담을 Statement를 만들고 실행하기
				PreparedStatement ps = c.prepareStatement(
                "select * from users where id = ?"
        );
        ps.setString(1, id);

				ResultSet rs = ps.executeQuery();
        rs.next();

        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        // 3. 리소스 반납하기
        ps.close();
        c.close();
	}
}
```

- 분리와 확장을 고려하여 관심사를 분리하자
1. 중복된 DB 연결 코드를 getConnection()이라는 이름의 독립적인 메소드로 만든다
2. 다른 종류의 DB를 사용하는 클라이언트를 위해 getConnection()을 추상 메서드로 만들고 상속을 통해 UserDao를 확장한다

![Untitled](https://github.com/yj-leez/BackEnd-WIL/assets/77960090/cc180071-3429-414a-ad3e-ed095280eb1c)

```java
public abstract class UserDao {
    public void add(User user) throws SQLException, ClassNotFoundException {

    }

    public User get(String id) throws SQLException, ClassNotFoundException {

    }

    public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}
public class NUserDao extends UserDao{
    @Override
    public Connection getConnection() throws ClassNotFoundException, SQLException {
				// NDrive 연결
    }
}
public class DUserDao extends UserDao {
    @Override
    public Connection getConnection() throws ClassNotFoundException, SQLException {
				// DDrive 연결
    }
}
```

- 관심사가 다르고 변화의 성격이 다른 두 코드를 클래스로 분리하자
- 두 클래스가 서로 긴밀하게 연결되어 있지 않도록 중간에 인터페이스로 추상화하자

![Untitled 1](https://github.com/yj-leez/BackEnd-WIL/assets/77960090/6bbfa6e8-5986-4a23-a0f3-86085b7cc350)

```java
public class UserDao {

	private SimpleConnectionMaker simpleConnectionMaker;

	public UserDao(){
		simpleConnectionMaker = new DConnectionMaker();
	}
}
```

- `connectionMaker = new DConnectionMaker();` : 여전히 UserDao가 ConnectionMaker의 구체 클래스에 의존하고 있는 문제점이 존재한다 → DIP, OCP 위반
- 관심사를 분리하여 오브젝트 간 관계를 맺는 책임을 클라이언트에게 떠넘기자

![Untitled 2](https://github.com/yj-leez/BackEnd-WIL/assets/77960090/54befeb5-7475-4291-92ce-f61e0737c140)

```java
public UserDao(SimpleConnectionMaker simpleConnectionMaker) {
        this.simpleConnectionMaker = simpleConnectionMaker;
}
```

## SOLID

### SRP: 단일 책임 원칙(single responsibility principle)

- 한 클래스는 하나의 책임만 가져야 한다
- 변경이 있을 때 파급 효과가 적도록 설계하자

### OCP: 개방-폐쇄 원칙 (Open/closed principle)

- 클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야한다
- 높은 응집도와 낮은 결합도
    - 응집도가 높다는 것은 변화가 일어날 때 해당 모듈에서만 변하는 부분이 크다는 것이다
    - 결합도가 낮다는 것은 관계를 유지하는 데 꼭 필요한 최소한의 방법만 간접적인 형태로 제공하고 나머지는 서로 독립적이고 알 필요도 없게 만들어주는 것이다
- eg) UserDao는 DB 연결 방법이라는 기능을 확장하는데는 열려 있지만 UserDao 자신의 핵심 기능을 구현한 코드는 그런 변화에 영향을 받지 않고 유지함으로 변경에는 닫혀 있다.

### LSP: 리스코프 치환 원칙 (Liskov substitution principle)

- 프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다

### ISP: 인터페이스 분리 원칙 (Interface segregation principle)

- 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다

### DIP: 의존관계 역전 원칙 (Dependency inversion principle)

- 구현 클래스에 의존하지 말고, 인터페이스에 의존하도록 설계하라

## 제어의 역전(IoC)

- 현재 UserDaoTest가 기존에 UserDao가 직접 담당하던 기능, 즉 어떤 ConnectionMaker 구현 클래스를 사용할지 결정하는 책임을 엉겁결에 맡게 되었다
- 객체의 생성 방법을 결정하고 그렇게 만들어진 오브젝트를 돌려주는 클래스를 만들어서 성격이 다른 책임이나 관심사는 분리하자

![Untitled 3](https://github.com/yj-leez/BackEnd-WIL/assets/77960090/77ea87d0-a389-4e0e-935d-ba0f52ea5926)

*참고: 김영한님의 스프링 핵심 원리- 기본편에서는 AppConfig로 정의하였다

```java
public class DaoFactory{

	public UserDao userDao(){
		return new UserDao(connectionMaker);
	}

	public ConnectionMaker connectionMaker(){
//		return new NConnectionMaker();
		return new DConnectionMaker();
}
```

- DaoFactory 객체는 ConnectionMaker 객체를 생성하고 그 참조값을 UserDao를 생성하면서 생성자로 전달한다
- `의존관계 주입` :
    
    애플리케이션 **실행 시점(런타임)**에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서
    클라이언트와 서버의 실제 의존관계가 연결 되는 것이다
    
    클라이언트인 UserDao 입장에서 보면 의존관계를 마치 외부에서 주입해주는 것 같기 때문이다
    
- DaoFactory처럼 객체를 생성하고 관리하면서 의존관계를 설정해주는 것을 IoC 컨테이너 혹은 DI 컨테이너라고 한다
- 라이브러리를 사용하는 애플리케이션 코드는 직접 애플리케이션 흐름을 제어한다. 단지 동작하는 중에 필요한 기능이 있을 때 능동적으로 라이브러리를 사용한다.
- 반면 프레임워크는 제어의 역전 개념이 적용된 대표적인 기술이다. 보통 프레임워크 위에 개발한 클래스를 등록해두고, 프레임워크가 개발자가 만든 애플리케이션 코드를 사용하면서 소프트웨어의 흐름을 제어하는 방식이다.

## 스프링의 IoC

- 스프링 빈: 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트
    
    스프링 컨테이너가 생성과 관계 설정, 사용을 제어해주는 제어의 역전이 적용된 오브젝트
    
- 애플리케이션 컨텍스트: 빈의 생성과 관계 설정 같은 제어를 담당하는 IoC 오브젝트인 빈 팩토리를 확장한 엔진
- DaoFactory, AppConfig는 위에서 IoC 엔진이었지만 여기서는 자바 코드로 만든 애플리케이션 컨텍스트의 설정정보로 활용되는 것이다. 즉 건물을 만들 때 설계도면과 같은 역할을 한다.
- `@Bean`: 오브젝트 생성을 담당하는 IoC용 메소드라는 표시
- `@Configuration`: 애플리케이션 컨텍스트 또는 빈 팩토리가 사용할 정보라는 표시
    
    `@Bean` 이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다.
    
- 애플리케이션 컨텍스트를 사용할 때 장점
    1. 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다
    2. 애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해준다
    3. 애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다. 타입만으로 빈을 검색하거나 특별한 애노테이션 설정이 되어 있는 빈을 찾을 수 있다.
