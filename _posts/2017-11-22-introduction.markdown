---
layout: post
title: "Spring Integration 프로젝트"
date: "2017-11-22 15:36:35 +0900"
category: spring-integration-project
comments: true
---

Spring Integration 프로젝트를 시작합니다.
Spring Integration 프로젝트란 말 그대로 Spring Integration을 이용한 프로젝트를
말합니다.
Spring은 다양한 Framework을 제공하고 있죠.
그 중 제가 최근에 사용해 본 Spring Integration에 대해 가상의 시나리오를 만들고
그 시나리오에 맞춰서 프로젝트를 제작/운영하는 이야기입니다.
이 프로젝트는 API를 제공하는 서버이며, 소켓과 HTTP를 이용할 예정입니다.

## 전문 통신
전문 통신에 대해 생소하신 분들도 계시리라 생각합니다.
전문 통신은 소켓을 통한 구조화된 문자열을 주고 받아 통신하는 방식입니다.
(소켓 통신을 실무에서는 전문 통신이라 부릅니다.)
예를 들면 다음과 같습니다. 아래 전문을 `포스트 획득 API`라고 가정해 봅시다.
```
GET_POST  20171122153635lifeclue            2017-11-22-written-post            \n
```
이것은 다음과 같은 의미입니다.(␣는 공백을 시각화한 것입니다.)

|전문 구분[10자]|날짜[8자]|시각[6자]|블로그 구분[20자]|포스트 ID[35자]|전문 종료자[1자]|
|:-|:-|:-|:-|:-|:-|
|GET_POST␣␣|20171122|153635|lifeclue␣␣␣␣␣␣␣␣␣␣␣␣|2017-11-22-written-post␣␣␣␣␣␣␣␣␣␣␣␣|\n|

보통 전문은 Header와 Body로 나눕니다. 자세한 내용은 이야기 속에서 하겠습니다.

## HTTP 통신
여러분이 흔히 알고 계시는 HTTP API입니다.
예를 들어 위에서 언급한 `포스트 획득 API`라고 한다면
```
http://sample.com/api/blog/lifeclue/2017-11-22-written-post
```
이것은 다음과 같은 API 명세를 따른 것입니다.
```
요청 GET /blog/{Blog ID}/{Post ID}
응답 HTML
```

## 시나리오
시나리오는 대충 회사에서 서비스를 운영하고 있고, 제휴사와 통신할 일이 생기면서
벌어지는 일들을 다룰 예정입니다.

즐거운 이야기가 되길 바라며...
