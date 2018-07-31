---
layout: post
title: Go 언어 - 의존성 관리 도구 dep과 배포
categories: Golang
description: 최근 공부하는 Go 언어의 의존성 관리와 배포자동화 방법
tags:
- DevOps
- Golang
- 배포자동화
---

# go dep, AWS code pipeline

개발 환경 및 사용한 기술

* os : os x
* lang : Go
* dependency : dep
* build server : Aws CodeBuild
* deploy server : Aws Ec2 T2 micro
* etc : Aws CodePipeline, Aws Elastic beanstalk , GitHub,  ...

# 개요

저번 금요일 회사에서 크롤러도 짜고 oAuth도 해보고 GO언어 스터디를 하다가 제이슨이 단순히 코드, 문법 공부는 
학부생 수준에서도 하는 일이고 개발자는 의존성, 배포, 관리 이슈를 
생각해야된다고 했습니다. 이 말을 듣고 가슴이 철렁했습니다.
저는 개발을 잘하려면 한참 멀었군요... 어째튼 그렇게 되서 문법 공부하던 중에 의존성 관리와
배포환경 구성을 시작했습니다.


공부하면서 짠 부분은 `Go`로 google oauth 예시를 배끼고 로그인 api 개발한 소스였습니다.

# 의존성 관리

    go get [repo]

위의 방법으로 패키지를 다운 받다가 `dep`을 쓰기로 했습니다.

`dep`은 의존성 관리 도구로 공식문서엔 "official experiment" 라고 합니다.

설치 방법은 다음 3가지 중 한 가지로 설치하면 됩니다.

    # 1번
    brew install dep
    brew upgrade dep
    
    # 2번
    curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
    
    # 3번
    go get -u github.com/golang/dep/cmd/dep


도큐멘트를 보고 싶으면 다음 링크를 참조하세요

https://golang.github.io/dep/docs/introduction.html

## dep 로컬

처음에는 유저 밑에 go 폴더에 패키지들을 담아두고 사용하고 있었습니다.

    /usr/local/go/
    
    ~/go
    

`dep`을 로컬로 따로 구성하기 위해 Go의 환경변수 중 $GOPATH를 프로젝트 파일로 잡았습니다.

    /worksapce/pojectName
    
    projectName
    |
    |- src -- main.go
    |      |
    |      |-- etc
    
 
src 파일로 이동해 dep의 5개 밖에 없는 명령어 중 init로 환경을 구성합니다.

    dep init
    
    ls
    
    ## 다음 3가지가 새로 생김
    Gopkg.toml Gopkg.lock vendor/

이제 코드를 빌드하기 위해 실제 의존성을 추가해줍시다.

    dep ensure
    
`dep ensure`는 vendor 디렉토리에 패키지들을 가져옵니다. add 명령어로 직접 추가도 가능합니다.

    dep ensure - add github/pak/name/...
    
어떤 패키지들을 사용하는지 확인하려면 Gopkg.toml을 확인하면 됩니다.

{% highlight toml %}

    Gopkg.toml example
    
    Refer to https://golang.github.io/dep/docs/Gopkg.toml.html
    for detailed Gopkg.toml documentation.
    
    required = ["github.com/user/thing/cmd/thing"]
    ignored = ["github.com/user/project/pkgX", "bitbucket.org/user/project/pkgA/pkgY"]
    
    [[constraint]]
      name = "github.com/user/project"
      version = "1.0.0"
    
    [[constraint]]
      name = "github.com/user/project2"
      branch = "dev"
      source = "github.com/myfork/project2"
    
    [[override]]
      name = "github.com/x/y"
      version = "2.4.0"
    
    [prune]
      non-go = false
      go-tests = true
      unused-packages = true

{% endhighlight %}

로컬에서 환경을 구성하면서 src 파일 밖에서 init를 한다던가, GOPATH를 이상하게 잡하는 실수로
꽤 많은 시간을 사용했습니다. GOPATH를 현재 프로젝트 경로로 주의해서 잡으면 될 것 같습니다.

## github에서 dep

https://golang.github.io/dep/docs/new-project.html 의 1번에 따라 
github에서 여럿이서 프로젝트를 공유할 수 있게 세팅을 바꾸니 프로젝트 구성도 조금 바꾸게 되었습니다.

    $GOPATH/src/github.com/유저이름/프로젝트이름
        
    #workspace/프로젝트이름/src/github.com/유저이름/프로젝트이름

실제 프로젝트가 위의 경로로 들어가게 세팅을 바꿨어야 했습니다.

이대 구성대로 git에 `/프로젝트이름` 부분을 공유했습니다.

주의할 점으로는 로컬구성과 다르게 로컬 패키지의 import를 수정해줘야 합니다.

{% highlight go %}

    import (
    	"./router"
    	"log"
    	"net/http"
    )
   
   	import (
    	"net/http"
    	"log"
    	"html/template"
    	"../utils"
    	"../models"
    )
    
{% endhighlight %}

위의 import를 아래처럼 수정됩니다.

{% highlight go %}

    import (
    	"github.com/username/project/router"
    	"log"
    	"net/http"
    )
   
   	import (
    	"net/http"
    	"log"
    	"html/template"
    	"github.com/username/project/utils"
    	"github.com/username/project/models"
    )

{% endhighlight %}

---

# 배포 자동화 AWS Code Pipeline

배포 환경은 Aws의 `Elastic Beanstalk`으로 구성했습니다.

Elastic Beanstalk은 Ec2, S3, 보안그룹 설정을 쉽게할 수 있는 배포 환경입니다.

구성의 특별한 어려움은 없고 Aws 콘솔에서 클릭 몇 번으로 끝나기에 생략하겠습니다.

배포할 서버가 갖추어졌으면 배포 자동화를 시켜야합니다.

제 부끄러운 스터디 소스를 남들에게 공개하고 싶지 않고 돈이 들더라도 회사의 갓갓 자기계발비로 해결되서
`Travis`(오픈소스에 한해서만 무료 이용 가능)가 아닌 `Aws CodePipeline`으로 배포 자동화 환경을 구성했습니다.

앞에 부끄러운건 두 번째 이유고 진짜 이유는
`Aws CodePipeline`의 장점으로 Aws 기술 스택을 쓴 다면 연동이 매우 편리합니다.
다른 CI 와 다르게 특별한 쉘이나 .yml 없이도 알아서 연동이 가능했습니다.
    
    Github(소스 저장소) -> Aws Code build(빌드 서버) -> Aws elastic beanstalk(배포 서버)

처음 파이프라인을 구성할 때 소스 제공을 Github으로 로그인 하시고 저장소를 설정하면 빌드 환경을 구성하는 부분이 나옵니다.

이 때 Jenkins 와 Aws CodePipeline을 선택할 수 있는데 저는 CodePipeline을 선택하고 프로젝트 루트에 buildspec.yml 을 추가했습니다.

{% highlight yml %}

    version: 0.2
    
    phases:
      install:
        commands:
          - go get -u github.com/golang/dep/cmd/dep
          - mkdir src
          - cd ./src
          - mkdir github.com
          - cd github.com
          - mkdir username
          - cd username
          - git clone https://github.com/username/project
          - cd project
          - dep ensure
      build:
        commands:
          - go build -o bin/application main.go
    artifacts:
      files:
        - src/github.com/username/project/bin/application
      discard-paths: yes

{% endhighlight %}   

---

# 후기

하루 이상을 의존성 관리와 배포 자동화에 사용했습니다. 그런데 막상 정리하니 별로 분량이 없군요...
사실 소스야 인터넷에 좋은게 너무 널려서 devops와 설치가 제일 어려운 것 같습니다.

아직까지 주니어 개발자라서 프로젝트 구조를 다양한 스택을 이용해 마이크로서비스로 어떻게 한다느니 폭포수 모델 개발을 하겠다느니 이런 일을 하지는 않지만
앞으로 개인적인 공부를 해도 테스트, 배포, 자동화를 어느정도 생각하고 공부해야겠습니다.