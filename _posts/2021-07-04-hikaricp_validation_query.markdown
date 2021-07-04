---
layout: post
title: HikariCP Validation Query
description: HikariCP
date:   2021-07-04 11:22:00 +0530
categories: code
tags: [ Spring, HikariCP ]
author: easywaldo
comments: true
---

#### HikariCP

DB 와의 커넥션을 관리할 수 있도록 해주는 여러 라이브러리 들이 있는데 이중 하나가
HikariCP 이다.

여기 사이트에서 자세한 설명을 볼 수 있다.
- https://github.com/brettwooldridge/HikariCP


공식 문서에서는 오버헤드가 없으며 매우 가볍고 단순하다고 설명을 하고 있다.

>Fast, simple, reliable. HikariCP is a "zero-overhead" production ready JDBC connection pool. At roughly 130Kb, the library is very light. Read about how we do it here.
   "Simplicity is prerequisite for reliability."
         - Edsger Dijkstra

 
 HikariCP 의 특징 중에서 DB 와의 커넥션 풀을 계속 유지하지는 않는다는 것이다.
 다시 말하자면 커넥션 타임아웃 설정을 하더라도 데이터베이스 차원에서의 커넥션 타임아웃이 발생하게 되는 경우 HikariCP 는 이를 존중한다는 것이다.

 따라서 기존의 커넥션 풀에서 활동을 했던 어플리케이션 스레드가 커넥션이 유효한지를 확인을 함으로써 유효하지 않는 경우에는 다시 커넥션을 생성함으로써 정상적으로 DB 작업을
 수행할 수 있도록 해야한다.

 이를 가능하게 해주는 것이 바로 Validation Query 이다.

 ```java
         HikariConfig config = new HikariConfig();
        config.setPoolName("xxxx_pool");
        config.setDriverClassName(hottDriverClassName);
        config.setConnectionInitSql("SET NAMES 'utf8mb4'");
        config.setJdbcUrl(...);
        config.setUsername(...);
        config.setPassword(...);
        config.setMaximumPoolSize(maximumPoolSize);
        config.setConnectionTestQuery(connectionTestQuery);
        config.setMaxLifetime(maxLifeTime);
        config.setConnectionTimeout(connectionTimeout);
        config.setValidationTimeout(validationTimeout);
        config.addDataSourceProperty("cachePrepStmts ", "true");
        config.addDataSourceProperty("prepStmtCacheSize  ", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit  ", "2048");
        return new HikariDataSource(config);

 ```

 위의 코드를 보면 setConnectionTestQuery 메서드를 볼 수 있다.
 여기에 아래와 같이 쿼리를 최소화 하여 실행이 되는지만 확인이 가능하도록 할 수 있다.
 > select 1

 이렇게 설정을 하게 되면 HikariCP 가 SQL 질의를 하기 전에 해당 쿼리를 먼저 수행하여
 커넥션 검증을 할 수 있는 것이다.

 만약 검증하여 커넥션이 끊긴 상황이라면 HikariCP 는 어떤 작업을 수행하는지 알아보자.

 1. DB 와의 커넥션이 끊기게 될때 커넥션 실패 오류가 발생이 된다.

 2. 이후 커넥션을 생성하게 된다.

 3. 다시 유효성 확인 쿼리 실행이 되고 정상 확인이 되면 본래의 SQL 질의가 이뤄진다.

 ![hikari_cp_validation_fail]({{ "images/post/hikaricp/hikari_cp_validation_fail.png" | absolute_url }})


 커넥션이 정상적인 경우에는 아래와 같이 유효성 쿼리가 실행이 된다.

 ![hikari_cp_validation_success]({{ "images/post/hikaricp/hikari_cp_validation_success.png" | absolute_url }})


