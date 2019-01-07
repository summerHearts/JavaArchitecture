##Spring IOC

- 1、IOC:
  - 控制反转，是一种设计思想，意味着将设计好的对象交给Ioc容器控制，而不是传统的在对象的内部直接控制。
  - IOC容器把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象。
  - Spring Ioc容器管理的对象：Bean
- 2、Bean的作用域
 - singleton：单例，指一个Bean 在容器中只存在一份，类似于设计模式中的单例模式

 - prototype：每次请求都会创建新的实例，destroy方法不生效，类似工厂模式

 - request：每次的http请求创建一个实例，且仅在当前的request内有效

 - session：同上，但是当前session有效，而session的作用域相对较大，

 - golbal session：基于portlet的web中有效，如果是web中，同session，

 - portlet ： 基于java的一个web组件，由portlet容器管理，并且由容器处理请求，生产动态内容

 五种作用域中，request、session和global session三种作用域仅在基于web的应用中使用（不必关心你所采用的是什么web应用框架），只能用在基于web的Spring ApplicationContext环境。

 (1)当一个bean的作用域为Singleton，那么Spring IoC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。Singleton是单例类型，就是在创建起容器时就同时自动创建了一个bean的对象，不管你是否使用，他都存在了，每次获取到的对象都是同一个对象。注意，Singleton作用域是Spring中的缺省作用域。要在XML中将bean定义成singleton，可以这样配置：
 
  ```
  <bean id="ServiceImpl" class="com.wangpu.service.ServiceImpl"   scope="singleton">
  ```
  
  (2)当一个bean的作用域为Prototype，表示一个bean定义对应多个对象实例。Prototype作用域的bean会导致在每次对该bean请求（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）时都会创建一个新的bean实例。Prototype是原型类型，它在我们创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象。根据经验，对有状态的bean应该使用prototype作用域，而对无状态的bean则应该使用singleton作用域。在XML中将bean定义成prototype，可以这样配置：
  
  ```
  <bean id="account" class="com.wangpu.DefaultAccount" scope="prototype"/>  
 或者
<bean id="account" class="com.wangpu.DefaultAccount" singleton="false"/> 
  ```
  
  (3)当一个bean的作用域为Request，表示在一次HTTP请求中，一个bean定义对应一个实例；即每个HTTP请求都会有各自的bean实例，它们依据某个bean定义创建而成。该作用域仅在基于web的Spring ApplicationContext情形下有效。考虑下面bean定义：
  
  ```
  <bean id="loginAction" class=com.wangpu.LoginAction" scope="request"/>
  ```
  (4)当一个bean的作用域为Session，表示在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。考虑下面bean定义：
  
  ```
  <bean id="userPreferences" class="com.wangpu.UserPreferences" scope="session"/>
  ```
　针对某个HTTP Session，Spring容器会根据userPreferences bean定义创建一个全新的userPreferences bean实例，且该userPreferences bean仅在当前HTTP Session内有效。与request作用域一样，可以根据需要放心的更改所创建实例的内部状态，而别的HTTP Session中根据userPreferences创建的实例，将不会看到这些特定于某个HTTP Session的状态变化。当HTTP Session最终被废弃的时候，在该HTTP Session作用域内的bean也会被废弃掉。
 
 (5)当一个bean的作用域为Global Session，表示在一个全局的HTTP Session中，一个bean定义对应一个实例。典型情况下，仅在使用portlet context的时候有效。该作用域仅在基于web的Spring ApplicationContext情形下有效。考虑下面bean定义：
 
 ```
 <bean id="user" class="com.wangpu.Preferences "scope="globalSession"/>
 ```
global session作用域类似于标准的HTTP Session作用域，不过仅仅在基于portlet的web应用中才有意义。Portlet规范定义了全局Session的概念，它被所有构成某个portlet web应用的各种不同的portlet所共享。在global session作用域中定义的bean被限定于全局portlet Session的生命周期范围内。

