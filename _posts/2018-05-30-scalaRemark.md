---
layout: post
title: 스칼라주석
tags:
- 스칼라
---


# 스칼라 주석 가이드

----------------

## scaladoc
intelliJ에서 스칼라 주석을 통한 *scalaDocs*을 만드는 기능이 있다.

상단 -> tool -> Generate scaladoc -> 각각 설정 후 ok 버튼

## docs를 만들기 위한 태그들
https://docs.scala-lang.org/overviews/scaladoc/for-library-authors.html

## 주석 예시들

다음 예시들은 scala.util과 scala 공식문서를 발취하고 참조했다

## 예시
        /** The empty list.
         *
         *  @author  Martin Odersky
         *  @version 1.0, 15/07/2003
         *  @since   2.8
         */
        @SerialVersionUID(0 - 8256821097970055419L)
        case object Nil extends List[Nothing] {
          override def isEmpty = true
          override def head: Nothing =
            throw new NoSuchElementException("head of empty list")
          override def tail: List[Nothing] =
            throw new UnsupportedOperationException("tail of empty list")
          // Removal of equals method here might lead to an infinite recursion similar to IntMap.equals.
          override def equals(that: Any) = that match {
            case that1: scala.collection.GenSeq[_] => that1.isEmpty
            case _ => false
          }
        }

## object
        /** Factory for [[mypackage.Person]] instances. */
        object Person {
          /** Creates a person with a given name and age.
           *
           *  @param name their name
           *  @param age the age of the person to create
           */
          def apply(name: String, age: Int) = {}
        
          /** Creates a person with a given name and birthdate
           *
           *  @param name their name
           *  @param birthDate the person's birthdate
           *  @return a new Person instance with the age determined by the
           *          birthdate and current date.
           */
          def apply(name: String, birthDate: java.util.Date) = {}
        }

## package

        package my.package
        /** Provides classes for dealing with complex numbers.  Also provides
         *  implicits for converting to and from `Int`.
         *
         *  ==Overview==
         *  The main class to use is [[my.package.complex.Complex]], as so
         *  {{{
         *  scala> val complex = Complex(4,3)
         *  complex: my.package.complex.Complex = 4 + 3i
         *  }}}
         *
         *  If you include [[my.package.complex.ComplexConversions]], you can
         *  convert numbers more directly
         *  {{{
         *  scala> import my.package.complex.ComplexConversions._
         *  scala> val complex = 4 + 3.i
         *  complex: my.package.complex.Complex = 4 + 3i
         *  }}}
         */
        package complex {}

### @constructor

        /** A person who uses our application.
         *
         *  @constructor create a new person with a name and age.
         *  @param name the person's name
         *  @param age the person's age in years
         */
        class Person(name: String, age: Int) {
        }


### @autor
    //@author 만든사람

### @version
    //@version 버전 날짜

### @since 
    //@since 언제부터사용했는지

### @param
    //@param 변수이름 설명

### @tparam
    //@tparam T curring에 사용되는 파라미터 설명(T타입 인자)

### Returns

	/** Returns `true` if this value is less than or equal to x, `false` otherwise. */
    def <=(x: Byte): Boolean

### @example

        	  /**
           *  @example {{{
           *  // Given a list
           *  val letters = List('a','b','c','d','e')
           *
           *  // `slice` returns all elements beginning at index `from` and afterwards,
           *  // up until index `until` (excluding index `until`.)
           *  letters.slice(1,3) // Returns List('b','c')
           *  }}}
           */
          override def slice(from: Int, until: Int): List[A] = {
            val lo = scala.math.max(from, 0)
            if (until <= lo || isEmpty) Nil
            else this drop lo take (until - lo)
          }

## 익명함수의 경우

가끔 익명함수에 주석을 달게될수 있으나 그런 경우 *intellij*에서는 밑줄을 쳐준다.
예를 들면  *scalatra* 에서 각 API URL들... get(){} 형식의 servlet에 구현한 것들은 익명함수를 호출하는 것이므로 주석을 달아도 빨간줄이 생기고 docs가 생성되지 않는다.

          /**
           * ---
           * @param..
           * @param..
           */
          get ("/") {

            .
            .
            .
            }
