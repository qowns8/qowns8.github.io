---
layout: post
title:  튜토리얼 - mysql 접속 툴 추천
categories: database
description: 끄적이다 나온 mysql 접속 툴 추천
tags:
- mysql
- DB
---

## 접속 툴

DB 중에서도 mysql에 접속하는 툴에 대해 소개하려고 합니다.

DB를 접속 할 때는 주로 리눅스 계열/윈도우 계열로 설치 방법이 나뉨니다.

주로 추천하는 툴은 다음과 같습니다.

- 공통
    - Mysql Workbench
    - Jetbrains DataGrip
    - PhpAdmin
- Window
    - HeidiSQL
- Mac&Linux
    - Sequal Pro
    
공통은 컴퓨터의 운영체제 상관없이 지원되는 툴이고 나머지는 각 운영체제를 쓸 때 추천하는 툴입니다.

차례로 추천 해보겠습니다.


### Mysql Workbench

![워크 밴치](https://dev.mysql.com/doc/workbench/en/images/wb-performance-dashboard.png)

mysql 공식 사이트에서 받을 수 있는 툴 입니다. 운영체제 상관 없이 쓸 수 있습니다.
제일 중요한 장점은 *무료* 툴 입니다. 땡큐 브레이브 하트!

장점

- 원도우, 맥, 리눅스 모든 버전을 지원한다.
- 컴퓨터에 mysql 설치와 같이 받을 수 있다.
- 무료

단점

- 프로그램이 너무 무겁다.

---

### Jetbrains DataGrip

![데이터 그리프](https://res.cloudinary.com/canonical/image/fetch/q_auto,f_auto,w_430/https://dashboard.snapcraft.io/site_media/appmedia/2017/11/1.png)

한 번 쓰면 마약 같은 중독성으로 다른 툴은 못 쓰게 하는 툴입니다.
30일 체험만 가능한 유료 툴이지만 기능이 강력해서 추천합니니다.

장점

- 자동완성, 믿줄, 자동 줄 맞춤, 플러그인 무슨 말이 더 필요할까...

단점

- 굳이 이런걸 쓸까? 싶은 다양한 기능 때문에 복잡해 보임
- 구매를 고민되는 가격

### PhpMyAdmin

![phpmyadmin](https://www.phpmyadmin.net/static/images/screenshots/structure.png)

많이 이상하고 취약점도 많아 많은 개발자들이 오! 갓뎀 php를 외치지만 자료가 많은 PhpMyAdmin 입니다. 

장점

- 자료가 많고 생활 코딩에 강의가 있다.

단점

- 생긴게 별로다
- 굳이 이걸...

### HeidiSQL

![heidisql](https://i.stack.imgur.com/Am3kM.jpg)

가볍지만 왠 만한 작업은 다하는 윈도용 툴입니다.
역시 무료하는 장점이 큼니다.

장점

- 무료
- 가볍지만 왠 만한 작업을 하기에 무리가 없음

단점

- only window

### Sequel Pro

![sequel](https://blog.calevans.com/wp-content/uploads/2015/10/Screen-Shot-2015-10-07-at-8.47.32-AM.png)

맥에서 쓸만한 툴입니다.

장점

- 무료

단점

- 가끔 입력한 쿼리가 많으면 느려짐

# 출처

- https://dev.mysql.com/doc/workbench/en/images/wb-performance-dashboard.png
- https://res.cloudinary.com/canonical/image/fetch/q_auto,f_auto,w_430/https://dashboard.snapcraft.io/site_media/appmedia/2017/11/1.png
- https://blog.calevans.com/wp-content/uploads/2015/10/Screen-Shot-2015-10-07-at-8.47.32-AM.png