# [3] private 생성자나 열거 타입으로 싱글턴임을 보증하라
## 싱글턴(Singleton)

- 인스턴스를 오직 하나만 생성할 수 있는 클래스
- 예시 : 무상태 객체(함수 등), 설계상 유일해야하는 시스템 컴포넌트

```java
// 싱글턴 클래스
public class SingletonService {
    private static final SingletonService instance = new SingletonService();

    // private 생성자
    private SingletonService() {}

    // 싱글턴 인스턴스 반환
    public static SingletonService getInstance() {
        return instance;
    }

    public void performService() {
        System.out.println("Service is performed");
    }
}

// 싱글턴을 사용하는 클라이언트 코드
public class Client {
    private final SingletonService service;

    public Client() {
        this.service = SingletonService.getInstance();
    }

    public void doSomething() {
        service.performService();
    }
}
```

### 장점

- 한 번의 객체 생성으로 재사용 가능하므로 메모리 낭비를 줄일 수 있음

### 문제점

- 클라이언트를 테스트하기 어려움
    - 타입을 인터페이스로 정의하고 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 가짜(mock) 구현으로 대체 할 수 없기 때문
        
        ⇒ 싱글톤 클래스 자체는 가짜 객체(Mock)을 만들 수 없음
        
        ⇒ 인터페이스를 구현한 싱글톤 클래스는 가짜 객체(Mock)을 만들 수 있음
        
        ** mock : 테스트 할 때 필요한 실제 객체와 동일한 모의 객체를 만들어 테스트의 효용성을 높이기 위해 사용
        
- private 생성자를 갖고 있어 상속이 불가능

## 싱글턴 만드는 방식

생성자는 private으로 감춰두고, 유일하게 인스턴스로 접근할 수 있는 수단으로 public static 멤버를 마련

### [1] public static final 필드 방식

```java
// Elvis 클래스 정의
public class Elvis {
		public static final Elvis INSTANCE = new Elvis();
		private Elvis() { ... }  // private 생성자로 외부에서 인스턴스화하지 못하도록 막음
		
		public void sing() {
        System.out.println("Singing..");
    }
}

// 사용
public class Main {
    public static void main(String[] args) {
        // Elvis 싱글턴 인스턴스 가져오기
        Elvis elvis = Elvis.INSTANCE;

        // 메서드 호출
        elvis.sing(); // 출력: Can't help falling in love...
    }
}
```

- `private` 생성자는 `public static final` 필드인 `Elvis.INSTANCE`를 초기화 할 때 딱 1번만 호출됨
- `public`이나 `protected` 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장됨

**장점**
- 해당 클래스가 싱글턴임이 API에 명확하게 드러남
- `public static` 필드에 `final`을 붙여 재할당을 막았기 때문에 다른 객체 참조 불가능
- 간결함

**단점**
- 클라리언트가 사용하지 않더라도 인스턴스가 항상 생성됨

    ⇒ 메모리 낭비
    

**[예외]**

- 권한이 있는 클라리언트는 리플렉션 API인 `AccessibleObject.setAccessible`을 사용하여 private 생성자 호출 가능
    
    ```java
    import java.lang.reflect.Constructor;
    
    public class Main {
        public static void main(String[] args) {
            // Elvis 싱글턴 인스턴스
            Elvis elvis = Elvis.INSTANCE;
    
            try {
                // 리플렉션을 사용하여 Elvis의 private 생성자에 접근
                Constructor<Elvis> constructor = Elvis.class.getDeclaredConstructor();
                constructor.setAccessible(true);
    
                // 새로운 인스턴스 생성 시도 (예외 발생 예상)
                Elvis elvis2 = constructor.newInstance();
            } catch (Exception e) {
                System.out.println(e.getMessage());
            }
        }
    }
    
    ```
    
- 공격 방어를 위해 생성자에서 두 번째 객체를 생성하려고 할때 예외 던지기
    
    ```java
    // Elvis 클래스
    public class Elvis {
        // 싱글턴 인스턴스
        public static final Elvis INSTANCE = new Elvis();
    
        // 인스턴스가 이미 생성되었는지 여부를 추적하는 플래그
        private static boolean created = false;
    
        // private 생성자
        private Elvis() {
            if (created) {
                throw new IllegalStateException("인스턴스가 이미 존재합니다.");
            }
            created = true;
        }
    
        public void sing() {
            System.out.println("Singing..");
        }
    }
    ```

### [2] 정적 팩터리 메서드 방식

```java
public class Elvis {
		// 싱글턴 인스턴스 (private)
		private static final Elvis INSTANCE = new Elvis();
		private Elvis() { ... }
		public static Elvis getInstance() { 
				return INSTANCE; 
		}
		
		public void sing() {
        System.out.println("Singing..");
    }
}

public class Main {
    public static void main(String[] args) {
        // Elvis 싱글턴 인스턴스를 정적 팩터리 메서드를 통해 가져오기
        Elvis elvis = Elvis.getInstance();

        // 메서드 호출
        elvis.sing();
    }
}
```

- `getInstance()`는 항상 같은 객체의 참조를 반환하므로 싱글톤이 보장됨
    
    (제 2의 Elvis 인스턴스는 만들어지지 않음)
    
- 리플렉션 API를 통한 접근은 여전히 가능하므로 `public static final` 방식과 같이 예외처리가 필요함

**장점**

- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있음
    
    ```java
    public class Elvis {
    		// 싱글턴 인스턴스 (private)
    		private static final Elvis INSTANCE = new Elvis();
    		private Elvis() { ... }
    		public static Elvis getInstance() { 
    				// INSTANCE 대신에 새로운 인스턴스 생성
    				return new Elvis(); 
    		}
    		
    		public void sing() {
            System.out.println("Singing..");
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Elvis elvis = Elvis.getInstance();
            
            elvis.sing();
        }
    }
    ```
    
    - 유일한 인스턴스를 반환하는 정적 팩터리 메서드가 호출하는 스레드마다 다른 인스턴스를 넘겨주게 설정할 수 있음
- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있음
    
    ** 제네릭 싱글턴 팩터리 : 제네릭으로 타입설정 가능한 인스턴스를 만들어두고, 반환 시에 제네릭으로 받은 타입을 이용해 타입을 결정하는 것
    
- 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있음
    
    ex) `Elvis::getInstance`를 `Supplier<Elvis>`로 사용 가능
    
    ```java
    // 정적 팩터리 메서드를 Supplier로 참조
    Supplier<Elvis> elvisSupplier = Elvis::getInstance;
    // Supplier를 통해 Elvis 인스턴스를 얻음
    Elvis elvis = elvisSupplier.get();
    ```
    

**단점**

- 클라이언트가 사용하지 않더라도 인스턴스가 항상 생성됨
    
    ⇒ 메모리 낭비

### [1], [2] 방식의 직렬화

**문제**

- 위 2가지 방식으로 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것은 부족
- 직렬화한 인스턴스를 역직렬화할때마다 새로운 인스턴스가 생김
    - 직렬화된 인스턴스를 역직렬화하면 리플렉션 API가 사용되고 리플렉션을 통해 인스턴스가 생성됨
    
        ( 기본 생성자를 호출하지 않고 인스턴스 값을 복사해서 새로운 인스턴스를 반환 )
        

**해결방법**

모든 인스턴스를 일시적(transient)이라고 선언하고 `readResolve` 메서드를 제공해야함

** `transient` : 필드의 상태값을 전송하지 않는다는 키워드

```java
public class Elvis {
		// 싱글턴 인스턴스 (private)
		private static final Elvis INSTANCE = new Elvis();
		private Elvis() { ... }
		public static Elvis getInstance() { 
				return INSTANCE; 
		}
		
		// 싱글턴임을 보장해주는 readResolve 메서드
		private Object readResolve() {
				// 진짜 Elvis를 반환하고 가짜 Elvis는 가비지 컬렉터에 맡김
				return INSTANCE;
		}
		
		public void sing() {
        System.out.println("Singing..");
    }
}
```

- 역직렬화 과정에서 싱글톤으로 만들어둔 진짜 Elvis를 반환하도록 설정


### [3] 열거 타입 방식

```java
public enum Elvis {
		INSTANCE;
}
```

**장점**

- `public` 필드 방식과 비슷하지만 더 간결함
- 추가 노력 없이 직렬화 가능
    - 복잡한 직렬화 상황이나 리플레션 공격에도 제2의 인스턴스가 생기는 일을 완벽히 막아줌

대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법

만드려는 싱글턴이 `enum` 외의 클래스를 상속해야한다면 열거 타입 방식은 사용할 수 없음

- 열거 타입이 다른 인터페이스를 구현하도록 선언 가능
    
    ```java
    // Elvis는 ElvisInterface의 sing 메서드를 구현
    public interface ElvisInterface{
    		void sing();
    }
    
    // Elvis 열거 타입은 싱글턴을 구현, ElvisInterface를 구현
    public enum Elvis implements ElvisInterface {
    		INSTANCE;
    		
    		@Override
    		public void sing() {
    				System.out.println("singing~");
    		}
    }
    ```