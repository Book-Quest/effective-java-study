## @Override란?

메서드 선언에만 달 수 있음. 상위 타입의 메서드를 재정의했음을 의미.

이를 일관되게 사용하면 버그를 예방할 수 있음.

### 예시

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) // 10번 반복
            for (char ch = 'a'; ch <= 'z'; ch++) // 26개
                s.add(new Bigram(ch, ch)); // 똑같은 소문자 2개
        System.out.println(s.size());
    }
}
```

위의 코드는 영어 알파벳 2개로 구성된 문자열을 표현하는 클래스.

### 의도

똑같은 소문자 2개로 구성된 바이그램 객체 26개를 10번 반복해 집합에 추가한 후 그 집합의 크기를 출력.

### 문제

Set은 중복을 제거하니 26이 출력될 것 같지만, 실제로 260이 출력됨.

### 원인

equals, hashcode를 재정의 한 게 아니라, 다중정의 했기 때문.

(Override가 아니라 Overloading)

즉, Object의 equals, hashcode를 Override한게 아니라 그냥 새로 정의한 것이 되어버림.

‘==’는 객체의 식별성만 확인하므로 각각 다른 객체로 인식함.

### 해결

```java
public class Bigram2 {
    private final char first;
    private final char second;

    public Bigram2(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Bigram2))
            return false;
        Bigram2 b = (Bigram2) o;
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram2> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram2(ch, ch));
        System.out.println(s.size());
    }
}
```

이처럼 @Override 애너테이션을 달고 수정 가능

Object를 상속받아 바이그램 클래스로 형변환하여 코드를 수정.

즉, 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달아야함.

## @Override의 사용법

구체 클래스에서 상위 클래스의 메서드를 재정의할 때는 굳이 @Override를 달지 않아도 됨.

구체 클래스인데 아직 구현하지 않은 추상메서드가 있다면 컴파일러가 바로 에러를 띄우기 때문에.

또한 대부분의 IDE는 재정의할 메서드를 선택하면 자동으로 @Override를 붙여줌.

- 구체 클래스

Concrete clas로 구체, 구현, 구상 클래스라고 번역되며 추상 클래스가 아닌 인스턴스를 생성할 수 있는 모든 클래스를 통칭함.

IDE는 @Override를 일관되게 사용하도록 함.

관련 설정이 활성화 되어 있다면 @Override를 붙이지 않고 재정의가 되어 있다면 경고를 줌.

(컴파일 오류의 보완재 역할)

@Override는 클래스뿐 아니라 인터페이스의 메서드를 재정의할 때도 사용할 수 있음.

디폴트 메서드를 지원하기 시작하면서, 인터페이스 메서드를 구현한 메서드에서도 @Override를 사용 가능.

구현하려는 인터페이스에 디폴트 메서드가 없다면 @Override 생략해 코드를 깔끔하게 작성해도 됨.

But, 추상클래스, 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 다는 것이 좋음.

⇒  재정의한 것인지 아닌지 명확히 구분되기 때문.

ex) Set 인터페이스는 Collection 인터페이스를 확장했지만, 새로 추가한 메서드는 없었음.

그래서 모든 메서드 선언에 @Override를 달아 실수로 추가한 메서드가 없음을 보장함.

## Summary

재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 실수시에 컴파일러가 알려줌.

한 가지 예외는 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우에는 @Override를 달지 않아도 됨.(달아도 상관없긴 함.)
