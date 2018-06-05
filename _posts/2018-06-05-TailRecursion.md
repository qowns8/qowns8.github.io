---
layout: post
title: 재귀함수 + 메모이제이션 +  trailRecursion(꼬리재귀) + 파스칼의 삼각형
categories: algorithm
description: 비효율의 극치인 그냥 재귀함수만 알던 이들에게 꼬리재귀와 메모이제이션을 소개합니다.
tags:
- scala
- 스칼라
- 알고리즘
- 파스칼의 삼각형
---

## 서론

재귀함수를 써본적이 있으십니까? 알고리즘 문제풀이 거의 제일처음에 분할 정복을 배우기 위해 나오는 코딩 기법입니다. 하지만 실재로 코딩하다보면 재귀함수를 잘 사용하지는 않습니다.
저도 `Java`, `C`를 할때까지만 해도 재귀함수는 그냥 `for`문의 대체제에 불과하단 생각을 했지만 `Scala`와 함수형 문법들을 사용하면서 생각보다 휠씬 유용하다는 생각을 하게 되었습니다.

## 재귀함수

재귀함수는 말 그대로 자기 자신을 호출합니다.

제가 가장 처음 재귀함수를 썼던건 링크드리스트를 공부하면서 리스트를 `C`로 구현할때 겉멋으로 `print`를 구현했을때 였습니다. 지금 기억을 살려보면 다음과 같은 모습이었습니다.
    
    void printNode(Node *nodes) {
        if(node->data == NULL) {
            return;
        } else {
            printf("%s, ", node->data);
            printNode(node->next);
        }
    }

재귀함수는 특정 조건에 걸릴때 루프에서 빠져나오고 아닐 경우 자기자신을 호출합니다. 또 다른 예시를 살펴봅시다.

    int fibo(int n){
        if(n == 0) return 0; // 루프 탈출
        else if(n == 1) return 1; // 루프 탈출
        else return fibo(n-1) + fibo(n-2); // n이 0, 1이 될때까지 계속 호출됨
    }
    
방금 보신 함수는 피보나치 수열을 재귀함수로 만든 함수입니다. 때로는 재귀함수로 만들면 저렇게 많이 줄어들기도 합니다.
만약 위의 2개의 함수를 보셔도 이해가 안가시면 `병합정렬`과 `트리`를 생각하시면 됩니다. 윈하는 결과가 나올때가지 트리가 계속 뻗어나가 (호출해) 마지막에
결과(트리의 모든 노드)를 다 병합하는 느낌입니다.


재귀함수의 시간복잡도는 다음을 참조하세요
https://ratsgo.github.io/data%20structure&algorithm/2017/09/11/recurrence/


## 재귀함수의 문제와 해결법

재귀함수는 몇가지 문제점을 가지고 있습니다. 제일 첫 번째는 계산한 결과를 또 계산한다는 문제가 있습니다.

### 메모이제이(Memoization)
위에서 예시로 들은 피보나치 수열을 생각해봅시다.

    // fibo(5)를 호출했을때 과정
    // fibo는 1과 0일때 끝난다
        
                               fibo(5)
                      /                     \
                     fibo(4)               fibo(3)
                 /           \             /       \
             fibo(3)        fibo(2)      fibo(2)  fibo(1)
            /       \      /       \      /     \
         fibo(2) fibo(1) fibo(1) fibo(0) fibo(1) fibo(0)  
        /    \
     fibo(1) fibo(0)

우리 생각하는 수학으로는 

    fibo(3) + fibo(4) = fibo(5)

고작 몇번 더하면 될걸 재귀함수로 구한 피보나치수열은 똑같은 숫자를 계속 구하는 매우 비효율적인 알고리즘입니다. 만약의 fibo의 인자가 커진다면
함수호출의 수도 기하급수적으로 늘어나고 똑같은 함수를 수시로 계산하게 됩니다.

이런 문제를 해결하기 위해 메모이제이션이라는 방법을 사용합니다.

메모이제이션이란 프로그래밍을 할 때 반복되는 결과를 메모리에 저장해서 다음에 같은 결과가 나올 때 계산을 하는게 아닌 그냥 값을 꺼내쓰는 기법을 말합니다.

    allocate array for memo, setting all entries to zero;
    initialize memo[1] and memo[2] to 1;
    
    fib(n) {
       if memo[n] is zero:
           memo[n] = fib(n-1) + fib(n-2);
       return memo[n];
    }
    
소스출처: https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EC%9D%B4%EC%A0%9C%EC%9D%B4%EC%85%98

### 꼬리재귀(Tail Recursion)

C언어 예시를 살펴볼려면 다음 글을 참조하시길 바랍니다.
http://bozeury.tistory.com/entry/%EA%BC%AC%EB%A6%AC-%EC%9E%AC%EA%B7%80-%EC%B5%9C%EC%A0%81%ED%99%94Tail-Recursion

방금 링크의 글을 요약하면

- 재귀 함수는 호출 시 마다 스택 공간을 이용하므로 무리하게 호출하면 스택 오버플로우가 일어날 수 있다.
- 재귀 함수의 호출 횟수는 스택의 남은 공간과 재귀 함수의 지역 변수 사이즈에 따라 달라진다.

위의 두가지 문제를 해결하기 위해 꼬리 재귀를 사용합니다.

많은 함수형 언어들의 컴파일러는 꼬리재귀를 최적화 해줍니다. 스칼라의 예시로는 `@tailrec` 어노테이션을 통해 꼬리재귀 최적화가 가능합니다.
그외에 결과값을 두번째 인자로 넘기는 방법도 사용 가능합니다.

    // 팩토리얼 구하기
     @tailrec
     def factorial(num: Int)(r: Int): Int = {
       if(num == 1) r
       else factorial(num - 1)(r * num)
     }

주의해야 할 점으론 스칼라에서 꼬리재귀 함수는 재귀밖에서 계산되면 안됩니다.

    // bad.. 꼬리재귀가 아니다...
    def factorial(num: Int): Int = {
      if (num == 1) num
      else num * factorial(num - 1)
    }

## 파스칼의 삼각형

예전에 재미로 재귀함수만 사용하여 파스칼의 삼각형이 있습니다.(꼬리재귀 아닌 것도 있습니다)
for문까지 전부 마음 먹으면 재귀함수로 대체할 수 있지만 평소 코딩할땐 굳이 그럴 필요는 없어 보입니다.

        object RecPascal {
        
        
          def combination(c: Int, r: Int): Int = {
            def factorial(num: Int): Int = {
              def loop(N1: Int, N2: Int): Int = {
                if (N2 <= 1) N1
                else loop(N1 * N2, N2 - 1)
              }
              loop(1, num)
            }
            factorial(r) / (factorial(c) * factorial(r - c))
          }
        
          def pascal(line: Int): Unit = {
            def loop1(row: Int, line: Int): Int ={
              def loop2(col: Int, row: Int): Int ={
                print(combination(col, row) + " ")
                if (col == row) return row
                else loop2(col + 1, row)
              }
              println()
              loop2(0, row)
              if(row == line) row
              else loop1(row + 1, line)
            }
            loop1(0, line)
          }
        
          def main(args: Array[String]): Unit = {
            //println(combination(12, 6))
            pascal(10)
          }
        }