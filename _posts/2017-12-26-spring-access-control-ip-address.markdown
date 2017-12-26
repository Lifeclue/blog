
---
layout: post
title: "Spring boot X RemoteAddrFilter IP 주소 접근 제어"
date: "2017-12-26 22:11:43 +0900"
---
Spring으로 프로그램을 짜다 보면 간혹 접속하는 클라이언트의 IP가 특정 주소인 경우 허용 또는 차단하고자 하는 욕망에 사로 잡히는 경우가 생깁니다.
특정 클라이언트 IP 주소를 차단/허용하는 방법에는 몇가지가 있습니다. 이 포스트에서는 RemoteAddrFilter를 이용하는 방법을 소개하겠습니다.

# RemoteAddrFilter
[RemoteAddrFilter](https://tomcat.apache.org/tomcat-7.0-doc/api/org/apache/catalina/filters/RemoteAddrFilter.html)는 Tomcat에서 제공되는 필터입니다. javadoc에서 소개하는 내용은 다음과 같습니다.
> 원격 클라이언트의 문자열로 표현된 IP 주소를 기반으로 필터링하는 RequestFilter의 구현체

내용을 보아하니 [RequestFilter](https://tomcat.apache.org/tomcat-7.0-doc/api/org/apache/catalina/filters/RequestFilter.html)의 구현체라고 소개하고 있습니다.
그렇다면 RequestFilter는 어떤 역할을 할까요? javadoc의 내용을 대충 요약하면 다음과 같습니다.
> 요청의 속성을 정규식과 비교하여 필터링하는 필터의 구현체

두 내용으로 미루어 보아 RemoteAddrFilter는 요청의 원격지 IP 주소 속성을 미리 설정해 둔 정규식과 비교하여 허용 또는 차단하는 필터라는 것을 알 수 있습니다.

이것을 사용하는 방법은 생각보다 쉬운데, 설정 클래스만 하나 만들어주면 됩니다. 아무 패키지 안에 넣어두시면 됩니다만 각자의 프로젝트 패키지 구조에 맞게 선정하시면 됩니다.

```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean remoteAddrFilterRegistrationBean() {
        RemoteAddrFilter filter = new RemoteAddrFilter();
        filter.setAllow("127.0.0.1|(0:){7}1");
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(filter);
        filterRegistrationBean.addUrlPatterns("/api/tools/*");
        return filterRegistrationBean;
    }
}
```

위와 같이 FilterRegistrationBean를 Bean으로 등록해 주기만 하면 스프링이 알아서 필터를 등록하고 잘 작동하도록 해줍니다.

여기서 주목해야 할 부분은 filter.setAllow() 메서드와 filterRegistrationBean.addUrlPatterns() 메서드입니다.  
