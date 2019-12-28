---
layout: post
categories: Mybatis
title: mybatis 的 dao 接口跟 xml 文件里面的 sql 是如何建立关系的？
tagline: by 淼淼之森
tags: 
    - 淼淼之森
---

我们在使用Mybatis的时候，都用的是Dao接口和XML文件里的SQL一一对应来进行使用的。那你是否思考过二者是如何建立关系的？
<!--more-->

在开始正文之前，首先解释Dao接口和XML文件里的SQL是如何一一对应的？

一句话讲完就是：mybatis 会先解析这些xml 文件，通过 xml 文件里面的命名空间 （namespace）跟dao 建立关系；然后 xml 中的每段 sql 会有一个id 跟 dao 中的接口进行关联。

那么问题来了: "如果 我有两个这个xml 文件 都跟这个dao 建立关系了，那不是就是冲突了？"

带着这个疑问我们就要开始下面的正题了！

## 一、初始化
首先我们要知道每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为中心的，SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。

但`SqlSessionFactory`是一个接口，它里面其实就两个方法：`openSession`、`getConfiguration`

其中，`openSession`方法是为了获取一个SqlSession对象，完成必要数据库增删改查功能。但是，SqlSessionFactory属性太少了，所以需要`getConfiguration`的配合；来配置mapper映射文件、SQL参数、返回值类型、缓存等属性。
```
/**
 * Creates an {@link SqlSession} out of a connection or a DataSource
 * 
 * @author Clinton Begin
 */
public interface SqlSessionFactory {

  SqlSession openSession();

  SqlSession openSession(boolean autoCommit);
  SqlSession openSession(Connection connection);
  SqlSession openSession(TransactionIsolationLevel level);

  SqlSession openSession(ExecutorType execType);
  SqlSession openSession(ExecutorType execType, boolean autoCommit);
  SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level);
  SqlSession openSession(ExecutorType execType, Connection connection);

  Configuration getConfiguration();

}
```
可以看到getConfiguration是属于Configuration类的一个方法。你可以把它当成一个配置管家。MyBatis所有的配置信息都维持在Configuration对象之中，基本每个对象都会持有它的引用。


但日常开发中我们都是将Mybatis与Spring一起使用的，所以把实例化交给Spring处理。

因此我们可以看下`org.mybatis.spring.SqlSessionFactoryBean`，它实现了InitializingBean接口。这说明，在这个类被实例化之后会调用到afterPropertiesSet()。它只有一个方法
```
public void afterPropertiesSet() throws Exception {
	this.sqlSessionFactory = buildSqlSessionFactory();
}
```
而这个`afterPropertiesSet`方法只有一个动作，就是`buildSqlSessionFactory`。它可以分为两部分来看:
- 1、从配置文件的property属性中加载各种组件，解析配置到configuration中
- 2、加载mapper文件，解析SQL语句，封装成MappedStatement对象，配置到configuration中。

## 二、mapper接口方法是怎样被调用到的？

大致有如下两种方式：

- Mybatis提供的API

使用Mybatis提供的API进行操作，通过获取SqlSession对象，然后根据Statement Id 和参数来操作数据库。
```
String statement = "com.mmzsblog.business.dao.MemberMapper.getMemberList";
List<Member> result = sqlsession.selectList(statement);
```

- mapper接口

定义Mapper接口，并在里面定义一系列业务数据操作方法。在Service层通过注入mapper属性，调用其方法就可以执行数据库操作。就像下面这样
```
public interface MemberMapper {	
	List<Member> getMemberList();
}

@Service
public class MemberServiceImpl implements MemberService{
	@Resource
	private MemberMapper memberMapper;
	
	@Override
	public List<Member> getMemberList() {
		return memberMapper.getMemberList();
	}
}
```
那么，MemberMapper 只是个接口，并没有任何实现类。我们在调用它的时候，它是怎样最终执行到我们的SQL语句的呢？

## 三、Mapper接口的代理创建过程
### 3.1、首先我们会配置需要扫描的基本包路径
通过注解的方式配置：
```
@MapperScan({"com.mmzsblog.business.dao"})
```
或者xml的方式配置：
```
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<property name="basePackage" value="com.mmzsblog.business.dao" />
	<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
</bean>
```
### 3.2、开始扫描
我们来到`org.mybatis.spring.mapper.MapperScannerConfigurer`这个类，可以看到它实现了几个接口。

其中的重点是`BeanDefinitionRegistryPostProcessor`。它可以动态的注册Bean信息，方法为`postProcessBeanDefinitionRegistry()`。
```
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
        if (this.processPropertyPlaceHolders) {
            this.processPropertyPlaceHolders();
        }
        
        // 创建ClassPath扫描器，设置属性，然后调用扫描方法
        ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
        scanner.setAddToConfig(this.addToConfig);
        scanner.setAnnotationClass(this.annotationClass);
        scanner.setMarkerInterface(this.markerInterface);
        scanner.setSqlSessionFactory(this.sqlSessionFactory);
        scanner.setSqlSessionTemplate(this.sqlSessionTemplate);
        scanner.setSqlSessionFactoryBeanName(this.sqlSessionFactoryBeanName);
        scanner.setSqlSessionTemplateBeanName(this.sqlSessionTemplateBeanName);
        scanner.setResourceLoader(this.applicationContext);
        scanner.setBeanNameGenerator(this.nameGenerator);
        // 创建ClassPath扫描器，设置属性，然后调用扫描方法
        scanner.registerFilters();
        scanner.scan(StringUtils.tokenizeToStringArray(this.basePackage, ",; \t\n"));
    }
```
`ClassPathMapperScanner`继承自Spring中的类`ClassPathBeanDefinitionScanner`，所以它的scan方法会调用到父类`ClassPathBeanDefinitionScanner`的scan方法，
```
public class ClassPathBeanDefinitionScanner extends ClassPathScanningCandidateComponentProvider {
    ……
    public int scan(String... basePackages) {
        // 
        int beanCountAtScanStart = this.registry.getBeanDefinitionCount();
        this.doScan(basePackages);
        if (this.includeAnnotationConfig) {
            AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
        }

        return this.registry.getBeanDefinitionCount() - beanCountAtScanStart;
    }
    ……
}    
```
而在父类的scan方法中又调用到子类`ClassPathMapperScanner`重写的doScan方法。
```
public class ClassPathMapperScanner extends ClassPathBeanDefinitionScanner {
    ……
    public Set<BeanDefinitionHolder> doScan(String... basePackages) {
        Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);
        if (beanDefinitions.isEmpty()) {
            this.logger.warn("No MyBatis mapper was found in '" + Arrays.toString(basePackages) + "' package. Please check your configuration.");
        } else {
            this.processBeanDefinitions(beanDefinitions);
        }

        return beanDefinitions;
    }
    ……
}    
```
此处`super.doScan(basePackages)`是Spring中的方法,就不贴代码多叙述了，想详细了解的话，可以自己翻一下源码哦。

### 3.3、bean注册完成并创建sqlSession代理
并且经过上面这些步骤，此时已经扫描到了所有的Mapper接口，并将其注册为BeanDefinition对象。而注册的时候就是用到了上面`doScan`方法中的`processBeanDefinitions`方法。
```
public class ClassPathMapperScanner extends ClassPathBeanDefinitionScanner {
    ……
    // 设置beanClass
    private MapperFactoryBean<?> mapperFactoryBean = new MapperFactoryBean();
    ……
    
    private void processBeanDefinitions(Set<BeanDefinitionHolder> beanDefinitions) {
        Iterator var3 = beanDefinitions.iterator();

        while(var3.hasNext()) {
            BeanDefinitionHolder holder = (BeanDefinitionHolder)var3.next();
            GenericBeanDefinition definition = (GenericBeanDefinition)holder.getBeanDefinition();
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Creating MapperFactoryBean with name '" + holder.getBeanName() + "' and '" + definition.getBeanClassName() + "' mapperInterface");
            }
            // 将mapper接口的名称添加到构造参数
            definition.getConstructorArgumentValues().addGenericArgumentValue(definition.getBeanClassName());
            // 设置BeanDefinition的class
            definition.setBeanClass(this.mapperFactoryBean.getClass());
            // 添加属性addToConfig
            definition.getPropertyValues().add("addToConfig", this.addToConfig);
            boolean explicitFactoryUsed = false;
            // 添加属性sqlSessionFactory
            if (StringUtils.hasText(this.sqlSessionFactoryBeanName)) {
                definition.getPropertyValues().add("sqlSessionFactory", new RuntimeBeanReference(this.sqlSessionFactoryBeanName));
                explicitFactoryUsed = true;
            } else if (this.sqlSessionFactory != null) {
                definition.getPropertyValues().add("sqlSessionFactory", this.sqlSessionFactory);
                explicitFactoryUsed = true;
            }

            if (StringUtils.hasText(this.sqlSessionTemplateBeanName)) {
                if (explicitFactoryUsed) {
                    this.logger.warn("Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.");
                }

                definition.getPropertyValues().add("sqlSessionTemplate", new RuntimeBeanReference(this.sqlSessionTemplateBeanName));
                explicitFactoryUsed = true;
            } else if (this.sqlSessionTemplate != null) {
                if (explicitFactoryUsed) {
                    this.logger.warn("Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.");
                }

                definition.getPropertyValues().add("sqlSessionTemplate", this.sqlSessionTemplate);
                explicitFactoryUsed = true;
            }

            if (!explicitFactoryUsed) {
                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Enabling autowire by type for MapperFactoryBean with name '" + holder.getBeanName() + "'.");
                }

                definition.setAutowireMode(2);
            }
        }
    }
    ……
}    
```
处理的过程相对比较简单，只是往BeanDefinition对象中设置了一些属性。例如：

- 设置beanClass

设置BeanDefinition对象的BeanClass为`MapperFactoryBean<?>`。这就相当于使用MemberMapper注册时：当前的mapper接口在Spring容器中，beanName是memberMapper，beanClass是MapperFactoryBean.class。故在Spring的IOC初始化的时候，实例化的对象就是MapperFactoryBean对象。

- 设置sqlSessionFactory属性

为BeanDefinition对象添加属性sqlSessionFactory，是为了BeanDefinition对象设置PropertyValue的时候，方便调用到setSqlSessionFactory()。

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2019-12/02/1.png) 

### 3.4、创建sqlSession代理类
最终在setSqlSessionFactory这个方法里，sqlSession获取到的是SqlSessionTemplate实例。而在SqlSessionTemplate对象中，主要包含sqlSessionFactory和sqlSessionProxy，而sqlSessionProxy实际上是SqlSession接口的代理对象。实际调用的是代理类的invoke方法。
```
public class MapperProxy<T> implements InvocationHandler, Serializable {
  ……
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, args);
      } else if (isDefaultMethod(method)) {
        return invokeDefaultMethod(proxy, method, args);
      }
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    return mapperMethod.execute(sqlSession, args);
  }
  ……
}  
```


### 3.5、小结
Mapper接口的代理创建过程大致如下：
- 1、扫描mapper接口基本包路径下的所有对象，将其注册为BeanDefinition对象
- 2、设置BeanDefinition的对象的beanClass和sqlSessionFactory属性（而其中获取BeanDefinition对象的时候，调用其工厂方法getObject，返回mapper接口的代理类）
- 3、设置sqlSessionFactory属性的时候，会调用SqlSessionTemplate的构造方法，创建SqlSession接口的代理类

最后我们在Service层，通过
```
@Resource 
private MemberMapper memberDao;
```
注入属性的时候，返回的就是代理类。执行memberDao的方法的时候，实际调用的也是代理类的invoke方法。
## 四、回答最开始的问题
Mybatis在初始化SqlSessionFactoryBean的时候，找到配置需要扫描的基本包路径去解析里面所有的XML文件。重点就在如下两个地方：

**1、创建SqlSource**

Mybatis会把每个SQL标签封装成SqlSource对象。然后根据SQL语句的不同，又分为动态SQL和静态SQL。其中，静态SQL包含一段String类型的sql语句；而动态SQL则是由一个个SqlNode组成。
![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2019-12/02/2.png) 

**2、创建MappedStatement**

XML文件中的每一个SQL标签就对应一个MappedStatement对象，这里面有两个属性很重要。

- id

全限定类名+方法名组成的ID。

- sqlSource

当前SQL标签对应的SqlSource对象。
创建完`MappedStatement`对象，会将它缓存到`Configuration#mappedStatements`中。

前面初始化中提到的Configuration对象，我们知道它就是Mybatis中的配置大管家，基本所有的配置信息都维护在这里。


例如下面这样一段代码：
```xml
<!-- namespace的值就是全限定类名 -->
<mapper namespace="com.java.mmzsblog.dao.MemberMapper">
    ……
    <!-- select标签中id的值就是方法名，它和全限定类中的方法名是对应的 -->
    <select id="getMemberById" resultType="com.java.mmzsblog.entity.member">
        select * from member
        <where>
            <if test="memberId!=null">
                and member_id=#{memberId}
            </if>
        </where>
    </select>
    ……
</mapper>    
```
把所有的XML都解析完成之后，Configuration就包含了所有的SQL信息。然后解析完成的XML大概就是这样了：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2019-12/02/3.png) 

看到上面的图示，聪明如你，也许就大概知道了。当我们执行Mybatis方法的时候，就通过`全限定类名+方法名`找到MappedStatement对象，然后解析里面的SQL内容，执行即可。


