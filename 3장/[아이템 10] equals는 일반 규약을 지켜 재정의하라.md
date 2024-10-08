## [아이템 10] equals는 일반 규약을 지켜 재정의하라

### 1. equals 재정의는 언제 사용 되는가?

- 필요한 케이스:
    - 객체 식별성(Object Identity; 두 객체가 물리적으로 같은가)이 아닌 논리적 동치성(Logical Equality; 두 객체의 논리적이 값이 같은가)을 비교하도록 재정의 되어야 할 때
- 필요하지 않은 케이스:
    - 각 인스턴스가 본질적으로 고유할 때
        - 예시: Thread, Enum → 논리적 동치성과 객체 식별성이 사실상 똑같은 의미를 지닌다.

### 2. equals 재정의 시 따라야 하는 일반 규약

- 반사성:
    - `x.equals(x)`는 항상 `true` 여야 한다. 즉, 객체는 자기 자신과 같아야 한다.
- 대칭성:
    - `x.equals(y)`가 `true`이면 `y.equals(x)`도 `true` 여야 한다.
    - 메서드를 호출하는 객체와 비교하려는 객체를 반대로 바꿔서 비교했을 때도 동일한 결과를 얻을 수 있어야 한다.
- 추이성:
    - `x.equals(y)`가 `true`이고 `y.equals(z)`가 `true`이면 `x.equals(z)`도 `true`여야 한다.
    - 하위 클래스에 equals 비교에 영향을 주는 필드를 추가했을 때, 무시하고 비교 시 추이성이 깨질 뿐만 아니라,  무한 재귀에 빠질 수 있다.
        - **따라서, 구체 클래스**를 확장하여 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 없다. 객체지향적 추상화의 이점을 포기하지 않는 한 말이다.
            - 다형성 (상속)을 구현하기 위해 하위 클래스를 만들었지만, 하위 클래스를 추가할 때마다 class 타입이 동일한지(==) 확인하도록 상위 클래스의 equals를 오버라이딩한다면 추이성은 준수할 수 있지만, 다형성의 원리(리스코프의 치환 원칙)을 준수하지 못한다.
            - 상속 대신 컴포지션을 이용하면 나름대로 괜찮은 우회 방법이 될 수 있다.
            - 물론, 아무런 값을 갖지 않는 추상 클래스의 하위 클래스라면, equals 규약을 지키면서도 확장가능하다. (값을 추가할 수 있다.)
- 일관성:
    - `x.equals(y)`의 결과는 `x`나 `y`의 상태가 변경되지 않는 한 항상 같은 값을 반환해야 한다.
    - 가변 객체는 비교시점에 따라 다를 수도 같을 수도 있지만, 불변 객체는 한번 다르면 끝까지 달라야 한다.
    - equals 판단에 신뢰할 수 없는 자원이 끼어들면 안된다.
    - 예시: java.net.URL의 equals
        - 호스트 이름 해석의 불확실성(DNS 캐싱, 호스트 이름의 변경, 또는 호스트 이름이 여러 IP 주소를 가리킬 수 있는 경우 URL.equals가 예상과 다르게 동작할 수 있다.)
- Not Null:
    - equals 메서드는 동치 관계(equivalence relation)를 구현하며, 비교하는 두 참조 값은 null이 아니어야 한다.
        - 명시적으로 ≠null 코드를 삽입하기 보다는 묵시적인 null 검사(instanceof를 활용한 클래스 타입 검사)가 권장된다.
        - null도 정상 값으로 취급하는 참조 타입 필드인 경우, NPE를 방지하기 위해 equals 대신 `Objects.equal(Object, Object)`를 사용하는 것이 적절하다.

> #### 객체지향의 다형성
> 
> 
> 객체 지향 프로그래밍에서 다형성이란 한 타입의 참조변수를 통해 여러 타입의 객체를 참조할 수 있도록 만든 것을 의미한다. 좀 더 구체적으로, 상위 클래스 타입의 참조변수로 하위 클래스의 객체를 참조할 수 있도록 하는 원리이다.
> 

> #### 리스코프의 치환 원칙
> 
> 
> 리스코프 치환 원칙은 1988년 바바라 리스코프(Barbara Liskov)가 올바른 상속 관계의 특징을 정의하기 위해 발표한 것으로,서브 타입은 언제나 기반 타입으로 교체할 수 있어야 한다는 것을 뜻한다. 교체할 수 있다는 말은, 자식 클래스는 최소한 자신의 부모 클래스에서 가능한 행위는 수행이 보장되어야 한다는 의미이다.
> 

### 3. 결론

- equals를 올바르게 오버라이딩하려면, instanceOf로 올바른 타입인지 확인해야 하며, 비교하는 두 객체의 핵심 필드들이 모두 일치하는지 검사해야 한다.
- instanceof는 검사하려는 객체가 null이면(첫번째 피연산자), false를 반환하기 때문이다.
- 어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기 때문에, 최상의 성능을 바란다면 다를 가능성이 크거나 비교하는 비용이 싼 필드를 먼저 비교하자.
- (아이템 11) equals를 재정의할 땐 hash code(주소값으로 만든 고유한 숫자값)도 반드시 재정의하자.
- object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
    - 구체 타입을 매개변수로 받는 경우는 Object.equals를 재정의한 게 아니라 다중정의를 한 것이다.
    - 이 때, `@Override` 어노테이션을 사용하면 긍정 오류(실제 문제와 무관하게)를 발생시킨다.
- **꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 이런 규칙들을 전부 수용하며 준수하기는 까다롭기 때문에, 부주의하게 실수할 바에는 재정의하지 않는 겟 낫다.**

### References

- [객체 지향 프로그래밍의 4가지 특징ㅣ추상화, 상속, 다형성, 캡슐화](https://www.codestates.com/blog/content/%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%ED%8A%B9%EC%A7%95)
- [💠 완벽하게 이해하는 LSP (리스코프 치환 원칙)](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-LSP-%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84-%EC%B9%98%ED%99%98-%EC%9B%90%EC%B9%99)
