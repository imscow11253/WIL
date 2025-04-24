
53p ~ 144p

스프링은 자바의 객체지향 프로그래밍을 지향하는 프레임워크 (강제는 x)
오브젝트의 관리를 적극적으로 도와준다. 

>[!note] 스프링을 공부한다는 건 개발자 스스로 문제 제기와 의문에 대한 답을 찾아나가는 과저이다. 스프링은 기계적인 답변이나 성급한 결론을 주지 않는다. 최종 결론은 스프링을 이용해 개발자 스스로 만들어내는 것이다. 스프링은 단지 좋은 결론을 내릴 수 있도록 객체지향 기술과 자바 개발의 선구자들이 먼저 고민하고 제안한 방법에 대한 힌트를 제공할 뿐이다. 

1장에서는 ==객체지향 프로그래밍에 기반하여 오브젝트의 설계와 구현, 동작원리에 집중==한다. 
spring 안 배우고 왜 이것부터 배움? 
spring이 타겟팅 하는 걸 알아야 spring을 이해하기도 편하기 때문이다. 

## UserDao 리팩토링

User entity를 DB에 저장하는 ==UserDao 클래스를 naive 하게 만들어두고 발전 시켜나가는 방식==으로 진행한다.

다음은 navie하게 구현한 UserDao class로, JDBCTemplate을 사용하지 않고, jdbc 객체를 직접 가져오고, DB와의 Connection도 직접 가져와서 query를 보내는 형식이다. 진짜 naive하다.

```java
public class userDao{
	public void add(User user) throws ClassNotFoundException, SQLException{
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(
			"jdbc:mysql://localhost:3306/springbook", "spring", "book"
		);

		PreparedStatement ps = c.prepareStatement(
			"INSERT INTO users(id,name,password) VALUES (?,?,?)"
		);
		ps.setString(1, user,getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());
		
		ps.executeUpdate();
		
		ps.close();
		c.close();
	}
	
	public User get(String id) throws ClassNotFoundException, SQLException{
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(
			"jdbc:mysql://localhost:3306/springbook", "spring", "book"
		);

		PreparedStatement ps = c.prepareStatement(
			"SELECT * FROM users WHERE id = ?"
		);
		ps.setString(1, id);
		
		ResultSet rs = ps.executeQuery();
		re.next();
		User user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));
		
		rs.close();
		ps.close();
		c.close();
		
		return user;
	}
}
```


위의 class에서 어떤 관심 사항들이 있는지 분석해보자. 
1. DB와의 연결을 위한 connection 객체 생성
2. DB에 보낸 SQL 문장을 담을 Statement 객체 생성
3. 생성한 Statement와 Connection 객체를 close 시켜주는 작업

우선 Connection을 생성하는 코드를 분리해보자. 

```java
public void add(User user) throws ClassNotFoundException, SQLException {
	Connection c = getConnection();
	...
}

public User get(String id) throws ClassNotFoundException, SQLException {
	Connection. c = getConnection();
	...
}

private Connection getConnection() throws ClassNotFoundException, SQLException {
	Class.forName("com.mysql.jdbc.Driver");
	Connection c = DriverManager.getConnection(
		"jdbc:mysql://localhost:3306/springbook", "spring", "book"
	);
	return c;
}
```

이렇게 분리하면 DB연결하는 부분에 DB url이 변경되는 등의 코드 변화가 발생해도 한 곳만 수정하면 된다. 

더 개선해보자. 
만약 UserDao를 판매하게 되었다고 하자.
그럼 판매사의 DB 정보가 달라지게 된다. UserDao를 판매할 때마다 DB url 정보 바꾸고 컴파일한 바이너리 파일을 넘겨줘야 할까? 그러다가 판매사가 나중에 url을 바꾸게 된다면? 

상속을 사용하면 쉽게 해결된다. 
추상 클래스를 판매사에게 판매하고, UserDao를 구입한 판매사는 상속 클래스를 만들어서 getConnection 부분만 직접 구현하면 된다. 

```java
public abstract class UserDao{
	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
	}
	
	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection. c = getConnection();
		...
	}

	public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}
```

이렇게 슈퍼 클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브 클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 방법을 ==템플릿 메소드 패턴==이라고 한다. 
그리고 비슷해보이지만 ==팩토리 메소드 패턴==은 약간 다르다. 서브클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을 의미한다. 
 
>[!note] 템플릿 메소드 패턴은 그냥 구체적인 구현법을 서브클래스에게 맡기는 것을 통틀어서 의미하고, 팩토리 메서드 패턴은 객체 생성에 대한 것을 서브클래스에게 맡기는 것이다. 즉 템플릿 메소드 패턴은 행동을 추상화, 팩토리 메서드 패턴은 객체 생성을 추상화 한다고 보면 된다.

정리하면, 이전 코드에 템플릿 메소드 패턴을 적용함으로써 Connection 생성 책임을 서브 클래스에게 넘겨버린 것이다. 


근데, 상속의 단점이 있다. 
1. 여러 개 상속이 안된다.
2. 슈퍼-서브 클래스간 의존성이 강하다. (슈퍼 클래스가 변경되면 모든 서브 클래스를 수정해야 한다.)

관심사를 상속 관계인 서브 클래스가 아닌 아예 별도의 클래스로 분리하고, UserDao에서 해당 클래스를 멤버로 가지게 해서 사용하면 어떨까?

```java 
public class UserDao{
	private SimpleConnectionMaker simpleConnectionMaker;

	public UserDao(){
		simpleConnectionMaker = new SimpleConnectionMaker();
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = simpleConnectionMaker.makeNewConnection();
		...
	}
	
	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection. c = simpleConnectionMaker.makeNewConnection();
		...
	}
}

public class SimpleConnectionMaker{
	public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(
			"jdbc:mysql://localhost:3306/springbook", "spring", "book"
		);
		return c;
	}
}

```

근데 이러면 아까 해결했던 판매사에게 바이너리 코드로 UserDao를 판매할 때, DB url 변경 문제가 해결 되지 않는다. 
UserDao에서 SimpleConnectionMaker의 메소드명을 다 알고 있어야 한다는 단점이 존재한다.

판매사 A에서는 Connection 생성 메소드 명을 openConnection으로 했다면 UserDao는 오류를 일으킬 것이다. 

그래서 인터페이스를 도입한다. 
구현체는 인터페이스를 따라야만 하므로 UserDao는 인터페이스만 알면 된다.

==⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️  중요  ⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️==
>[!note] 인터페이스를 통한 연관관계를 맺는 것은 무슨 차이냐? 라고 의문을 가질 수 있다. 
>명확한 답변이 있다. 
>바로 클래스 간의 관계를 오브젝트 간 관계를 바꾸는 것이다. 
>코드에서는 두 클래스 간 관계가 없다가 런타임 시에만 관계가 생기는 것이다. --> 그러면 코드상의 변경은 코드 상에서의 관계가 없으므로 확산되지 않는다. 


```java
public interface ConnectionMaker{
	public Connection makeConnection() throws ClassNotFoundException, SQLException;
}

public class AConnectionMaker implements ConnectionMaker{
	public Connection makeConnection() throws ClassNotFoundException, SQLException {
		...
	}
}

public class UserDao{
	private ConnectionMaker connectionMaker;

	public UserDao(){
		connectionMaker = new AConnectionMaker(); //근데 여기서 구체적인 구현체 이름을 알아야 한다...
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.getConnection();
		...
	}
	
	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection. c = connectionMaker.getConnection();
		...
	}
}
```


해결되었으나, UserDao 의 생성자에서 구현체의 생성자를 알고 있어야 한다는 단점이 존재한다. 

이는 UserDao가 책임을 하나 더 가지고 있기 때문이다. 
==바로 UserDao에서 사용할 ConnectionMaker를 직접 결정하는 책임이다. ==

이 책임을 또 분리시켜주면 된다.
우리는 UserDao를 사용하는 외부 class에게 해당 책임을 줄 것이다. 
즉, UserDao와 ConnectionMaker의 런타임 의존관계를 만들어주는 책임을 UserDao를 사용하는 외부 class에게 넘기는 것이다. 

```java
public class UserDao{
	private ConnectionMaker connectionMaker;

	public UserDao(ConnectionMaker connectionMaker){
		connectionMaker = connectionMaker; // 파라미터로 넘겨진 값을 할당해주는 걸로 변경
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.getConnection();
		...
	}
	
	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection. c = connectionMaker.getConnection();
		...
	}
}
```

이렇게 되면 UserDao를 사용하는 외부 class에서 UserDao를 생성할 때 ConnectionMaker의 구현체를 넣어주어야만 한다. 

==⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️  중요  ⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️==
>[!note] 나는 예전에 interface로 의존도를 약화시킨다는게 이해가 잘 되지 않았다. interface를 사용하더라도 어차피 interface도 변경되면 똑같은 것 아닌가?
>interface를 사용하는 의의는 다음과 같다. 
>1. 구현체를 갈아끼우기 편하다
>2. 클래스 간 의존도를 오브젝트 간 의존도로 변경시킨다. (정적 의존도 -> 런타임 의존도)
>3. 인터페이스 자체는 쉽게 변하지 않게 '기능의 약속'만 정의하는 것이 best practice 이다.

이렇게 객체간 관계를 맺는 책임을 외부 class에서 맡게 하지 말고 별도의 class로 분리하자. 
이런 역할을 맡은 클래스를 보통 ==팩토리 클래스==라고 한다. 

팩토리 클래스는 서비스의 실질적인 로직을 담당하는 것과는 달리, ==서비스의 구조를 결정하는 책임==을 맡고 있다. 

```java
public class DaoFactory{
	public UserDao userDao(){
		return new UserDao(connectionMaker());
	}
	public ConnectionMaker connectionMaker(){
		return new AConnectionMaker();
	}
}
```


## 제어의 역전 (IoC) 이란?

기존 프로그램의 흐름은 '자신이 사용할 오브젝트를 직접 생성하고 메소드를 호출하는 방식' 이었다. 

예를 들어
main 에서 A를 생성, 메소드 호출 -> A에서 B를 생성, 메소드 호출 -> .... 
이런 식이다. 
즉, 모든 오브젝트가 능동적으로 자신이 사용할 클래스를 결정하고, 만드는 것을 관장한다. 

제어의 역전이란,
이런 개념을 뒤집는 것이다. 외부에서 내가 사용할 오브젝트를 결정하고, 생성해서 넣어주는 것이다. 


>[!note] 저번에 성준이형, 서준이형이랑 라이브러리, 프레임워크의 차이에 대해서 의논한 적이 있었다. 결론을 못냈는데 이 책에서 명확하게 설명해준다.
>라이브러리와 프레임 워크의 차이는 제어의 역전(IoC)이다.
>라이브러리는 우리가 짠 코드가 프로그램의 흐름을 직접 제어한다. 필요한 시점에 자기가 직접 라이브러리 코드를 가져다가 사용한다.
>반면에 프레임워크는 거꾸로, 우리가 작성한 코드가 프레임워크에 의해 사용된다. 보통 프레임워크 위에 개발한 클래스를 등록해두면 프레임워크가 흐름을 주도하는 중에 개발자가 만든 코드를 사용하는 방식이다. 


spring에서는 스프링 컨텍스트라는 클래스가 모든 IoC를 런타임 시에 결정해준다. 
스프링 컨텍스트에서 IoC 대상으로 관리하는 클래스는 빈(bean) 이라고 한다. 
우리는 스프링 컨텍스트에게 IoC 의존 관계만 알려주는 configuration 클래스만 생성해서 어노테이션을 붙여주기만하면 스프링 컨텍스트가 의존관계를 인지하고 런타임 시에 DI를 해준다.

```java
@Configuration
public class DaoFactory{
	@Bean
	public UserDao userDao(){
		return new UserDao(connectionMaker());
	}
	@Bean
	public ConnectionMaker connectionMaker(){
		return new AConnectionMaker();
	}
}
```


## 스프링 컨테이너

같은 말로 어플리케이션 컨텍스트라고도 한다. 

IoC를 담당할 뿐만 아니라 빈을 싱글톤으로 관리하는 싱글톤 레지스트리 역할도 한다. 
왜 싱글톤으로 관리할까?

스프링은 서버를 만드는데 주로 사용되고, 처음 구상도 서버를 위해 탄생했다. 
근데 요청 당 오브젝트를 생성하면 굉장히 많은 오브젝트가 생성될 것이다. 그래서 서버와 같은 엔터프라이즈 분야에서는 서비스 오브젝트라는 개념이 예전부터 사용되었다. 서비스 오브젝트가 싱글톤이 적용된 오브젝트이다. 


### 자바에서 싱글톤 패턴의 한계

기존 자바에서 그냥 naive하게 싱글톤 패턴을 구현하면 다음과 같은 한계가 존재한다.
- 싱글톤이 적용된 클래스는 private 생성자를 가진다. 그럼 상속이 안된다.
- 싱글톤은 테스트하기 힘들다.
- 서버 환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다. 
  클래스 로더의 구성, JVM 여러 개의 분산 시스템 등 같은 경우에는 오브젝트가 여러 개 생길 수 있다. 
- 전역 상태를 만들 수 있기 때문에 바람직하지 못하다.
  아무 객체가 같은 오브젝트를 공유하니까 전역 변수처럼 쓰일 수 있다. 

그래서 나온게 싱글톤 레지스트리이다. 

### 싱글톤 레지스트리

스프링의 싱글톤 레지스트리는 private 생성자, static 메서드 등을 안쓰고도 싱글톤을 구현해준다. 
오브젝트 생성 권한이 모두 스프링 컨테이너에 있기 때문이다. 

==테스트 환경과 같이 싱글톤으로 관리되어야 하지 않아도 되는 경우에는 직접 오브젝트를 생성해도 무관하다. ==


### 주의점

서버에서는 여러 요청을 동시에 받기 위해서 멀티 스레드 방식으로 요청을 처리한다.

근데 빈은 싱글톤으로 관리된다. 
즉, 여러 스레드가 하나의 오브젝트에 동시에 접근할 수 있음을 의미한다. 

이럴 경우에는 빈이 상태를 가지지 않는 ==무상태 방식==으로 만들어야 한다. 
필드 값에 저장하는 것이 아니라 파라미터, 로컬 변수, 리턴 값으로 값을 다루어야 한다. 
>[!note] 파라미터, 로컬변수, 리턴 값은 스레드 별로 독립적으로 관리된다.



## IoC와 DI의 차이

IoC는 매우 느슨하게 정의돼서 폭넓게 사용되는 용어이다. 
IoC를 만족하는 방법은 굉장히 다양한데, 그중에 하나가 DI, 의존관계 주입이다. 

DI 의 조건이 있는데 다음과 같다. 
- 클래스 UML 모델이나 code에는 런타임 시점의 의존관계가 드러나지 않는다. --> 인터페이스에만 의존해야 한다.
- 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제 3의 존재가 결정한다.
- 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공해줌으로써 만들어진다.



다음으로 쭉 DI의 장점을 설명하고, XML로 의존관계를 명시하는 방법이 나오는데 정리는 안하고 그냥 읽고 넘겼다.






