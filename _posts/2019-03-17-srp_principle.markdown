---
layout: post
title: Single Responsibility Principle - SOLID 원칙 중
description: 소프트웨어 설계 원칙, SRP
date:   2019-03-17 16:20:00 +0530
categories: code
tags: [ SOLID, SRP ]
author: easywaldo
---

 `SOLID` 원칙에 대해서 글을 쓰고 있지만 이 원칙을 알게 된지는 정말 오래 되지 않았다.
불과 18년도에 근무를 하게 된 회사에서 그 용어를 처음 접할 수 있었다.
개발 업무를 시작한지가 2005년도 부터이니 그야말로 장님과 같이 눈먼채로 소프트웨어 개발만 하고 있었던 것 같다.

>클린코드라는 책으로 널리 알려진 Uncle Bob Martin이 2005년도에 `The Principle of OOD` 라는 아티클에서 소개한 소프트웨어 개발 >원칙이다. 하지만 이 글에서 Bob은 1995년부터 OOD 를 위한 원칙들을 여러번 소개해왔다고 밝히고 있다. 

SOLID 원칙은 객체지향 프로그래밍을 수행함에 있어서 
기능들 간에 결합도를 낮추고 응집도를 높일 수 있게 도와 주는 필수 원칙이라고 할 수 있겠다.

그 목록은 아래와 같다.
- Single Responsibility Principle
- Open Closed Principle
- Liskov Substitution Principle
- Interface Segregation Principle
- Dependency Inversion Principle

이 중 SRP 에 대해서 알아보고자 한다.


# Single Responsibility

소프트웨어를 설계할 때 하나의 기능을 구현하기 위해서 

먼저 떠오르는 것이 클래스를 새롭게 추가하거나 혹은 기존 클래스에 메서드를 추가한다던지 하는 행위를 생각할 것이다.

상품을 전시하고 판매하는 일련의 과정을 설계해보자.

여기서 전시 서비스와 주문 서비스를 생각을 해볼 수 있을 것이고 이를 
DisplayProductsService , OrderProductsService 2개의 클래스로 구분을 한다고 해보자.

진열이 된 상품들을 보기 위해 DisplayProductsSerivce 에서는

GetProducts(string displayCode) 라는 메서드를 추가하여 각 진열위치에 따르는 상품들을 나열할 수 있다.

{% highlight csharp %}
public class DisplayProductsService
{
    public IEnumerable<Product> DisplayProducts(string displayCategory)
    {
        ...
    }
}
{% endhighlight %}

해당 상품 중 마음에 드는 상품을 선택 하여 OrderService 클래스에 있는 
OrderProducts(OrderInfo orderInfo) 라는 메서드에 해당 상품정보를 전달하여 주문을 할 수 있다.

{% highlight csharp %}
public class OrderService
{
    public void OrderProducts(OrderInfo orderInfo)
    {
        ...
    }
}
{% endhighlight %}

이후 주문한 상품들에 대해서 리스트를 보고자 하는 메서드가 필요하다.

두 클래스 중 어떤 클래스에 해당 메서드를 추가하는 것이 적절할까?

상품진열 과 상품주문 이라는 역할과 책임이 있는데 
여기서 주문된 상품보기 라는 역할을 어느 서비스에서 책임을 지는 것이 적절한지 고민해보는 것이다.

상품진열 보다는 상품주문 서비스가 해당 역할을 담당하기에 좀 더 가깝게 느껴질 것이다.
그래서 DisplayOrderedProducts 라는 메서드를 OrderService 에 추가로 만들었다.

{% highlight csharp %}
public class OrderService
{
    public void OrderProducts(OrderInfo orderInfo)
    {
        ...
    }

    public IEnumerable<Product> DisplayOrderedProducts(Guid orderId)
    {
        ...
    }
}
{% endhighlight %}

이렇게 되면 각각의 서비스에서는 하나의 책임만 담당할 수 있게 되며 이것이 단일책임원칙에 부합이 되는 것이라 할 수 있을 것이다.

여기서 만약 주문한 상품리스트와 함께 주문결제정보를 보고자 한다고 가정하자.

그렇다면 DisplayOrderedProducts 메서드를 수정하여 결제내역 정보와 함께 리턴을 하고자 할 수 있을 것이다.

만약 위와 같이 생각을 했다면 이것은 SRP 를 위배한 것이다.
DisplayOrderedProducts 메서드는 주문내역의 상품리스트 만을 조회할 역할을 가지고 있다가
결제정보를 조회하는 역할을 추가적으로 가지게 되면 이로 인해 첫번째는 리턴 타입이 변경이 됨으로써
이 메서드를 이용하고 있는 계층들에게까지 영향을 미치게 되며 수정해야 할 커버리지가 넓어지게 되는 결과를 가져온다.

따라서 적당한 해법은 결제 정보를 조회하는 메서드를 추가로 만들어
OrderService 를 이용해오던 Consumer 들에게 영향을 미치지 않게 하고
결제정보와 주문상품을 함께 보고자 하는 Counsumer 의 경우라면 2개의 메서드를 같이 이용하여 비즈니스 요구를 풀어나가는 것이
어플리케이션 관리 측면에서 보다 유연한 전략이 될 것이다.

위에서 설명한 내용은 Open Closed Principle 인 `개방폐쇄 원칙` 과도 맞닿아 있다.
즉 기능을 추가하는 것에는 열려 있으며 기능을 수정하는 것에는 닫혀 있다. 라는 원칙으로 소프트웨어를 디자인 하는 것이다.
이렇게 되면 `Side Effect / UpStream Effect / DownStream Effect` 를 최소화 할 수 있는 환경을 마련하게 된다.