# Iterator Pattern (반복자 패턴)
## 정의
- 반복자(Iterator) 패턴은 일련의 데이터 집합에 대하여 순차적인 접근(순회)을 지원하는 패턴이다.
  - 데이터 집합이란 객체들을 그룹으로 묶어 자료의 구조를 취하는 컬렉션을 말한다. 대표적인 컬렉션으로 한번쯤은 들어본 리스트나 트리, 그래프, 테이블 등이 있다.
  - 보통 배열이나 리스트 같은 경우 순서가 연속적인 데이터 집합이기 때문에 간단한 for문을 통해 순회할 수 있다.
- 그러나 해시, 트리와 같은 컬렉션은 데이터 저장 순서가 정해지지 않고 적재되기 때문에, 각 요소들을 어떤 기준으로 접근해야할지 애매해진다.
  - 예를 들어 아래와 같이 트리 구조가 있다면 어떤 상황에선 깊이(세로)를 우선으로 순회 해야 할 수도 있고, 너비(가로)를 우선으로 순회할수도 있기 때문이다.
- 이처럼 복잡하게 얽혀 있는 자료 컬렉션들을 순회하는 알고리즘 전략을 정의하는 것을 __이터레이터 패턴__ 이라고 한다.
- 컬렉션 객체 안에 들어 있는 모든 원소들에 대한 접근 방식이 공통화 되어 있다면 어떤 종류의 컬렉션에서도 이터레이터만 뽑아내면 여러 전략으로 순회가 가능해 보다 다형적인 코드를 설계할 수 있다.
- 이 밖에도 이터레이터 패턴은 별도의 이터레이터 객체를 반환받아 이를 이용해 순회하기 때문에, 집합체의 내부 구조를 노출하지 않고 순회할 수 있다는 장점도 있다.

## 구조
<p align="center"><img src="./images/iterator_adapter_struct.png" width="600"></p>

- 🐳 Aggregate (인터페이스) : ConcreateIterator 객체를 반환하는 인터페이스를 제공한다.
  - 🍀 iterator() : ConcreateIterator 객체를 만드는 팩토리 메서드
- 🐳 ConcreateAggregate (클래스) : 여러 요소들이 이루어져 있는 데이터 집합체
- 🐳 Iterator (인터페이스) : 집합체 내의 요소들을 순서대로 검색하기 위한 인터페이스를 제공한다.
  - 🍀 hasNext() : 순회할 다음 요소가 있는지 확인 (true / false)
  - 🍀 next() : 요소를 반환하고 다음 요소를 반환할 준비를 하기 위해 커서를 이동시킴
- 🐳 ConcreateIterator (클래스) : 반복자 객체
  - 🍀 ConcreateAggregate가 구현한 메서드로부터 생성되며, ConcreateAggregate의 컬렉션을 참조하여 순회한다.
  - 🍀 어떤 전략으로 순회할지에 대한 로직을 구체화한다.

## 흐름
### 클래스 구성
```java
// 집합체 객체 (컬렉션)
interface Aggregate {
    Iterator iterator();
}

class ConcreteAggregate implements Aggregate {
    Object[] arr; // 데이터 집합 (컬렉션)
    int index = 0;

    public ConcreteAggregate(int size) {
        this.arr = new Object[size];
    }

    public void add(Object o) {
        if(index < arr.length) {
            arr[index] = o;
            index++;
        }
    }

    // 내부 컬렉션을 인자로 넣어 이터레이터 구현체를 클라이언트에 반환
    @Override
    public Iterator iterator() {
        return new ConcreteIterator(arr);
    }
}
```
```java
// 반복체 객체
interface Iterator {
    boolean hasNext();
    Object next();
}

class ConcreteIterator implements Iterator {
    Object[] arr;
    private int nextIndex = 0; // 커서 (for문의 i 변수 역할)

    // 생성자로 순회할 컬렉션을 받아 필드에 참조 시킴
    public ConcreteIterator(Object[] arr) {
        this.arr = arr;
    }

    // 순회할 다음 요소가 있는지 true / false
    @Override
    public boolean hasNext() {
        return nextIndex < arr.length;
    }

    // 다음 요소를 반환하고 커서를 증가시켜 다음 요소를 바라보도록 한다.
    @Override
    public Object next() {
        return arr[nextIndex++];
    }
}
```

### 클래스 흐름
```java
public static void main(String[] args) {
    // 1. 집합체 생성
    ConcreteAggregate aggregate = new ConcreteAggregate(5);
    aggregate.add(1);
    aggregate.add(2);
    aggregate.add(3);
    aggregate.add(4);
    aggregate.add(5);

    // 2. 집합체에서 이터레이터 객체 반환
    Iterator iter = aggregate.iterator();

    // 3. 이터레이터 내부 커서를 통해 순회
    while(iter.hasNext()) {
        System.out.printf("%s → ", iter.next());
    }
}
```

## 특징
### 사용 시점
- 컬렉션에 상관없이 객체 접근 순회 방식을 통일하고자 할 때
- 컬렉션을 순회하는 다양한 방법을 지원하고 싶을 때
- 컬렉션의 복잡한 내부 구조를 클라이언트로부터 숨기고 싶은 경우 (편의 + 보안)
- 데이터 저장 컬렉션 종류가 변경 가능성이 있을 때
  - 클라이언트가 집합 객체 내부 표현 방식을 알고 있다면, 표현 방식이 달라지면 클라이언트 코드도 변경되어야 하는 문제가 생긴다.

### 장점
- 일관된 이터레이터 인터페이스를 사용해 여러 형태의 컬렉션에 대해 동일한 순회 방법을 제공한다.
- 컬렉션의 내부 구조 및 순회 방식을 알지 않아도 된다.
- 집합체의 구현과 접근하는 처리 부분을 반복자 객체로 분리해 결합도를 줄일 수 있다.
  - Client에서 iterator로 접근하기 때문에 ConcreteAggregate 내에 수정 사항이 생겨도 iterator에 문제가 없다면 문제가 발생하지 않는다.
- 순회 알고리즘을 별도의 반복자 객체에 추출하여 각 클래스의 책임을 분리하여 단일 책임 원칙(SRP)를 준수한다.
- 데이터 저장 컬렉션 종류가 변경되어도 클라이언트 구현 코드는 손상되지 않아 수정에는 닫혀 있어 개방 폐쇄 원칙(OCP)를 준수한다.

### 단점
- 클래스가 늘어나고 복잡도가 증가한다.
  - 만일 앱이 간단한 컬렉션에서만 작동하는 경우 패턴을 적용하는 것은 복잡도만 증가할 수 있다.
  - 이터레이터 객체를 만드는 것이 유용한 상황인지 판단할 필요가 있다.
- 구현 방법에 따라 캡슐화를 위배할 수 있다.

## 예시
### 이터레이터로 순회 전략 나누기
- 요구사항은 게시글을 최근 글, 작성 순으로 정렬해서 나열할 수 있게 해달라고 한다. 즉, 두 가지 정렬 전략을 구현해야 되는 것이다.

#### ❌ 클린하지 않은 문제의 코드
```java
// 게시글
class Post {
    String title; // 게시글 제목
    LocalDate date; // 게시글 발행일

    public Post(String title, LocalDate date) {
        this.title = title;
        this.date = date;
    }
}

// 게시판
class Board {
    // 게시글을 리스트 집합 객체로 저장 관리
    List<Post> posts = new ArrayList<>();

    public void addPost(String title, LocalDate date) {
        this.posts.add(new Post(title, date));
    }

    public List<Post> getPosts() {
        return posts;
    }
}
```
```java
public static void main(String[] args) {
    // 1. 게시판 생성
    Board board = new Board();

    // 2. 게시판에 게시글을 포스팅
    board.addPost("디자인 패턴 강의 리뷰", LocalDate.of(2020, 8, 30));
    board.addPost("게임 하실분", LocalDate.of(2020, 2, 6));
    board.addPost("이거 어떻게 하나요?", LocalDate.of(2020, 6, 1));
    board.addPost("이거 어떻게 하나요?", LocalDate.of(2021, 12, 22));

    List<Post> posts = board.getPosts();
    
    // 3. 게시글 발행 순서대로 조회하기
    for (int i = 0; i < posts.size(); i++) {
        Post post = posts.get(i);
        System.out.println(post.title + " / " + post.date);
    }

    // 4. 게시글 날짜별로 조회하기
    Collections.sort(posts, (p1, p2) -> p1.date.compareTo(p2.date)); // 집합체를 날짜별로 정렬
    for (int i = 0; i < posts.size(); i++) {
        Post post = posts.get(i);
        System.out.println(post.title + " / " + post.date);
    }
}
```
- 일반적으로 for문을 돌려 집합체의 요소들을 순회하였다. 그러나 이러한 구성 방식은 Board에 들어간 Post를 순회할 때, Board가 어떠한 구조로 이루어져 있는지를 클라이언트에 노출된다.
- 따라서 이를 보다 객체지향적으로 구성하기 위해 이터레이터 패턴을 적용해보자.

#### 이터레이터 패턴을 적용한 코드 ✔️
- 위에서 이터레이터 인터페이스를 직접 만들어 썼지만, 자바에선 이미 이터레이터 인터페이스를 지원한다. 자바의 내부 이터레이터를 재활용해서 메서드 위임을 통해 코드를 간단하게 구현할 수도 있다.
- 순회 전략으로는 리스트 저장 순서대로 조회와 날짜 순서대로 조회 두 가지가 존재한다. 따라서 이에 대한 이터레이터 클래스 역시 두 가지 생성해주면 된다.
```
ListPostIterator : 저장 순서 이터레이터
DatePostIterator : 날짜 순서 이터레이터
```
- 그리고 ListPostIterator와 DatePostIterator 객체를 반환하는 팩토리 메서드를 Board 클래스에 추가만 해주면 완성된다.

```java
// 저장 순서 이터레이터
class ListPostIterator implements Iterator<Post> {
    private Iterator<Post> itr;

    public ListPostIterator(List<Post> posts) {
        this.itr = posts.iterator();
    }

    @Override
    public boolean hasNext() {
        return this.itr.hasNext(); // 자바 내부 이터레이터에 위임해 버림
    }

    @Override
    public Post next() {
        return this.itr.next(); // 자바 내부 이터레이터에 위임해 버림
    }
}

// 날짜 순서 이터레이터
class DatePostIterator implements Iterator<Post> {
    private Iterator<Post> itr;

    public DatePostIterator(List<Post> posts) {
        // 최신 글 목록이 먼저 오도록 정렬
        Collections.sort(posts, (p1, p2) -> p1.date.compareTo(p2.date));
        this.itr = posts.iterator();
    }

    @Override
    public boolean hasNext() {
        return this.itr.hasNext(); // 자바 내부 이터레이터에 위임해 버림
    }

    @Override
    public Post next() {
        return this.itr.next(); // 자바 내부 이터레이터에 위임해 버림
    }
}
```
```java
// 게시판
class Board {
    // 게시글을 리스트 집합 객체로 저장 관리
    List<Post> posts = new ArrayList<>();

    public void addPost(String title, LocalDate date) {
        this.posts.add(new Post(title, date));
    }

    public List<Post> getPosts() {
        return posts;
    }

    // ListPostIterator 이터레이터 객체 반환
    public Iterator<Post> getListPostIterator() {
        return new ListPostIterator(posts);
    }

    // DatePostIterator 이터레이터 객체 반환
    public Iterator<Post> getDatePostIterator() {
        return new DatePostIterator(posts);
    }
}
```
```java
public static void main(String[] args) {
    // 1. 게시판 생성
    Board board = new Board();

    // 2. 게시판에 게시글을 포스팅
    board.addPost("디자인 패턴 강의 리뷰", LocalDate.of(2020, 8, 30));
    board.addPost("게임 하실분", LocalDate.of(2020, 2, 6));
    board.addPost("이거 어떻게 하나요?", LocalDate.of(2020, 6, 1));
    board.addPost("이거 어떻게 하나요?", LocalDate.of(2021, 12, 22));

    // 게시글 발행 순서대로 조회하기
    print(board.getListPostIterator());

    // 게시글 날짜별로 조회하기
    print(board.getDatePostIterator());
}

public static void print(Iterator<Post> iterator) {
    Iterator<Post> itr = iterator;
    while(itr.hasNext()) {
        Post post = itr.next();
        System.out.println(post.title + " / " + post.date);
    }
}
```

- 이제 클라이언트는 게시글을 순회할 때 Board 내부가 어떤 집합체로 구현(Array, List, Tree, Queue 등) 되어 있는지 알 수 없게 감추고 전혀 신경 쓸 필요가 없게 되었다.
- 그리고 순회 전략을 각 객체로 나눔으로써 때에 따라 적절한 이터레이터 객체만 받으면 똑같은 이터레이터 순회 코드로 다양한 순회 전략을 구사할 수 있게 되었다.

#### ✔️ 집합체 구현에 상관 없이 순회를 표현
- Iterator를 사용하는 또다른 이유는 이터레이터를 사용함으로써 집합체 구현과 분리할 수 있기 때문이다.
```java
Iterator<Post> itr = iterator;
while(itr.hasNext()) {
    Post post = itr.next();
    System.out.println(post.title + " / " + post.date);
}
```
- 이터레이터 객체를 반환하면 컬렉션을 순회할때 hasNext()와 next()라는 Iterator의 메소드만을 이용하기 때문에, 집합체인 Board의 내부 구성을 감출수 있게 된다. 즉, 위의 while문은 Board의 구현에 의존하지 않는 것이다.
- 이 말은 만일 추후에 Board의 집합체를 수정하더라도 Board 클래스가 올바른 Iterator만을 반환해 준다면 클라이언트의 코드(위의 while 루프)는 변경하지 않아도 되게 된다.

## Iterator 패턴의 실사용
### Java
- java.util.Enumeration과 java.util.Iterator
- Java StAX (Streaming API for XML)의 Iterator 기반 API
- XmlEventReader, XmlEventWriter
 
#### java.util.Iterator
- hasNext() : 순회할 요소가 있는지 확인
- next() : 현재 커서의 요소를 출력하고 다음으로 커서를 이동
- remove() : Iterator에서 next()로 받았던 해당 엘리먼트를 삭제.
  - 모든 이터레이터에서 다 지원하는 것은 아니다. UnsupportedOperationException()을 던지는 경우가 많다.
  - 보통 이 기능은 동시에 다발적으로 같은 명령을 수행해도 안전한 컬렉션에서 제공한다.
- forEachRemaining() : 함수형 인터페이스를 통해 순회 코드를 심플화 해준다

```java
public static void main(String[] args) {
    Set<Integer> aggregate = new TreeSet<>(List.of(1,2,3,4,5));

    // hasNext(), next(), remove()
    Iterator<Integer> itr = aggregate.iterator();
    while(itr.hasNext()) {
        System.out.printf("%d 삭제", itr.next());
        itr.remove();
    }

    System.out.println(aggregate); // []
}
```
```java
public static void main(String[] args) {
    Set<Integer> aggregate = new TreeSet<>(List.of(1,2,3,4,5));

    // 이터레이터의 while 문 코드를 for문으로 축약할 수 도 있다.
    for (Iterator<Integer> i = aggregate.iterator(); i.hasNext();) {
        System.out.println(i.next());
    }

    // forEachRemaining()
    aggregate.iterator().forEachRemaining(System.out::println);
}
```
