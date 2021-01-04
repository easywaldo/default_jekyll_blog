---
layout: post
title: Domain Driven Design!? - 개념적 설명
description: 소프트웨어 설계전략 스터디
date:   2018-09-01 01:55:00 +0530
categories: code
tags: [ software development ]
author: easywaldo
---


### 도메인 주도 설계 또는 개발
라고 해석할 수 있을법한 이 말은 내게는 불과 18년초까지만해도 생소한 용어였다.

DDD 를 구글에서 검색을 해보았더니 가장 먼저 조회된 결과는 아래와 같이 EXID 의 노래..

![ddd_capture]({{ "images/post/ddd_capture.png" | absolute_url }})

얼마전까지의 나의 모습 기준으로 DDD 를 접했을 당시의 내 머릿속을 보는 것과 같다!!

<br/>
## 그렇다면 도메인은 무엇인가요?
위키피디아에서는 아래와 같이 정의가 되어 있다.

>도메인은 일반적인 요구사항, 전문 용어, 그리고 컴퓨터 프로그래밍 분야에서 문제를 풀기위해 설계된 어떤 소프트웨어 프로그램에 대한 기능성을 정의하는 연구의 한 영역이다. 도메인 엔지니어링 이라고도 알려져 있다.

<br/>
### 소프트웨어 설계에 참여하는 사람은 개발자 만이 아니다.

기획자와 개발을 의뢰한 고객 모두가 설계를 담당하는 사람들이다.
이들이 하나로 뭉쳐 소프트웨어를 개발하기 위해서는 무엇보다 본인이 알고있는 도메인 지식에 대한 의사 교환이 필요한데
이때 등장하는 용어가 바로 *Ubiquitous Language* 이다.

<br/>
### DDD 에서 나오는 개념은 아래와 같다.

* Context
* Domain
* Model
* Ubiquitous Language

<br/>
### Ubiquitous Language
서로가 알고 있는 도메인 지식이 다르고 같은 행위를 지칭하는 용어가 모두 다를 수 있기 떄문에
이를 하나의 공통된 언어로 정의하는 것이 중요하다.

위키피디아에서는 아래와 같이 정의를 하고 있다.
> A language structured around the domain model and used by all team members to connect all the activities of the team with the software.

<br/>
### Context
> The setting in which a word or statement appears that determines its meaning;
>> 의미를 결정하는 단어나 문장에서 나타나는 설정 (번역)
이라고 위키피디아에서는 정의가 되어 있는데.. 사실 의미가 잘 와닿지는 않는다.
내 방식대로 의미를 부여해보자면 어떠한 업무를 정의해주는 의미있는 단어와 문장들 이라고 하고 싶다.

<br/>
### Domain
> A sphere of knowledge (ontology), influence, or activity. The subject area to which the user applies a program is the domain of the software;

쉽게 내가 이해할 수 있는 의미로는 소프트웨어가 나타내는 주제 그리고 그 안에서 일어나는 활동과 그로인한 영향 및 소프트웨어에 포함된 모든 지식을 통틀어서 도메인이라고
부를 수 있을 것 같다.

<br/>
### Model
> A system of abstractions that describes selected aspects of a domain and can be used to solve problems related to that domain;

도메인을 표현하기 위해 추상화한 시스템이라 할 수 있으며 도메인과 연관된 문제들을 해결하기 위해 사용이 될 수 있는 선택된 상 이라 할 수 있을 것 같다.

<br/>
### 도메인 디자인
-----------
이상적으로는 하나의 통합된 모델이 바람직할 수 있겠으나 이것은 그야말로 개념적인 순수한 목표이며
도메인을 디자인을 하면서 역할이 다른 여러 서브 도메인이 추출이 되며 서브 도메인 내에서는 문제를 해결하기 위해 필요한 추상화된 여러 모델들이 만들어 지게 된다.
전략적으로 도메인 디자인을 하기 위해서는 모델 무결성을 유지될 수 있게 해야하며 도메인 모델의 분류 그리고 다양한 모델들로 도메인이 작동될 수 있도록 하는 것이다.

이때 수행해야 할 과정들이 있는데 아래와 같다.

+ Bounded Context
+ Continuous Integration
+ Context Map

<br/>
### Bounded Context
쉽게 말해서 서로 성격이 다른 담당하는 주제 영역이 다른 도메인 간의 경계 영역이라고 지칭할 수 있을 것 같다.
별개의 모델들로 혼합이 된 코드 기반에서 만들어진 소프트웨어는 신뢰할 수 없게 되고 이해하기도 어려워진다.
서로 다른 팀들은 의사소통을 하기가 어렵게 되고 불명확하고 적용이 되지 않아야 하는 컨텍스트들과 모델들이 자주 만들어지게 된다.

따라서 팀 조직에서의 관점 및 어플리케이션의 구체적인 측면에서의 사용법 그리고 코드 수준 또는 데이터베이스 스키마와 같은 수준의 물리적인 명세들을
명시적으로 정의해야 한다.

이는 모델들이 경계 사이에서 모순이 없도록 유지를 해야함을 의미하며 외부의 이슈에 의해 혼동이 되거나 흐트러지지 않아야 함을 의미한다.
위키피디아에서 설명하고 있는 원문 내용이다.

> Multiple models are in play on any large project. Yet when code based on distinct models is combined, software becomes buggy, unreliable, and difficult to understand. Communication among team members becomes confusing. It is often unclear in what context a model should not be applied.

> Therefore: Explicitly define the context within which a model applies. Explicitly set boundaries in terms of team organization, usage within specific parts of the application, and physical manifestations such as code bases and database schemas. Keep the model strictly consistent within these bounds, but don’t be distracted or confused by issues outside.

<br/>
### Continuous integration
몇 명의 팀원들이 같은 경계 영역안에서 일을하고 있을 때 모델이 깨어지게 되는 경향이 강하다.
3 ~ 4명 같은 더 작은 수의 사람들은 심각한 문제에 직면하게 된다. 점점 더 작아지는 컨텍스트들은 통합 수준에서의 가치를 상실하게 된다.

그러므로 모든 코드들을 다른 구현물들과 자주 병합하는 절차를 마련해야 하며 자동화된 테스트로 컨텍스트 측면에서 깨진 부불을 빠르게 표시해야 하며
가차없이 Ubiquitous Language 로 다른 사람들의 머리들 안에서 진화된 개념들 만큼 모델들을 고쳐나가야 한다.

위키피디아 원문 설명은 아래와 같다.
> When a number of people are working in the same bounded context, there is a strong tendency for the model to fragment. The bigger the team, the bigger the problem, but as few as three or four people can encounter serious problems. Yet breaking down the system into ever-smaller contexts eventually loses a valuable level of integration and coherency.

> Therefore: Institute a process of merging all code and other implementation artifacts frequently, with automated tests to flag fragmentation quickly. Relentlessly exercise the ubiquitous language to hammer out a shared view of the model as the concepts evolve in different people’s heads.


위의 내용을 내가 이해 수준에서 정리하자면 컨텍스트를 구성하는 모델들과 이를 구현하는 코드들은 시간이 지남에 따라서
통합된 컨텍스트 측면에서 벗어나는 경향이 있기 마련이며 이를 방지하기 위해서는 자동화된 테스트들과 지속적인 통합 절차가 필요함을 설명하는 것 같다.

내가 생각하는 컨텍스트 수준의 모델 개념과 구현 코드들이 다른 사람이 생각하는 컨텍스트 수준의 모델 개념과 구현 코드들과 차이가 발생할 수 있기 때문일 것이다.

