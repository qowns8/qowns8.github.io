---
layout: post
title:  Scala - 스칼라의 고차 함수들 정리 (flatMap, filter, sort, groupBy)
categories: scala
description: Scala의 Iterators와 각종 컬랙션에 쓰는 고차 함수들 map, flatMap, filter, filterNot, groupBy, sorted, size 등의 메소드에 대해 사용법, 예시, 차이점 등을 전부 정리 했습니다.
tags:
- scala
- 스칼라
- 컬랙션
- Iterator
- Collections
- 고차함수
- flatMap
- 함수형
---

# 서론

스칼라의 `List`, `Map` 등을 쓸때 자주 `map`, `flatMap`, `filter`등의 메소드를 호출 합니다. 문제는 스칼라에 이런 것들이 너무 많아서 어렵습니다. 이게 과연 지원 되는건가 아니면 내가 구현해야 되는건가? 고민되는건 일단... 스칼라에서 다 메소드로 있습니다.
그래서 나중에 이게 있었구나 하고 깨닫고 내가 헛수고 했다며 다시 짜는 코딩하게 됩니다. 이런 문제를 방지하기 위해 어느 정도 많이 쓰이는 함수들을 정리할 필요성을 느꼈습니다

---

# map, flatMap, filter, filterNot 등은 어디서 왔는가?

컬랙션을 순회하면서 필터링을 하거나 정렬을 하는 등의 그 많은 메소드는 어디서 왔을까요?

![type_system](https://camo.githubusercontent.com/7ec1a4f561d68332fcf0b4f7dbcd971ef706af37/687474703a2f2f6c6d617a792e766572726563682e6e65742f77702d636f6e74656e742f75706c6f6164732f323031312f30322f7363616c615f747970655f6869657261726368792e706e67)

일단 사진을 보시면 다음과 같이 각종 컬랙션들이 `Iterators`를 상속받는 모습을 볼수 있습니다.

`Iterators`는 map과 flatMap 함수가 구현되어 있습니다. 제가 `Map` 을 타고 들어가본 결과 `Map`은 `Traversable` 과 `Iterators`에서 우릭 사용하는 각종 `flatMap,` `filter`, `sorted` 등의 메소드를 가져옵니다.

어째튼 이런 트레잇을 컬랙션들이 공통적으로 상속받고 있었습니다.

---

# 고차함수 (higher-order function)

한줄설명: f: A => B 같은 인자를 함수가 가진다.

함수를 인자로 가진 함수를 고차 함수입니다. 대표적인 예시로 `filter`나 `map`등이 있습니다.
예시:
    
        def highFunc(f: int => Boolean)
        
---
    
# 각종 고차 함수 설명과 사용법

## map

`map`은 고차함수의 대표격이자 함수형의 존재 이유가 아닐까 싶을 정도로 만능칼 처럼 사용됩니다. 사실 `map`만 가지고 노력만 하면 아래의 거의 모든 함수를 구현할수 있습니다.
`map`은 단순하게 인자로 받은 함수를 컬랙션 순회하면서 각각 실행 시키는 일을 합니다.

다음 예시는 실용적이지 않은 그냥 예시입니다.

예시

        case class Word(w: Any, word_type: String)
        val words = List(
          Word("프란시스 드레이크", "명사"),
          Word("캡틴 쿡", "명사"),
          Word("검은 수염", "명사"),
          Word("움직이다", "동사"),
          Word(5, "수사"),
          Word("반짝이는", "형용사")
        )
    
        val pie = words.map(word => {
          if (word.w.isInstanceOf[String] && word.word_type == "명사") {
            ("해적", word.w)
          }
        })
    
        println(pie) // 결과: List((해적,프란시스 드레이크), (해적,캡틴 쿡), (해적,검은 수염), (), (), ())

## flatMap

`flatMap`은 `flatten` + `map` 함수입니다. 한가지 특이한 점은 `flatMap`, `map`은 `for`문과 동일해서 서로 대체가 가능합니다. 지금 당장 알 필요는 없지만 이는 모나드를 이해하는데 중요한 요소입니다.

다음 예시는 `map`을 한 다음 그 값들을 꺼내서 하나의 컬랙션으로 만들어줍니다.

예시

        val list = List(Some(1), Some("2"), None, Some(4), None)
        val numbers = list.flatMap(item => Some(item.getOrElse("0").toString.toInt))
        println(numbers) // 결과 List(1, 2, 0, 4, 0)


## reduceRight

`reduceRight`는 이전과 현재 item으로 된 함수를 받고 한번 순회 할때 마다 이전의 순회한 item을 버리면서 함수를 호출합니다. 주로 리스트로 된 문자열 집합을 다 더할때 많이 사용합니다.

사용예시

        val queryStr = List(
          ("id" -> "38"),
          ("sort" -> "false"),
          ("type" -> "json")
        ).map(item => item._1 ++ "=" ++ item._2).reduceRight(_ ++ "&" ++ _)
    
        val domain = "http://api?"
    
        println(domain + queryStr) // 결과 http://api?id=38&sort=false&type=json

## reducetLeft

`reduceLeft`는 `reduceRight`와 똑같지만 순회하는 방향만 정반대로 다릅니다.

사용예시

        val queryStr = List(
          ("id" -> "38"),
          ("sort" -> "false"),
          ("type" -> "json")
        ).map(item => item._1 ++ "=" ++ item._2).reduceLeft(_ ++ "&" ++ _)
    
        val domain = "http://api?"
    
        println(domain + queryStr) // 결과 http://api?id=38&sort=false&type=json

## zipWithIndex

`zipWitIndex`는 id값을 붙여줍니다.

다음 예시는 scala 공식 doc에서 가져왔습니다.
    
사용에시
   
    List("a", "b", "c").zipWithIndex = List(("a", 0), ("b", 1), ("c", 2))

## filter

`filter` 함수는 인자로 받은 함수를 기준으로 함수에 해당되는 값들을 돌려줍니다.

다음 예시는 유저 타입이 2 이하인 친구들만 걸러내는 예시입니다.

예시


    case class User(id: Long, name: String, user_type: Int)

    val users = List (
      User(0, "파인만", 2),
      User(0, "알랙산더", 1),
      User(0, "라마누잔", 1),
      User(0, "하스켈", 1),
      User(0, "아인슈타인", 3),
      User(0, "뉴턴", 3)
    )

    println(users.filter(_.user_type < 2))
    // 결과 List(User(0,알랙산더,1), User(0,라마누잔,1), User(0,하스켈,1))

## filterNot

`filterNot`은 인자로 준 함수에 해당되지 않는 값들을 돌려줍니다.

    println(users.filterNot(_.user_type < 2))
    // 결과 List(User(0,파인만,2), User(0,아인슈타인,3), User(0,뉴턴,3))

## 각종 sort 함수

list를 정렬하는데 무려 3가지의 방법이 있습니다. 그 차이점과 사용방법을 쉽고 명확하게 설명합니다.

### sorted

`sorted` 함수는 단순 숫자나, 문자열로 정렬을 하고 싶을 때 사용합니다.

사용예시 

        val needSort1 = List(3, 4, 1, 7)
        val needSort2 = List("ba","ab", "ca", "cb")
        println(needSort1.sorted) //결과 List(1, 3, 4, 7)
        println(needSort2.sorted) //결과 List(ab, ba, ca, cb)


        val needSort3 = List(123,"ab", "ca", "cb")
        println(needSort3.sorted) // 결과: error

    
### sortBy

`sortBy`는 정렬 기준을 정해서 정렬할 수 있습니다. 예를 들면 문장길이나 리스트 안에 클래스에서 정렬 기준이 되길 원하는 맴버를 정하거나 할수 있습니다.
밑의 예시는 사각형 클래스의 넓이로 사각형들을 정렬한 예시 입니다.

사용에시

        case class square(width: Long, height: Long)
        val needSort1 = List(
          square(4, 5), // 20
          square(3, 6), // 18
          square(19, 1), // 19
          square(1, 1) // 1
        )
    
        //println(needSort1.sorted) error
        println(needSort1.sortBy(s => s.height * s.width)) // 결과 List(square(1,1), square(3,6), square(19,1), square(4,5))

### sortWith

`sortWith`는 정렬 기준을 직접 만들수 있습니다. 인자로 함수를 받는데 인자로 넘겨준 함수 대로 정렬해 줍니다.
밑의 예시에서는 방금전에 에시로 들은 사각형 리스트를 높이에 따라 오름차순 정렬을 하는 코드입니다.

    
        println(needSort1.sortWith((s1, s2) => {
          s1.height < s2.height
        }))
        // 결과 List(square(19,1), square(1,1), square(4,5), square(3,6))

### groupBy

`groupBy`는 인자로 준 함수를 기준으로 묶습니다. mysql의 `groupBy`와 비슷합니다.

        val weathers = List(
          Map("date" -> "201906012", "local" -> "서울", "weather" -> "맑음"),
          Map("date" -> "201906012", "local" -> "대전", "weather" -> "맑음"),
          Map("date" -> "201906013", "local" -> "대전", "weather" -> "비"),
          Map("date" -> "201906013", "local" -> "부산", "weather" -> "맑음")
        )
        val gorup = weathers.groupBy(weather => weather.getOrElse("date",""))
        println(gorup)
        
        // 결과
        Map(
             201906013 -> List(
                Map(date -> 201906013, local -> 대전, weather -> 비),
                Map(date -> 201906013, local -> 부산, weather -> 맑음)
             ), 
             201906012 -> List(
                Map(date -> 201906012, local -> 서울, weather -> 맑음),
                Map(date -> 201906012, local -> 대전, weather -> 맑음)
             )
        )
## 기타 컬랙션 소개

* `foreach` : 반환 값이 없는 `map` 함수
* `zip` : `zipWithIndex`가 id값을 붙여준 것 처럼 인자를 붙여줌
* `drop` : 인자번째까지 떨구고 남은 것을 돌려줌
* `find` : 조건에 맞는 값을 돌려줌
* `reverse` : 순서가 반대인 리스트를 돌려줌

---

# 후기

제법 많은 고차 함수들을 살펴봤습니다. 예제 만드는라 힘들었습니다. ㅠㅠ
많이 쓰이는 힘수들을 소개 했지만 아직도 몇 가지 더 있습니다. 이런 것들은 스칼라 doc을 통해 찾아보시길 바랍니다.

---

참고

http://hamait.tistory.com/606

https://www.scala-lang.org/api/current/index.html

http://knight76.tistory.com/entry/scala-groupBy-distinct-sortWith-sortBy-%EC%98%88%EC%8B%9C

https://alvinalexander.com/scala/collection-scala-flatmap-examples-map-flatten

https://bradcollins.com/2015/06/06/scala-saturday-the-sorted-sortby-and-sortwith-methods/