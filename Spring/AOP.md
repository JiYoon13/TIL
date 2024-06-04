## AOP

### AOP가 필요한 상황

* 모든 메소드의 호출 시간을 축정하고 싶다면 ?
* 공통 관심 사항 (cross-cutting concern) vs 핵심 관심 사항 (core concern)
* 회원 가입 시간, 회원 조회 시간을 측정하고 싶다면 ?



### AOP 적용

* AOP : Aspect Oriented Programming
* 공통 관심 사항과 핵심 관심 사항을 분리



*TimeTraceAop*

`````java
@Aspect
@Component
// @Component 써도 되고 직접 Spring Bean에 등록해도 됨
public class TimeTraceAop {
  
  @Around("execution(* hello.hellospring..*(..))")
  // hello > hellospring 패키지 하위에 모두 적용
	public Object execute(ProceedingJoinPoint joinPoint) trows Throwable {
		long start = System.currentTimeMillis();
		System.out.println("START: " + joinPoint.toString());
		try {
			return joinPoint.proceed();
		} finally {
			long finish = System.currentTimeMillis();
			long timeMs = finish - start;
			System.out.println("END : " + joinPoint.toString() + " " + timeMs + "ms");
		}
		
	}
}
`````

* TimeTraceAop를 적용하며 핵심 관심 사항과 시간을 측정하는 공통 관심 사항을 분리
* 시간을 측정하는 로직을 별도의 공통 로직으로 만듬
* 핵심 관심 사항을 깔끔하게 유지 가능
* 변경이 필요하면 TimeTraceAop 만 수정하여 관리 가능
* 원하는 적용 대상을 선택 가능



### AOP 동작 방식

* AOP 적용 전 의존 관계
    * helloController -> memberService
* AOP 적용 후 의존 관계
    * helloController -> (proxy) memberService -> memberService
    * memberService 복제 본인 가짜 memberService (proxy)에서 aop가 실행이 되고
    * joinPoint.proceed()를 하면 실제 서비스가 호출됨



### 부가기능이 적용되는 시점의 차이

* 컴파일 시점 : 실제 대상 코드에 Aspect를 통한 부가 기능 호출 코드가 포함됨. AspectJ를 직접 사용해야 함
    * 바이트 코드를 실제 조작해 실제 코드에 부가 기능 코드가 추가됨
* 클래스 로딩 시점 : 실제 대상 코드에 Aspect를 통한 부가 기능 호출 코드가 포함됨. AspectJ를 직접 사용해야 함
    * 바이트 코드를 실제 조작해 실제 코드에 부가 기능 코드가 추가됨
* 런타임 시점 : 실제 대상 코드는 그대로 유지. 대신 프록시를 통해 부가 기능이 적용되어 항상 프록시를 통해야 부가 기능을 사용 가능
    * 스프링 AOP 는 런타임 시점 사용
    * 런타임 시점에 프록시를 사용하여 AOP를 적용



### AOP 적용 위치

* 적용 가능 지점 (JoinPoint) : 생성자, 필드 값 접근, static 메서드 접근, 메서드 실행
    * 컴파일 시점, 클래스 로딩 시점은 바이트 코드를 실제 조작하기 때문에 모든 지점에 다 적용 가능
    * 프록시 방식을 사용하는 스프링 AOP는 **메서드 실행 지점**에만 AOP 적용 가능
        * 프록시 방식을 사용하는 스프링 AOP는 스프링 컨테이너가 관리할 수 있는 스프링 빈에만 AOP 적용 가능



### 포인트컷, 어드바이스, 어드바이저

* 포인트컷(Pointcut) : 어디에 부가 기능을 적용할지, 적용하지 않을 지 판단하는 필터링 로직
    * 주로 클래스와 메서드 이름으로 필터링
* 어드바이스(Advice) : 프록시가 호출하는 부가 기능(프록시 로직)
* 어드바이저(Advisor) : 포인트컷1 + 어드바이스1
* 부가 기능 로직을 적용할 때, 포인트컷으로 **어디에** 적용할지 선택하고, 어드바이스로 **어떤 로직**을 적용할지 선택



### AOP 용어 정리

* 조인프인트 (JoinPoint)
    * 어드바이스가 적용될 수 있는 위치, 메소드 실행, 생성자 호출, 필드 값 접근, static 메서드 접근 같은 프로그램 실행 중 지점
    * AOP를 적용할 수 있는 모든 지점
    * **스프링 AOP는 프록시 방식을 사용하므로 항상 메소드 실행 지점으로 제한됨**
* 포인트컷 (Pointcut)
    * 조인포인트 중에서 어드바이스가 적용될 위치를 선별하는 기능
    * 주로 ApsectJ 표현식을 사용해서 지정

* 타켓 (Target)
    * 어드바이스를 받는 객체
    * 포인트컷으로 결정
* 어드바이스 (Advice)
    * 부가기능
    * @Around, @Before, @After
* 애스팩트 (Aspect)
    * 어드바이스 + 포인트컷을 모듈화 한 것
    * @Aspect
    * 여러 어드바이스와 포인트컷이 함께 존재 가능
* 어드바이저 (Advisor)
    * 특별한 Aspect
    * 하나의 어드바이스와 하나의 포인트컷으로 구성
    * 스프링 AOP에서만 사용되는 특별한 용어
* 위빙 (Weaving)
    * 포인트컷으로 결정한 타켓의 조인포인트에 어드바이스를 적용하는 것
* AOP 프록시
    * AOP 기능을 구현하기 위해 만든 프록시 객체
    * 스프링 AOP 프록시는 JDK 동적 프록시 또는 CGLIB 프록시





### @Aspect로 AOP 구현하는 방법

`````java
@Aspect
@Component
public class AspectV1 {
  @Around("execution(* hello.aop.order..*(..))")
  public Object doLog(ProceddingJoinPoint joinPoint) throws Throwable {
    // join point 시그니처 : void hello.aop.order.OrderService.orderItem(String) 
    log.info("[log] {}", joinPoint.getSignature());
    // joinPoint.proceed()를 해야 target이 호출됨
    return joinPoint.proceed();
  }
}
`````

* 포인트컷 : execution(* *hello.aop.order..*(..))
* 어드바이스 : doLog()



### 포인트컷 분리

`````java
@Aspect
@Component
public class AspectV2 {
	// hello.aop.order 패키지의 하위 패키지 
  @Pointcut("execution(* hello.aop.order..*(..))")
  private void allOrder() {} // pointcut signature
  
  @Around("allOrder()")
  public Object doLog(ProceddingJoinPoint joinPoint) throws Throwable {
    // join point 시그니처 : void hello.aop.order.OrderService.orderItem(String) 
    log.info("[log] {}", joinPoint.getSignature());
    // joinPoint.proceed()를 해야 target이 호출됨
    return joinPoint.proceed();
  }
}
`````

* @PointCut에 포인트컷 표현식 사용
* 메서드 이름과 파라미터를 합쳐 포인트컷 시그니처라 함
* 반환 타입은 void 여야하고 코드 내용은 비어야 함
* Public으로 접근제어자 설정하면 다른 클래스에 있는 외부 어드바이스에서도 사용 가능



### 어드바이스 추가

`````java
@Aspect
@Component
public class AspectV2 {
	// hello.aop.order 패키지의 하위 패키지 
  @Pointcut("execution(* hello.aop.order..*(..))")
  private void allOrder() {} // pointcut signature
  
  
  // 클래스 이름 패턴이 *Service 
  @Pointcut("execution(* *..*Service.(..))")
	private void allService() {}
  
  
  // hello.aop.order 패키지와 하위 패키지이면서 동시에 클래스 이름 패턴이 *Service
  @Around("allOrder() && allService()")
  public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
    try {
      log.info("[트랜잭션 시작] {}, joinPoint.getSignature()");
      Object result = joinPoint.proceed();
    	log.info("[트랜잭션 커밋] {}, joinPoint.getSignature()");	
      return result;
    } catch (Exception e) {
      log.info("[트랜잭션 롤백] {}, joinPoint.getSignature()");	
    } finally {
      log.info("[리소스 릴리즈] {}, joinPoint.getSignature()");	
    }
  }
}
`````

* && (AND) 가능
* || (OR) 가능
* ! (NOT) 가능



### 포인트컷 참조

* 별도의 외부 클래스에 포인트컷들만 모아 두는 방법

`````java
public class pointcuts {
  
  @Pointcut("execution(* hello.aop.order..*(..))")
  public void allOrder() {}
  
  @Pointcut("execution(* *..*Service.(..))")
	public void allService() {}
  
  @Pointcut("allOrder() && allService()")
  public void orderAndService() {}
}
`````

`````java
@Aspect
public class AspectV4Pointcut {
  @Around("hello.aop.order.aop.Pointcuts.allOrder()")
  public Object doLog(ProceddingJoinPoint joinPoint) throws Throwable {
    // join point 시그니처 : void hello.aop.order.OrderService.orderItem(String) 
    log.info("[log] {}", joinPoint.getSignature());
    // joinPoint.proceed()를 해야 target이 호출됨
    return joinPoint.proceed();
  }
}
`````





### 어드바이스 순서

* 어드바이스는 기본적으로 순서 보장 X
* 순서를 지정하고 싶으면 여러 애스팩트로 분리해야 함

`````java
public class AspectV4Order {
  @Aspect
  @Order(2)
  public static class LogAspect {
    @Around("hello.aop.order.aop.Pointcuts.allOrder()")
  	public Object doLog(ProceddingJoinPoint joinPoint) throws Throwable {
      log.info("[log] {}", joinPoint.getSignature());
      return joinPoint.proceed();
  	}
  }
  
  @Aspect
  @Order(1)
  public static class TxAspect {
    @Around("allOrder() && allService()")
  	public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
      try {
        log.info("[트랜잭션 시작] {}, joinPoint.getSignature()");
        Object result = joinPoint.proceed();
        log.info("[트랜잭션 커밋] {}, joinPoint.getSignature()");	
        return result;
      } catch (Exception e) {
        log.info("[트랜잭션 롤백] {}, joinPoint.getSignature()");	
      } finally {
        log.info("[리소스 릴리즈] {}, joinPoint.getSignature()");	
      }
  	}
  }
}
`````





### 어드바이스 종류

* @Around
    * 메소드 호출 전 후에 수행
    * 가장 강력한 어드바이스
    * 조인 포인트 실행 여부 선택 : joinPoint.proceed()
    * 전달 값 변환 :  joinPoint.proceed(args[])
    * 반환 값 변환
    * 예외 변환
    * 트랜잭션 처럼 try ~ catch ~ finally 모두 들어가는 구문 처리 가능
* @Before
    * 조인 포인트 실행 이전에 실행
* @After Returning
    * 조인 포인트가 정상 완료 후 실행
* @After Throwing
    * 메서드가 예외를 던지는 경우 실행
* @After
    * 조인 포인트가 정상 또는 예외에 관계없이 실행 (finally)



`````java
  @Around("allOrder() && allService()")
  public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
    try {
      // @Before
      log.info("[트랜잭션 시작] {}, joinPoint.getSignature()");
      Object result = joinPoint.proceed();
      
      // @After Returning
      log.info("[트랜잭션 커밋] {}, joinPoint.getSignature()");	
      return result;
    } catch (Exception e) {
      // @After Throwing
      log.info("[트랜잭션 롤백] {}, joinPoint.getSignature()");	
    } finally {
      // @After
      log.info("[리소스 릴리즈] {}, joinPoint.getSignature()");	
    }
  }
`````



`````java
@Aspect
@Component
public class AspectV6Advice {
  /**
  	* @Before
  	*/
	@Before("hello.aop.order.aop.Pointcuts.orderAndService()")
  public void doBefore(JoinPoint joinPoint) {
    log.info("[before] {}", joinPoint.getSignature());
    
    // 이후 알아서 타겟이 호출됨
    // JoinPoint 파라미터 제거해도 됨
  }
  
  
  /**
  	* @After Returning
  	*/
	@AfterReturning(value = "hello.aop.order.aop.Pointcuts.orderAndService()", returning = "result")
  public void doReturn(JoinPoint joinPoint, Object result) {
    log.info("[return] {} : {}", joinPoint.getSignature(), result);
  }
  
  
  /**
  	* @After Throwing
  	*/
	@AfterThrowing(value = "hello.aop.order.aop.Pointcuts.orderAndService()", throwing = "ex")
  public void doThrowing(JoinPoint joinPoint, Exception ex) {
    log.info("[ex] {} : {}", joinPoint.getSignature(), ex);
  }
  
 
  
  /**
  	* @After
  	*/
	@After("hello.aop.order.aop.Pointcuts.orderAndService()")
  public void doAfter(JoinPoint joinPoint) {
    log.info("[after] {}", joinPoint.getSignature());
  }
}
`````



* @Around 어드바이스만 사용해도 사실 필요한 기능은 모두 수행 가능
* 다른 어드바이스들은 JoinPoint를 첫번째 파라미터에 사용할 수 있는데 @Around는 ProceedingJoinPoint를 사용

#### JoinPoint vs ProceedingJoinPoint

* JoinPoint
    * getArgs() : 메서드 인수를 반환
    * getThis() : 프록시 객체를 반환
    * getTarget() : 대상 객체를 반환
    * getSignature() : 조언되는 메서드에 대한 설명을 반환
    * toString() : 조언되는 방법에 대한 유용한 설명을 인쇄
* ProceedingJoinPoint
    * proceed() : 다음 어드바이스나 타겟을 호출







### 포인트컷 지시자

* execution : 메소드 실행 조인 포인트를 매칭, 가장 많이 사용
* within : 특정 타입 내의 조인 포인트를 매칭
* args : 인자가 주어진 타입의 인스턴스인 조인 포인트
* this : 스프링 빈 객체(스프링 AOP 프록시)를 대상으로 하는 조인 포인트
* target : Target 객체(스프링 AOP 프록시가 가르키는 실제 대상)를 대상으로 하는 조인 포인트
* @target : 실행 객체의 클래스에 주어진 타입의 애노테이션이 있는 조인 포인트
* @within : 주어진 어노테이션이 있는 타입 내 조인포인트
* @annotation : 메서드가 주어진 어노테이션을 가지고 있는 조인 포인트를 매칭
* @args : 전달된 실제 인수의 런타임 타입이 주어진 타입의 어노테이션을 갖는 조인 포인트
* bean : 스프링 전용 포인트컷 지시자, 빈의 이름으로 포인트컷 지정



### 포인트컷 지시자 예제

`````java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ClassAop {

}
`````

`````java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MethodAop {
  String value();

}
`````

`````java
public interface MemberService {
	String hello(String param);
}
`````

`````java
@ClassAop
@Component
public class MemberServiceImpl implements MemberService {
	@Override
  @MethodAop("test value")
	public String hello(String param) {
		return "ok";
	}
	
	public String internal(String param) {
		return "ok";
	}
}
`````

`````java
@SpringBootTest
public class ExecutionTest {
	AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
	Method helloMethod;
	
	@BeforeEach
	public void init() throws NoSuchMethodException {
		helloMethod = MemberServiceImpl.class.getMethod("hello", String.class);
	}
	
	@Test
	void printMethod() {
    // public java.lang.String hello.aop.member.MemberServiceImpl.hello(java.lang.String)
		log.info("helloMethod={}", helloMethod);
	}
}
`````



### 포인트컷 지시자 - execution

`````
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)

execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)
`````

* 물음표는 생략 가능
* '*' 같은 패턴 지정 가능



* 하나의 메소드 정확하게 지정

`````java
@SpringBootTest
public class ExecutionTest {
	AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
	Method helloMethod;
	
	@BeforeEach
	public void init() throws NoSuchMethodException {
		helloMethod = MemberServiceImpl.class.getMethod("hello", String.class);
	}
	
	@Test
	void printMethod() {
    // public java.lang.String hello.aop.member.MemberServiceImpl.hello(java.lang.String)
		log.info("helloMethod={}", helloMethod);
	}
	
  /**
  	* 정확하게 매칭되는 경우
  	*/
	@Test
	void exactMatch() {
		// public java.lang.String hello.aop.member.MemberServiceImpl.hello(java.lang.String)
		pointcut.setExpression("execution(public String hellop.aop.member.MemberServiceImpl.hello(String))");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
}
`````

* 접근제어자? :  public
* 반환타입 :  String
* 선언타입? :  hello.aop.member.MemberServiceImpl
* 메서드이름 : hello
* 파라미터 : (String)
* 예외? : 생략



* 모든 조건을 만족하는 경우 지정

`````java
@SpringBootTest
public class ExecutionTest {
	AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
	Method helloMethod;
	
	@BeforeEach
	public void init() throws NoSuchMethodException {
		helloMethod = MemberServiceImpl.class.getMethod("hello", String.class);
	}
	
  /**
  	* 가장 많이 생략한 포인트컷
  	* 모든 조건을 만족하는 경우
  	*/
	@Test
	void allMatch() {
		pointcut.setExpression("execution(* *(..))");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
}
`````

* 접근제어자? : 생략
* 반환타입 :  '*'
* 선언타입? :  생략
* 메서드이름 : '*'
* 파라미터 : (..)
* 예외? : 생략



* 다양한 예제

`````java
@SpringBootTest
public class ExecutionTest {
	AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
	Method helloMethod;
	
	@BeforeEach
	public void init() throws NoSuchMethodException {
		helloMethod = MemberServiceImpl.class.getMethod("hello", String.class);
	}
	
  /**
  	* 메소드명 매칭
  	*/
	@Test
	void nameMatch() {
		pointcut.setExpression("execution(* hello(..))");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
  
  /**
  	* 패키지 매칭
  	*/
	@Test
	void pakageMatch() {
		pointcut.setExpression("execution(* hello.aop.member.MemberServiceImpl.hello(..))");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
  
  /**
  	* 서브패키지 매칭
  	*/
	@Test
	void pakageMatch() {
		pointcut.setExpression("execution(* hello.aop.member..*.*(..))");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
}
`````

* '.' : 정확하게 해당 위치의 패키지
* '..' : 해당 위치의 패키지와 그 하위 패키지도 포함



* 타입 매칭

`````java
@SpringBootTest
public class ExecutionTest {
	AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
	Method helloMethod;
	
	@BeforeEach
	public void init() throws NoSuchMethodException {
		helloMethod = MemberServiceImpl.class.getMethod("hello", String.class);
	}
	
  /**
  	* 타입 매칭 - 부모 타입 허용
  	*/
	@Test
	void typeMatchSuperType() {
    // MemberService : 부모, MemberServiceImpl : 자식일 때 Execution에 부모를 지정해도 매칭 가능
		pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
  
  /**
  	* 타입 매칭 - 부모 타입에 있는 메서드만 허용
  	* MemberService : hello()
  	* MemberServiceImpl : hello(), internal()
  	*/
	@Test
	void typeMatchNoSuperTypeMethodFalse() throws NoSuchMethodException {
		pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");
    
    Method internalMethod = MemberServiceImpl.class.getMethod("internal", String.class);
		Assertions.assertThat(pointcut.matches(internalMethod, MemberServiceImpl.class)).isFalse();
	}
}
`````



* 파라미터 매칭

`````java
@SpringBootTest
public class ExecutionTest {
	AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
	Method helloMethod;
	
	@BeforeEach
	public void init() throws NoSuchMethodException {
		helloMethod = MemberServiceImpl.class.getMethod("hello", String.class);
	}
	
  /**
  	* String 타입인 파라미터 허용
  	*/
	@Test
	void argsMatch() {
		pointcut.setExpression("execution(* *(String))");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
	
	/**
  	* 파라미터가 없을 때 허용
  	*/
	@Test
	void argsMatchNoArgs() {
		pointcut.setExpression("execution(* *())");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();
	}
	
	/**
  	* 정확히 하나의 파라미터 허용, 모든 타입 허용
  	*/
	@Test
	void argsMatchStar() {
		pointcut.setExpression("execution(* *(*))");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
	
	/**
  	* 숫자와 무관하게 모든 파라미터, 모든 타입 허용
  	*/
	@Test
	void argsMatchAll() {
		pointcut.setExpression("execution(* *(..))");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();
	}
	
	/**
  	* String 타입으로 시작하고 숫자와 무관하게 모든 파라미터, 모든 타입 허용
  	*/
	@Test
	void argsMatchComplex() {
		pointcut.setExpression("execution(* *(String, ..))");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();
	}
}
`````





### 포인트컷 - within

* 해당 타입이 매칭되믄 그 안의 메서드들이 자동으로 매칭됨
* execution에서 타입 부분만 사용

`````java
@SpringBootTest
public class WithinTest {
	AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
	Method helloMethod;
	
	@BeforeEach
	public void init() throws NoSuchMethodException {
		helloMethod = MemberServiceImpl.class.getMethod("hello", String.class);
	}
	
	/**
		* 타입 매칭
		*/
	@Test
	void withinExact() {
		pointcut.setExpression("within(hello.aop.member.MemberServiceImpl)");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
	
	@Test
	void withinStar() {
		pointcut.setExpression("within(hello.aop.member.*Service*)");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
  
  /**
		* 타겟의 타입에만 직접 적용, 인터페이스를 선정하면 안됨
		*/
	@Test
	void withinSuperTypeFalse() {
		pointcut.setExpression("within(hello.aop.member.MemberService)");
		Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();
	}
}
`````

* within 타입 사용시 주의 : 부모 타입을 지정하면 안됨, 정확하게 타입이 맞아야 함 (execution과 차이점)



### 포인트컷 - args

* execution의 args
* execution은 파라미터 타입이 정확하게 매칭되어야 함
* args는 부모 타입을 허용

`````java
@SpringBootTest
public class ArgsTest {

	Method helloMethod;
	
	@BeforeEach
	public void init() throws NoSuchMethodException {
		helloMethod = MemberServiceImpl.class.getMethod("hello", String.class);
	}
  
  private AspectJExpressionPointcut pointcut(String expression) {
		AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
    pointcut.setExpression(expression);
    return pointcut;
  }
	
	@Test
	void args() {
		assertThat(pointcut("args(String)")
      .matches(helloMethod, MemberServiceImpl.class)).isTrue();
    assertThat(pointcut("args(Object)")
      .matches(helloMethod, MemberServiceImpl.class)).isTrue();
	}
  
  /**
		* execution(* *(java.io.Serializable)) : 메서드의 시그니처로 판단 (정적)
		* args(java.io.Serializable) : 런타임에 전달된 인수로 판단 (동적)
		*/
	@Test
	void argsVsExecution() {
		assertThat(pointcut("args(String)")
      .matches(helloMethod, MemberServiceImpl.class)).isTrue();
    assertThat(pointcut("args(Object)")
      .matches(helloMethod, MemberServiceImpl.class)).isTrue();
    assertThat(pointcut("args(java.io.Serializable)")
      .matches(helloMethod, MemberServiceImpl.class)).isTrue();
    
    assertThat(pointcut("execution(* *(String))")
      .matches(helloMethod, MemberServiceImpl.class)).isTrue();
    assertThat(pointcut("execution(* *(java.io.Serializable))") // 매칭 실패
      .matches(helloMethod, MemberServiceImpl.class)).isFalse();
    assertThat(pointcut("execution(* *(Object))") // 매칭 실패
      .matches(helloMethod, MemberServiceImpl.class)).isFalse();
	}
}
`````

* args 지시자는 단독으로 사용되기 보다는 뒤에서 설명할 파라미터 바인딩에서 주로 사용





### 포인트컷 지시자 - @target, @within

* @target : 실행 객체의 클래스에 주어진 타입의 어노테이션에 있는 조인 포인트
    * 인스턴스의 모든 메서드를 조인 포인트로 적용
* @within : 주어진 어노테이션이 있는 타입 내 조인 포인트
    * 해당 타입 내에 있는 메서드만 조인 포인트로 적용

![image-20240304093326570](/Users/jyoonk/MUNPIA/STUDY/JAVA&SPRING/image-20240304093326570.png)

`````java
@Import({AtTargetAtWithinTest.Config.class})
@SpringBootTest
public class AtTargetAtWithinTest {

	@Autowired
	Child child;
	
	static class Parent {
    public void parentMethod() {}
  }
  
  @ClassAop
  static class Child extends Parent {
    public void childMethod() {}
  }
  
  @Aspect
  static class AtTargetAtWithinAspect {
    // @target
    @Around("execution(* hello.aop..*(..)) && @target(hello.aop.member.annotation.ClassAop)")
    public Object atTarget(ProceedingJoinPoint joinPoint) throws Throwable {
      log.info("[@target] {}", joinPoint.getSignature());
      return joinPoint.proceed();
    }
    
    // @within
    @Around("execution(* hello.aop..*(..)) && @within(hello.aop.member.annotation.ClassAop)")
    public Object atTarget(ProceedingJoinPoint joinPoint) throws Throwable {
      log.info("[@within] {}", joinPoint.getSignature());
      return joinPoint.proceed();
    }
  }
  
  static class Config {
    @Bean
    public Child child() {
      return new Child();
    }
    @Bean
    public AtTargetAtWithinAspect atTargetAtWithinAspect() {
      return new AtTargetAtWithinAspect();
    }
  }
  
  @Test
  void success() {
    log.info("child Proxy={}", child.getClass());
    child.childMethod();
    child.parentMethod();
  }
}
`````

`````java
// success() 결과
child Proxy = class hello.aop.pointcut.AtTargetAtWithinTest$Child$$EnhancerBySrpingCGLIB
[@target] void hello.aop.pointcut.AtTargetAtWithinTest$Child.childMethod()
[@within] void hello.aop.pointcut.AtTargetAtWithinTest$Child.childMethod()
[@target] void hello.aop.pointcut.AtTargetAtWithinTest$Parent.parentMethod()


// @ClassAop는 Child에 걸려있음
// @target은 Child, Parent 모두 어드바이스 적용
// @within은 Child만 어드바이스 적용
`````

* @target, @within 지시자는 파라미터 바인딩에서 함께 사용됨

* args, @args, @target은 단독으로 사용하면 안되고 execution과 함께 사용





### 포인트컷 지시자 - @annotation, @args

* @annotation

`````java
@ClassAop
@Component
public class MemberServiceImpl implements MemberService {
	@Override
  @MethodAop("test value")
	public String hello(String param) {
		return "ok";
	}
	
	public String internal(String param) {
		return "ok";
	}
}
`````

`````java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MethodAop {
  String value();

}
`````

`````java
@Import(AtAnnotationTest.AtAnnotationAspect.class)
@SpringBootTest
public class AtAnnotationTest {
	@Autowired
	MemberService memberService;
	
	@Test
	void success() {
		log.info("memberService Proxy={}", memberService.getClass());
		memberService.hello("helloA");
	}
	
	@Aspect
	static class AtAnnotationAspect {
		@Around("@annotaion(hello.aop.member.annotation.MethodAop)")
		public Object doAtAnnotation(ProceedingJoinPoint joinPoint) throws Throwable {
			log.info("[@annotation] {}", joinPoint.getSignature());
			return joinPoint.proceed();
		}
	}
}
`````



* @args
    * 전달된 인수의 런 타임 타입에 @Check 어노테이션이 있는 경우에 매칭
    * @args(test.Check)





### 포인트컷 지시자 - bean

`````java
@Import(BeanTest.BeanAspect.class)
@SpringBootTest
public class BeanTest {
	@Autowired
	OrderService orderService;
	
	@Test
	void success() {
	orderService.orderItem("itemA");
	}
	
	@Aspect
	static class BeanAspect {
    @Around("bean(orderService) || bean(*Repository)")
    public Object doLog(ProceedingJoinPoint joinPoint) {
      log.info("[bean] {}", joinPoint.getSignature());
      return joinPoint();
		}
	}
}
`````



### AOP - 매개변수 전달

* 포인트컷 표현식을 사용해 어드바이스에 매개변수를 전달하는 방법
* this, target, args, @target, @within, @annotation, @args

`````java
@Import(PrameterTest.ParameterAspect.class)
@SpringBootTest
public class ParameterTest {
	@Autowired
	MemberService memberService;
	
	@Test
	void success() {
		log.info("memberService Proxy={}", memberService.getClass());
		memberService.hello("helloA");
	}
	
	@Aspect
	static class ParameterAspect {
		@Pointcut("execution(* hello.aop.member..*.*(..))")
		private void allMember() {}
		
		@Around("allMember()")
		public Object logArgs1(ProceedingJoinPoint joinPoint) throws Throwable {
			Object arg1 = joinPoint.getArgs()[0];
			// arg = helloA
			log.info("[logArgs1]{}", joinPoing.getSignature(), arg1);
			return joinPOint.proceed();
		}
    
    @Around("allMember() && args(arg, ..)")
    public Object logArgs2(ProceedingJoinPoint joinPoint, Object arg) throws Throwable {
			// arg = helloA
			log.info("[logArgs2]{}", joinPoing.getSignature(), arg);
			return joinPOint.proceed();
		}
    
    @Before("allMember() && args(arg, ..)")
    public void logArgs3(String arg) {
      // arg = helloA
      log.info("[logArgs3] arg={}", arg);
    }
    
    @Before("allMember() && this(obj)")
    public void thisArgs(JoinPoint joinPoint, MemberService obj) {
      // obj = class hello.aop.member.MemberServiceImpl$$EnhancerBySrpingCGLIB$$a4e...
      log.info("[this] {}, obj={}", joinPoint.getSignature(), obj.getClass());
    }
    
    @Before("allMember() && target(obj)")
    public void targetArgs(JoinPoint joinPoint, MemberService obj) {
      // obj = class hello.aop.member.MemberServiceImpl
      log.info("[target] {}, obj={}", joinPoint.getSignature(), obj.getClass());
    }
    
    @Before("allMember() && @target(annotation)")
    public void atTarget(JoinPoint joinPoint, ClassAop annotation) {
      // obj = @hello.aop.member.annotation.ClassAop()
      log.info("[@target] {}, obj={}", joinPoint.getSignature(), annotation);
    }
    
    @Before("allMember() && @within(annotation)")
    public void atTarget(JoinPoint joinPoint, ClassAop annotation) {
      // obj = @hello.aop.member.annotation.ClassAop()
      log.info("[@within] {}, obj={}", joinPoint.getSignature(), annotation);
    }
    
    @Before("allMember() && @annotation(annotation)")
    public void atAnnotation(JoinPoint joinPoint, MethodAop annotation) {
      // annotation value = test value
      log.info("[@annotation] {}, annotation value={}", joinPoint.getSignature(), annotation.value());
    }
	}
}
`````

* this : 스프링 컨테이너 안에 있는 애
* target : 프록시가 아닌 실제 대상



### this, target

* this, target은 적용 타입 하나를 정확하게 지정해야 함
    * '*' 같은 패턴을 사용할 수 없음
    * 부모 타입을 허용함
* this : 스프링 빈 객체(**스프링 AOP 프록시**)를 대상으로 하는 조인 포인트
* target : Target 객체(**스프링 AOP 프록시가 가르키는 실제 대상**)를 대상으로 하는 조인 포인트



* 프록시 생성 방식에 따른 차이
    * JDK 동적 프록시 : 인터페이스가 필수, 인터페이스를 구현한 프록시 객체를 생성
        * MemberServiceImpl을 가지고 프록시를 생성 못함. MemberService를 가지고 프록시 생성 가능
        * this(MemberService), target(MemberService) 모두 AOP 적용 가능
        * this(MemberServiceImpl), target(MemberServiceImpl) 에서는 this일 경우 AOP 적용 대상이 아님 (target은 AOP 적용 대상임)

![image-20240304231335941](/Users/jyoonk/MUNPIA/STUDY/JAVA&SPRING/image-20240304231335941.png)

* CGLIB : 인터페이스가 있어도 구체 클래스를 상속 받아서 프록시 객체를 생성
    * MemberServiceImpl로 프록시 생성
    * this(MemberService), target(MemberService) 모두 AOP 적용 가능
    * this(MemberServiceImpl), target(MemberServiceImpl) 모두 AOP 적용 가능

![image-20240304231630967](/Users/jyoonk/MUNPIA/STUDY/JAVA&SPRING/image-20240304231630967.png)

* 프록시를 대상으로 하는 this의 경우 프록시 생성 전략에 따라 다른 결과가 나올 수 있다

`````yaml
# application.yml

spring:
	aop:
		proxy-target-class: true // CGLIB
		proxy-target-class: false // JDK 동적 프록시
		
# test에서만 사용한다면
@SpringBootTest(properties = "spring.aop.proxy-target-class=false")
`````



### 로그 출력 AOP 예제

`````java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Trace {

}
`````

`````java
@Aspect
public class TraceAspect {
	@Before("@annotation(hello.aop.exam.annotation.Trace)")
	public void doTrace(JoinPoint joinPoint) {
		Object[] args = joinPoint.getArgs();
		log.info("[trace] {} args = {}", joinPoint.getSignature(), args);
	}
}
`````

`````java
@Repository
public class ExamRepository {
	private static int seq = 0;
	
	/**
		* 5번에 1번 실패하는 요청
		*/
  @Trace
  public String save(String itemId) {
  	seq++;
  	if (seq % 5 == 0) {
  		throw new IllegalStateException("예외 발생");
  	}
  	return "ok";
  }
}
`````

`````java
@Service
@RequiredArgsConstructor
public class ExamService {
	private final ExamRepository examRepository;
	
  @Trace
	public void request(String itemId) {
		examRepository.save(itemId);
	}
}
`````

`````java
@Import(TraceAspect.class)
@SpringBootTest
public class ExamTest {
	@Autowired
	ExamService examService;
	
	@Test
	void test() {
    for (int i = 0; i < 5; i++) {
    examService.request("data" + i);
    }
  }
}
`````





### 재시도 AOP 예제

`````java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Retry {
	int value() default 3;
}
`````

`````java
@Aspect
public class RetryAspect {
	@Around("@annotation(retry)")
	public Object doRetry(ProceedingJoinPoint joinPoint, Retry retry) throws Throwable {
		log.info("[retry] {} retry = {}", joinPoint.getSignature(), retry);
		
		int maxRetry = retry.value();
		Exception exceptionHolder = null;
    
		for (int retryCount = 1; retryCount <= maxRetry; retryCount++) {
      try {
      	log.info("[retry] try count = {}/{}", retryCount, maxRetry);
        return joinPoint.proceed();
      } catch (Exception e) {
				exceptionHolder = e;
      }
		}
		
		throw exceptionHolder;
	}
}
`````

`````java
@Repository
public class ExamRepository {
	private static int seq = 0;
	
	/**
		* 5번에 1번 실패하는 요청
		*/
  @Trace
  @Retry(4)
  public String save(String itemId) {
  	seq++;
  	if (seq % 5 == 0) {
  		throw new IllegalStateException("예외 발생");
  	}
  	return "ok";
  }
}
`````

