---
layout: post
title: spring data jpa进阶
category: java
tags: [jpa,spring,数据库,mybatis]
---

* content
{:toc}

不了解spring data jpa的建议先看我之前的文章

[spring data jpa入门示例](http://www.machengyu.net/tech/2019/06/29/spring-data-jpa-demo.html)

本篇的源码地址：

https://github.com/pony-maggie/spring-boot-jpa-advance

## 自定义SQL查询

mybatis擅长大量自定义查询的场景，spring data jpa虽然优势不在这里，但是也支持自定义SQL查询。它主要是通过@Query注解支持的。

```java
@Transactional
@Modifying
	@Query("update Student u set u.name = ?1 where u.id = ?2")
	int modifyNameById(String  name, Long id);

@Query("select u from Student u where u.age = ?1")
	Student findByMyAge(int age);

```

@Query 注解中编写更新类语句, 但必须使用 @Modifying 进行修饰. 以通知 SpringData, 这是一个 UPDATE 或 DELETE 操作

@Transactional表示开启事务，如果不加的话会下面的异常：

```xml
javax.persistence.TransactionRequiredException
```

然后我们在controller里加入测试接口进行测试。

http://localhost:8080/student/byAge?age=16

http://localhost:8080/student/modifyNameById?name=zhangsan&id=3

## 复杂查询如何实现

有些业务场景下我们需要复杂一些的查询，比如指定某些条件进行分页，关联查询等。spring data jpa给我们提供了JpaSpecificationExecutor用来支持这样的场景。


JpaSpecificationExecutor提供了以下接口

```java
public interface JpaSpecificationExecutor<T> {

    T findOne(Specification<T> spec);

    List<T> findAll(Specification<T> spec);

    Page<T> findAll(Specification<T> spec, Pageable pageable);

    List<T> findAll(Specification<T> spec, Sort sort);

    long count(Specification<T> spec);
}
```

传进去的参数Specification是个接口，我们主要的工作就是实现这个接口。我这里实现了一个：

```java
public class MyTestSpec implements Specification {

    @Override
    public Predicate toPredicate(Root root, CriteriaQuery criteriaQuery, CriteriaBuilder criteriaBuilder) {
        Path id = root.get("id");
        Predicate predicate = criteriaBuilder.gt(id, 3);

        return  predicate;

    }
}
```

然后我们在service里调用测试下，

```java
//指定条件分页查询，参数分别是页数和每页的大小
    public Page<Student> findByMySpec(int page, int pageSize){
        Sort sort = new Sort(Sort.Direction.DESC, "id");
        Pageable pageable = PageRequest.of(page, pageSize);

        Page<Student> students = studentRepository.findAll(new MyTestSpec(), pageable);
        return students;
    }
```
打印的SQL如下:

```sql
Hibernate: select student0_.t_id as t_id1_0_, student0_.t_age as t_age2_0_, student0_.t_name as t_name3_0_, student0_.t_school as t_school4_0_ from t_student student0_ where student0_.t_id>3 limit ?, ?
Hibernate: select count(student0_.t_id) as col_0_0_ from t_student student0_ where student0_.t_id>3
```

可以看出我们通过自定义Specification实现了一个id大于3的分页查询操作。所以利用Specification我们可以自定义各种复杂的查询，甚至可以把各种复杂的查询组合起来使用。 


大家可以下载源码自己测试看看效果。


## 关于事务

JpaRepository提供的增删改查接口方法默认都是支持事务的，我们不需要再单独声明。这是因为JpaRepository都有一个默认实现SimpleJpaRespositry，它的定义如下：

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID extends Serializable> implements JpaRepository<T, ID>,
        JpaSpecificationExecutor<T> {
    ......
    @Transactional
    public void delete(ID id) {
    ......
    }

    @Transactional
    public void delete(T entity) {
    ......
    }

    @Transactional
    public void delete(Iterable<? extends T> entities) {
    ......
    }

    @Transactional
    public void deleteInBatch(Iterable<T> entities) {
    ......
    }

    @Transactional
    public void deleteAll() {
    ......
    }

    @Transactional
    public void deleteAllInBatch() {
    ......
    }

    @Transactional
    public <S extends T> S save(S entity) {
    ......
    }

    @Transactional
    public <S extends T> S saveAndFlush(S entity) {
    ......
    }

    @Transactional
    public <S extends T> List<S> save(Iterable<S> entities) {
    ......
    }

    ......
}
```

如果我们再service层调用的DAO方法不是自定义的，那么spring会通过动态代理机制调用SimpleJpaRespositry中的方法。

对于自定义的SQL如果要支持事务只要在方法上加上注解@Transactional即可。比如前面自定义SQL查询中的例子。

除了DAO层，有时候我们需要在service层引入事务，比如有个service方法会执行两步DAO层的操作（删除和插入），我们的业务场景需要这两个步骤要么都成功，要么都失败。

```java
@Transactional
    public void testWithTransactional() throws Exception {

        studentRepository.deleteById(4L);

        Student student = new Student();
        student.setName("test2");
        student.setAge(31);
        student.setSchool("test2 school");
        studentRepository.save(student);

        if (1 == 1) {
            throw new RuntimeException("事务测试抛出一个运行时异常");
        }


    }

    //不带事务注解的两步操作
    public void testWithoutTransactional() throws Exception {

        studentRepository.deleteById(4L);

        Student student = new Student();
        student.setName("test3");
        student.setAge(33);
        student.setSchool("test3 school");
        studentRepository.save(student);

        if (1 == 1) {
            throw new RuntimeException("非事务测试抛出一个运行时异常");
        }
    }
```

```java
//事务异常演示
	@RequestMapping(value = "/twt", method = RequestMethod.GET)
	public void twt() {
		try {
			studentService.testWithTransactional();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	@RequestMapping(value = "/twot", method = RequestMethod.GET)
	public void twot() {
		try {
			studentService.testWithoutTransactional();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```

我写了两个方法演示，一个是带事务注解，一个不带，两个方法都会抛出异常，带事务的方法抛出异常后会进行回滚，而不带事务的方法不会回滚。

**这里需要注意一定是遇到运行时异常才会进行回滚**

## 原理概述

spring在启动的时候会实例化一个Repositories,它会去扫描所有的class，然后找出由我们定义的、继承自org.springframework.data.repository.Repository的接口，然后遍历这些接口，针对每个接口依次创建如下几个实例:

* SimpleJpaRespositry——用来进行默认的DAO操作，是所有Repository的默认实现
* JpaRepositoryFactoryBean——装配bean，装载了动态代理Proxy，会以对应的DAO的beanName为key注册到DefaultListableBeanFactory中，在需要被注入的时候从这个bean中取出对应的动态代理Proxy注入给DAO
* JdkDynamicAopProxy——动态代理对应的InvocationHandler，负责拦截DAO接口的所有的方法调用，然后做相应处理，比如findByUsername被调用的时候会先经过这个类的invoke方法


**参考**

http://www.luckyzz.com/java/spring-data-jpa/

https://blog.csdn.net/J080624/article/details/84581231