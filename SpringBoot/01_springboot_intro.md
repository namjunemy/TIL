# 1. 스프링 부트 시작하기

> [백기선 - 스프링 부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)

## Intro

크게 세가지 파트로 나눠서 스프링 부트를 학습한다.

* 스프링 부트 원리
  * 의존성 관리
  * 자동 설정
  * 내장 서블릿 컨테이너
* 스프링 부트 활용
  * 외부 설정
  * 로깅
  * 웹 개발
  * 데이터 기술 연동
  * 테스트
* 스프링 부트 운영
  * 엔드포인트
  * 모니터링
  * 매트릭스

## 01. 스프링 부트 소개

* [스프링 부트 공식 문서에 나와 있는 스프링 부트 소개](https://docs.spring.io/spring-boot/docs/2.0.3.RELEASE/reference/htmlsingle/#getting-started-introducing-spring-boot)

* 요약하자면,
* 스프링 부트는 production 수준의 독립적인 애플리케이션을 빠르고 쉽게 만들 수 있도록 도와준다.
* Opinionated view라고 불리는 컨벤션. 즉, 가장 널리 쓰인다고 하는 설정들을 스프링 부트가 기본 설정으로 제공한다. 그 설정에는 서드파티 플랫폼(가장 대표적으로 톰캣)에 대한 설정도 포함 된다.
* 모든 스프링 개발에 있어서 더 빠르고 더 폭넓은 사용성을 제공해준다.
* Opinionated view를 기본적으로는 제공하지만, 원하는대로 요구사항에 맞게 쉽고 빠르게 바꿀 수 있다. 이 부분 때문에 스프링부트가 계속해서 널리 사용될 수 있다.
* 비즈니스 로직에 필요한 기능 뿐만아니라, non-functional features를 제공한다.(예를 들어, embedded servers, security, metrics, health checks and externalized configuration)
* 자바 8 이상 부터, 서블릿 3.1 이상 버전을 사용하면 문제없다.

## 02. 스프링 부트 시작하기

