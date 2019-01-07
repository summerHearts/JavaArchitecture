
##1、权限管理系统核心表设计

![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-137_15-17-38.png)
 
```
SET NAMES utf8;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
--  Table structure for `sys_acl`
-- ----------------------------
DROP TABLE IF EXISTS `sys_acl`;
CREATE TABLE `sys_acl` (
  `id` int(11) NOT NULL COMMENT '权限id',
  `code` varchar(20) NOT NULL COMMENT '权限码',
  `name` varchar(20) NOT NULL COMMENT '权限名称',
  `acl_module_id` int(11) NOT NULL COMMENT '权限模块的id',
  `url` varchar(100) NOT NULL COMMENT '请求的url,可以填正则表达式',
  `type` int(11) NOT NULL COMMENT '类型(1: 菜单，2:按钮，3:其他)',
  `status` int(11) NOT NULL DEFAULT '1' COMMENT '状态(1: 正常，0:停用)',
  `seq` int(11) NOT NULL DEFAULT '0' COMMENT '权限在当前层级下的顺序，由大到小',
  `remark` varchar(200) DEFAULT '' COMMENT '备注',
  `operator` varchar(20) NOT NULL DEFAULT '' COMMENT '操作者',
  `operate_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最后一次操作的时间',
  `operate_ip` varchar(20) NOT NULL DEFAULT '' COMMENT '最后一次操作的ip',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
--  Table structure for `sys_acl_module`
-- ----------------------------
DROP TABLE IF EXISTS `sys_acl_module`;
CREATE TABLE `sys_acl_module` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '权限id',
  `name` varchar(20) NOT NULL DEFAULT '' COMMENT '权限名称',
  `parent_id` int(11) NOT NULL DEFAULT '0' COMMENT '上级权限id',
  `level` varchar(20) NOT NULL DEFAULT '' COMMENT '权限层级',
  `status` int(11) NOT NULL DEFAULT '1' COMMENT '状态(1: 正常，0:停用)',
  `seq` int(11) NOT NULL DEFAULT '0' COMMENT '权限在当前层级下的顺序，由大到小',
  `remark` varchar(200) DEFAULT '' COMMENT '备注',
  `operator` varchar(20) NOT NULL DEFAULT '' COMMENT '操作者',
  `operate_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最后一次操作的时间',
  `operate_ip` varchar(20) NOT NULL DEFAULT '' COMMENT '最后一次操作的ip',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
--  Table structure for `sys_dept`
-- ----------------------------
DROP TABLE IF EXISTS `sys_dept`;
CREATE TABLE `sys_dept` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '部门id',
  `name` varchar(20) NOT NULL DEFAULT '' COMMENT '部门名称',
  `parent_id` int(11) NOT NULL DEFAULT '0' COMMENT '上级部门id',
  `level` varchar(20) NOT NULL DEFAULT '' COMMENT '部门层级',
  `seq` int(11) NOT NULL DEFAULT '0' COMMENT '部门在当前层级下的顺序，由大到小',
  `remark` varchar(200) DEFAULT '' COMMENT '备注',
  `operator` varchar(20) NOT NULL DEFAULT '' COMMENT '操作者',
  `operate_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最后一次操作的时间',
  `operate_ip` varchar(20) NOT NULL DEFAULT '' COMMENT '最后一次操作的ip',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
--  Table structure for `sys_log`
-- ----------------------------
DROP TABLE IF EXISTS `sys_log`;
CREATE TABLE `sys_log` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `type` int(11) NOT NULL COMMENT '权限更新的类型(1:部门，2:用户，3：权限模块，4:权限，5:角色，6:角色用户关系，7:角色权限关系)',
  `tagret_id` int(11) NOT NULL COMMENT '基于type后指定的对象的id,比如用户、权限、角色表的主键',
  `old_value` text,
  `new_value` text,
  `operator` varchar(20) NOT NULL DEFAULT '' COMMENT '操作者',
  `operate_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最后一次操作的时间',
  `operate_ip` varchar(20) NOT NULL DEFAULT '' COMMENT '最后一次操作的ip',
  `status` int(11) NOT NULL COMMENT '当前是否复原过(0: 没有，1:复原过)',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
--  Table structure for `sys_role`
-- ----------------------------
DROP TABLE IF EXISTS `sys_role`;
CREATE TABLE `sys_role` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '角色id',
  `name` varchar(20) NOT NULL DEFAULT '' COMMENT '角色名称',
  `type` int(11) NOT NULL DEFAULT '1' COMMENT '角色类型(1: 管理员，2:其他)',
  `status` int(11) NOT NULL DEFAULT '1' COMMENT '状态(1: 正常，0:停用)',
  `remark` varchar(200) DEFAULT '' COMMENT '备注',
  `operator` varchar(20) NOT NULL DEFAULT '' COMMENT '操作者',
  `operate_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最后一次操作的时间',
  `operate_ip` varchar(20) NOT NULL DEFAULT '' COMMENT '最后一次操作的ip',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
--  Table structure for `sys_role_acl`
-- ----------------------------
DROP TABLE IF EXISTS `sys_role_acl`;
CREATE TABLE `sys_role_acl` (
  `id` int(11) NOT NULL,
  `role_id` int(11) NOT NULL COMMENT '角色id',
  `acl_id` int(11) NOT NULL COMMENT '权限id',
  `operator` varchar(20) NOT NULL DEFAULT '' COMMENT '操作者',
  `operate_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最后一次操作的时间',
  `operate_ip` varchar(20) NOT NULL DEFAULT '' COMMENT '最后一次操作的ip',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
--  Table structure for `sys_role_user`
-- ----------------------------
DROP TABLE IF EXISTS `sys_role_user`;
CREATE TABLE `sys_role_user` (
  `id` int(11) NOT NULL,
  `role_id` int(11) NOT NULL COMMENT '角色id',
  `user_id` int(11) NOT NULL,
  `operator` varchar(20) NOT NULL DEFAULT '' COMMENT '操作者',
  `operate_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最后一次操作的时间',
  `operate_ip` varchar(20) NOT NULL DEFAULT '' COMMENT '最后一次操作的ip',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
--  Table structure for `sys_user`
-- ----------------------------
DROP TABLE IF EXISTS `sys_user`;
CREATE TABLE `sys_user` (
  `id` int(11) NOT NULL COMMENT '用户id',
  `username` varchar(20) NOT NULL COMMENT '用户名称',
  `phone` varchar(13) NOT NULL COMMENT '用户手机号',
  `email` varchar(20) NOT NULL COMMENT '用户邮箱',
  `password` varchar(40) NOT NULL COMMENT '用户密码',
  `dept_id` int(11) NOT NULL COMMENT '部门id',
  `status` int(11) NOT NULL DEFAULT '1' COMMENT '状态(1: 正常，0:停用)',
  `remark` varchar(200) DEFAULT NULL COMMENT '备注信息',
  `operator` varchar(20) NOT NULL COMMENT '操作人',
  `operate_time` datetime NOT NULL COMMENT '最后一次的操作时间',
  `operate_ip` varchar(20) NOT NULL COMMENT '操作的ip地址',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

SET FOREIGN_KEY_CHECKS = 1;
```

  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-137_15-27-10.png)


##2、Spring MVC开发环境搭建
  
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <groupId>com.wangpu</groupId>
  <artifactId>permission</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>permission Maven Webapp</name>
  <url>http://maven.apache.org</url>

  <properties>
    <springframework.version>4.3.10.RELEASE</springframework.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.springframework/spring-beans -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>${springframework.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${springframework.version}</version>
    </dependency>

    <!--Spring MVC + Spring web -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${springframework.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${springframework.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${springframework.version}</version>
    </dependency>

    <!-- mybatis -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.0</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.0</version>
    </dependency>

    <!-- druid -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.0.20</version>
    </dependency>

    <!-- mysql -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.30</version>
    </dependency>

    <!-- lombok -->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.16.12</version>
    </dependency>

    <!-- Jackson -->
    <dependency>
      <groupId>com.fasterxml.jackson.datatype</groupId>
      <artifactId>jackson-datatype-guava</artifactId>
      <version>2.5.3</version>
    </dependency>

    <!-- logback -->
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-core</artifactId>
      <version>1.1.8</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.1.8</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.22</version>
    </dependency>

    <!-- jsp api -->
    <dependency>
      <groupId>org.apache.tomcat</groupId>
      <artifactId>jsp-api</artifactId>
      <version>6.0.36</version>
    </dependency>

    <!-- validator -->
    <dependency>
      <groupId>javax.validation</groupId>
      <artifactId>validation-api</artifactId>
      <version>1.1.0.Final</version>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-validator</artifactId>
      <version>5.2.4.Final</version>
    </dependency>

    <!-- tools -->
    <dependency>
      <groupId>commons-collections</groupId>
      <artifactId>commons-collections</artifactId>
      <version>3.2.2</version>
    </dependency>
    <dependency>
      <groupId>commons-codec</groupId>
      <artifactId>commons-codec</artifactId>
      <version>1.10</version>
    </dependency>

    <!-- jackson -->
    <dependency>
      <groupId>org.codehaus.jackson</groupId>
      <artifactId>jackson-core-asl</artifactId>
      <version>1.9.13</version>
    </dependency>
    <dependency>
      <groupId>org.codehaus.jackson</groupId>
      <artifactId>jackson-mapper-asl</artifactId>
      <version>1.9.13</version>
    </dependency>

    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.5</version>
    </dependency>

    <!-- email -->
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-email</artifactId>
      <version>1.4</version>
    </dependency>

    <!-- redis -->
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>2.8.1</version>
      <type>jar</type>
    </dependency>

  </dependencies>

  <build>
    <finalName>permission</finalName>
  </build>
</project>
```

web.xml配置

```
<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <!-- Spring beans 配置文件所在目录 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>

  <!-- spring mvc 配置 -->
  <servlet>
    <servlet-name>spring</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>spring</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

  <!-- Encoding -->
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
      <param-name>forceEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</webapp>
```

/与/*的区别：

  - /可以拦截到/login等，但拦截不到/login.jsp.而/*所有路径都可以拦截到

在WEB-INF下创建spring-servlet.xml文件（请求相关的配置）

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    
    <context:annotation-config />
    <!-- 启动注解驱动的spring mvc 功能 -->
    <mvc:annotation-driven/>
    <context:component-scan base-package="com.wangpu.controller"/>
    <context:component-scan base-package="com.wangpu.service"/>

    <!-- 请求映射-->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
    <!-- 返回的数据格式-->
    <bean class="org.springframework.web.servlet.view.BeanNameViewResolver"/>
    <bean id="jsonView" class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
    <!-- 内部视图资源的解析  文件路径 返回格式-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/" /> 
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

applicationContext.xml配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.alibaba.com/schema/stat http://www.alibaba.com/schema/stat.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="ignoreUnresolvablePlaceholders" value="true"/>
        <property name="locations">
            <list>
                <value>classpath:settings.properties</value>
            </list>
        </property>
    </bean>


    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="driverClassName" value="${db.driverClassName}" />
        <property name="url" value="${db.url}" />
        <property name="username" value="${db.username}" />
        <property name="password" value="${db.password}" />
        <property name="initialSize" value="3" />
        <property name="minIdle" value="3" />
        <property name="maxActive" value="20" />
        <property name="maxWait" value="60000" />
        <!--<property name="filters" value="stat,wall" />-->
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="configLocation" value="classpath:mybatis-config.xml" />
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations" value="classpath:mapper/*.xml" />
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.wangpu.dao" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
    </bean>

    <!-- tx -->
    <!--<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">-->
        <!--<property name="dataSource" ref="dataSource" />-->
    <!--</bean>-->

    <!--<tx:annotation-driven transaction-manager="transactionManager" />-->

    <!-- druid -->
    <!--<bean id="stat-filter" class="com.alibaba.druid.filter.stat.StatFilter">-->
        <!--<property name="slowSqlMillis" value="3000" />-->
        <!--<property name="logSlowSql" value="true" />-->
        <!--<property name="mergeSql" value="true" />-->
    <!--</bean>-->
    <!--<bean id="wall-filter" class="com.alibaba.druid.wall.WallFilter">-->
        <!--<property name="dbType" value="mysql" />-->
    <!--</bean>-->
</beans>
```

mybatis-config.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <setting name="safeRowBoundsEnabled" value="true"/>
        <setting name="cacheEnabled" value="false" />
        <setting name="useGeneratedKeys" value="true" />
    </settings>
    </configuration>
```

Druid如何配置

首先是过滤器filter的配置,在web.xml中添加如下配置

```
<servlet>
      <servlet-name>DruidStatView</servlet-name>
      <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
  </servlet>
  <servlet-mapping>
      <servlet-name>DruidStatView</servlet-name>
      <url-pattern>/sys/druid/*</url-pattern>
  </servlet-mapping>
  
 <filter> 
   <filter-name>druidWebStatFilter</filter-name> 
  <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class> 
  <init-param> 
   <param-name>exclusions</param-name> 
   <param-value>/public/*,*.js,*.css,/druid*,*.jsp,*.swf</param-value> 
  </init-param> 
  <init-param> 
   <param-name>principalSessionName</param-name> 
   <param-value>sessionInfo</param-value> 
  </init-param> 
  <init-param> 
   <param-name>profileEnable</param-name> 
   <param-value>true</param-value> 
  </init-param> 
 </filter> 
 
 <filter-mapping> 
  <filter-name>druidWebStatFilter</filter-name> 
  <url-pattern>/*</url-pattern> 
 </filter-mapping>
```

把上面servlet配置添加到项目web.xml即可。然后运行Tomcat，浏览器输入 http://IP:PROT/druid 
就可以打开Druid的监控页面了.


##3、mybatis generator
- 1.pom.xml文件引入mybatis maven plugin

```
plugin>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-maven-plugin</artifactId>
        <version>1.3.2</version>
        <configuration>
          <verbose>true</verbose>
          <overwrite>true</overwrite>
        </configuration>
</plugin>
```

- 2.创建 generatorConfig.xml到resource目录下

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--导入属性配置-->
    <properties resource="settings.properties"></properties>

    <!--指定特定数据库的jdbc驱动jar包的位置-->
    <classPathEntry location="${db.driverLocation}"/>

    <context id="default" targetRuntime="MyBatis3">

        <!-- optional，旨在创建class时，对注释进行控制 -->
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--jdbc的数据库连接 -->
        <jdbcConnection
                driverClass="${db.driverClassName}"
                connectionURL="${db.url}"
                userId="${db.username}"
                password="${db.password}">
        </jdbcConnection>


        <!-- 非必需，类型处理器，在数据库类型和java类型之间的转换控制-->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>


        <!-- Model模型生成器,用来生成含有主键key的类，记录类 以及查询Example类
            targetPackage     指定生成的model生成所在的包名
            targetProject     指定在该项目下所在的路径
        -->
        <!--<javaModelGenerator targetPackage="com.wangpu.pojo" targetProject=".\src\main\java">-->
        <javaModelGenerator targetPackage="com.wangpu.model" targetProject="./src/main/java">
            <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="false"/>
            <!-- 是否对model添加 构造函数 -->
            <property name="constructorBased" value="true"/>
            <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
            <property name="trimStrings" value="true"/>
            <!-- 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
            <property name="immutable" value="false"/>
        </javaModelGenerator>

        <!--mapper映射文件生成所在的目录 为每一个数据库的表生成对应的SqlMap文件 -->
        <!--<sqlMapGenerator targetPackage="mappers" targetProject=".\src\main\resources">-->
        <sqlMapGenerator targetPackage="mappers" targetProject="./src/main/resources">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>

        <!-- 客户端代码，生成易于使用的针对Model对象和XML配置文件 的代码
                type="ANNOTATEDMAPPER",生成Java Model 和基于注解的Mapper对象
                type="MIXEDMAPPER",生成基于注解的Java Model 和相应的Mapper对象
                type="XMLMAPPER",生成SQLMap XML文件和独立的Mapper接口
        -->

        <!-- targetPackage：mapper接口dao生成的位置 -->
        <!--<javaClientGenerator type="XMLMAPPER" targetPackage="com.wangpu.dao" targetProject=".\src\main\java">-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.wangpu.dao" targetProject="./src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>


        <table tableName="sys_user" domainObjectName="SysUser" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />
        <table tableName="sys_dept" domainObjectName="SysDept" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />
        <table tableName="sys_acl" domainObjectName="SysAcl" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />
        <table tableName="sys_acl_module" domainObjectName="SysAclModule" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />
        <table tableName="sys_role" domainObjectName="SysRole" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />
        <table tableName="sys_role_acl" domainObjectName="SysRoleAcl" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />
        <table tableName="sys_role_user" domainObjectName="SysRoleUser" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />
        <table tableName="sys_log" domainObjectName="SysLog" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />

        <!-- geelynote mybatis插件的搭建 -->
    </context>
</generatorConfiguration>
```

- 3.resource目录下创建settings.properties

```
db.driverLocation=/Users/yintingting/Desktop/mysql-connector-java-5.1.46.jar //其实可以不填
db.driverClassName=com.mysql.jdbc.Driver
db.url=jdbc:mysql://localhost:3306/permission?useUnicode=true&characterEncoding=UTF-8
db.username=root
db.password=123456
```

- 4.从maven仓库下载和pom.xml中的mysql驱动一样版本的jar包，把它的路径写入：db.driverLocation

- 5.IDEA右侧Maven Projects-plugins-mybatis-generator双击-mybatis-generator generate双击，新文件生成。


##4、异常处理

自定义一个异常

```
public class PermissionException extends RuntimeException {

    public PermissionException() {
        super();
    }

    public PermissionException(String message) {
        super(message);
    }

    public PermissionException(String message, Throwable cause) {
        super(message, cause);
    }

    public PermissionException(Throwable cause) {
        super(cause);
    }

    protected PermissionException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

异常处理器

```
@Slf4j
public class SpringExceptionResolver implements HandlerExceptionResolver {

　　项目中发生异常都会在这个方法里处理，这里写我的项目中的处理方式。你可以根据自己的项目处理发生的异常。
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        String url = request.getRequestURL().toString();
        ModelAndView mv;
        String defaultMsg = "System error";

        // 这里我们要求项目中所有请求json数据，都使用.json结尾
        if (url.endsWith(".json")) {
            if (ex instanceof PermissionException) {
                JsonData result = JsonData.fail(ex.getMessage());
                mv = new ModelAndView("jsonView", result.toMap());
            } else {
                log.error("unknown json exception, url:" + url, ex);
                JsonData result = JsonData.fail(defaultMsg);
                mv = new ModelAndView("jsonView", result.toMap());//不需要有jsonView.jsp,因为
<bean id="jsonView" class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
            }
        } else if (url.endsWith(".page")){ // 这里我们要求项目中所有请求page页面，都使用.page结尾
            log.error("unknown page exception, url:" + url, ex);
            JsonData result = JsonData.fail(defaultMsg);
            mv = new ModelAndView("exception", result.toMap());//回去找exception.jsp页面
        } else {
            log.error("unknown exception, url:" + url, ex);
            JsonData result = JsonData.fail(defaultMsg);
            mv = new ModelAndView("jsonView", result.toMap());
        }

        return mv;
    }
}
```

views下创建exception.jsp
在spring-servlet.xml中配置SpringExceptionResolver

```
<!--全局异常处理-->
<bean class="com.wangpu.common.SpringExceptionResolver”/>
```

```
<!--使用视图名称来解析视图,下面两个bean一起,才能正确返回jsonView-->
    <bean class="org.springframework.web.servlet.view.BeanNameViewResolver" />
    <bean id="jsonView" class="org.springframework.web.servlet.view.json.MappingJackson2JsonView" />
```

验证：

```
Controller
@RequestMapping("/test")
@Slf4j
public class TestController {

    @RequestMapping("/hello.json")
    @ResponseBody
    public JsonData hello(){
        log.info("hello");
        return JsonData.success("hello, permission");
    }
}
```

输入localhost:8080/test/hello.json,页面显示

```
{
  "ret": true,
  "msg": null,
  "data": "hello, permission"
}
```
制造异常

```
@Controller
@RequestMapping("/test")
@Slf4j
public class TestController {

    @RequestMapping("/hello.json")
    @ResponseBody
    public JsonData hello(){
        log.info("hello");
        throw new PermissionException("test exception");
//        return JsonData.success("hello, permission");
    }
}
```
localhost:8080/test/hello.json显示

```
{
  "ret": false,
  "msg": "System error",
  "data": null
}
```

##5、json转换工具

```
import lombok.extern.slf4j.Slf4j;
import org.codehaus.jackson.map.DeserializationConfig;
import org.codehaus.jackson.map.ObjectMapper;
import org.codehaus.jackson.map.SerializationConfig;
import org.codehaus.jackson.map.annotate.JsonSerialize;
import org.codehaus.jackson.map.ser.impl.SimpleFilterProvider;
import org.codehaus.jackson.type.TypeReference;

@Slf4j
public class JsonMapper {

    private static ObjectMapper objectMapper = new ObjectMapper();

    static {
        // config
        objectMapper.disable(DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES);
        objectMapper.configure(SerializationConfig.Feature.FAIL_ON_EMPTY_BEANS, false);
        objectMapper.setFilters(new SimpleFilterProvider().setFailOnUnknownId(false));
        objectMapper.setSerializationInclusion(JsonSerialize.Inclusion.NON_EMPTY);
    }

    public static <T> String obj2String(T src) {
        if (src == null) {
            return null;
        }
        try {
            return src instanceof String ? (String) src : objectMapper.writeValueAsString(src);
        } catch (Exception e) {
            log.warn("parse object to String exception, error:{}", e);
            return null;
        }
    }

    public static <T> T string2Obj(String src, TypeReference<T> typeReference) {
        if (src == null || typeReference == null) {
            return null;
        }
        try {
            return (T) (typeReference.getType().equals(String.class) ? src : objectMapper.readValue(src, typeReference));
        } catch (Exception e) {
            log.warn("parse String to Object exception, String:{}, TypeReference<T>:{}, error:{}", src, typeReference.getType(), e);
            return null;
        }
    }
}
``` 

##6、获取Spring上下文工具

```
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component("applicationContextHelper")
public class ApplicationContextHelper implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    public void setApplicationContext(ApplicationContext context) throws BeansException {
        applicationContext = context;
    }

    public static <T> T popBean(Class<T> clazz) {
        if (applicationContext == null) {
            return null;
        }
        return applicationContext.getBean(clazz);
    }

    public static <T> T popBean(String name, Class<T> clazz) {
        if (applicationContext == null) {
            return null;
        }
        return applicationContext.getBean(name, clazz);
    }
}
```
spring-servlet中配置

```
<!--不让它懒加载,启动的时候直接加载-->
<bean class="com.wangpu.common.ApplicationContextHelper" lazy-init="false"/>
```

验证JsonMapper和ApplicationContextHelper

```
//TestController
    @RequestMapping("/select")
    @ResponseBody
    public JsonData validate(TestVo vo)  {

        SysAclModuleMapper moduleMapper = ApplicationContextHelper.popBean(SysAclModuleMapper.class);
        SysAclModule module = moduleMapper.selectByPrimaryKey(1);

        return JsonData.success(module);
    }
```
运行tomcat,结果

```
curl http://localhost:8080/test/validate1.json
{"ret":false,"msg":"{str=不能为空, id=id 不可以为空哦, msg=不能为空}","data":null} 
```

使用mybatis的时候注意，mybatis和mybatis-spring一定要对应，当mybatis版本在3.4.0以上，需要mybatis-spring在1.3.0以上

##7、Http请求前后监听工具

工具作用：

 - 1. 输出每次请求的参数

 - 2. 接口的请求时间

```
import javax.servlet.http.HttpServletRequest;

public class RequestHolder {

    private static final ThreadLocal<SysUser> userHolder = new ThreadLocal<SysUser>();

    private static final ThreadLocal<HttpServletRequest> requestHolder = new ThreadLocal<HttpServletRequest>();

    public static void add(SysUser sysUser) {
        userHolder.set(sysUser);
    }

    public static void add(HttpServletRequest request) {
        requestHolder.set(request);
    }

    public static SysUser getCurrentUser() {
        return userHolder.get();
    }

    public static HttpServletRequest getCurrentRequest() {
        return requestHolder.get();
    }

//进程结束的需要remove,否则会造成内存溢出
    public static void remove() {
        userHolder.remove();
        requestHolder.remove();
    }
}
```

 
```
import com.wangpu.util.JsonMapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Map;

@Slf4j
public class HttpInterceptor extends HandlerInterceptorAdapter {

    private static final String START_TIME = "requestStartTime";

    //每个请求前的方法
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String url = request.getRequestURI().toString();
        Map parameterMap = request.getParameterMap();
        log.info("request start. url:{}, params:{}", url, JsonMapper.obj2String(parameterMap));
        long start = System.currentTimeMillis();
        request.setAttribute(START_TIME, start);
        return true;
    }
    //正常处理一个http请求后调用的方法
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
//        String url = request.getRequestURI().toString();
//        long start = (Long) request.getAttribute(START_TIME);
//        long end = System.currentTimeMillis();
//        log.info("request finished. url:{}, cost:{}", url, end - start);
        removeThreadLocalInfo();
    }
    //任何情况下，包括异常情况下请求后都会调用的方法
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        String url = request.getRequestURI().toString();
        long start = (Long) request.getAttribute(START_TIME);
        long end = System.currentTimeMillis();
        log.info("request completed. url:{}, cost:{}", url, end - start);

        removeThreadLocalInfo();
    }

    public void removeThreadLocalInfo() {
        RequestHolder.remove();
    }
}
```

使用mvc:interceptors标签来声明需要加入到SpringMVC拦截器链中的拦截器

```
<mvc:interceptors>  
    <!-- 使用bean定义一个Interceptor，直接定义在mvc:interceptors根下面的Interceptor将拦截所有的请求 -->  
    <bean class="com.wangpu.common.interceptor.AllInterceptor"/>  
    <mvc:interceptor>  
        <mvc:mapping path="/test/number.do"/>  
        <!-- 定义在mvc:interceptor下面的表示是对特定的请求才进行拦截的 -->  
        <bean class="com.wangpu.common.interceptor.HttpInterceptor"/>  
    </mvc:interceptor>  
</mvc:interceptors>  
```


