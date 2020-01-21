---
layout: post
title: "API 오류 처리하기"
date: "2018-04-03 21:39:06 +0900"
categories: kotlin spring
comments: true
tags: 스프링 Spring 스프링부트 SpringBoot
---
여러분의 API는 안녕하신가요? 모든 API가 성공적으로 응답하는 것은 물론 아니겠지요. 요청에 실수가 있었을 수도 있고 API에 버그가 있을 수도 있습니다. 네트워크는 항상 불신해야 합니다. DB에 문제가 생길 수도 있습니다. 이런 경우에 API는 오류를 반환합니다.

## Spring 기본 오류

Famphlet이라는 프로젝트가 있다고 해 봅시다. Famphlet의 API는 `/sites`로 시작합니다. 그런데 API 사용자가 오타를 내면 어떨까요?

```
http://localhost:8080/site
```

응답으로 `404` 상태 코드가 옵니다. 그리고 아래처럼 오류 정보를 같이 제공해줍니다. 이것은 스프링이 기본으로 제공하는 오류입니다.

```json
{
    "timestamp": "2018-04-03T12:49:03.331+0000",
    "status": 404,
    "error": "Not Found",
    "message": "No message available",
    "path": "/site"
}
```

스프링에서 기본으로 제공해주는 오류 응답은 위와 같은 형식입니다. 여기서 값만 바뀌는거죠.

- `timestamp` 오류가 발생한 시각입니다.
- `status` HTTP 상태 코드를 오류 정보에서도 제공하는 것입니다.
- `error` HTTP 상태 코드의 의미를 자연어로 풀어 놓은 것입니다.
- `message` 추가적으로 서버에서 제공하는 오류 문구입니다.
- `path` 오류가 발생한 경로입니다.

## 처리되지 않은 오류

스프링이 기본으로 제공해주는 오류는 처리되지 않은 오류입니다. 우리가 처리한 적이 없는데 없는 주소로 접근했을 때 위처럼 응답을 내려주었죠? 하지만 사용자는 `No message available` 같은 오류를 보고 당혹스러울 수밖에 없습니다. 스프링이 기본 오류를 응답하도록 가만히 두고 볼 수만은 없겠네요.

## 오류 처리

그렇다면 오류 처리는 어떻게 할 수 있을까요? 스프링에서 소개하는 오류 처리 방법 몇 가지가 있습니다. 이제부터의 내용은 [Exception Handling in Spring MVC](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc)를 재해석하는 내용입니다.

- HTTP 상태 코드 사용
- @Controller 기반 오류 처리
  - @ExceptionHandler 사용
  - 오류 화면 제공
- 전역 오류 처리
  - @ControllerAdvice 사용
- 저수준 처리
  - HandlerExceptionResolver 사용
  - SimpleMappingExceptionResolver 사용
  - SimpleMappingExceptionResolver 확장
  - ExceptionHandlerExceptionResolver 확장

몇 가지라기에는 좀 많긴 하군요.

### HTTP 상태 코드 사용

재정의한 예외가 발생했을 때 HTTP 상태 코드를 임의로 정의해서 사용할 수 있습니다. 먼저 예외를 만들어봅시다.  
Famphlet에서는 Site를 찾을 수 없는 경우 예외가 발생합니다. 우리는 그 예외를 `IllegalArgumentException`으로 발생 시킬 수 있습니다.

```kotlin
override fun getSiteItem(id: Long): SiteItem {
    return itemStore.findById(id) ?: throw IllegalArgumentException("${id}에 해당하는 즐겨찾기를 찾을 수 없습니다.")
}
```

넓은 의미에서는 요청값이 잘못된 것이 맞지만 좀 더 명확한 예외를 만들어서 처리해봅시다.

```kotlin
class SiteNotFoundException(val id: Long) : Throwable()
```

[코틀린의 예외](https://kotlinlang.org/docs/reference/exceptions.html)는 모두 Throwable을 확장합니다. 특히 코틀린에는 확인 필 예외가 없다는 점이 눈길을 끕니다. [Checked Exceptions에 관한 문서](https://kotlinlang.org/docs/reference/exceptions.html#checked-exceptions)를 보면 여러 관점을 소개하며 확인 필 예외가 얼마나 피로감을 주는지에 대해 설명하고 있으니 일독 하셔도 좋을 것 같습니다.

예외를 만들었다면 여기에 어노테이션을 붙여 줌으로써 HTTP 상태 코드를 의미 있는 값으로 지정하여 응답에 내려줄 수 있습니다.

```kotlin
@ResponseStatus(value= HttpStatus.NO_CONTENT)
class SiteNotFoundException(val id: Long) : Throwable()
```

상태 코드가 `NO_CONTENT`면 `204`코드로 응답을 하고, 이는

> 요청은 실패없이 처리했으나 제공할 컨텐츠가 없음

이라는 뜻입니다. `@ResponseStatus`의 value 속성과 더불어 reason 속성을 추가로 부여할 수 있습니다. 여기에는 오류의 내용을 입력할 수 있습니다. 그러나 204는 [제공할 컨텐츠가 없는 상태](https://tools.ietf.org/html/rfc7231#section-6.3.5)이므로 reason을 설정해도 아무런 응답 본문이 내려가지 않습니다.

### @Controller 기반 오류 처리

#### @ExceptionHandler 사용

컨트롤러에 `@ExceptionHandler` 어노테이션이 붙은 메서드를 정의하면 `@RequestMapping` 이 붙은 메서드를 처리하다가 발생한 예외를 그 메서드에서 처리할 수 있습니다. 이 메서드에서는 다음과 같은 일을 할 수 있습니다.

- 이미 만들어져 있어서 `@ResponseStatus` 어노테이션을 붙일 수 없는 예외들 처리하기
- 예외 별로 응답 본문을 다르게 정의하기
- 오류 페이지를 만들고 그 화면으로 보내기

위의 예제를 `@ExceptionHandler` 어노테이션을 이용해 처리해봅시다.

##### @ExceptionHandler와 @ResponseStatus

```kotlin
@ExceptionHandler(SiteNotFoundException::class)
@ResponseStatus(code = HttpStatus.NO_CONTENT)
fun handleSiteNotFound() {
    // ...
}
/* 혹은 */
@ExceptionHandler
@ResponseStatus(code = HttpStatus.NO_CONTENT)
fun handleSiteNotFound(e: SiteNotFoundException) {
    // ...
}
```

`@ExceptionHandler`에서 `SiteNotFoundException`을 처리하는 방법은 두 가지 입니다. 어노테이션에 처리할 예외를 정의하는 것과 실제 처리하는 메서드가 처리할 예외를 인자로 받는 것이지요. 둘 중에 한 곳에는 반드시 처리할 예외를 정의해주어야 합니다. 그렇지 않으면 이런 예외를 만나게 됩니다.

```java
java.lang.IllegalStateException: No exception types mapped to public void com.lifeclue.blog.famphlet.SiteItemController.handleSiteNotFound
```

저는 주로 인자로 받아서 처리하는 편입니다. 예외 중에서는 멤버 변수에 쓸 만한 값들을 갖도 있는 경우도 있기 때문입니다.

##### 오류 응답 재정의

응답을 서비스 문맥에 맞게 변경할 수도 있습니다.

```kotlin
@ExceptionHandler
@ResponseStatus(code = HttpStatus.BAD_REQUEST)
fun handleSiteNotFound(e: SiteNotFoundException): Map<String, Any> {
    return mapOf(
            "error" to mapOf(
                    "code" to 9999,
                    "message" to "존재하지 않는 사이트입니다: ID=${e.id}"
            )
    )
}
```

요청이 잘못된 것(`400`)으로 처리하고 응답을 자세히 내려주었습니다. 이렇게 처리하면 응답을 다음과 같이 받습니다.

```json
{
    "error": {
        "code": 9999,
        "message": "존재하지 않는 사이트입니다: ID=1"
    }
}
```

물론 Map보다는 오류 클래스를 정의하여 사용하시는 것을 권장합니다.

##### 전용 오류 화면으로 안내

만약 웹서비스라면 오류 화면으로 안내할 수도 있습니다.

```kotlin
@Controller
class SiteItemController(val siteItemService: SiteItemService) {
    // ...
    @ExceptionHandler(SiteNotFoundException::class)
    @ResponseStatus(code = HttpStatus.BAD_REQUEST)
    fun handleSiteNotFound(): String {
        return "site_not_found"
    }
    // ...
}
```

컨트롤러의 어노테이션을 `@RestController`에서 `@Controller`로 변경한 것에 주의하십시오. `@RestController`는 기본으로 `@ResponseBody`가 붙은 것으로 동작합니다. 그러니까 HTML을 응답으로 줄 수 없다는 것입니다. 그래서 `@RestController`에서 위와 같이 처리하면 응답이 `site_not_found`라는 문자열로 나갈 겁니다. 페이지로 이동하기 위해서는 `@Controller`로 변경해주시고 View의 이름을 반환하면 됩니다.

#### 오류 화면에서 예외 정보 제공

오류를 전용 오류 화면으로 안내할 때 오류 화면에 예외에 대한 정보를 출력할 수 있습니다. 이것이 일반 사용자들에게는 사실 중요한 정보는 아닙니다. 오히려 눈살을 찌푸리게 할 수도 있지요. 하지만 고객 지원 업무를 담당하는 직원이 고객과 개발자 사이에서 소통할 때 이 정보를 활용할 수도 있습니다. 또는 어떤 서비스냐에 따라 활용도가 높을 수도 있지요. 결국 개발자가 선택해야 할 문제입니다.

오류 화면에서 예외의 정보를 제공하려면 `ModelAndView`를 이용하시면 됩니다.

```kotlin
@Controller
class SiteItemController(val siteItemService: SiteItemService) {
    // ...
    @ExceptionHandler
    @ResponseStatus(code = HttpStatus.BAD_REQUEST)
    fun redirectToSiteNotFound(e: SiteNotFoundException): ModelAndView {
        var mav = ModelAndView()
        mav.addObject("exception", e)
        mav.viewName = "site_not_found"
        return mav
    }
    // ...
}
```

### 전역 오류 처리

컨트롤러 내에서 사용한 `@ExceptionHandler`는 해당 컨트롤러에서 발생한 예외만 처리하게 됩니다. 또한 `Throwable`을 확장한 예외 클래스의 `@ResponseStatus`는 이 어노테이션이 있는 예외에서만 작동합니다. 하지만 시간이 흐르고 흘러 기능이 추가되고 사용자들이나 회사 내부의 기획자 등등으로부터 요구사항이 발생하면서 변화가 누적되었을 때, 그 때는 컨트롤러도 여러 개일 테고 직접 작성한 예외도 많을 겁니다. 이것들을 일일이 찾아다니면서 예외 처리 구문을 추가해 주려면 생각만 해도 끔찍하네요.

전역 오류 처리는 이럴 때 유용합니다.

#### @ControllerAdvice 사용

`@ControllerAdvice`가 붙은 클래스를 정의하면 이 클래스는 `controller-advice`가 되고 세 가지 메서드가 지원됩니다. (스프링에 의해 호출이 된다는 뜻입니다.)

- `@ExceptionHandler` 어노테이션이 붙은 메서드에서 예외 처리
- `@ModelAttribute` 어노테이션이 붙은 메서드에서 Model에 추가 attribute 제공 (정상 동작할 때 작동하기 때문에 `@ExceptionHandler`와 같이 호출되지는 읺습니다.)
- `@InitBinder` 어노테이션이 붙은 메서드에서 Binder 초기화

우리가 볼 것은 물론 `@ExceptionHandler`입니다. 그리고 이것은 `@Controller` 기반 오류 처리를 했을 때와 거의 유사합니다.

```kotlin
@ControllerAdvice
class GlobalExceptionHandler {
    // ...
    @ExceptionHandler
    @ResponseStatus(code = HttpStatus.BAD_REQUEST)
    fun handleSiteNotFound(e: SiteNotFoundException): Map<String, Any> {
        return mapOf(
                "error" to mapOf(
                        "code" to 9999,
                        "message" to "존재하지 않는 ID입니다: ${e.id}"
                )
        )
    }
    // ...
}
```

클래스를 새로 만들고 `@ControllerAdvice` 어노테이션을 붙여주었습니다. 그리고 메서드 정의는 위에서 `@ExceptionHandler`를 썼던 것과 같습니다. 그러니까 `@ExceptionHandler`가 `@ControllerAdvice`에 있다는 것이 유일한 차이점이죠. 다만 이렇게 `@ControllerAdvice`를 오류 처리기로 사용하면 모든 컨트롤러의 처리되지 않은 예외가 이곳에서 처리됩니다.
즉, 예외 처리를 중앙 집중화할 수 있는 것이죠.

### 저수준 처리

저수준 처리는 직접 코드를 추가해주는 것입니다. 더 많은 것을 제어할 수 있지만 더 많은 고생을 해야 합니다. 이 부분은 많은 설명을 하지 않습니다. [문서](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#going-deeper)를 참조하시는 것이 제가 설명을 드리는 것보다 오해도 없고 빠를 것 같군요.

#### HandlerExceptionResolver 인터페이스 구현

인터페이스를 직접 구현합니다. [resolveException 메서드](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/HandlerExceptionResolver.html#resolveException-javax.servlet.http.HttpServletRequest-javax.servlet.http.HttpServletResponse-java.lang.Object-java.lang.Exception-)를 구현하면 `HttpServletRequest`와 `HttpServletResponse`를 이용해 직접 예외를 처리할 수 있습니다.

```java
  @Nullable
  ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex);
```

#### SimpleMappingExceptionResolver 사용

`SimpleMappingExceptionResolver` 클래스를 사용할 수 있습니다. XML 또는 어노테이션 기반으로 빈 정의를 할 때 [setExceptionMappings 메서드](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/handler/SimpleMappingExceptionResolver.html#setExceptionMappings-java.util.Properties-)를 이용해 예외 맵을 설정하면 됩니다.

#### SimpleMappingExceptionResolver 확장

로거 설정 및 사용, 또는 예외 처리의 추가 작업 등을 위해 예외 맵을 추가하지 않고 클래스를 확장(상속)하여 사용할 수 있습니다.

#### ExceptionHandlerExceptionResolver 확장
`ExceptionHandlerExceptionResolver`를 확장하여 `doResolveHandlerMethodException`메서드를 재정의하는 것도 방법입니다. 상속받은 `order` 속성을 잘 정의하면 `ExceptionHandlerExceptionResolver`의 기본 객체보다 앞서 호출되게 설정할 수도 있습니다.

---

우리는 종종 API가 `500` 코드로 응답하는 것에 당황하였습니다. 하지만 이제는 그럴 필요는 없게 되었네요. 이제 적절한 처리를 할 수 있기 때문에, 응답을 받은 사용자가 무엇이 잘못됐고 **무엇을 해야 하는지** 안내할 수 있게 되었습니다. **사용자가 무엇을 해야 하는지 알려주는 것**은 매우 중요합니다. 기능이 성공했을 땐 다음 단계에 대해 쉽게 설명하고, 실패했을 때는 대안이 무엇인지 쉽게 설명하는 것입니다. 오늘 공부해 본 오류 처리 방법으로 사용자가 API 오류 또는 실패 응답을 받았을 때 무엇을 해야 하는지 친절하게 알려줍시다.
