---
title:  "Java 제네릭"
category: posts
tags:
  - 이펙티브 자바
  - java
  - generic
comments: true
last_modified_at: 2019-12-13T12:30:00
---

> 제네릭(generic)은 자바 5부터 사용할 수 있다. 제네릭을 사용하면 컬렉션이 담을 수 있는 타입을 컴파일러에 알려주게 된다.

## 비검사 경고를 제거하라

```java
Set<Lark> exaltation = new HashSet(); // 컴파일러가 무엇이 잘못됐는지 친절히 설명해준다.
Set<Lark> exaltation = new HashSet<Lark>(); // 컴파일러가 알려준 대로 수정하면 경고가 사라진다.
Set<Lark> exaltation = new HashSet<>(); // 자바 7부터 지원하는 다이아몬드 연산자(<>)만으로 해결할 수 있다.
```

> **할 수 있는 한 모든 비검사 경고를 제거하라**. 모두 제거하면 그 코드는 타입 안정성이 보장된다. 즉, 런타임에 ClassCastException이 발생할 일이 없다.

- 경고를 제거할 수는 없지만 타입이 안전하다고 확신한다면 `@SuppressWarnings("unchecked")` 애너테이션을 달아 경고를 숨기자.

  - 해당 코드는 경고 없이 컴파일되겠지만, 런타임에는 여전히 ClassCastException을 던질 수 있다.
  - 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다. 하지만 항상 가능한 한 좁은 범위에 적용하자.

  ```java
  // ArrayList에서 가져온 toArray 메스드 예
  public <T> T[] toArray(T[] a) {
    if (a.length < size) {
      return (T[]) Arrays.copyOf(elements, size, a.getClass()); // 컴파일 경고
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) {
      a[size] = null;
    }
    return a;
  }
  ```

  ```java
  public <T> T[] toArray(T[] a) {
    if (a.length < size) {
      // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
      @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass()); // 지역변수를 추가해 @SuppressWarnings의 범위를 좁힌다.
      return result; // 애너테이션은 선언에만 달 수 있기 때문에 return문에 직접 다는 게 불가능하다.
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) {
      a[size] = null;
    }
    return a;
  }
  ```

  - @SuppressWarnings("unchecked") 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.

## 배열보다는 리스트를 사용하라

- 배열과 제네릭 타입의 차이

  1. 배열은 공변(convariant, 즉 함께 변한다는 뜻)이다. 반면, 제네릭은 불공변(invariant)이다.

     - 배열 Sub[]는 배열 Super[]의 하위 타입이 된다.

       ```java
       // 코드 28-1 런타임에 실패한다.
       Object[] objectArray = new Long[1];
       objectArray[0] = "타입이 달라 넣을 수 없다." // ArrayStoreException을 던진다.
       ```

     - `List<Type1>` 은 `List<Type2>`의 하위 타입도 아니고 상위 타입도 아니다.

       ```java
       // 코드 28-2 컴파일되지 않는다.
       List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다.
       ol.add("타입이 달라 넣을 수 없다.")
       ```

  2. 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 반면, 제네릭은 원소 타입을 컴파일타임에만 검사하며 타입 정보가 런타임에는 소거된다.

- 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.

  - 타입 안전하지 않기 때문에 제네릭 배열을 만들지 못하게 막혀있다.

  ```java
  // 코드 28-3 제네릭 배열 생성을 허용하지 않는 이유 - L:2에서 컴파일 오류가 발생한다.
  List<String>[] stringLists = new List<String>[1]; // 만약 제네릭 배열 생성이 허용된다면,
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList; // stringLists 배열에 List<Integer> 인스턴스가 저장된다.
  String s = stringLists[0].get(0); // 여기서 런타임 ClassCastException이 발생한다.
  ```

- 실체화 불가 타입(non-reifiable type) : 실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입

  - `E`, `List<E>`, `List<String>` 같은 타입
  - 소거 메커니즘 때문에 매개변수화 타입 가운데 실체화 될수 있는 타입은 `List<?>`와 `Map<?,?>` 같은 비한정적 와일드카드 타입뿐이다.
    - 배열을 만들 수는 있지만, 유용하게 쓰일 일은 거의 없다.

- 배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열은 `E[]` 대신 컬렉션인 `List<E>`를 사용하면 해결된다.

- 비검사 형변환 경고를 제거하려면 배열 대신 리스트를 쓰면 된다. 컴파일 오류나 경고를 만나면, 배열을 리스트로 대체하는 방법을 적용하자.

  - 코드양이 늘거나 느려질 수 있지만, 런타임에 ClassCastException으로부터 안전하다.

## 이왕이면 제네릭 타입으로 만들라

```java
// 코드 29-1 Object 기반 스택 - 제네릭이 절실한 강력 후보!
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }
  
  public Object pop() {
    if (size == 0) {
      throw new EmptyStackException();
    }
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
  }
  
  public boolean isEmpty() {
    return size == 0;
  }
  
  private void ensureCapacity() {
    if (elements.length == size) {
      elements == Arrays.copyOf(elements, 2 * size + 1);
    }
  }
}
```

- 코드 29-1의 클래스는 원래 제네릭 타입이어야 마땅하다. - 런타임 오류가 날 위험이 있다.

```java
public class Stack<E> {
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  // 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
  // 따라서 타입 안전성을 보장하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[]다.
  @SuppressWarnings("unchecked")
  public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY]; // 실체화 불가 타입 E로는 (new E[DEFAULT_INITIAL_CAPACITY])배열을 만들 수 없다.
  }
  
  public void push(E e) {
    ensureCapacity();
    elements[size++] = e;
  }
  
  public E pop() {
    if (size == 0) {
      throw new EmptyStackException();
    }
    // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
    @SuppressWarnings("unchecked")
    E result = (E) elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
  }
  
  public boolean isEmpty() {
    return size == 0;
  }
  
  private void ensureCapacity() {
    if (elements.length == size) {
      elements == Arrays.copyOf(elements, 2 * size + 1);
    }
  }
}
```

## 이왕이면 제네릭 메서드로 만들라

- Collections의 **알고리즘** 메서드(binarySearch, sort 등)는 모두 제네릭이다.

- (타입 매개변수들을 선언하는) 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.

  ```java
  // 코드 30-2 제네릭 메서드
  public static <E> Set<E> union(Set<E> s1, Set<E> s2) { // 타입 매개변수 목록 : <E>, 반환 타입 Set<E>
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
  }
  
  ```

- 제네릭 싱글턴 팩터리 패턴 : 요청한 타입 매개 변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리

  - 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 경우
    - 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있다.

  ```java
  // 코드 30-4 제네릭 싱글턴 팩터리 패턴
  private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;
  
  @SuppressWarnings("unchecked")
  public static <T> UnaryOperator<T> identityFunction() { // 항등함수 : 입력 값을 수정 없이 그대로 반환하는 함수
    return (UnaryOperator<T>) IDENTITY_FN;
  }
  
  ```

  ```java
  // 코드 30-5 제네릭 싱글턴을 사용하는 예
  public static void main(String[] args) {
    String[] strings = {"삼베", "대마", "나일론"};
    UnaryOperator<String> sameString = identityFunction();
    for (String s : strings)
      System.out.println(sameString.apply(s));
    
    Number[] numbers = {1, 2.0, 3L};
    UnaryOperator<Number> sameNumber = identityFunction();
    for (Number n : numbers)
      System.out.println(sameNumber.apply(n)); // 형변환을 하지 않아도 컴파일 오류나 경고가 발생하지 않는다.
  }
  
  ```

- 재귀적 타입 한정(recursive type bound) : 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정

  ```java
  public interface Comparable<T> {
    int compareTo(T o); // 타입 매개변수 T는 Comparable<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의
  }
  
  ```

  ``` java
  // 코드 30-6 재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현했다.
  public static <E extends Comparable<E>> E max(Collection<E> c);
  // 타입 한정인 <E extends Comparable<E>>는 모든 타입 E는 자신과 비교할 수 있다는 의미
  
  ```

## 한정적 와일드카드를 사용해 API 유연성을 높이라

- 앞서 소개한 내용에서 나온 매개변수화 타입은 불공변(invariant)이다. 하지만 때론 불공변 방식보다 유연한 **무언가**가 필요하다.

  ```java
  // 코드 31-1 와일드카드 타입을 사용하지 안흔 pushAll 메서드 - 결함이 있다!
  public void pushAll(Iterable<E> src) {
    for (E e : src)
      push(e);
  }
  // 코드 31-3 와일드카드 타입을 사용하지 않은 popAll 메서드 - 결함이 있다!
  public void popAll(Collection<E> dst) {
    while(!isEmpty())
      dst.add(pop());
  } 
  
  ```

  ``` java
  Stack<Number> numberStack = new Stack<>();
  Iterable<Integer> integers = ...;
  numberStack.pushAll(integers); // 매개변수화 타입이 불공변이기 때문에 Integer가 Number의 하위 타입이어도 컴파일되지 않는다.
  
  ```

- 한정적 와일드카드

  ```java
  // 코드 31-2 E 생산자(producer) 매개변수에 와일드카드 타입 적용
  public void pushAll(Iterable<? extends E> src) { // 입력 매개변수 타입은 'E의 하위 타입이 Iterable'이어야 한다.
    for (E e : src)
      push(e); // src 매개변수는 Stack이 사용할 E 인스턴스를 생산
  }
  
  ```

  ```java
  // 코드 31-4 E 소비자(consumer) 매개변수에 와일드카드 타입 적용
  public void popAll(Collection<? super E> dst) { // 매개변수의 타입이 E의 상위 타입의 Collection이어야 한다
    while(!isEmpty()) 
      dst.add(pop()); // dst 매개변수는 Stack으로부터 E 인스턴스를 소비
  }
  
  ```

> 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.

- 매개변수화 티입 T가 생산자라면 `<? extends T>`를 사용하고, 소비자라면 `<? super T>`를사용하라. 

  > **펙스(PECS) : producer-extends, consumer-super** 의 준말로 와일드카드 타입을 사용하는 기본 원칙

- **반환 타입에는 한정적 와일드카드 타입을 사용하면 안된다.**

  ```java
  // PECS 공식에 따라 와일드 카드 타입 사용
  public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2);
  
  // 선언 사용하기. 클라이언트 입장에서는 와일드카드 타입을 신경쓸 필요가 없어야 정상이다.
  Set<Integer> integers = Set.of(1, 3, 5);
  Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
  Set<Number> numbers = union(integers, doubles);
  // Set<Number> numbers = Union.<Number>union(integers, doubles); // 코드 31-6 자바 7까지는 명시적 타입 인수를 사용해야 한다.
  
  ```

- **매개변수와 인수**의 차이 개념 정리

  ```
  매개변수(parameter)는 메서드 선언에 정의한 변수이고, 인수는 메서드 호출시 넘기는 '실젯값'
  
    void add(int value) { ... }
    add(10)
  => value는 매개변수이고 10은 인수
    
    class Set<T> { ... }
    Set<Integer> = ...;
  => T는 타입 매개변수이고 Integer는 타입 인수
  
  ```

- 좀 더 복잡한 메서드 선언 예제 이해하기

  ```java
  public static <E extends Comparable<E>> E max(List<E> list) // 수정 전 메서드 선언
  public static <E extends Comparable<? super E>> E max(List<? extends E> list); // 수정 후 메서드 선언
  // 입력 매개변수애서는 E 인스턴스를 생산하므로 원래의 List<E>를 List<? extends E>로 수정
  // 타입 매개변수에서는 E가 Comparable<E>를 확장함, 이때 Comparable<E>는 E 인스턴스를 소비하기 때문에 Comparable<? super E>로 대체
  
  ```

  ```java
  // 다음 리스트는 오직 위에서 정의된 max 메서드로만 처리할 수 있다.
  List<ScheduledFuture<?>> scheduledFutures = ...; 
  // 한정적 와일드카드를 사용하지 않고 정의된 메서드로는 처리할 수 없다.
  // ScheduledFuture가 Comparable<ScheduledFuture>을 구현하지 않았기 때문
  
  ```

  ```java
  public interface Comparable<E>
  public interface Delayed extends Comparable<Delayed>
  public interface ScheduledFuture<V> extends Delayed, Future<V>
  
  ```

  - 즉, Comparable(혹은 Comparator)을 **직접** 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 **와일드 카드**가 필요하다.
  - `Comparator`, `Comparable`은 언제나 소비자이다. 따라서 일반적으로 `Comparable<? Super E>`를 사용하는 편이 낫다.

- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하라.

  ```java
  // 코드 31-7 swap 메서드의 두 가지 선언
  public static <E> void swap(List<E> list, int i, int j);
  public static void swap(List<?> list, int i, int j);
  
  ```

  - 이때 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.

  ```java
  public static void swap(List<?> list, int i, int j) {
    // list.set(i, list.set(j, list.get(i)))); // List<?>에는 null 외에는 어떤 값도 넣을 수 없기 때문에 컴파일 오류가 나온다.
    swapHelper(list, i, j);
  }
  
  // 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
  private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i))); 
  }
  
  ```

  - 솔직히 위에 코드 이점을 잘 이해 못하겠다. 가독성만 떨어진 느낌.

## 제네릭과 가변인수를 함께 쓸 때는 신중하라

- 가변인수(varargs) 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다.

  `warning: [unchecked] Possible heap pollution from parameterized vararg type List<String>`

  - 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.
  - 즉 제네릭 타입 시스템이 약속한 타입 안전성의 근간이 흔들린다.

  ```java
  // 코드 32-1 제네릭과 varargs를 혼용하면 타입 안전성이 깨진다!
  static void dangerouse(List<String> ... stringLists) { // 가변인수를 호출하면 가변인수를 담기 위한 배열이 자동으로 생성된다.
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // 힙 오염 발생, 가변인수를 담은 배열이 클라이언트에 노출 되어있기 때문에 작성될 수 있는 코드
    String s = stringLists[0].get(0); // ClassCastException
  }
  
  ```

  - 따라서 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

- 자바 7에서는 `@SafeVarags` 애너테이션이 추가되어 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다.

  - @SafeVarags 애너테이션 : 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치.
  - 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않지만, 
    1. 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용하기 때문에 
    2. 메서드가 안전하다는게 확실하다면 @SafeVarags 애너테이션을 달아라.

- 메서드가 안전한지 확인하는 방법

  1. **메서드가 배열(varargs 매개변수를 담는 제네릭 배열)에 아무것도 저장하지 않고,**
  2. **그 배열의 참조가 밖으로 노출되지 않아야 한다.**
  3. 즉 varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면 그 메서드는 안전하다.

  ```java
  // 코드 32-2 자신의 제네릭 매개변수 배열의 참조를 노출한다. - 안전하지 않다! 예제
  static <T> T[] toArray(T... args) {
    return args;
  }
  
  // 언뜻 보면 문제 없어보이는 PickTwo 메서드
  static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
      case 0: return toArray(a, b); // 1. 컴파일러는 toArray에 넘길 T 인스턴스 2개를 담을 varargs 매개변수 배열을 만든다.
      case 1: return toArray(a, c); // 2. 이때 배열의 타입은 Object[] 이다.
      case 2: return toArray(b, c); // 3. (pickTwo에 어떤 타입의 객체를 넘기더라도 담을 수 있는 가장 구체적인 타입이 Object이기 때문)
    }
    throw new Assertion Error(); // 도달할 수 없다.
  }
  
  // pickTwo 사용하기
  String[] attributes = pickTwo("좋은", "빠른", "저렴한"); // ClassCastException 발생
  // 4. Object[]는 String[]의 하위 타입이 아니므로 형변환에 실패한다.
  
  ```

  ```java
  // 코드 32-3 제네릭 varargs 매개변수를 안전하게 사용하는 메서드
  @SafeVarargs
  static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
      result.addAll(list);
    return result;
  }
  
  ```

> 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarags를 달라. 즉 안전하지 않은 varargs 메서드는 절대 작성해서는 안된다.

- @SafeVarargs 애너테이션은 재정의할 수 없는 메서드에만 달아야 한다.

- varargs 매개변수를 List 바꿀 수도 있다.

  ```java
  // 코드 32-4 제네릭 varargs 매개변수를 List로 대체한 예 - 타입 안전하다.
  static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
      result.addAll(list);
    return result;
  }
  
  ```

## 타입 안전 이종 컨테이너를 고려하라

- 타입 안전 이종 컨테이너 패턴(type safe heterogeneous container pattern) : 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공

  ```java
  // 코드 33-3 타입 안전 이종 컨테이너 패턴 - 구현
  public class Favorites { // 타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorite 클래스
    private Map<Class<?>, Object> favorites = new HashMap<>();
    
    public <T> void putFavorite(Class<T> type, T instance) {
      favorites.put(Objects.requireNonNull(type), type.cast(instance)); // 동적 형변환을 통해 instance의 타입이 type으로 명시한 타입과 같은지 확인
    }
    
    public <T> T getFavorite(Class<T> type) {
      return type.cast(favorites.get(type)); // Class의 cast 메서드를 사용해 이 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환한다.
    }
  }
  
  ```

  - 여기서 Favorites 클래스는 실체화 불가 타입에는 사용할 수 없다.
    - `List<String>`용 Class 객체를 얻을 수 없기 때문이다.
