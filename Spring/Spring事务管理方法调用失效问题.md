## Spring事务管理
- Spring - 在同一个类中一个普通方法调用另一个有@Transcational注解的方法时，Spring事务管理还启作用吗？

 ![](https://www.icheesedu.com/images/qiniu/Xnip2018-08-22_21-17-41.png)
 
 
 ![](https://www.icheesedu.com/images/qiniu/Xnip2018-08-22_21-18-36.png)
 
 ![](https://www.icheesedu.com/images/qiniu/Xnip2018-08-22_21-19-17.png)
 
 答案是不会起作用的。此处的this指向目标对象，因此调用this.b()将不会执行b事务切面，即不会执行事务增强，因此b方法的事务定义@Transactional(propagation = Propagation.REQUIRES_NEW)将不会实施.
 
- 只要给目标类AServiceImpl的某个方法加上注解@Transactional，spring就会为目标类生成对应的代理类，以后调用AServiceImpl中的所有方法都会先走代理类（即使调用未加事务注解的方法a，也会走代理类），即在通过getBean("AServiceImpl")获得的业务类时，实际上得到的是一个代理类，假设这个类叫做AServiceImplProxy ，spring为AServiceImpl生成的代理类类似于如下代码：

  ```
  public class AServiceImplProxy implements AService{
 
       public void a() {
        //启动事务的代码
        //反射调用目标类的a方法
        //事务提交的代码
       }
     
       public void b() {
       //启动事务的代码
       //反射调用目标类的b方法
       //事务提交的代码
       }
}
  ```
 
- 使用AOP 代理后的方法调用执行流程：

![](https://www.icheesedu.com/images/qiniu/a7d8d493-e387-34e9-a637-8a4d8d438602.jpg)
 
![](https://www.icheesedu.com/images/qiniu/Xnip2018-08-22_21-22-03.png)

- 首先调用的是AOP代理对象而不是目标对象，首先执行事务切面，事务切面内部通过TransactionInterceptor环绕增强进行事务的增强，即进入目标方法之前开启事务，退出目标方法时提交/回滚事务。

- Spring在处理@Tranasctional注解时，会“proxy”掉当前的类。
- 如果A和B两个类都有@Transactional时，实际上运行的是A的代理类A‘, A，B的代理类B'， B四个类的instance。
- 一个外部服务调用A，实际上是 外部-->A'-->A-->B'-->B这样执行的。而抛出异常的代码实际上是在B‘做的。
- 但是如果是同一个类内部方法直接调用的话，那么就是简单的方法直接调用，即 外部-->A'-->A方法1-->A方法2。 A方法1不会找到A'去调用。于是，“传播”的规则不会生效。
- 这个问题的答案尽管简单，但是问题在于本来是希望注解可以完全隐藏掉内部的复杂性的，但是实际上往往出现“盖不住露馅“的情况。有的答友会强调“边界”啦，“概念“啦，这是Java这套体系为了维护一种模型，为了自洽，不得不搞出这样的抽象规则——这并不是什么好事。
- 其实，如果用SQL直接写就是 “开始事务"--->“处理数据”--->“提交/回滚“这3步，简单直接。至少我暂时没看出来搞出很多@Transactional+传播属性有任何可以简化设计，提高代码可维护性上的好处。
- 所以对于简单事务，不妨统一做一层Service，在入口加@Transactional，然后把数据源（或者等价的jdbcTemplate/mapper）往下层传。而下层就不需要考虑事务这件事了。
 
## 2、推荐阅读
- Spring事务处理 https://www.cnblogs.com/mxmbk/p/5341258.html
- spring事务使用心得 https://www.cnblogs.com/zhuanghuang/p/5515830.html
- Spring事务管理的一些注意点 https://blog.csdn.net/liujinye/article/details/78286721
- Spring入门第4天--Spring事物管理  https://blog.csdn.net/lutianfeiml/article/details/51756569


