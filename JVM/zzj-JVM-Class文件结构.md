# Class文件结构

##  语言无关性

jvm运行的是 `.class文件` ，class文件可以从多种高级语言（ruby, groovy等）编译而来，所以jvm可以说是语言无关性的。

##  文件结构

![jvm-文件结构字段](E:\OneDriver\OneDrive\Pictures\java\jvm-文件结构字段.png)

[![jvm.png](https://i.postimg.cc/Rhx0LwCh/jvm.png)](https://postimg.cc/zbp5JbgY)

**其中`ux` 代表了 x 个字节**



## ##  魔数

- magic u4

  - 0xCAFEBABE

  class文件都是以此开头的。



## ##  版本

![jvm-版本](E:\OneDriver\OneDrive\Pictures\java\jvm-版本.png)

[![jvm.png](https://i.postimg.cc/tCCbQBcy/jvm.png)](https://postimg.cc/DmDH1PFY)

向下兼容，就是依赖此版本。



## ##  常量池

![JVM-文件结构常量池说明](E:\OneDriver\OneDrive\Pictures\java\JVM-文件结构常量池说明.png)

[![JVM.png](https://i.postimg.cc/k4n16pKg/JVM.png)](https://postimg.cc/06BdhZVT)



## ##  访问符

![JVM-访问标识符](E:\OneDriver\OneDrive\Pictures\java\JVM-访问标识符.png)

[![JVM.png](https://i.postimg.cc/XYLQMgTM/JVM.png)](https://postimg.cc/687dnCWz)



## ##  类、超类、接口

- this_class u2
- super_class u2
- interface_count u2
- interfaces
  - 

## ##  字段

- field_count
- fields
- field
- access_flags

- name_index u2
  - 常量池引用，表示字段的名字
- descriptor_index
  - 表示字段的类型

[![jvm-class.png](https://i.postimg.cc/pXYB8ZV9/jvm-class.png)](https://postimg.cc/BXbDf2Ks)



## ##  方法

- methods_count
  - 方法数量
- methods
  - methods_count 个method_info
- access flag
- name_index u2
  - 方法名字，常量池UTF-8 索引
- descriptor_index u2
  - 描述符，用于表达方法的参数和返回值



## ##  属性



