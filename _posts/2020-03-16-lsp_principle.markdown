---
layout: post
title: 리스코브 치환원칙
description: 소프트웨어 설계전략 스터디 - 서브타이핑과 계층추상화 전략
categories: code
tags: [ software development ]
author: easywaldo
comments: true
---

### LSP Principle

> 1987년 바바라 리스코브가 자료 추상화와 계층이라는 제목으로 컨퍼런스에서 처음 소개한 내용이다.

객체지향 설계 원칙 중 SRP 와 함께 객체들의 관계에서의 타입을 설계하고 이러한 타입들간의 계층을 어떻게 구성할 것인지에 대해서 고민할 때
활용되는 원칙이라 할 수 있을 것 같다.

간단한 그림으로 설명을 해본다면

![lsp_image_02]({{ "images/post/lsp_image/lsp_img_02.png" | absolute_url }})

Motor 라는 수퍼타입이 존재하고 해당 타입의 서브타입인 `OilMotor` 와
`ElectronicMotor` 가 존재할 때 `Motor` 타입을 사용하는 소비자 즉 이들 서브타입의 인스턴스를 생성하여 사용하는 사용자 프로그램에서는 Motor Type 이 아닌 OilMotor 또는 Electronic Motor 타입으로 대체하여 사용할 수 있어야 함을 의미한다.

이는 부모타입과 계약을 맺고 실행이 되는 프로그램 계약관계는 해당 부모타입의 자식타입과도 동일한 계약을 맺을 수 있음을 의미하는 것이다.