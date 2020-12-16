---
title : "[우테코] 프리코스 3주차"

categories :

- Education

tags :

- Education
---
해당 글은 우테코 프리코스 3주차를 진행하면서 배우고 고민한 내용을 정리한 글입니다.

## 1. Set

이번 과제를 하는 도중 Set을 사용하여 구현을 하고자 했다. 평소 자주 사용하기는 했지만 사용법만 알았기 때문에 금방 벽에 막혀버리고 말았다. 쉽게 길을 찾지 못하고 Set에서 헤매이며 내가 Set에 대해서 많이 알지 못한다는 것을 깨달아 Set의 특징을 새로 배우고 정리하였다.

### 1-1. Set의 특징
Set의 일반적인 특징은 다음과 같다.

1. 입력 순서를 보장하지 않는다.
2. 중복 원소를 가질 수 없다.

이러한 Set의 특징은 AbstractSet<E> 를 상속받은 HashSet의 특징이다.

참고로 Set의 상속 구조는 다음과 같다.

![Untitled](https://user-images.githubusercontent.com/56301069/102365429-e453c080-3ffa-11eb-8180-cacbdc662309.png)

그렇다면 Set이 이러한 특징을 갖는 이유를 알아보자.

### 1-2. 입력 순서를 보장하지 않는 이유

Hash가 붙은 이름에서 알 수 있듯이 Hashing을 사용하여 원소를 저장하기 때문이다. 입력받은 객체의 hashCode를 토대로 해시 버킷의 인덱스에 해당 객체를 저장한다. 객체의 hashCode에 따라서 해시 버킷에 저장되는 위치가 다르기 때문에 입력 순서를 보장하지 않는다.

![Untitled](https://user-images.githubusercontent.com/56301069/102365497-f6cdfa00-3ffa-11eb-934c-a4209356105f.png)

### 1-3. 중복 원소를 가지지 않는 이유

Set은 Map을 속성으로 가지고 있고, 이를 사용하여 Set을 구현하였기 때문에 중복 원소를 가지지 않는다. 다음은 HashSet의 구현 코드이다.

![스크린샷 2020-12-16 오후 8 57 19](https://user-images.githubusercontent.com/56301069/102365531-ffbecb80-3ffa-11eb-9905-2cbf20a7d194.png)

HashSet은 HashMap을 속성으로 가지고 있다.

HashSet은 map을 사용하여 삽입할 값을 가지고 있는지 여부를 쉽게 알 수 있다.

다음은 HashSet이 값을 삽입하는 코드이다.

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

add() 메소드가 호출되면 가지고 있는 Map에 인자로 들어온 E를 삽입한다. map.put()은 리턴값으로 해당 키에 대한 기존 밸류를 반환한다. map이 E를 키로 갖고 있지 않을 경우 Map은 null을 반환하고, E를 가지고 있을 경우 이미 저장되어 있는 값 `PRESENT` 를 반환한다.

해당 원소가

- map에 저장되어 있지 않다면, map은 null을 반환하고 원소 추가에 성공했다는 의미로 true를 반환한다.

- map에 이미 원소가 저장되어 있다면, PRESENT를 반환하고 원소 추가에 실패했다는 의미로 false를 반환한다.

여기서 PRESENT는 위 사진의 객체인 Object 타입의 PRESENT 이다. 이 객체는 Dummy 객체로 새로운 원소가 저장될 때, 해당 값이 들어가있는지 없는지를 판단하기 위한 용도로 사용된다.

### 1-4. LinkedHashSet의 순서 보장

HashSet은 Hashing을 사용하기 때문에 입력 순서가 보장되지 않는다. 하지만 LinkedHashSet은 입력 순서를 보장한다는 특징을 가지고 있다. 같은 Hash를 사용하는 LinkedHashSet은 어떻게 입력 순서를 보장할까?

LinkedHashSet의 기본 생성자는 다음과 같다.

```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
				
        ...

        public LinkedHashSet() {
            super(16, .75f, true);
        }
        
        ...
}
```

상위 클래스의 생성자 super()를 따라 올라가면, 다음과 같은 HashSet의 생성자가 나온다.

```java
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

이 생성자를 통하여 LinkedHashSet은 LinkedHashMap을 속성으로 갖는 Set이라는 것을 알 수 있다. LinkedHashSet을 보면 따로 입력 순서를 보장하는 코드가 없다. 그러므로 LinkedHashMap이 순서를 보장해준다는 것을 확인할 수 있다. 그렇다면 이 LinkedHashMap은 어떻게 순서를 보장할까?

LinkedHashMap의 javadoc 설명에서 힌트를 찾을 수 있었다.

> 예측 가능한 순서를 가진 Map 인터페이스의 Hash Table 및 연결 리스트 구현체이다. 이 구현체는 실행되는 모든 엔트리를 통해 이중 연결 리스트를 유지한다는 점에서 HashMap과 다르다.

LinkedHashMap은 다음과 같이 이중 연결 리스트를 가지고 있으며, 삽입되는 순서대로 연결 리스트에 저장하여 입력 순서를 보장한다.

![스크린샷 2020-12-16 오후 10 15 55](https://user-images.githubusercontent.com/56301069/102365628-1c5b0380-3ffb-11eb-93e4-8e1ad6993278.png)

## 2. 하나의 인스턴스만을 사용하는 객체는 유틸 클래스? 싱글톤? Map?

현재 진행하고 있는 프로젝트에선 Domain 패키지와 View 패키지에서 유효성 검사를 실행한다. 유효성 검사는 유효성 검사를 진행할 클래스의 객체를 생성한 후 해당 객체의 validate() 메소드를 사용한다. 또, 모든 유효성 검사 클래스는 Validator 클래스를 상속받는다.

```java
public LottoNumber inputBonusBall() {
    return new LottoNumber(printMessageAndInput(ASK_BONUS_BALL, new LottoNumberValidator()));
}

class LottoNumberValidator {

    ...

    public void validate() {

    ...

    }
}
```

유효성 검사를 하는 클래스는 모두 인스턴스 변수를 가지고 있지 않은 클래스이다.

그러므로 다른 객체의 행동에 의해 상태가 변하지 않는 클래스이다. 그러므로 새로운 인스턴스를 계속 생성하는 것은 기존 객체를 재사용하는 것보다 비효율적이다. 새로운 인스턴스를 생성하면 그만큼 메모리를 사용하기 때문이다.

그렇다면 새로운 인스턴스를 생성하지 않고 유효성 검사를 진행할 수 있는 방법은 무엇이 있을까? 생각을 해보았고, 유틸 클래스를 먼저 떠올리게 되었다.

### 2-1. 유틸 클래스

유틸 클래스는 속성이 없고, 정적 메소드만을 갖고 있는 클래스를 의미한다. 모든 메소드가 정적 메소드이기 때문에 생성자는 private이다.

다음은 유틸 클래스의 예시이다.

```java
class UtilClass {

    private UtilClass() {}

    public static void perform() {
        System.out.println("유틸클래스입니다.");
    }
}
```

유틸 클래스는 정적 메소드만을 갖고 있기 때문에 새로운 유틸클래스 인스턴스를 생성하지 않아도 된다.

이 유틸 클래스는 새로운 인스턴스를 생성하지 않는다는 조건은 만족하지만, Validator를 상속받는다는 점에서 문제가 발생한다.

유효성 검사 클래스는 최상위에 null 체크를 하는 클래스가 있으며, 자식들이 이를 상속받아 사용한다.

```java
public class Validator {
    
    static final String NULL_ERROR = "null 값을 입력하셨습니다.";

    public void validate(String input) {
        checkNull(input);
    }

    private void checkNull(String input) {
        if (input == null) {
            throw new IllegalArgumentException(NULL_ERROR);
        }
    }
}

public class NumberIndex extends Validator {

    public static final String NOT_NUMERIC_ERROR = "숫자만 입력해주세요.";

    @Override
    public void validate(String input) {
        super.validate(input);
        checkNumeric(input);
    }

    private void checkNumeric(String input) {
        try {
            Integer.parseInt(input);
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException(NOT_NUMERIC_ERROR);
        }
    }
}
```

문제는 자식 클래스가 메소드를 Override한다는 점에서 발생한다. 유효성 검사 클래스를 유틸 클래스로 만들기 위해선 메소드를 정적 메소드로 설정해야 하는데, 자바에서 정적 메소드는 오버라이드 하지 못하기 때문이다. 정적 메소드는 오버라이드 하지 못하는 이유는 [정적 메소드를 오버라이드 하지 못하는 이유](https://hsik0225.github.io/java/Static-Override/) 을 참조해주길 바란다.

그러면 새로운 인스턴스를 계속 생성하지 않고 하나의 인스턴스만을 생성 후 호출하면 되지 않을까? 라는 생각을 하였고, 이어 전에 배운 싱글톤을 떠올리게 되었다.

### 2-2. 싱글톤

싱글톤이란, 클래스가 하나의 인스턴스만을 생성하는 객체로, 해당 객체를 사용할 때마다 이미 생성된 객체를 반환하는 클래스를 의미한다.

싱글톤의 원리만을 알고 사용하는 여러 방법에 대해서는 잘 모르기 때문에 싱글톤에 대해서 공부하고 [디자인패턴 - 싱글톤 패턴]()에 정리하였다.

유효성 검사 클래스는 4개 정도 있으며, 3개의 클래스는 1개의 기본 유효성 검사 클래스를 상속받고 있다. 처음에 생각한 로직은 유효성 검사 인스턴스를 가져오는 정적 메소드를 최상위 클래스에 작성한 후 하위 클래스에서 상위의 메소드를 이용하여 싱글톤 객체를 생성하는 것이었다.

생각한 샘플 코드는 다음과 같다.

```java
public class Validator {

    private Validator() {}

    public static Validator getInstance(Class<? extends Validator> clazz)
            throws InstantiationException, IllegalAccessException {
        new ValidatorFactory(clazz);
        return ValidatorFactory.INSTANCE;
    }

    private static class ValidatorFactory {
        public static Validator INSTANCE = null;

        private ValidatorFactory(Class<? extends Validator> tClass)
                throws IllegalAccessException, InstantiationException {
            INSTANCE = tClass.newInstance(); // (1)
        }
    }
}
```

하지만 `Validator#getInstance()` 을 할 때마다 새로운 ValidatorFactory가 생성되며, ValidatorFactory.INSTANCE에 새로운 인스턴스가 생성되고 할당된다. 이것은 본래 생각한 싱글톤 패턴이 전혀 아닌 그저 객체 생성을 어렵게 작성한 코드에 불과한 것 같다.

그렇게 고민을 하던 도중 수업에서 포비님이 객체들을 Map에 담아서 캐싱하던 것이 떠올랐다.

### 2-3. HashMap을 이용한 싱글톤

다음 코드는 ConcurrentHashMap을 이용하여 Singleton들을 갖는 Map을 생성하는 코드이다.

```java
public class ValidatorPool {

    private static final Map<Class<? extends Validator>, Validator> validatorPool =
            new ConcurrentHashMap<>();

    public static Validator getInstance(Class<? extends Validator> validatorClass) {
        return validatorPool.computeIfAbsent(validatorClass, validator -> {
                    try {
                        return validator.newInstance();
                    } catch (InstantiationException | IllegalAccessException e) {
                        e.printStackTrace();
                        return null;
                    }
                }
        );
    }
}
```

만약 Map에 해당 Validator 클래스가 없으면 해당 Validator 클래스 인스턴스를 생성하고, 만약 있다면 해당 객체를 꺼내온다.

Validator 클래스들을 생성할 수 있으며, ConcurrentHashMap을 사용하기 때문에 Thread-safe한 싱글톤 객체를 생성할 수 있다.

다만 문제는 예외가 발생했을 시 일어난다.

위 코드는 `InstantiationException` ,`IllegalAccessException` 예외가 발생했을 경우 어떻게 할 수가 없다. 싱글톤 패턴은 하나의 인스턴스만을 생성하여 재사용하는 패턴이다. 즉, 하나의 인스턴스만 반횐되는 것을 생성하는 것을 보장해주는 패턴이다. 그런데 지금 보이는 것처럼 예외가 발생했을 때 반환해야할 값을 설정을 무엇으로 하면 좋을지 몰라 정확한 예외처리를 못하고 있다.

또, `return null` 을 하는 것은 하나의 인스턴스 반환을 보장해주는 싱글톤 패턴과 맞지 않다. 그렇다고 아무 Validator를 리턴해줄 수 있는 것도 아니다.

결국 예외 처리가 문제이므로, 다음과 같이 예외가 발생하지 않도록 new 연산자를 사용하여 객체를 생성하기로 하였다.

```java
public class ValidatorPool {

    private ValidatorPool() {}

    public static Validator getInstance(Class<? extends Validator> validator) {
        return ValidatorFactory.validatorPool.get(validator);
    }

    private static class ValidatorFactory {

        private static final Map<Class<? extends Validator>, Validator> validatorPool =
                new ConcurrentHashMap<>();

        static {
            validatorPool.put(IndexValidator.class, new IndexValidator());
            validatorPool.put(LineNameValidator.class, new LineNameValidator());
            validatorPool.put(StationNameValidator.class, new StationNameValidator());
            validatorPool.put(Validator.class, new Validator());
        }
    }
}
```

깔끔하지 못하지만 더 좋은 방법을 찾지 못해 이 싱글톤 패턴을 작용하였다.
