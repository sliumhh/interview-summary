![image-20200811091151928](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200811091151928.png)

![image-20200811091232865](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200811091232865.png)

![image-20200811091246796](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200811091246796.png)



## 类的加载过程

![image-20200811091435781](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200811091435781.png)

1. 字节码文件其实是物理磁盘上的一个文件,类加载器负责字节码文件的加载,然而类加载器其实是通过类的全限定名来控制二进制字节流完成的加载.将字节码文件读入JVM中,这个字节码文件被成为DNA元数据模板,它按照虚拟机所需的格式存储在方法区之中.

2. 然后在内存中对应创建一个java.lang.Class对象-->这个对象会被放入字节码信息中,这个Class对象,就对应加载那个字节码信息,这个对象将被作为程序访问方法区中的这个类的各种数据的外部接口。(Java中万事万物皆对象,通过对象来访问具体的一些功能,这个Class对象就充当这个外部接口的角色)

## 类加载器

1. JVM支持两种类型的类加载器，分别为引导类加载器(Bootstrap ClassLoader)和自定义类加载器(User-Defined ClassLoader)。
   -->JVM规范中提到的.
   规范中同时提到,凡是直接或者间接的继承自ClassLoader的类加载器都划分为自定义类加载器。

2. 扩展类加载器(Extension ClassLoader) -->对应ExtClassLoader类,是Launcher的内部类
   系统类加载器(System ClassLoader) -->对应AppClassLoader,是Launcher的内部类
   用户自己定义的加载器(User DefinedClassLoader) -->继承ClassLoader
   都间接的继承自ClassLoader -->是一个抽象类,
   所以他们属于自定义类加载器.

3. 注意1:
   Bootstrap ClassLoader是用c和c++来编写的,引导类加载器嵌入JVM中,是JVM的一部分.它加载核心类库中内容,所以Extension ClassLoader,System ClassLoader都和核心类库的,所以这两个加载器也是靠Bootstrap ClassLoader来加载的.
   Extension ClassLoader,System ClassLoader都是用java编写的,都是是Launcher的内部类 。

   3.1 引导类加载器:sun.misc.Launcher.getBootstrapClassPath()，加载核心类库中的jar包

   3.2 扩展类加载器：加载lib/ext下的

   3.3 自定义的类：

4. 注意2:
   这些加载器有等级关系:(不是子类父类的关系,是一个等级的关系,好比阶级社会人与人之间的关系)
   Bootstrap ClassLoader>Extension ClassLoader>System ClassLoader>用户自己定义的加载器
   (好比阶级社会,有的类你根本没资格被Bootstrap ClassLoader加载,惨兮兮) 

## 双亲委派机制



1. ### 工作原理:

   1)如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行;|
   2)如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归,请求最终将到达项层的启动类加载器;
   3)如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派机制。

2. ### 那么为什么要采用双亲委派机制这样做呢?

   因为如果随意的就去执行我自己写的String了,那么你说你的一个很完美的项目,我要是恶意攻击你给你发送一个String类,那么你的项目岂不是直接就崩溃了?这也是一种保护啊!
   (1)避免核心API被篡改 (2)避免类的重复加载

## 链接

1. 经过加载阶段之后,已经可以在内存中生成Class类的实例对象了.
   接下来进入到链接阶段:

   ### 1 验证(Verify) :

   **目的在于验证字节码文件是否符合当前JVM要求，保证被加载类的正确性，不会危害虚拟机自身安全。**

   **文件格式验证，元数据验证，字节码验证，符号引用验证。(这些均在编译期已生成在字节码文件中,所以你加载入内存的字节码文件也有)**

   因为这个字节码都是一些二进制,所以其实完全可以由人为的手动敲一个这样的文件,或者如果受到了拦截遭到了恶意篡改也是有可能的,所以为了避免这样的事情,验证一下,只要验证不通过,就会报Error的.

2. ### 准备(Prepare) :

**为类变量(static变量)分配内存并且设置该类变量的默认初始值，即零值。**
就比如下面这个代码中的类变量age,在准备阶段它会被放入字节码文件中的常量池中(常量池属于字节码文件的一部分),并且初始值为0不是10, 而是在准备阶段的下一个阶段初始化的时候才会被赋值为10.
数据的类型不一样,那么它的初始值也是不同的:
(byte 0 ,short0 ,int0, long 0 ,float 0.0F, double 0.0, char '\u0000', boolean false ,引用类型null)

```java
public class Test{
        static int age = 10;//类变量age
        public static void main(String[] args){	
                System.out.println(age);	
        }
}
```

**如果age被final修饰了,那么直接初始化值就是10,因为这个值后续不会被更改了**

```java
public class Test{
        final static int age = 10;//类变量age
        public static void main(String[] args){	
                System.out.println(age);	
        }
}
```

3. ### 解析(Resolve) :

   **将常量池内的符号引用转换为直接引用的过程。**
   事实上，解析操作往往会伴随着JVM在执行完初始化之后再执行。
   符号引用就是一组符号来描述所引用的目标。符号引用的字面量形式明确定义在《java虚拟机规范》的Class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。
   解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。对应常量池中的CONSTANT Class info、 CONSTANT Fieldref info、 CONSTANT Methodref info等。

5. ## 初始化:

   初始化阶段就是执行类构造器方法<clinit>()的过程。(cliinit--class-init)此方法不需定义，是javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来。
   类构造器方法中指令按语句在源文件中出现的顺序执行。
   ***注意:类向内存中加载,只能被加载一次,<clinit>()也只能被调用一次,如果两个线程同时要加载这个类的话,只能有一个线程可以加载,同时加载过程中加锁,另一个线程没有办法加载.***

   6.栈的内部结构：

   栈中最基本的单位就是栈帧,那么栈帧内部由五部分:
   局部变量表(Local variables)
   操作数栈(operand Stack) (或称做:表达式栈)
   动态链接(Dynamic Linking) (或称做:指向运行时常量池的方法引用)
   方法返回地址(Return Address) (或称做:方法正常退出或者异常退出的定义)一些附加信息

   ![image-20200811102118863](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200811102118863.png)