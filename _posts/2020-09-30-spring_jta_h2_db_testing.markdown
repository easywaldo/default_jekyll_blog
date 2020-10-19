---
layout: post
title: Springboot JTA Manager H2 DB 기반으로 테스팅
description: 스프링부트 JTA
categories: code
tags: [ software development, spring boot, jta, h2 ]
author: easywaldo
comments: true
---

### H2 로컬환경에 대한 독립적인 환경 구성의 필요성

> JTA 관련한 의존성 주입이 이뤄지는 경우 MySQL 로 구성된 개발서버 및 운영서버는  DDL 수행등이 될 필요가 없다. 하지만 단위 테스트 등의 테스트 환경에서는 h2 DB 를 로컬로 구동하게 되어 있으며
각 테이블에 대한 스키마 또한 독립적인 sql 파일로 관리가 되고 있으므로 하나의 빈으로 설정하는 것이 불가능하다.
Mysql 에서는 DDL 작업이 수행이 될 필요가 없기 때문이다.

### JTA 를 스프링부트에서 h2 로 구동하는 환경 구성 방법

> JpaVendorAdapter 인터페이스 타입에 대한 DI(의존성 주입) 을 H2 로 설정을 해주면 된다.

아래의 코드를 보면 local 프로필 환경인 경우에 대해서 아래의 프로퍼티를 설정할 수 있다.
- showSql 
- setGenerateDdl
- setDatabase
- setDatabasePlatform 

```java

// 로컬환경이 아닌 DBMS (mysql) 을 이용하는 경우
@Profile({"localdev", "dev", "localqa", "qa", "prod"})
@Bean
public JpaVendorAdapter jpaVendorAdapter() {
    HibernateJpaVendorAdapter hibernateJpaVendorAdapter = new HibernateJpaVendorAdapter();
    hibernateJpaVendorAdapter.setShowSql(true);
    return hibernateJpaVendorAdapter;
}

// 로컬환경에서 인메모리 h2 로 구동되는 경우
@Profile("local")
@Bean
public JpaVendorAdapter jpaVendorAdapterForLocal() {
    HibernateJpaVendorAdapter hibernateJpaVendorAdapter = new HibernateJpaVendorAdapter();
    hibernateJpaVendorAdapter.setShowSql(true);
    hibernateJpaVendorAdapter.setGenerateDdl(true);
    hibernateJpaVendorAdapter.setDatabase(Database.H2);
    hibernateJpaVendorAdapter.setDatabasePlatform("org.hibernate.dialect.H2Dialect");
    return hibernateJpaVendorAdapter;
}
```

위의 코드에서와 같이 JpaVendorAdapter 를 구현한 클래스 타입들에 대해서 의존성 주입이 이뤄지게 되며
이후 EntityManagerFactory 를 설정하는 빈 설정에서 DI 를 수행하게 된다.



```java
    @Bean(name = "entityManagerFactory")
    @DependsOn("jtaTransactionManager")
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
        entityManagerFactoryBean.setDataSource(dataSource());
        entityManagerFactoryBean.setPackagesToScan("com.domain.entity");
        entityManagerFactoryBean.setPersistenceUnitName("master");

        Properties properties = new Properties();
        properties.put("hibernate.transaction.jta.platform", AtomikosJtaPlatform.class.getName());
        properties.put("javax.persistence.transactionType", "JTA");


        // HibernateJpaVendorAdapter 타입에 대한 의존성이 로드가 된다.
        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        vendorAdapter.getJpaPropertyMap().put("hibernate.dialect", hibernateDialect);
        
        // h2 JPA Vendor 의존성을 EntityManager 에 주입한다.
        entityManagerFactoryBean.setJpaVendorAdapter(vendorAdapter);

        entityManagerFactoryBean.setJpaProperties(properties);
        entityManagerFactoryBean.afterPropertiesSet();

        return entityManagerFactoryBean;
    }
```