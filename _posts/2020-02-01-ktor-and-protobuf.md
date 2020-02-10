---
layout: post
title: "Ktor + ProtoBuf"
date: "2020-02-01 01:39:06 +0900"
categories: Ktor ProtoBuf
comments: true
image: /assets/images/page/ktor-protobuf/20141121_080941.jpg
image2: /assets/images/page/ktor-protobuf/20141121_080941_ptrt.jpg
tags: 케이터 Ktor 프로토버퍼 Protobuf 프로토콜버퍼 ProtocolBuffer
---

[케이터<sup>Ktor</sup>](https://ktor.io/)는 [젯브레인<sup>Jetbrains</sup>](https://www.jetbrains.com/)에서 만든 웹 서버 프레임워크입니다. 그리고 [프로토콜 버퍼<sup>Protocol Buffer</sup>](https://developers.google.com/protocol-buffers)는 구글<sup>Google</sup>에서 만든 구조적 데이터 전송 방식입니다.
보통 웹 서버 프레임워크에서는 데이터 전송 방식으로 제이슨<sup>JSON</sup>을 많이 채택하고 있어서 웹 서버에 프로토콜 버퍼<sup>Protobuf</sup>를 적용하는 경우는
아마 보기 드문 경우일 수 있습니다. 일단 저부터도 프로토콜 버퍼를 써본 적이 없습니다.
그래서 준비했습니다. 웹 서버 인터페이스로 프로토콜 버퍼 사용하기!

## 케이터

케이터는 오로지 [코틀린<sup>Kotlin</sup>](https://kotlinlang.org)으로만 작성된 젯브레인의 오픈소스 웹 서버 프레임워크입니다. 설정이 매우 쉽고 빠르게 서버를 띄울 수 있습니다.

```kotlin
// https://ktor.io/quickstart/index.html
// Main.kt
import io.ktor.application.*
import io.ktor.http.*
import io.ktor.response.*
import io.ktor.routing.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*

fun main(args: Array<String>) {
    val server = embeddedServer(Netty, port = 8080) {
        routing {
            get("/") {
                call.respondText("Hello World!", ContentType.Text.Plain)
            }
        }
    }
}
```

## 프로토콜 버퍼

프로토콜 버퍼는 언어 중립적이고 플랫폼 중립적인 구조적 데이터 전송 방식입니다. 서버와 클라이언트가 자바<sup>Java</sup>를 쓰던 코틀린을 쓰던 스위프트<sup>Swift</sup>를 쓰던 상관이 없다는 거죠. 그러나 프로토콜 버퍼는 인터페이스 정의 언어<sup>IDL:Interface Definition Language</sup>이기 때문에 인터페이스를 정의하기 위한 언어를 배워야 합니다. 이 언어로 작성한 프로토콜 버퍼 명세를 여러 언어로 컴파일할 수 있습니다. 이렇게 컴파일하면 API 명세를 문서로 만들어서 전달하거나 각자 JSON을 변환하기 위해 애쓰지 않아도 됩니다. 컴파일하면 바이트 배열<sup>ByteArray</sup>을 컴파일된 함수에 넣어서 컴파일된 객체로 바로 받을 수 있으니까요. 클라이언트도 마찬가지입니다.

또한 서로 데이터 구조를 잘못 짜서 오류가 생기는 일도 없겠죠? 클라이언트가 안드로이드<sup>Android</sup>와 아이폰<sup>iOS</sup>, 웹<sup>Web</sup> 등 여러개라면 그 효과는 배가 될 겁니다.

```protobuf
// https://developers.google.com/protocol-buffers/docs/overview#how-do-they-work
// Person.proto
message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phone = 4;
}
```

## 케이터 시작하기

먼저 프로젝트를 만들어 보겠습니다. 아무래도 코틀린을 사용하기에는 [인텔리제이<sup>IntelliJ</sup>](https://www.jetbrains.com/idea/)를 사용하는 것이 여러모로 이득입니다. 물론 인텔리제이 역시 젯브레인에서 만들었다는 것이 한몫 하겠죠?

인텔리제이에서 새 프로젝트 창을 띄우면 케이터 프로젝트를 만들 수 있습니다. 이를 통해 쉽게 케이터 프로젝트를 만들고 초기 설정을 건너뛸 수 있습니다.

![인텔리제이 케이터 새 프로젝트 만들기]({{"/assets/images/page/ktor-protobuf/new_project_ktor.PNG" | absolute_url}})

일단 이것 저것 선택하지 않고 만들었습니다. 제가 만진 거라곤 `GradleKotlinDsl`뿐입니다.

프로젝트가 완성되면 `build.gradle`에 케이터의 의존성<sup>Dependencies</sup>이 추가되어 있을 겁니다.

```kotlin
dependencies {
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version")
    implementation("io.ktor:ktor-server-netty:$ktor_version")
    implementation("ch.qos.logback:logback-classic:$logback_version")
    testImplementation("io.ktor:ktor-server-tests:$ktor_version")
}
```

- 코틀린 의존성
- 케이터의 [네띠<sup>Netty</sup>](https://netty.io/) 서버 의존성: 케이터의 웹서버로 네띠를 사용합니다. [다른 서버를 사용할 수도 있습니다.](https://ktor.io/servers/configuration.html#embedded-server)
- 로그백 의존성: 케이터는 로그 기록을 위해 로그백을 사용합니다.
- 케이터의 서버 테스트 의존성

추가로 필요한 기능이나 사양이 있다면 의존성을 추가하는 것으로 사용하실 수 있습니다.
또한 애초에 케이터 프로젝트 생성 마법사를 통해 만들지 않은 프로젝트라도 위의 의존성을 추가하면
기존 애플리케이션에 케이터를 연동할 수 있습니다.

저처럼 케이터 프로젝트 생성 마법사를 통해 프로젝트를 만드셨다면 의존성도 추가되어 있고 애플리케이션<sup>Application</sup>도 만들어져 있을겁니다.
아래는 이미 만들어진 애플리케이션 파일입니다.

```kotlin
// src/Application.kt
import io.ktor.application.*
import io.ktor.response.*
import io.ktor.request.*

fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

@Suppress("unused") // Referenced in application.conf
@kotlin.jvm.JvmOverloads
fun Application.module(testing: Boolean = false) {
}
```

뭐 별거 없는 것 같은데 `main`함수를 실행하면 서버가 실행됩니다. 서버의 포트 등의 설정은 [application.conf](https://ktor.io/servers/configuration.html#available-config)파일에 있습니다.

```kotlin
// resources/application.conf
ktor {
    deployment {
        port = 8080
        port = ${?PORT}
    }
    application {
        modules = [ com.ruewid.ApplicationKt.module ]
    }
}
```

자 이제 웹서버에 HTTP 종단점<sup>Endpoint</sup>을 하나 추가해보겠습니다.

```kotlin
import io.ktor.application.*
import io.ktor.response.*
import io.ktor.request.*
import io.ktor.routing.get // 추가됨
import io.ktor.routing.routing // 추가됨

fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

@Suppress("unused") // Referenced in application.conf
@kotlin.jvm.JvmOverloads
fun Application.module(testing: Boolean = false) {
    routing { // HTTP 요청 분기 설정 추가됨
        get("/") { // GET 방식의 '/' 경로 종단점 추가됨
            call.respondText("Hello, Ktor")
        }
    }
}
```

이제 다시 `main`함수를 호출하여 웹서버를 실행하고 HTTP 요청을 보내봅시다.

```
D:\Workspace\ktor-protobuf-test>curl http://localhost:8080/
Hello, Ktor
```

응답 잘 받으셨나요? 이 HTTP 종단점은 요청 데이터는 없지만 응답으로 글을 줍니다. 이제 프로토콜 버퍼를 응답으로 받아볼까요?

## 프로토콜 버퍼 시작하기

먼저 프로토콜 버퍼의 의존성을 추가합니다.

```kotlin
// build.gradle.kts
...

plugins {
    application
    kotlin("jvm") version "1.3.61"
    id("com.google.protobuf") version "0.8.11" // 추가됨
}
...
dependencies {
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version")
    implementation("io.ktor:ktor-server-netty:$ktor_version")
    implementation("ch.qos.logback:logback-classic:$logback_version")
    implementation("com.google.protobuf:protobuf-java:$protobuf_version") // 추가됨
    testImplementation("io.ktor:ktor-server-tests:$ktor_version")
}
```

먼저 그레이들<sup>Gradle</sup> 플러그인 의존성을 추가해줍니다. 이는 그레이들 빌드 스크립트에서 프로토콜 버퍼 소스셋<sup>SourceSets</sup> 경로 설정이나 `protobuf`블록을 사용하기 위함입니다. 다음 `protobuf-java` 의존성을 추가해줍니다. 현재 프로토콜 버퍼의 명세를 코틀린으로 컴파일할 수는 없기 때문에 자바에 의존합니다. 컴파일된 프로토콜 버퍼 클래스를 사용하기 위해 추가합니다.

그리고 프로토콜 버퍼의 명세 파일(`.proto`)이 어디에 위치할 지 지정해주어야 합니다. 

```kotlin
// build.gradle.kts
...
kotlin.sourceSets["main"].kotlin.srcDirs("src")
kotlin.sourceSets["test"].kotlin.srcDirs("test")

sourceSets["main"].resources.srcDirs("resources")
sourceSets["test"].resources.srcDirs("testresources")

sourceSets["main"].proto.srcDirs("protobuf") // 추가됨
```

인텔리제이의 케이터 프로젝트 생성 마법사에 의해 생성된 이 프로젝트에서 프로토콜 버퍼 명세가 놓일 곳이 마땅치 않은 것 같아 저는 `src` 디렉토리와 같은 수준에 `protobuf` 디렉토리를 만들고 이곳에 `.proto` 명세 파일을 저장하겠습니다.

그리고 명세를 한번 작성해 봅시다.

```protobuf
// protobuf/responses.proto
syntax = "proto3";

message Article {
    string id = 1;
    string title = 2;
    string body = 3;
    int32 viewCount = 4;
}
```

이제 그레이들의 `build` 작업을 실행시키면 `Responses.java`파일이 생성되는 것을 볼 수 있습니다.

```
gradlew build
```

`Responses.java`파일이 어디에 생겼나요? 다음의 경로를 따라가시면 확인할 수 있습니다.

```
build/generated/source/proto/main/java
```

이는 프로토콜 버퍼 컴파일러가 컴파일 결과물을 출력하는 기본 위치입니다. [물론 이 위치는 변경이 가능합니다.](https://github.com/google/protobuf-gradle-plugin#change-where-files-are-generated)

이제 `.java` 소스 파일도 만들어 졌겠다, 응답으로 보내볼까요?

```kotlin
// Application.kt
...
fun Application.module(testing: Boolean = false) {
    routing {
        get("/") {
            call.respondText("Hello, Ktor")
        }
        // 아래 get 블록 추가됨
        get("/article") {

            val article = Responses.Article.newBuilder()
                .setId("ID")
                .setTitle("TITLE")
                .setBody("BODY")
                .setViewCount(8)
                .build()
            call.respondBytes(article.toByteArray(), ContentType("application", "x-protobuf"))
        }
    }
}
```

프로토콜 버퍼의 자바 버전은 newBuilder()를 이용하여 객체를 빌드합니다. 그런데 `Responses`부터 임포트<sup>Import</sup>가 되지 않을 겁니다. 클래스패스<sup>Classpath</sup>를 지정해주지 않으면 사용할 수 없습니다. 다시 그레이들 빌드 스크립트로 가볼까요?

```kotlin
kotlin.sourceSets["main"].kotlin.srcDirs("src")
kotlin.sourceSets["test"].kotlin.srcDirs("test")

sourceSets["main"].resources.srcDirs("resources")
sourceSets["test"].resources.srcDirs("testresources")

sourceSets["main"].proto.srcDirs("protobuf")
sourceSets["main"].java.srcDirs("build/generated/source/proto/main/java") // 추가됨
```

컴파일된 `.java`파일이 있는 경로를 소스 디렉토리로 추가해주면 됩니다.

다시 아까 코드로 가면 임포트가 가능해졌을 겁니다. 이 서버를 띄워봅시다. 그리고 추가된 종단점을 호출해볼까요?

```
D:\Workspace\ktor-protobuf-test>curl http://localhost:8080/article

╗ID║TITLE╝BODY
```

뭔가 알아볼 듯 알아보기 힘든 결과가 나왔네요. 
자 이것을 `Responses.Article`로 받아봅시다. 
이를 위해 테스트 코드를 작성할 겁니다. 
`src`와 같은 수준에 `test`라는 디렉토리를 만들고 다음과 같은 테스트를 작성해 봅시다.

```kotlin
// test/HttpProtobufTest.kt
import com.ruewid.module
import io.ktor.http.HttpHeaders
import io.ktor.http.HttpMethod
import io.ktor.server.testing.handleRequest
import io.ktor.server.testing.withTestApplication
import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.fail

class HttpProtobufTest {
    @Test
    fun respondProtobuf(): Unit = withTestApplication({
        module(true) // Application.kt의 module()을 호출함
    }) {
        handleRequest(HttpMethod.Get, "/article") // 모의 요청을 전송함
            .apply {
                if (response.headers[HttpHeaders.ContentType] != "application/x-protobuf") { // Content-Type이 올바른지 확인
                    fail()
                }
                with(Responses.Article.parseFrom(response.byteContent)) { // 역직렬화
                    assertEquals("ID", id) // 역직렬화된 각각의 속성 검증
                    assertEquals("TITLE", title)
                    assertEquals("BODY", body)
                    assertEquals(8, viewCount)
                }
            }
    }
}
```

모의 요청을 전송하면 실제 서버가 네트워크를 통해 요청을 받은 것처럼 엮인 코드들이 실행되고 응답을 반환합니다.
반환 받은 응답의 헤더<sup>Header</sup>와 페이로드<sup>Payload</sup>가 예상한 값인지 검증합니다.
당연하게도 이 테스트는 성공입니다.

이번엔 서버에서 프로토콜 버퍼를 요청 페이로드로 받아볼까요?

```kotlin
// Application.kt
...
fun Application.module(testing: Boolean = false) {
    routing {
        ...
        post("/article") {
            val received = call.receive<ByteArray>()
            val article = Responses.Article.parseFrom(received) // 요청으로 받은 페이로드를 객체화
            call.respondBytes(article.toByteArray(), ContentType("application", "x-protobuf")) // 이를 다시 응답으로 전송
        }
    }
}
```

이번엔 요청에 프로토콜 버퍼를 실어 보내면 서버는 그것을 객체화했다가 다시 응답에 실어 보냅니다. 이것은 어떻게 테스트해 볼 수 있을까요?

```kotlin
// test/HttpProtobufTest.kt
...
    @Test
    fun postProtobuf(): Unit = withTestApplication({
        module(true)
    }) {
        handleRequest(HttpMethod.Post, "/article") {
            addHeader(HttpHeaders.ContentType, "application/x-protobuf") // 요청에 헤더를 추가함
            setBody(Responses.Article.newBuilder() // 요청에 페이로드를 추가함
                .setId("POST_ID")
                .setTitle("POST_TITLE")
                .setBody("POST_BODY")
                .setViewCount(88)
                .build()
                .toByteArray())
        }
            .apply {
                if (response.headers[HttpHeaders.ContentType] != "application/x-protobuf") {
                    fail()
                }
                with(Responses.Article.parseFrom(response.byteContent)) {
                    assertEquals("POST_ID", id) // 요청으로 보냈던 프로토콜 버퍼의 속성들과 응답을 검증
                    assertEquals("POST_TITLE", title)
                    assertEquals("POST_BODY", body)
                    assertEquals(88, viewCount)
                }
            }
    }
}
```

보낸 값이 응답에 똑같이 들어있는지 확인합니다. 당연히 성공이겠죠?

## 컨텐츠 협상<sup>Content Negotiation</sup>

우리는 이제 프로토콜 버퍼를 이용하여 HTTP API의 요청과 응답으로 데이터를 주고 받을 수 있게 되었습니다.
하지만 이미 웹 서버 프레임워크로 개발해 보신 분들은 조금 의아해 하실 수도 있습니다.
모든 HTTP 종단점에서 ByteArray를 처리해야 할까요?
항상 ByteArray를 받아서 객체화하고, 객체를 ByteArray로 변환한 후에 전송해야 할까요? 이는 비용입니다.
매 종단점마다 지루한 코드가 삽입됩니다. 이를 개선하기 위해 우리는 케이터의 컨텐츠 협상을 사용할 수 있습니다.
케이터의 컨텐츠 협상이란 컨텐츠형<sup>Content Type</sup>에 따라 네트워크로 드나드는 데이터를
애플리케이션에서 원하는 자료형으로 변환해주는 것입니다.

먼저 기능을 사용하기 위해 케이터에게 컨텐츠 협상을 사용하겠다고 알려주어야 합니다.

```kotlin
// src/Application.kt
fun Application.module(testing: Boolean = false) {
    install(ContentNegotiation) // 추가됨
    routing {
        ...
    }
}
```

그리고 프로토콜 버퍼의 컨텐츠 협상을 설정합니다.

```kotlin
// src/Application.kt
fun Application.module(testing: Boolean = false) {
    install(ContentNegotiation) {
        register(ContentType("application", "x-protobuf"), ProtobufConverter()) // 추가됨
    }
    routing {
        ...
    }
}
```

`ProtobufConverter`가 뭐죠? 쉽게, 네트워크로 드나드는 데이터를 변환해 줄 클래스가 필요합니다.
`ProtobufConverter`는 직접 작성해야 하며 저는 [케이터의 예제](https://ktor.io/servers/features/content-negotiation.html#content-converter)를 참고하여 작성하였습니다.

```kotlin
// src/ProtobufConverter.kt
import com.google.protobuf.MessageLite
import io.ktor.application.ApplicationCall
import io.ktor.features.ContentConverter
import io.ktor.http.ContentType
import io.ktor.http.content.ByteArrayContent
import io.ktor.request.ApplicationReceiveRequest
import io.ktor.util.pipeline.PipelineContext
import io.ktor.utils.io.ByteReadChannel
import io.ktor.utils.io.core.readBytes
import io.ktor.utils.io.readRemaining

class ProtobufConverter: ContentConverter { // ContentConverter를 구현해야 합니다.
    override suspend fun convertForReceive(context: PipelineContext<ApplicationReceiveRequest, ApplicationCall>): Any? {
        val request = context.subject
        val channel = request.value as? ByteReadChannel ?: return null
        return channel.readRemaining().readBytes() // 페이로드를 획득합니다.
            .let {
                /* 지금은 Article만 만들었지만 프로토콜 버퍼가 늘어나게 되면
                 * 결국 특정 타입만 변환하는 코드는 살아남지 못합니다.
                 * 어떤 타입이든 애플리케이션이 원하는 자료형(request.type)으로
                 * 변환할 수 있도록 리플렉션을 활용합니다.*/
                request.type.javaObjectType.getMethod("parseFrom", ByteArray::class.java).invoke(null, it)
            }
    }

    override suspend fun convertForSend(
        context: PipelineContext<Any, ApplicationCall>,
        contentType: ContentType,
        value: Any
    ): Any? {
        if (value !is MessageLite) {
            return null; // null을 반환하면 케이터는 다른 변환기로 변환을 시도합니다. (이것이 컨텐츠 협상)
        }
        /* 모든 프로토콜 버퍼 객체는 MessageLite를 구현하고 있고
         * toByteArray()는 MessageLite의 인터페이스이므로
         * value를 MessageLite로 형변환하여 toByteArray()를 호출합니다.*/
        return ByteArrayContent(value.toByteArray(), contentType)
    }
    }
}
```

이렇게 만들어진 변환기<sup>Converter</sup>를 어떻게 쓸 수 있죠? 아까 보셨듯 컨텐츠 협상에 설정할 수 있습니다.

```kotlin
// src/Application.kt
fun Application.module(testing: Boolean = false) {
    install(ContentNegotiation) {
        // 해당 컨텐츠형은 설정한 변환기를 이용
        register(ContentType("application", "x-protobuf"), ProtobufConverter())
    }
    routing {
        ...
    }
}
```

그렇다면 기존 종단점은 어떻게 바뀔까요?

```kotlin
// src/Application.kt
    ...
        get("/article") {
            val article = Responses.Article.newBuilder()
                .setId("ID")
                .setTitle("TITLE")
                .setBody("BODY")
                .setViewCount(8)
                .build()
//            call.respondBytes(article.toByteArray(), ContentType("application", "x-protobuf"))
            call.respond(article) // 객체를 반환합니다.
        }
        post("/article") {
//            val received = call.receive<ByteArray>()
//            val article = Responses.Article.parseFrom(received)
//            call.respondBytes(article.toByteArray(), ContentType("application", "x-protobuf"))
            val article = call.receive<Responses.Article>() // 객체로 받습니다.
            call.respond(article)
        }
    ...
```

이제는 ByteArray를 더 이상 신경쓰지 않아도 됩니다. 

## 마치며

아마 위의 예제보다 더 좋은 방법으로 HTTP를 통해 프로토콜 버퍼를 주고 받는 방법은 얼마든지 있을 겁니다.
다만, 이 포스트에서는 케이터(또는 다른 웹 서버 프레임워크)에서 HTTP를 통해 데이터를 전달하는 여러 방법에
프로토콜 버퍼도 있다는 것을 전달하고자 하였습니다.

주고 받는 데이터의 정의는 프로토콜 버퍼에 의해 정해지므로 프로토콜 버퍼 명세만 공유하면 서버와 클라이언트가
서로 다른 데이터 모델을 사용하는 문제는 없을 겁니다.

또한 서로 다른 매퍼(또는 파서 또는 직렬자<sup>Serializer</sup>)를 사용함으로 인한 기능 격차 문제도 해소할 수 있습니다.
예를 들어 제이슨을 자료형으로 사용했을 때 서버측에서 사용하는 제이슨 라이브러리로 쉽게 변환되는 구성 또는 상속 받은 데이터가
클라이언트에서 변환에 애를 먹을 수 있습니다. 프로토콜 버퍼는 데이터 명세는 물론 변환 코드까지 생성해 주기 때문에
이런 문제는 쉽게 해결할 수 있다고 생각합니다.

저도 아직 실전에서는 써보지 못했습니다. 모쪼록 HTTP를 인터페이스로 하는 애플리케이션에서 프로토콜 버퍼 채택을 고민하는 분들께
조금이나마 도움이 됐으면 합니다.

읽어주셔서 고맙습니다.