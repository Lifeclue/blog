---
layout: post
title: "Spring boot로 '안녕, 여러분' 출력하기"
date: "2018-02-21 23:01:43 +0900"
category: kotlin
comments: true
---

코틀린 프로젝트도 만들었겠다, Spring boot 의존성을 추가하고 HTTP 요청을 통해 '안녕, 여러분'을 출력해보도록 하겠습니다.  
(본 포스트 중간 중간에 코틀린 문서를 참고하여 우리말로 작성한 내용은 상당히 의역 되었으므로 반드시 원문을 같이 참고하시기 바랍니다. 참고로 제 토익 점수는...)

# Spring boot with Kotlin

이 간단한 기능은 코틀린 사이트의 [Creating a RESTful Web Service with Spring Boot](https://kotlinlang.org/docs/tutorials/spring-boot-restful.html)를 참고하여 짜보도록 하겠습니다.

위 문서에서는 코틀린으로 스프링 부트가 매우 잘 작동한다고 설명합니다. 그러나 약간의 차이가 있다며 몇가지 필요 사항에 대해 언급하는데요. 그래들 예제 코드를 보면 가장 먼저 `Required for Kotlin integration`이라는 주석을 만나게 됩니다. 이것은 코틀린 프로젝트를 생성했을 때에도 build.gradle 스크립트에 기본으로 작성되어 있는 것으로, 그래들을 이용하여 코틀린을 빌드하기 위해 필수적인 요소입니다.

[이전 포스트](https://lifeclue.github.io/blog/kotlin/2018/02/02/intro-favorite-sites.html)에서 만든 프로젝트를 보면 기본적으로 작성되어 있는 것을 보실 수 있습니다. 전 포스트를 따라하고 오셨다면 이미 build.gradle에 buildscript 블록이 생성되어 있을 것입니다. 그리고 아래 내용이 모두 채워져 있을 것입니다.

{% highlight gradle %}
buildscript {
    ext.kotlin_version = '1.2.21'
    repositories {
        jcenter()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: "kotlin"
{% endhighlight %}

다시 문서를 봅시다. 이번엔 `kotlin-allopen`이라는 플러그인이 등장했네요. 친절하게 [참고할 문서](https://kotlinlang.org/docs/reference/compiler-plugins.html#kotlin-spring-compiler-plugin)의 URL을 주석으로 달아주었습니다. 문서의 내용은 대충 이렇습니다.

>기본적으로 코틀린의 클래스와 포함된 멤버들은 final이다. 그런데 이러한 사실은 클래스가 **열려있어야** 하는 스프링 AOP등의 라이브러리나 프레임워크들에겐 불편한 사실이다. `all-open compiler plugin`은 명시적으로 `open` 키워드를 붙이지 않아도 특정 어노테이션(스프링의 경우 @Configuration, @Service 등)이 명시된 클래스와 그 멤버들을 open 상태로 만들어준다.

`코틀린은 기본적으로 final이다!` 이것이 무슨 말일까요? Classes and Inheritance의 [Inheritance](https://kotlinlang.org/docs/reference/classes.html#inheritance)절을 보면 다음과 같은 내용이 있습니다.

>`open`은 자바의 `final`과 반대다. open으로 정의해야 상속을 허용한다. [Effective Java](https://books.google.co.kr/books/about/Effective_Java.html?id=ka2VUBqHiWkC&redir_esc=y)의 17번 규칙 `상속을 위한 설계와 문서를 갖추지 못한다면 상속을 금지하라`에 따라서 코틀린의 모든 클래스는 기본적으로 final이다.

어쨌든 코틀린은 `final` 클래스와 멤버가 기본인데 스프링을 쓰려면 `open`이어야 한다는 것입니다. 그렇다고 스프링이 다뤄야 하는 클래스들에 일일이 `open`을 붙여주기 번거로우므로 코틀린에서는 플러그인을 제공하여 번거로운 작업을 없애겠다는 것이지요.

자 다시 그래들 코드로 돌아가 봅시다. Compiler Plugins의 [Spring Support](https://kotlinlang.org/docs/reference/compiler-plugins.html#spring-support)절을 보면 다음과 같은 플러그인 설정을 하라고 합니다. 위 코드에 붙여보죠.

{% highlight gradle %}
buildscript {
    ext.kotlin_version = '1.2.21'
    repositories {
        jcenter()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        // all-open 플러그인 종속성 추가
        classpath "org.jetbrains.kotlin:kotlin-allopen:$kotlin_version"
    }
}

apply plugin: "kotlin"
// kotlin-allopen이 아니라 kotlin-spring
apply plugin: "kotlin-spring"
{% endhighlight %}

`kotlin-spring`의 설정으로 다음의 어노테이션이 붙으면 열린 클래스가 됩니다.

- @Component
- @Async
- @Transactional
- @Cacheable
- @SpringBootTest

아래 어노테이션은 @Component가 메타 어노테이션으로 붙어있기 때문에 역시 열린 클래스가 됩니다.

- @Configuration
- @Controller
- @RestController
- @Service
- @Repository

당연히 스프링 부트의 그래들 플러그인도 붙여 주어야겠죠? 토이 프로젝트이니 안정성을 포기하더라도 2.0.0.RC2 버전을 써보겠습니다.

{% highlight gradle %}
buildscript {
    ext.kotlin_version = '1.2.21'
    ext.spring_boot_version = '2.0.0.RC2'

    repositories {
        jcenter()
        // 정식 버전이 아니므로 저장소 설정이 별도로 필요합니다.
        maven { url 'https://repo.spring.io/milestone' }
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "org.jetbrains.kotlin:kotlin-allopen:$kotlin_version"
        classpath "org.springframework.boot:spring-boot-gradle-plugin:$spring_boot_version"
    }
}

apply plugin: "kotlin"
apply plugin: "kotlin-spring"
apply plugin: 'org.springframework.boot'
{% endhighlight %}

이제 의존성 설정만 남겨 놓았네요. 의존성엔 역시 코틀린 기본 설정과 스프링 부트 설정을 해주면 됩니다. repositories 블럭과 dependencies 블럭은 이미 있을 것입니다. 없는 코드만 붙여 넣어 봅시다.

{% highlight gradle %}
repositories {
    jcenter()
    // 역시 정식 버전이 아니므로 저장소 설정이 별도로 필요합니다.
    maven { url 'https://repo.spring.io/libs-milestone' }
}
dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    compile "org.springframework.boot:spring-boot-starter-web:$spring_boot_version"
    testCompile("org.springframework.boot:spring-boot-starter-test:$spring_boot_version")
}
{% endhighlight %}

드디어 그래들 설정이 끝났습니다! 이제 코드를 작성해 볼 차례입니다.  
일단 전 포스트에서 자바와 코틀린을 같이 선택하였지요. 그래서 src 폴더에는 java와 kotlin폴더가 있습니다. 노파심에 자바를 선택하였지만 모험심을 발휘하여 java 폴더는 지우도록 하겠습니다.

![프로젝트 소스 파일 구조]({{"/assets/image/kotlin-famphlet/project-tree.PNG" | absolute_url}}) > > >
![프로젝트 소스 파일 구조]({{"/assets/image/kotlin-famphlet/greeting/project-tree.PNG" | absolute_url}})

코틀린은 디렉토리 구조를 만들 때 공통 패키지를 제외하는 것이 기본 규약입니다. (Coding Conventions의 [Directory structure](https://kotlinlang.org/docs/reference/coding-conventions.html#directory-structure)절 참고)  
또한 소스 코드의 package와 디렉토리 구조를 맞출 필요도 없습니다. (Basic Syntax의 [Defining Packages](https://kotlinlang.org/docs/reference/basic-syntax.html)절 참고)  
Famphlet 프로젝트의 기본 패키지는 com.lifeclue.blog.famphlet이지만 디렉토리 구조를 다음의 구조처럼 만들지 않아도 된다는 이야기입니다.  
(기본 패키지가 com.lifeclue.blog.famphlet인 이유는 제가 프로젝트를 만들 때 이렇게 지었기 때문입니다. 여러분도 정하고 싶은 대로 정하시면 됩니다.)

```
src/main/kotlin/com/lifeclue/blog/famphlet
```

일단 시작하는 단계이니 하위 디렉토리 없이 작성하겠습니다. 먼저 스프링 부트로 시작하고 있으니 스프링 부트 애플리케이션부터 만들어 볼까요? 코드를 작성하기 전에 클래스 파일을 만들겠습니다. 이 파일에는 하나의 클래스가 포함될 예정이니 클래스 이름으로 파일을 만들어야 합니다. (Coding Conventions의 [Source code organization >> Source file names](https://kotlinlang.org/docs/reference/coding-conventions.html#source-file-names)절 참고)

```
src/main/kotlin/GreetingApplication.kt
```

{% highlight kotlin %}
package com.lifeclue.blog.famphlet

import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication

@SpringBootApplication
class GreetingApplication

fun main(args: Array<String>) {
    SpringApplication.run(GreetingApplication::class.java, *args)
}
{% endhighlight %}

자 이제 스프링 부트를 실행할 수 있게 되었습니다.

한번 실행해 볼까요?

매우 잘 실행이 됩니다. 이제 Controller를 붙이고 "안녕, 여러분!"을 출력해봅시다.

```
src/main/kotlin/GreetingController.kt
```

{% highlight kotlin %}
package com.lifeclue.blog.famphlet

import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController

@RestController
class GreetingController {

    @GetMapping("/greeting")
    fun greeting() = "안녕, 여러분!"
}
{% endhighlight %}

이제 `/greeting`을 호출하면 "안녕, 여러분!"이 브라우저에 출력될 겁니다.

정말일까요?

저는 예외를 만나면서 서버가 종료되었습니다.

```
Caused by: java.lang.ClassNotFoundException: kotlin.reflect.full.KClasses
```

무슨 일일까요? 로그를 조금 거슬러 올라가 보겠습니다. 몇몇 로그들 사이에 이런 로그가 보이네요.

```
For Jackson Kotlin classes support please add "com.fasterxml.jackson.module:jackson-module-kotlin" to the classpath
```

Controller에 @RestController 어노테이션을 붙였으므로 이 컨트롤러는 기본적으로 @ResponseBody입니다. 즉, 응답을 위해 몇몇 HttpMessageConverter를 준비하는데 그 중 하나가 MappingJackson2HttpMessageConverter입니다. 이름에서 볼 수 있듯이 잭슨을 사용하고 있습니다.로그로 미루어 짐작해 보면 잭슨 코틀린 클래스를 지원하기 위해 추가 설정이 필요해 보입니다.
build.gradle의 dependencies 블럭에 다음의 의존성을 추가합시다.

{% highlight gradle %}
dependencies {
  compile group: 'com.fasterxml.jackson.module', name: 'jackson-module-kotlin', version: '2.9.4.1'
}
{% endhighlight %}

이제 다시 실행해 볼까요?

```
Started GreetingApplicationKt in 3.884 seconds (JVM running for 4.493)
```

스프링이 실행되었습니다. 브라우저에서 인사 API를 호출해 봅시다.

```
http://localhost:8080/greeting
```

브라우저에 "안녕, 여러분!"이 선명하게 써지는 군요.

다음 포스트에서는 Famphlet의 핵심 기능인 사이트 주소를 입력하면 해당 주소를 저장하는 기능을 만들어 보겠습니다.
