# State Pattern (상태 패턴)

## 정의
- 상태 패턴(State Pattern)은 객체가 특정 상태에 따라 행위를 달리하는 상황에서, 상태를 조건문으로 검사해서 행위를 달리하는 것이 아닌 상태를 __객체화__ 하여 상태가 행동을 할 수 있도록 위임하는 패턴을 말한다.
- 객체 지향 프로그래밍에서의 클래스는 꼭 사물/생물만을 가리키는 데이터만 표현할 수 있는게 아니다. 경우에 따라서 무형태의 행위/동작도 클래스로 묶어 표현할 수 있다.
- 그래서 상태를 클래스로 표현하면 클래스를 교체해서 __상태의 변화__ 를 표현할 수 있고, 객체 내부 상태 변경에 따라 객체의 행동을 상태에 특화된 행동들로 분리해낼 수 있으며, 새로운 행동을 추가하더라도 다른 행동에 영향을 주지 않는다.

- 전략 패턴(Strategy Pattern)이 '전략 알고리즘'을 클래스로 표현한 패턴이라면, 상태 패턴(State Pattern)은 '객체 상태'를 클래스로 표현한 패턴이라고 보면 된다.
  - 그래서 그런지 상태 패턴의 클래스 다이어그램을 보면 전략 패턴과 매우 유사하다는 점을 볼 수 있다.
  - 왜냐하면 전략 패턴은 전략을 객체화한 것이고, 상태 패턴은 상태를 객체화한 것인데 어찌됐던 둘 다 똑같은 클래스 묶음이기 때문이다.

## 구조
<p align="center"><img src="./images/state_pattern_struct.png" width="600"></p>

- 🐳 __State 인터페이스__ : 상태를 추상화한 고수준 모듈.
- 🐳 __ConcreteState__ : 구체적인 각각의 상태를 클래스로 표현. State 역할로 결정되는 인터페이스(API)를 구체적으로 구현한다. 다음 상태가 결정되면 Context에 상태 변경을 요청하는 역할도 한다.
- 🐳 __Context__ : State를 이용하는 시스템. 시스템 상태를 나타내는 State 객체를 합성(composition)하여 가지고 있다. 클라이언트로부터 요청받으면 State 객체에 행위 실행을 위임한다.

## 흐름
### 클래스 구성
```java
interface AbstractState {
    void requestHandle(Context cxt);
}

class ConcreteStateA implements AbstractState {
    @Override
    public void requestHandle(Context cxt) {}
}

class ConcreteStateB implements AbstractState {
    @Override
    public void requestHandle(Context cxt) {
        // 상태에서 동작을 실행한 후 바로 다른 상태로 바꾸기도 함
        // 예를 들어 전원 on 상태에서 끄기 동작을 실행한후 객체 상태를 전원 off로 변경 하듯이
        cxt.setState(ConcreteStateC.getInstance());
    }
}

class ConcreteStateC implements AbstractState {
    @Override
    public void requestHandle(Context cxt) {}
}
```
```java
class Context {
    AbstractState state; // composition

    void setState(AbstractState state) {
        this.state = state;
    }

    // 상태에 의존한 처리 메소드로서 state 객체에 처리를 위임함
    void request() {
        state.requestHandle(this);
    }
}
```

### 클래스 흐름
```java
class Client {
    public static void main(String[] args) {
        Context context = new Context();

        // 1. StateA 상태 설정
        context.setState(new ConcreteStateA());

        // 2. 현재 StateA 상태에 맞는 메소드 실행
        context.request();

        // 3. StateB 상태 설정
        context.setState(new ConcreteStateB());

        // 4. StateB 상태에서 StateC 상태로 변경
        context.request();

        // 5. StateC 상태에 맞는 메소드 실행
        context.request();
    }
}
```

## 특징
### 사용 시점
- 객체의 행동(메서드)가 상태(state)에 따라 각기 다른 동작을 할 때
- 상태 및 전환에 걸쳐 대규모 조건 분기 코드와 중복 코드가 많을 경우
- 조건문의 각 분기를 별도의 클래스에 넣는 것이 상태 패턴의 핵심
- 런타임 중 객체의 상태를 유동적으로 변경해야 할 때

### 장점
- 상태(State)에 따른 동작을 개별 클래스로 옮겨서 관리할 수 있다.
- 상태(State)와 관련된 모든 동작을 각각의 상태 클래스에 분산시킴으로써, 코드 복잡도를 줄일 수 있다.
- 단일 책임 원칙을 준수할 수 있다. (특정 상태와 관련된 코드를 별도의 클래스로 구성)
- 개방 폐쇄 원칙을 준수할 수 있다. (기존 State 클래스나 컨텍스트를 변경하지 않고 새 State를 도입할 수 있음)
- 하나의 상태 객체만 사용하여 상태 변경을 하므로 일관성 없는 상태 주입을 방지하는데 도움이 된다.

### 단점
- 상태별로 클래스를 생성하므로, 관리해야 할 클래스 수 증가
- 상태 클래스 개수가 많고 상태 규칙이 자주 변경된다면, Context의 상태 변경 코드가 복잡해지게 될 수 있다.
- 객체에 적용할 상태가 몇 가지 밖에 없거나 거의 상태 변경이 이루어지지 않는 경우 패턴을 적용하는 것이 과도할 수 있다.

## 예시
### 노트북 전원 상태에 따른 동작 설계
- 노트북에서 전원 버튼을 누르게 되면 나타나는 상태 변화는 다음과 같이 3단계로 이루어 진다.
1. ON 상태에서 전원 버튼을 누르면 노트북이 전원 OFF 상태로 변경
2. OFF 상태에서 전원 버튼을 누르면 노트북이 전원 ON 상태로 변경
3. 절전 모드 상태에서 전원 버튼을 누르면 노트북이 전원 ON 상태로 변경

#### ❌ 클린하지 않은 문제의 코드
```java
class Laptop {
    // 상태를 나타내는 상수
    public static final int OFF = 0;
    public static final int SAVING = 1;
    public static final int ON = 2;

    // 상태를 저장하는 변수
    private int powerState;

    Laptop() {
        this.powerState = Laptop.OFF; // 초기는 노트북이 꺼진 상태
    }

    // 상태 변경
    void changeState(int state) {
        this.powerState = state;
    }

    // 전원 버튼 클릭
    void powerButtonPush() {
        if (powerState == Laptop.OFF) {
            System.out.println("전원 on");
            changeState(Laptop.ON);
        } else if (powerState == Laptop.ON) {
            System.out.println("전원 off");
            changeState(Laptop.OFF);
        } else if (powerState == Laptop.SAVING) {
            System.out.println("전원 on");
            changeState(Laptop.ON);
        }
    }

    void setSavingState() {
        System.out.println("절전 모드");
        changeState(Laptop.SAVING);
    }

    void typebuttonPush() {
        if (powerState == Laptop.OFF) {
            throw new IllegalStateException("노트북이 OFF 인 상태");
        } else if (powerState == Laptop.ON) {
            System.out.println("키 입력");
        } else if (powerState == Laptop.SAVING) {
            throw new IllegalStateException("노트북이 절전 모드인 상태");
        }
    }

    void currentStatePrint() {
        if (powerState == Laptop.OFF) {
            System.out.println("노트북이 전원 ON 상태입니다.");
        } else if (powerState == Laptop.ON) {
            System.out.println("노트북이 전원 ON 상태입니다.");
        } else if (powerState == Laptop.SAVING) {
            System.out.println("노트북이 절전 모드 상태입니다.");
        }
    }
}
```
```java
class Client {
    public static void main(String args[]) {
        LaptopContext laptop = new LaptopContext();
        laptop.currentStatePrint();
        
        // 노트북 켜기 : OffState -> OnState
        laptop.powerButtonPush();
        laptop.currentStatePrint();
        laptop.typebuttonPush();
      
        // 노트북 절전하기 : OnState -> SavingState
        laptop.setSavingState();
        laptop.currentStatePrint();
    
        // 노트북 다시 켜기 : SavingState -> OnState
        laptop.powerButtonPush();
        laptop.currentStatePrint();

        // 노트북 끄기 : OnState -> OffState
        laptop.powerButtonPush();
        laptop.currentStatePrint();
    }
}
```
- 상태 변수는 간단한 솔루션처럼 보이지만 실무에선 어떤 경우에도 좋지 않은 방법이다.
  - 상태 변수는 변수와 행위와의 결합을 만들어 내고, 이 과정에서 조건문들을 부수적으로 생산해내기 때문이다.
  - 따라서 언어적으로 허용되는 한 상태 변수는 최대한 없애주는 것이 좋다. (enum을 사용해도 마찬가지이다. 핵심은 상태의 상수화 자제이다.)
- 위의 코드에 대한 단점을 나열하면 다음과 같다.
  1. 객체 지향적 코드가 아니다. (하드코딩 스타일)
  2. 상태 전환이 복잡한 조건 분기문에 나열되어 있어 가독성이 좋지 않다.
  3. 바뀌는 부분들이 캡슐화되어 있지 않아 노출되어 있다.
  4. 만일 상태 기능을 새로 추가할 경우 메소드를 수정해야 하기 때문에, OCP 원칙에 위배된다.

#### ✔️ 상태 패턴을 적용한 코드
```java
interface PowerState {
    void powerButtonPush(LaptopContext cxt);

    void typebuttonPush();
}

class OnState implements PowerState {
    @Override
    public void powerButtonPush(LaptopContext cxt) {
        System.out.println("노트북 전원 OFF");
        cxt.changeState(new OffState());
    }

    @Override
    public void typebuttonPush() {
        System.out.println("키 입력");
    }

    @Override
    public String toString() {
        return "노트북이 전원 ON 상태입니다.";
    }
}

class OffState implements PowerState {
    @Override
    public void powerButtonPush(LaptopContext cxt) {
        System.out.println("노트북 전원 ON");
        cxt.changeState(new OnState());
    }

    @Override
    public void typebuttonPush() {
        throw new IllegalStateException("노트북이 OFF 인 상태");
    }

    @Override
    public String toString() {
        return "노트북이 전원 OFF 상태입니다.";
    }
}

class SavingState implements PowerState {
    @Override
    public void powerButtonPush(LaptopContext cxt) {
        System.out.println("노트북 전원 on");
        cxt.changeState(new OnState());
    }

    @Override
    public void typebuttonPush() {
        throw new IllegalStateException("노트북이 절전 모드인 상태");
    }

    @Override
    public String toString() {
        return "노트북이 절전 모드 상태입니다.";
    }
}
```
```java
class LaptopContext {
    PowerState powerState;

    LaptopContext() {
        this.powerState = new OffState();
    }

    void changeState(PowerState state) {
        this.powerState = state;
    }

    void setSavingState() {
        System.out.println("노트북 절전 모드");
        changeState(new SavingState());
    }

    void powerButtonPush() {
        powerState.powerButtonPush(this);
    }

    void typebuttonPush() {
        powerState.typebuttonPush();
    }

    void currentStatePrint() {
        System.out.println(powerState.toString());
    }
}
```
```java
class Client {
    public static void main(String[] args) {
        LaptopContext laptop = new LaptopContext();
        laptop.currentStatePrint();
    
        // 노트북 켜기 : OffState -> OnState
        laptop.powerButtonPush();
        laptop.currentStatePrint();
        laptop.typebuttonPush();
 
        // 노트북 절전하기 : OnState -> SavingState
        laptop.setSavingState();
        laptop.currentStatePrint();

        // 노트북 다시 켜기 : SavingState -> OnState
        laptop.powerButtonPush();
        laptop.currentStatePrint();
 
        // 노트북 끄기 : OnState -> OffState
        laptop.powerButtonPush();
        laptop.currentStatePrint();
    }
}
```
- 상태 패턴의 핵심은 '상태'를 객체화하라는 것이다. (객체를 지향하라)
- 노트북의 상태 3가지를 모두 클래스로 구성한다. 그리고 인터페이스나 추상클래스로 묶어 추상화 / 캡슐화(정보 은닉)를 한다. 상태를 클래스로 분리하였으니, 상태에 따른 행동 메소드도 각 상태 클래스마다 구현을 해준다.
- 결과적으로 코드의 전체 라인수가 길어지고 클래스도 많아져서 읽기 힘들 것 같지만, 오히려 해당 방식이 추후에 유지보수를 용이하게 해준다.

#### 👨‍🔧 리팩토링 (싱글톤 적용)
- 하지만 위의 코드에는 문제가 있다. 상태를 변경할 때마다 새로 객체를 생성한다는 점이다.
- 물론 연결이 끊긴 상태 객체는 JVM의 가비지 컬렉션(GC)에 의해 자동으로 지워지겠지만, 이런 가비지 값이 늘어나게 되면 나중에 객체 제거 과정에서 Stop-the-world가 일어나게 된다.
- 웬만한 상황에선 상태는 새로 인스턴스화할 필요가 전혀 없다. 괜히 메모리 낭비인 셈이다. 따라서 각 상태 클래스들을 싱글톤(Singleton)화 시킨다.

```java
class OnState implements PowerState {

	// Thread-Safe 한 싱글톤 객체화
    private OnState() {}
    private static class SingleInstanceHolder {
        private static final OnState INSTANCE = new OnState();
    }
    public static OnState getInstance() {
        return SingleInstanceHolder.INSTANCE;
    }

    @Override
    public void powerButtonPush(LaptopContext cxt) {
        System.out.println("노트북 전원 OFF");
        cxt.changeState(OffState.getInstance()); // 싱글톤 객체 얻기
    }

    @Override
    public void typebuttonPush() {
        System.out.println("키 입력");
    }

    @Override
    public String toString() {
        return "노트북이 전원 ON 상태입니다.";
    }
}

// ...
```

## State vs Strategy

### 유사점
- 전략 패턴과 상태 패턴은 클래스 다이어그램이 거의 동일하고 코드 사용법도 비슷하다.
- 둘 다 난잡한 조건 분기를 극복하기 위해 전략 / 상태 형태를 객체화
- 둘 다 합성(composition)을 통해 상속의 한계를 극복
- 둘 다 객체의 일련의 행동이 캡슐화되어 객체 지향 원칙을 준수한다.
- State는 Strategy의 확장으로 간주될 수도 있다.

### 차이점
- 전략 패턴과 상태 패턴의 구조는 거의 같지만 어떤 목적을 위해서 사용되는가에 따라 차이가 있다.
- 전략 패턴은 알고리즘을 객체화하여 클라이언트에서 유연적으로 전략을 제공/교체를 한다. 상태 패턴은 객체의 상태를 객체화하여 클라이언트와 상태 클래스 내부에서 다른 상태로 교체를 한다.
- 전략 패턴의 전략 객체는 그 전략만의 알고리즘 동작을 정의 및 수행한다. (만일 전략을 상태화하면 클래스 폭발이 일어날 수 있다) 상태 패턴의 상태 객체는 상태가 적용되는 대상 객체가 할 수 있는 일련의 모든 행동들을 정의 및 수행한다.
- 전략 패턴의 전략 객체는 입력값에 따라 전략 형태가 다양하게 될 수 있으니 인스턴스로 구성한다. 상태 패턴의 상태 객체는 정의된 상태를 서로 스위칭하기에 메모리 절약을 위해 싱글톤으로 구성한다.


