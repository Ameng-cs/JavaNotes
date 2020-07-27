- ## 包
    - mysql-connector-java-8.0.11.jar
    - mybatis-3.5.4.jar
    - log4j-1.2.17.jar((该包是日志包,可以记录mybatis的一些关键日志输出,他依赖log4j.xml文件))
- ## 全局配置文件
- ### 指导mybatis正确运行的一些全局设置,比如连接哪个数据库
- ### properties 属性

```xml
resource:从类路径下开始引用
url:引用磁盘路径或者是网络路径的资源
<properties resource="DBconfig.properties"></properties>
<properties url=""></properties>
之后可以用${}来取出配置文件中的值
<dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
</dataSource>

```
- ### settings 设置
    - MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为

```xml
<settings>
name:配置项的key;value:配置项的值
    <setting name="" value=""/>
</settings>
```
- ### typeAliases 类型命名
    - 该标签可以简化java中类名的限定名,给一个类起一个别名,之后再的时候用别名就行了
    ==但是推荐就用全类名==

```xml
<!--单个取别名-->
<typeAliases>
  <!--别名默认是类名,也可以用alias来指定-->
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
</typeAliases>
<!--批量取别名-->
<typeAliases>
  <!--为该包下的所有bean取别名,默认为类名-->
  <package name="domain.blog"/>
</typeAliases>
若在批量取名后还想为单个bean取别名,则可以在那个类的前面加上@Alias("别名")该注释
```
很多java的基础类型mybatis以及为他们取好了别名,我们就不用在取了
_byte 	byte
_long 	long
_short 	short
_int 	int
_integer 	int
_double 	double
_float 	float
_boolean 	boolean 

- ### typeHandlers 类型处理器(了解)
    - 无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型
- ### objectFactory 对象工厂(了解)
    - MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。 如果想覆盖对象工厂的默认行为，则可以通过创建自己的对象工厂来实现
    - ObjectFactory 接口很简单，它包含两个创建用的方法，一个是处理默认构造方法的，另外一个是处理带参数的构造方法的。 最后，setProperties 方法可以被用来配置 ObjectFactory，在初始化你的 ObjectFactory 实例后， objectFactory 元素体中定义的属性会被传递给 setProperties 方法
- ### plugins 插件
    - 插件是Mybatis提供的一个非常强大的机制,我们可以通过插件来修改Mybatis的一些核心行为.插件通过动态代理机制,可以介入四大对象的任何一个方法的执行.
- ### environments 环境
    - #### environment 环境变量
        - ##### transactionManager 事务管理器
            - **type="JDBC":** 这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域。
            - **type="MANAGED":** 这个配置几乎没做什么。  
        - ##### dataSource 数据源
            - UNPOOLED
            - POOLED
            - JNDI

```xml
default="":表示默认使用环境
environments下可以有很多的environment,通过切换environment可以改变连接的数据库
<environments default="development">
    id用来唯一标识一个环境
    <environment id="development">
        事务管理器:可以通过Spring来完成
        <transactionManager type="JDBC"/>
        Srping使用C3P0的更加强大
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/test?useSSL=false"/>
            <property name="username" value="root"/>
            <property name="password" value="168168"/>
        </dataSource>
    </environment>
</environments>
```

- ### databaseIdProvider 数据库厂商标识
    - Mybatis用来考虑数据库的移植性的

```xml
<databaseIdProvider type="DB_VENDOR">
    name:数据库厂商的表示
    value:给这个标识起的别名
    <property name="SQL Server" value="sqlserver"/>
    <property name="MySQL" value="mysql"/>
    <property name="Oracle" value="oracle" />
</databaseIdProvider>

像这样写很多不同数据库的sql语句,在其后面的databaseId写上对应的数据库,
这样Mybatis就会在不同的数据库环境下执行对应的sql语句
<select id="getEmpById" resultType="com.mybatis.bean.Emp" databaseId="mysql">
    select * from emp where id = #{id}
</select>

<select id="getEmpById" resultType="com.mybatis.bean.Emp" databaseId="oracle">
select * from emp where id = #{id}
</select>
```

- ### mappers 映射器
    - 写好的sql映射文件需要使用mappers注册进来

```xml
<mappers>
    resource:在类路径下找到sql映射文件
    <mapper resource:在类路径下找到sql映射文件="org/mybatis/builder/AuthorMapper.xml"/>
    url:从磁盘或是网络路径引用
    <mapper url="file:///var/mappers/AuthorMapper.xml"/>
    class:直接引用接口的全类名,需要将xml文件放在和dao接口同目录下,且文件名和接口名一直
    若是注解版的dao,也可以使用class这个属性,因为没得xml文件了
    <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
批量注册:把包下的xml 都注册\
name:要批量注册的包名
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>

```
- ### 整体
```xml
<?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
                PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
                "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/>
        <!-- 配置连接池-->
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/test"/>
            <property name="username" value="root"/>
            <property name="password" value="168168"/>
        </dataSource>
    </environment>
</environments>
<!--引入我们自己编写的每一个接口的实现文件-->
<mappers>
    <!--表示从类路径下查找-->
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
</mappers>
</configuration>
```
- ## SQL映射文件:相当于对Dao接口的一个实现描述
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--名称空间:写接口的全类名,相当于告诉mybatis这个配置文件是实现哪个接口的-->
<mapper namespace="com.mybatis.dao.EmpDao">
    <!--select:定义一个查询操作
        id:方法名,相当于这个配置是对于某个方法的实现
        resultType:指定方法运行后的返回值类型(查询操作必须指定)
        #{属性名}:代表取该方法的名为id的参数值-->
    <select id="getEmpById" resultType="com.mybatis.bean.Emp">
    select * from emp where id = #{id}
    </select>
</mapper>
```
- ### delete,update,insert
    - id:命名空间的唯一标识,用来绑定方法
    - timeot:设置方法的超时时间
    - statmentType:默认为PREPARED
    - databaseId:指定CRUD属于的数据库  
    - useGeneratedKeys,keyProperties
        - useGeneratedKeys="true":使用自动生成的主键
        - keyProperties="属性名"指定将刚刚自增的id封装给哪个属性
- ### select
    - #### 参数   
        - 传单个参数
            - 取值:#{随便写}  
        - 传多个参数
            - 取值:#{参数索引(0,1,2...)}或者#{param1,param2...}
            - 原因:只要传入了多个参数,Mybatis会自动将这些参数封装在一个map中,封装时使用的key就是参数的索引和param1
            - 若想使用参数名来获取该参数则要==在参数前添加@Parma("指定参数名")注释==来指定获取参数时写的值 
        - 传入map
            - 取值:#{key/属性名}
        - 传入POJO
            - 取值:#{POJO的属性名}
    - #### 返回值
        - 查询返回list :
            - resultType如果返回的是集合写的是集合里面元素的类型
        - 查询返回map:
            - resultType="map"
            - 若是单个记录,Mybatis会是列名作为key,数据作为value
            - 若是多条记录,则默认key是这条记录的主键,value则是这条记录封装的对象
            - 若要指定一列作为map的key的值则可以在该方法前加上@MapKey("列名")注释
            - 若是查询多个记录的情况下,resultType就要写成map里面元素的类型
- ### #{key}取值的时候可以设置的规则
    - id=#{id,jdbcType=INTEGER} 
    - javaType、jdbcType、mode、numericScale、
   resultMap、typeHandler、jdbcTypeName
    - 只有jdbcType才可能是需要被指定的
    - 如果传入的数据时null,oracle识别不了null
- ### #{}和${}的区别
    - #{属性名}:是参数预编译的方式,参数的位置都是用?替代,参数后来都是预编译设置进去的(安全,不会有sql注入风险)
    - ${属性名}:不是参数预编译,而是直接和sql语句进行拼串(不安全,有风险)
    - ==SQL语句只有参数位置是支持预编译的,在不支持预编译的地方使用${}就可以动态的获取例如表名这些不支持预编译的值==
- ### 自定义结果集
    - Mybatis默认自动封装结果集
    - 按照列名和属性名一一对应的规则(不区分大小写)
    - 若果不一一对应
        - 开启驼峰命名法(满足驼峰命名规则)
        - 起别名
        - 自定义结果集:自己定义每一列数据和javabean的映射规则
        - 将resultType改成resultMap
        
```xml
type:指定为那个javabean自定义封装规则
id:唯一标识
<resultMap type="全类名" id="id">
    指定主键列的对应规则
    column:指定表的哪一列是主键列
    peoperty:指定类的哪一个属性封装了主键列数据
    <id property="id" column="id"/>
    指定普通列的对应规则
    <result property="name" column="cname">
</resultMap>

```
- ### 联合查询
    - 当某bean的属性中存在其他bean的时候就要进行级联查询,这就要用到自定义封装规则
- #### 使用级联属性封装联合查询后的所有结果 

```xml
    <select id="getEmpById" resultMap="emp">
        select e.id eid,e.name ename,e.deptno deptno d.name dname from emp e
            left join dept d on e.deptno= d.deptno
            where e.id=#{id}
    </select>

    <resultMap id="emp" type="com.mybatis.bean.Emp">
        <id property="id" column="id"></id>
        <result property="ename" column="ename"></result>
        <result property="department.deptno" column="deptno"></result>
        <result property="department.name" column="dname"></result>
    </resultMap>

```
- #### 使用association定义联合查询的对象的封装规则
    - association表示联合了一个对象

```xml
<select id="getEmpById" resultMap="emp">
    select e.id eid,e.name ename,e.deptno deptno d.name dname from emp e
        left join dept d on e.deptno= d.deptno
        where e.id=#{id}
</select>
<resultMap id="emp" type="com.mybatis.bean.Emp">
    <id property="id" column="id"></id>
    <result property="ename" column="ename"></result>
    <association property="department" javaType="com.mybatis.bean.Department">
        定义department属性对应的这个Department对象如何封装
        <id property="deptno" column="deptno"></id>
        <result property="dname" column="dname"></result>
    </association>
</resultMap> 
```
- #### collection定义集合类型的封装规则
    - 查询的bean的属性中存在list
    - collection:定义集合元素的封装

```xml
    <resultMap id="emp" type="com.mybatis.bean.Emp">
        <id property="id" column="id"></id>
        <result property="ename" column="ename"></result>
        property:指定哪个属性是集合属性
        ofType:指定集合里面元素的类型
        <collection property="mgr" ofType="com.mybatis.bean.Emp">
            标签体中指定集合中这个元素的封装规则
            <id property="id" column="id"></id>
            <result property="ename" column="ename"></result>
        </collection>
</resultMap>

```
- #### 使用select属性指定分步查询
    - 这样需要执行多个sql语句,会加重数据库的负担

```xml
<resultMap id="emp" type="com.mybatis.bean.Emp">
    <id property="id" column="id"></id>
    <result property="ename" column="ename"></result>
    告诉mybatis自己去调用一个查询去查询该员工的上级们
    select:指定一个查询sql的唯一标识,Mybatis会自动调用指定的sql将查出来的结果集给对应的属性
    column:指定将哪一列的数据传递给该select供查找
    fetchType="lazy":开启延迟加载,懒加载
    <association property="Mgr" select="包名.前面声明过的select标签的id" column="要传递给select的的列名"
    fetchType="lazy">
        <id property="deptno" column="deptno"></id>
        <result property="dname" column="dname"></result>
    </association>
</resultMap> 
```
collection的分步查询:

```xml
    <resultMap id="emp" type="com.mybatis.bean.Emp">
        <id property="id" column="id"></id>
        <result property="ename" column="ename"></result>
        select:指定一个查询sql的唯一标识,Mybatis会自动调用指定的sql将查出来的结果集给对应的属性
        column:指定将哪一列的数据传递给该select供查找
        fetchType="lazy":开启延迟加载,懒加载
        <collection property="mgr" select="包名.前面声明过的select标签的id" column="要传递给select的的列名">
            标签体中指定集合中这个元素的封装规则
            <id property="id" column="id"></id>
            <result property="ename" column="ename"></result>
        </collection>
</resultMap>
```

- 为了减轻分步查询对数据库的负担,可以==开启按需加载策略==
    - 在setting标签里配置延迟加载

```xml
    <settings>
        开启延迟加载功能
        <setting name="lazyLoadingEnabled" value="true"/>
        开启属性按需加载功能
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
```
- ### 动态sql
- #### if标签:判断 ****

```xml
效果:如果满足if标签中的test属性的条件,就为sql语句拼上if标签中你编辑的内容.
这样仍有句尾and这个词无法处理的问题
    <select id="getTeacherByCondition"  resultMap="teacherMap">
        select * from t_teacher where 
        <!--test="判断条件"-->
        <!--id!=null:取出传入的javabean属性中的id值,判断是否为空
        -->
        <if test="id!=null">
            id>#{id} and
        </if>
        <!--连接的逻辑运算符:
            &&:and
            ||:or
            -->
        <if test="name!=null and name!=''">
            teacherName like #{name} and
        </if>
        <!--条件中还支持java的方法-->
        <if test="birth!=null and !birth.equals("")">
            birth_date>#{birth}
        </if>
    </select>
```
- #### where标签 **** 

```xml
效果帮我们处理每个if标签中句子前面的and,也就是说当有某个条件达不到时会出现句子前面多出and的问题,where会帮你解决这个问题
    <select id="getTeacherByCondition"  resultMap="teacherMap">
        select * from t_teacher 
        <where>
            <if test="id!=null">
                id>#{id}
            </if>
            <if test="name!=null and name!=''">
                and teacherName like #{name} 
            </if>
            <!--条件中还支持java的方法:OGNL表达式-->
            <if test="birth!=null and !birth.equals("")">
                and birth_date>#{birth}
            </if>
        </where>
    </select>
```
- #### trim标签

```xml
截取字符串
    prefix:为整体添加一个前缀
    prefixOverrides:去除if里前面多余的字符串
    suffix:为整体添加一个后缀
    suffixOverrides:去除if里后面多余的字符串
<select id="getTeacherByCondition"  resultMap="teacherMap">
    select * from t_teacher  
    <trim prefix="where" suffixOverrides="and"    >
        <if test="id!=null">
            id>#{id} and
        </if>
        <if test="name!=null and name!=''">
            teacherName like #{name} and
        </if>
        <!--条件中还支持java的方法:OGNL表达式-->
        <if test="birth!=null and !birth.equals("")">
            birth_date>#{birth} and 
        </if>
    </trim>
</select>
```
- #### foreach标签

```xml
<!--public List<Teacher> getTeacherByids(@param("ids")List<integer> ids)-->
<select id="getTeacherByCondition"  resultMap="teacherMap">
    select * from t_teacher where id in 
    <!--
    collection:指定要遍历的集合的key
    close:指定以什么结束
    index="i"指定的变量保存了当前的索引
    item:指定当前遍历对象的变量名,可以用#{变量名}取值
    open:指定以什么开始
    separator:指定每个元素之间的分隔符
    -->
    <foreach collection="ids" open="("item="id_item" separator="," close=")"></foreach>
    foreach后的效果:(item1,item2...)
</select>
```
- #### choose标签

```xml
<!--public List<Teacher> getTeacherByConditionChosser(Teacher teacher)-->
<select id="getTeacherByConditionChosser"  resultMap="teacherMap">
    select * from t_teacher  
    <where>
        <choose>
            <when test="id!=null">
                id=#{id}
            </when>
            <when test="name!=null and name.equals('')">
                teacherName=#{name}
            </when>
            <when test="birth_date!=null">
                birth_date=#{birth}
            </when>
            <otherwise>
                1=1
            </otherwise>
        </choose>
    </where>
</select>
```
- #### set标签 ****

```xml
<!--public int updateTeacher(Teacher teacher)-->
<update id="updateTeacher">
    UPDATE emp 
    <set>
        <if test="name!=null and name!=''">
            teacherName=#{name}
        </if>
        <if test="birth!=null">
            birth_date=#{birth}
        </if>
    </set>
    <where>
        id=#{id}
    </where>
</update>
```
- #### bind标签

```xml
绑定一个表达式的值到一个变量上
<bind name="_name" value="'%'+name+'%'"/>
<if test="name!=null">
    teacherName like #{_name}
</if>

```
- #### sql标签
- 抽取可重用的sql语句

```xml
<sql id="sqlid">select * from t_teacher</sql>
<select id="" resultMap="">
    <include refid="sqlid"></include>
    where id =#{id}
</select>
```
- ## 缓存
    - 作用:暂时存储一些数据,加快系统的查询速度
    - 查询顺序:先看二级缓存有无数据,没有就看一级缓存中有无数据,没有就发送sql查询.(二一库)
    - 一级缓存和二级缓存中不会有同一个数据,因为二级缓存是在一级缓存消失后出现的
    - ### 一级缓存:
        - SQLSession级别的缓存
        - 之前查询过的数据,Mybatis会保存在一个缓存中(map),下次获取直接从缓冲中拿
        - 每次查询,先查看一级缓存中有没有改数据,没有就去发送sql语句,每个sqlSession拥有自己的一级缓存
        - #### 一级缓存失效的情况:
            - 不同的sqlSession会使用不同的一级缓存,只有在同一个sqlSession期间查询到的数据会保存在这个sqlSession的缓存中, 下次使用这个sqlSession查询会从缓存中拿
            - 同一个方法不同测参数会使用不同的一级缓存,因为之前没查过.
            - 在这个sqlSession期间执行过任何一次增删改查操作, 增删改操作会把缓存清空
            - 手动清空缓存会使缓存失效:sqlSession.clearCache()
    - ### 二级缓存:
    - 全局范围的缓冲
    - 默认不开启,需要手动配置
    - ==二级缓存在sqlSession关闭或者提交之后才会生效(一级缓存关闭)==
    - **使用步骤**
        - 在myBatis全局配置中设置开启二级缓存
            <settings><setting name ="cacheEnabled" value="true"></settings>
        - 配置某个dao.xml让其使用二级缓存在mapper下加上<cache></cache>即可
        - 对应的bean要实现serializable接口(序列化)
    - ### Mybatis对缓存的设置
        - 全局配置文件的cacheEnable:配置二级缓存的开关
        - select标签的useCache属性:这个属性配置这个select是否使用二级缓存
        - sql标签的flushCache属性:增删改默认flushCache="true",sql执行后会清空一级缓存和二级缓存,查询默认flushCache="false"
        - sqlSession.clearCache():该方法用于清除一级缓存
        
        






​    