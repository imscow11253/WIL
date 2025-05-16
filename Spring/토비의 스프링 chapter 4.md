
279p ~ 315p 

## 4.1 사라진 SQLException

3장에서 만들었던 JdbcContext 클래스를 JdbcTemplate으로 바꾸는 과정에서 SQLException 을 throw 하는 로직이 없어졌다. 

```java
public void deleteAll() throws SQLException{
	this.jdbcContext.executeSql("delete from users");
}

public void deleteAll(){
	this.jdbcTemplate.update("delete from users");
}
```

### 4.1.1 초난감 예외처리

#### 예외 블랙홀 

```java
try{

}catch(SQLException e){
 // 아무것도 안함
}catch(....){
	System.out.println(e);
}catch(....){
	e.printStackTrace();
}
```

try-catch로 예외를 잡았으니 catch 문 안에서 아무것도 하지 않는 행위는 안된다.
그리고 단순히 콘솔에 출력하는 것도 안된다. 운영서버라고 생각하면 로그를 뜯어보기 쉽지 않기 때문이다. 

>[!tip] 예외처리 원칙은 한 가지다.
>모든 예외는 적절하게 복구되든지 아니면 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보해야 한다.

#### 무의미하고 무책임한 throws

예외를 적절하게 처리해주지 않고, 계속 호출 함수 스택에 따라 throw 하는 것은 좋지 않다. 
예외를 복구할 수 잇는 기회를 박탈당하기도 구체적으로 어디에서 예외가 발생했는지 알기도 어렵다. 


### 4.1.2 예외의 종류와 특징

자바에서 throw 키워드를 통해 발생시킬 수 있는 예외는 3가지 있다 

- ==Error==
  java.lang.Error 라이브러리에 정의된 클래스이다. 주로 jvm에서 발생시키는 것이라서 코드에서 잡으려고 하면 안된다. 
  OutOfMemory, ThreadDeath 등이 있다.
- ==Exception과 체크예외==
  java.lang.Exception 라이브러리에 정의된 클래스이다. Exception 클래스는 다시 체크 예외와 언체크 예외로 나뉜다. 언체크 예외는 RuntimeException을 상속한 것을 말한다.
  체크 예외는 try-catch 와 같이 코드 상에서 명시적으로 해결해주지 않으면 컴파일이 되지 않는다. 
- ==RuntimeException과 언체크/런타임 예외==
  java.lang.RuntimeException 라이브러리 클래스를 상속한 예외로, 명시적인 처리를 하지 않아도 된다. 
  NullPointerException, illegalArgumentException 등이 있다. 


### 4.1.3 예외처리 방법

#### 예외 복구
예외가 발생했을 때 문제를 해결해서 정상상태로 돌려놓는 것.

#### 예외처리 회피
예외처리를 자신이 담당하지 않고 자신을 호출한 함수로 던져버리는 것.

jdbcContext나 jdbcTemplate에서 사용되는 콜백 함수들은(ResultSet, PreparedStatement 등) 자신이 예외를 처리하지 않고 템플릿 코드로 던져버린다. 

#### 예외 전환
이것도 외부 메서드로 예외를 던진다는 점은 예외 회피와 같지만 적절한 예외로 전환해서 던진다는 점이 다르다.

목적

```java
public void add(User user) throws DuplicateUserIdException, SQLException{
	try{
		
	}catch(SQLException e){
		if(e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY){
			throw DuplicateUserIdException();
		}
		else 
			throw e;
	}
}
```

1. 외부에서도 어떤 상황인지 이해하도록 하기 위해서 예외 상황에 대한 의미를 부여하는 것이다. 
   예를 들어서 유저 로그인일 때, DB에서는 단순히 SQLException을 던지지만 service 클래스에게는 MultipleIdException 등 왜 sql 예외가 발생했는지 상황을 추가해주는 것이다.
   
   보통은 새로 만들 예외 안에 기존 예외를 포함시켜서 중첩예외를 만들어 주는 것이 외부에서도 세부적으로 파악할 수 있어서 좋다. 
```java
catch(SQLException e){
	throw DuplicateUserIdException(e);
}

catch(SQLException e){
	throw DuplicateUserIdException().initCause(e);
}
```

2. 예외를 처리하기 쉽게 포장하는 것이다. 1번 방법과는 다르게, 의미를 명확하게 하는 것이 아니다. 주로 예외처리를 강제하는 체크예외를 언체크 예외로 전환하는 경우가 해당된다.


### 4.1.4 예외처리 전략

>[!info] 체크 예외와 언체크 예외 차이
>- RuntimeException을 상속하는가 안하는가
>- 예외 처리를 강제하는가
>- 복구 가능한 예외인가
>- 코드 예외인가 시스템 예외인가

#### 런타임 예외의 보편화

자바의 환경이 서버로 이동하면서 체크 예외의 활용도와 가치가 떨어졌다. 체크 예외를 사용하면 throws 로 점철되고 try-catch로 범벅된 코드를 낳을 수 있다. 서버에서는 각 요청이 각자의 thread로 동작하니까 그냥 언체크 예외를 발생해서 해당 thread만 종료시키고 개발자에게 알리는 것이 낫다. 

#### add() 메소드의 예외처리

```java
public class DuplicateUserIdException extends RuntimeException{
	public DuplicateUserIdException(Throwable cause){ //중첩 예외
		super(cause);
	}
}

public void add() throws DuplcateUserIdException{
	try{
		
	}catch(SQLException e){
		if(e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
			throw new DuplicateUserIdExcpetion(e);
		else 
			throw new RuntimeException(e);
	}
}
```

#### 애플리케이션 예외

런타임 예외 중심의 전략은 ==낙관적인 예외처리 기법==이다. 
기존 전통적인 언체크 예외에서처럼 시스템 또는 외부의 예외상황이 아니라 어플리케이션 자체의 로직에 의해 의도적으로 발생시키고, 반드시 catch 해서 무엇인가 조치를 취하도록 요구하는 예외를 ==애플리케이션 예외==라고 한다. 이런 경우는 기본적으로 체크 예외로 만든다. 

### 4.1.5 SQLException은 어떻게 됐나?

맨 처음 이야기 했던 것처럼 jdbcTemplate을 사용하는 메서드에서는 SQLException을 명시적으로 던져주지 않았다.

SQLException은 복구가 불가능한 예외이다. 
이건 체크 예외라서 DAO에서 controller까지 해결도 못하는 예외를 계속 던져야 한다. 
그래서 언체크/런타임 예외로 전환해줘야 한다. 

그래서 JdbcTemplate에서는 자체적으로 SQLException을 런타임 예외인 DataAccessException으로 포장해서 던져준다. 


## 4.2 예외 전환

예외를 전환하는 목적은 앞서 설명한 것처럼 2가지 방법이 있다. 
1. 런타임 예외로 포장해서 굳이 필요하지 않은 catch/throw 줄여주기.
2. 좀 더 의미있고 추상화된 예외로 바꿔서 던져주기.

### 4.2.1 JDBC의 한계

JDBC는 어떤 DB가 오더라도 개발자에게 Connection, Statement, ResultSet과 같은 일관된 인터페이스 객체를 제공한다. 
하지만 이는 쉽지 않다. 

#### 걸림돌 1 : 비표준 SQL

JDBC에서 사용하는 SQL이 특정 DB에서는 다른 형태로 제공되거나 비표준 문법일 경우이다. 

#### 걸림돌 2 : 호환성 없는 SQLException의 DB 에러정보

DB를 사용하다가 발생하는 예외의 원인을 다양하다. 근데 JDBC는 데이터 처리 중에 발생하는 다양한 예외를 그냥 SQLException 하나에 모두 담아버린다. 
SQLException에서는 getErrorCode() 로 DB 에러코드를 가져올 수 있다. 근데 이 에러 코드는 DB 벤더 사마다 다르다. 
그래서 SQLException은 예외가 발생했을 때 DB 상태를 담은 SQL 상태 정보를 부가적으로 제공한다. getSQLState() 메서드를 사용하면 된다. Open Group의 XOPEN SQL 스펙에 정의된 SQL 상태 코드를 따르게 되어 있다. 근데 이것도 DB의 driver 마다 제대로 안 담아주는 경우가 허다해서 믿을만 한 것이 안된다.

### 4.2.2 DB 에러 코드 매핑을 통한 전환

DB 종류가 바뀌어도 DAO를 수정하지 않으려면 위 두가지 문제를 해결해야 한다. 

DB마다 SQL 문법이 다르다는 1번 걸림돌은 나중에 해결하고, 2번 걸림돌을 해결해보자.
해결 방법은 DB별 에러 코드를 참고해서 발생한 예외의 원인이 무엇인지 해석해주는 기능을 만드는 것이다. 

각 DB별 키 에러 중복 에러 코드가 감지되면 DuplicateKeyException을 반환하는 것이다. 
근데 문제는 모든 DB의 에러 코드를 다 알아야 하는 것일까? 그래서 스프링은 DB 별 에러 코드를 분류해서 예외 클래스와 매핑해놓은 에러 코드 매핑 정보 테이블을 만들어두고 이를 이용한다. 

즉, JDBCTemplate은 SQLException을 단순하게 RuntimeException으로 포장하는 것이 아니라, SQLException의 DB 에러 코드를 바탕으로 매핑된 에러 클래스를 반환하는 것이다. 

jdbcTemplate이 예외를 이렇게 포장해주기 때문에 add() 메서드에서는 따로 포장해줄 필요가 없다. 
즉, jdbcTemplate을 쓰면 DB 관련 예외는 거의 신경쓰지 않아도 된다. 

근데 직접 정의한 예외로 바꾸고 싶으면 catch로 jdbcTemplate이 정의한 예외를 잡아서 내가 정의한 예외로 전환해주면 된다. 


### 4.2.3 DAO 인터페이스와 DataAccessException 계층구조

DataAccessException 예외는 JDBC의 SQLException을 전환하는 용도로만 사용되는 것이 아니다. 
JPA 등과 같은 다양한 ORM 에서도 쓰이고, 다양하게 쓰인다. 

#### DAO 인터페이스와 구현의 분리

DAO를 왜 굳이 따로 분리하는 것일까? 
데이터 접근 로직을 분리하기 위해서이다. 이렇게 분리된 로직에 전략 패턴을 적용해서 구현 방법을 변경해서 쓰려고 하는 것이다. 

근데 문제는 데이터 접근 로직을 사용할 때 발생하는 예외에서 문제가 될 수 있다. 
DAO 인터페이스를 다음과 같이 만들었다고 해보자. 
```java 
public interface UserDao{
	public void add(User user);
} 
```

근데 JDBCTemplate이라면 해당 add 메서드에 throws SQLException을 붙여주어야 한다. 
JDBCTemplate 이 아닌 다른 DB 라이브러리라면 또 달라질 수 있다. 

런타임 Exception을 던지면 상관 없지만 체크 예외면 저렇게 표시를 해줘야 한다. 체크 예외더라도 add 메서드 내에서 런타임 예외로 전환해서 던지면 되긴 한다. 하지만 특정 예외에 따른 커스텀 예외를 던지고 싶을 수도 있다. 

즉, DAO를 사용하는 클라이언트 입장에서든 DAO의 사용기술에 따라서 예외처리 방법이 달라져야 하고, 이는 DAO의 기술에 의존적일 수 밖에 없다는 것이다. 

#### 데이터 엑세스 예외 추상화와 DataAccessException 계층구조

그래서 spring은 DAO에서 발생하는 모든 예외를 DataAccessException으로 추상화해서 서브 클래스들로 반환한다. 

모든 DAO에서 발생할 수 있는 예외를 다 포함하고 있기 때문에 JPA, Hibernate에서 발생하지만 JDBCTemplate에서 발생하지 않는 예외를 포함하고 있는 경우가 허다하다. 공통적인 것도 있고, 개별적인 것도 있다.


### 4.2.4 기술에 독립적인 UserDao 만들기

UserDao를 인터페이스로 분리하고 발생되는 예외에 대한 테스트 코드를 수정한다. 

#### DataAccessException 활용 시 주의사항

spring이 모든 DAO 예외를 DataAccessException의 하위 클래스로 전환해서 반환한다고 했다. 그래서 DataAccessException의 하위 클래스로 통일되어서 예외가 발생할 것이라 예상하지만 그렇지만은 않다. 

JDBC 같은 경우에는 DB의 에러코드를 바로 해석하지만 JPA나 Hibernate 같은 경우에는 자체 예외를 다시 해석해서 DataAccessException으로 전환하기 때문에 좀 다를 수 있다. 


