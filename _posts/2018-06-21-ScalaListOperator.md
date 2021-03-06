---
layout: post
title:  Scala - 타입의 계보 List와 List 연산자 정리
categories: scala
description: 악명이 높은지는 모르겠지만 스칼라의 List 연산자는 악명이 높아야 합니다. 매우 더럽게 생겼고 복잡하고 연산자 오버로딩이 되는 메소드라서 `.`으로 함수 호출하듯 사용하면 결과가 다릅니다. 그래서 정리합니다.
tags:
- scala
- 스칼라
- Collections
- List
- 연산자
---

# 서론

악명이 높은지는 모르겠지만 스칼라의 List 연산자는 악명이 높아야 합니다. 매우 더럽게 생겼고 복잡하고 연산자 오버로딩이 되는 메소드라서 `.`으로 함수 호출하듯 사용하면
또 결과가 바뀝니다. 그래서 정리합니다.

본론으로 들어가기 전에 이 글의 예제는 scala의 공식문서를 많이 발취했음을 밝힙니다.

https://www.scala-lang.org/api/current/scala/collection/immutable/List.html

---

# List 엄청 간단한 소개

`List`에도 여러 종류가 있지만 일반적으로 각각의 노드들로 이루어져 있고 노드가 서로 연결된 시퀀스입니다.

![LinkedList](http://www.epnc.co.kr/news/photo/201109/45353_42547_3.jpg)

출처: http://www.epnc.co.kr/news/articleView.html?idxno=45353

C를 처음 배울 때 많이 구현하는 자료형으로 배열처럼 많이 사용하지만 다른 것 입니다. 리스트는 배열과 다르게 메모리만 충분하다면 계속 동적으로 노드를 이어 붙일 수 있습니다

이 글의 주 독자는 이미 리스트에 대해 어느정도 알고 있을 것이기에 여기까지 설명하고 넘어가겠습니다.

(리스트를 모르면 스칼라 이전에 자료구조와 Java, C 같은 언어를 먼저 공부하세요 List 모르면 코딩 못합니다.)

---

# 연산자, 메소드 정리

## ++

`++` 은 리스트 두 개를 합치는 메소드입니다.    

{% highlight scala %}

    scala> val a = List(1, 2)
    a: List[Int] = List(1, 2)
    
    scala> val b = List(3, 4)
    b: List[Int] = List(3, 4)
    
    scala> a ++ b
    res0: List[Int] = List(1, 2, 3, 4)

{% endhighlight %}

 
## ::
 
`::는` 값 뒤에 리스트를 연결할 때 사용합니다. 주의할 점으로 `.::`로 호출하면 결과가 반대로 됩니다.
 
{% highlight scala %}

    1 :: List(2, 3) = List(2, 3).::(1) = List(1, 2, 3)

{% endhighlight %}
    
주로 패턴매치에서 많이 사용합니다.
  
{% highlight scala %}
  
        // for, while 안 쓰고 재귀함수로 모든 노드들을 특정 문자열로 연결하는 함수
        @tailrec
        def join(str: String, items: List[String])(resultStr: String = ""): String = {
          items match {
            case x :: Nil => {join(str, Nil)(resultStr + x)}
            case x :: xs => {join(str, xs)(resultStr + x + str)}
            case Nil => {resultStr}
          }
        }
                                                             
       
       --- 실행 결과
        
        scala> val data = List("서울", "대전", "대구", "부산", "광주")
        data: List[String] = List(서울, 대전, 대구, 부산, 광주)           
                                                                
        scala> join(", ", data)()                               
        res6: String = 서울, 대전, 대구, 부산, 광주   
                            
{% endhighlight %}
        
 
## :::
 
`:::`도 `::`와 비슷하지만 다른점은 `값 + 리스트` 연산이 아니라 `리스트 + 리스트` 연산을 합니다
 
{% highlight scala %}
       List(1, 2) ::: List(3, 4) = List(3, 4).:::(List(1, 2)) = List(1, 2, 3, 4)
{% endhighlight %}
 
## .:: .::: 메소드로 호출하면?

위의 두가지 `::`, `:::` 연산자는 참 신기하게 `.::`, `.:::`와 반대로 호출합니다.

예를 들자면 1,2 3,4 가 되야될게 `.`으로 호출하면 그 반대로 3,4,1,2 순서로 들어갑니다. 주의하세요 ㅠㅠ
 
## 의미 없는 연산자들

`++:`

`+:`
 
## 기타 함수들

* `take` : 인자 n 번째를 가저온다
* `tail` : 리스트 마지막 값을 가저온다
* `head` : 리스트 제일 첫 번째 값을 가져온다
* `sum` : 합계
* `indexOf` : 인자의 값이 몇 번째에 들어있는지 int로 돌려준다. 
* `isEmpty` : 비어있면 true 아니면 flase

## 고차 함수들
 
리스트의 수 많은 고차 함수들에 대해서는 전편인 고차 함수편을 참고해주세요 20개에 가까운 고차함수들을 소개했고
그 중 많이 쓰이는 열 개 이상은 쓸만한 예제와 설명이 달려있습니다.

---

# 뒷담

스칼라의 리스트는 뭔 크리링 머리의 점 같은 연산자로 계산합니다. 괜히 스칼라 진입 장벽을 높이는 안 좋은 원인 중 하나라고 생각합니다

이 글을 읽으면서 알아둘 점으로는 이 글의 리스트는 수정이 불가능한 리스트입니다. 특정 값을 합친 새로운 리스트를 리턴하지 기존의 리스트를 수정하는건 아니란 것을 알아두세요
 