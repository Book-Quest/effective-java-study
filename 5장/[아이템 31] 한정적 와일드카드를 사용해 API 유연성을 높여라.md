## [아이템 31] 한정적 와일드카드를 사용해 API 유연성을 높여라

### 1. 특별한 매개변수화 타입, 한정적 와일드 카드 타입
- 매개변수화 타입은 불공변(invariant)이다.
  - String은 Object의 하위 타입이지만, `List<String>`은 `List<Object>`의 하위 타입이 아니다.
  - `List<String>`은 `List<Object>`가 하는 일을 제대로 수행하지 못하므로, 리스코프 치환 원칙에 어긋난다.
- 따라서, 유연성을 극대화하려면 **한정적 와일드카드 타입**을 사용하면 된다.

### 2. 와일드 카드 타입의 2가지 선언 방법, 생산자용 그리고 소비자용

- **입력** 매개변수가 생산자인지 소비자인지에 따라 타입을 확실하게 지정해야 한다.

> PECS 공식: producer-extends, consumer-super
> 
- 입력 매개변수화 타입 T
    - 생산자: `<? extends T>`
        
        ```java
        public void pushAll(Iterable<? extends E> src) {
        	for (E e : src) push(e);
        }
        ```
        
    - 소비자: `<? super T>`
        
        ```java
        public void popAll(Iterable<? super E> dst) {
        	while(!isEmpty()) dst.add(pop());
        }
        ```
        
- **입력** 매개변수가 생산자와 소비자 역할을 **동시에** 한다면 와일드카드 타입을 사용하지 말자.
- 또한, **반환 타입**에 한정적 와일드카드를 사용하면, 클라이언트 코드에서도 와일드카드 타입을 사용해야 하므로, 사용하지 말자.
    - 하지만, 자바 7에서는 제네릭 메서드를 사용 시, 타입 인수를 명시적으로 지정해야 한다. (타입 추론 기능이 부족하기 때문이다.)
        
        ```java
        Set<Number> numbers = Union.<Number>union(integers, doubles);
        ```
        

### 3. 와일드카드 타입은 다형성을 더욱 잘 활용할 수 있게 한다
- 와일드카트 타입을 사용하면 오히려 더 복잡하게 보일 수 있지만, 그럼에도 와일드카드 타입을 사용하는 것이 낫다.
- `extends`하는 상위 타입을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원할 수 있기 때문이다.
- 예시: 재귀적 타입 한정 코드
    - 와일드카드 타입 미사용:
        
        ```java
         public static <E extends Comparable<E>> E max (List<E> list>
        ```
        
    - **[권장]** 와일드카드 타입 사용 :
        
        ```java
        public static <E extends Comparable<? super E>> E max (
        				List<? extends E> list)
        ```
        
    - Comparable를 구현한 인터페이스 Sup, 그리고 Sup의 하위 인터페이스 Sub가 있다면, Sub는 Comparable(혹은 Comparator)를 직접 구현하지 않고도 `max()` 를 사용할 수 있다.
- 따라서, 소비자인 Comparator는 `Comparator<E>` 보다는 `Comparator<? super E>`를 사용하는 것이 낫다. (Comparable도 마찬가지)

### 4. 대외적으로는 와일드카드 타입을 사용하고 내부적으로는 제네릭을 사용하는 것도 방법이다
- 메서드를 정의할 때 와일드카드 타입과 제네릭 모두 사용 가능하고, 타입 매개변수가 한 번만 나오면, 와일드카드를 사용해라.
- 하지만, 와일드카드 타입의 컬렉션에는 null 외에는 어떤 값도 넣을 수 없다.
- 따라서, 대외적으로는 아래와 같이 와일드 카드를 사용하며,  와일드카드 타입의 실제 타입을 알려주는 메서드를 내부적으로 작성하여 활용하면 컴파일 오류도 나지 않으며 **public API**를 깔끔하고 유연하게 지원할 수 있다.
    
    ```java
    
    public static void swap(List<?> list, int i, int j) {
    	swapHelper(list, i, j);
    }
    
    private static <E> void swapHelper(List<E> list, int i, int j) {
    	list.set(i, list.set(j, list.get(i)));
    }
    ```
    

### 5. 결론

- 한정적 와일드카드는 제네릭 타입의 유연성을 극대화하고 다형성을 활용하는 데 중요한 역할을 한다.
- `<? extends T>`와 `<? super T>`를 사용하여 메서드의 생산자와 소비자 역할을 명확히 구분할 수 있으며, 이를 통해 API의 설계가 더 직관적이고 유지보수가 용이해진다.
