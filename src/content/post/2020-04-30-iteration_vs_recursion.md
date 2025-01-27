---
layout: post
title: "반복문(iteration) vs 재귀(recursion)"
author: [2기_보스독]
tags: ["clean-code", "recursion"]
date: "2020-04-30T12:00:00.000Z"
draft: false
image: ../teaser/recursive.png
---

## 반복문과 재귀함수

프로그램은 반복되는 작업을 수행하도록 설계된다. 따라서 반복을 구현하는 로직은 필수적이고 프로그래밍 언어마다 for, while 같은 기본적인 반복 제어문을 지원하고 있다.

반복되는 작업은 기본 제어문을 통해서 뿐만 아니라 재귀함수로도 구현할 수 있다.

재귀함수는 복잡한 문제를 단순화해서 풀 수 있는 알고리즘으로 많이 알려져 있는데, 원래 정의는 **하나의 함수가 자신을 다시 호출하여 반복되는 작업을 수행하는 함수**를 말한다.

반복문과 재귀함수는 서로 반복을 수행하는 구조는 다르지만,  재귀 함수로 작성할 수 있다면 반복문으로도 작성할 수 있고 그 역도 성립한다.



## 어떤 방식이 더 좋은가? 

아래 코드는 자동차 경주게임을 구현한 일부 예시 코드이다. 사용자로부터의 입력이 올바른지 검증하고 그렇지 않았을 경우 계속 입력을 재요청하는 역할을 하고 있다. 이 함수를 반복문과 재귀함수를 통해 구현한 코드로 각 방식의 장단점을 알아보자.

먼저 재귀를 사용했을 경우다.

``` java
// 재귀
public static List<String> inputCarNames() {
    System.out.println("경주할 자동차를 입력하세요. (이름은 쉼표(,) 기준으로 구분");
    try {
        return scanner.nextLine().split(",");
    catch (Exception e) {
        System.out.println(e.getMessage() + "다시 입력해주세요!");
        return inputCarNames();
    }
}
```

잘못된 입력 형식으로 함수 내부에서 예외가 발생할 경우 재귀 호출을 통해 입력을 재요청하고 있다. 특별한 로직없이 내부가 단순하게 구현되어 있어 구조적으로도 이해가 쉽다. 

그렇다면 이제 같은 내용을 반복문으로 구현해보면 어떨까.

``` java
// 반복문
public static List<String> inputCarNames() {
    System.out.println("경주할 자동차를 입력하세요. (이름은 쉼표(,) 기준으로 구분");
    String carNames;
    boolean result;
    do {
        carNames = scanner.nextLine();
        try {
            result = InputValidator.validateForm(carNames);
        } catch (Exception e) {
            result = false;
            System.out.println(e.getMessage() + "다시 입력해주세요!");
        }
    } while (!result);
    return Arrays.asList(carNames.split(","));
}
```

한 눈에 보아도 함수의 길이가 길어졌고, carNames, result 같이 내부에서 사용하는 변수도 많아졌다. 무엇보다도 코드만 보아서는 이 함수가 어떤 역할을 하는지 직관적으로 알기 어렵다.

이처럼 반복문보다 재귀를 사용하면 분명 코드의 가독성이 좋아진다. 사용하는 변수의 개수도 줄어들고 구현이 반복문보다 간단하기 때문에 어렵고 복잡한 문제도 빠르고 단순하게 접근할 수 있다. 이러한 장점을 가진 재귀 함수는 개발자가 지향해야 할 **읽기 좋은 코드**와 가깝다고 할 수 있다.  

그렇다면 반복문 보다 재귀 함수로 접근하는게 무조건 옳은 것일까?

꼭 그렇다고 할 수는 없다. 재귀 함수는 장점에 버금가는 치명적인 단점을 가지고 있기 때문이다 .

재귀 함수는 기본적으로 스택 메모리를 사용하는데 재귀의 깊이가 깊어졌을 때, **stack overflow**가 발생하면서 프로그램이 비정상적으로 종료 될 수 있다.  언제나 안전한 프로그램을 개발해야 하는 입장에서, 충분히 에러가 발생할 수 있는 여지를 남겨놓는 것은 바람직하지 않다. 

또한 함수가 호출되고 종료될 때 **스택 프레임을 구성하고 해제하는 과정**에서 반복문보다 오버헤드가 들기 때문에 속도도 훨씬 느려지게 된다. 

> #### stack overflow란?
>
> 함수를 호출하면 함수의 매개변수, 지역변수, 리턴 값, 그리고 함수 종료 후 돌아가는 위치가 스택 메모리에 함께 저장된다.
>
> 재귀함수를 쓰게되면, 함수를 반복적으로 호출하므로, 스택 메모리에 콜 스택이 쌓이게 된다. 함수를 호출하는 횟수가 많아진다면 스택 메모리를 초과하여 stack overflow가 발생할 수 있다.
>
> 그러나 일반적인 반복문을 사용하면 지역 변수들이 호출될 때 한번만 할당되기 때문에 그러한 비효율이 발생하지 않는다.

> #### Overhead란?
>
> 오버헤드(Overhead)란 어떤 처리를 하기 위해 들어가는 간접적인 처리 시간 · 메모리 등을 말한다.

 

## 꼬리 재귀 (tail call recursion) 

재귀함수의 이러한 단점을 보완하고자 꼬리 재귀라는 기법을 사용할 수 있다.

단, 컴파일러가 꼬리 재귀 최적화를 지원해야만 실질적으로 단점을 보완할 수 있다. 

코드를 통해 살펴보면 일반 재귀 함수와 꼬리재귀 함수는 각각 다음과 같은 형태를 가진다. 

``` java
public int recursive(int n) {
  if (n == 1) {
    return 1;
  }
  return n + recursive(n - 1);
}

public int tailRecursive(int n, int acc) {
  if (n == 1) {
    return acc;
  }
  return tailRecursive(n - 1, n + acc);
}
```

두 함수의 큰 차이는 다음 호출을 위한 파라미터의 연산이 어디서 일어나는가 이다. 즉, 다시말해 return 문에 연산이 있느냐 없느냐 차이라고 볼 수 있다. 

함수가 리턴된 후에 아무 작업도 하지 않도록 하는 것을 `꼬리 호출(tail call)`이라 하고, 이런 구조를 `꼬리 재귀(tail recursion)`라고 하며, 이런 함수를 `꼬리재귀함수(tail recursion function)`라고 한다.

꼬리 재귀는 연산이 return 문 이전에 이루어지고 다음 함수 호출 시 파라미터를 통해 필요한 연산의 결과를 전달한다. 함수가 호출되는 시점에 컴파일러는 꼬리재귀를 최적화하게 되는데, **이 과정에서 꼬리 재귀는 반복문으로 변경된다.** 기존의 재귀 형태를 최적화 가능한 형태로 변경하면서 컴파일시 반복문으로 해석될 수 있도록 만들면 기존에 문제였던 메모리와 성능에 대한 문제를 해결할 수 있다. 

``` java
// 컴파일러에 의해 꼬리 재귀가 최적화된 모습
public int recursive(int n, int acc) {
  int value = 0;
 
  do {
    if (n == 1) 
      return value + n;
    value = value + n;
    n = n - 1;
  } while(true);
}
```



## 결론 

재귀를 사용하여 반복 구조를 구현하면 복잡한 문제도 단순한 로직으로 해결하고, 함수 내에서 사용하는 변수의 개수와 코드의 길이가 줄어들기 때문에 읽기 좋은 코드가 될 수 있다.

하지만 스택메모리를 사용하는 재귀함수는 콜스택이 쌓이게 되면 **stack overflow** 를 유발할 수 있고 속도나 성능이 반복문 보다 현저히 떨어지는 것은 분명하다.

성능이 중요했던 과거에는 반복문으로 구현하는게 당연했겠지만, 하드웨어의 발전으로 소프트웨어의 자체 성능에 대한 중요도가 낮아졌기 때문에 오히려 협업이 강조되고 있는 요즘에는 코드의 가독성도 충분히 고려할 필요가 있다.

게다가 컴파일러가 꼬리 재귀의 최적화를 지원해준다면 앞서 살펴본 재귀의 성능 문제도 해결할 수 있으니 `가독성`과 `성능` 모두를 얻을 수 있을 것이다. 

물론 그렇다고 무작정 재귀함수의 **stack overflow**를 방심할 수는 없다. 이왕이면 재귀 함수도 재귀 깊이가 예측 가능한 경우 위주로 사용하는 것이 더 안전하고 현명할 것이다. 

---

#### 참고 자료

[재귀함수와 반복문의 차이](https://wonillism.github.io/algorithm/Algorithm-recursion-iteration/)

[꼬리 재귀 최적화](https://bozeury.tistory.com/entry/%EA%BC%AC%EB%A6%AC-%EC%9E%AC%EA%B7%80-%EC%B5%9C%EC%A0%81%ED%99%94Tail-Recursion](https://bozeury.tistory.com/entry/꼬리-재귀-최적화Tail-Recursion))




