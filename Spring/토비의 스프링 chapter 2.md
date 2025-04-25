
145p ~ 207p


토비는 스프링이 개발자에게 제공하는 가장 중요한 가치는 ==객체지향==과 ==테스트== 라고 한다. 
1장에서 객체지향 좀 배웠으니까 2장에선 테스트에 대해서 살펴본다.

스프링으로 개발하면서 테스트를 만들지 않는 것은 스프링의 가치를 절반 포기하는 것이라고 한다.😱

스프링에서의 테스트는 작은 단위의 테스트가 가능하도록 해준다. --> 단위 테스트

초기에는 테스트 코드를 작성하는 것의 장점에 대해서 줄줄 설명한다. 

## UserDaoTest 코드 개선

여기서도 1장에서처럼 코드를 개선해나가며 테스트의 개념을 설명한다. 
1장에서 UserDao 클래스의 동작을 검증하기 위해 만들었던 UserDaoTest 라는 코드를 개선한다. 

UserDaoTest는 main 함수에서 호출되는 형태이고, 문제점이 2가지 존재한다.

1. 수동 확인 작업 
   개발자가 직접 DB 변경 여부를 확인해야 한다.
2. 실행 작업의 번거로움
   굉장히 많은 Dao가 만들어져서 DaoTest도 많이 만들어지면 main 메소드가 무거워진다.

1번 문제를 우선 해결해보면, DB에 저장 후 다시 불러와서 저장하려고 했던 값과 일치하는지 안하는지를 콘솔에 출력하는 방식으로 리팩토링할 수 있다. 

```java
if(!user.getName().equals(user2.getName())){
	System.out.println("테스트 실패 (name)");
}
else if(!user.getPassword().equals(user2.getPassword())){
	System.out.println("테스트 실패 (password)");
}
else{
	System.out.println("조회 테스트 성공");
}
```

2번 문제는 JUnit이라는 테스팅 프레임워크를 사용해서 해결할 수 있다.

```java
public class UserDaoTest{
	@Test
	public void addAndGet() throws SQLException{
		ApplicationContext context = new 
			ClassPathXmlApplicationContext("applicationContext.xml");
		
		UserDao dao = context.getBean("UserDao", UserDao.class);
		...
	}
}
```

이런 식으로 public 선언하고 @Test 어노테이션을 붙이고, 파라미터가 없으며, return 값이 void로 메소드를 선언하면 JUnit 프레임워크를 사용할 수 있다. 

그리고 위에서 if-else 문으로 값을 비교 검증했던 것을 assertThat이라는 static method를 사용해보자.

```java
assertThat(user2.getName(), is(user.getName()));
```

assertThat 메소드는 
첫 번째 파라미터의 값을 두 번째 파라미터인 Matcher와 비교해서 일치하면 넘어가고, 아니면 실패한다. 

is()는 매처의 일종으로 equals와 같다.

==JUnit 프레임워크를 실행==하기 위해서는 어디에는 main 메소드를 하나 추가하고, 그 안에 JUnitCore 클래스의 main 메소드를 호출하는 코드를 넣어주면 된다.
그리고 인자값으로 실행할 클래스 path를 지정해주면 된다.

```java
public static void main(String[] args){
	JUnitCore.main("springbook.user.dao.UserDaoTest");
}
```


다만 문제가 하나 더 있다. test를 한 번 실행하고 나면 DB에 user 가 생성되어서 다음번 test를 돌릴 때엔 실패할 것이다. 

그래서 UserDao에 deleteAll() 과 getCount() 메서드를 새로 만들어서 이를 활용하면 된다.

```java
@Test
public void addAndGet() throws SQLException{
	...
	dao.deleteAll();
	assertThat(dao.getCount(), is(0));

	User user = new User();
	user.setId("gyumee");
	user.setName("박성철");
	user.setPassword("springno1");

	dao.add(user);
	assertThat(dao.getCount(), is(1));

	User user2 = dao.get(user.getId());

	assertThat(user2.getName(), is(user.getName()));
	assertThat(user2.getPassword(), is(user.getPassword()));
}
```

그리고 책에서는 getCount() 메소드에 대해서 추가적인 테스트를 작성한다.

>[!tip] JUnit은 여러개의 test 메소드들이 존재할 때 실행 순서를 보장하지 않는다.
>만약 순서에 영향을 받도록 test code를 짰으면 잘못 짠거다.

그 외에도 get() 메소드, add() 메소드에 대해서 예외 상황 등의 추가적인 테스트 코드들을 작성한다. 
예외가 던져지는지 확인하는 것으로 확인하는데,
예외가 던져지는지 확인하는 것은 다음과 같은 구문을 사용하면 된다.

```java
@Test(expected = EmptyResultDataAccessException.class)
```

## TDD  테스트 주도 개발

테스트 코드는 잘 작성된 기능명세서로 쓰일 수 있다. 

장점
- 꼼꼼한 기능 명세
- 개발 코드에 대한 즉각적인 피드백

>[!warning] 오히려 개발 속도가 오래 걸리지 않을까?
>테스트는 작성하기 쉬우며, 각 테스트가 독립적이다. 테스트를 작성하는데 많은 시간이 들지 않으며, 한 번 작성해놓으면 계속 쓸 수 있으니 오히려 좋다.


## JUnit 의 부가적인 기능들

JUnit이 하나의 테스트 클래스를 실행하는 방식은 다음과 같다. 
1. 테스트 클래스에서 @Test가 붙은 public 이고, void 형이며, 파라미터가 없는 메소드를 찾는다.
2. 테스트 클래스의 오브젝트를 하나 만든다.
3. @Before가 붙은 메소드가 있으면 먼저 실행한다.
4. @Test가 붙은 메소드를 하나 호출하고 결과를 저장해준다. 
5. @After가 붙은 메소드가 있으면 실행한다.
6. 나머지에 대해서도 3~5 번을 반복한다. 
7. 모든 테스트 결과를 종합해서 돌려준다.

실제로는 더 복잡하다.
즉, 각 test를 실행하기 전과 후에 before, after 메소드를 실행한다는 것이다. 

>[!warning] Before, After 메소드를 test 메소드에서 직접 호출하는게 아니라서 서로 주고 받을 정보나 인스턴스가 있으면 test class의 필드 값을 이용해야 한다.

>[!info] 각 테스트 메소드를 실행할 때마다 테스트 클래스의 오브젝트를 새로 만든다. 이전 테스트를 위한 오브젝트는 버려진다.
>Why?
>각 테스트가 서로 영향을 주지 않고 독립적으로 실행됨을 보장하기 위해서 JUnit 개발자가 그렇게 짰다. 
>그래서 만약 공유해야 하는 자원이 있다면 별도의 클래스의 메소드로 분리하는 것이 좋다.

### 픽스처 Fixture

테스트를 수행하는데 필요한 정보나 오브젝트들을 의미한다.



## SpringTest

기존 Before 코드는 문제가 있다. 

```java
@Before 
public void setUp(){
	ApplicationContext context = new GenericXmlApplicationContext("application.xml");
	this.dao = context.getBean("userDao", UserDao.class);
}
```

⭐️⭐️⭐️⭐️⭐️⭐️⭐️
이러면 Test 메소드가 실행될 때마다 Before 메소드가 실행되므로 어플리케이션 컨텍스트가 test 메소드 개수만큼 생성된다. 

테스트 코드는 각 test가 독립적으로 동작하는 것이 원칙이지만
어플리케이션 컨텍스트처럼 무거운 것은 테스트 코드 전체가 공유하도록 만들기도 한다. 

공유하도록 만들고 싶은데 JUnit이 테스트 메소드를 실행할 때마다 테스트 클래스의 오브젝트를 새로 생성한다는 점이다. 
즉, 어플리케이션 컨텍스트를 오브젝트 레벨에 저장해두면 공유할 수 없다는 뜻이다. 

static에 저장해두면?
@BeforeClass 라는 어노테이션이 붙은 메소드는 테스트 클래스에 걸쳐서 한 번만 실행된다. 
이걸로 static 변수에 저장해둘 수 있겠지만

==스프링은 어플리케이션 컨텍스트를 테스트에서 쓸 수 있도록 지원==한다. 
정확히 말하면, 스프링은 JUnit을 이용하는 테스트 컨텍스트 프레임워크를 제공하는 것이다.
어노테이션만으로 어플리케이션 컨텍스트를 모든 테스트 코드가 공유하도록 만들 수 있다. 

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest{
	@Autowired
	private ApplicationContext context;
	//이렇게 말고 UserDao를 바로 Autowired로 DI 받아도 된다.
	...

	@Before 
	public void setUp(){
		this.dao = this.context.getBean("userDao", UserDao.class);
	}
}
```

이렇게 @RunWith 와 @ContextConfiguration 어노테이션을 붙이면 
@Autowired로 어플리케이션 컨텍스트를 주입 받을 수 있다. 

==@RunWith==
JUnit 프레임워크의 테스트 실행 방법을 확장할 때 사용한다.
여기서는 SpringJUnit4ClassRunner 라는 JUnit 용 테스트 컨텍스트 확장 클래스를 같이 사용했다. 

==@ContextConfiguration==
어플리케이션 컨텍스트의 xml 파일을 지정하는 것이다. 

>[!note]  라이브러리를 추가해줘야 한다.
>org.springframework.test-3.0.7.RELEASE.jar


## 테스트에서 DI

이런 의문을 가질 수 있다. DB Connection 정보는 변경하지 않을 건데 굳이 DI 해야해?
그냥 객체 생성해서 쓰면 되잖아?

1. 변경되지 않으리란 보장이 없다.
2. 다른 차원의 서비스를 추가하기가 좋다.
3. 작은 단위의 테스트가 쉽다.

만약 배포용 DB에다가 test를 돌리게 되면 배포 DB가 손상될 우려가 있다.
그래서 Before 메소드를 사용해서 다음과 같이 DI 시킬 수 있다.

```java
@DirtiesContext
public class UserDaoTest{
	@Autowired
	UserDao dao;

	@Before 
	public void setUp(){
		DataSource dataSource = new SingleConnectionDataSource(
			"jdbc:mysql://localhost/testdb", "spring", "book", true
		);
		dao.setDataSource(dataSource);
	}
}
```

여기서 UserDaoTest 클래스에 붙는 @DirtiesContext는 
테스트 메소드에서 어플리케이션 컨텍스트의 구성이나 상태를 변경한다는 것을 테스트 컨텍스트 프레임워크에게 알려주는 것이다. 그리고 이 어노테이션이 붙은 클래스에는 어플리케이션 컨텍스트 공유를 허용하지 않게 해서 다른 테스트 클래스에 영향이 가지 않도록 한다.

이러면 xml 을 수정하지 않아도 오브젝트 관계를 재구성할 수 있다. 

그러지 말고 test용 DataSource 빈을 하나 새로 만드는 것을 추천한다.
혹은 test용 xml 파일을 만들자.

## 그래서 어떤거?

DI를 이용한 test 방법에는 여러 가지가 존재한다. 
1. 스프링 컨테이너 이용 && DirtiesContext
2. 스프링 컨테이너 이용 && test 용 bean 
3. 스프링 컨테이너 안쓰고 직접 빈 생성

>[!summary] 그래서 뭐써야 하는데?
저자는 ==테스트에서는 스프링 컨테이너 안쓰는 것을 추천==한다. 
스프링 컨테이너를 띄우는데 시간이 오래 걸리기 떄문이다. 
>
근데 오브젝트가 복잡한 의존관계를 맺고 있다면 스프링 컨테이너 써서 xml 설정을 이용한 DI를 추천한다.
이 경우 test용 xml을 따로 만들라고 한다.
>
>테스트용 xml을 만들었다고 해도 예외적인 의존관계가 필요할 경우, 수동 DI하고 다른 테스트에 영향이 안가도록 @DirtiesContext를 붙이라고 한다.


## 학습 테스트 작성으로 스프링을 배워보자?

내가 작성한 코드가 아닌 남이 쓴 코드에 테스트 코드를 작성하는 경우가 있을까?

학습테스트가 그런 경우다. 
학습테스트란 자신이 사용할 API나 프레임워크의 기능을 테스트 코드를 작성하면서 사용 방법을 익히는 것이다. 
자신이 테스트를 만들려고 하는 기술이나 기능에 대해 얼마나 이해하고 있는지, 사용방법은 잘 아는지를 검증하는 것이 목적이다. 

직접 간단한 예제를 만들면서 배워도 되는데 왜 굳이 테스트를 쓰나?

1. 다양한 조건에 따른 기능을 확인 가능
2. 개발 중에 학습 테스트 코드를 참고 가능
3. 프레임워크, 제품을 업그레이드 할 때 호환성 검증
4. 테스트 작성에 좋은 훈련
5. 새로운 기술 공부가 즐거워짐(?)

스프링을 공부할 때도 스프링 프레임워크에 대한 테스트 코드를 직접 만들어보라고 한다.


### 학습 테스트 예제

JUnit이 테스트 메소드를 수행할 때마다 새로운 오브젝트를 만든다고 했다. 
이걸 테스트 코드로 검증해보자. 

새로운 테스트 클래스를 만들고, 3개의 테스트 메소드를 만든다.
테스트 클래스 자신의 타입으로 static 변수를 하나 선언해서, 매 테스트 메소드에서 현재 static 변수에 담긴 자신을 비교하는 방식으로 같은지 다른지 판단하는 테스트 코드를 만들어보자. 

```java
public class JUnitTest{
	static JUnitTest testObject;

	@Test
	public void test1(){
		assertThat(this, is(not(sameInstance(testObject))));
		testObject = this;
	}

	@Test
	public void test2(){
		assertThat(this, is(not(sameInstance(testObject))));
		testObject = this;
	}

	@Test
	public void test3(){
		assertThat(this, is(not(sameInstance(testObject))));
		testObject = this;
	}
}
```

좀 여러가지 matcher가 추가되었다. 
is(not()) 은 첫번쨰 파라미터와 달라야 성공하는 matcher 이다. 
sameInstance는 같은 오브젝트인지 비교한다. 

여기서는 직전 object랑만 비교하니까 다음과 같이 개선할 수 있다. 

```java
public class JUnitTest{
	static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();

	@Test
	public void test1(){
		assertThat(testObjects, not(hasItem(this)));
		testObjects.add(this);
	}
	@Test
	public void test2(){
		assertThat(testObjects, not(hasItem(this)));
		testObjects.add(this);
	}
	@Test
	public void test3(){
		assertThat(testObjects, not(hasItem(this)));
		testObjects.add(this);
	}
}
```



다른 학습 테스트도 작성해보자. 
테스트를 위한 스프링 컨테이너는 테스트 개수와 관계없이 하나만 생성된다고 했다. 

이를 검증해보자.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration
public class JUnitTest{
	@Autowired ApplictionContext context;

	static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();
	static ApplicationContext contextObject = null;

	@Test
	public void test1(){
		asssertThat(testObjects, not(hasItem(this)));
		testObjects.add(this);
		assertThat(contextObject == null || contextObject == this.context, is(true));
		contextObject = this.context;
	}

	@Test
	public void test2(){
		asssertThat(testObjects, not(hasItem(this)));
		testObjects.add(this);
		assertTrue(contextObject == null || contextObject == this.context);
		contextObject = this.context;
	}

	@Test
	public void test3(){
		asssertThat(testObjects, not(hasItem(this)));
		testObjects.add(this);
		assertThat(contextObject, either(is(nullValue())).or(is(this.context)));
		contextObject = this.context;
	}
}
```













