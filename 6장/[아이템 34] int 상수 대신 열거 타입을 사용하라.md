# [34] int 상수 대신 열거 타입을 사용하라 

## 정수 열거 패턴 (int enum pattern)

- 자바에서 열거 타입을 지원하기 전에 사용

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

### 단점

1. **타입 안전을 보장할 방법이 없고 표현력이 좋지 않음**
    
    ex) 오렌지를 건네야 할 메서드에 사과를 보내고 동등연산자로 비교해도 컴파일러는 아무런 경고 메시지를 출력하지 않음 (`int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;`)
    
    - 자바가 정수 열거 패턴을 위한 별도의 네임스페이스를 지원하지 않으므로 접두어를 써서 이름 충돌을 방지
2. **정수 열거 패턴을 사용한 프로그램을 깨지기 쉬움**
    - 상수를 나열한 것뿐이라 컴파일 시 값이 클라이언트 파일에 그대로 새겨지므로 상수의 값이 바뀌면 클라이언트도 다시 컴파일 해야함
    - 다시 컴파일 하지 않은 클라이언트는 엉뚱하게 동작
3. **정수 상수는 문자열로 출력하기가 다소 까다로움**
4. **같은 정수 열거 그룹에 속한 모든 상수를 한바퀴 순회하는 방법이 마땅치 않음**
    - 같은 정수 열거 그룹에 속한 모든 상수를 얻을 수 없을뿐더러 그안에 상수가 몇개인지 알 수 없음

## 열거 타입 (enum type)

> 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입
> 

ex) 사계절, 태양계의 행성, 카드게임의 카드 종류 등

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

- 자바의 열거타입은 완전한 형태의 **클래스** (다른 언어의 열거 타입보다 훨씬 강력)
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개
- 열거 타입 선언으로 만들어진 인스턴스는 딱 하나만 존재 (인스턴스 통제)
    - 밖에서 접근 가능한 **생성자를 제공하지 않아 사실상 final**이므로 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없음
    - 싱글턴은 원소가 하나인 열거타입, 열거타입은 싱글턴의 일반화 형태

### 장점

1. **열거 타입은 컴파일타임 타입 안전성을 제공**
    
    ```java
    // Apple 열거 타입을 매개변수로 받는 메서드
    public static void describeApple(Apple apple) {
        switch (apple) {
            case FUJI:
                System.out.println("A sweet and crisp Fuji apple.");
                break;
            case PIPPIN:
                System.out.println("A tart and green Pippin apple.");
                break;
            case GRANNY_SMITH:
                System.out.println("A sour and firm Granny Smith apple.");
                break;
        }
    }
    ```
    
    - Apple 열거 타입을 매개변수로 받는 메서드를 선언했다면 건네받은 참조는 null이 아니라면 Apple의 3가지 값 중 하나 (다른 타입을 넘기면 컴파일 오류 발생)
2. **각자의 이름 공간이 있어서 이름이 같은 상수도 평화롭게 공존**
    - 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않으므로, 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 됨
3. **열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어줌**
4. **열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수 있음**
    - Object 메서드, Comparable, Serializable을 구현했으며, 그 직렬화 형태도 웬만큼 변형을 가해도 문제없이 동작하게끔 구현해놓음
    
    ex) 과일의 색을 알려주거나 과일 이미지를 반환하는 메서드 추가
    
    ```java
    public enum Apple {
        FUJI("Red", "https://example.com/images/fuji.png"),
        PIPPIN("Green", "https://example.com/images/pippin.png"),
        GRANNY_SMITH("Green", "https://example.com/images/granny_smith.png");
    
        private final String color;
        private final String imageUrl;
    
        // 생성자: 각 열거 상수에 대해 색상과 이미지 URL을 설정
        Apple(String color, String imageUrl) {
            this.color = color;
            this.imageUrl = imageUrl;
        }
    
        // 과일의 색을 반환하는 메서드
        public String getColor() {
            return color;
        }
    
        // 과일의 이미지 URL을 반환하는 메서드
        public String getImageUrl() {
            return imageUrl;
        }
    }
    ```

    ### 열거 타입의 사용

```java
// 태양계 8개의 행성
public enum Planet {
		// 각 열거 상수는 행성의 질량과 반지름 (생성자에 넘겨지는 매개변수)
    MERCURY(3.302e+23,2.439e6),
    VENUS(4.869e+24,6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23,3.393e6),
    JUPITER(1.899e+27,7.149e7),
    SATURN(5.685e+26,6.027e7),
    URAUS(8.683e+25,2.556e7),
    NEPTUNE(1.024e+26,2.477e7);

    private final double mass;
    private final double radius;
    //표면중력
    private final double surfaceGravity;

    //중력상수
    private static final double G = 6.67300E-11;

		// 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {
        return mass;
    }

    public double radius() {
        return radius;
    }

    public double surfaceGravity() {
        return surfaceGravity;
    }
    
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```

- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장
- 필드를 private으로 두고 별도의 public 접근자 메서드를 두는 것이 좋음

```java
public class WeightTable {
		public static void main(String[] args) {
				// 지구의 무게를 입력받기
				double earthWeight = Double.parseDouble(args[0]);
				// 8개의 행성 무게를 출력
				double mass = earthWeight / Planet.EARTH.surfaceGravity();
				for (Planet p : Planet.values())
						System.out.printf("%s에서의 무게는 %f이다.%n, p, p.surfaceWeight(mass));
		}
}
```

- 열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적메서드 `values` 제공 (값들은 선언된 순서로 저장)
- 열거 타입에서 상수를 하나 제거한다면?
    - 제거한 상수를 참조화하지 않는 클라이언트에는 아무 영향이 없음
    - 제거된 상수를 참조하는 클라이언트에서는 클라리언트 프로그램을 다시 컴파일하면 제거된 상수를 참조하는 줄에서 컴파일 오류가 발생
- 기능을 클라이언트에 노출해야할 이유가 없다면 private, package-private으로 선언
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만들기

### 상수별 메서드 구현

> 상수마다 동작이 달라져야 하는 경우
> 

**[값에 따라 분기하는 열거 타입] - switch문**

```java
public enum Operation {
    PLUS,MINUS,TIMES,DIVDE;
    
		// 상수가 뜻하는 연산을 수행
    public double apply(double x, double y) {
        switch (this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산:" + this);
    }
}
```

- 마지막 throw문은 실제로 도달할 일이 없지만 기술적으로 도달할 수 있기 때문에 생략하면 컴파일이 안됨
- 깨지기 쉬운 코드이며, 새로운 상수를 추가하면 case문을 추가해야함
    
    case문에 연산을 추가하지 않은 경우, 런타임 오류가 발생
    

**[상수별 메서드 구현 방식]**

- 열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 클래스 몸체, 즉 각 상수에서 자신에 맞게 재정의하는 방식

```java
public enum Operation {
		// 상수별 메서드 구현(apply)
		// 각 열거 상수는 연산자를 나타내는 symbol을 가지고 있음
    PLUS("+") { public double apply(double x, double y) { return x + y; } },
    MINUS("-") { public double apply(double x, double y) { return x - y; } },
    TIMES("*") { public double apply(double x, double y) { return x * y; } },
    DIVDE("/") { public double apply(double x, double y) { return x / y; } };

		// 연산자 문자열 저장 필드
    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

		// toString을 재정의해 해당 연산을 뜻하는 기호를 반환
    @Override
    public String toString() {
        return symbol;
    }

		// 각 열거 상수는 이 메서드를 오버라이드하여 자신의 연산을 구현
    public abstract double apply(double x, double y);
}
```

- 새로운 상수 추가 시 apply를 재정의해야한다는 사실을 알 수 있으며, 재정의되지 않았다면 컴파일 오류 발생
- `valueOf(String)` 메서드 : 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 메서드 (자동 생성)
    
    ex) `Operation i = Operation.valueOf("PLUS");`
    
- 열거 타입의 `toString` 메서드를 재정의할 때, `toString`이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 `fromString` 메서드도 함께 제공하는 것을 고려
    
    ```java
    // 열거타입 정적 필드의 생성 시점
    private static final Map<String, Operation> stringToEnum =
            Stream.of(values())
                  .collect(toMap(Object::toString, e -> e));
    
    // 지정한 문자열에서 해당하는 Operation을 반환
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
    ```

    **[단점]**

- 열거 타입 상수끼리 코드를 공유하기 어려움

### 전략 열거 타입 패턴

**[기존 코드] 열거타입 상수의 단점 : 열거 타입 상수끼리 코드 공유가 어려움**

```java
// 급여명세서를 사용할 요일 열거 타입
// 일당 계산 메서드 포함
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        int overtimePay;
        switch (this) {
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 
                    0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```

- 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case문을 잊지 말고 쌍으로 넣어줘야하므로 관리 관점에서 위험하다

**[전략 열거 타입 패턴]**

```java
enum PayrollDay {
		// 각 요일에 대한 적절한 PayType을 할당
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), 
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;  // 전략 열거 타입을 사용

    PayrollDay(PayType payType) { this.payType = payType; }

    // 근무 시간을 받아 잔업 수당을 포함한 최종 급여를 계산하는 메서드
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

		// private 중첩 열거 타입
    enum PayType {
        WEEKDAY {  // 평일에 대한 잔업 수당 계산 전략
          int overtimePay(int minsWorked, int payRate){
              return minsWorked <= MINS_PER_SHIFT ? 
                  0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
          }
        },
        WEEKEND {  // 주말에 대한 잔업 수당 계산 전략
            int overtimePay(int minsWorked, int payRate){
                return minsWorked * payRate / 2;
            }
        };
				// 각 전략에 대한 잔업 수행 계산 메서드
        abstract int overtimePay(int minsWorked, int payRate);
        // 기본 근무 시간 계산
        private static final int MINS_PER_SHIFT = 8 * 60;
				// 최종 급여를 계산하는 메서드
        public int pay(int minsWorked, int payRate){
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

}
```

- 잔업 수당 계산을 private 중첩 열거 타입(PayType)으로 옮기고 PayrollDay 열거 타입의 생성자에서 적당한 것을 선택
- PayrollDay 열거 타입은 잔업 수당 계산을 전략 열거 타입에 위임하여, switch문이나 상수별 메서드 구현이 필요 없게 됨
- switch문 보다 복잡하지만 더 안전하고 유연

### switch문이 적합한 경우

기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch문이 더 좋은 선택

```java
public static Operation inverse(Operation op) {
  switch (op) {
    case PLUS:      return Operation.MINUS;
    case MINUS:     return Operation.PLUS;
    case TIMES:     return Operation.DIVIDE;
    case DIVIDE:    return Operation.TIMES;
    default: throw new AssertionError("알 수 없는 연산: " + this);
  }
}
```

- 추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 이 방식 적용하는 것이 좋음

### 열거 타입을 언제 사용할까?

- 대부분의 경우에 열거 타입의 성능은 정수 상수와 별반 다르지 않음
- 필요한 원소를 컴파일타입에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자
    - 본질적으로 열거 타입인 타입 (태양계 행성, 한주의 요일, 체스말 등)
    - 허용하는 값 모두를 컴파일 타임에 알고 있을 때
    (메뉴 아이템, 연산 코드, 명령줄 플래그 등)
- 열거 타입에 정의된 상수 개수가 영원이 고정일 필요는 없음

## 정리

- 열거 타입은 더 읽기 쉽고 안전하고 강력함
- 대다수의 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요
- 하나의 메서드가 상수별로 다르게 동작해야할 때는 상수별 메서드 구현을 사용하자
- 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자
