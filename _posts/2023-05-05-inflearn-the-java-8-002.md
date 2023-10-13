---
layout: single
title: "더 자바, Java 8 - 002"
categories:
  - "더 자바, Java 8"
tags:
  - "강좌"
---

[https://www.inflearn.com/course/the-java-java8/dashboard](https://www.inflearn.com/course/the-java-java8/dashboard)

## Optional

비어있을 수도 있고 값이 있을 수도 있는 상황

java 에서 종종 만나는 NullPointerException

- null 체크를 빼먹지 않고 개발하는 것은 힘들다.
- 또한 null 을 리턴하는 것 자체가 문제일 수도 있다.

아래는 개발중 값을 제대로 리턴할 수 없으면 선택하는 방법이다.

- 예외를 쓰는 것
    - 비용이 비싸다.
- null 을 리턴하는 것
    - 비용은 문제없다.
    - 클라이언트 코드에서 알아서 사용해야 한다.
- Optional 을 리턴하는 것
    - 명시적으로 빈 값이 올 수도 있음을 표현한다.
    - 클라이언트에서 빈 값인 경우에 대한 처리를 수행하도록 강제한다.

### 주의사항

- 리턴값으로만 쓰기를 권장함
  - 파라미터로 Optional 을 쓰면?
    - 있으면 메소드를 수행하고, 없으면 수행안하고 해야하나?
  - 맵의 키 타입
    - key 는 비어있을 수 없다.
  - 인스턴스 필드의 타입
    - 상태자체가 있을 수도 있고 없을 수도 있다?
- Optional 을 리턴하는 메소드에서 null 을 리턴하지 말 것
  - Optional.empty(); 로 리턴
- primitive 타입용 optional 이 따로 있다.
  - OptionalInt, OptionalLong
- Collection, Map, Stream Array, Optional 은 Optional 로 감싸지 말 것
  - 자체에 비어있음을 판단할 수 있는 컨테이너 성격의 타입들

### 참고

- [https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)
- [https://www.oracle.com/technical-resources/articles/java/java8-optional.html](https://www.oracle.com/technical-resources/articles/java/java8-optional.html)
- 이팩티브 자바 3판, 아이템 55 적절한 경우 Optional을 리턴하라.

## Optional API

### isPresent()

```java
Optional<OnlineClass> spring = springClasses.stream()
        .filter(oc -> oc.getTitle().startsWith("spring"))
        .findFirst();
        
boolean present = spring.isPresent(); // true
```

isEmpty(Java 11 부터)

### get()

optional 에서 값을 꺼내기

값이 없는데 get 을 하면 NoSuchElementException

그럴 때는 ifPresent 로 진행

```java
Optional<OnlineClass> spring = springClasses.stream()
        .filter(oc -> oc.getTitle().startsWith("AAA"))
        .findFirst();
        
OnlineClass optional = spring.get(); // 없으면 NoSuchElementException

// 권장되는 방법
optional.ifPresent(oc -> {
	System.out.println(oc.getTitle());
});
```

### orElse()

optional 에 값이 없을 때 새로 만들기

```java
// 없을 때 새로 만들기
OnlineClass onlineClass = optional.orElse(createNewOnlineClass());
```

인거 같았는데, 값이 있어도 createNewOnlineClass() 로 생성

### orElseGet()

optional 에 값이 null 일 때 새로 만들기

```java
// null 일 때 새로 만들기
OnlineClass onlineClass = 
    optional.orElseGet(App::createNewOnlineClass());
```

### 참고
- [https://ysjune.github.io/posts/java/orelsenorelseget](https://ysjune.github.io/posts/java/orelsenorelseget)
- [https://kdhyo98.tistory.com/40](https://kdhyo98.tistory.com/40)

### orElseThrow()

optional 에 값이 없을 때 throw

```java
// 없을 때 예외
OnlineClass onelineClass = 
    optional.orElseThrow(IllegalStateException::new);
```

### filter()

Optional 에 조건을 넣어서 조건이 맞으면 Optional 반환

```java
// 필터링
Optional<OnlineClass> onlineClass = 
    optional.filter(oc -> !oc.isClosed());
```

### map()

optional 을 변환

```java
Optional<Integer> integer = optional.map(OnlineClass::getId);
System.out.println(integer.isPresent());
```

### flatMap()

optional 안의 인스턴스가 optional 인 경우에 사용

### 참고
- [https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#put-K-V-](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#put-K-V-)

## Data 와 Time API

지금까지 사용된 java.util.Date 클래스는 mutable 해서 thread safe 하지가 않았다.

그래서 Java 8 부터는 새로운 Date-Time API 를 생성함

디자인 철학

- Clear
- Fluent
- Immutable
- Extensible

### Date/Time API

기계용 시간 (Epoch Time)

```java
Instant now = Instant.now();
System.out.println(now); // 현재시간의 UTC 를 리턴
System.out.println(now.atZone(ZoneId.of("UTC")));

ZonedDateTime zonedDateTime = now.atZone(ZoneId.systemDefault());
System.out.println(zonedDateTime); // 시스템의 타임존 시간을 리턴

ZonedDateTime zoneddateTimeOfUTC = now.atZone(ZoneId.of("UTC"));
System.out.println(zoneddateTimeOfUTC); // UTC 타임존 시간을 리턴
```

인간용 시간

- LocalDateTime.now(): 현재 시스템 Zone에 해당하는(로컬) 일시를 리턴한다.
- LocalDateTime.of(int, Month, int, int, int, int): 로컬의 특정 일시를 리턴한다.
- ZonedDateTime.of(int, Month, int, int, int, int, ZoneId): 특정 Zone의 특정 일시를 리턴한다.

### 기간 표현하기

Period

```java
LocalDate today = LocalDate.now();
LocalDate dDay = LocalDate.of(2023, Month.JULY, 1);
Period between = Period.between(today, dDay);
System.out.println(between.getDays()); // 일수

Period until = today.until(dDay); // 일수
System.out.println(between.get(ChronoUnit.DAYS)); // 일수
```

Duration 은 기계용 시간 비교 시 사용

### 포매팅

```java
LocalDate today = LocalDate.now();
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MM/d/yyyy");

System.out.println(today.format(formatter));

LocalDate date = LocalDate.parse("07/15/1982", formatter);
System.out.println(date);
```

### 레거시에서 변환

GregorianCalendar와 Date 타입의 인스턴스를 Instant나 ZonedDateTime으로 변환 가능

```java
Date date = new Date();
Instant instant = date.toInstant();
Date newDate = Date.from(instant);

GregorianCalendar gregorianCalendar = new GregorianCalendar();
ZonedDateTime dateTime = gregorianCalendar.toInstant().atZone(ZoneId.systemDefault());
GregorianCalendar from = GregorianCalendar.from(dateTime);

ZoneId zoneId = TimeZone.getTimeZone("PST").toZoneId();
TimeZone timeZone = TimeZone.getTimeZone(zoneld);
```

### 참고
- [https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#predefined](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#predefined)
- [https://jcp.org/en/jsr/detail?id=310](https://jcp.org/en/jsr/detail?id=310)
- [https://codeblog.jonskeet.uk/2017/04/23/all-about-java-util-date/](https://codeblog.jonskeet.uk/2017/04/23/all-about-java-util-date/)
- [https://docs.oracle.com/javase/tutorial/datetime/overview/index.html](https://docs.oracle.com/javase/tutorial/datetime/overview/index.html)
- [https://docs.oracle.com/javase/tutorial/datetime/iso/overview.html](https://docs.oracle.com/javase/tutorial/datetime/iso/overview.html)

## CompletableFuture

Concurrent 소프트웨어란 동시에 여러 작업을 할 수 있는 소프트웨어를 말한다.

자바에서 지원하는 Concurrent 프로그래밍은 멀티 프로세싱과 멀티 쓰레드

### 자바 멀티쓰레드 프로그래밍

Thread 클래스를 상속받아 만드는 방법

```java
public static void main(String[] args) {
    HelloThread helloThread = new HelloThread();
    helloThread.start();
    System.out.println("hello : " + Thread.currentThread().getName());
}

static class HelloThread extends Thread {
    @Override
    public void run() {
        System.out.println("world : " + Thread.currentThread().getName());
    }
}
```

Runnable 인터페이스를 구현하는 방법

```java
// Runnable 인터페이스의 람다식
Thread thread = new Thread(() -> {
    System.out.println("world : " + Thread.currentThread().getName());
});
thread.start();
System.out.println("hello : " + Thread.currentThread().getName());
```


쓰레드의 주요 기능

- sleep():
  - 현재 쓰레드 멈춰두기
  - 다른 쓰레드가 처리할 수 있도록 기회를 주지만 그렇다고 락을 놔주진 않는다. (잘못하면 데드락 걸릴 수 있겠죠.)
- interupt():
  - 다른 쓰레드 깨우기
  - 다른 쓰레드를 깨워서 interruptedExeption을 발생
  - 그 에러가 발생했을 때 할 일은 코딩하기 나름. 종료 시킬 수도 있고 계속 하던 일 할 수도 있고.
- join():
  - 다른 쓰레드 기다리기
  - 다른 쓰레드가 끝날 때까지 기다린다.

### 참고
- [https://docs.oracle.com/javase/tutorial/essential/concurrency/](https://docs.oracle.com/javase/tutorial/essential/concurrency/)
- [https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#interrupt--](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#interrupt--)

이렇게 쓰레드이 기능이 너무 어렵기 때문에 Executors 가 나오게 되었음.

### Executors

High-Level Concurrency 프로그래밍을 위해 사용하는 인터페이스

쓰레드를 만들고 관리하는 작업을 애플리케이션에서 분리해서 위임

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
executorService.submit(() -> {
    System.out.println("Hello :" + Thread.currentThread().getName());
});



// 처리중인 작업 기다렸다가 종료
executorService.shutdown(); 

// 당장 종료
executorService.shutdownNow(); 
```

ScheduledExecutorService 도 있음

고정된 쓰레드풀에서 여러개의 쓰레드 만들려면 newFixedThreadPool

```java
ExecutorService executorService = Executors.newFixedThreadPool(2);
```

ExecutorService 내부에 Blocking Queue 가 있다.

Blocking Queue 에 작업을 쌓아두고 Thread 에 작업을진행함

Runnable 은 리턴값이 없어서 작업의 결과를 받아올 수 없다.

그래서 리턴값이 필요할 때는 Callable 을 사용한다.

### 참고

- [https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html](https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html)

## Callable 과 Future

ExecutorService 를 이용해서 Callable 을 수행할 수 있다.

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();

Callable<String> hello = new Callable<String>() {
	@Override
    public String call() throws Execption {
    	return "Hello";
    }
};

// Funcional Interface
// Callable<String> hello = () -> "Hello";

Future<String> helloFuture = executorService.submit(hello);

helloFuture.isDone();

helloFuture.get();
```

isDone() 메소드로 동작 상태를 알 수 있다. true / false

cancel() 메소드로 작업을 취소할 수 있다.

get() 메소드로 결과를 받아올 수 있다.
get() 메소드는 blocking call 이므로 대기시간이 발생한다.

invokeAll() 메소드는 모든 쓰레드의 작업이 끝난 후 이후 작업을 수행한다.

```java
ExecutorService executorService = Executors.newFixedThreadPool(4);

List<Future<String>> futures = executorService.invokeAll(Arrays.asList(/* 여러 쓰레드 */));

for (Future<String> f : futures) {
	System.out.println(f.get());
}
```

invokeAny() 메소드는 쓰레드 중 작업이 빠르게 끝난 Future 가 있으면 먼저 값을 반환하고, 작업을 수행한다.

### 참고

- [https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html)

## CompletableFuture

Future 는 Future 의 get() 메소드를 호출하기 전에는 다른 작업을 수행할 수 없는 문제가 있다.

```java
Future<String> future = executorServicei.submit(() -> "hello");

future.get(); // future.get() 을 호출하기 전에는 다른 작업을 수행할 수 없다.
```

이 외의 다른 문제들

Future로는 하기 어렵던 작업들

- Future를 외부에서 완료 시킬 수 없다. 취소하거나, get()에 타임아웃을 설정할 수는 있다.
- 블로킹 코드(get())를 사용하지 않고서는 작업이 끝났을 때 콜백을 실행할 수 없다.
- 여러 Future를 조합할 수 없다, 예) Event 정보 가져온 다음 Event에 참석하는 회원 목록 가져오기
- 예외 처리용 API를 제공하지 않는다.

그래서 CompletableFuture 가 나왔다.

ComapletableFuture 는 자바에서 비동기(Asynchronous) 프로그래밍을 가능케하는 인터페이스이다.

```java
CompletableFuture<String> future = new CompletableFuture<>();
future.complete("keesun");

future.get();
```

비동기로 작업 실행하기
- 
- 리턴값이 없는 경우: runAsync()
- 리턴값이 있는 경우: supplyAsync()
- 원하는 Executor(쓰레드풀)를 사용해서 실행할 수도 있다. (기본은 ForkJoinPool.commonPool())

```java
- // 리턴값이 없는 경우
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    System.out.println("hello");
});
future.get();

// 리턴값이 있는 경우
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "hello";
});
futurue.get();
```

콜백을 이용하여 비동기적인 수행을 할 수 있다.

콜백 제공하기

- thenApply(Function): 리턴값을 받아서 다른 값으로 바꾸는 콜백
- thenAccept(Consumer): 리턴값을 또 다른 작업을 처리하는 콜백 (리턴없이)
- thenRun(Runnable): 리턴값 받지않고 다른 작업을 처리하는 콜백
- 콜백 자체를 또 다른 쓰레드에서 실행할 수 있다.

```java
// 리턴값이 없는 경우
CompletableFuture<String> future = CompletableFuture.runAsync(() -> {
    return "hello";
}).thenApply((s) -> {
    return s.toUpperCase();
})
future.get();

// 리턴값이 있는 경우
CompletableFuture<String> future = CompletableFuture.runAsync(() -> {
    return "hello";
}).thenAccept((s) -> {
    s.toUpperCase();
})
future.get();

// 리턴값을 받지 않는 경우
CompletableFuture<String> future = CompletableFuture.runAsync(() -> {
    return "hello";
}).thenRun(() -> {
    System.out.println("hello");
})
future.get();
```

### 참고
- [https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)
- [https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html)
- [https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html)
- [https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)

조합하기

- thenCompose(): 두 작업이 서로 이어서 실행하도록 조합
- thenCombine(): 두 작업을 독립적으로 실행하고 둘 다 종료 했을 때 콜백 실행
- allOf(): 여러 작업을 모두 실행하고 모든 작업 결과에 콜백 실행
- anyOf(): 여러 작업 중에 가장 빨리 끝난 하나의 결과에 콜백 실행 

thenCompose() : 두 작업을 서로 이어서 실행하도록 조합하기

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        System.out.println("Hello" + Thread.currentThread().getName ());
        return "Hello";
    });
    
    CompletableFuture<String> future = hello.thenCompose(App::getWorld);
    
    System.out.printin(future.get());
}
    
private static CompletableFuture<String> getWorld(String message) {
    return CompletableFuture.supplyAsync(() -› {
        System.out.printin("World" + Thread.currentThread().getName ());
        return message + " World";
    });
}
```

thenCombine() : 서로 관계가 없는 작업을 조합하기

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        System.out.println("Hello" + Thread.currentThread().getName ());
        return "Hello";
    });
    
    CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> {
        System.out.println("World" + Thread.currentThread().getName ());
        return "World";
    });
    
    CompletableFuture<String> future = hello.thenCombine(world, (h, w) -> {
        return h + " " + w;
    })
    future.get();
}
```

allOf() : 여러 작업을 모두 실행하고 모든 작업 결과에 콜백 실행

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        System.out.println("Hello" + Thread.currentThread().getName ());
        return "Hello";
    });
    
    CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> {
        System.out.println("World" + Thread.currentThread().getName ());
        return "World";
    });
    
    List<CompetableFuture> futures = Arrays.asList(hello, world);
    CompletableFuture[] futuresArray = futures.toArray(new CompletableFuture[futures.size()]);
    
    CompletableFuture<List<Object>> futureResults = CompletableFuture.allOf(futuresArray).thenArray(v -> {
        return futures.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.toList()
    });
    
    futureResults.get();
}
```

anyOf() : 여러 작업 중에 가장 빨리 끝난 하나의 결과에 콜백 실행


```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        System.out.println("Hello" + Thread.currentThread().getName ());
        return "Hello";
    });
    
    CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> {
        System.out.println("World" + Thread.currentThread().getName ());
        return "World";
    });
    
    CompletableFuture<Void> future = CompletableFuture.anyOf(hello, world).thenAccept(System.out::println);
    future.get();
}
```

예외처리하기

- exeptionally(Function)
- handle(BiFunction)

exceptionally

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    boolean throwError = true; // 강제로 에러 발생
    
    CompletableFutre<String> hello = CompletableFuture.supplyAsync(() -> {
        if (throwError) {
            throw new IllegalArgumentException();
        }
    
        System.out.println("Hello " + Thread.currentThread().getName());
        return "Hello";
    }).exceptionally(ex -> {
        return "Error!";
    });
    
    System.out.println(hello.get());
}
```

handle(정상적인 결과값, 에러)

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    boolean throwError = true; // 강제로 에러 발생
    
    CompletableFutre<String> hello = CompletableFuture.supplyAsync(() -> {
        if (throwError) {
            throw new IllegalArgumentException();
        }
    
        System.out.println("Hello " + Thread.currentThread().getName());
        return "Hello";
    }).handle((result, ex) -> {
        if (ex != null) {
            System.out.println(ex);
            return "Error!";
        }
        return result;
    });
    
    System.out.println(hello.get());
}
```

## 그밖에

### 애노테이션의 변화

-  자바 8부터 애노테이션을 타입 선언부에도 사용할 수 있게 됨.
- 자바 8부터 애노테이션을 중복해서 사용할 수 있게 됨.

TYPE_PARAMETER : 애노테이션을 타입에 붙일 수 있게 되었다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE_PARAMETER)
public @interface Chicken {

}

// ...

static class FeelsLikeChicken<@Chicken T> {

    public static <@Chicken C> void print(C c) {
        
    }
}
```

TYPE_USE : 모든 타입에 붙일 수 있다.

```java
public static void main(@Chicken String[] args) throws @Chicken RuntimeException {
    List<@Chicken String> names = Arrays.asList("keesun");
}
```

중복 사용할 수 있는 애노테이션

- 중복 사용할 애노테이션 만들기
- 중복 애노테이션 컨테이너 만들기
- 컨테이너 애노테이션은 중복 애노테이션과 @Retention 및 @Target이 같거나 더 넓어야 한다.

애노테이션

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE_USE)
@Repeatable(ChickenContainer.class)
public @interface Chicken {
    String value();
}
```

애노테이션 컨테이너 애노테이션


```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE_USE)
public @interface ChickenContainer {
    Chicken[] value();
}
```

사용법

```java
@Chicken("양념")
@Chicken("간장")
public static void main(String[] args) {
    // 애노테이션
    Chicken[] chickens = App.class.getAnnotationsByType(Chicken.class);
    Arrays.stream(chicken).forEach(c -> {
        System.out.println(c.value());
    });
    
    // 애노테이션 컨테이너 애노테이션
    ChickenContainer chickenContainer = App.class.getAnnotation(ChickenContainer.class);
    Arrays.stream(chickenContainer.value()).forEach(c -> {
        System.out.println(c.value());
    });
}
```

## 배열 정렬

Arrays.parallelSort() : Fork/Join 프레임워크를 이용한 배열 병렬 정렬 기능

병렬 정렬 알고리즘

- 배열을 적당한 사이즈까지 계속 양분한다.
- 합치면서 정렬한다.

```java
int size = 1500;
int[] numbers = new int[size];
Random random = new Random();
IntStream.range(0, size).forEach(i -> numbers[i] = random.nextInt());

long start = System.nanoTime();
Arrays.sort(numbers);
System.out.println("serial sorting took " + (System.nanoTime() - start));

IntStream.range(0, size).forEach(i -> numbers[i] = random.nextInt());
start = System.nanoTime();
Arrays.parallelSort(numbers);
System.out.println("parallel sorting took " + (System.nanoTime() - start));
```

알고리즘 효율은 같다.

시간: O(n logN), 공간: O(n)

### Metaspace

Java 8 부터 바뀐거

JVM의 여러 메모리 영역 중에 PermGen 메모리 영역이 없어지고 Metaspace 영역이 생김

PermGen(Permanent Generation)

- 클래스 메타데이터를 담는 곳
- Heap 영역에 속함
- 기본값으로 제한된 크기를 가지고 있음 -> 동적으로 많은 클래스를 만들면 에러 발생
- -XX:PermSize=N, PermGen 초기 사이즈 설정
- -XX:MaxPermSize=N, PermGen 최대 사이즈 설정

Metaspace

- 클래스 메타데이터를 담는 곳.
- Heap 영역이 아니라, Native 메모리 영역
- 기본값으로 제한된 크기를 가지고 있지 않음 -> 필요한 만큼 계속 늘어남
- 자바 8부터는 PermGen 관련 java 옵션은 무시
- -XX:MetaspaceSize=N, Metaspace 초기 사이즈 설정
- -XX:MaxMetaspaceSize=N, Metaspace 최대 사이즈 설정

## 참고

- [http://mail.openjdk.java.net/pipermail/hotspot-dev/2012-September/006679.html](http://mail.openjdk.java.net/pipermail/hotspot-dev/2012-September/006679.html)
- [https://m.post.naver.com/viewer/postView.nhn?volumeNo=23726161&memberNo=36733075](https://m.post.naver.com/viewer/postView.nhn?volumeNo=23726161&memberNo=36733075)
- [https://m.post.naver.com/viewer/postView.nhn?volumeNo=24042502&memberNo=36733075](https://m.post.naver.com/viewer/postView.nhn?volumeNo=24042502&memberNo=36733075)
- [https://dzone.com/articles/java-8-permgen-metaspace](https://dzone.com/articles/java-8-permgen-metaspace)
- [https://johngrib.github.io/wiki/java8-why-permgen-removed/](https://johngrib.github.io/wiki/java8-why-permgen-removed/)
- [https://openjdk.org/jeps/122](https://openjdk.org/jeps/122)
