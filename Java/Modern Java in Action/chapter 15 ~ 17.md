
## Chapter 17 - CompletableFuture와 리액티브 프로그래밍 컨셉의 기초
---

프로그래밍이 획기적으로 변하기 시작한데는 2가지 큰 이슈가 있다. 
1. 멀티 코어 HW의 등장으로 인한 병렬 프로그래밍
2. MSA, Open API로 인해서 네트워크 통신 증가

이런 상황에서 마주할 수 있는 문제가 어떤 것이 있을까?

바로 ==동시성 제어==이다. 하나의 스레드가 특정 스레드의 작업을 기다리기 위해 block 상태가 되는 것은 cpu 클럭 자원 낭비이다. 

이 장에서는 자바 8에서 등장한 Future 인터페이스를 사용해서 CompetableFuture를 구현하여 동시성을 제어하는 것을 배운다.

### 자바의 동시성 제어 발전

처음엔 Runnable, Thread를 동기화된 클래스와 메서드로 잠갔다. 
이후 자바 5에서는 ExecutorService, Callable`<`T`>`, Future`<`T`>`를 지원하여 동시성을 지원했다. 
이후 멀티코어 CPU가 등장하면서 병렬 프로그래밍에서의 기능이 필요했고, 자바 7 에서는 divide & conquer 알고리즘의 fork/join 을 지원하는 ==java.util.concurrent.RecursiveTask== 가 추가되었다. 
이후에 자바 9에서는 ==java.util.concurrent.Flow== 가 추가되어 발행-구독 프로토콜을 지원해준다. 

예시
1,000,000 크기의 배열에 저장된 숫자의 합을 구하는 로직을 단순하게 작성하면 다음과 같다. 
```java
long sum =0;
for(int i =0; i< 1_000_000; i++){
	sum += arr[i];
}
```

이걸 4개의 스레드로 병렬처리 하면 다음과 같다. 
```java
long sum0 =0;
for(int i =0; i< 250_000; i++){
	sum0 += arr[i];
}

long sum1 =0;
for(int i =250_000; i< 500_000; i++){
	sum1 += arr[i];
}

...
```

근데 stream api를 사용하면 다음과 같이 추상화 시킬 수 있다. 
```java
sum = Arrays.stream(arr).parallel.sum();
```

자바 5에서 제공하는 ExecutorService로도 추상화가 가능하다. 그전에 ExecutorService의 개념과 thread pool이 뭔지부터 배워야한다.

### Executor와 스레드 풀

thread의 문제가 뭘까? 
보통 HW마다 최대 thread가 정해져있다. 근데 무분별하게 thread를 생성하면 속도가 병렬 프로그래밍을 하는 것보다 오히려 저하될 것이다. 

이때 thread pool을 사용하면 좋다. 
미리 thread를 생성해두고 사용자가 task를 전달하면 가용가능한 thread에게 task를 할당해주면 되는 것이다. 

ExecutorService는 task를 제출하면 나중에 결과를 수집할 수 있는 interface를 제공한다. 다음과 같은 팩토리 메서드를 사용해서 ExecutorService를 생성할 수 있다. 
```java
ExecutorService newFixedThreadPool(int nThreads)
```

근데 thread pool의 단점도 존재한다.
thread pool이 생성한 thread 보다 많은 task가 들어오면 초과된 task는 기다려야 한다. 특히나 I/O 작업을 기다리는 task가 있으면 thread를 혼자 오래동안 차지하므로 비효율적일 수 있다. 

### setDaemon() 메서드

main 메서드가 특정 thread를 실행하고, 특정 thread보다 main 함수가 먼저 종료될 때 생성한 thread는 어떻게 처리해야 할까? 두 가지 방법이 있다. 

1. 생성한 thread가 종료될 때까지 기다리기
2. 강제 종료 시키기

setDaemon() 메서드를 사용해 생성한 thread를 daemon thread로 만들면 main 메서드가 종료될 때 thread를 강제종료 시킬 수 있고, daemon thread가 아닌 경우에는 main 메서드가 기다린다. 

### 비동기 API 

```java 
int y = f(x);
int z = g(x);
System.out.println(y+z);
```

위의 코드를 f(x), g(x) 에 대해서 병렬로 처리하도록 만들고 싶다. 그리고 main thread는 f(x), g(x)가 완료되도록 기다려야 한다. 

Runnable을 사용해서 다음과 같이 처리할 수 있다. 
```java
public static void main(String[] args) throws InterruptedException {
	int X = 1337;
	Result result = new Result();
	
	Thread t1 = new Thread(() -> { result.left = f(x); });
	Thread t2 = new Thread(() -> { result.right = g(x); });
	t1.start();
	t2.start();
	t1.join();
	t2.join();
	System.out.printin(result.left + result.right)
}

private static class Result {
	private int left;
	private int right;
)
```

간단한 코드가 굉장히 복잡해졌다. 
Runnable 대신 Future API와 thread pool을 사용하면 다음과 같이 만들 수 있다. 
```java 
public static void main(String[] args) throws ExecutionException, InterruptedException {

	int X = 1337;
	
	ExecutorService executorService = Exec니tors.newFixedThreadPo이(2);
	Future<Integer> y = executorService.submit(() -> f(x));
	Future<Integer> z = executorService.submit(() -> g(x));
	System.out.println(y.get() + z,get());
	
	exectorService.shtdown();
}
```

Future는 executor 스레드 풀이 반환하는 인터페이스로, 비동기 thread 처리를 위한 것이다. 비동기 작업의 상태를 알 수 있도록 다양한 메서드를 지원해준다. y.get(), z.get() 메서드에서는 각 future 의 작업이 완료되기를 기다린다.

위의 코드도 명시적인 submit 메서드 호출로 인해 오염되었다고 책은 말한다. stream api에서 외부 반복을 내부 반복으로 바꾼 것처럼 이 문제를 해결해야 한다. 

1. 자바 8에서는 Future의 기능을 확대한 CompletableFuture를 제공해주어서 이 문제를  해결한다. 
2. 자바 9에서는 Flow 인터페이스를 제공해서 발행-구독 프로토콜로 해결한다.

각 클래스가 어떤 식으로 해결하는지는 16, 17 장에서 살펴보고 각 방식을 직접 구현해보자. 

첫 번째로 Future 형식의 API를 활용하는 것을 유지하고 f(), g()의 방식을 바꾸어서 다음과 같이 표현할 수 있다. 
```java 
Future<Integer> f(int x){
	...
}

Future<Integer> g(int x){
	...
}

Future<Integer> y = f(x);
Future<Integer> z = g(x);
System.out.println(y.get() + z.get());
```

두 번째 방식을 활용한 발행-구독 프로토콜의 리액티브 형식의 API는 다음과 같이 표현된다. 
```java
void f(int x, IntConsumer y){
	...
}

void g(int x, IntConsumer y){
	...
}

f(x, (int y) -> {
	result.left = y;
	System.out.println(result.left + result.right);
})
g(x, (int z) -> {
	result.right = z;
	System.out.println(result.left + result.right);
})
```

두 방법 모두 코드를 복잡하게 만든다. 하지만 명시적으로 스레드를 처리하는 코드에 비해 사용코드를 단순하게 만들어준다. (= 코드의 높은 수준을 유지할 수 있게 해준다.) 

### 그리고 또 블록킹에 대해서

스레드 풀을 사용함에 있어서 block 되는 thread를 경계해야 한다고 앞에서 언급했다. 근데 thread를 사용하다보면 block 되는 경우는 다반사다. 이를 특정 디자인 패턴으로 해결해보자. 

다음과 같은 task가 있다고 하자. 
```java
work1();
Thread.sleep(10000);
work2();
```

이 task를 thread pool에 submit 해서 실행하면 해당 thread는 10초 동안 block 상태로 있을 것이다. 

근데 다음과 같이 작성한다면 어떨까?
```java
ScheduledExecutorService scheduleExecutorService = Executors.newScheduledThreadPool(1);

work1();
scheduledExecutorService.schedule(ScheduledExecutorServiceExample::work2, 10, TimeUnit.SECONDS);

scheduledExecutorService.shutdown();
```

work1 이 끝난 뒤에 thread를 반환하고, work2를 10초 뒤에 task 큐에 추가하는 것이다. 

즉 처음 코드는 10초를 기다리는 동안 thread를 점유하고 있는 것이고, 두 번째 코드는 work1 끝나고 thread 반환, 10초 기다리고, work2를 위한 thread를 다시 할당 받는 것이다. 

이런 디자인 패턴은 코드를 복잡하게 만든다. 자바 8에서 제공하는 CompletableFuture 인터페이스는 이런 코드를 추상화해서 제공한다. 

>[!info] 네트워크 프로그래밍에서 배웠던 nio 는 java 1.4에서 제공하는 방법인데 복잡해서 잘 사용하지 않는다고 한다. 

>[!tip] 네트워크 서버의 블록/비볼록 API를 제공하는 Netty 라는 라이브러리를 사용하는 것도 도움이 된다.

### 그외에..

비동기 API에서의 예외처리도 CompletableFuture와 Flow 별로 설명해준다. 해당 클래스 사용 x, 구현 느낌으로 

### 동시성 처리

위에서 계속 언급한 것처럼 Future만으로 동시성을 제어하는 데는 많은 문제가 있다. 
1. 명시적으로 thread를 제어 해주어야 한다. 
2. get() 호출이 많아지면 큰일
3. 비동기 처리
4. thread가 block 되면 성능 저하
5. thread의 예외 처리

CompletableFuture에서는 compose(), andThen() 같은 ==콤비네이터==를 사용해서 동시성 제어를 쉽게 한다. 
