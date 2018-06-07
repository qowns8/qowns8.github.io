---
layout: post
title: Scala - 타입의 계보 null 편
categories: scala
description: Scala의 Nil, Null, None, Option, Nothing, null 등에 대해 알아봅시다. 
tags:
- scala
- 스칼라
---

### 서론

스칼라에서 빈 값을 표현하는 방법은 정말 많습니다. 그래서 만들었습니다 scala 신들의 계.. 타입의 계보 null편

### Null

우리가 생각하는 그 `null`이 아니라 Null trait 입니다. 주의해서 사용이 필요합니다. ㅠㅠ

### Nothing
`Nothing`은 빈 값을 나타내는 `trait`입니다. 컬랙션에서 `Nil`, `None`등 을 구현할때 제네릭으로 많이 사용됩니다.

### Nil

`Nil`은 빈 리스트를 의미합니다. 패턴매치할때 종종 사용합니다.

    List() == Nil == List.empty
    
    // 리스트의 마지막을 찾음
    def findLastItem(item: List[T]) => { item match {
        case x :: xs => something(xs)
        case x :: Nil => x
        case Nil => Nil
      }
    }

### None

`None`은 빈 Option을 말합니다. 스칼라에서 get 함수로 값을 꺼낼때 `Option`으로 감싸져서 나오는 것을 많이 보실 수 있습니다. 
이런 방식을 Optional 이라고 하는데 최근 java와 코틀린에서도 볼 수 있습니다. 장점으로는 `null`을 최대한 피할 수 있습니다.
Scala에서 `Option[T]`은 `Try[T]`, `Future[T]`와 같이 모나드의 대표적인 예시로 들수도 있습니다. 3개다 모나드의 3가지 원칙을 제껴두고 생각하면
와일트카드 제네릭 타입을 감싸고 있죠... 모나드의 이야기는 여기까지만 하고 예시를 살펴봅시다.

    val c = Map("미국" -> "워싱턴", "그리스", "아테네", "로마" -> "로마")
    val usa = c.get("미국") // Option[String]
    val un = c.get("유엔") // Option[String]
    
    usa.get // 워싱턴
    un.get // error
    
    c.getOrElse(sometihng, "없는 나라입니다.")
    //key가 있으면 String으로 된 value 값 없으면 ' "없는 나라입니다" '를 돌려줌

`None`의 구현을 보시면 다음과 같습니다.

    trait Option[E]
    case class Some[E](value: E) extends Option[E]
    case object None extends Option[Nothing]
    
위에 소스를 보면 다음 3 개가 똑같다는걸 알 수 있습니다.

    Some() == Option() == None

그러면 빈 `Option`을 만들때 빼고 어디서 `None`을 쓸까요? `Nil`과 마찬가지로 주로 패턴매치에서 많이 사용합니다.

    someOpt match {
        case x : Option[String] => println(x.get)
        case x: None => println("빈 값")
        case _=> println("nonString")
    }
    
### Unit

`Unit`은 c언어의 `void`와 비슷힙니다 아무것도 리턴하지 않는 메소드의 리턴 타입으로 사용합니다.

    def print(data: Any): Unit = print(data.toString)

### null

우리가 아는 그 자바의 `null`을 객체로 만들었습니다.



### 참고 자료

scala util
http://hamait.tistory.com/652?category=79134
https://stackoverflow.com/questions/16173477/usages-of-null-nothing-unit-in-scala
