##Maven详细教程

- 环境搭建
  - 1、http://maven.apache.org/download.cgi 

     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_20-47-09.png)

  - 2、解压即可 

  - 3、配置环境变量
  
     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_20-57-51.png)
     
     具体命令如下
     
     ```
       Documents kevin$ touch .bash_profile
       Documents kevin$ vim .bash_profile 
    
        MAVEN_HOME=/Users/kevin/Documents/apache-maven-3.5.4
        PATH=$MAVEN_HOME/bin:$PATH
        export MAVEN_HOME
        export PATH
        
        Documents kevin$ source .bash_profile 
        Documents kevin$ mvn -v
        Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-18T02:33:14+08:00)
        Maven home: /Users/kevin/Documents/apache-maven-3.5.4
        Java version: 1.8.0_172, vendor: Oracle Corporation, runtime: /Library/Java/JavaVirtualMachines/jdk1.8.0_172.jdk/Contents/Home/jre
        Default locale: zh_CN, platform encoding: UTF-8
        OS name: "mac os x", version: "10.13.5", arch: "x86_64", family: "mac"
     ```
- 2、替换maven阿里云中央仓库 

  - 修改maven根目录下的conf文件夹中的setting.xml文件，内容如下：
  
        ```
        <mirrors>
    
            <mirror>
              <id>alimaven</id>
              <name>aliyun maven</name>
              <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
              <mirrorOf>central</mirrorOf>        
            </mirror>
    
        </mirrors>
        ```

##2、配置IDEA 主题

- 建议在设置里的Plugins中下载一个插件主题  **Material Theme UI**
 
 ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_21-29-57.png)
 
 效果如下
 
 ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_21-30-13.png)


##3、Nexus搭建Maven私服
- 在开发过程中，有时候会使用到公司内部的一些开发包，显然把这些包放在外部是不合适的。另外，由于项目一直在开发中，这些内部的依赖可能也在不断的更新。可以通过搭建公司内部的Maven服务器，将第三方和内部的依赖统一管理，同时也可以节省网络带宽，当然前提是项目所需要的构件在私服中已经存在

  
  ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_21-35-32.png)

- 那么怎么在本地搭建私服呢
   
  -  1.下载对应的安装包
       https://www.sonatype.com/oss-thank-you-mac-tgz

      注意：目前的版本有2.X 和 3.X ，2.X的支持对Maven更友好一点，3.X的支持范围更广，支持ruby和docker。如果单纯的maven私服，建议使用2.x

  -  2.解压安装包，并进入对应的bin目录下启动nexus

       ```
       MAVEN_HOME=/Users/kevin/Documents/apache-maven-3.5.4
        NEXUS_HOME=/Users/kevin/Documents/nexus-3.12.1-01-mac/nexus-3.12.1-01
        
        PATH=$MAVEN_HOME/bin:$PATH:$NEXUS_HOME/bin
        export MAVEN_HOME NEXUS_HOME
        export PATH
       ```
       ```
       source .bash_profile 
       
       nexus
       Usage: /Users/kevin/Documents/nexus-3.12.1-01-mac/nexus-3.12.1-01/bin/nexus {start|stop|run|run-redirect|status|restart|force-reload}
       ```

      ```
      nexus start
      Starting nexus
      ```
    注意：3.X要求JDK的版本在1.8以上

  -  3.访问地址，3.x默认是127.0.0.1:8081 2.x默认是127.0.0.1:8081/nexus ,默认的登陆账户密码为admin/admin123
     
      修改端口或者密码，在etc下的nexus-default.properties
      
      ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_22-32-04.png)

  -  4.简单介绍一下Repository
     - Repository的type属性有：proxy，hosted，group三种。
     
       - proxy：即你可以设置代理，设置了代理之后，在你的nexus中找不到的依赖就会去配置的代理的地址中找
       - hosted：你可以上传你自己的项目到这里面
       - group：它可以包含前面两个，是一个聚合体。一般用来给客户一个访问nexus的统一地址。
       
   简单的说，就是你可以上传私有的项目到hosted，以及配置proxy以获取第三方的依赖（比如可以配置中央仓库的地址）。前面两个都弄好了之后，在通过group聚合给客户提供统一的访问地址
   
     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_22-35-59.png)
     
     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_22-36-44.png)
     
     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_22-40-58.png)
     
     如果想要上传到hostd仓库，必须把类型改为allow redeploy,不然传不上来。

 - 5.如何上传jar包
 
    - 1、 配置项目的部署仓库

       - 先设置settings.xml,这里主要是配置用户名和密码，注意这里的id要和respositoryId对应

        ```
        <servers>
           <server>
              <id>JavaPods</id>
              <username>admin</username>
              <password>admin123</password>
           </server>
        </servers>
       ```

   - 2、配置pom文件
       
         ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_23-50-44.png)
     
         
   - 3、配置deploy

         ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_23-48-42.png)
         
   - 4、打包上传私服
   
         ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_23-49-13.png)
         
   - 5、在 nexus中查看是否上传成功
      
         ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-06_23-44-50.png)
 
   

##4、MAVEN Scope使用

在Maven的依赖管理中，经常会用到依赖的scope设置。这里整理下各种scope的使用场景和说明，以及在使用中的实践心得。
 
- **scope的使用场景和说明**

  - 1.**compile**
编译范围，默认scope，在工程环境的classpath（编译环境）和打包（如果是WAR包，会包含在WAR包中）时候都有效。
 
  - 2.**provided**
容器或JDK已提供范围，表示该依赖包已经由目标容器（如tomcat）和JDK提供，只在编译的classpath中加载和使用，打包的时候不会包含在目标包中。最常见的是j2ee规范相关的servlet-api和jsp-api等jar包，一般由servlet容器提供，无需在打包到war包中，如果不配置为provided，把这些包打包到工程war包中，在tomcat6以上版本会出现冲突无法正常运行程序（版本不符的情况）。
 
  - 3.**runtime**
一般是运行和测试环境使用，编译时候不用加入classpath，打包时候会打包到目标包中。一般是通过动态加载或接口反射加载的情况比较多。也就是说程序只使用了接口，具体的时候可能有多个，运行时通过配置文件或jar包扫描动态加载的情况。典型的包括：JDBC驱动 mysql-connector-jdbc等。
 
  - 4.**test**
测试范围，一般是单元测试场景使用，在编译环境加入classpath，但打包时不会加入，如junit等。
 
  - 5.**system**
系统范围，与provided类似，只是标记为该scope的依赖包需要明确指定基于文件系统的jar包路径。因为需要通过systemPath指定本地jar文件路径，所以该scope是不推荐的。如果是基于组织的，一般会建立本地镜像，会把本地的或组织的基础组件加入本地镜像管理，避过使用该scope的情况。
 
  -  **实践：**
provided是没有传递性的，也就是说，如果你依赖的某个jar包，它的某个jar的范围是provided，那么该jar不会在你的工程中依靠jar依赖传递加入到你的工程中。
provided具有继承性，上面的情况，如果需要统一配置一个组织的通用的provided依赖，可以使用parent，然后在所有工程中继承。

##5、通过mvn dependency:tree 查看依赖树,解决依赖jar冲突问题

```
mvn dependency:tree 
```

 使用mvn dependency:tree --> tree.txt命令导出依赖树到txt文件，便于查看。

 最短路径法则



