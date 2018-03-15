---
layout: post
title: "Spring method profiling"
date: "2018-03-15 10:48:29 +0900"
comments: true
tags: 스프링 Spring 스프링부트 SpringBoot
---

참고자료
http://www.baeldung.com/spring-performance-logging


{% highlight java %}
@Configuration
@EnableAspectJAutoProxy
@Aspect
public class ProfilingConfig {
    @Bean
    public PerformanceMonitorInterceptor performanceMonitorInterceptor() {
        return new PerformanceMonitorInterceptor(false);
    }

    @Bean
    public Advisor performanceMonitorAdvisor() {
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(public * com.lifeclue.blog..*.*(..)");
        return new DefaultPointcutAdvisor(pointcut, performanceMonitorInterceptor());
    }
}
{% endhighlight %}

{% highlight xml %}
<logger name="org.springframework.aop.interceptor.PerformanceMonitorInterceptor" level="TRACE"/>
{% endhighlight %}
