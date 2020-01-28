---
layout: post
title: Github.io 둥지 만들기
description: 깃허브에 새로운 정적 블로그를 만들었습니다. 
date:   2018-08-04 00:00:00 +0530
categories: code
tags: [ csharp, immutablearray ]
author: easywaldo
---

# 개발자스러운 정적 블로그를 만들어보고자 
# 그리고 멋지게 운영해보고자!!

## 마크다운도 공부를 해봐야겠지??

관심있는 개발 언어를 표현을 해보자면..아래 정도가 될 것 같다.

*python* *csharp* *javascript* *java*

이 중에서 **c#** 이 가장 주력 언어이며 python 은 최근 틈틈이 스터디 중이다.

이와 함께 아래의 웹 기술도 나의 기술 스택이다.

* ASP.NET with C#
* ASP.NET MVC Framework & Razor Tag
* jQuery


>시작이 반이라는 말이 있듯이..

>한주 동안 또는 주기적으로 새로이 배운 기술들을
>
> 시간이 되는 틈틈이 이곳에 기록을 하려고 노력할 것이다.


**Code:** 코드는 아래와 같이 지킬 마크다운에 따라서 표현이 될 수 있다.

{% highlight c# %}

ImmutableArray<int>.Builder builder = ImmutableArray.CreateBuilder<int>();
builder.Add(1);
builder.Add(2);
builder.Add(3);
ImmutableArray<int> oneTwoThree = builder.ToImmutable();

{% endhighlight %}

<script id="dsq-count-scr" src="//easywaldo.disqus.com/count.js" async></script>