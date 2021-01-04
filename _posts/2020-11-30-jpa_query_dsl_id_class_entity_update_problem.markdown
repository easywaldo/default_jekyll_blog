---
layout: post
title: Id class 가 설정된 Jpa entity 를 QueryDSL 업데이트 시 문제점
description: ELK Stack
date:   2020-11-30 23:55:00 +0530
categories: code
tags: [ JPA, QueryDSL ]
author: easywaldo
comments: true
---

##### 엔터티 조회 후 객체 수정 
> JPA 엔터티를 Repository 에서 조회하여 이를 @Transactional 범위 안에서 업데이트를 한다.


```java
@Transactional
public void updatePassword(InfluencerSaveRequestDto requestDto {
    Member member = this.memberRepository.findBySeqNo(requestDto.getMemberSeq())
        .orElseThrow(() -> new IllegalArgumentException("not exists member."));

    member.updatePassword(
        SHAEncryptServiceImpl.getSHA512(requestDto.getNewPassword()), editPwd);
    memberRepository.save(member);
```

##### 복합키로 설정된 엔터티에 대한 QueryDSL 업데이트
```java
@Getter
@NoArgsConstructor
@Entity
@IdClass(OrderCartId.class)
@Table(name = "order_cart")
public class OrderCart {

    @Id
    @Column(name = "user_id")
    private String userId;

    @Id
    @Column(name = "product_num")
    private String productlNum;

    @Id
    @Column(name = "product_option")
    private String productOption;

    @Id
    @Column(name = "product_option2")
    private String productOption2;

    @Column(name = "count")
    private BigInteger productCount;

    @Column(name = "product_option_seq")
    private Long productOptionSeq;

```

위와 같이 여러개의 컬럼들이 복합키로 설정되어 있는 엔터티를 QueryDSL 로 업데이트 하고자 하는 경우에는
주의 사항이 있다.
아래와 같이 실행을 하였을 때 실제 실행되는 Query 는 다르다는 것을 확인할 수 있었다.

```java
    jpaQueryFactory.update(qOrderCart)
        .where(qOrderCart.userId.eq(userCart.getUserId()))
        .set(qOrderCart.productNum, "update-prod")
        .execute();
```

 ```sql
    update OrderCart
    set product_num = 'update-prod'
    where user_id = '...'
    and product_num = '...'
    and product_option = '...'
    and product_option2 = '...'
    and product_option_seq = '...'
 ```

 분명 QueryDSL 에서의 조건문을 userId 로만 설정을 하였음에도 엔터티는 @Id 로 설정된 필드에 대해서
 수행을 한 것이다.
 @IdClass 로 복합키로 설정이 된 엔터티의 경우 업데이트를 하게 되는 update(엔터티 인스턴스) 에 대한
 식별을 @Id 로 정의된 필드들로 파악을 하기 때문에 이러한 문제가 발생한 듯 한다.


```java

public class OrderCartId implements Serializable {
    private String userId;
    private String productNum;
    private String productOption;
    private String productOption2;
}
```

##### JPQL 를 이용하여 수정을 시도해보자!!
정상적으로 의도한 데로 쿼리 실행이 되었음을 확인할 수 있었다.
```java
@Transactional
@Modifying
@Query(value = "UPDATE user_cart" +
        " set product_num = ?2 " +
        " where user_id = ?1", nativeQuery = true)
void updateState(String hottOrderNum, String state);
```

##### 다른 방법을 이용한 수정은 ??
JpaRepository inteface 에서의 delete 메서드는 제공이 되지만 수정을 하기 위한 update 메서드는
제공되지 않는다.

개인적으로 수정 쿼리가 복잡한 비즈니스의 경우에는 아래의 방식들이 현실적으로 이용이 가능한 방법이라고 생각이 든다.

- nativeQuery를 이용한 JPQL 방식
- 조회 후 필드 수정 방식
- entityManager 를 이용한 native Query 방식



