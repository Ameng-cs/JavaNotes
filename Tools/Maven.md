###  Maven是一款自动化构建工具，专注服务于Java平台的项目构建和依赖管理
- ## Maven的核心概念   
- ### 约定的目录结构
    - src
        - main 存放主程序
            - java 源代码
            - resources 配置文件和资源文件
        - test 存放测试程序
            - java 
            - resources
    - target
    - pom.xml
- ### POM

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

<modelVersion>4.0.0</modelVersion>

<!-- 坐标-->
<groupId>com.ct.Maven</groupId>
<artifactId>MavenTest</artifactId>
<version>1.0-SNAPSHOT</version>
<packaging>war</packaging>
<!-- Maven项目类型
jar 表示java项目，默认值
war 表示web项目
pom 表示父工程
-->
<!-- 项目名-->
<name>MavenTest Maven Webapp</name>

    <!-- 全局变量声明-->
<properties>
    <!--声明版本号-->
    <spring.version> 5.2.5.RELEASE</spring.version>
</properties>

<!--当前工程继承了某一工程-->
<parent>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <!-- 指定父工程的pom文件的位置-->
    <relativePath>../pom.xml</relativePath>
</parent>

<!--  依赖管理配置声明:声明配置,当前项目并不会直接引入jar包.
      子项目继承父项目,子项目不能直接使用jar包,子项目想用,必须声明才能使用
      好处:由父工程管理版本,子工程不需要管理版本-->
 <dependencyManagement>
    <dependencies>
       <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependencies>
  </dependencyManagement>

<!-- 依赖管理：需要引用第三方jar包，通过dependencies标签来进行配置 
     jar会被当前项目引用,子项目可以继承当前项目,并可以直接使用-->
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <!--如果继承成了其他工程,建议不写version和依赖范围 ,直接由父工程来管理-->
        <version>4.11</version>
        <scope>test</scope>
        <!-- 依赖范围
        compile:编译范围,默认值
             这个范围的包,可以给main和test下面的类使用; 参与部署
        test:测试范围
           这个范围的包只给test下面的类使用,main下面的类不能使用;不参与部署
        provided: 提供范围
                这个范围的包,可以给main和test下面的类使用;不参与部署
        -->
    </dependency>
     <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <!--使用上面声明的版本号-->
        <version>${spring.version}</version>
    </dependency>
</dependencies>
    
</project>
```
- ### 坐标
    - groupId：公司或组织的域名倒叙+当前项目名称
    - artifactId：当前项目模块名称
    - version：当前模块的版本
- ### 依赖
    - #### 依赖的范围
        - **compile:** 编译范围,默认值
                  这个范围的包,可以给main和test下面的类使用;
                  参与部署
        - **test:** 测试范围
           这个范围的包只给test下面的类使用,main下面的类不能使用;不参与部署
        - **provided:** 测试范围
           这个范围的包只给test下面的类使用,main下面的类不能使用;不参与部署
    - #### 依赖的传递性
        - compile范围具有传递性 A->B->C->D  如果D为compile A可以看到D
        - test和provided不具有传递性 A->B->C->D  如果D为test或者provided A可以看不到D
    - #### 依赖原则
        - ##### 路径最短者优先
            如果间接依赖了不同版本号的jar包,maven会选择离自己依赖距离最近的那个项目的对应版本
        - ##### 路径相同时先声明者优先
            如果依赖了多个项目但是这几个项目依赖了不同版本的一个项目,这时候maven就会选择先声明(dependency 里面的顺序)的项目的版本
    - #### 依赖的排除
        对所依赖的包的其他包进行依赖排除,把用不上的包进行排除,不需要传递.
- ### 仓库
    - 本地仓库
    - 远程仓库
        - 私服
        - 中央仓库
        - 中央仓库镜像
            - 阿里云镜像服务器
        
```xml
 <mirror>
　　<id>alimaven</id>
　　<mirrorOf>central</mirrorOf>
　　<name>aliyun maven</name>
　　<url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

- ### 生命周期
    -  Clean Lifecycle 在进行真正的构建之前进行的一些清理工作
    -  Default Lifecycle 构建核心部分,编译测试,打包,安装,部署等等
    -  Site Lifecycle 生成项目报告,站点,发布站点
    -  这三个生命周期是相互独立的
- ### 继承

```xml

<!--当前工程继承了某一工程-->
<parent>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <!-- 指定父工程的pom文件的位置-->
    <relativePath>../pom.xml</relativePath>
</parent>

<!--  依赖管理配置声明:声明配置,当前项目并不会直接引入jar包.
      子项目继承父项目,子项目不能直接使用jar包,子项目想用,必须声明才能使用
      好处:由父工程管理版本,子工程不需要管理版本-->
 <dependencyManagement>
    <dependencies>
       <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependencies>
  </dependencyManagement>
  <!-- 依赖管理：需要引用第三方jar包，通过dependencies标签来进行配置 
     jar会被当前项目引用,子项目可以继承当前项目,并可以直接使用-->
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <!--如果继承成了其他工程,建议不写version和依赖范围 ,直接由父工程来管理-->
        <!--<version>4.11</version>-->
        <!--<scope>test</scope>-->

    </dependency>
</dependencies>
```

- ### 聚合

```xml
<!-- 聚合:对当前项目进行任何操作,被聚合的项目都会做相同的操作-->
<modules>
    <module>A</module>
    <module>B</module>
    <module>C</module>
    <module>D</module>
</modules>

```