# -Java-package-
 Java中 package 分析


由于大家对package的使用存在太多困惑，我在这里将自己对于package的使用的领悟进行一点总结：

　　package中所存放的文件

　　所有文件，不过一般分一下就分这三种

　　1，java程序源文件，扩展名为.java。

　　2，编译好的java类文件，扩展名为.class。

　　3，其他文件，其他任何文件,也称为resource

　　例如图片文件，xml文件，mp3文件，avi文件，文本文件……

　　package是什么

　　package好比java用来组织文件的一种虚拟文件系统。package把源代码.java文件，.class文件和其他文件有条理的进行一个组织，以供java来使用。package是将文件组织在一颗类似unix，linux文件系统的树结构里面，它有一个根"/"，然后从根开始有目录和文件，目录中也还有文件和目录。

　　package怎么实现的呢？

　　源代码的要求最严格，而一旦源代码自己声明了在哪个package路径之下，class也就有了自己在哪个package下面的信息，就是那句程序开头的"package xx.xx.xx"。有人问，为什么要有这个信息，直接放目录结构里不就好了么？是啊，直接放目录中确实可以找到.class和.java，但是如果我要输出这个.class是属于哪个package的，该怎么办？所以我们需要在.class里面留一个package的信息。如果我们要区分同样名称为A.class的类怎么办？所以我们需要在.class里面留一个package的信息。

　　.java文件是一个独立的编译单元，类似c++里面的cpp文件，但是它不需要.h文件，只要.java就足够了，一个.java文件里面可以包含一个public的类，若干package类(package类特征是没有任何访问控制修饰)，还有内隐类的话，则还可以包含若干protected和private的类。每个类，都会在编译的时候生成一个独立的.class文件，所以.java和.class不是一对一，而是一对多的关系，不过.java和public的类是一对一的。所有这些.class，都由这个.java开头的package语句来确定自己在package中的位置。

　　package xx.bb.aa;

　　说明这个.java编译单元中的所有类都放到xx.bb.aa这个package里面。而对应的，必须把这个.java文件放在xx目录下bb目录下的aa目录里面。如果一个.java文件没有任何package语句，那么这个.java里面的所有类都在package的"/"下面，也称之为default package。可以看出你一般从任何java教科书上写的第一个hello world程序的那个类是在defaultpackage里面的。有了package语句，情况就复杂一点了。这个编译单元.java必须放在package名对应的目录之下。而生成的class文件也要放在对应的目录结构之下才能正常运作。

　　例如：

    /* A.java */ 
　　package aaa.bbb.ccc; 
　　public class A{ 
　　B b=new B(); 
　　} 
　　/* B.java*/ 
　　package aaa.bbb.ccc; 
　　public class B{}

　　编译时候怎么填参数呢？我根据package+文件名的格式来写，

　　javac aaa.bbb.ccc.A.java

　　漂亮吧？可惜不工作。非要使用合法的路径名才行：

　　javac aaa/bbb/ccc/A.java

　　但是你发现生成的class丢失了目录结构，直接出现在当前目录下……

　　最好的方式是

　　javac -d bin aaa/bbb/ccc/A.java

　　这样就会在当前目录的bin目录下看到完整的目录结构以及放置妥当的class文件。

　　package与classpath不得不说的事

　　对于java来讲，所有需要的程序和资源都要以package的形式来组织和读取。

　　那么classpath又是什么呢？

　　所有放到设定到classpath里面的东西就是package所包纳的资源。classpath的写法如同path，只是里面可以写的一般只有zip文件、jar文件和目录。多个元素之间用当前系统路径分隔符间隔开了，linux上分隔符号是":"，windows上是";"。classpath在java里面是被一个叫做classloader的东西所使用的，classloader顾名思义，就是load class用的，但它也可以load其它在package里面的东西。现在的java里面classloader是有阶层关系的，一般我们所常接触到的CLASSPATH环境变量，javac,java的-cp,-classpath参数所给的classpath信息是被appclassloader所使用的。而appclassloader其实是第三层的classloader，最高层的classloader叫做bootstrap classloader，它不是java写的classloader，而是c++写成的，第二层叫做extclassloader，默认包纳是jre/lib/ext里面的classes目录和所有jar文件作为内容。第三层才是我们命令行参数，或者不用命令行参数，用系统环境变量指定classpath的使用者app classloader，这是最基本的java se。如果是java ee，有了服务器，容器，还有更多层的classloader，他们在app classloader的更下面，例如tomcat的某web应用程序的web-inf/lib中的jar,zip和classes目录，是app之下好几层的classloader使用的。

　　你可以建立自己的classloader，都在app classloader之下，实际上tomcat本身也是这样建立classloader的。分层的目的是为了安全，试想你加入搞了一个classloader，从网络上读取class，而在里面写上格式化硬盘的代码，人家一读运行，那不就挂了，所以分层之后，首先从最高层读，没有再往下找，找到就不着了。一般java所必须的rt.jar里面的若干class，是在bootstrap classloader启动的时候读入的，而jmf使用的几个jar，是在ext classloader里面读入的。也就是说，读入这些class的时候，我们的appclassloader还在娘胎里呢，所以你在CLASS PATH中指定rt.jar是完全愚蠢多余的。java绝对不会到这里找rt.jar，而bootstrapclassloader如果你不特别要修改，它是常量，不需要你care。

　　import干吗用的？

　　import只是一种让你偷点懒少打字的方法，绝对不会影响你的classpath，这点你要好好记住，没有非用import不可的理由，用了import也不会起到类似c里面嵌入某文件内容的效果，它只是一种省事的办法。不在classpath中的class，任你再import也无济于事。

　　如果你不用import，你用ArrayList这个类，就需要写

　　java.util.ArrayList。

　　而用了import java.util.ArrayList;的话

　　以后代码中写ArrayList就可以了，省事。import可以使用通配符*，*代表某package下所有的class，不包括子目录。

　　import java.awt.*

　　不等于

　　import java.awt.*

　　import java.awt.event.*

　　如果你要简写java.awt.event下和java.awt下的类，你不能偷懒，两个都要import。
