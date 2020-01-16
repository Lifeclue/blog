---
layout: post
title: "사이트 즐겨찾기 서비스로 Kotlin 시작하기"
date: "2018-02-02 22:31:47 +0900"
category: kotlin
comments: true
tags: Kotlin 스프링 Spring 스프링부트 SpringBoot 인텔리제이 IntelliJ
published: false
---

요새 제 주변에서 [Kotlin](https://kotlinlang.org) 칭찬하는 사람들을 많이 접합니다. 그래서 Kotlin 사이트에 들어가봤지요. 그들의 주장으로는 Java 개발자가 쉽게 시작할 수 있다고 합니다. 그래서 제가 한 번 도전해 보겠습니다.

# Kotlin
저도 잘 모르는 Kotlin을 조금 소개해 보자면 Kotlin은 [JetBrains](https://www.jetbrains.com/)에서 만들었습니다. JVM에서 구동 가능하고 Java와 100% 호환이 됩니다. 문체는 간결하고 NullPointerException에서 자유로운 등의 안정성도 갖추었다고 하는군요. 사이트의 예제 코드 몇몇을 보니 쪼끔 간결해진 것 같긴 합니다만 역시 전시물만 봐서는 잘 모르겠네요.

뭐든 간단한 이야기나 프로젝트로 시작을 하는 것이 좋지요. 저는 사이트 주소를 입력하면 입력한 사이트들을 모아 즐겨찾기로 보여주는 서비스를 개발해보겠습니다.

개발 환경은 다음과 같습니다.
- 통합 개발 환경으로 IntelliJ
- SDK로 Java
- Framework으로 Spring

IntelliJ에서 새로운 프로젝트를 만들어보았습니다.  
![새 프로젝트 만들기 창]({{"/assets/images/page/kotlin-famphlet/new-project-1.png" | absolute_url}})

Gradle 프로젝트을 골라보았습니다. Kotlin DSL build script라는 게 있군요. 일단 Kotlin을 사용해 볼 예정이니 이것도 사용해 봅시다. 당연히 Kotlin으로 작업할 예정이므로 Libraries and Frameworks는 Kotlin으로 선택합니다.

간단히 프로젝트의 기본 정보를 입력한 후  
![새 프로젝트 기본 정보 입력 창]({{"/assets/images/page/kotlin-famphlet/new-project-2.png" | absolute_url}})

빌드 도구 설정시 Gradle Wrapper를 사용하는 것으로 설정하였더니 프로젝트에 gradlew 파일이 생성되지 않았습니다. IntelliJ의 문제인지는 잘 모르겠습니다. 사실 IntelliJ는 회사에서만 써보고 Community판은 처음 써보는군요. 그래서 [Gradle](https://gradle.org/install/#helpful-information)을 급하게 설치하였습니다.  
![새 프로젝트 빌드 설정 창]({{"/assets/images/page/kotlin-famphlet/new-project-3.png" | absolute_url}})

마지막으로 프로젝트 이름을 정한 후 프로젝트 생성을 완료하였습니다.  
![새 프로젝트 이름 입력 창]({{"/assets/images/page/kotlin-famphlet/new-project-4.png" | absolute_url}})

프로젝트를 생성하자마자 문제에 봉착하였습니다.
Gradle이 `CacheOpenException`을 발생시키며 다음과 같이 이야기합니다.
```
Could not open cache directory chv7ymti9wfycl8xz9b3wa14h (C:\Users\Lifeclue\.gradle\caches\4.5.1\gradle-kotlin-dsl\chv7ymti9wfycl8xz9b3wa14h).
```
서로 다른 gradle process가 작동하고 있어서 서로 Lock을 걸려다 결국 작동을 중지한다는 이야기가 있군요.
`gradle -stop` 명령을 내려봤습니다.
`2 Daemons stopped` 라고 하니 뭐가 돌고 있긴 했나봅니다.
그런데 여전히 빌드되지 않습니다.
[Gradle의 Kotlin DSL Github 프로젝트](https://github.com/gradle/kotlin-dsl/issues/487)에 참고할 만한 이슈가 올라왔군요. 먼저 Gradle wrapper를 사용하라고 합니다. 하지만 전 이미 Gradle을 설치하였으므로 따르지 않겠습니다. 이 이슈는 작성자가 캐시를 직접 지우고 나니 해결되었다며 닫혔습니다. 나중에 Gradle wrapper를 쓸 때 비슷한 문제가 발생하면 시도해봐야 겠군요.

프로젝트를 다시 만들어보죠. 이번에는 Gradle Kotlin DSL은 사용하지 않기로 하겠습니다.  
![새 프로젝트 이름 입력 창]({{"/assets/images/page/kotlin-famphlet/new-project-5.png" | absolute_url}})  
매우 잘 작동합니다. 뭐가 문제일까요? 여전히 Kotlin DSL Build script를 사용하여 프로젝트를 생성하면 같은 문제에 봉착합니다. 일단 이것을 사용하는 게 필수는 아니므로 이대로 시작하겠습니다. 저야 당연히 익숙한 쪽이 좋지요.
여전히 gradlew는 생성되지 않았군요. 회사의 Ultimate판에서는 프로젝트를 생성할 때마다 매번 생성되었는데 IntelliJ 버전 차이인건지.. 모르겠군요. IDE의 노예라 이런 일이 생기면 어리둥절 합니다.

![프로젝트 소스 구조도]({{"/assets/images/page/kotlin-famphlet/project-tree.PNG" | absolute_url}})  
프로젝트가 만들어졌습니다. 빌드에 성공하여 빌드 폴더도 생겼군요. Java와 Kotlin을 모두 선택하여 프로젝트를 생성하였기 때문에 소스 폴더에 java와 kotlin이 같이 생겼습니다.

흥미롭군요. 다음 포스트에서는 HTTP 요청을 받아 `안녕, 여러분`이라는 메시지를 반환해보도록 하겠습니다.
