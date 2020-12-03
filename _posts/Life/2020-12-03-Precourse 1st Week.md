---
title : "[우테코] 프리코스 1주차 회고"

categories :

- Education

tags :

- Education

---
해당 글은 우테코 프리코스 1주차를 진행하면서 느낀 점과 고민했던 부분을 간단하게 정리한 글입니다.

## 1. 시작

11월 초, 우테코 3기를 모집한다는 글을 보았습니다. 국내 개발회사들 중 가장 유명한 회사들 중 하나인 **우아한 형제**에서 진행하는 교육 과정이기에 설레임을 안고 신청하게 되었습니다. 처음 있는 코딩테스트는
7문제 중 6문제를 풀었으며 며칠 후, 코딩테스트에 합격했다는 메일을 받았습니다.

<img width="1332" alt="합격" src="https://user-images.githubusercontent.com/56301069/101057101-05fe8200-35cf-11eb-8a5c-0de13cf365bf.png">

## 2. 숫자 야구 게임
프리코스 1주차 과제는 숫자 야구 게임이었습니다. 고등학교 때 자주했었고, 한 번 간단하게 구현해 본 경험이 있기 때문에 도메인 지식을 쉽게 받아들일 수 있었습니다.

하지만, 쉬운 도메인과는 다르게 프로그래밍을 하면서 몇 몇 부분에서 벽을 느끼고 고민을 했습니다.

## 2-1. 기능 목록 정리
코딩을 하기 전에 기능 목록을 작성해야 했습니다. 즉, Todo List를 작성하는 것입니다.

저는 처음부터 완벽한 설계를 하기 위해 노력했습니다. 기능 목록을 정리하는 것이 아닌 클래스 다이어그램을 만들었습니다.

처음 구현은 정말 쉬웠습니다. 클래스 다이어그램을 미리 만들어놓았으니 한컴타자연습과 같이 따라 치기만 하면 되었습니다. 하지만, 
문제는 설계 중 생각지 못한 부분을 만났을 때 발생했습니다. 설계 중 고려하지 못한 부분을 처리함에 따라 기존 정의한 클래스 설계는 
무너지기 시작했고, 점점 로직은 복잡해지기 시작했습니다. 클래스 간의 관계가 복잡해짐에 따라 코드는 더 읽기 어려워졌고, 객체의 
책임과 역할이 섞이게 되어 나중엔 **이 객체는 뭐하는 객체이지?**라는 생각이 스스로 들 정도로 스파게티 코드가 되었습니다.

처음부터 완벽한 설계를 짜기 위해 노력하기 보다는 프로그래밍을 하며, 기존 설계를 보완해 나가는 것이 좋겠다는 것을 느꼈습니다.
## 2-2. 단위 테스트 이름
기존 단위 테스트 이름은 다음과 깉이 진행해야할 테스트의 이름과 맞게 영어로 작명하였습니다.
```java
@Test
@DisplayName("3스트라이크일 경우 isAnswer() true 반환 테스트")
public void hasThreeStrikeAnswerTest() {

    // given
    scoreBoard = new ScoreBoard(3, 0);

    // when, then
    assertTrue(scoreBoard.isAnswer());
}
```

하지만, 다음과 같은 단위 테스트 이름은 메소드명을 봐서는 알 수 없다는 단점이 있습니다.
물론, @DisplayName이 이를 정확히 무슨 테스트를 하는지 한글로 알려주긴 하지만,
메소드 명으로도 해당 단위 테스트가 무엇을 하는지 알릴 필요가 있다고 느꼈습니다.

좋은 테스트 이름을 검색하던 중 [이 문서](https://docs.microsoft.com/ko-kr/dotnet/core/testing/unit-testing-best-practices) 에서 단위 테스트
이름의 작명방법을 알게 되어 앞으로는 해당 명명 방법을 따를 생각입니다.

문서에서의 테스트 이름 지정 방식은 다음과 같습니다.
### 테스트 이름 지정
테스트의 이름은 다음 세 부분으로 구성되어야 합니다.

- 테스트할 메서드의 이름입니다.
- 테스트 중인 시나리오입니다.
- 시나리오에서 호출될 때 예상되는 동작입니다.

테스트는 단순히 코드가 작동하는지 확인하는 것 이상이며, 문서도 제공합니다. 
단위 테스트 도구 모음을 살펴봄으로써 코드 자체를 조회하지 않고도 코드의 동작을 유추할 수 있습니다. 
또한 테스트가 실패하는 경우 기대치를 충족하지 못하는 시나리오를 정확히 확인할 수 있습니다.

- Bad
```java
public void Test_Single() {
    var stringCalculator = new StringCalculator();

    var actual = stringCalculator.Add("0");

    Assert.Equal(0, actual);
}
```

- Better
```java
public void Add_SingleNumber_ReturnsSameNumber() {
    var stringCalculator = new StringCalculator();

    var actual = stringCalculator.Add("0");

    Assert.Equal(0, actual);
}
```

[인기있는 Java Unit Test 네이밍 규칙](https://hilucky.tistory.com/216)

### 테스트 정렬
정렬, 동작, 어설션 은 단위 테스트 시 일반적인 패턴입니다. 
이름에서 알 수 있듯이 세 가지 주요 작업으로 구성됩니다.
- 개체를 정렬 하고 필요에 따라 만들고 설정합니다.
- 개체의 동작 입니다.
- 특정 항목이 예상대로라고 assert 합니다.

테스트할 항목을 정렬 및 assert 단계에서 명확하게 구분합니다.
가독성은 테스트를 작성할 때 가장 중요한 측면 중 하나입니다. 
테스트 내에서 이러한 작업을 각각 구분하면 코드를 호출하는 데 필요한 종속성, 코드 호출 방법 및 어설션하려는 대상이 명확하게 강조 표시할 수 있습니다. 
몇 가지 단계를 결합하여 테스트의 크기를 줄일 수 있지만, 기본적인 목표는 가능한 한 테스트를 최대한으로 읽을 수 있도록 하는 것입니다.
- Bad
```java
public void Add_EmptyString_ReturnsZero() {
    // Arrange
    var stringCalculator = new StringCalculator();

    // Assert
    Assert.Equal(0, stringCalculator.Add(""));
}
```

- Better
```java
public void Add_EmptyString_ReturnsZero() {
    // Arrange
    var stringCalculator = new StringCalculator();

    // Act
    var actual = stringCalculator.Add("");

    // Assert
    Assert.Equal(0, actual);
}
```


## 2-3. 재귀로 인한 stackoverflow
다음과 같이 게임을 한 후, 재시작을 원한다면 재귀호출로 다시 게임을 실행하는 코드가 있습니다.
```java
public void run() {
    while (!게임 끝) {
        게임();
    }
    
    if (재시작을 원한다면) {
        run();
    }
}
```

이 코드에는 한 가지 큰 문제가 있습니다. 바로 사용자가 이 게임을 너무 좋아해서 게임을 끝내고 싶지 않은 경우, 
즉 재시작을 무한히 원한다면, run() 메소드가 닫히지 않아 스택에 run() 메소드가 계속 쌓인다는 점입니다.

이렇게 쌓이게 된 run() 메소드는 결국 **stackoverflow** 를 발생시킵니다.

이와 같은 문제를 해결하기 위한 방법은 run() 메소드 안의 if() 문을 while문으로 변경하는 것입니다.
위 코드는 다음과 같이 바꿀 수 있습니다.
```java
public void run() {
    boolean wantsRestart = true;
    while (wantsRestart) {
        while (!게임 끝) {
            게임();
        }
        
        wantsRestart = 사용자가_재시작_입력();
    }
} 
```

위의 코드와 같이 수정하면 안의 메소드들은 실행 후 닫히게 되고 스택에 메소드들이 쌓이지 않게 됩니다.

해당 글에 대한 더 자세한 설명은 [다음 글](https://woowacourse.github.io/javable/2020-04-30/iteration_vs_recursion)을 참고해주세요

## 2-4. 유효성 검사는 어디서 해야할까
기존에는 사용자 입력에 대한 유효성 검사를 항상 InputView 에서만 했었다. 이건 사용자 입력에 대해서만 
유효성 검사를 하는 경우에는 분명 유효하다고 생각됩니다. 하지만, 문제는 사용자 입력이 아닌 다른 부분에서 유효성 검사를 해야하는 경우에서 발생합니다.

예를 들어, 주사위는 1~6까지의 숫자만을 가질 수 있습니다.
주사위가 사용자 입력과 관련이 없다면, 주사위를 생성할 때, 주사위가 가진 눈의 유효성 검사는 어디서 진행해야할까요?
모든 유효성 검사는 View에서 했었으므로, View에서 해야할까요?
이 주사위 객체는 사용자로부터 입력과 전혀 상관이 없으므로, 주사위의 생성자에서 유효성 검사를 해야합니다.
예시 코드는 다음과 같습니다.
```java
public class Dice {
    int number;
    
    public Dice(int number) {
        if (number < 1 || number > 6) {
            throw new IllegalArgumentException();
        }
        
        this.number = number;
    }
}
```

이렇게 되면 고민에 빠지게 됩니다. 유효성 검사를 View 만이 아닌 Domain에서도 하게 되기 때문입니다.
즉, 유효성 검사가 모든 레이어 전반에 걸쳐 진행되게 됩니다. 또, View에서 했던 검사를 Domain에서 다시 하게 될 수도 있습니다.

그렇다면 유효성 검사를 View에서 하지 않고 Domain에서만 진행하면 어떨까요?
분명 유효성 검사를 한 번만 진행한다면 된다는 장점이 있습니다.
하지만 2가지 문제점을 갖고 있습니다.
1. 유효성 검사가 모든 도메인에 흩어져 있습니다. 유효성 검사라는 하나의 클래스의 역할이 모든 객체에 흩어져 있게 되니 도메인 객체의 
복잡도가 늘어납니다
2. 생성 후 예외가 발생하였을 때 캐치하기 어렵습니다. 예를 들어, 위의 코드에서 주사위 눈과 사용자의 이름을 사용자로부터 입력받는다고 합시다. 
예외가 발생했을 시 사용자로부터 입력을 다시 받는다면 다음과 같은 코드가 될 것입니다.
   

```java
private User createUser() {
    Dice dice;
    User user;
    
    try {
        dice = new Dice(diceNumber);
        user = new User(userName, dice);
    } catch (IllegalArgumentException e) {
        return createUser();
    }

    return user;
}
```
서 혹시 모를 예외를 위해 try-catch 문을 적용해주야 할 것입니다. 이는 예외가 발생하는 
도메인을 생성하는 모든 코드에 적용되야 할 것입니다. 이는 코드의 길이가 늘어나고 가독성이 떨어지는 결과를 초래합니다.

그러므로 도메인에서 혹은 View에서만 유효성 검사를 하는 것이 아닌, 유효성 검사를 두 번 혹은 그 이상 하더라도 View, Domain 두 군데 모두에서 유효성 검사를 하는 것이 좋다고 생각합니다.
## 3. 다른 참가자들
1주차 프리코스 결과물은 해당 프리코스 깃허브에 풀리퀘를 보내는 형식이기에 다른 참가자들의 풀리퀘를 보며 소스들을 볼 수 있습니다. 
코드들을 살펴보니 앞서 코딩테스트를 진행했기 때문인지 참가자들의 수준이 높았습니다.
120명 중 약 50명 정도만을 선발하는 만큼 더 열심히 해야겠습니다.

## Reference
- https://xlffm3.github.io/java/etc/Woowacourse_precourse_baseball/
- [인기있는 Java Unit Test 네이밍 규칙](https://hilucky.tistory.com/216)
- [Microsoft Unit Test](https://docs.microsoft.com/ko-kr/dotnet/core/testing/unit-testing-best-practices)
- [반복문(iteration) vs 재귀(recursion)](ttps://woowacourse.github.io/javable/2020-04-30/iteration_vs_recursion)
