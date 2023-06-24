---
title: "JPQL 조건에 따른 쿼리문 생성"

categories:
- SpringBoot

tags:
- Spring
- JPQL
---

# 동적쿼리

게시글에서 검색 조건에 따라 정렬 기준을 
다르게 나오게 하고 싶어서 JPQL에 order by 만 추가하면 된다고 생각했다.

```java
public List<PostsResponseDto> getPostList(Pageable pageable, String order) {
    if(order.equals("date"))
        order = "updated_date";

    return postRepository.findAll(order, pageable).stream()
            .map(PostsResponseDto::from)
            .collect(Collectors.toList());
}
```

이렇게 service 단에서 findAll문을 통해 가져오려 했는데

```java
@Query(value = "select p from Post p left join fetch p.comments where p.isDeleted = false order by :order desc",
    countQuery = "select count(p) from Post p where p.isDeleted = false")
public Page<Post> findAll(@Param("order") String order, Pageable pageable);
```


JPQL은 일반적인 SQL과 다르게 엔티티 객체를 대상으로 쿼리를 날려서 동작방식이 좀 다르고
파라미터 바인딩시에 :를 이용해서 변수명으로 쿼리를 조작할 수는 없다고 한다.
심지어 @Query에서 변수명을 넣으면 이 변수명이 그대로 문자열로 바인딩 되어서 내가 원하는 결과가
나오지 않았다.

<br>
그래서

```java
@Query(value = "select p from Post p left join fetch p.comments where p.isDeleted = false order by "
            + "case when :order = 'views' then p.views end desc, "
            + "case when :order = 'likes' then p.likes end desc, "
            + "case when :order = 'updated_date' then p.updatedDate end desc",
            countQuery = "select count(p) from Post p where p.isDeleted = false")
    public Page<Post> findAll(@Param("order") String order, Pageable pageable);
```

이러한 방식으로 case when 문을 이용해서 각각 변수명에 따라 다르게 파라미터 값을 바인딩
시켜줘야 내가 원하는 대로 결과가 적용된다.


코드 가독성 측면도 있고 컴파일 시점에 오류 체크도 가능하게 하려면
동적 쿼리가 많이 활용되는 곳에서는 QueryDSL을 활용하는 것도 나쁘지 않다.