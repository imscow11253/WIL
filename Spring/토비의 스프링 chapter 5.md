
317p ~ 

자바 뿐만 아니라 어떤 언어든 라이브러리가 같은 역할을 하지만 사용법이나 문법이 다른 경우가 많다.
이를 추상화 시켜서 일관된 방법으로 사용하는 법을 배워보자. 


## 5.1 사용자 레벨 관리 기능 추가

지금껏 만든 UserDao에서 유저의 기능을 추가해보자. 

![[Pasted image 20250517015343.png]]

### 5.1.1 필드 추가

#### Level 이늄(Enum)

```java
public enum Level{
	BASIC(1), SILVER(2), GOLD(3);

	private final int value;
	
	Level(int value){
		this.value = value;
	}
	
	public int intValue(){
		return value;
	}
	
	public static Level valueOf(int value){
		switch(value){
			case 1: return BASIC;
			case 2: return SILVER;
			case 3: return GOLD;
			default: throw new AssertionError("Unknown value : " + value);
		}
	}
}

public class User{
	Level level;
	int login;
	int recommend;

	public Level getLevel(){
		return level;
	}

	public void setLevel(Level level){
		this.level = level;
	}
}


public class UserDaoJdbc implements UserDao{
	private RowMapper<User> userMapper = 
		new RowMapper<User>(){
			public User mapRow(ResultSet rs, int rowNum) throws SQLException{
				User user - new User();
				user.setId(rs.getString("id"));
				user.setName(rs.getString("Name"));
				user.setPassword(rs.getString("password"));
				user.setLevel(Level.valueOf(rs.getInt("level")));
				user.setLogin(rs.getInt("login"));
				user.setRecommend(rs.getInt("recommend"));
				return user;
			}
		};
	
	public void add(User user){
		this.jdbcTemplate.update(
			"insert into users(id, name, password, level, login, recommend) " + "values(?,?,?,?,?,?)", user.getId(), user.getName(), user.getPassword(), user.getLevel(),intValue(), user.getLogin(), user.getRecommend()
		);
	}
}
```



### 5.1.2 사용자 수정 기능 추가

#### UserDao와 UserDaoJdbc 수정

```java
public void update(User user){
	this.jdbcTemplate.update(
		"update users set name = ?, password = ?, level = ?, login = ?, recommend = ?, where id = ?", user.getName(), user.getPassword(), user.getLevel().intValue(), user.getLogin(), user.getRecommend(), user.getId()
	);
}
```


### 5.1.3 UserService.upgradeLevels()

```java
public class UserService{
	UserDao userDao;
	
	public void setUserDao(UserDao userDao){
		this.userDao = userDao;
	}
	
	public void upgradeLevels(){
		List<User> users = userDao.getAll();
		for(User user : users){
			Boolean changed = null;
			if(user.getLevel() == Level.BASIC && user.getLogin() >= 50){
				user.setLevel(Level.SILVER);
				changed = true;
			}
			if(user.getLevel() == Level.SILVER && user.getRecommend() >= 30{
				user.setLevel(Level.GOLD);
				changed = true;
			}
			else if (user.getLevel() == Level.GOLD){
				changed = false;
			}
			else changed = false;
			
			if(changed) userDao.update(user);
		}
	}
}

```


### 5.1.4 UserService.add()

add 함수에서 User를 생성할 때 default level 값이 BASIC이도록 수정.


### 5.1.5 코드 개선

![[Pasted image 20250517022219.png]]

#### upgradeLevels() 메소드 코드의 문제점

for 루프 안에 있는 if-else문이 좀 지저분하다. 
레벨이 추가 되면 새로운 if 문이 추가되기도 해야하고, level 조건이 복잡해지면 if 문의 조건문도 복잡해진다.


#### upgradeLevels() 리팩토링

```java
public void upgradeLevels(){
	List<User> users = userDao.getAll();
	for(User user : users){
		if(canUpgradeLevel(user)){
			upgradeLevel(user);
		}	
	}
}

private boolean canUpgradeLevel(User user){
	Level currentLevel = user.getLevel();
	switch(currentLevel){
		case BASIC: return (user.getLogin() >= 50);
		case SILBER: return (user.getRecommend() >= 30);
		case GOLD: return false;
		default: throw new IllegalArgumentException("Unknown Level : " + currentLevel);
	}
}

private void upgradeLevel(User user){
	if(user.getLevel() == Level.BASIC) user.setLevel(Level.SILVER);
	else if(user.getLevel() == Level.SILVER) user.setLevel(Level.GOLD);
	userDao.update(user);
}
```

근데 upgradeLevel 코드가 예쁘지 않다. level이 늘어남에 if 문도 늘어나고, 레벨 변경 로직이 노골적으로 노출되어있다. 그리고 예외 상황 처리가 없으며, GOLD인 경우엔 아무작업 없이 user를 update 한다. 

다음 레벨로 변경 되는 것은 Level 클래스 자체에게 맡기자. 이걸 호출하는 것도 User 자체에게 맡기자. 

```java 
public enum Level{
	GOLD(3,null), SILVER(2.GOLD), BASIC(1, SILVER);
	
	private final int value;
	private final Level next;
	
	Level(int value, Level next){
		this.value = value;
		this.next = next;
	}
	
	public Level nextLevel(){
		return this.next;
	}
}

public class User{
	public void upgradeLevel(){
		Level nextLevel = this.level.nextLevel();
		if(nextLevel == null){
			throw new IllegalStateException(this.level + "은 업그레이드 불가");
		}
		else{
			this.level - nextLevel;
		}
	}
}
```


함수 호출은 값을 가져와서 직접 판단, 변경하는 것이 아니라 하청을 맡기는 형테이다. 


## 5.2 트랜잭션 서비스 추상화


만약 서버의 갑작스러운 다운으로 인해 특정 사용자는 level이 업데이트 되었으나, 특정 사용자는 안되었다면? 
이런 상황에서 모두 업데이트가 안되도록 하고 싶다. 

### 5.2.1 모 아니면 도

중간에 예외를 발생해야 하는 등의 기존 클래스 로직이 아니라 특정 상황을 만들고 싶으면 기존 클래스를 상속하여 오버라이딩으로 로직을 새로 구현한 확장 클래스를 만드는 것이 좋을 것 같다. 
이를 위해 새로운 파일을 만들지 말고, test 클래스 내부에 static class로 만들면 편하다. 

![[Pasted image 20250517024521.png]]

>[!tip] 테스트를 위한 기존 클래스 접근 제한자 수정

```java
static class TestUserService extends UserService{
	private String id;
	
	private TestUserService(String id){
		this.id = id;
	}
	
	protected void upgradeLevel(User user){
		if(user.getId().equals(this, id)) throw new TestUserServiceException();
		super.upgradeLevel(user);
	}
}
```

특정 id에서 예외가 발생한 상황을 test 해보기 위해서 이런 확장 클래스를 생성했다. 특정 id 값에 대해 수정을 진행해야 하는 타이밍에 예외가 발생하도록 했다. 예외도 test를 위해서 만든 예외다. 

테스트 코드는 다음과 같다. 
```java
@Test
public void upgradeAllOrNothing(){
	UserService testUserService = new TestUserService(user.get(3).getId());
	testUserService.setUserDao(this.userDao);
	
	userDao.deleteAll();
	for(User user : users) userDao.add(user);
	
	try{
		testUserService.upgradeLeveles();
		fail("TestUserServiceException expected");
	}catch(TestUserServiceException e){
	}
	checkLevelUpgraded(users.get(1), false);
}
```

5명의 user 중에서 2번, 4번의 level을 update 해야되는 상황이다. 
근데 3번에서 예외를 터트렸고, 원하는 것은 4번이 update 되기 전에 예외가 발생했으므로, 2번도 update가 안되고 되돌려지는 것이다. 

근데 당연히 실패할 것이다. 

#### 테스트 실패의 원인

각 user의 update 쿼리가 하나의 DB transaction 내에서 실행되지 않았기 때문이다. 

### 5.2.2 트랜잭션 경계설정

#### JDBC 트랜잭션의 트랜잭션 경계설정

jdbc에서는 개발자가 트랜잭션의 시작, 커밋, 롤백을 마음대로 설정할 수 있도록 지원한다. 
다음은 jdbc의 트랜잭션을 적용한 예제다. 

```java
Connection c = dataSource.getConnection();

c.setAutoCommit(false);
try{
	PreparedStatement st1 = c.prepareStatement("update ....");
	st1.executeUpdate();
	
	PreparedStatement st2 = c.prepareStatement("update ....");
	st2.executeUpdate();
	
	c.commit();
}catch(Exception e){
	c.rollback();
}

c.close();
```

c.setAutoCommit(false); 
이 구문을 사용해서 DB가 자동으로 transaction의 범위를 생각해서 커밋하지 않도록 설정해야 한다. 
새로운 트랜잭션을 시작한다고 생각해도 좋다. 

#### UserService와 UserDao의 트랜잭션 문제

이전 테스트 코드가 실패했던 이유는 user를 update 하는 로직이 각각 DB connection을 따로 맺어서 각각 다른 transaction 에서 돌아갔기 때문이다. 

![[Pasted image 20250517033008.png]]


#### 비즈니스 로직 내의 트랜잭션 경계설정

그럼 upgradeLevels() 로직을 DAO 객체 안으로 옮겨야 하나?
근데 이건 비즈니스 로직과 데이터 로직을 분리시키지 않고 합쳐야 한다는 단점이 존재한다. 

그러면 트랜잭션의 경계를 upgradeLevels() 메서드 안으로 옮겨야 한다. 

![[Pasted image 20250517033453.png]]

근데 이렇게 되면 DAO 객체가 upgradeLevels() 메서드 내에서 선언된 connection 객체를 인자로 받아야 한다. 

#### UserService 트랜잭션 경계설정의 문제점

>[!Bug] JDBC와 JDBCTemplate
>JDBC는 DB에 접근할 수 있게 해주는 코드, 이걸 템플릿-콜백 패턴으로 추상화시킨게 JDBCTemplate


1. JdbcTemplate을 더이상 활용하지 못하고, JDBC API를 직접 사용하는 초기 방식으로 돌아가야 한다. 즉 다시 try-catch 범벅이 된다.
2. 비즈니스 로직이 있는 UserService 클래스에 Connection이 추가된다.
3. UserDao가 인자로 Connection을 받게 되면 DB에 종속적이게 된다.
4. Test 코드도 connection을 받아야 한다. 


### 5.2.3 트랜잭션 동기화 

스프링은 이런 딜레마를 해결해준다. 

#### Connection 파라미터 제거

upgradeLevels() 메소드가 트랜잭션 경계설정을 해야하는 건 피할 수 없다. 
따라서 Connection을 파라미터로 전달하는게 아니라 그 안에서 생성하고 시작, 종료를 해야 한다. 

이를 위해 스프링이 제안하는 것은 ==독립적인 트랜잭션 동기화 방식==이다. 
UserService에서 트랜잭션을 시작하기 위햇 만든 Connection 객체를 특별한 보관소에 두고, 이후 호출되는 DAO의 메소드에서는 저장된 Connection을 가져다 쓰게 만드는 것이다. 

![[Pasted image 20250517035811.png]]

(1) UserService는 Connection을 생성
(2) TransactionSynchronizations 저장소에 생성한 Connection 저장
(3) UserService는 첫 번째 update 메서드 호출
(4) TransactionSynchronizations에 Connection이 있는지 확인
(5) Connection으로부터 sql을 실행시킬 PreparedStatement 생성 후 실행
 ~ 반복
(12) UserService가 Connection의 commit을 호출
(13) TransactionSynchronizations 에서 Connection 제거


#### 트랜잭션 동기화 적용

스프링에서는 트랜잭션 동기화를 위한 TransactioSynchronizationManager 클래스를 제공한다.
![[Pasted image 20250517040342.png]]

여기서 Connection을 직접 가져오지 않고, DataSourceUtils 클래스의 static method로 가져오는 이유는 Connection 오브젝트 생성 뿐만 아니라 트랜잭션 동기화에 사용하도록 저장소에 바인딩 해주기 때문이다. 


#### JdbcTemplate과 트랜잭션 동기화

지금껏 JdbcTemplate의 update() , query() 같은거 호출하면 알아서 내부적으로 Connection, preparedStatment 등의 생성을 해준다고 했는데, 트랜잭션 동기화를 어떻게 하는걸까?

영리하게 동작한다. 
이미 connection이 생성되어 있으면 동기화해서 쓰고, 아니면 새로 생성한다.


### 5.2.4 트랜잭션 서비스 추상화

드디어 이 5장의 주제인 추상화가 나왔다. 

#### 기술과 환경에 종속되는 트랜잭션 경계설정 코드

만약 DB가 여러개고 DAO도 각각 존재한다.
그리고 하나의 트랜잭션에서 이렇게 벤더사가 다른 DB에 작업하는 로직을 다루고 싶다. 
그러면 어떻게 해야 하나? 

자바는 JDBC 외에 이런 글로벌 트랜잭션을 지원하는 JTA(Java Transaction API )를 제공한다. 
![[Pasted image 20250517041127.png]]

#### 트랜잭션 API의 의존관계 문제와 해결책

UserService가 DAO 패턴을 사용해 구현 데이터 액세스 기술을 유연하게 바꿔서 사용할 수 있게 했지만 UserService에서 트랜잭션의 경계 설정을 해줘야 하므로 다시 특정 데이터 액세스 기술에 종속되는 구조가 되었다. 

트랜잭션의 경계설정을 담당하는 코드는 일정한 패턴을 갖는 유사한 구조다. 
이렇게 공통점이 있으면 추상화를 할 수 있다. 

JDBC, JPA, JDO, JMS, JTA 모두 트랜잭션의 개념을 가지고 있으니, 트랜잭션 경계 설정을 추상화 할 수 있을 것이다. 

![[Pasted image 20250517043402.png]]

스프링이 제공하는 트랜잭션 경계설정을 위한 추상 인터페이스는 PlatformTransactionManager이다. 
JDBC의 트랜잭션 경계 설정을 하고 싶으면 DataSourceTransactionManager 구현체를 사용하면 된다. 
PlatformTransactionManager에서는 Connection을 생성할 필요 없이 getTransaction( ) 메서드를 호출하기만 하면 된다. 

getTransaction() 메서드의 인자로 넘겨주는 DefaultTransactionDefinition 오브젝트는 transaction의 속성을 담고 있다. 

똑같이 PlatformTransactionManager에서 생성된 트랜잭션은 트랜잭션 동기화 저장소에 저장된다. DataSourceTransactionManager는 JDBCTemplate에서 사용가능한 방식으로 트랜잭션을 관리해준다. 


#### 트랜잭션 기술 설정의 분리 

이를 JTA를 이용하는 글로벌 트랜잭션으로 변경하고 싶으면? 
PlatformTransactionManager을 JTATransactionManager 클래스로 변경하면 된다.

Hibernate면 HibernateTransactionManager,
JPA면 JPATransactionManager를 사용하면 된다.

이것도 모두 어떤 구현체를 사용할지 UserService 클래스가 알고 있는건 이상하다. 
구현체를 DI 해주는 것이 맞다. 



## 5.3 서비스 추상화와 단일 책임 원칙

#### 수직, 수평 계층 구조와 의존관계

이렇게 기술과 서비스에 대한 추상화 기법을 이용하면 특정 기술환경에 종속되지 않는 포터블한 코드를 만들 수 있다. 

UserDao와 UserService는 각각 담당하는 코드의 기능에 따라 분리했다. ==같은 계층에서 수평적인 분리==라고 볼 수 있다. 
트랜잭션 추상화는 다르다. 어플리케이션의 비즈니스 로직과 그 하위의 로우레벨의 트랜잭션 기술을 분리한 ==수직적인 분리==이다. 

![[Pasted image 20250517044702.png]]

수직, 수평적인 구조 모두 DI를 사용했기에 의존성은 낮다.


#### 단일 책임 원칙

단일 책임을 주려고 하니까 계속 클래스 분리하고 메서드 나누고 한 것이다. 
위에서 UserService에서 Connection 생성을 분리한 것처럼.


## 5.4 메일 서비스 추상화 

메일 서비스를 추상화 해보는 실습이다.
