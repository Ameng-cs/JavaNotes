# 在xml文件中配置bean

### 配置bean的简单属性并为之赋值

```xml
<bean id="person01" class="com.spring.bean.Persons">
<!-- 使用property标签为Person对象的属性赋值
    name="name",指定属性名
    value="张三":为这个属性赋值
    -->
    <property name="lastname" value="小王"></property>
    <property name="age" value="22"></property>
    <property name="gender" value="nan"></property>
</bean>
```

### 调用有参构造器创建对象并赋值

```xml
<bean id="person02" class="com.spring.bean.Persons">
    <!--调用有参构造器进行创建对象,并赋值-->
    <!--还可以直接写value不写name 但是顺序必须按照构造函数的参数顺序
        或者增加 index="1" 指定该value对应第二个参数但是遇到重载函数时会出现不明确的问题
    -->
    <constructor-arg name="lastname" value="小王"></constructor-arg>
    <constructor-arg name="age" value="12"></constructor-arg>
    <constructor-arg name="gender" value="男"></constructor-arg>
</bean>
```

### 为对象内的特殊成员赋值(对象)

```xml
    <bean id="person02" class="com.spring.bean.Persons" parent="person02">
    <!-- parent标签可以继承其他的bean -->
    <!-- 1.可以引用已存在的bean 通过ref="bean的id"       -->
    <!--<property name="car" ref="car"></property>-->
   
     <!-- 2.可以使用内部bean标签创建且内部的bean不能被获取到-->
        <property name="car">
            <bean class="com.spring.bean.Car">
                <property name="name" value="奔驰"></property>
                <property name="price" value="321"></property>
            </bean>
        </property>
    </bean>
```

### 为对象内的特殊成员赋值(List)

```xml
<bean id="person03" class="com.spring.bean.Persons" parent="person02">
    <property name="books">
        <!--  光写一对list标签等于声明一个引用,他的值为null  -->
        <list>
            <bean class="com.spring.bean.Book">
                <property name="price" value="123"></property>
                <property name="bookname" value="西游记"></property>
            </bean>
            <ref bean="book"></ref>
        </list>
    </property>
```

### 为对象内的特殊成员赋值(Map)

```xml
<bean id="person04" class="com.spring.bean.Persons" parent="person03">
    <property name="maps">
     <!--  光写一对map标签等于声明一个引用,他的值为null  -->
        <map>
            <entry key="key01" value="张三"></entry>
            <entry key="key02" value="18"></entry>
            <entry key="key03" value-ref="book"></entry>
            <entry key="key04">
                <bean class="com.spring.bean.Car">
                    <property name="price" value="321"></property>
                    <property name="name" value="夏利"></property>
                </bean>
            </entry>
        </map>
    </property>
</bean>
```

### 为级联属性(属性的属性)赋值

```xml
<bean id="person05" class="com.spring.bean.Persons" >
    <!--   为级联属性赋值,即为属性的属性赋值     -->
    <!--  需要注意的是原来的bean的值可能会被修改  -->
    <property name="car" ref="car"></property>
    <property name="car.price" value="999"></property>
</bean>
```

### 创建一个模板bean

```xml
<bean id="person05" class="com.spring.bean.Persons" abstract="true" >
<!-- abstract="true"  这个bean的配置是抽象的,不能获取他的实例,只能被别人继承 -->
    <property name="lastname" value="小天"></property>
    <property name="age" value="22"></property>
    <property name="gender" value="nan"></property>
</bean>
```

### bean之间的依赖

```xml
<!--bean的原始创建顺序是按照以下配置顺序创建的-->
<!-- 可以用depends-on="bean" 来决定创建顺序-->
<bean id="car" class="com.spring.bean.Car"></bean>
<bean id="person" class="com.spring.bean.Person"></bean>
<bean id="book" class="com.spring.bean.Book" depends-on="car"></bean>
```

### 利用工厂方法创建bean

- - **实现了FactoryBean接口的类是Spring可以认识的工厂类,spring会自动创建工厂方法创建实例**

- - **静态工厂(不需要new该工厂对象)**

```xml
factory-method="工厂方法" 通过该属性指定创建实例的工厂方法,在通过有参构造器创建实例
<bean id="tool" class="com.spring.factory.ToolFactory" factory-method="getTool">
    <constructor-arg name="属性名" value="属性值"></constructor-arg>
    <constructor-arg name="属性名" value="属性值"></constructor-arg>
    <constructor-arg name="属性名" value="属性值"></constructor-arg>
 </bean>
```

- 实例工厂

  ```xml
  1.先配置实例工厂对象
  2.再配置我们要创建的对象,并指出使用哪个工厂
      factory-bean:指出使用呢个工厂实例
      factory-method:指出使用哪个工厂方法
  factory-method="工厂方法" 通过该属性指定该实例工厂中那个是工厂方法,在通过有参构造器创建实例
  <bean id="ToolInstanceFactory" class="com.spring.factory.ToolInstanceFactory" factory-method="getTool">
   </bean>
   factory-bean="实例工厂" 指定当前对象创建使用哪个工厂
   <bean id="Tool" class="com.spring.factory.Tool" factory-method="getTool" factory-bean="">
       <constructor-arg name="属性名" value="属性值"></constructor-arg>
       <constructor-arg name="属性名" value="属性值"></constructor-arg>
   </bean>
  ```

### 引用外部属性文件

```xml
数据库连接池应是单例的,可以让spring帮我们创建连接池对象(管理连接池)
context:property-placeholder:这个标签用来加载外部的的配置文件
location属性中classpath为固定写法:表示引用类路径下的一个资源

<context:property-placeholder location="classpath:c3p0_config.properties"></context:property-placeholder>
    <!-- 加载外部配置文件  -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <!--值得注意的是username是Spring内部的一个关键字所以在配置文件中取名的时候,可以加其他的东西以区别-->
    <property name="user" value="${username}"/>
    <!-- ${key}动态的取出配置文件中某个key对应的值 -->
    <property name="password" value="${password}"></property>
    <property name="jdbcUrl" value="${jdbcUrl}"></property>
    <property name="driverClass" value="${driverClass}"></property>
</bean>
或者不用配置文件直接将${}这个内容替换成对应的值
```

# Annotation

Spring通过注解(Annotation)可以快速便捷的将bean添加到IOC容器中去,在类前添加以下四个注解:

1. **@Repository-->持久化层**
2. **@Service-->业务层**
3. **@Controller-->视图控制器**
4. **@Component-->组件**

Spring不会检查你添加的类与注释是否对应,注解可以不对应类的功能,但是我们还是应该按照规则添加注释,使得可读性更强

```xml
@Service("xxx")              <!--这样可以改变bean的Id-->
@Scope(value="prototype")    <!--该注解可以修改bean的作用域,通过注解创建的bean默认是单实例的,这样可以建立多实例bean-->
```

另外使用注解声明bean的话要在Spring的ApplicationContext.xml中做如下配置

```xml
<!--base-package属性:指定扫描的基础包,把基础包及他下面的所有的包的所有加了注解的类自动的扫描进ioc容器中-->
     <!--type="annotation":指定排除规则:按照注解进行排除,标注了指定注解的组件不要;expression="":注解的全类名-->
     <!--type="assignable":指定排除某个具体的类; expression="":要排除类的全类名-->
<context:component-scan base-package="com.spring"></context:component-scan>
<!--指定扫描的基础包,在扫描的时候忽略指定类-->
<context:component-scan base-package="com.spring.Services">
     <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>	
</context:component-scan>
<!--指定扫描的基础包,在扫描的时候只扫描指定类,use-default-filter该属性默认是true也就是全部都扫描,所以要关闭该属性,只扫描指定的类-->
<context:component-scan base-package="com.spring.Controllers" use-default-filter="false"/>
     <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"></context:component-scan>
</context:component-scan>
```

### 依赖注入(DI)--@Autowired

- **装配策略**
  - ​	按照类型(byType)去IOC容器中寻找Bean的实例
    - 找到了一个指定类型的Bean实例:直接装配
    - 找到了指定类型的bean实例
      - 按照id(byName)寻找对应的Bean实例
        - 找到:装配
        - 未找到:抛出异常:NoUniqueBeanDefinitionException
      - (如果有)按照@Qualifier注解内的id进行查找
        - 找到:装配
        - 未找到:抛出异常
    - 没找到则抛出异常:raiseNoSuchBeanDefinitionException
- **默认属性**

```xml
@Autowired(request=true)        <!--默认:找不到符合的实例bean 就报错-->
@Autowired(request=false)		<!--找不到符合的实例bean就装配 null-->
```

- **@Qualifier("name")**

  - 指定一个名字作为id,让Spring不使用变量名作为id
  - 使用场景:当使用@Autowired自动装配的时候若在容器中找到多个相同的类,就会再按变量名的id去查找, ==如果要改变量名,那么该类里所有使用了该变量的方法都要改==,这样就加大的工作量,    所以引入@Qualifier注解来让Spring不再使用变量名作为id进行查找,而是按照注解指定的名字进行查找

  