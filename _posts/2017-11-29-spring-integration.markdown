---
layout: post
title: "Spring Integration"
date: "2017-11-29 23:32:23 +0900"
category: getting-started
---
Spring Integration Framework으로 제품 개발을 시작해 보려고 합니다. Spring Integration의 특징은 [프로젝트 웹페이지](https://projects.spring.io/spring-integration/)를 참조하시면 쉽게 파악하실 수 있습니다. 하지만 제 관심사인 `어떤 역할을 수행할 수 있는가`를 프로젝트 웹페이지를 인용하여 나열해보겠습니다.

- Integration with External Systems
- ReST/HTTP
- FTP/SFTP
- Twitter
- WebServices (SOAP and ReST)
- TCP/UDP
- JMS
- RabbitMQ
- Email

보시는 것처럼 Spring Integration은 외부 시스템과의 통합을 위한 Framework입니다.

사실 이 포스트에서 할 것은 프로젝트 페이지에 있는 Quick Start를 따라하는 정도입니다. 하나씩 차근차근 따라가봅시다.
{% include dev_env.markdown %}
## 프로젝트 만들기
새로운 프로젝트를 만듭시다. Gradle 프로젝트이고 Java는 9를 쓰겠습니다. 회사였다면 Java 9를 쓰기 힘들었겠지만 지금은 자유로우니 Java 9를 체험해보겠습니다.  
![인텔리제이의 새 프로젝트 만들기 창]({{"/assets/image/getting-started/spring-integration-new-project.png" | absolute_url}})

## 의존성 설정
프로젝트 웹페이지의 Quick Start 란을 보시면 Download 칸을 볼 수 있습니다. 저는 개발 환경이 Gradle이므로 Download 옆 버전 옆의 빌드 환경을 Gradle로 눌러 변경하겠습니다. Maven을 사용하시는 분들은 안누르셔도 됩니다.  
이 Download 칸에 의존도 설정을 위한 코드가 있습니다. 이것은 그냥 바로 복사하여 프로젝트에 붙여넣으시면 됩니다.
현재 시점으로는 5.0.0 버전이군요.
{% highlight gradle %}
dependencies {
    compile 'org.springframework.integration:spring-integration-core:5.0.0.RELEASE'
}
{% endhighlight %}
