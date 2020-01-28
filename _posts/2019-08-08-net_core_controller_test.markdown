---
layout: post
title: .Net Core Controller 에 대한 테스트
description: TDD 및 UnitTest
date:   2019-08-08 01:10:00 +0530
categories: code
tags: [ software development ]
author: easywaldo
---

테스트를 하고자 하는 대상은 Controller 의 특정 Action 이다.

해당 Action 에 대해서 테스트를 하고자 한다면 먼저 Controller 에 대한 인스턴스를 생성해야 하는데

Controller Class 의 Constructor (생성자) 가 전달받아야 하는 Parameter 들이 준비가 되어 있어야 한다.

Unit Test 를 작성할 때의 기본 절차는 아래와 같이 정리가 된다.

- 테스트를 할 대상이 무엇인가?
- 대상이 정해졌다면 테스트를 하기 위해 필요한 환경(의존성)은 무엇인가?
    * 의존성은 결국 테스트를 하고자 하는 대상이 제대로 작동하기 위해서 필요한 클래스, 인터페이스 등을 의미한다.
- 필요한 것들을 파악했다면 이를 준비한다.
    * 이러한 준비 작업을 Arrange 단계로 명명할 수 있다.
- 준비한 것들을 가지고 테스트 대상을 실행하도록 한다.
- 테스트가 의도하던 결과로 완료가 되었는지 검증을 한다.

이러한 절차를 바탕으로 .net core Controller Class 가 필요로 하는 의존성을 준비해보자.

아래의 코드는 테스트를 하기 위해 MallController 이라는 클래스의 생성자에서 필요로 하는 것들을 보여주고 있다.

```csharp
public class MallController : BaseController
{
    private readonly IAuthService _coreService;
    private readonly IProductService _productService;

    public MallController(
        IAuthService coreService,
        IProductService productService)
    {
        this._coreService = core;
        this._productService = productService;
    }
}
```

따라서 테스트 코드에서는 IAuthService 와 IProductService 가 준비가 되어 있어야만 MallController 클래스의 인스턴스 생성이 가능해진다.

어떻게 준비하지??

여기서 준비를 하는 방법은 2가지로 나누어 볼 수 있을 것 같다.

1. 실제 테스트 대상이 필요로 하는 클래스 또는 인터페이스의 구현체를 준비시키는 방법

2. 실제 테스트 대상이 필요로 하는 클래스 또는 인터페이스에 의존하는 테스트를 하기 위한 목적의 환경을 준비시키는 방법
    * 이를 TestDouble 이라 하며 테스트 대역이라고 번역하기도 한다.

    > TestDouble 은 테스트 대상에만 집중할 수 있도록 환경을 조성하기에 좋은 경우도 많지만 남용하거나 오용을 할 경우
    실제 테스트가 의도한 방향으로 흘러가지 않을 수 있는 가능성을 제공하기도 하며 
    (물론 1번의 경우도 테스트 코드가 잘못된 경우라면 마찬가지다.)
    테스트를 위해 무수히 많은 TestDouble 을 만들어 관리하기 어렵고 복잡한 Testing 환경을 초래하기도 한다. 
    (내 경험담..;;)


단위테스트 코드 작성을 해보자!!


