---
layout: post
title:  Scala - implicit (암시적 파라미터, 암시적 변환)
categories: scala
description:  scala의 독특한 문법 implicit에 대해 알아봅시다.
tags:
- scala
- 스칼라
- implicit
---

## 서론

모나드를 깨우치고 한 동안 실수가 없을줄 알았으나... `implicit`을 실수해서 삽질을 조금 했습니다. ㅠㅠ

---

## Implicit

참조 자료
https://docs.scala-lang.org/ko/tutorials/tour/implicit-parameters.html.html
http://hamait.tistory.com/605

DB에 관한 코딩을 하다 제가 짠 인터페이스 어딘가에서 DB session을 `implicit` 키워드를 이용해 파라미터로 받는게 엉뚱한 DB session을 가져와
함수를 호출하는 문제를 찾았습니다. 트랜잭션 중간에 다른 DB session을 가져와서 당연히 트랜잭션은 안됬습니다. 그렇게 삽질하다 위의 두 글을 보고
`implicit`에 대한 공부를 하게 되었습니다. (일단 버그는 사수님의 도움으로 고쳤습니다.)

위의 두 글이 상당히 잘 정리해줘서 제가 새로운 이야기를 하지는 않습니다.

### 파라미터

`implicit` 키워드를 이용해서 매 번 인자를 직접 주지 않고 함수가 알아서 인자를 입력 받게 할 수 있습니다.

다음 예시를 보며 특징을 살펴봅시다.

* `implicit`은 주로 인자를 받는 두 번째 괄호에서만 사용합니다.
* 한 번 `implict` 으로 선언한 인자가 있으면 그 괄호 안에 모든 인자는 `implicit` 취급입니다.
* 똑같은 타입의 `implicit`을 두 번 할 수 없습니다.



    trait number
    case class monoid[A](point: A) extends number
    def main(args: Array[String]): Unit = {
  
      implicit val zdfs = monoid(0)
      implicit val yfds = monoid(1.11)
      def printXYZ(x: Int)(implicit y: monoid[Int], z: monoid[Double]): Unit = println(s"${x}, ${y.point}, ${z.point}") // y, z 둘 다 묵시적으로 인자를 받아온다.
      printXYZ(5) // 결과: 5, 0, 1.11
      
      implicit val world = "world"
      def printS1AndS2(s1: String)(implicit s2: String) = println(s1 + " " + s2)
      printS1AndS2("hello") // 결과: hello world
    }

두 번째 `implicit`을 호출한 함수는 일반적인 사용법입니다.
첫 번째 `implicit`을 호출한 함수는 `implicit`을 이용해 제네릭으로 들어오는 타입을 강제했고 인자도 두 개 입니다.

한 가지 단점을 찾자면 라이브러리를 만든 사람은 `implicit`을 사용해 인자를 알아서 가져올 것이라 생각하지만 라이브러리를 사용하는 입장에서는 어떤 인자를 `implicit` 시켜야 어떤 함수가 호출되는지 고민해야 될 것 같습니다.
좀 더 리뷰에 앞서 출처를 발히면 아까 위에 참조한 첫 번째 링크에서 가져왔습니다.  

    /** 이 예제는 어떻게 암묵적인 파라미터들이 동작하는지 보여주기 위해, 추상적인 수학으로부터 나온 구조를 사용한다. 
    하위 그룹은 관련된 연산(add를 말함/한쌍의 A를 합치고 또다른 A를 리턴)을 가진 집합 A에 대한 수학적인 구조이다. */
    abstract class SemiGroup[A] {
      def add(x: A, y: A): A
    }
    /** monoid는 A의 구분된 요소(unit을 말함/A의 어떤 다른 요소와 합산될때, 다른 요소를 다시 리턴하는)를 가진 하위 그룹이다 */
    abstract class Monoid[A] extends SemiGroup[A] {
      def unit: A
    }
    object ImplicitTest extends App {
      /** 암묵적 파라미터들의 동작을 보여주기 위해서, 우리는 먼저 문자열과 숫자에 대한 monoid를 정의했다.
       implicit 키워드는 해당하는 object가 암묵적으로 어떤 함수의 implicit으로 표시된 파라미터에 이 scope안에서 사용될수 있음을 나타낸다. */
      implicit object StringMonoid extends Monoid[String] {
        def add(x: String, y: String): String = x concat y
        def unit: String = ""
      }
      implicit object IntMonoid extends Monoid[Int] {
        def add(x: Int, y: Int): Int = x + y
        def unit: Int = 0
      }
      
      /** 이 메서드는 List[A]를 가지며 A를 리턴한다. 그리고 리턴된 A는 전체 리스트에 걸쳐, 지속적으로 monoid 연산이 적용되 합쳐진 값을 이야기 한다. 
      파라미터 m을 암시적으로 만든다는 것은, 우리가 반드시 호출 시점에 xs파라미터를 제공해야한다는 것을 의미하고, 그 이후에 우리가 List[A]를 가진다면, A가 실제로 어떤 타입인지, 어떤타입의 Monoid[A]가 필요한지도 알게 된다. 
      그 후, 현재 scope에서 어떤 val 또는 object가 해당 타입을 가지고 있는지 암묵적으로 알게 되며, 명시적으로 표현할 필요 없이 그것을 사용할 수 있다. */
      def sum[A](xs: List[A])(implicit m: Monoid[A]): A =
        if (xs.isEmpty) m.unit
            else m.add(xs.head, sum(xs.tail))
      /** 아래코드에서 우리는 각각 하나의 파라미터로 sum을 두번 호출한다.
       sum의 두번째 파라미터인 m이 암시적이기 때문에 그 값은 현재 scope에서 각 케이스마다 요구되는 monoid의 타입을 기준으로 탐색된다.
        두 표현은 완전히 계산될 수 있음을 의미한다. */
      println(sum(List(1, 2, 3)))          // uses IntMonoid implicitly
      println(sum(List("a", "b", "c")))    // uses StringMonoid implicitly
    }

저 소스를 일단 간략 설명하면 모노이드에 관한 것은 다 무시하시고 `sum` 메소드에서 두 번째 인자로 `implicit` m 이라는 파라미터를 확인하시면 됩니다.
일단 모르긴 몰라도 맨 밑 두 가지 `print` 문에서 신기하게도 상황에 알맞은 변수를 파라미터로 알아서... 묵시적으로... 암시적으로... 인자로 가져가 함수를 호출했습니다.

일단 저는 저기까지만 확인하고 코딩을 했었는데 제가 짠 트랜잭션 코드에서 엉뚱한 DB session을 가져다 쓴 이유는 저 모노이드와 다른 구조의 session 인 문제 외에도
애초에 제가 중간에 `implicit` 변수의 이름을 계속 오락가락 바꿔 써서 다른 session을 가져왔습니다. 만약 다음에 같은 부모 클래스를 가진 다른 타입의 변수로 `implicit` 인자가 바뀐다면
한 번 의심해 볼만한 문제입니다...

사실 코드 네이밍에 주의를 기울였다면 문제가 없었을 일이지만 그래도 `implicit`에 대해 찾아보는 계기가 되어 나름 괜찮았습니다. 

### 타입변환

먼저 소스의 출처를 밝히면 마틴오더스키 교수의 프로그래밍 스칼라 3판에서 가져왔습니다

    implict def IntToRational (n : Int)= new Rational(n)

위와 같이 `implicit`을 이용해 타입간의 변환을 지원할 수 도 있습니다.

스칼라는 정적 타입 언어이면서도 이런 꼼수?가 지원되는 특하게 발전된 정적 타입 언어인 것 같습니다.

뭔가 이 한 줄만 보고는 감이 안 오실 것 같아서 몇 가지 블로그들을 링크로 남깁니다.

분수 클래스 만들기: http://yujuwon.tistory.com/entry/Scala-%EB%B6%84%EC%88%98-%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%A7%8C%EB%93%A4%EA%B8%B0
암시적 변환: https://docs.scala-lang.org/ko/tutorials/tour/implicit-conversions.html.html
스칼라 공부할 때 맨날 보는 하마님 implicit: http://hamait.tistory.com/605
간단한? 블로그: http://partnerjun.tistory.com/21