---
layout: post
title: Logstash 경험해보기
description: ELK Stack
date:   2019-01-19 11:00:00 +0530
categories: code
tags: [ ELK, Logstash ]
author: easywaldo
---

# Logstash
Log 와 Stash 이 두개의 단어의 의미를 생각해본다면 이것이 어떠한 역할을 하는 놈인지
조금은 감을 잡는데 도움이 될 것 같았으나..
나는 전혀 그렇지 못하게 로그를 한데 다 모아놓는 저장공간이라고 잘못 이해를 했다.
Git의 Stash 기능과 혼동이 되어 그랬을 수 있고 아니면 그냥 바보여서 그랬을 수도..

# 일단 설치를 해보자
Install 방식으로 하지 않고 source 그대로 서비스를 실행해보고 싶어서
elastic 사이트에서 zip 파일을 내려 받았다.
그 다음으로 압축을 푼 다음 bin 폴더에 실행을 하는 logstash.bat 파일이 있다는 것을 확인한 후
콘솔에서 실행.. 했더니

>could not find java set java_home or ensure java is in path logstash

Java Runtime 환경이 없다는 메시지가 출력이 되었다.
그래서 java runtime 도 installer 로 설치하지 않고 source 를 로컬에 다운로드 완료.

# Java 를 어떤 버전으로?
OpenJDK 와 OracleJDK 의 차이점에 대해서 알아보고자 여기 글을 보고 차이점을 먼저 살펴봄.
Oracle 에 왠지 손을 들어주기 싫어 OpenJDK 로 선택 !!

처음에 10 버전으로 선택하였으나 ELK 에 안정된 JDK 버전은 8버전임을 확인하여 다시 8버전으로 환경 구성을 하였다.
10으로 하게 될 경우 아래오 같은 오류 메시지를 출력하였다.

>.. Unrecognized VM option 'UseParNewGC

java 를 설정하면서 이상했던 건 java 환경변수 설정을
bin 폴더로 하여 콘솔에서 java --version 커맨드로 출력이 이상없이 됨을 확인하였지만
logstash 는 여전히 java 환경구성이 되어 있지 않다는 이상한 메시지만 출력..

혹시나 해서 bin 폴더로 하지 않고 한단계 상위 폴더로 설정하자 드디어 logstash 가 실행이 되었다.
음..내가 뭘 잘못 이해하고 있는건가??

여튼 구동이 되었으니 이제 뭘 해볼까?

# Hello World

모든 프로그래밍 언어에서 처음 접하게 되는 Hello World 출력해보기.
그래서 Lostash 에서도 콘솔로 간단하게 출력해볼 수 있는게 있지 않을까? 하고 문서를 찾아보던 중
바로 아래와 같이 간단하게 메시징을 콘솔로 입력받아 콘솔로 출력 하는 예제가 있음을 확인하였다.

>bin/logstash -e 'input { stdin { } } output { stdout {} }'

위와 같이 실행하여 콘솔에 아래와 같이 입력을 하고 출력이 되는걸 확인하였다.

hello world
2013-11-21T01:22:14.405+0000 0.0.0.0 hello world

그래서 이번에는 logstash 명령만 입력 후 실행하려 했더니..
오류 발생..
>ERROR: Pipelines YAML file is empty. Location: .../Study/Els/logstash-6.5.4/config/pipelines.yml

그래서 logstash yml 파일에 대해서 들여다보기 시작

# yml 파일을 어떻게 수정하지?
아래와 같이 환경설정 역할을 하는 수 많은 옵션들이 있었는데 몇 가지 옵션들을 사용해서 구동을 해도
오류가 발생하긴 마찬가지..

> Settings file in YAML
>....
>   pipeline.batch.size: 125
>   pipeline.batch.delay: 50

# conf 파일 설정
결국 문서를 찾다 찾다 -f 옵션을 주고 별도의 conf 파일을 만들어서 구동을 해야 함을 알았다.
conf 파일의 기본 구성은 아래와 같이 구성이 된다.

1. input
2. filter
3. output

input 은 데이터소스를 외부로부터 가져오기 위한 부분이고
filter 는 데이터소스를 원하는 포맷으로 만들어주기 위한 부분이라 할 수 있다.
마지막으로 output 은 데이터를 원하는 결과값으로 만들어 Elasticsearch 또는 다른 데이터 저장소로
보내주기 위해서 사용되는 부분이다.

이것이 바로 Logstash 의 역할이었던 것이다.

# 어마무시하게 방대한 필터
ELK Stack 에서 데이터의 관문 역할을 담당하고 있다고 나는 이것을 평가하고 싶다.
그래서 거기에 맞는 데이터 가공이 필요하며 이를 위해서 수많은 다양한 필터가 기본 내장이 되어 있으며
추가적으로 설치도 가능하다.

또한 루비 코드를 이용하여 필터링도 가능하다고 하니 오픈소스 진영에서 Logstash 를 이용한
데이터 수집 파이프라이닝에서의 위상을 가늠해 볼 수 있을 것 같다.
[Logstash Filter] 에 대한 자세한 정보는 아래에서 확인이 가능하다.

[Logstash Filter]: https://www.elastic.co/guide/en/logstash/current/filter-plugins.html