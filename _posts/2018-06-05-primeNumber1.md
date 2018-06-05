---
layout: post
title: Stream을 이용해 3줄로 엄청 많은 소수점 찾기
categories: algorithm
description: 제목 그대로 Scala Stream을 이용해 엄청난 크기의 소수점을 찾는 방법입니다.
tags:
- scala
- 스칼라
- 알고리즘
- 소수
---

## 서론

알고리즘의 단골 문제 소수찾기... 이제 식상합니다. 그래도 왜 이렇게 많이 나오는지를 생각해보면 소수가 암호화와 깊은 연관이 있어서 그렇다고 생각합니다.
이 글에선 for문으로 순회해서 소수를 찾는 방법 말고 Stream을 이용하여 찾는 방법을 소개합니다.

## 에라토스테네스의 체

다음 링크들의 글을 참조하세요
https://ko.wikipedia.org/wiki/%EC%97%90%EB%9D%BC%ED%86%A0%EC%8A%A4%ED%85%8C%EB%84%A4%EC%8A%A4%EC%9D%98_%EC%B2%B4
http://ledgku.tistory.com/61

에라토스테네스의 체를 요약하면 2, 3, 5, 7의 배수는 모두 소수가 아니니깐 걸러내자입나다. 매우 고전적이지만 효과적인 알고리즘 입니다.
성능이 구린것만 빼면요...

우리는 이 고전적인 방법으로 백만 단위에 소수를 구할 것 입니다.

## 매우 간단하다...

Stream은 아직 만들어지지 않은 수들로 이뤄진 배열로 생각하시면 편합니다. 근데 왜 Stream을 써야할까요? 만약에 range 함수로 그냥 list를 만들때 크기가
매우 큰 list를 만들면 jvm이 뻗을수도 있습니다. 너무 오래걸려요

    scala> val t = List.range(1, 10000000)
    java.lang.OutOfMemoryError: GC overhead limit exceeded
      at java.lang.Integer.valueOf(Integer.java:832)
      at scala.runtime.BoxesRunTime.boxToInteger(BoxesRunTime.java:65)
      at scala.math.Numeric$IntIsIntegral$.plus(Numeric.scala:58)
      at scala.math.Numeric$Ops.$plus(Numeric.scala:215)
      at scala.collection.generic.GenTraversableFactory.range(GenTraversableFactory.scala:224)
      at scala.collection.generic.GenTraversableFactory.range(GenTraversableFactory.scala:206)


그에 비해 Stream은 계산될때 그 값을 만들기에 이런 걱정은 없죠

    val t = Stream.range(1, 10000000)
    t: scala.collection.immutable.Stream[Int] = Stream(1, ?)

이제 딱 1줄만 쓰면됩니다.

    scala> t.filterNot(_ % 2 == 0).filterNot(_ % 3 == 0).filterNot(_ % 5 == 0).filterNot(_ % 7 == 0).print

filterNot은 조건 값에 해당된 값을 걸러버리는 함수입니다.

적당히 천만개만 돌려봅시다

    3791969, 3791971, 3791981, 3791983, 3791987, 3791989, 3791993, 3791999, 3792001, 3792007, 3792011, 3792013, 3792017, ...

순식간에 백만자리까지 찾고 print로 인해 버벅거리는 모습을 보실 수 있습니다.

왠만한 방법보단 Stream과 고전적인 에라토스테네스의 체를 사용한 방법이 휠씬 좋네요 자료형만 바꿔도 이렇게 성능이 좋아집니다.