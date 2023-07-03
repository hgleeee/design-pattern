# Proxy Pattern (프록시 패턴)
## 정의
- 프록시 패턴(Proxy Pattern)은 대상 원본 객체를 대리하여 대신 처리하게 함으로써 로직의 흐름을 제어하는 행동 패턴이다.
- 프록시(Proxy)의 사전적인 의미는 '대리인'이라는 뜻이다. 즉, 누군가에게 어떤 일을 대신 시키는 것을 의미하는데, 이를 객체 지향 프로그래밍에 접목해보면 클라이언트가 대상 객체를 직접 쓰는게 아니라 중간에 프록시(대리인)을 거쳐서 쓰는 코드 패턴이라고 보면 된다.
- 따라서 대상 객체(Subject)의 메소드를 직접 실행하는 것이 아닌, 대상 객체에 접근하기 전에 프록시(Proxy) 객체의 메서드를 접근한 후 추가적인 로직을 처리한 뒤 접근하게 된다.

## 사용 이유
- 그냥 객체를 이용하면 되지, 왜 이렇게 번거롭게 중계 대리자를 통해 이용하는 방식을 취할까?
- 대상 클래스가 민감한 정보를 가지고 있거나 인스턴스화 하기에 무겁거나, 추가 기능을 가미하고 싶은데 원본 객체를 수정할 수 없는 상황일 때를 극복하기 위해서이다.
- 대체적으로 정리하자면 다음과 같은 효과를 누릴 수 있다고 볼 수 있다.

1. 보안(Security) : 프록시는 클라이언트가 작업을 수행할 수 있는 권한이 있는지 확인하고 검사 결과가 긍정적인 경우에만 요청을 대상으로 전달한다.
2. 캐싱(Caching) : 프록시가 내부 캐시를 유지하여 데이터가 캐시에 아직 존재하지 않는 경우에만 대상에서 작업이 실행되도록 한다.
3. 데이터 유효성 검사(Data validation) : 프록시가 입력을 대상으로 전달하기 전에 유효성을 검사한다.
4. 지연 초기화(Lazy initialization) : 대상의 생성 비용이 비싸다면 프록시는 그것을 필요로 할 때까지 연기할 수 있다.
5. 로깅(Logging) : 프록시는 메소드 호출과 상대 매개 변수를 인터셉트하고 이를 기록한다.
6. 원격 객체(Remote objects) : 프록시는 원격 위치에 있는 객체를 가져와서 로컬처럼 보이게 할 수 있다.

## 구조
- 프록시는 다른 객체에 대한 접근을 제어하는 개체이다. 여기서 다른 객체를 대상(Subject)라고 부른다.
- 프록시와 대상은 동일한 인터페이스를 가지고 있으며 이를 통해 다른 인터페이스와 완전히 호환되도록 바꿀수 있다.

<p align="center"><img src="./images/proxy_pattern_struct.png" width="600"></p>

- 🐳 __Subject__ : Proxy와 RealSubject를 하나로 묶는 인터페이스 (다형성)
  - 🍀 대상 객체와 프록시 역할을 동일하게 하는 추상 메소드 operation() 를 정의한다.
  - 🍀 인터페이스가 있기 때문에 클라이언트는 Proxy 역할과 RealSubject 역할의 차이를 의식할 필요가 없다.
- 🐳 __RealSubject__ : 원본 대상 객체
- 🐳 __Proxy__ : 대상 객체(RealSubject)를 중계할 대리자 역할
  - 🍀 프록시는 대상 객체를 합성(composition)한다.
  - 🍀 프록시는 대상 객체와 같은 이름의 메서드를 호출하며, 별도의 로직을 수행 할수 있다 (인터페이스 구현 메소드)
  - 🍀 프록시는 흐름제어만 할 뿐 결과값을 조작하거나 변경시키면 안 된다.
- 🐳 __Client__ : Subject 인터페이스를 이용하여 프록시 객체를 생성해 이용.
  - 🍀 클라이언트는 프록시를 중간에 두고 프록시를 통해서 RealSubject와 데이터를 주고 받는다.


## 종류
- Proxy 패턴은 단순하면서도 자주 쓰이는 패턴이며, 그 활용 방식도 다양하다. 같은 프록시 객체라도 어떠한 로직을 짜느냐에 따라 그 활용도는 천차만별이 된다.
- Proxy 패턴의 기본형을 어떤 방식으로 변형하느냐에 따라 프록시 종류가 나뉘어지게 된다.

### 기본형 프록시 (Normal Proxy)
```java
interface ISubject {
    void action();
}

class RealSubject implements ISubject {
    public void action() {
        System.out.println("원본 객체 액션 !!");
    }
}
```
```java
class Proxy implements ISubject {
    private RealSubject subject; // 대상 객체를 composition

    Proxy(RealSubject subject) {
        this.subject = subject;
    }

    public void action() {
        subject.action(); // 위임
        /* do something */
        System.out.println("프록시 객체 액션 !!");
    }
}

class Client {
    public static void main(String[] args) {
        ISubject sub = new Proxy(new RealSubject());
        sub.action();
    }
}
```

### 가상 프록시 (Virtual Proxy)
- 지연 초기화 방식
- 가끔 필요하지만 항상 메모리에 적재되어 있는 무거운 서비스 객체가 있는 경우
- 이 구현은 실제 객체의 생성에 많은 자원이 소모되지만 사용 빈도는 낮을 때 쓰는 방식이다.
- 서비스가 시작될 때 객체를 생성하는 대신에 객체 초기화가 실제로 필요한 시점에 초기화될 수 있도록 지연시킬 수 있다.

```java
class Proxy implements ISubject {
    private RealSubject subject; // 대상 객체를 composition

    Proxy() {
    }

    public void action() {
    	// 프록시 객체는 실제 요청(action(메소드 호출)이 들어 왔을 때 실제 객체를 생성한다.
        if(subject == null){
            subject = new RealSubject();
        }
        subject.action(); // 위임
        /* do something */
        System.out.println("프록시 객체 액션 !!");
    }
}

class Client {
    public static void main(String[] args) {
        ISubject sub = new Proxy();
        sub.action();
    }
}
```

### 보호 프록시 (Protection Proxy)
- 프록시가 대상 객체에 대한 자원으로의 엑세스 제어(접근 권한)
- 특정 클라이언트만 서비스 객체를 사용할 수 있도록 하는 경우
- 프록시 객체를 통해 클라이언트의 자격 증명이 기준과 일치하는 경우에만 서비스 객체에 요청을 전달할 수 있게 한다.

```java
class Proxy implements ISubject {
    private RealSubject subject; // 대상 객체를 composition
    boolean access; // 접근 권한

    Proxy(RealSubject subject, boolean access) {
        this.subject = subject;
        this.access = access;
    }

    public void action() {
        if(access) {
            subject.action(); // 위임
            /* do something */
            System.out.println("프록시 객체 액션 !!");
        }
    }
}

class Client {
    public static void main(String[] args) {
        ISubject sub = new Proxy(new RealSubject(), false);
        sub.action();
    }
}
```

### 로깅 프록시 (Logging Proxy)
- 대상 객체에 대한 로깅을 추가하려는 경우
- 프록시는 서비스 메서드를 실행하기 전달하기 전에 로깅하는 기능을 추가하여 재정의한다.

```java
class Proxy implements ISubject {
    private RealSubject subject; // 대상 객체를 composition

    Proxy(RealSubject subject) {
        this.subject = subject;
    }

    public void action() {
        System.out.println("로깅..................");
        
        subject.action(); // 위임
        /* do something */
        System.out.println("프록시 객체 액션 !!");

        System.out.println("로깅..................");
    }
}

class Client {
    public static void main(String[] args) {
        ISubject sub = new Proxy(new RealSubject());
        sub.action();
    }
}
```

### 원격 프록시 (Remote Proxy)
- 프록시 클래스는 로컬에 있고, 대상 객체는 원격 서버에 존재하는 경우
- 프록시 객체는 네트워크를 통해 클라이언트의 요청을 전달하여 네트워크와 관련된 불필요한 작업들을 처리하고 결과값만 반환
- 클라이언트 입장에선 프록시를 통해 객체를 이용하는 것이니 원격이든 로컬이든 신경 쓸 필요가 없으며, 프록시는 진짜 객체와 통신을 대리하게 된다.

### 캐싱 프록시 (Caching Proxy)
- 데이터가 큰 경우 캐싱하여 재사용을 유도
- 클라이언트 요청의 결과를 캐시하고 이 캐시의 수명 주기를 관리


## 특징
### 사용 시점
- 접근을 제어하거가 기능을 추가하고 싶은데, 기존의 특정 객체를 수정할 수 없는 상황일 때
- 초기화 지연, 접근 제어, 로깅, 캐싱 등 기존 객체 동작에 수정 없이 가미하고 싶을 때
### 장점
- 개방 폐쇄 원칙(OCP) 준수
  - 기존 대상 객체의 코드를 변경하지 않고 새로운 기능을 추가할 수 있다.
- 단일 책임 원칙(SRP) 준수 
  - 대상 객체는 자신의 기능에만 집중 하고, 그 이외 부가 기능을 제공하는 역할을 프록시 객체에 위임하여 다중 책임을 회피할 수 있다.
- 원래 하려던 기능을 수행하며 그 외 부가적인 작업(로깅, 인증, 네트워크 통신 등)을 수행하는데 유용하다.
- 클라이언트는 객체를 신경쓰지 않고, 서비스 객체를 제어하거나 생명 주기를 관리할 수 있다.
- 사용자 입장에서는 프록시 객체나 실제 객체나 사용법은 유사하므로 사용에 문제되지 않는다.
### 단점
- 많은 프록시 클래스를 도입해야 하므로 코드의 복잡도가 증가한다.
  - 예를 들어 여러 클래스에 로깅 기능을 가미시키고 싶다면, 동일한 코드를 적용함에도 각각의 클래스에 해당되는 프록시 클래스를 만들어서 적용해야 하기 때문에 코드량이 많아지고 중복이 발생한다.
  - 자바에서는 리플렉션에서 제공하는 동적 프록시(Dynamic Proxy) 기법을 이용해서 해결할 수 있다.
- 프록시 클래스 자체에 들어가는 자원이 많다면 서비스로부터의 응답이 늦어질 수 있다.

## 예시
### 보호 프록시 패턴 구현하기
- 어느 회사에선 직원들이 각 회사 구성원들의 정보들을 아무 제약 없이 모두 열람할 수 있다고 한다. 그래서 인사팀에서 보안을 위해 인사 정보에 대한 데이터 접근을 직책 단위로 세분화하고자 한다.
- 예를 들어 사원은 오로지 자신 직책과 같은 사원들 정보만 열람할 수 있으며, 그 위의 과장이나 상무 정보는 열람할 수 없는 식이다.
- 그래서 기존의 프로그램의 로직을 업데이트할 필요가 있는데, 그런데 기존의 프로그램을 수정하기에는 너무나 방대하고 복잡해서 난관에 부딪혔다고 한다. 이를 어떻게 해결할까?

```java
// 직책 상수
enum RESPONSIBILITY {
    STAFF, // 사원
    MANAGER, // 과장
    DIRECTOR // 상무
}
``` 
- 먼저 회사의 직책을 표현하는 RESPONSIBILITY enum 상수 클래스를 정의하였다. 차례대로 사원(Staff), 과장(Manager), 상무(Director)를 표현하는 상수를 가지고 있다.

```java
// 구성원 클래스
class Employee {
    private String name; // 이름
    private RESPONSIBILITY position; // 직위

    public Employee(String name, RESPONSIBILITY position) {
        this.name = name;
        this.position = position;
    }

    public String getName() {
        return name;
    }

    public RESPONSIBILITY getGrade() {
        return position;
    }

    public String getInfo(Employee viewer) {
        return "Display " + getGrade().name() + " '" + getName() + "' personnel information.";
    }
}
```
- 그리고 직원의 정보를 클래스화한 Employee 클래스를 정의하였다. 이 클래스에는 구성원의 이름과 직책의 정보를 한번에 알 수 있는 getInfo() 메서드를 지원한다.

```java
class PrintEmployeeInfo {
    Employee viewer; // 조회하려는 자
    
    PrintEmployeeInfo(Employee viewer) {
        this.viewer = viewer;
    }
	
    // Employee 객체 리스트를 받아 직원들의 정보를 순회하여 조회
    void printAllInfo(List<Employee> employees) {
        employees.stream()
                .map(employee -> employee.getInfo(viewer))
                .forEach(System.out::println);
    }
}
```
- 마지막으로 회사의 전 구성원을 모두 출력하는 PrintEmployeeInfo 클래스 프로그램이 있다.
- 이 클래스는 생성자의 인자로 누가 조회하는지 대상자(viewer)를 받고 모든 구성원 리스트를 인자로 받아 출력해주는 printAllInfo() 메서드를 지원한다.

#### ❌ 클린하지 않은 문제의 코드
- 지금 이 상태로는 어느 누구든지 PrintEmployeeInfo 객체를 통해 printAllInfo() 메서드를 실행시켜 모든 직원의 리스트를 볼 수 있게 된다.

```java
public static void main(String[] args) {
    // 직원별 개인 객체 생성
    Employee CTO = new Employee("Dragon Jung", RESPONSIBILITY.DIRECTOR);
    Employee devManager = new Employee("Cats Chang", RESPONSIBILITY.MANAGER);
    Employee financeManager = new Employee("Dell Choi", RESPONSIBILITY.MANAGER);
    Employee devStaff = new Employee("Dark Kim", RESPONSIBILITY.STAFF);
    Employee financeStaff = new Employee("Pal Yoo", RESPONSIBILITY.STAFF);

    // 직원들을 리스트로 가공
    List<Employee> employees = Arrays.asList(CTO, devManager, financeManager, devStaff, financeStaff);

    /*-----------------------------------------------------------------------------------------*/

    // 나 : 일개 사원 직책
    Employee me = new Employee("inpa", RESPONSIBILITY.STAFF);

    System.out.println("\n================================================================");
    System.out.println("시나리오1. 일개 사원인 내가 회사 인원 인사 정보 조회");
    System.out.println("================================================================");
    PrintEmployeeInfo view = new PrintEmployeeInfo(me); // 모든 직원 정보를 출력하는 클래스
    view.printAllInfo(employees); // 일개 사원에 불구하고 모든 직원 조회
}
```
- 따라서 직위에 따라 정보 열람 접근 제한을 두어야 하는데, 기존의 프로그램을 수정하기엔 비용이 많이 든다. 이럴 때, 프록시 객체를 통해 기존의 프로그램의 일부 기능을 제어하도록 하면 된다.

#### ✔️ 프록시 패턴을 적용한 코드
- 보호 프록시는 프록시 객체가 사용자의 실제 객체에 대한 접근을 제어한다. 여기선 직책에 따른 정보 열람 접근 제어이다.
- 프록시를 구성하기에 앞서, 우선 대상 객체와 프록시 객체를 모두 묶어주는 인터페이스를 선언해준다.

```java
// 구성원 인터페이스
interface IEmployee {
    String getName(); // 구성원의 이름
    RESPONSIBILITY getGrade(); // 구성원의 직책
    String getInfo(IEmployee viewer); // 구성원의 인사정보
}
```
- 그리고 이 때까지 Employee 클래스 타입으로 받은 모든 변수와 매개변수의 타입을 인터페이스로 재설정해준다.
- 이제 본격적인 프록시 클래스를 설정할 차례이다. 그동안 모든 구성원을 출력해주던 대상 객체의 getInfo() 메서드를 제어 로직을 추가하고 대상 객체의 메서드를 위임 호출해줌으로써 보호 프록시를 구성할 수 있게 된다.

```java
// 보호 프록시 : 인사정보가 보호된 구성원 (인사 정보 열람 권한 없으면 예외 발생)
class ProtectedEmployee implements IEmployee {
    private IEmployee employee;

    public ProtectedEmployee(IEmployee employee) {
        this.employee = employee;
    }

    @Override
    public String getInfo(IEmployee viewer) {

        RESPONSIBILITY position = this.employee.getGrade(); // 조회당하는 자의 직책
		
        // 매개변수로 받은 조회자의 직책에 따라 출력을 제어
        switch (viewer.getGrade()) {
            case DIRECTOR:
                // 부사장은 과장, 사원들을 볼 수 있다.
                return this.employee.getInfo(viewer);
            case MANAGER:
                // 과장은 같은 직무와 그 아래 사원들을 볼 수 있다. 사장 정보는 볼 수 없다.
                if (position != RESPONSIBILITY.DIRECTOR) {
                    return this.employee.getInfo(viewer);
                }
            case STAFF:
                // 사원은 같은 직무인 사람들만 볼 수 있다. 과장, 사장 정보는 볼 수 없다.
                if (position != RESPONSIBILITY.DIRECTOR && position != RESPONSIBILITY.MANAGER) {
                    return this.employee.getInfo(viewer);
                }
            default: return "다른 사람의 인사정보를 조회할 수 없습니다";
        }
    }

    @Override
    public String getName() {
        return employee.getName();
    }

    @Override
    public RESPONSIBILITY getGrade() {
        return employee.getGrade();
    }
}
```
```java
class Client {
    public static void main(String[] args) {
        // 직원별 개인 객체 생성
        Employee CTO = new Employee("Dragon Jung", RESPONSIBILITY.DIRECTOR);
        Employee devManager = new Employee("Cats Chang", RESPONSIBILITY.MANAGER);
        Employee financeManager = new Employee("Dell Choi", RESPONSIBILITY.MANAGER);
        Employee devStaff = new Employee("Dark Kim", RESPONSIBILITY.STAFF);
        Employee financeStaff = new Employee("Pal Yoo", RESPONSIBILITY.STAFF);

        // 직원들을 리스트로 가공
        List<Employee> employees = Arrays.asList(CTO, devManager, financeManager, devStaff, financeStaff);

        /*-----------------------------------------------------------------------------------------*/

        // 기존에 등록된 리스트를 수정할 수 없으니, 동적으로 기존의 Employee 객체를 프록시 객체 ProtectedEmployee로 Wrap하는 작업을 실행한다.
        List<IEmployee> protectedEmployees = new ArrayList<>();
        for (Employee e : employees) {
            protectedEmployees.add(new ProtectedEmployee((IEmployee) e));
        }

        /*-----------------------------------------------------------------------------------------*/

        // 나 : 일개 사원 직책
        Employee me = new Employee("inpa", RESPONSIBILITY.STAFF);

        System.out.println("\n================================================================");
        System.out.println("시나리오1. 일개 사원인 내가 회사 인원 인사 정보 조회");
        System.out.println("================================================================");
        PrintEmployeeInfo view = new PrintEmployeeInfo(me); 
        view.printAllInfo(protectedEmployees); 

        System.out.println("\n================================================================");
        System.out.println("시나리오2. 과장이 회사 인원 인사 정보 조회");
        System.out.println("================================================================");
        PrintEmployeeInfo view2 = new PrintEmployeeInfo(devManager);
        view2.printAllInfo(protectedEmployees); 

        System.out.println("\n================================================================");
        System.out.println("시나리오3. 상무가 회사 인원 인사 정보 조회");
        System.out.println("================================================================");
        PrintEmployeeInfo view3 = new PrintEmployeeInfo(CTO);
        view3.printAllInfo(protectedEmployees);
    }
}
```


## Proxy 패턴의 실사용
### Java
- java.lang.reflect.Proxy
- java.rmi.* (원격 프록시 모듈)
- javax.ejb.EJB 
- javax.inject.Inject
- javax.persistence.PersistenceContext

#### Dynamic Proxy
- 자바 JDK에서는 별도로 프록시 객체 구현 기능을 지원하는데, 이를 동적 프록시(Dynamic Proxy) 기법이라고 불리운다.
- 동적 프록시는 개발자가 직접 일일히 프록시 객체를 생성하는 것이 아닌, 애플리케이션 실행 도중 java.lang.reflect.Proxy 패키지에서 제공해주는 API를 이용하여 동적으로 프록시 인스턴스를 만들어 등록하는 방법
- 자바의 Reflection API 기법을 응용한 연장선의 개념이다. 그래서 별도의 프록시 클래스 정의없이 런타임으로 프록시 객체를 동적으로 생성해 이용할 수 있다는 장점이 있다.
```java
// 대상 객체와 프록시를 묶는 인터페이스
interface Animal {
    void eat();
}

// 프록시를 적용할 타겟 객체
class Tiger implements Animal{
    @Override
    public void eat() {
        System.out.println("호랑이가 음식을 먹습니다.");
    }
}
```
```java
public class Client {
    public static void main(String[] arguments) {
		
        // newProxyInstance() 메서드로 동적으로 프록시 객체를 생성할 수 있다.
        Animal tigerProxy = (Animal) Proxy.newProxyInstance(
                Animal.class.getClassLoader(), // 대상 객체의 인터페이스의 클래스로더
                new Class[]{Animal.class}, // 대상 객체의 인터페이스
                new InvocationHandler() { // 프록시 핸들러
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        Object target = new Tiger();

                        System.out.println("----eat 메서드 호출 전----");

                        Object result = method.invoke(target, args); // 타겟 메서드 호출

                        System.out.println("----eat 메서드 호출 후----");

                        return result;
                    }
                }
        );

        tigerProxy.eat();
    }
}
```

### Spring
#### Spring AOP
- 스프링 프레임워크에서는 내부적으로 프록시 기술을 정말 많이 사용하고 있다. (AOP, JPA 등)
  - 스프링에서는 Bean을 등록할 때 Singleton을 유지하기 위해 Dynamic Proxy 기법을 이용해 프록시 객체를 Bean으로 등록한다.
  - 또한 Bean으로 등록하려는 기본적으로 객체가 Interface를 하나라도 구현하고 있으면 JDK를 이용하고 Interface를 구현하고 있지 않으면 내장된 CGLIB 라이브러리를 이용한다.

```java
@Service
public class GameService {
	public void startDame() {
    	System.out.println("이 자리에 오신 여러분을 진심으로 환영합니다.");
    }
}
```
```java
@Aspect
@Comonent
public class PerfAspect {
	@Around("bean(gameService)")
	public void timestamp(ProceedingJoinPoint point) throws Throwable {
    	System.out.println("프록시 실행 1");
        
        point.proceed(); // 대상 객체의 원본 메서드를 실행
        
        System.out.println("프록시 실행 2");
    }
}
```











