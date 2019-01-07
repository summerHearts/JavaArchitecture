##1、注射式攻击的原理

- SQL注射能使攻击者绕过认证机制，完全控制远程服务器上的数据库。SQL是结构化查询语言的简称，它是访问数据库的事实标准。目前，大多数Web应用都使用SQL数据库来存放应用程序的数据。几乎所有的Web应用在后台都使用某种SQL数据库。跟大多数语言一样，SQL语法允许数据库命令和用户数据混杂在一起的。如果开发人员不细心的话，用户数据就有可能被解释成命令，这样的话，远程用户就不仅能向Web应用输入数据，而且还可以在数据库上执行任意命令了

  ```
public static void main(String[] args) throws Exception {
        Scanner sc = new Scanner(System.in);
        System.out.println("账号：");
        String uid = sc.nextLine();
        System.out.println("密码：");
        String pwd = sc.nextLine();
        
        Class.forName("com.mysql.jdbc.Driver");
        
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb?characterEncoding=GBK","root","");
        
        Statement state = conn.createStatement();
        
        String sql = "select * from users where user ='"+uid+"' and password ='"+pwd+"' ";
        ResultSet rs = state.executeQuery(sql);
        boolean ok = rs.next();
        if(ok){
            System.out.println("欢迎"+rs.getString(3)+"回来");
        }
        else{
            System.out.println("您输入的账号密码错误");
        }
        
        conn.close();
  ```
  我们正常输入账号密码是运行正确的，但是当我们账号输入：kjaskj' or 1=1 #  密码输入：klkjl;  就会出现以下的结果：
  
  ```
  欢迎张三回来
  ```
这里，关键在  账号里面的那个单引号 “ ‘ ’”和后面 or 1=1以及#号（我们这里用的是mysql，oracle后面用 --）。这样查询语句就变成了：
 
  ```
select * from users where user ='kjaskj'or1=1#' and password ='"+pwd+"' 
  ```

 该双划符号#告诉SQL解析器，右边的东西全部是注释，所以不必理会。这样，查询字符串相当于：

 select * from users where user =''OR1=1.   这样输出的就是ture。  就能不用账号密码直接进入。


##2、如何防御？

- 1）对一些特殊字符作转义；

```
/**正则表达式**/
    private static String reg = "(?:')|(?:--)|(/\\*(?:.|[\\n\\r])*?\\*/)|"
                                + "(\\b(select|update|union|and|or|delete|insert|trancate|char|into|substr|ascii|declare|exec|count|master|into|drop|execute)\\b)";
 
// \\b  表示 限定单词边界  比如  select 不通过   1select则是可以的
    private static Pattern sqlPattern = Pattern.compile(reg, Pattern.CASE_INSENSITIVE);
 
 private boolean isValid(String str)
    {
        if (sqlPattern.matcher(str).find())
        {
            logger.error("未能通过过滤器：str=" + str);
            return false;
        }
        return true;
    }  
```

- 2）拼写SQL的时候不要拼接参数，要使用绑定变量的形式；

- 3）使用mybatis框架的话，占位符要使用#{}，不要使用`${}，#{}默认会把所有参数用引号包起来，${}`不会；




