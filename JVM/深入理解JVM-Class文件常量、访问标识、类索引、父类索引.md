##深入理解JVM-Class文件常量、访问标识、类索引、父类索引

- Java字节码类文件（.class）是Java编译器编译Java源文件（.java）产生的“目标文件”。它是一种8位字节的二进制流文件， 各个数据项按顺序紧密的从前向后排列， 相邻的项之间没有间隙， 这样可以使得class文件非常紧凑， 体积轻巧， 可以被JVM快速的加载至内存， 并且占据较少的内存空间（方便于网络的传输）。

- Java源文件在被Java编译器编译之后， 每个类（或者接口）都单独占据一个class文件， 并且类中的所有信息都会在class文件中有相应的描述， 由于class文件很灵活， 它甚至比Java源文件有着更强的描述能力。

  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_11-42-54.png)

- class文件中的信息是一项一项排列的， 每项数据都有它的固定长度， 有的占一个字节， 有的占两个字节， 还有的占四个字节或8个字节， 数据项的不同长度分别用u1, u2, u4, u8表示， 分别表示一种数据项在class文件中占据一个字节， 两个字节， 4个字节和8个字节。 可以把u1, u2, u3, u4看做class文件数据项的“类型” 。Class文件格式采用一种类似C语言结构体的伪结构来存储数据，这种伪结构中只有两种数据类型：无符号数和表。

- 1.1.无符号数

     无符号数属于基本的数据类型，根据这些值长度的不同分为：u1、u2、u4、u8，分别代表1字节的无符号数、2字节的无符号数、4字节的无符号数、8字节的无符号数。无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。
     
- 1.2.表

    表示由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯地以"_info结尾"。表用于描述有层次关系的复合结构的数据，整个Class文件本质上就是一张表，它由表6-1所示的数据项构成。

   一个典型的class文件分为：MagicNumber，Version，Constant_pool，Access_flag，This_class，Super_class，Interfaces，Fields，Methods 和Attributes这十个部分，用一个数据结构可以表示如下：

  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_13-46-26.png)


##2、Class文件的构成

- 2.1.魔数
   - class文件的头4个字节称为魔数，它的唯一作用是确定这个文件能否为一个能被虚拟机接受的Class文件。魔数的作用就相当于文件后缀名，只不过后缀名容易被修改，不安全，因此在class文件中标示文件类型比较合适。class文件的魔数是用16进制表示的“CAFEBABE”。
     ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_13-56-35.png)
- 2.2.版本号
   - 紧接着魔数的四个字节是class文件的此版本号和主版本号。

   - 随着Java的发展， class文件的格式也会做相应的变动。 版本号标志着class文件在什么时候， 加入或改变了哪些特性。 举例来说， 不同版本的javac编译器编译的class文件， 版本号可能不同， 而不同版本的JVM能识别的class文件的版本号也可能不同， 一般情况下， 高版本的JVM能识别低版本的javac编译器编译的class文件， 而低版本的JVM不能识别高版本的javac编译器编译的class文件。 如果使用低版本的JVM执行高版本的class文件，JVM会抛出java.lang.UnsupportedClassVersionError 。


     ```
     JDK 1.8 = 52 
JDK 1.7 = 51 
JDK 1.6 =50 
JDK 1.5 = 49 
JDK 1.4 = 48 
JDK 1.3 = 47 
JDK 1.2 = 46 
JDK 1.1 = 45
     ```
     ![](http://ovsiiuil2.bkt.clouddn.com/1111111.jpeg)    16进制转化为10进制是51,说明编译使用的是JDK 1.7 。
     

- 2.3. constant_pool[]常量池

   - Java虚拟机指令执行时依赖常量池（constant_pool）表中的符号信息。

   -  常量池大小需要固定，所以需要两个字节表示。00 22 ,10进制为34.表示有34个常量。从1到33

   - 0a表示 10 ，根据tag有效的类型和对应的取值。表示 `CONSTANT_MethodType_info` .  此时因为0a是第一个常量池位置
   - 00 06 表示符号引用。十进制表示6 ，表示指向第6个位置。
   - 00 14 十进制表示20，表示指向第20个位置。
   - 09 十进制表示9，`CONSTANT_Fieldref_info`字段的符号引用。
    - 依次类推
    
      ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-05-07.png)
   
   - 所有的常量池项都具有如下通用格式：
   
      ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_15-13-21.png)
      
      info[]项的内容tag由的类型所决定。tag有效的类型和对应的取值在下表列出
      
      ![](http://ovsiiuil2.bkt.clouddn.com/constant_pool.PNG)

- 3.0 CONSTANT_Class_info结构

    -  表示类或接口
    
     ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_15-25-59.png)
     
     name_index必须是对常量池的一个有效索引
     
- 3.1 CONSTANT_Fieldref_info， CONSTANT_Methodref_info和CONSTANT_InterfaceMethodref_info结构

    -  字段：
    
      ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_15-28-16.png)
      
    -  方法：
      
       ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_15-29-21.png)
       
    -  接口方法：
         
      ![](http://ovsiiuil2.bkt.clouddn.com/31111.png)
      
    - class_index必须是对常量池的有效索引,常量池在该索引处的项必须是CONSTANT_Class_info结构，表示一个类或接口，当前字段或方法是这个类或接口的成员。
    
    - CONSTANT_Methodref_info结构的class_index项的类型必须是类（不能是接口）。CONSTANT_InterfaceMethodref_info结构的class_index项的类型必须是接口（不能是类）。CONSTANT_Fieldref_info结构的class_index项的类型既可以是类也可以是接口。
    - name_and_type_index必须是对常量池的有效索引，表示当前字段或方法的名字和描述符。
    - 在一个CONSTANT_Fieldref_info结构中，给定的描述符必须是字段描述符。而CONSTANT_Methodref_info和CONSTANT_InterfaceMethodref_info中给定的描述符必须是方法描述符。

- 3.2 CONSTANT_String_info结构     

     - 用来表示String的结构
     
       ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-13-53.png)
       
       string_index必须是对常量池的有效索引，常量池在该索引处的项必须是CONSTANT_Utf8_info 结构，表示一组Unicode码点序列，这组Unicode码点序列最终会被初始化为一个String对象。

- 3.4CONSTANT_Integer_info和CONSTANT_Float_info结构

  - 表示4字节（int和float）的数值常量：
  
     ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-15-53.png)

- 3.5CONSTANT_Long_info和CONSTANT_Double_info结构

   - 表示8字节（long和double）的数值常量

     ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-17-21.png)
     
- 3.6 CONSTANT_NameAndType_info结构

   - 表示字段或方法，但是和前面介绍的3个结构不同，CONSTANT_NameAndType_info结构没有标识出它所属的类或接口
    ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-19-14.png)
    
   - name_index项的值必须是对常量池的有效索引，常量池在该索引处的项必须是CONSTANT_Utf8_info结构，这个结构要么表示特殊的方法名，要么表示一个有效的字段或方法的非限定名（Unqualified Name）。
   - descriptor_index项的值必须是对常量池的有效索引，常量池在该索引处的项必须是CONSTANT_Utf8_info结构，这个结构表示一个有效的字段描述符或方法描述符。
   
- 3.7 CONSTANT_Utf8_info结构
   
   - 用于表示字符串常量的值

       ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-22-10.png)
       
       CONSTANT_Utf8_info结构中的内容是以length属性确定长度的

- 3.8 CONSTANT_MethodHandle_info结构

  - 表示方法句柄

      ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-26-39.png)
      
      reference_kind项的值必须在1至9之间（包括1和9），它决定了方法句柄的类型。

   - 如果reference_kind项的值为1（REF_getField）、2（REF_getStatic）、3（REF_putField）或4（REF_putStatic），那么常量池在reference_index索引处的项必须是CONSTANT_Fieldref_info结构，表示由一个字段创建的方法句柄。
   - 如果reference_kind项的值是5（REF_invokeVirtual）、6（REF_invokeStatic）、7（REF_invokeSpecial）或8（REF_newInvokeSpecial），那么常量池在reference_index索引处的项必须是CONSTANT_Methodref_info结构，表示由类的方法或构造函数创建的方法句柄。
   - 如果reference_kind项的值是9（REF_invokeInterface），那么常量池在reference_index索引处的项必须是CONSTANT_InterfaceMethodref_info结构，表示由接口方法创建的方法句柄。
   - 如果reference_kind项的值是5（REF_invokeVirtual）、6（REF_invokeStatic）、7（REF_invokeSpecial）或9（REF_invokeInterface），那么方法句柄对应的方法不能为实例初始化（）方法或类初始化方法（）。
如果reference_kind项的值是8（REF_newInvokeSpecial），那么方法句柄对应的方法必须为实例初始化（）方法。


- 3.9 CONSTANT_MethodType_info结构

   - 表示方法类型
    
      ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-30-07.png)
      
- 3.10 CONSTANT_InvokeDynamic_info结构

   - 表示invokedynamic指令所使用到的引导方法（Bootstrap Method）、引导方法使用到动态调用名称（Dynamic Invocation Name）、参数和请求返回类型、以及可以选择性的附加被称为静态参数（Static Arguments）的常量序列。

      ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-31-24.png)
      
   -  bootstrap_method_attr_index项的值必须是对当前Class文件中引导方法表的bootstrap_methods[]数组的有效索引。
   - name_and_type_index项的值必须是对当前常量池的有效索引，常量池在该索引处的项必须是CONSTANT_NameAndType_info结构，表示方法名和方法描述符。

##4. access_flags:访问标志

- 访问标志，access_flags是一种掩码标志，用于表示某个类或者接口的访问权限及基础属性。access_flags的取值范围和相应含义见下表。
   
   - 多个修饰符使用异或的方式组合二进制
   ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-45-11.png)
   
    ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-54-14.png)
    
    ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_16-53-01.png)

##5. this_class:类索引

- this_class的值必须是对constant_pool表中项目的一个有效索引值。

   - 是一个对constant_pool表中项目的一个有效索引值，表示指向常量池的第几个位置。

      ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-214_17-03-07.png)
      
      ![](http://ovsiiuil2.bkt.clouddn.com/3232323.png)


##6. super_class：父类索引

- 表示这个Class文件所定义的类的直接父类,如果Class文件的super_class的值为0，那这个Class文件只可能是定义的是java.lang.Object类，只有它是唯一没有父类的类
- 是一个对constant_pool表中项目的一个有效索引值，表示指向常量池的第几个位置。




