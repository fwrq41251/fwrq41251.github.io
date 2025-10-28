---
title:  spring data jpa fetch mode
date: 2016-05-26
tags: 
  - jpa

slug:  spring-data-jpa-fetch-mode
---

虽然hibernate里好几种fetch mode.

* SELECT
* SELECT with BatchSize
* JOIN
* SUBSELCT

但是JPA里只有两种类型的fetching:`EAGER`和`LAZY`。在使用spring data jpa的时候，为了解决n+1查询问题，我们需要在查询中手动的控制fetch mode。

```java
public void testPagedSpecificationProjection() {
        Specification<Person> spec = new Specification<Person>() {
            public Predicate toPredicate(Root<Person> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                if (Long.class != query.getResultType()) {
                    root.fetch(Person_.addresses);
                }
                return cb.conjunction();
            }
        };
        Pageable pageable = new PageRequest(0, 1);
        Page<Person> page = personRepo.findAll(spec, pageable);
    }

```

第四行判断了返回类型是否为Long，是因为分页查询时spring data会先执行一次count查询，此时关联对count查询是没有意义的，会一个抛异常：

> org.hibernate.QueryException: query specified join fetching, but the owner of the fetched association was not present in the select list

如果不是分页查询，则不需要判断返回类型。

顺带一提，关于jpa里的@OneToOne关联，如果关联的表允许为空，即`optional=true`，那么`fetch = FetchType.LAZY`是不起作用的，具体的解释参考：[SomeExplanationsOnLazyLoadingone-to-one](https://developer.jboss.org/wiki/SomeExplanationsOnLazyLoadingone-to-one)。`@OneToOne`的双向关联一定要慎用，一不小心就会造成n+1查询问题。

参考:[https://jira.spring.io/browse/DATAJPA-105](https://jira.spring.io/browse/DATAJPA-105)

[http://jdpgrailsdev.github.io/blog/2014/09/09/spring_data_hibernate_join.html](http://jdpgrailsdev.github.io/blog/2014/09/09/spring_data_hibernate_join.html)
