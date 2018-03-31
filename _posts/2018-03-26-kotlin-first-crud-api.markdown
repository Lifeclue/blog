---
layout: post
title: "Kotlin 프로젝트에 첫 CRUD API 추가하기"
date: "2018-03-26 22:57:50 +0900"
category: kotlin
comments: true
toc: true
tags: 코틀린 Kotlin 스프링 Spring 스프링부트 SpringBoot RestController HTTP
---

이제 Famphlet 프로젝트에 읽고, 만들고, 바꾸고, 지우는 API를 추가해 보겠습니다. 이 프로젝트는 기본적으로 즐겨찾기를 관리하는 프로젝트이기 때문에 이러한 API가 기본으로 제공될 필요가 있습니다. 설계 없이 프로젝트를 막 진행하는 감이 있긴 합니다만 어쨌든 기본 기능이니 건너뛰고 구현해 봅시다.  
(혹시 만들고자 하는 것이 있다면, 여러분은 반드시 설계를 먼저 진행하세요. 꼭입니다. 꼭이요!)

# HTTP API에 대한 이해

API를 만들기 전에 HTTP API에 대해 이해해야 할 부분을 짚고 넘어가겠습니다. HTTP API란 HTTP를 통해 애플리케이션과 소통하는 방식입니다. 애플리케이션과는 여러 방식으로 소통할 수 있지만 여기서는 HTTP로 하겠다는 것입니다. HTTP로 읽고 만들고, 바꾸고, 지우는 일을 하기 위해서는 HTTP Request Methods에 대해 알아둘 필요가 있습니다.

## HTTP Request Methods

HTTP 요청 방식이란 HTTP를 어떤 목적으로 요청하는가를 명시적으로 정의한 것입니다. [RFC 2616 문서의 5.1.1 섹션](https://tools.ietf.org/html/rfc2616#page-36)을 보면 HTTP/1.1버전에서 처음 정의한 HTTP 요청 방식에 대해 서술하고 있습니다. 요청 방식은 다음과 같습니다.

- OPTIONS
- GET
- HEAD
- POST
- PUT
- DELETE
- TRACE
- CONNECT

각각의 요청 방식에 대한 설명은 [Mozilla의 HTTP request methods](https://developer.mozilla.org/ko/docs/Web/HTTP/Methods)를 참고해주세요. Mozilla의 문서를 보시면 PATCH라는 것이 더 있을 겁니다. PATCH는 이후에 추가된 요청 방식으로 그 명세는 [RFC 5789 문서](https://tools.ietf.org/html/rfc5789)를 참고해보실 수 있습니다. 명세의 서두에는 대략 이렇게 써있습니다.

>  
HTTP를 확장한 어떤 애플리케이션은 자원을 부분적으로 수정할 수 있어야 합니다. 기존의 HTTP PUT 방식은 자원을 대체하는 것만 허용하였습니다. 이 문서에서는 존재하는 HTTP 자원을 수정할 수 있는 PATCH라는 새로운 HTTP 방식을 제안합니다.

네, 그렇습니다. 자원의 일부를 수정하기 위해서는 PATCH 방식을 사용하는 것이 옳아 보입니다. 여기서 우리가 사용해 볼 방식은 네 개입니다.

- GET: 즐겨찾기를 조회합니다.
- POST: 즐겨찾기를 추가합니다.
- PATCH: 즐겨찾기를 수정합니다.
- DELTE: 즐겨찾기를 삭제합니다.

사실 GET과 POST는 너무 유명해서 많이 듣거나 보셨을 겁니다. 그리고 API를 GET과 POST만으로도 충분히 만들 수 있는 것이 사실입니다. 그러나 클라이언트는 HTTP 요청 방식을 보고 결과를 기대하므로 우리는 서로 약속한 방식으로, 약속된 동작을 만들어 보도록 합시다.

## HTTP API vs. RESTful API

같은 듯 다른 용어이지요? 우리가 만들어 볼 것은 HTTP API입니다. URI도 아름답게 짜고 HTTP 요청 방식의 의미도 잘 따라서 만들 건데 왜 RESTful API가 아니냐는 물음이 생길 수도 있습니다. 넓은 의미에서 RESTful API도 HTTP API입니다. REST라는 것이 HTTP 위에서 태동했기 때문이지요. 그러나 REST의 철학은 더욱 명료합니다. URI가 그 자원을 잘 설명했다거나 HTTP 요청 방식을 의미에 맞게 잘 사용했다는 정도로는 REST 축에 끼지도 못합니다. RESTful API를 만들고 싶으신 분은 [이응준님의 REST 슬라이드](http://slides.com/eungjun/rest) 일독을 권장합니다.

# HTTP API 구현

그럼 위에서 찾아본 지식들을 얹어서 API를 직접 구현해 봅시다.

## 클라이언트와 서버의 약속 - Interface

일단 클라이언트와 서버가 통신할 인터페이스부터 만들어 봅시다. Controller가 아닌 Interface를 만드는 겁니다. 이게 무슨 말이냐구요? 프로젝트 내에서도 서버와 클라이언트가 존재합니다. 메서드를 호출하는 쪽(클라이언트)에서는 어떠한 결과를 기대하며 메서드를 호출할 겁니다. 호출 되는 (서버 쪽의)메서드는 호출됨과 동시에 호출한 클라이언트가 만족할 수 있도록 결과를 만들어 반환(return)합니다. 이 둘간의 약속이 필요합니다.

우리는 [Kotlin Interfaces 문서](https://kotlinlang.org/docs/reference/interfaces.html)를 참고하여 인터페이스를 만들어 볼 겁니다. 만들고자 하는 디렉토리에 오른쪽 버튼을 누르고 `New` 메뉴를 확장하여 Kotlin File/Class를 선택합시다. (Alt + Insert, Command + N)

![코틀린 인터페이스 생성 대화창]({{"/assets/image/kotlin-famphlet/first-api/new-site-item-service-interface.PNG" | absolute_url}})

일단 모든 소스 코드의 package는 다음과 같습니다. 이 인터페이스에도 첫 줄에 이 패키지가 들어갑니다.

{% highlight kotlin %}
package com.lifeclue.blog.famphlet
{% endhighlight %}

만들어 볼 인터페이스는 즐겨찾기를 추가하거나, 조회하거나, 수정하거나, 삭제할 수 있는 소통 수단이지요? 이러한 사실에 근거하여 아래와 같이 짜보았습니다.

{% highlight kotlin %}
interface SiteItemService {
    fun createSiteItem(itemName: String, siteUri: String): SiteItem
    fun getSiteItem(id: Long): SiteItem
    fun getSiteItems(): List<SiteItem>
    fun modifySiteItem(id: Long, itemName: String, siteUri: String): SiteItem
    fun removeSiteItem(id: Long)
}
{% endhighlight %}

짜다 보니 `SiteItem`이 필요해졌습니다. 아마 `createSiteItem()`메서드의 인자를 참고하여 쉽게 구현할 수 있을 것 같습니다.
(하지만 여러분이 프로젝트를 진행하게 된다면 짜다 보니 필요해서 만들지 마시고, 반드시 모델링부터 하세요! 꼭!)

## 객체 설계 - Class

인텔리제이에서 방금 짠 인터페이스를 보면 `SiteItem`이 모두 빨간 색일 겁니다. 이곳에 커서를 놓고 Alt + Enter를 눌러보죠. 현재 이곳에 오류가 있고 오류를 해결하기 위한 방안들이 나열됩니다. 여기서 `Create class 'SiteItem'`을 누르면 클래스를 빠르게 만들 수 있습니다. 다음 단계에서 `Choose class container`라는 선택지가 나오는데 저는 개별 파일로 작성할 예정이므로 `Extract to separate file`을 골랐습니다.

![빠른 클래스 생성]({{"/assets/image/kotlin-famphlet/first-api/quick-create-class.PNG" | absolute_url}})

![생성될 클래스 위치 선정]({{"/assets/image/kotlin-famphlet/first-api/choose-class-container.PNG" | absolute_url}})

{% highlight kotlin %}
// 축약 코드
class SiteItem (var itemName: String, var siteUri: String) {
    var id: Long = 0
}
{% endhighlight %}

SiteItem은 쉽게 만들었네요. 사실 SiteItem을 조직화할 요소를 같이 만들어야 하지만 일단 없는 채로 진행하겠습니다. 여러개의 SiteItem을 묶어줄 DirectoryItem 같은 조직 단위 요소가 나중에 추가될 겁니다.

SiteItem을 보시면 생성자가 재밌게 생긴 것을 보실 수 있습니다. 위 코드는 축약된 형태로, 사실은 다음과 같은 형태와 같습니다.

{% highlight kotlin %}
// 본 코드
class SiteItem {
    var id: Long = 0
    var itemName: String
    var siteUri: String

    constructor(itemName: String, siteUri: String) {
        this.itemName = itemName
        this.siteUri = siteUri
    }
}
{% endhighlight %}

[Kotlin Constructors 문서](https://kotlinlang.org/docs/reference/classes.html#constructors)에 따르면, 생성자에 어노테이션이나 가시성 제어자(Visibility Modifiers)를 생략할 수 있을 때 `축약 코드`처럼 쓸 수 있다고 합니다. constructor에 @Autowired 등의 어노테이션이나 private 등의 가시성 수정자를 붙이려면 `본 코드`처럼 명시적으로 constructor 키워드를 쓰고 그 앞에 붙이면 됩니다.

`축약 코드`로 작성하면 생성자와 필드(Member variables)가 자동으로 선언되고 필드 초기화 코드가 선언된 생성자에 암묵적으로 생성됩니다. `본 코드` 처럼 말이죠.

(Getter, Setter 등의 접근자에 대해 궁금하신가요? 이후에 설명하겠습니다.)

## 사용자와 애플리케이션의 인터페이스 - Controller

자, `SiteItem`을 추가함으로써 인터페이스가 완성되었습니다. 아마도 이 인터페이스의 클라이언트는 `Controller`가 될 것입니다. 인터페이스가 있으니 서버나 클라이언트 중 아무거나 만들어도 상관 없을 것 같습니다만 일단 클라이언트를 만들어 보겠습니다.

{% highlight kotlin %}
@RestController
class SiteItemController(val siteItemService: SiteItemService) {

    @GetMapping("/sites")
    fun getSites() = responseOf(siteItemService.getSiteItems())

    @GetMapping("/sites/{id}")
    fun getSite(@PathVariable id:Long) = responseOf(siteItemService.getSiteItem(id))

    @PostMapping("/sites")
    fun createSite(@RequestBody payload: SiteRequest) = responseOf(siteItemService.createSiteItem(payload.name, payload.uri))

    @PatchMapping("/sites/{id}")
    fun modifySite(@PathVariable id:Long, @RequestBody payload: SiteRequest)
            = responseOf(siteItemService.modifySiteItem(id, payload.name, payload.uri))

    @DeleteMapping("/sites/{id}")
    fun removeSite(@PathVariable id:Long) = responseOf(siteItemService.removeSiteItem(id))

    private fun responseOf(newSite: SiteItem): SiteResponse {
        return SiteResponse(newSite.id, newSite.itemName, newSite.siteUri)
    }

    private fun responseOf(newSite: List<SiteItem>): SiteListResponse {
        return SiteListResponse(
                newSite.stream()
                .map(this::responseOf)
                .collect(Collectors.toList())
        )
    }
}
{% endhighlight %}

HTTP 요청을 처음 맞이하는 곳이 Controller입니다. 여기서 각 요청의 URI를 정의합니다. @...Mapping 이라는 어노테이션이 보이시나요? 저것이 HTTP 요청 방식을 정의하는 어노테이션입니다. 예를 들어  `http://.../sites/1/`으로 GET 요청을 날리면 즐겨찾기가 조회되겠지만 DELETE 요청을 날리면 즐겨찾기가 지워지는 겁니다.

private 함수를 보시면 SiteItem을 SiteResponse로 매핑하는 것을 볼 수 있습니다. 이는 애플리케이션에서 사용하는 객체와 사용자에게 제공되는 객체를 분리하기 위함입니다. 이것으로 내부 로직이 변경되더라도 사용자에게 제공되는 인터페이스에는 변경이 필요 없게 됩니다. (물론 불가피하게 같이 변경할 일이 생길 수도 있습니다.)

{% highlight kotlin %}
data class SiteResponse(val id:Long, val itemName: String, val siteUri: String)
{% endhighlight %}

Data Class는 매우 유용합니다. 클래스에 `data` 키워드를 붙여주면 컴파일러가 다음의 일들을 자동으로 해줍니다.

- equals()/hashCode() 함수 생성
- "User(name=John, age=42)" 형식의 toString() 함수 생성
- 클래스의 속성이 선언된 순서대로 그에 대응되는 componentN() 함수 생성
- copy() 함수 생성

데이터의 전달이 주 목적인 클래스는 데이터 클래스로 정의하여 사용합시다.

HTTP 인터페이스 관점에서 클라이언트는 사용자입니다. 사용자는 브라우저라는 훌륭한 구현체가 있으므로 그 클라이언트는 안만들어도 되겠네요. 이제 다시 돌아와서 `SiteItemService`의 서버를 만들어 봅시다.

## 비지니스 로직

눈치 채셨습니까? 우린 앞서 논의한 인터페이스가 실제로 어떻게 동작하는지는 아직 맛도 보지 못했습니다. 서버 코드에는 즐겨찾기를 만들고, 저장하고, 혹은 조회하여 변경하고, 지우는 코드가 들어갈 것입니다. 보통 이런 작업들을 저는 비지니스 로직이라고 합니다. 로직이 들어갈 클래스를 작성해 봅시다.

{% highlight kotlin %}
class StandardSiteItemService: SiteItemService {
}
{% endhighlight %}

클래스를 생성하면 `class StandardSiteItemService`에 빨간 밑줄이 쳐져 있을 겁니다. 역시 이곳에서 Alt + Enter를 누르면 오류를 어떻게 해결 할지 선택할 수 있습니다. (Intellij 만세)

![인터페이스 멤버 구현]({{"/assets/image/kotlin-famphlet/first-api/implement-members.PNG" | absolute_url}})

구현할 인터페이스 멤버 선택 화면이 나오면 모두 선택하세요.

![구현할 인터페이스 멤버 선택]({{"/assets/image/kotlin-famphlet/first-api/choose-members-will-implement.PNG" | absolute_url}})

이제 구현해야 할 멤버 함수들에 코드를 채워넣어 봅시다.

{% highlight kotlin %}
class StandardSiteItemService(val itemStore: SiteItemStore) : SiteItemService {
    override fun createSiteItem(itemName: String, siteUri: String): SiteItem {
        val item = SiteItem(itemName, siteUri)
        itemStore.save(item)
        return item
    }

    override fun getSiteItem(id: Long): SiteItem {
        return itemStore.findById(id)
    }

    override fun getSiteItems(): List<SiteItem> {
        return itemStore.findAll()
    }

    override fun modifySiteItem(id: Long, itemName: String, siteUri: String): SiteItem {
        val item: SiteItem = itemStore.findById(id)
        item.itemName = itemName
        item.siteUri = siteUri
        itemStore.save(item)
        return item
    }

    override fun removeSiteItem(id: Long): SiteItem {
        return itemStore.deleteById(id)
    }
}
{% endhighlight %}

코드 자체는 간단합니다. 요청을 수신하면 요청 인자를 전달하여 SiteItem 객체를 생성하고 저장소에 저장한 후에 반환합니다. 혹은 ID로 SiteItem을 검색하여 반환하기도 하고 ID에 해당하는 SiteItem 객체의 내용을 요청 인자로 변경하여 저장하기도 합니다. 모든 코드는 매우 단순하며 대부분 저장소에 넣거나 빼는 것으로 이루어집니다.

## 저장소

코드를 채워 넣다 보니 SiteItem 객체를 저장할 저장소가 필요해 보입니다. 어디에 저장할 지 아직 정하지 않기도 했고 추후에 저장소가 바뀔 우려도 있으므로 저장소와 소통하는 코드는 인터페이스를 고려해서 작성했습니다. 추후에 [Spring Data의 Repository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)를 사용할 것을 고려해 비슷하게 만든 것이 보이시나요?

`StandardSiteItemService`를 작성하고 나면 `SiteItemStore`와 관련 함수 호출부에서 오류가 발견될 것입니다. 아직 없는 소스이기 때문이지요. 역시 Alt + Enter를 누르시면 인터페이스나 함수를 손쉽게 만드실 수 있습니다.

{% highlight kotlin %}
interface SiteItemStore {
    fun save(item: SiteItem)
    fun findById(id: Long): SiteItem
    fun findAll(): List<SiteItem>
    fun deleteById(id: Long): SiteItem
}
{% endhighlight %}

인터페이스도 만들었으니 구현체를 만들어볼까요? 당장 저장할 곳이 마땅치 않으니 메모리에 저장해 봅시다. 이제 감을 잡으신 분들도 계시겠지만 interface 이름인 `SiteItemStore`에 커서를 두고 역시 Alt + Enter를 누르시면 `implement interface`메뉴를 바로 쓰실 수 있습니다.

{% highlight kotlin %}
class InMemorySiteItemStore : SiteItemStore {
    val id: AtomicLong = AtomicLong(1)
    val storage: MutableMap<Long, SiteItem> = mutableMapOf()
    override fun save(item: SiteItem) {
        if (item.id < 1) {
            item.id = id.getAndIncrement()
        }
        storage[item.id] = item
    }

    override fun findById(id: Long): SiteItem? {
        return storage[id]
    }

    override fun findAll(): List<SiteItem> {
        return storage.values.toList()
    }

    override fun deleteById(id: Long): SiteItem? {
        return storage.remove(id)
    }
}
{% endhighlight %}

인터페이스를 구현하면서 인터페이스를 수정할 일이 생겼습니다. 바로 `findById()`와 `deleteById()`의 응답입니다. 두 함수는 ID를 이용해 조회하거나 지우기 때문에 저장소에 없는 ID가 요쳥되는 일이 분명 있을 것입니다. 그럴 땐 null을 이용해야 하는데 함수의 자료형에 `?`를 붙이는 것은 [코틀린이 null을 취급하는 한 방법](https://kotlinlang.org/docs/reference/basic-syntax.html#using-nullable-values-and-checking-for-null)입니다. 코틀린의 모든 자료형은 기본으로 널 불가형(Not null)입니다. 그러나 `?`가 붙으면 널 가능형(Nullable)입니다. 코틀린은 널 참조를 피하기 위해 많은 고민을 했습니다. 고민의 결과는 [코틀린의 Null 안전성](https://kotlinlang.org/docs/reference/null-safety.html) 문서를 참고하세요. 일독 권합니다. 내용을 대충 요약하자면 다음과 같습니다.

>  
코틀린의 자료형 체계는 널 참조의 위험을 제거하는 것을 목표로 합니다. (a.k.a [The Billion Dollar Mistake](https://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions))  
그러나 우리는 결국 널 참조를 사용해야 하는 상황에 직면하게 됩니다. 이 문서에서는 널 참조를 다루는 몇가지 방법을 소개합니다.  
첫 번째 방법으로는, 널 가능형을 참조하기 전에 널 검사를 하는 것입니다.  
다음으로는 Safe call 연산자(`?.`)를 사용하는 것입니다.  
자매 연산자인 엘비스 연산자(`?:`)도 알아두시면 유용하게 사용하실 수 있습니다.  
세 번째 방법은 NPE 성애자를 위한 방법으로, non-null assertion 연산자(!!)를 사용하는 것입니다.

코틀린은 null을 웬만해서는 허용하지 않겠다는 것이 이 문서의 골자입니다.

널 참조 덕분에 인터페이스가 변경되었네요! 이 인터페이스를 사용하고 있는 `StandardSiteItemService` 클래스에도 변화가 불가피합니다. 우선 널 가능형을 반환하는 메서드를 사용하는 쪽을 수정해주어야 합니다.

{% highlight kotlin %}
    override fun getSiteItem(id: Long): SiteItem {
        return itemStore.findById(id) ?: throw IllegalArgumentException("${id}에 해당하는 즐겨찾기를 찾을 수 없습니다.")
    }
{% endhighlight %}

엘비스 연산자를 사용해보았습니다. `findById()`의 반환값이 널이 아니면 그 값을 그대로 쓰고 아니면 예외를 던지는 코드가 되었습니다. 흥미로운 것이 하나 있는데 문자열 조합을 위해 `id + "..."`같은 구문을 쓰는 것이 아니라 [문자열 템플릿](https://kotlinlang.org/docs/reference/basic-types.html#string-templates)을 사용한 것입니다. 변수 앞 뒤로 변수명에 쓸 수 없는 문자, 즉 변수가 구분된다면 중괄호는 생략할 수 있습니다. 예를 들면 `입력하신 '$id'는 존재하지 않습니다.`처럼 말이죠.

위의 코드를 조금 더 설명하자면, 사실 모르는 ID를 조회한다는 것은 있어선 안되는 상황입니다. 그러므로 이 서비스는 널일 때 `요청 자체는 옳고 모든 동작도 문제 없이 완료했지만 결과가 비었다`는 뉘앙스 보다는 `요청이 잘못되었다`는 뉘앙스로 응답하기로 하였습니다.

다음 코드를 봅시다.

{% highlight kotlin %}
    override fun modifySiteItem(id: Long, itemName: String, siteUri: String): SiteItem {
        val item: SiteItem = itemStore.findById(id) ?: throw notFound(id)
        item.itemName = itemName
        // ...
        return item
    }

    override fun removeSiteItem(id: Long): SiteItem {
        return itemStore.deleteById(id) ?: throw notFound(id)
    }

    private fun notFound(id: Long) = IllegalArgumentException("{$id}에 해당하는 즐겨찾기를 찾을 수 없습니다.")
{% endhighlight %}

즐겨찾기 수정, 삭제 함수도 피해갈 수 없습니다. 여기서는 예외를 생성하는 코드가 이전의 즐겨찾기 조회 함수와 중복되기 때문에 팩토리 함수를 정의하여 사용하였습니다. 상수로 정의해서 사용할 수도 있지만 요청마다 ID가 다르므로 팩토리 함수를 정의하였습니다.

이제 얼추 완성이 되어가고 있군요. 이 시점에 빌드를 해보면 문제 없이 성공할 겁니다. 한 번 실행해 볼까요? 애플리케이션을 실행하면 아마 다음과 같은 오류를 만나게 되실 겁니다.

>Parameter 0 of constructor in com.lifeclue.blog.famphlet.SiteItemController required a bean of type 'com.lifeclue.blog.famphlet.SiteItemService' that could not be found.

`SiteItemController` 생성자의 0번째 인자로 SiteItemService가 필요하지만 찾을 수 없다는 뜻입니다. 그러면서 스프링이 해결책을 제시해 줍니다.

>Consider defining a bean of type 'com.lifeclue.blog.famphlet.SiteItemService' in your configuration.

설정을 통해 `SiteItemService`형의 빈을 정의해보지 않겠니? 라고 상냥하게 안내해주네요. 클래스를 빈으로 정의하면 스프링이 그 클래스(빈) 객체의 생명 주기를 관리해 줍니다. 클래스를 빈으로 등록하는 방법은 몇가지가 있는데, 예전에는 xml 설정으로 주로 등록하였습니다. 아래와 같은 형태였지요.

{% highlight xml %}
<bean id="siteItemService" class="com.lifeclue.blog.famphlet.SiteItemController" />
{% endhighlight %}

지금 우리는 어노테이션을 붙여서 빈으로 정의하고 있습니다.

{% highlight kotlin %}
@RestController
class SiteItemController(val siteItemService: SiteItemService) {
  // ...
}
{% endhighlight %}

`SiteItemController`의 위에 `@RestController`어노테이션이 붙었기 때문에 스프링이 이를 감지(Scan)하고 객체를 생성하여 객체가 소멸될 때까지 관리합니다. 다른 방법으로는 설정 클래스에서 등록하는 방법입니다.

{% highlight kotlin %}
@Configuration
class ApplicationConfig {
    @Bean
    fun siteItemController(siteItemService: SiteItemService) : SiteItemController {
        return SiteItemController(siteItemService)
    }
}
{% endhighlight %}

우리는 손쉬운 어노테이션으로 빈 정의를 해보도록 하겠습니다. 일단 스프링의 오류를 보면 `SiteItemController`를 생성하기 위해 `SiteItemService`의 객체가 필요한데, 객체를 찾을 수 없다고 하니 `SiteItemService`의 객체를 스프링이 쓸 수 있도록 빈으로 정의하겠습니다.

{% highlight kotlin %}
@Service
class StandardSiteItemService(val itemStore: SiteItemStore) : SiteItemService {
  // ...
}
{% endhighlight %}

@Service 어노테이션은 이 클래스를 서비스 역할을 하는 빈으로 정의한다는 뜻입니다. [스프링 @Service 어노테이션 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html)를 보면 서비스에 대한 정의를 찾아볼 수 있습니다. 또한 [위키피디아 Domain Driven Design 문서의 Service절](https://en.wikipedia.org/wiki/Domain-driven_design)도 꼭 참고하시기 바랍니다.

`SiteItemService`가 그 의미를 다 채울 수 있는 역할을 당장은 못할 수도 있습니다. 그러나 일단 지금은 Service로 쓰겠습니다. 나중에 이름과 역할이 맞지 않다고 판단되면 바꾸면 되니까요.

`StandardSiteItemService`에 어노테이션을 붙이면서 눈치를 채셨는지 모르겠습니다만 여기에도 생성자가 있습니다. @Service 어노테이션으로 이 클래스는 빈 객체가 되기 때문에 스프링에 의해 객체가 생성이 되고 스프링은 그 객체를 관리할겁니다. 그런데 생성자에 스프링이 알 수 없는 인자를 받고 있으므로 역시 오류가 날겁니다. 우린 이미 배웠으므로 `InMemorySiteItemStore`에도 어노테이션을 붙여줍시다.

{% highlight kotlin %}
@Component
class InMemorySiteItemStore : SiteItemStore {
  // ...
}
{% endhighlight %}

이번엔 [@Component 어노테이션](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html)을 붙였습니다. 이 어노테이션 역시 Bean으로 정의하기 위해 붙입니다.

이제 다시 실행해봅시다. 이번엔 스프링 부트가 잘 실행이 되는군요. 테스트를 위해 `/sites` API를 호출해 볼까요? API를 호출하려면 호스트와 포트를 알아야겠죠? 호스트는 내 컴퓨터의 애플리케이션을 호출할 것이므로 localhost로 하면 됩니다. 포트를 알아내면 되는데, 스프링 부트 로그를 보시면 다음과 같은 로그를 확인하실 수 있을겁니다.

```
2018-04-01 01:08:16.650  INFO 13344 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
```

톰캣이 8080포트로 시작되었군요. 이제 API를 호출해볼까요? 브라우저를 열고 다음의 주소를 입력해봅시다.

```
http://localhost:8080/sites
```

화면에 찍힌 응답이 보이시나요?

{% highlight json %}
{"list":[]}
{% endhighlight %}

이제 즐겨찾기를 추가해봅시다. POST 요청을 해야 하므로 브라우저가 아닌 다른 도구가 필요할 것 같네요. 저는 [Postman](https://www.getpostman.com/)을 사용하겠습니다.

![포스트맨을 통한 포스트 요청]({{"/assets/image/kotlin-famphlet/first-api/postman-request-post.PNG" | absolute_url}})

우리의 `/sites` 포스트 요청은 인자에 @RequestBody 어노테이션이 붙어있으므로 요청이 JSON 형태여야 합니다. 만약 포스트맨을 쓰신다면 Body를 raw로 설정하시고 가장 오른쪽의 선택지를 `JSON(application/json)`로 선택해주세요. 우리의 `/sites` 포스트 요청은 헤더의 `Content-Type`이 `application/json`이어야 하기 때문입니다.
응답은 다음처럼 옵니다.

{% highlight json %}
{
  "id": 1,
  "itemName": "Github",
  "siteUri": "https://github.com/"
}
{% endhighlight %}

즐겨찾기가 만들어졌습니다. ID는 1이네요. 조회를 해볼까요?

```
http://localhost:8080/sites/1
```

아래처럼 응답이 온다면 조회가 잘 되는 겁니다. 그것은 생성이 잘 되었다는 뜻이기도 합니다.

{% highlight json %}
{
  "id": 1,
  "itemName": "Github",
  "siteUri": "https://github.com/"
}
{% endhighlight %}

수정을 해봅시다.

![포스트맨을 통한 패치 요청]({{"/assets/image/kotlin-famphlet/first-api/postman-request-patch.PNG" | absolute_url}})

응답을 보면 ID는 같은데 내용이 다르게 왔음을 알 수 있습니다. 잘 수정되었네요.

{% highlight json %}
{
    "id": 1,
    "itemName": "Github Pages",
    "siteUri": "https://github.io/"
}
{% endhighlight %}

이제 지워볼까요?

![포스트맨을 통한 딜리트 요청]({{"/assets/image/kotlin-famphlet/first-api/postman-request-delete.PNG" | absolute_url}})

응답으로 지워진 즐겨찾기가 왔습니다.

{% highlight json %}
{
    "id": 1,
    "itemName": "Github Pages",
    "siteUri": "https://github.io/"
}
{% endhighlight %}

잘 삭제되었는지 확인하기 위해 ID 1을 조회해볼까요? 조회를 했더니 500 코드를 받았습니다. 뭐가 잘못된 걸까요? 이것은 다음 시간에 알아보도록 하겠습니다.
