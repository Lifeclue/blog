---
layout: post
title: "스프링 부트에서 다중 데이터 소스 사용하기"
date: "2018-03-02 11:27:08 +0900"
comments: true
tags: 스프링 Spring 스프링부트 SpringBoot 다중데이터소스 MultipleDataSource 다중디비 여러디비 MultipleDB
---

다중 데이터 소스를 사용하기 위해서는 각각의 데이터 소스와 엔터티 매니저 팩토리, 트랜잭션 매니저 3개의 설정을 한 묶음처럼 여기고 설정해주어야 한다.

[Spring boot의 Data Access 문서](https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/htmlsingle/#howto-data-access)(1.5.10 기준)만 따라해서는 데이터 소스와 엔터티 매니저만 설정하기 때문에 트랜잭션 매니저 설정에 대해서 꼭 기억해야 한다.

트랜잭션 매니저 설정을 해주지 않으면 Repository.save() 메서드를 이용해 엔터티를 저장할 때 오류 없이 메서드 호출이 성공하지만 실제 데이터베이스에 INSERT되지 않는 경험을 하게 될 것이다.

스프링에 의해 관리되지 않는 엔터티의 경우에는 Primary 설정을 쓰게 되는데 그럴 경우 트랜잭션 매니저도 스프링에서 기본적으로 설정해 둔 트랜잭션 매니저를 사용하게 될 것이고 그로 인해서 트랜잭션 없이 데이터 삽입이 이뤄지기 때문으로 보인다.

{% highlight java %}
@Configuration
@EnableJpaRepositories(basePackageClasses = {MatchOhlcRepository.class},
        entityManagerFactoryRef = "exchangeEntityManagerFactory"
)
@Profile({"!test"})
public class ExchangeDataConfig {

    @Bean
    @ConfigurationProperties("spring.datasource.exchange")
    public DataSource exchangeDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean exchangeEntityManagerFactory(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(exchangeDataSource())
                .packages(ExchangeMatchOhlcData.class)
                .persistenceUnit("exchange")
                .build();
    }
}
{% endhighlight %}
