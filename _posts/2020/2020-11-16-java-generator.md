---
layout: post
categories: java
title: 你还在手写crud吗，看完这篇文章，绝对赚了
tagline: by 炸鸡可乐
tags: 
  - 炸鸡可乐
---

看完本文，你一定会有所收获

<!--more-->

### 一、介绍
我记得最早刚步入互联网行业的时候，当时按照 MVC 的思想和模型，每次开发新功能，会依次编写 **dao、service、controller**相关服务类，包括对应的 **dto、entity、vo** 等等实体类，如果有多张单表，也会重复的编写相似的代码。

实际上，当仔细的总结一下，对于任何一张单表的操作，基本都是围绕**增（Create ）、删（Delete ）、改（Update ）、查（Retrieve ）四个方向**进行数据操作，简称 CRUD！

他们除了表名和存储空间不一样，基本的 CRUD 思路基本都是一样的。

为了解决这些重复劳动的痛点，业界逐渐开源了一批代码生成器，目的也很简单，就是**为了减少手工操作的繁琐，集中精力在业务开发上，提升开发效率**。

而今天，我们所要介绍的也是**代码生成器**，很多初学者可能觉得代码生成器很高深。代码生成器其实是一个很简单的东西，一点都不高深，当你看完本文的时候，你会完全掌握代码生成器的逻辑，甚至可以根据自己的项目情况，进行深度定制。

### 二、实现思路
下面我们就以`SpringBoot`项目为例，数据持久化操作采用`Mybatis`，数据库采用`Mysql`，编写一个自动生成增、删、改、查等基础功能的代码生成器，内容包括`controller`、`service`、`dao`、`entity`、`dto`、`vo`等信息。

实现思路如下：

* 第一步：获取表字段名称、类型、表注释等信息
* 第二步：基于 freemarker 模板引擎，编写相应的模板
* 第三步：根据对应的模板，生成相应的 java 代码


#### 2.1、获取表结构
首先我们创建一张`test_db`表，脚本如下：
```sql
CREATE TABLE test_db (
  id bigint(20) unsigned NOT NULL COMMENT '主键ID',
  name varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '名称',
  is_delete tinyint(4) NOT NULL DEFAULT '0' COMMENT '是否删除 1：已删除；0：未删除',
  create_time datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  update_time datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (id),
  KEY idx_create_time (create_time) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='测试表';
```

表创建完成之后，基于`test_db`表，我们查询对应的表结果字段名称、类型、备注信息，**这些信息收集将用于后续进行代码生成器所使用**！

```sql
# 获取对应表结构
SELECT column_name, data_type, column_comment FROM information_schema.columns WHERE table_schema = 'yjgj_base' AND table_name = 'test_db'
```

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-generator/01.jpg)

同时，获取对应表注释，**用于生成备注信息**！

```sql
# 获取对应表注释
SELECT TABLE_COMMENT FROM INFORMATION_SCHEMA.TABLES WHERE table_schema = 'yjgj_base' AND table_name = 'test_db'
```
![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-generator/02.jpg)


#### 2.2、编写模板

* 编写`mapper`模板，涵盖新增、修改、删除、查询等信息

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="${daoPackageName}.${daoName}">

	<!--BaseResultMap-->
	<resultMap id="BaseResultMap" type="${entityPackageName}.${entityName}">
        <#list columns as pro>
            <#if pro.proName == primaryId>
				<id column="${primaryId}" property="${primaryId}" jdbcType="${pro.fieldType}"/>
            <#else>
				<result column="${pro.fieldName}" property="${pro.proName}" jdbcType="${pro.fieldType}"/>
            </#if>
        </#list>
	</resultMap>

	<!--Base_Column_List-->
	<sql id="Base_Column_List">
        <#list columns as pro>
            <#if pro_index == 0>${pro.fieldName}<#else>,${pro.fieldName}</#if>
        </#list>
	</sql>

	<!--批量插入-->
	<insert id="insertList" parameterType="java.util.List">
		insert into ${tableName} (
        <#list columns as pro>
            <#if pro_index == 0>${pro.fieldName},<#elseif pro_index == 1>${pro.fieldName}<#else>,${pro.fieldName}</#if>
        </#list>
		)
		values
		<foreach collection ="list" item="obj" separator =",">
			<trim prefix=" (" suffix=")" suffixOverrides=",">
                <#list columns as pro>
                    ${r"#{obj." + pro.proName + r"}"},
                </#list>
			</trim>
		</foreach >
	</insert>

	<!--按需新增-->
	<insert id="insertPrimaryKeySelective" parameterType="${entityPackageName}.${entityName}">
		insert into ${tableName}
		<trim prefix="(" suffix=")" suffixOverrides=",">
            <#list columns as pro>
				<if test="${pro.proName} != null">
                    ${pro.fieldName},
				</if>
            </#list>
		</trim>
		<trim prefix="values (" suffix=")" suffixOverrides=",">
            <#list columns as pro>
				<if test="${pro.proName} != null">
                    ${r"#{" + pro.proName + r",jdbcType=" + pro.fieldType +r"}"},
				</if>
            </#list>
		</trim>
	</insert>

	<!-- 按需修改-->
	<update id="updatePrimaryKeySelective" parameterType="${entityPackageName}.${entityName}">
		update ${tableName}
		<set>
            <#list columns as pro>
                <#if pro.fieldName != primaryId && pro.fieldName != primaryId>
					<if test="${pro.proName} != null">
                        ${pro.fieldName} = ${r"#{" + pro.proName + r",jdbcType=" + pro.fieldType +r"}"},
					</if>
                </#if>
            </#list>
		</set>
		where ${primaryId} = ${r"#{" + "${primaryId}" + r",jdbcType=BIGINT}"}
	</update>

	<!-- 按需批量修改-->
	<update id="updateBatchByIds" parameterType="java.util.List">
		update ${tableName}
		<trim prefix="set" suffixOverrides=",">
            <#list columns as pro>
                <#if pro.fieldName != primaryId && pro.fieldName != primaryId>
					<trim prefix="${pro.fieldName}=case" suffix="end,">
						<foreach collection="list" item="obj" index="index">
							<if test="obj.${pro.proName} != null">
								when id = ${r"#{" + "obj.id" + r"}"}
								then  ${r"#{obj." + pro.proName + r",jdbcType=" + pro.fieldType +r"}"}
							</if>
						</foreach>
					</trim>
                </#if>
            </#list>
		</trim>
		where
		<foreach collection="list" separator="or" item="obj" index="index" >
			id = ${r"#{" + "obj.id" + r"}"}
		</foreach>
	</update>

	<!-- 删除-->
	<delete id="deleteByPrimaryKey" parameterType="java.lang.Long">
		delete from ${tableName}
		where ${primaryId} = ${r"#{" + "${primaryId}" + r",jdbcType=BIGINT}"}
	</delete>

	<!-- 查询详情 -->
	<select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.Long">
		select
		<include refid="Base_Column_List"/>
		from ${tableName}
		where ${primaryId} = ${r"#{" + "${primaryId}" + r",jdbcType=BIGINT}"}
	</select>

	<!-- 按需查询 -->
	<select id="selectByPrimaryKeySelective" resultMap="BaseResultMap" parameterType="${entityPackageName}.${entityName}">
		select
		<include refid="Base_Column_List"/>
		from ${tableName}
	</select>

	<!-- 批量查询-->
	<select id="selectByIds" resultMap="BaseResultMap" parameterType="java.util.List">
		select
		<include refid="Base_Column_List"/>
		from ${tableName}
		<where>
			<if test="ids != null">
				and ${primaryId} in
				<foreach item="item" index="index" collection="ids" open="(" separator="," close=")">
                    ${r"#{" + "item" + r"}"}
				</foreach>
			</if>
		</where>
	</select>

	<!-- 根据条件查询 -->
	<select id="selectByMap" resultMap="BaseResultMap" parameterType="java.util.Map">
		select
		<include refid="Base_Column_List"/>
		from ${tableName}
	</select>

	<!-- 查询${entityName}总和 -->
	<select id="countPage" resultType="int" parameterType="${dtoPackageName}.${dtoName}">
		select count(${primaryId})
		from ${tableName}
	</select>

	<!-- 查询${entityName}列表 -->
	<select id="selectPage" resultMap="BaseResultMap" parameterType="${dtoPackageName}.${dtoName}">
		select
		<include refid="Base_Column_List"/>
		from ${tableName}
		limit ${r"#{" + "start,jdbcType=INTEGER" + r"}"},${r"#{" + "end,jdbcType=INTEGER" + r"}"}
	</select>

</mapper>
```

* 编写`dao`数据访问模板

```
package ${daoPackageName};

import com.example.generator.core.BaseMapper;
import java.util.List;
import ${entityPackageName}.${entityName};
import ${dtoPackageName}.${dtoName};

/**
*
* @ClassName: ${daoName}
* @Description: 数据访问接口
* @author ${authorName}
* @date ${currentTime}
*
*/
public interface ${daoName} extends BaseMapper<${entityName}>{

	int countPage(${dtoName} ${dtoName?uncap_first});

	List<${entityName}> selectPage(${dtoName} ${dtoName?uncap_first});
}
```

* 编写`service`服务接口模板

```
package ${servicePackageName};

import com.example.generator.core.BaseService;
import com.example.generator.common.Pager;
import ${voPackageName}.${voName};
import ${dtoPackageName}.${dtoName};
import ${entityPackageName}.${entityName};

/**
 *
 * @ClassName: ${serviceName}
 * @Description: ${entityName}业务访问接口
 * @author ${authorName}
 * @date ${currentTime}
 *
 */
public interface ${serviceName} extends BaseService<${entityName}> {

	/**
	 * 分页列表查询
	 * @param request
	 */
	Pager<${voName}> getPage(${dtoName} request);
}
```

* 编写`serviceImpl`服务实现类模板

```
package ${serviceImplPackageName};

import com.example.generator.common.Pager;
import com.example.generator.core.BaseServiceImpl;
import com.example.generator.test.service.TestEntityService;
import org.springframework.beans.BeanUtils;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.List;


import ${daoPackageName}.${daoName};
import ${entityPackageName}.${entityName};
import ${dtoPackageName}.${dtoName};
import ${voPackageName}.${voName};


@Service
public class ${serviceImplName} extends BaseServiceImpl<${daoName}, ${entityName}> implements ${serviceName} {

	private static final Logger log = LoggerFactory.getLogger(${serviceImplName}.class);

	/**
	 * 分页列表查询
	 * @param request
	 */
	public Pager<${voName}> getPage(${dtoName} request) {
		List<${voName}> resultList = new ArrayList();
		int count = super.baseMapper.countPage(request);
		List<${entityName}> dbList = count > 0 ? super.baseMapper.selectPage(request) : new ArrayList<>();
		if(!CollectionUtils.isEmpty(dbList)){
			dbList.forEach(source->{
				${voName} target = new ${voName}();
				BeanUtils.copyProperties(source, target);
				resultList.add(target);
			});
		}
		return new Pager(request.getCurrPage(), request.getPageSize(), count, resultList);
	}
}
```

* 编写`controller`控制层模板

```
package ${controllerPackageName};

import com.example.generator.common.IdRequest;
import com.example.generator.common.Pager;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.Objects;

import ${servicePackageName}.${serviceName};
import ${entityPackageName}.${entityName};
import ${dtoPackageName}.${dtoName};
import ${voPackageName}.${voName};

/**
 *
 * @ClassName: ${controllerName}
 * @Description: 外部访问接口
 * @author ${authorName}
 * @date ${currentTime}
 *
 */
@RestController
@RequestMapping("/${entityName?uncap_first}")
public class ${controllerName} {

	@Autowired
	private ${serviceName} ${serviceName?uncap_first};

	/**
	 * 分页列表查询
	 * @param request
	 */
	@PostMapping(value = "/getPage")
	public Pager<${voName}> getPage(@RequestBody ${dtoName} request){
		return ${serviceName?uncap_first}.getPage(request);
	}

	/**
	 * 查询详情
	 * @param request
	 */
	@PostMapping(value = "/getDetail")
	public ${voName} getDetail(@RequestBody IdRequest request){
		${entityName} source = ${serviceName?uncap_first}.selectById(request.getId());
		if(Objects.nonNull(source)){
			${voName} result = new ${voName}();
			BeanUtils.copyProperties(source, result);
			return result;
		}
		return null;
	}

	/**
	 * 新增操作
	 * @param request
	 */
	@PostMapping(value = "/save")
	public void save(${dtoName} request){
		${entityName} entity = new ${entityName}();
		BeanUtils.copyProperties(request, entity);
		${serviceName?uncap_first}.insert(entity);
	}

	/**
	 * 编辑操作
	 * @param request
	 */
	@PostMapping(value = "/edit")
	public void edit(${dtoName} request){
		${entityName} entity = new ${entityName}();
		BeanUtils.copyProperties(request, entity);
		${serviceName?uncap_first}.updateById(entity);
	}

	/**
	 * 删除操作
	 * @param request
	 */
	@PostMapping(value = "/delete")
	public void delete(IdRequest request){
		${serviceName?uncap_first}.deleteById(request.getId());
	}
}
```


* 编写`entity`实体类模板

```
package ${entityPackageName};

import java.io.Serializable;
import java.math.BigDecimal;
import java.util.Date;

/**
 *
 * @ClassName: ${entityName}
 * @Description: ${tableDes!}实体类
 * @author ${authorName}
 * @date ${currentTime}
 *
 */
public class ${entityName} implements Serializable {

	private static final long serialVersionUID = 1L;
	
	<#--属性遍历-->
	<#list columns as pro>
	<#--<#if pro.proName != primaryId
	&& pro.proName != 'remarks'
	&& pro.proName != 'createBy'
	&& pro.proName != 'createDate'
	&& pro.proName != 'updateBy'
	&& pro.proName != 'updateDate'
	&& pro.proName != 'delFlag'
	&& pro.proName != 'currentUser'
	&& pro.proName != 'page'
	&& pro.proName != 'sqlMap'
	&& pro.proName != 'isNewRecord'
	></#if>-->
	/**
	 * ${pro.proDes!}
	 */
	private ${pro.proType} ${pro.proName};
	</#list>

	<#--属性get||set方法-->
	<#list columns as pro>
	public ${pro.proType} get${pro.proName?cap_first}() {
		return this.${pro.proName};
	}

	public ${entityName} set${pro.proName?cap_first}(${pro.proType} ${pro.proName}) {
		this.${pro.proName} = ${pro.proName};
		return this;
	}
	</#list>
}
```

* 编写`dto`实体类模板

```
package ${dtoPackageName};

import com.example.generator.core.BaseDTO;
import java.io.Serializable;

/**
 * @ClassName: ${dtoName}
 * @Description: 请求实体类
 * @author ${authorName}
 * @date ${currentTime}
 *
 */
public class ${dtoName} extends BaseDTO {

}
```

* 编写`vo`视图实体类模板

```
package ${voPackageName};

import java.io.Serializable;

/**
 * @ClassName: ${voName}
 * @Description: 返回视图实体类
 * @author ${authorName}
 * @date ${currentTime}
 *
 */
public class ${voName} implements Serializable {

	private static final long serialVersionUID = 1L;
}
```

可能细心的网友已经看到了，在模板中我们用到了`BaseMapper`、`BaseService`、`BaseServiceImpl`等等服务类。

之所以有这三个类，是因为在模板中，我们有大量的相同的方法名包括逻辑也相似，除了所在实体类不一样意以外，其他都一样，因此我们可以借助泛型类来将这些服务抽成公共的部分。

* `BaseMapper`，主要负责将`dao`层的公共方法抽出来

```
package com.example.generator.core;

import org.apache.ibatis.annotations.Param;

import java.io.Serializable;
import java.util.List;
import java.util.Map;

/**
 * @author pzblog
 * @Description
 * @since 2020-11-11
 */
public interface BaseMapper<T> {

    /**
     * 批量插入
     * @param list
     * @return
     */
    int insertList(@Param("list") List<T> list);

    /**
     * 按需插入一条记录
     * @param entity
     * @return
     */
    int insertPrimaryKeySelective(T entity);

    /**
     * 按需修改一条记录（通过主键ID）
     * @return
     */
    int updatePrimaryKeySelective(T entity);

    /**
     * 批量按需修改记录（通过主键ID）
     * @param list
     * @return
     */
    int updateBatchByIds(@Param("list") List<T> list);

    /**
     * 根据ID删除
     * @param id 主键ID
     * @return
     */
    int deleteByPrimaryKey(Serializable id);

    /**
     * 根据ID查询
     * @param id 主键ID
     * @return
     */
    T selectByPrimaryKey(Serializable id);

    /**
     * 按需查询
     * @param entity
     * @return
     */
    List<T> selectByPrimaryKeySelective(T entity);

    /**
     * 批量查询
     * @param ids 主键ID集合
     * @return
     */
    List<T> selectByIds(@Param("ids") List<? extends Serializable> ids);

    /**
     * 查询（根据 columnMap 条件）
     * @param columnMap 表字段 map 对象
     * @return
     */
    List<T> selectByMap(Map<String, Object> columnMap);
}
```

* `BaseService`，主要负责将`service`层的公共方法抽出来

```
package com.example.generator.core;

import java.io.Serializable;
import java.util.List;
import java.util.Map;

/**
 * @author pzblog
 * @Description 服务类
 * @since 2020-11-11
 */
public interface BaseService<T> {

    /**
     * 新增
     * @param entity
     * @return boolean
     */
    boolean insert(T entity);

    /**
     * 批量新增
     * @param list
     * @return boolean
     */
    boolean insertList(List<T> list);

    /**
     * 通过ID修改记录（如果想全部更新，只需保证字段都不为NULL）
     * @param entity
     * @return boolean
     */
    boolean updateById(T entity);

    /**
     * 通过ID批量修改记录（如果想全部更新，只需保证字段都不为NULL）
     * @param list
     * @return boolean
     */
    boolean updateBatchByIds(List<T> list);

    /**
     * 根据ID删除
     * @param id 主键ID
     * @return boolean
     */
    boolean deleteById(Serializable id);

    /**
     * 根据ID查询
     * @param id 主键ID
     * @return
     */
    T selectById(Serializable id);

    /**
     * 按需查询
     * @param entity
     * @return
     */
    List<T> selectByPrimaryKeySelective(T entity);

    /**
     * 批量查询
     * @param ids
     * @return
     */
    List<T> selectByIds(List<? extends Serializable> ids);

    /**
     * 根据条件查询
     * @param columnMap
     * @return
     */
    List<T> selectByMap(Map<String, Object> columnMap);

}
```

* `BaseServiceImpl`，`service`层的公共方法具体实现类

```
package com.example.generator.core;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.transaction.annotation.Transactional;

import java.io.Serializable;
import java.util.List;
import java.util.Map;

/**
 * @author pzblog
 * @Description 实现类（ 泛型说明：M 是 mapper 对象，T 是实体）
 * @since 2020-11-11
 */
public abstract class BaseServiceImpl<M extends BaseMapper<T>, T> implements BaseService<T>{

    @Autowired
    protected M baseMapper;

    /**
     * 新增
     * @param entity
     * @return boolean
     */
    @Override
    @Transactional(rollbackFor = {Exception.class})
    public boolean insert(T entity){
        return returnBool(baseMapper.insertPrimaryKeySelective(entity));
    }

    /**
     * 批量新增
     * @param list
     * @return boolean
     */
    @Override
    @Transactional(rollbackFor = {Exception.class})
    public boolean insertList(List<T> list){
        return returnBool(baseMapper.insertList(list));
    }

    /**
     * 通过ID修改记录（如果想全部更新，只需保证字段都不为NULL）
     * @param entity
     * @return boolean
     */
    @Override
    @Transactional(rollbackFor = {Exception.class})
    public boolean updateById(T entity){
        return returnBool(baseMapper.updatePrimaryKeySelective(entity));
    }

    /**
     * 通过ID批量修改记录（如果想全部更新，只需保证字段都不为NULL）
     * @param list
     * @return boolean
     */
    @Override
    @Transactional(rollbackFor = {Exception.class})
    public boolean updateBatchByIds(List<T> list){
        return returnBool(baseMapper.updateBatchByIds(list));
    }

    /**
     * 根据ID删除
     * @param id 主键ID
     * @return boolean
     */
    @Override
    @Transactional(rollbackFor = {Exception.class})
    public boolean deleteById(Serializable id){
        return returnBool(baseMapper.deleteByPrimaryKey(id));
    }

    /**
     * 根据ID查询
     * @param id 主键ID
     * @return
     */
    @Override
    public T selectById(Serializable id){
        return baseMapper.selectByPrimaryKey(id);
    }

    /**
     * 按需查询
     * @param entity
     * @return
     */
    @Override
    public List<T> selectByPrimaryKeySelective(T entity){
        return baseMapper.selectByPrimaryKeySelective(entity);
    }

    /**
     * 批量查询
     * @param ids
     * @return
     */
    @Override
    public List<T> selectByIds(List<? extends Serializable> ids){
        return baseMapper.selectByIds(ids);
    }

    /**
     * 根据条件查询
     * @param columnMap
     * @return
     */
    @Override
    public List<T> selectByMap(Map<String, Object> columnMap){
        return baseMapper.selectByMap(columnMap);
    }

    /**
     * 判断数据库操作是否成功
     * @param result 数据库操作返回影响条数
     * @return boolean
     */
    protected boolean returnBool(Integer result) {
        return null != result && result >= 1;
    }

}
```

在此，还封装来其他的类，例如 dto 公共类`BaseDTO`，分页类`Pager`，还有 id 请求类`IdRequest`。

* `BaseDTO`公共类

```
public class BaseDTO implements Serializable {

    /**
     * 请求token
     */
    private String token;

    /**
     * 当前页数
     */
    private Integer currPage = 1;

    /**
     * 每页记录数
     */
    private Integer pageSize = 20;

    /**
     * 分页参数（第几行）
     */
    private Integer start;

    /**
     * 分页参数（行数）
     */
    private Integer end;

    /**
     * 登录人ID
     */
    private String loginUserId;

    /**
     * 登录人名称
     */
    private String loginUserName;

    public String getToken() {
        return token;
    }

    public BaseDTO setToken(String token) {
        this.token = token;
        return this;
    }

    public Integer getCurrPage() {
        return currPage;
    }

    public BaseDTO setCurrPage(Integer currPage) {
        this.currPage = currPage;
        return this;
    }

    public Integer getPageSize() {
        return pageSize;
    }

    public BaseDTO setPageSize(Integer pageSize) {
        this.pageSize = pageSize;
        return this;
    }

    public Integer getStart() {
        if (this.currPage != null && this.currPage > 0) {
            start = (currPage - 1) * getPageSize();
            return start;
        }
        return start == null ? 0 : start;
    }

    public BaseDTO setStart(Integer start) {
        this.start = start;
        return this;
    }

    public Integer getEnd() {
        return getPageSize();
    }

    public BaseDTO setEnd(Integer end) {
        this.end = end;
        return this;
    }

    public String getLoginUserId() {
        return loginUserId;
    }

    public BaseDTO setLoginUserId(String loginUserId) {
        this.loginUserId = loginUserId;
        return this;
    }

    public String getLoginUserName() {
        return loginUserName;
    }

    public BaseDTO setLoginUserName(String loginUserName) {
        this.loginUserName = loginUserName;
        return this;
    }

}
```

* `Pager`分页类

```
public class Pager<T extends Serializable> implements Serializable {

    private static final long serialVersionUID = -6557244954523041805L;

    /**
     * 当前页数
     */
    private int currPage;

    /**
     * 每页记录数
     */
    private int pageSize;

    /**
     * 总页数
     */
    private int totalPage;

    /**
     * 总记录数
     */
    private int totalCount;

    /**
     * 列表数据
     */
    private List<T> list;

    public Pager(int currPage, int pageSize) {
        this.currPage = currPage;
        this.pageSize = pageSize;
    }

    public Pager(int currPage, int pageSize, int totalCount, List<T> list) {
        this.currPage = currPage;
        this.pageSize = pageSize;
        this.totalPage = (int) Math.ceil((double) totalCount / pageSize);;
        this.totalCount = totalCount;
        this.list = list;
    }

    public int getCurrPage() {
        return currPage;
    }

    public Pager setCurrPage(int currPage) {
        this.currPage = currPage;
        return this;
    }

    public int getPageSize() {
        return pageSize;
    }

    public Pager setPageSize(int pageSize) {
        this.pageSize = pageSize;
        return this;
    }

    public int getTotalPage() {
        return totalPage;
    }

    public Pager setTotalPage(int totalPage) {
        this.totalPage = totalPage;
        return this;
    }

    public int getTotalCount() {
        return totalCount;
    }

    public Pager setTotalCount(int totalCount) {
        this.totalCount = totalCount;
        return this;
    }

    public List<T> getList() {
        return list;
    }

    public Pager setList(List<T> list) {
        this.list = list;
        return this;
    }
}

```

* `IdRequest`公共请求类

```
public class IdRequest extends BaseDTO {

    private Long id;

    public Long getId() {
        return id;
    }

    public IdRequest setId(Long id) {
        this.id = id;
        return this;
    }
}
```

#### 2.3、编写代码生成器
前两部分主要介绍的是如何获取对应的表结构，以及代码器运行之前的准备工作。

其实代码生成器，很简单，其实就是一个`main`方法，没有想象中的那么复杂。

处理思路也很简单，过程如下：

* 1、定义基本变量，例如包名路径、模块名、表名、转换后的实体类、以及数据库连接配置，我们可以将其写入配置文件
* 2、读取配置文件，封装对应的模板中定义的变量
* 3、根据对应的模板文件和变量，生成对应的java文件

##### 2.3.1、创建配置文件，定义变量
小编我用的是`application.properties`配置文件来定义变量，这个没啥规定，你也可以自定义文件名，内容如下：

```
#包前缀
packageNamePre=com.example.generator
#模块名称
moduleName=test
#表
tableName=test_db
#实体类名称
entityName=TestEntity
#主键ID
primaryId=id
#作者
authorName=pzblog
#数据库名称
databaseName=yjgj_base

#数据库服务器IP地址
ipName=127.0.0.1
#数据库服务器端口
portName=3306
#用户名
userName=root
#密码
passWord=123456

#文件输出路径，支持自定义输出路径，如果为空，默认取当前工程的src/main/java路径
outUrl=
```
##### 2.3.2、根据模板生成对应的java代码

* 首先，读取配置文件变量

```
public class SystemConstant {

    private static Properties properties = new Properties();

    static {
        try {
            // 加载上传文件设置参数：配置文件
            properties.load(SystemConstant.class.getClassLoader().getResourceAsStream("application.properties"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static final String tableName = properties.getProperty("tableName");
    public static final String entityName = properties.getProperty("entityName");
    public static final String packageNamePre = properties.getProperty("packageNamePre");
    public static final String outUrl = properties.getProperty("outUrl");
    public static final String databaseName = properties.getProperty("databaseName");
    public static final String ipName = properties.getProperty("ipName");
    public static final String portName = properties.getProperty("portName");
    public static final String userName = properties.getProperty("userName");
    public static final String passWord = properties.getProperty("passWord");
    public static final String authorName = properties.getProperty("authorName");

    public static final String primaryId = properties.getProperty("primaryId");

    public static final String moduleName = properties.getProperty("moduleName");
}
```

* 然后，封装对应的模板中定义的变量

```
public class CodeService {

    public void generate(Map<String, Object> templateData) {
        //包前缀
        String packagePreAndModuleName = getPackagePreAndModuleName(templateData);

        //支持对应实体插入在前面，需要带上%s
        templateData.put("entityPackageName", String.format(packagePreAndModuleName + ".entity",
                templateData.get("entityName").toString().toLowerCase()));

        templateData.put("dtoPackageName", String.format(packagePreAndModuleName + ".dto",
                templateData.get("entityName").toString().toLowerCase()));

        templateData.put("voPackageName", String.format(packagePreAndModuleName + ".vo",
                templateData.get("entityName").toString().toLowerCase()));

        templateData.put("daoPackageName", String.format(packagePreAndModuleName + ".dao",
                templateData.get("entityName").toString().toLowerCase()));

        templateData.put("mapperPackageName", packagePreAndModuleName + ".mapper");


        templateData.put("servicePackageName", String.format(packagePreAndModuleName + ".service",
                templateData.get("entityName").toString().toLowerCase()));

        templateData.put("serviceImplPackageName", String.format(packagePreAndModuleName + ".service.impl",
                templateData.get("entityName").toString().toLowerCase()));

        templateData.put("controllerPackageName", String.format(packagePreAndModuleName + ".web",
                templateData.get("entityName").toString().toLowerCase()));

        templateData.put("apiTestPackageName", String.format(packagePreAndModuleName + ".junit",
                templateData.get("entityName").toString().toLowerCase()));


        templateData.put("currentTime", new SimpleDateFormat("yyyy-MM-dd").format(new Date()));

        //======================生成文件配置======================
        try {
            // 生成Entity
            String entityName = String.format("%s", templateData.get("entityName").toString());
            generateFile("entity.ftl", templateData, templateData.get("entityPackageName").toString(), entityName+".java");

            // 生成dto
            String dtoName = String.format("%sDTO", templateData.get("entityName").toString());
            templateData.put("dtoName", dtoName);
            generateFile("dto.ftl", templateData, templateData.get("dtoPackageName").toString(),
                    dtoName + ".java");

            // 生成VO
            String voName = String.format("%sVO", templateData.get("entityName").toString());
            templateData.put("voName", voName);
            generateFile("vo.ftl", templateData, templateData.get("voPackageName").toString(),
                    voName + ".java");

            // 生成DAO
            String daoName = String.format("%sDao", templateData.get("entityName").toString());
            templateData.put("daoName", daoName);
            generateFile("dao.ftl", templateData, templateData.get("daoPackageName").toString(),
                    daoName + ".java");

            // 生成Mapper
            String mapperName = String.format("%sMapper", templateData.get("entityName").toString());
            generateFile("mapper.ftl", templateData, templateData.get("mapperPackageName").toString(),
                    mapperName+".xml");


            // 生成Service
            String serviceName = String.format("%sService", templateData.get("entityName").toString());
            templateData.put("serviceName", serviceName);
            generateFile("service.ftl", templateData, templateData.get("servicePackageName").toString(),
                    serviceName + ".java");

            // 生成ServiceImpl
			String serviceImplName = String.format("%sServiceImpl", templateData.get("entityName").toString());
			templateData.put("serviceImplName", serviceImplName);
			generateFile("serviceImpl.ftl", templateData, templateData.get("serviceImplPackageName").toString(),
                    serviceImplName + ".java");

            // 生成Controller
			String controllerName = String.format("%sController", templateData.get("entityName").toString());
			templateData.put("controllerName", controllerName);
			generateFile("controller.ftl", templateData, templateData.get("controllerPackageName").toString(),
                    controllerName + ".java");

//			// 生成junit测试类
//            String apiTestName = String.format("%sApiTest", templateData.get("entityName").toString());
//            templateData.put("apiTestName", apiTestName);
//            generateFile("test.ftl", templateData, templateData.get("apiTestPackageName").toString(),
//                    apiTestName + ".java");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 生成文件
     * @param templateName 模板名称
     * @param templateData 参数名
     * @param packageName 包名
     * @param fileName 文件名
     */
    public void generateFile(String templateName, Map<String, Object> templateData, String packageName, String fileName) {
        templateData.put("fileName", fileName);

        DaseService dbService = new DaseService(templateData);

        // 获取数据库参数
        if("entity.ftl".equals(templateName) || "mapper.ftl".equals(templateName)){
            dbService.getAllColumns(templateData);
        }
        try {
            // 默认生成文件的路径
            FreeMakerUtil freeMakerUtil = new FreeMakerUtil();
            freeMakerUtil.generateFile(templateName, templateData, packageName, fileName);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 封装包名前缀
     * @return
     */
    private String getPackagePreAndModuleName(Map<String, Object> templateData){
        String packageNamePre = templateData.get("packageNamePre").toString();
        String moduleName = templateData.get("moduleName").toString();
        if(StringUtils.isNotBlank(moduleName)){
            return packageNamePre + "." + moduleName;
        }
        return packageNamePre;
    }

}
```



* 接着，获取模板文件，并生成相应的模板文件

```
public class FreeMakerUtil {


    /**
     * 根据Freemark模板，生成文件
     * @param templateName:模板名
     * @param root：数据原型
     * @throws Exception
     */
    public void generateFile(String templateName, Map<String, Object> root, String packageName, String fileName) throws Exception {
        FileOutputStream fos=null;
        Writer out =null;
        try {
            // 通过一个文件输出流，就可以写到相应的文件中，此处用的是绝对路径
            String entityName = (String) root.get("entityName");
            String fileFullName = String.format(fileName, entityName);
            packageName = String.format(packageName, entityName.toLowerCase());

            String fileStylePackageName = packageName.replaceAll("\\.", "/");
            File file = new File(root.get("outUrl").toString() + "/" + fileStylePackageName + "/" + fileFullName);
            if (!file.getParentFile().exists()) {
                file.getParentFile().mkdirs();
            }
            file.createNewFile();

            Template template = getTemplate(templateName);
            fos = new FileOutputStream(file);
            out = new OutputStreamWriter(fos);
            template.process(root, out);
            out.flush();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (fos != null){
                    fos.close();
                }
                if(out != null){
                    out.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     *
     * 获取模板文件
     *
     * @param name
     * @return
     */
    public Template getTemplate(String name) {
        try {
            Configuration cfg = new Configuration(Configuration.VERSION_2_3_23);
            cfg.setClassForTemplateLoading(this.getClass(), "/ftl");
            Template template = cfg.getTemplate(name);
            return template;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

* 最后，我们编写一个`main`方法，看看运行之后的效果

```
public class GeneratorMain {

    public static void main(String[] args) {
        System.out.println("生成代码start......");

        //获取页面或者配置文件的参数
        Map<String, Object> templateData = new HashMap<String, Object>();
        templateData.put("tableName", SystemConstant.tableName);
        System.out.println("表名=="+ SystemConstant.tableName);

        templateData.put("entityName", SystemConstant.entityName);
        System.out.println("实体类名称=="+ SystemConstant.entityName);

        templateData.put("packageNamePre", SystemConstant.packageNamePre);
        System.out.println("包名前缀=="+ SystemConstant.packageNamePre);

        //支持自定义输出路径
        if(StringUtils.isNotBlank(SystemConstant.outUrl)){
            templateData.put("outUrl", SystemConstant.outUrl);
        } else {
            String path = GeneratorMain.class.getClassLoader().getResource("").getPath() + "../../src/main/java";
            templateData.put("outUrl", path);
        }
        System.out.println("生成文件路径为=="+ templateData.get("outUrl"));

        templateData.put("authorName", SystemConstant.authorName);
        System.out.println("以后代码出问题找=="+ SystemConstant.authorName);


        templateData.put("databaseName", SystemConstant.databaseName);
        templateData.put("ipName", SystemConstant.ipName);
        templateData.put("portName", SystemConstant.portName);
        templateData.put("userName", SystemConstant.userName);
        templateData.put("passWord", SystemConstant.passWord);

        //主键ID
        templateData.put("primaryId", SystemConstant.primaryId);

        //模块名称
        templateData.put("moduleName", SystemConstant.moduleName);
        CodeService dataService = new CodeService();

        try {
            //生成代码文件
            dataService.generate(templateData);
        } catch (Exception e) {
            e.printStackTrace();
        }

        System.out.println("生成代码end......");
    }
}
```

结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-generator/03.jpg)

* 生成的 Controller 层代码如下

```
/**
 *
 * @ClassName: TestEntityController
 * @Description: 外部访问接口
 * @author pzblog
 * @date 2020-11-16
 *
 */
@RestController
@RequestMapping("/testEntity")
public class TestEntityController {

	@Autowired
	private TestEntityService testEntityService;

	/**
	 * 分页列表查询
	 * @param request
	 */
	@PostMapping(value = "/getPage")
	public Pager<TestEntityVO> getPage(@RequestBody TestEntityDTO request){
		return testEntityService.getPage(request);
	}

	/**
	 * 查询详情
	 * @param request
	 */
	@PostMapping(value = "/getDetail")
	public TestEntityVO getDetail(@RequestBody IdRequest request){
		TestEntity source = testEntityService.selectById(request.getId());
		if(Objects.nonNull(source)){
			TestEntityVO result = new TestEntityVO();
			BeanUtils.copyProperties(source, result);
			return result;
		}
		return null;
	}

	/**
	 * 新增操作
	 * @param request
	 */
	@PostMapping(value = "/save")
	public void save(TestEntityDTO request){
		TestEntity entity = new TestEntity();
		BeanUtils.copyProperties(request, entity);
		testEntityService.insert(entity);
	}

	/**
	 * 编辑操作
	 * @param request
	 */
	@PostMapping(value = "/edit")
	public void edit(TestEntityDTO request){
		TestEntity entity = new TestEntity();
		BeanUtils.copyProperties(request, entity);
		testEntityService.updateById(entity);
	}

	/**
	 * 删除操作
	 * @param request
	 */
	@PostMapping(value = "/delete")
	public void delete(IdRequest request){
		testEntityService.deleteById(request.getId());
	}
}
```

至此，一张单表的90%的基础工作量全部开发完毕！

### 三、总结
代码生成器，在实际的项目开发中应用非常的广，本文主要以`freemaker`模板引擎为基础，开发的一套全自动代码生成器，一张单表的CRUD，只需要5秒钟就可以完成！

最后多说一句，如果你是项目负责人，那么代码生成器会是一个比较好的提升项目开发效率的工具，希望能帮到各位！
