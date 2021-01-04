---
layout: post
title: Springboot MySQL - H2 localDB 사용자정의함수 추상화 구현
description: 소프트웨어 설계전략 스터디 - 인터페이스를 활용한 추상화 전략
categories: code
tags: [ software development ]
author: easywaldo
comments: true
---

### RDBMS 의 사용자 개체를 어플리케이션에서 이용하는 방법 - OCP 준수 조건

> MySQL 서버에 함수를 만들어 이를 SpringBoot 에서 이용할 수 있는 방법은 무엇이 있을까? 첫번째로는 함수를 호출하는 Query 기반의 JPA 메서드를 만들 수 있을 것이고 JDBC 를 이용한 SQL 실행을 할 수 도 있을 것이다.

#### 인프라스트럭쳐에 대한 의존성??

> 위와 같은 기반으로 DB 함수 또는 프로시저 실행은 서비스 구현상 전혀 문제가 될 것은 없다.
DB가 앞으로도 계속 해당 벤더 제품의 호환성을 유지한다는 조건에서 말이다. 바로 이것이 인프라스터력쳐 영역에 대한 어플리케이션에서의 의존성이 발생하는 부분이다.

#### 의존성은 당연한 것 아닌가?

> 오래전에 만들어진 대다수의 어플리케이션 아키텍쳐는 수직적인 구조에 따라서 필요로 하는 데이터를 DB에 질의하고 해당 결과를 Object 타입으로 캐스팅하여 사용자와 상호 소통하는 `Interaction Area` 또는 `View Area`  에서 이를 소비하는 것이 일반적이 형태이다.

#### 함수 호출부를 추상화 해보자

> 함수 호출은 각 DB 마다 SQL 구문이 다를 수 있으므로 이 부분이 추상화 고려 대상이다.

```code
IStoredFunctionRepository
    - getDecryptData
    - getEncryptData
```

```code

IStoredFunctionRepository 인터페이스의 MySQL 구현체
IStoredFunctionRepository 인터페이스의 H2 구현체
...

```

위와 같이 추상화 및 구체 클래스를 구현할 수 있다.

또한 SpringBoot 의 경우 IoC 를 지원하는 프레임워크 이므로 의존성을 주입하는 컴포넌트를 추가하여 
어플리케이션 프로필 설정에 따라서 구체 클래스에 대한 인스턴스 주입이 가능하도록 하였다.

```java
// StoredFunctionFactory 의존성 주입 팩터리

@Component
public class StoredFunctionFactory {

    @Value("${spring.profiles.active}")
    private String profile;

    @Autowired
    private DataSource dataSource;

    public IStoredFunctionRepository getStoredFunction() {
        switch (this.profile) {
            case "local":
                return new H2StoredFunctionRepositoryImpl();
            case "dev":
            case "localdev":
            case "prod":
                return new MySqlStoredFunctionRepositoryImpl(this.dataSource);
            default:
                return new H2StoredFunctionRepositoryImpl();
        }
    }
}
```


```java
@Profile({"localdev", "dev", "localqa", "qa", "prod"})
@Repository("hottFunction")
    public class MySqlStoredFunctionRepositoryImpl implements IStoredFunctionHottRepository {

    private DataSource dataSource;
    private DecryptDataFunctionDB decryptDataFunctionDB;
    private EncryptDataFunctionDB encryptDataFunctionDB;

    public MySqlStoredFunctionRepositoryImpl(
            DataSource dataSource) {
        this.dataSource = dataSource;
        decryptDataFunctionDB = new DecryptDataFunctionDB(dataSource);
        encryptDataFunctionDB = new EncryptDataFunctionDB(dataSource);
    }

    public String getDecryptData(String data) {
        List<String> result = decryptDataFunctionDB.execute(data);
        return result.get(0);
    }

    public String getEncryptData(String data) {
        if (data.isEmpty()) return "";
        List<String> result = encryptDataFunctionDB.execute(data);
        return result.get(0);
    }
}
```

위와 같이 구현을 해놓을 경우 장점은 각 어플리케이션 프로필 환경에 따른 DB 의존성 구현이 가능하다는 점.

그리고 각 구현체들이 인터페이스 계약관계 안에서 독립적으로 유지보수가 될 수 있으며 추가적인 db 에 대한 구현체도 얼마든지 기존 구현체에 영향을 미치지 않고 적용이 가능하다.

#### 개방 폐쇄 원칙 (OCP) 에 기반한 인프라 스트럭쳐 코드 확장
`기존의 기능 수정(h2 또는 mysql)에 대해서는 닫혀있고 새로운 기능/환경 (새로운 DBMS 인프라스터럭쳐) 에 대해서는 열려있어야 한다.` 라는 원칙을 준수하는 것이기 때문이다.
에를 들어 oracle 에 대한 함수접근을 하고자 하는 경우에는 oracle 전용 함수 구현체를 직접작성하여 인터페이스 계약 준수를 함과 동시에 oracle 에 기반한 고유 메카니즘은 구현체에서 달성할 수 있기 때문이다.
