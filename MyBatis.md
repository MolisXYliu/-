# MyBatis
## MyBatis简介
1. MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架
2. MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集
3. MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录
4. Mybatis 是一个 半自动的ORM（Object   Relation  Mapping）框架

## MyBatis的搭建过程
1. 导入jar包
2. 创建mybatis的核心配置文件**mybatis-config.xml**并配置
3. 创建映射文件**xxxMapper.xml**，并配置
4. 创建mapper接口 实现两个绑定
   - 接口权限命名要和映射文件的namesqace保持一致
   - 接口中方法名和sql语句的id保持一致
5. 获取mybatis操作数据库的会话对象Sqlsession 通过getMapper方法获取接口的动态代理实现类
6. 测试


### mybbatie配置文件(xml)
标签顺序
**"(properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,reflectorFactory?,plugins?,environments?,databaseIdProvider?,mappers?)".**

````xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--
    "(properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,reflectorFactory?,plugins?,environments?,databaseIdProvider?,mappers?)".
-->

    <!--<properties>
        <property name="jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
    </properties>-->

    <!--
    <properties>用来设置或引入资源文件
    resource 在类路径下访问资源文件
    url 在网络路径或磁盘路径下访问资源文件
    -->
    <properties resource="jdbc.properties"></properties>
    
    
  <!--  <settings>
        <setting name="" value=""/>
    </settings>-->

   <!-- <typeAliases>
        &lt;!&ndash;
        为类型设置类型别名
        type:java 类型 若只设置type 默认的别名为类名 且不区分大小写
        &ndash;&gt;
&lt;!&ndash;        <typeAlias type="com.lxy.bean.User" alias="u"/>&ndash;&gt;
        &lt;!&ndash;包下所有的都会自动设置别名&ndash;&gt;
        <package name="com.lxy.bean"/>
    </typeAliases>-->
    <!--
    environments 设置连接数据库的环境
    default：设置默认使用的数据库的环境
    -->
    <environments default="mysql">
        <!--设置某个具体的数据库环境
            id：数据库环境的唯一标识
        -->
        <environment id="mysql">
            <!--type=JDBC|MANAGED-->
            <transactionManager type="JDBC"/>
            <!--type=POOLED|UNPOOLED|JNDI-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>

        <environment id="oracle">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/ssm?useSSL=false&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <!--引入映射文件-->
    <mappers>
        <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>
````

### 映射文件
````xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lxy.mapper.UserMapper">
    <!--
        <select>定义查询语句
        id 设置sql语句的唯一标识 不能重复
        resultype 结果类型，即实体类的权限命名
    -->
    <select id="getUserByUid" resultType="com.lxy.bean.User">
        select uid,user_name username,password,age,sex from user where uid = #{id}
    </select>
</mapper>
````

### mybatis的使用
````java
 InputStream is = Resources.getResourceAsStream("mybatis-config.xml");

        SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(is);

        SqlSession sqlSession = sqlSessionFactory.openSession();
        //getMapper()核心方法 会通过动态代理动态生成代理实现类
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        System.out.println(mapper.getClass().getName());

        User user = mapper.getUserByUid("1");

        System.out.println(user);
````

### mybatis获取参数值的两种方式
1. ${} insert into emp values(null,admin,23,男) statement对象 **必须使用字符串拼接的方式操作sql** 一定要注意单引号问题
2. #{} insert into emp values(null,?,?,?) PreparedStatement **可以使用通配符操作sql可以防止sql注入** 
3. 使用建议 **建议使用#{} 在特殊情况下需要使用${} 例如模糊查询和批量删除

#### 不同的参数类型，${}和#{}的不同取值方式
1. 当传输参数为单个String或基本数据类型和其包装类
   - #{}可以以任意的名字获取参数值
   - \${}只能以${value}或\${_paramter}获取
2. 当传输参数为javaBean时
   - \${}和#{}都可以通过属性名直接获取属性值，但是要注意\${}的''问题
3. 当传输多个参数时：mybatis会默认将这些参数放在map集合中两种方式：
   1. 键为0，1，2，3。。。n-1 以参数位置
   2. 键为param1,param2,param3...param n 以参数为值
   - #{0}.#{1};#{param1}.#{param2}
   - \${param1}.\${param2}
4. 当传输Map参数时：
   - ${}和#{}都可以通过键名直接获取属性值，但是要注意\${}的''问题
5. 命名参数：可以通过@Param("key")为map集合指定键的名字
````java
//根据eid和ename获取员工信息
    Emp getEmpByEidAndEnameByParam(@Param("eid") String eid,@Param("ename") String ename);
````
6. 当传输参数为List或Array，mybatis会将List或Array放在map中，List以list为键，Array以Array为键

### 主键
获取主键值
1. 若数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），则可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置到目标属性上。
````java
<insert id="insertEmployee" 	parameterType="com.atguigu.mybatis.beans.Employee"  
			databaseId="mysql"
			useGeneratedKeys="true"
			keyProperty="id">
		insert into tbl_employee(last_name,email,gender) values(#{lastName},#{email},#{gender})
</insert>
````
2. 而对于不支持自增型主键的数据库（例如 Oracle），则可以使用 selectKey 子元素：selectKey  元素将会首先运行，id  会被设置，然后插入语句会被调用
````java
<insert id="insertEmployee" 
		parameterType="com.atguigu.mybatis.beans.Employee"  
			databaseId="oracle">
		<selectKey order="BEFORE" keyProperty="id" 
                                       resultType="integer">
			select employee_seq.nextval from dual 
		</selectKey>	
		insert into orcl_employee(id,last_name,email,gender) values(#{id},#{lastName},#{email},#{gender})
</insert>
````

###  resultType自动映射 
1.	autoMappingBehavior默认是PARTIAL，开启自动映射的功能。唯一的要求是结果集列名和javaBean属性名一致
2.	如果autoMappingBehavior设置为null则会取消自动映射
3.	数据库字段命名规范，POJO属性符合驼峰命名法，如A_COLUMNaColumn，我们可以开启自动驼峰命名规则映射功能，mapUnderscoreToCamelCase=true

### resultMap自定义映射
1.	自定义resultMap，实现高级结果集映射
2.	id ：用于完成主键值的映射
3.	result ：用于完成普通列的映射
4.	association ：一个复杂的类型关联;许多结果将包成这种类型
5.	collection ： 复杂类型的集
````java
 <resultMap id="deptMapStep" type="com.lxy.bean.Dept">
        <id column="did" property="did"></id>
        <result column="dname" property="dname"></result>
        <collection property="emps" select="com.lxy.mapper.EmpDeptMapper.getEmpListByDid" column="did" fetchType="eager"></collection>
    </resultMap>
````

##  association
POJO中的属性可能会是一个对象,我们可以使用联合查询，并以级联属性的方式封装对象.使用association标签定义对象的封装规则
````java
 <resultMap id="empMap" type="com.lxy.bean.Emp">
        <id column="eid" property="eid"/>
        <result column="ename" property="ename"/>
        <result column="age" property="age"/>
        <result column="sex" property="sex"/>
        <association property="dept" javaType="com.lxy.bean.Dept">
            <id column="did" property="did"/>
            <result column="dname" property="dname"/>
        </association>
    </resultMap>
````
#### association 分步查询
1.	先通过员工的id查询员工信息
2.	再通过查询出来的员工信息中的外键(部门id)查询对应的部门信息. 
````xml
<resultMap id="empMapStep" type="com.lxy.bean.Emp">
        <id column="eid" property="eid"/>
        <result column="ename" property="ename"/>
        <result column="age" property="age"/>
        <result column="sex" property="sex"/>
        <!--
        select:分步查询的sql的id，即接口的权限命名.方法名或namespace的值.sql的id
        column：分步查询的条件，注意：此条件必须是从数据库查询过的
        -->
        <association property="dept" select="com.lxy.mapper.DeptMapper.getDeptById" column="did"></association>
  
   <!-- Dept getDeptById(String did);-->
    <select id="getDeptById" resultType="com.lxy.bean.Dept">
        select did,dname from dept where did=#{did}
    </select>

    </resultMap>
    <!--Emp getEmpStep(String eid);-->
    <select id="getEmpStep" resultMap="empMapStep">
        select eid,ename,age,sex,did from emp where eid=#{eid}
    </select>
````

#### association 分步查询使用延迟加载
1. 在分步查询的基础上，可以使用延迟加载来提升查询的效率，只需要在全局的
Settings中进行如下的配置:
````xml
<!-- 开启延迟加载 -->
<setting name="lazyLoadingEnabled" value="true"/>
<!-- 设置加载的数据是按需还是全部 -->
<setting name="aggressiveLazyLoading" value="false"/>
````

#### collection
POJO中的属性可能会是一个集合对象,我们可以使用联合查询，并以级联属性的方式封装对象.使用collection标签定义对象的封装规则
````xml
 <resultMap id="deptMap" type="com.lxy.bean.Dept">
        <id column="did" property="did"></id>
        <result column="dname" property="dname"></result>
        <!--collection 处理一对多和多对多的关系
            ofType 指集合中的类型，不需要指定javaType
            -->
        <collection property="emps" ofType="com.lxy.bean.Emp">
        <id column="eid" property="eid"></id>
        <result column="ename" property="ename"></result>
        <result column="age" property="age"></result>
        <result column="sex" property="sex"></result>
        </collection>
    </resultMap>
````

#### 扩展: association 或 collection的 fetchType属性 
1.	在<association> 和<collection>标签中都可以设置fetchType，指定本次查询是否要使用延迟加载。默认为 fetchType=”lazy” ,如果本次的查询不想使用延迟加载，则可设置为
fetchType=”eager”.
2.	fetchType可以灵活的设置查询是否需要使用延迟加载，而不需要因为某个查询不想使用延迟加载将全局的延迟加载设置关闭.

# MyBatis 动态SQL(重要)
### 简介
1.	动态 SQL是MyBatis强大特性之一。极大的简化我们拼装SQL的操作
2.	动态 SQL 元素和使用 JSTL 或其他类似基于 XML 的文本处理器相似
3.	MyBatis 采用功能强大的基于 OGNL 的表达式来简化操作
    - if
    -  choose (when, otherwise)
    - trim (where, set)
    - foreach
4.	**OGNL**（ Object Graph Navigation Language ）对象图导航语言，这是一种强大的表达式语言，通过它可以非常方便的来操作对象属性。 类似于我们的EL，SpEL等
    - 访问对象属性：		person.name
    - 调用方法：		    person.getName()
    - 调用静态属性/方法：	@java.lang.Math@PI	@java.util.UUID@randomUUID()调用构造方法：		new com.atguigu.bean.Person(‘admin’).name
    - 运算符：		     +,-*,/,%
    - 逻辑运算符：		 in,not in,>,>=,<,<=,==,!=

**注意：xml中特殊符号如”,>,<等这些都需要使用转义字符**

## if where
1.	If用于完成简单的判断.
2.	Where用于解决SQL语句中where关键字以及条件中第一个and或者or的问题 
````xml
	<select id="getEmpsByConditionIf" resultType="com.atguigu.mybatis.beans.Employee">
		select id , last_name ,email  , gender  
		from tbl_employee 
		<where>
			<if test="id!=null">
				and id = #{id}
			</if>
			<if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
				and last_name = #{lastName}
			</if>
			<if test="email!=null and email.trim()!=''">
				and email = #{email}
			</if>
			<if test="&quot;m&quot;.equals(gender) or &quot;f&quot;.equals(gender)">
				and gender = #{gender}
			</if>
		</where>
</select>
````
##  trim
1. Trim 可以在条件判断完的SQL语句前后添加或者去掉指定的字符
   - prefix: 添加前缀
   - prefixOverrides: 去掉前缀
   - suffix: 添加后缀
   - suffixOverrides: 去掉后缀
```xml
<select id="getEmpsByConditionTrim" resultType="com.atguigu.mybatis.beans.Employee">
		select id , last_name ,email  , gender  
		from tbl_employee 
		<trim prefix="where"  suffixOverrides="and">
			<if test="id!=null">
				 id = #{id} and
			</if>
			<if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
				 last_name = #{lastName} and
			</if>
			<if test="email!=null and email.trim()!=''">
				 email = #{email} and
			</if>
			<if test="&quot;m&quot;.equals(gender) or &quot;f&quot;.equals(gender)">
				gender = #{gender}
			</if>
		</trim>
</select>
````
## set
1.	set主要是用于解决修改操作中SQL语句中可能多出逗号的问题

## choose(when、otherwise)
**choose 主要是用于分支判断，类似于java中的switch case,只会满足所有分支中的一个**
````xml
<select id="getEmpsByConditionChoose" resultType="com.atguigu.mybatis.beans.Employee">
		select id ,last_name, email,gender from tbl_employee
		<where>
			<choose>
				<when test="id!=null">
					id = #{id}
				</when>
				<when test="lastName!=null">
					last_name = #{lastName}
				</when>
				<when test="email!=null">
					email = #{email}
				</when>
				<otherwise>
					 gender = 'm'
				</otherwise>
			</choose>
		</where>
</select>
````

## 批量操作
- delete：
   - delete from emp where eid in ();
   - delete from emp where eid=1 or eid=2 or eid=3
- select:
   - select from emp where eid in ();
   - select from emp where eid=1 or eid=2 or eid=3
- update:
  - 把每条数据修改成形同的内容
    - update emp set ... where eid in ();
    - update emp set ... where eid=1 or eid=2 or eid=3
  - 把每条数据修改成不同的内容(要修改**allowMultiQueries=true**)
    - update emp set ... where eid=1;
    - update emp set ... where eid=2;
    - update emp set ... where eid=3;
- insert:
  - insert into emp values(),(),()
## foreach
foreach 主要用于循环迭代
    - collection: 要迭代的集合
    - item: 当前从集合中迭代出的元素
    - open: 开始字符
    - close:结束字符
    - separator: 元素与元素之间的分隔符
    - index:
	    - 迭代的是List集合: index表示的当前元素的下标
		- 迭代的Map集合:  index表示的当前元素的key
 ````xml
 <select id="getEmpsByConditionForeach" resultType="com.atguigu.mybatis.beans.Employee">
		 select id , last_name, email ,gender from tbl_employee where  id in 
		 <foreach collection="ids" item="curr_id" open="(" close=")" separator="," >
		 		#{curr_id}
		 </foreach>
</select>
````

## sql
1.	sql 标签是用于抽取可重用的sql片段，将相同的，使用频繁的SQL片段抽取出来，单独定义，方便多次引用.
2.	抽取SQL: 
````xml
<sql id="selectSQL">
		select id , last_name, email ,gender from tbl_employee
</sql>
````
3. 引用SQL:
``<include refid="selectSQL"></include>``

# MyBatis 缓存机制
## 缓存机制简介
1.	MyBatis 包含一个非常强大的查询缓存特性,它可以非常方便地配置和定制。缓存可以极大的提升查询效率
2.	MyBatis系统中默认定义了两级缓存
一级缓存
二级缓存
3.	默认情况下，只有一级缓存（SqlSession级别的缓存，也称为本地缓存）开启。
4.	二级缓存需要手动开启和配置，他是基于namespace级别的缓存。
5.	为了提高扩展性。MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存

## 一级缓存的使用
1.	一级缓存(local cache), 即本地缓存, 作用域默认为sqlSession。当  Session flush 或 close 后, 该 Session 中的所有 Cache 将被清空。
2.	本地缓存不能被关闭, 但可以调用 clearCache() 来清空本地缓存, 或者改变缓存的作用域.
3.	在mybatis3.1之后, 可以配置本地缓存的作用域. 在 mybatis.xml 中配置
4.	一级缓存的工作机制
同一次会话期间只要查询过的数据都会保存在当前SqlSession的一个Map中
key: hashCode+查询的SqlId+编写的sql查询语句+参数


**mybatis中的一级缓存默认开启，是sqlSession级别的即同一个sqlSession对于一个sql语句执行之后就会存储到缓存中，下次执行相同的sql，直接从缓存中取**
## 一级缓存失效的几种情况
1.	不同的SqlSession对应不同的一级缓存
2.	同一个SqlSession但是查询条件不同
3.	同一个SqlSession两次查询期间执行了任何一次增删改操作（不管是否成功）
4.	同一个SqlSession两次查询期间手动清空了缓存

## 二级缓存的使用
1.	二级缓存(second level cache)，全局作用域缓存
2.	二级缓存默认不开启，需要手动配置
3.	MyBatis提供二级缓存的接口以及实现，缓存实现要求POJO实现Serializable接口
4.	二级缓存在 SqlSession 关闭或提交之后才会生效
5.	**二级缓存使用的步骤**:
    - 全局配置文件中开启二级缓存``<setting name="cacheEnabled" value="true"/>``
    - 需要使用二级缓存的映射文件处使用cache配置缓存``<cache />``
    - 注意：POJO需要实现Serializable接口
6.	二级缓存相关的属性
    1. eviction=“FIFO”：缓存回收策略：
    2. LRU – 最近最少使用的：移除最长时间不被使用的对象。
    3. FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
    4. SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
    5. WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
    6. **默认的是 LRU。**
    - flushInterval：刷新间隔，单位毫秒默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新
    - size：引用数目，正整数代表缓存最多可以存储多少个对象，太大容易导致内存溢出
    - readOnly：只读，true/false
      - true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。
      - false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是 false。

## 缓存的相关属性设置
1.	全局setting的cacheEnable：
配置二级缓存的开关，一级缓存一直是打开的。
2.	select标签的useCache属性：
配置这个select是否使用二级缓存。一级缓存一直是使用的
3.	sql标签的flushCache属性：
增删改默认flushCache=true。sql执行以后，会同时清空一级和二级缓存。
查询默认 flushCache=false。
4.	sqlSession.clearCache()：只是用来清除一级缓存。

## 整合第三方缓存 
1.	为了提高扩展性。MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存
2.	EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认的CacheProvider
3. 整合EhCache缓存的步骤:
    - 导入ehcache包，以及整合包，日志包**ehcache-core-2.6.8.jar、mybatis-ehcache-1.0.3.jar
    slf4j-api-1.6.1.jar、slf4j-log4j12-1.6.2.jar**
    - 编写ehcache.xml配置文件
````xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
 <!-- 磁盘保存路径 -->
 <diskStore path="D:\atguigu\ehcache" />
 
 <defaultCache 
   maxElementsInMemory="1000" 
   maxElementsOnDisk="10000000"
   eternal="false" 
   overflowToDisk="true" 
   timeToIdleSeconds="120"
   timeToLiveSeconds="120" 
   diskExpiryThreadIntervalSeconds="120"
   memoryStoreEvictionPolicy="LRU">
 </defaultCache>
</ehcache>
<!-- 
属性说明：
l diskStore：指定数据在磁盘中的存储位置。
l defaultCache：当借助CacheManager.add("demoCache")创建Cache时，EhCache便会采用<defalutCache/>指定的的管理策略
 
以下属性是必须的：
l maxElementsInMemory - 在内存中缓存的element的最大数目 
l maxElementsOnDisk - 在磁盘上缓存的element的最大数目，若是0表示无穷大
l eternal - 设定缓存的elements是否永远不过期。如果为true，则缓存的数据始终有效，如果为false那么还要根据timeToIdleSeconds，timeToLiveSeconds判断
l overflowToDisk - 设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上
 
以下属性是可选的：
l timeToIdleSeconds - 当缓存在EhCache中的数据前后两次访问的时间超过timeToIdleSeconds的属性取值时，这些数据便会删除，默认值是0,也就是可闲置时间无穷大
l timeToLiveSeconds - 缓存element的有效生命期，默认是0.,也就是element存活时间无穷大
 diskSpoolBufferSizeMB 这个参数设置DiskStore(磁盘缓存)的缓存区大小.默认是30MB.每个Cache都应该有自己的一个缓冲区.
l diskPersistent - 在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false。
l diskExpiryThreadIntervalSeconds - 磁盘缓存的清理线程运行间隔，默认是120秒。每个120s，相应的线程会进行一次EhCache中数据的清理工作
l memoryStoreEvictionPolicy - 当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出）
 -->
````
  - 配置cache标签 ``  <cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>``


# MyBatis 逆向工程
## 逆向工程简介
MyBatis Generator: 简称MBG，是一个专门为MyBatis框架使用者定制的代码生成器，可以快速的根据表生成对应的映射文件，接口，以及bean类。支持基本的增删改查，以及QBC风格的条件查询。但是表连接、存储过程等这些复杂sql的定义需要我们手工编写
官方文档地址
http://www.mybatis.org/generator/
官方工程地址
https://github.com/mybatis/generator/releases


## 逆向工程的配置
1.	导入逆向工程的jar包
mybatis-generator-core-1.3.2.jar
2.	编写MBG的配置文件（重要几处配置）,可参考官方手册
````xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <context id="DB2Tables" targetRuntime="MyBatis3Simple">
    <!--设置连接数据库的信息-->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/ssm?useSSL=false&amp;serverTimezone=UTC&amp;allowPublicKeyRetrieval=true&amp;allowMultiQueries=true"
                        userId="root"
                        password="123456">
        </jdbcConnection>

        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!--javaBean的生成策略-->
        <javaModelGenerator targetPackage="com.lxy.bean" targetProject=".\src">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!--映射文件生成策略-->
        <sqlMapGenerator targetPackage="com.lxy.mapper"  targetProject="./conf">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!--mapper接口的而生成策略-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.lxy.mapper"  targetProject=".\src">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!--设置要将数据库中的哪张表逆向生成一个javaBean-->
        <table tableName="emp" domainObjectName="Emp" ></table>
        <table tableName="dept" domainObjectName="Dept" ></table>

    </context>
</generatorConfiguration>
````
3. 运行代码
````java
 public void testMBG() throws Exception{
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        File configFile = new File("mbg.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }
````
# PageHelper分页插件

## 使用步骤
1.	导入相关包pagehelper-x.x.x.jar 和 jsqlparser-0.9.5.jar
2.	在MyBatis全局配置文件中配置分页插件
````xml
<plugins>
	<plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
</plugins>
````
3.	使用PageHelper提供的方法进行分页
4.	可以使用更强大的PageInfo封装返回结果

# 分页代码
````java
package com.lxy.ssm.util;

import com.github.pagehelper.PageInfo;
import com.lxy.ssm.bean.Emp;

import javax.servlet.http.HttpServletRequest;


/**
 * 首页 上一页 1 2 3 4 5 上一页 末页
 */
public class PageUtil {
    public static String getPageInfo(PageInfo<Emp> pageInfo, HttpServletRequest request){

        String path = request.getContextPath()+"/";

        StringBuilder builder=new StringBuilder();

        //拼接首页
        builder.append("<a href='"+path+"emps/1'>首页</a>");

        builder.append("&nbsp;&nbsp;");
        //拼接上一页码
        if(pageInfo.isHasPreviousPage()){
            builder.append("<a href='"+path+"emps/"+pageInfo.getPrePage()+"'>上一页</a>");
            builder.append("&nbsp;&nbsp;");

        }else {
            builder.append("上一页");
            builder.append("&nbsp;&nbsp;");

        }
        //拼接页码
        int[] nums = pageInfo.getNavigatepageNums();
        for (int i : nums) {
            if(i==pageInfo.getPageNum()){
            builder.append("<a style='color:red;' href='"+path+"emps/"+i+"'>"+i+"</a>");
            builder.append("&nbsp;&nbsp;");

            }else {
                builder.append("<a href='" + path + "emps/" + i + "'>" + i + "</a>");
                builder.append("&nbsp;&nbsp;");

            }
        }


        //拼接下一页
        if (pageInfo.isHasNextPage()){
            builder.append("<a href='"+path+"emps/"+pageInfo.getNextPage()+"'>下一页</a>");
            builder.append("&nbsp;&nbsp;");

        }else{
            builder.append("下一页");
            builder.append("&nbsp;&nbsp;");

        }

        //拼接尾页
        builder.append("<a href='"+path+"emps/"+pageInfo.getPages()+"'>尾页</a>");

        return builder.toString();

    }
}
````