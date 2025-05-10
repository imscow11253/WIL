
209p ~ 278p

템플릿이란
변경이 거의 일어나지 않으며 일정한 패턴으로 유지되는 특성을 가진 부분을 자유롭게 변경되는 성질을 가진 부분으로부터 독립시켜주는 기능을 말한다. 

계속 개선 해오던 UserDao 코드를 통해 템플릿을 소개한다. 

## 3.1 다시 보는 초난감 DAO

### 3.1.1 예외처리 기능을 갖춘 DAO

기존 UserDao도 많은 부분이 개선되었지만 예외 상황에 대한 처리가 아쉬웠다. 
특히나 deleteAll() 이라는 메서드를 보면 예외처리가 전혀 안되어 있음을 확인 할 수 있다. 

```java
public void deleteAll() throws SQLExcpetion{
	Connection c = dataSource.getConnection();

	PreparedStatement ps = c.prepareStatement("delete from users");
	ps.executeUpdate();

	ps.close();
	c.close();
}
```

Connection 객체와 PreparedStatement 라는 객체를 풀에서 가져오는데, 여기서 만약 오류가 터져서 함수가 제대로 종료 되지 않으면 close 함수가 호출되지 않아서 풀에 자원이 부족해질 수 있다. 

```java
public void deleteAll() throws SQLExcpetion{
	Connection c = null;
	PreparedStatement ps = null;

	try{
		c = dataSource.getConnection();
		ps = c.prepareStatement("delete from users");
		ps.executeUpdate();	
	} catch(SQLException e){
		throw e;
	} finally {
		if(ps != null){
			try{
				ps.close();
			}catch(SQLException e){
			}
		}
		if(c != null){
			try{
				c.close();
			} catch(SQLException e){
			}
		}
	}
}
```

기존에 만들어 두었던 getCount() 메서드도 똑같이 예외처리를 해준다.


## 3.2 변하는 것과 변하지 않는 것

### 3.2.1 JDBC try/catch/finally 코드의 문제점

이렇게 try-catch 문으로 예외처리를 한다면 코드의 Index가 깊어지고, 길어질 것이다.
예외처리 문은 구조 상 1장에서처럼 메소드나 클래스를 분리하는 것만으로 해결이 어렵다. 

### 3.2.2 분리와 재사용을 위한 디자인 패턴 적용

우선 변하는 것과 변하지 않는 코드를 분리할 필요가 있다. 

![[Pasted image 20250509180039.png]]

DB에 보낼 쿼리를 쓰는 부분은 메소드마다 달라질 것이고, Connection, PreparedStatement 객체를 가져오는 것은 쿼리를 보낼 때마다 공통으로 해야하는 작업이니까 변하지 않는다. 

이걸 분리 하는 방법에는 어떤 것들이 있을까?

#### 해결책 1 - 메소드 추출

```java
public void deleteAll() throws SQLException{
	...
	try{
		c = dataSource.getConnection();

		ps = makeStatement(c);  --> 이 부분이 변하는 부분이라서 메서드로 추출
		
		ps.execiteUpdate();
	}catch(SQLException e){
	}
	...
}

private PreparedStatement makeStatement(Connection c) throws SQLException{
	PreparedStatement ps;
	ps = c.prepareStatement("delete from users");
	return ps;
}
```

이건 의미가 없다 .
분리한 makeStatement 라는 메서드를 재사용할 수 있는게 아니기 떄문이다. 
뭔가 반대로 되었다.

#### 해결책 2 - 템플릿 메소드 패턴의 적용
```java
abstract protected PreparedStatement makeStatement(Connection c) throws SQLException;

public class UserDaoDeleteAll extends UserDao{
	protected PreparedStatement makeStatement(Connection c) throws SQLException{
		PreparedStatement ps = c.prepareStatement("delete from users");
		return ps;
	}
}
```

잘 만든 것 같다. 하지만 제한이 있다. 
DAO 로직마다 상속으로 새로운 클래스를 만들어야 한다는 점이 불편하다. 
DB에 쿼리 보내는 메서드 만들려면 상속을 통해 새로운 클래스를 각각 추가로 만들어야 한다는 단점이 있다. 

#### 해결책 3 - 전략 패턴의 적용

템플릿 메서드 패턴과 클래스 다이어그램 상으로 비슷한 느낌이다. 

근데 다만 좀 다른 것은
템플릿 메서드 패턴은 추상 클래스로 변하지 않는 것을 적어두고, 구현체에서 변하는 부분을 직접 작성했다면,
전략 패턴은 좀 반대로, 변하지 않는 것은 그대로 두고, 변하는 부분만 인터페이스로 만들어서 구현체를 가져다 끼워 사용하는 방식이다. 

```java
public interface StatementStrategy{
	PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}

public class DeleteAllStatement implements StatementStrategy{
	public PreparedStatement makePreparedStatement(Connection c) throws SQLException{
		PreparedStatement ps = c.prepareStatement("delete from users");
		return ps;
	}
}

public void deleteAll() throws SQLException{
	...
	try{
		c = dataSource.getConnection();
		
		StatementStrategy strategt = new DeleteAllStatement();
		ps = strategy.makePreparedStatement(c);
		
		ps.executeUpdate();
	}catch (SQLException e){
	}
	...
}
```

하지만 좀 아쉬운 점이 있다면 deleteAll() 이라는 매서드가 어떤 구현체를 가져야 하는지 알아야 한다는 점이다. 

#### 해결책 4 - DI 적용을 위한 클라이언트/컨텍스트 분리

전략 패턴은 실제로 외부에서 구현체를 DI 해주는 것이 정배다. 
chapter 1에서 했던 과정이랑 비슷하다. 

```java
public interface StatementStrategy{
	PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}

public class DeleteAllStatement implements StatementStrategy{
	public PreparedStatement makePreparedStatement(Connection c) throws SQLException{
		PreparedStatement ps = c.prepareStatement("delete from users");
		return ps;
	}
}

public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
	Connection c = null;
	PreparedStatement ps = null;
	
	try{
		c = dataSource.getConnection();
		
		ps = stmt.makePreparedStatement(c);
		
		ps.executeUpdate();
	}catch (SQLException e){
		throw e;
	}finally{
		if(ps != null) {try{ps.close();} catch (SQLException e){}}
		if(c != null) {try{c.close();} catch (SQLException e){}}
	}
}

public void deleteAll() throws SQLException{
	StatementStrategy st = new DeleteAllStatement();
	jdbcContextWithStatementStrategy(st);
}
```

이런 식으로 DI를 통한 전략 패턴 형식을 사용하면 try-catch 문에 의한 예외 처리도 책임에 따라 분리할 수 있다. 

## 3.3 JDBC 전략 패턴의 최적화

### 3.3.1 전략 클래스의 추가 정보

위와 같이 전략 패턴을 적용하면 OCP가 잘 지켜진다. 
예시로 user 를 추가하는 함수를 구현해보자. 

```java
public class AddStatement implements StatementStrategy{
	User user;
	
	public AddStatement(User user){
		this.user = user;
	}
	
	public PreparedStatement makePreparedStatement(Connection c){
		...
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());
		...
	}
}

public void add(User user) throws SQLException{
	StatementStrategy st = new AddStatement(user);
	jdbcContextWithStatementStrategy(st);
}
```

### 3.3.2 전략과 클라이언트 동거

작가는 여기서 만족하지 않는다. ~~눈이 높다.~~
2가지 아쉬운 점을 더 꼽았다. 

1. StatementStrategy의 구현 클래스를 직접 만들어야 한다. --> 클래스 파일이 많아진다.
2. DAO 메소드 (DI를 하는 클라이언트)가 StatementStrategy의 구현체를 생성할 때 생성자의 인자를 줄 때 번거롭게 만들어야 한다는 점이다. 

#### 해결책 1 - 로컬 클래스 

클래스 파일이 많아지는 건 메서드 안에 클래스를 정의해서 일회성으로 써버리는 것이다. 
이처럼 특정 메서드에서만 사용되는 클래스라면 로컬 클래스로 정의하면 된다.

```java
public void add(User user) throws SQLException{
	class AddStatement implements StatementStrategy{
		User user;
		
		public AddStatement(User user){
			this.user = user;
		}
		
		public PreparedStatement makePreparedStatement(Connection c){
			...
			ps.setString(1, user.getId());
			ps.setString(2, user.getName());
			ps.setString(3, user.getPassword());
			...
		}
	}
	
	StatementStrategy st = new AddStatement(user);
	jdbcContextWithStatement(st);
}
```

로컬 클래스의 장점은 선언된 메소드의 지역 변수에 접근 가능하다는 것이다. 
위에서처럼 굳이 생성자의 인자로 user 를 전달해줄 필요가 없다. 

#### 해결책2 - 익명 내부 클래스

이번엔 선언할 필요도 없이 사용하면서 바로 써버리는 익명 클래스로 만들어보자. 

```java
public void add(final User user) throws SQLException{
	jdbcContextWithStatementStrategy(
		new StatementStrategy(){
			public PreparedStatement makePreparedStatement(Connection c) throws SQLExcpetion{
				PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?);");
				ps.setString(1, user.getId());
				ps.setString(2, user.getName());
				ps.setString(3, user.getPassword());
				return ps;
			}
		}
	)
}
```


## 3.4 컨텍스트와 DI

### 3.4.1 JdbcContext의 분리

현재까지의 흐름으로 보자면 각 클래스, 메서드의 의미는 다음과 같다. 

UserDao 메서드 (deleteAll, add) : 클라이언트 (DI의 주체)
익명 내부 클래스 : 전략 패턴의 전략 (구현체)
jdbcContextWithStatementStrategy : 컨텍스트

여기에서 jdbcContextWithStatementStrategy 메서드는 다른 DAO에서도 사용할 수 있다. 그래서 클래스 외부로 분리하려고 한다.

#### 방법 1 - 클래스 외부로 분리 

JdbcContext라는 이름의 클래스로 분리시켜보자. 
```java
public class JdbcContext {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try {
            c = this.dataSource.getConnection();
            ps = stmt.makePreparedStatement(c);
            ps.executeUpdate();
        } catch (SQLException e) {
            throw e;
        } finally {
            if (ps != null) { try { ps.close(); } catch (SQLException e) {} }
            if (c != null) { try { c.close(); } catch (SQLException e) {} }
        }
    }
}

public class UserDao {
    private JdbcContext jdbcContext;

    public void setJdbcContext(JdbcContext jdbcContext) {
        this.jdbcContext = jdbcContext;
    }

    public void add(final User user) throws SQLException {
        this.jdbcContext.workWithStatementStrategy(
            new StatementStrategy() {
                // ...
            }
        );
    }

    public void deleteAll() throws SQLException {
        this.jdbcContext.workWithStatementStrategy(
            new StatementStrategy() {
                // ...
            }
        );
    }
}
```


여기서 JdbcContext를 DI 해주는데 보통 DI는 인터페이스로 받는 것이 보통이다.
하지만 이 경우 JdbcContext는 그 자체로 독립적인 JDBC 컨텍스트를 제공해주는 서비스 오브젝트로서 의미가 있을 뿐이고 구현 방법이 바뀔 가능성은 없어서 인터페이스로 구현하지 않았다. 

### 3.4.2 JdbcContext의 특별한 DI

지금 코드는 UserDao가 인터페이스 없이 직접적으로 JdbcContext라는 구현 클래스를 주입 받고 있다. 
문제 없나?

#### 스프링 빈으로 DI 

엄격하게 따지면 인터페이스를 두는 것이 맞다.
하지만 스프링의 DI를 넓게 보자면, 객체의 생성과 관계 설정에 대한 제어 권한을 오브젝트에서 제거하고 위임 했다는 IoC의 개념을 포괄한다. 

>[!info] 내가 DI를 인터페이스로 받아야 한다는 것을 알았을 때 잼스 톡방에 '우리 지금껏 DI를 안하고 있었어~!' 했었는데 주언이형이 마냥 DI를 안하고 있었다는 것은 아니라고 말했었다. 

여기서 인터페이스 없이 JdbcContext를 DI하는 것의 의의가 뭘까?

1. 싱글톤 레지스트리에서 관리되는 싱글톤이다. 
2. JdbcContext가 DI를 통해 다른 빈에 의존하고 있기 때문이다. 


## 3.5 템플릿과 콜백 

지금까지의 코드를 보면 전략 패턴의 기본 구조에 익명 내부 클래스를 활용한 방식이라고 볼 수 있다. 
이런 방식을 스프링에서는 ==템플릿/콜백 패턴==이라고 부른다. 

전략 패턴의 컨텍스트를 템플릿,
익명 내부 클래스인 오브젝트를 콜백

템플릿은 고정된 작업 흐름을 가진 코드를 재사용한다는 의미, 콜백은 템플릿 안에서 호출되는 것을 목적으로 만들어진 오브젝트이다. 

### 3.5.1 템플릿/콜백의 동작원리

#### 템플릿/콜백의 특징

전략 패턴에서 인터페이스는 여러 메소드를 가질 수 있다. 
근데 콜백은 하나의 메소드만 가지는 것이 정배다. 

즉, 콜백은 일반적으로 하나의 메소드를 가진 인터페이스를 구현한 익명 내부 클래스로 만들어진다고 보면 된다. 

![[Pasted image 20250510104031.png]]

>[!summary] 전체적인 구조가 DI 방식의 전략 패턴 구조라고 보면 된다.
>클라이언트가 템플릿 메소드를 호출하면서 콜백 오브젝트를 전달하는 것은 메소드 레벨에서 일어나는 DI다. 템플릿이 사용할 콜백 인터페이스를 구현한 오브젝트를 메소드를 통해 주입해주는 이만 작업의 클라이언트가 템플릿의 기능을 호출하는 것과 동시에 일어난다. 일반적인 DI라면 템플릿에 인스턴스 변수를 만들어두고 사용한 외조 오브젝트를 수정자 메소드로 받아서 사용할 것이다.
>
 반면에 템플릿/콜백 방식에서는 매번 메소드 단위를 사용할 오브젝트를 새롭게 전달받는다는 것이 특징이다. 콜백 오브젝트가 내부 클래스로서 자신을 생성한 클라이언트의 메소드 내의 정보를 직접 참조한다는 것도 템플릿/콜백의 고유한 특징이다. 
 클라이언트와 콜백이 강하게 결합된다는 면에서도 일반적인 DI와 조금 다르다. 템플릿/콜백 방식은 전략 패턴과 DI의 장점을 익명 내부 클래스 사용 전략과 결합한 독특한 활용법이라고 이해할 수 있다. 단순히 전략 패턴으로만 보기에 독특한 특징이 많으므로 템플릿/콜백을 하나의 고유한 디자인 패턴으로 기억해두면 편리하다. 다만 이 패턴에 녹아 있는 전략 패턴과 수동 DI를 이해할 수 있어야 한다.


### 3.5.2 편리한 콜백의 재활용

현재 매번 콜백 함수를 익명 클래스로 만들어서 활용하고 있는데 불편할 수 있다.
그래서 콜백을 분리하고 재활용 해보자. 

### 콜백의 분리와 재활용

기존 콜백 함수를 생성하는 부분은 다음과 같다 .
```java
public void deleteAll() throws SQLException {
    this.jdbcContext.workWithStatementStrategy(
        new StatementStrategy() {
            public PreparedStatement makePreparedStatement(Connection c)
                throws SQLException {
                return c.prepareStatement("delete from users");
            }
        }
    );
}
```

변하는 부분은 쿼리 부분 밖에 없는데 익명함수에서는 다소 복잡하며 변하지 않는 부분도 계속 선언한다. 

```java
public void deleteAll() throws SQLException{
	executeSql("delete from users");
}

private void executeSql(final String query) throws SQLException{
	this.jdbcContext.workWithStatementStrategy(
		new StatementStrategy(){
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException{
				return c.prepareStatement(query);
			}
		}
	)
}
```

이렇게 하면 executeSql 이라는 익명합수를 만들어서 전달하는 코드는 재사용이 가능하다. 

#### 콜백과 템플릿의 결합

executeSql 이라는 메서드를 굳이 DAO에서 가질 필요 없이 JdbcContext 안에 선언해서 public으로 열어두어도 된다. 

```java
public class JdbcContext{
	...
	public void executeSql(final String query) throws SQLException{
		this.jdbcContext.workWithStatementStrategy(
			new StatementStrategy(){
				public PreparedStatement makePreparedStatement(Connection c) throws SQLException{
					return c.prepareStatement(query);
				}
			}
		)
	}
}

public class UserDao{
	public void deleteAll() throws SQLException{
		this.jdbcContext.executeSql("delete from users");
	}
}
```


![[Pasted image 20250510110207.png]]

이런 구조가 되었다.

### 3.5.3 템플릿/콜백의 응용

템플릿/콜백 패턴을 사용할 수 있는 가장 전형적인 후보는 try-catch-finally 블록의 코드다. 

#### 테스트와 try-catch-finally

다음 코드는 파일에서 한 줄 씩 읽어와서 숫자의 합을 구하는 코드와 테스트 코드의 예제다. 

```java
public Integer calcSum(String filepath) throws IOException {
    BufferedReader br = null;

    try {
        br = new BufferedReader(new FileReader(filepath));
        Integer sum = 0;
        String line = null;
        while ((line = br.readLine()) != null) {
            sum += Integer.valueOf(line);
        }
        return sum;
    }
    catch(IOException e) {
        System.out.println(e.getMessage());
        throw e;
    }
    finally {
        if (br != null) {
            try { br.close(); }
            catch(IOException e) { System.out.println(e.getMessage()); }
        }
    }
}

public class CalcSumTest{
	@Test
	public void sumOfNumbers() throws IOException{
		Calculator calculator = new Calculator();
		int sum = calculator.calcSum(getClass().getResource(
		"numbers.txt").getPath());
		assertThat(sum, is(10));
	}
}
```

#### 중복의 제거와 템플릿/콜백 설계

근데 만약 모든 숫자의 합이 아니라 곱도 계산하는 기능을 추가해야 한다면?
이런 변경사항을 유연하게 대처하기 위해서 템플릿/콜백 패턴을 적용해보자.

템플릿과 콜백에 각각 어떤 코드를 넣으면 좋을지를 분리해야 한다. 

가장 쉬운 구조는 템플릿이 파일을 열고, 각 라인을 읽어오는 BufferedReader를 만들어서 콜백에게 주고, 콜백이 각 라인을 읽어서 알아서 처리 후 최종 결과만 템플릿에게 주는 구조이다. 

```java 
// 인터페이스
public interface BufferedReaderCallback{
	Integer doSomethingWithReader(BufferedReader br) throws IOException;
}

//템플릿
public Integer fileReadTemplate(String filepath, BufferedReaderCallback callback)
        throws IOException {
    BufferedReader br = null;
    try {
        br = new BufferedReader(new FileReader(filepath));
        int ret = callback.doSomethingWithReader(br);
        return ret;
    }
    catch(IOException e) {
        System.out.println(e.getMessage());
        throw e;
    }
    finally {
        if (br != null) {
            try { br.close(); }
            catch(IOException e) { System.out.println(e.getMessage()); }
        }
    }
}

//client
public class Calculator{
	public Integer calcSum(String filepath) throws IOException {
	    BufferedReaderCallback sumCallback = 
	        new BufferedReaderCallback() {
	            public Integer doSomethingWithReader(BufferedReader br) throws IOException {
	                Integer sum = 0;
	                String line = null;
	                while ((line = br.readLine()) != null) {
	                    sum += Integer.valueOf(line);
	                }
	                return sum;
	            }
	        };
	    return fileReadTemplate(filepath, sumCallback);
	}
	public Integer calcMultiply(String filepath) throws IOException {
	    BufferedReaderCallback multiplyCallback =
	        new BufferedReaderCallback() {
	            public Integer doSomethingWithReader(BufferedReader br) throws IOException {
	                Integer multiply = 1;
	                String line = null;
	                while ((line = br.readLine()) != null) {
	                    multiply *= Integer.valueOf(line);
	                }
	                return multiply;
	            }
	        };
	    return fileReadTemplate(filepath, multiplyCallback);
	}
}
```

#### 템플릿/콜백의 재설계

템플릿과 콜백을 찾아낼 때는, 변하는 코드의 경계를 찾고 그 경계를 사이에 두고 주고받는 일정한 정보가 있는지 확인하면 된다. 

잘 보면 콜백에서도 합을 계산하는 것과 곱을 계산하는 것에 변하는 부분이 크지 않다. 

다시 콜백을 정의하면 다음과 같이 될 수 있다. 

```java
//interface
public interface LineCallback{
	Integer doSomethingWithLine(String line, Integer value);
}

//template
public Integer lineReadTemplate(String filepath, LineCallback callback, int initVal) throws IOException {
    BufferedReader br = null;
    try {
        br = new BufferedReader(new FileReader(filepath));

        Integer res = initVal;
        String line = null;
        while ((line = br.readLine()) != null) {
            res = callback.doSomethingWithLine(line, res);
        }
        return res;
    }
    catch (IOException e) {
        // ...
    }
    finally {
        // ...
    }
}

//client
public class Calculator{
	public Integer calcSum(String filepath) throws IOException {
	    LineCallback sumCallback = 
	        new LineCallback() {
	            public Integer doSomethingWithLine(String line, Integer value) {
	                return value + Integer.valueOf(line);
	            }
	        };
	    return lineReadTemplate(filepath, sumCallback, 0);
	}
	
	public Integer calcMultiply(String filepath) throws IOException {
	    LineCallback multiplyCallback = 
	        new LineCallback() {
	            public Integer doSomethingWithLine(String line, Integer value) {
	                return value * Integer.valueOf(line);
	            }
	        };
	    return lineReadTemplate(filepath, multiplyCallback, 1);
	}
}
```

#### 제네릭스를 이용한 콜백 인터페이스 

반환 결과가 Integer가 아니라 다양한 type으로 하고 싶으면 제네릭 타입으로 선언하면 된다. 
이건 굳이 코드를 삽입하지 않겠다. 


## 3.6 스프링의 JdbcTemplate

### 3.6.1 update()

스프링이 제공하는 JDBC 코드용 기본 템플릿은 JdbcTemplate이다. 
이걸 사용해보자. 

```java 
public class UserDao{
	private JdbcTemplate jdbcTemplate;
	
	public void setDataSource(DataSource dataSource){
		this.jdbcTemplate = new JdbcTemplate(dataSource);
		this.dataSource = dataSource;
	}
	
	public void deleteAll() {
	    this.jdbcTemplate.update(
	        new PreparedStatementCreator() {
	            public PreparedStatement createPreparedStatement(Connection con)
	                    throws SQLException {
	                return con.prepareStatement("delete from users");
	            }
	        }
	    );
	}
}
```

PreparedStatementCreator 인터페이스의 createPreparedStatement 라는 메소드는 우리가 앞에서 작성했던 StateStrategy 인터페이스의 makePreparedStatement() 메소드랑 같은 역할을 한다.

```java
public void deleteAll(){
	this.jdbcTemplate.update("delete from users");
}
```

앞에서 모든 것을 다해주는 executeSql() 이라는 메서드를 만들었었는데 jdbcTemplate에서도 update라는 메서드가 비슷한 역할을 한다. 

### 3.6.2 queryForInt()

getCount라는 메서드에도 jdbcTemplate을 적용해보자.

여기서 쓸 수 있는 콜백은 2개이다. 
- PreparedStatementCreator
- ResultSetExtractor

첫 번째 PreparedStatementCreator 은 템플릿으로부터 Connection을 받아서 PreparedStatement를 돌려주는 역할을 하고,
ResultSetExtractor는 템플릿으로부터 쿼리의 결과인 ResultSet을 받아서 값을 추출해서 돌려준다. 

```java
public int getCount() {
    return this.jdbcTemplate.query(
        new PreparedStatementCreator() {
            public PreparedStatement createPreparedStatement(Connection con)
                    throws SQLException {
                return con.prepareStatement("select count(*) from users");
            }
        },
        new ResultSetExtractor<Integer>() {
            public Integer extractData(ResultSet rs) throws SQLException, DataAccessException {
                rs.next();
                return rs.getInt(1);
            }
        }
    );
}
```

jdbcTemplate의 query의 인자로 콜백을 2개 넘긴다. 
각각 PreparedStatementCreator 와 ResultSetExtractor 의 기능을 명시한 익명 구현체 클래스이다. 

ResultSetExtractor 은 제네릭 타입을 명시 받는다. 

이렇게 복잡한 구조도 jdbcTemplate에서는 queryForInt 라는 메서드를 제공해준다. 


```java
public int getCount(){
	return this.jdbcTemplate.queryForInt("select count(*) from users");
}
```

### 3.6.3 queryForObject()

여기서는 ResultSetExtractor 콜백이 아니라 RowMapper라는 콜백을 사용한다. 

RowMapper는 ResultSet의 로우 하나를 매핑하기 위한 콜백으로, 여러 번 호출될 수 있다. queryForObject는 row 가 하나인 것으로 기대한다. 

```java
public User get(String id) {
    return this.jdbcTemplate.queryForObject(
        "select * from users where id = ?",
        new Object[] {id},
        new RowMapper<User>() {
            public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                User user = new User();
                user.setId(rs.getString("id"));
                user.setName(rs.getString("name"));
                user.setPassword(rs.getString("password"));
                return user;
            }
        }
    );
}
```

첫 번째 인자는 쿼리다. 
두 번째 인자는 쿼리에 들어갈 파라미터 값이다. 
세 번째 인자는 ResultSet의 row 별로 어떻게 객체랑 매핑할 것인지에 대한 정보다.

### 3.6.4 query()

#### query() 템플릿을 이용하는 getAll() 구현

query() 메서드는 ResultSet의 결과가 row 하나가 아니라 여러개여도 된다. 

```java
public List<User> getAll() {
    return this.jdbcTemplate.query(
        "select * from users order by id",
        new RowMapper<User>() {
            public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                User user = new User();
                user.setId(rs.getString("id"));
                user.setName(rs.getString("name"));
                user.setPassword(rs.getString("password"));
                return user;
            }
        }
    );
}
```

query 템플릿은 SQL을 실행해서 얻은 ResultSet의 모든 로우를 열람하면서 로우마다 RowMapper 콜백을 호출한다. 

### 3.6.5 재사용이 가능한 콜백의 분리 

jdbcTemplate을 적용한 UserDao의 필요없는 코드 삭제 

- DI 를 위한 코드 정리
- 중복 제거
  RowMapper 에서 User 오브젝트에 mapping 하는 코드를 메서드로 분리시켰다. 