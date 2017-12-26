
---
layout: post
title: "Spring boot IP 주소 접근 제어"
date: "2017-12-26 22:11:43 +0900"
---

```java
@Configuration
public class WebCOnfig {

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
