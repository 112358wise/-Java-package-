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
　　贴二——package从实践到理论：


package就是catalog，所谓目录册。
什么是目录册呢？比较抽象的说，就是让你快速定位一个东西的册子。
比如电话簿，比如字典，比如搜索引擎。
java中，主要是class文件，当然也可以是任何别的文件，就是通过package来组织的。
package有点类似一个虚拟的文件系统，可以将所有的东西组织在树形结构之中。
这有一个根"/"，也就是我们什么package语句都不写的情况下class文件所应该处的位置。
直接赤裸裸放在classpath中的东西都是放在"/"下面，然后如果有package语句，则这个
class文件或者java源代码文件都要放在和package语句中包名对应的目录结构之中。


1，什么都别说，先跟着我来做一把
我们先找一个目录，比如C:\myjob
然后我们建立两个目录，一个叫做src，一个叫做bin
C:\myjob>md src
C:\myjob>md bin
C:\myjob>dir
驱动器 C 中的卷是 LIGHTNING
卷的序列号是 3DD1-83D9
C:\myjob 的目录
2005-12-25  14:33    <DIR>          .
2005-12-25  14:33    <DIR>          ..
2005-12-25  14:34    <DIR>          src
2005-12-25  14:34    <DIR>          bin
               0 个文件              0 字节
               4 个目录    305,123,328 可用字节
C:\myjob>
然后我们在src目录中去写程序
C:\myjob>cd src
C:\myjob\src>
我们写这么4个java文件
////A.java
package com.lightning;
public class A{
{System.out.println("com.lightning.A");}
}
////B.java
package com.lightning;
public class B{
{System.out.println("com.lightning.B");}
}
////C.java
package com;
public class C{
{System.out.println("com.C");}
}

////Test.java
package net.test;
import com.lightning.*;
import com.*;
public class Test{
public static void main(String[] args)
{
        new A();new B();new C();
        System.out.println("net.test.Test");
}
}
写好之后就是这样

C:\myjob\src>dir
驱动器 C 中的卷是 LIGHTNING
卷的序列号是 3DD1-83D9

C:\myjob\src 的目录

2005-12-25  14:34    <DIR>          .
2005-12-25  14:34    <DIR>          ..
2005-12-25  14:39                86 A.java
2005-12-25  14:40                86 B.java
2005-12-25  14:42               194 Test.java
2005-12-25  14:43                68 C.java
               4 个文件            434 字节
               2 个目录    305,106,944 可用字节

然后我们建立com目录，com\lightning\目录，net\test\目录
C:\myjob\src>md com
C:\myjob\src>md com\lightning
C:\myjob\src>md net\test\
我们将Test.java放入net\test\中去
将A.java，B.java放入com\lightning\中去
将C.java放入com\中去
C:\myjob\src>move Test.java net\test\
C:\myjob\src>move A.java com\lightning\
C:\myjob\src>move B.java com\lightning\
C:\myjob\src>move C.java com\

然后我们在c:\myjobs中发令：
C:\myjob\src>cd ..
C:\myjob>javac -sourcepath src -d bin src\net\test\Test.java
然后我们看看bin目录中多了什么
C:\myjob>dir bin /s
驱动器 C 中的卷是 LIGHTNING
卷的序列号是 3DD1-83D9

C:\myjob\bin 的目录

2005-12-25  14:34    <DIR>          .
2005-12-25  14:34    <DIR>          ..
2005-12-25  14:46    <DIR>          net
2005-12-25  14:46    <DIR>          com
               0 个文件              0 字节

C:\myjob\bin\net 的目录

2005-12-25  14:46    <DIR>          .
2005-12-25  14:46    <DIR>          ..
2005-12-25  14:46    <DIR>          test
               0 个文件              0 字节

C:\myjob\bin\net\test 的目录

2005-12-25  14:46    <DIR>          .
2005-12-25  14:46    <DIR>          ..
2005-12-25  14:46               520 Test.class
               1 个文件            520 字节

C:\myjob\bin\com 的目录

2005-12-25  14:46    <DIR>          .
2005-12-25  14:46    <DIR>          ..
2005-12-25  14:46    <DIR>          lightning
2005-12-25  14:46               338 C.class
               1 个文件            338 字节

C:\myjob\bin\com\lightning 的目录

2005-12-25  14:46    <DIR>          .
2005-12-25  14:46    <DIR>          ..
2005-12-25  14:46               354 A.class
2005-12-25  14:46               354 B.class
               2 个文件            708 字节

     所列文件总数:
               4 个文件          1,566 字节
              14 个目录    305,057,792 可用字节

然后我们执行，同样在c:\myjobs\下发令
C:\myjob>java -cp bin net.test.Test
com.lightning.A
com.lightning.B
com.C
net.test.Test

2，从实践到理论

刚才我用一个非常简单但是非常完整的例子给大家演示了java的package机制。
为什么以前脑海里面那么简单的javac会搞得这么复杂呢？

实际上它本就这么复杂，
并不是一个.java，一行javac一个当前目录中的class这么简单。

首先我要打破一些东西，然后才好建立一些东西。
javac并非只是给一个.java编译一个class的。javac自带有make机制，也就是说，如果在
javac的参数中java文件使用到的任何类，javac首先会去找寻这个类的class文件存在与否
，如果不存在，就会在sourcepath中找寻源代码文件来编译。

什么是sourcepath呢？sourcepath是javac的一个参数，如果你不加指定，那么sourcepath
就是classpath。

比如我们装好jdk之后，我说过不要设定classpath环境变量，因为大部分人一旦设定了
classpath，不是多此一举把什么dt.jar放进去。（我可以好好打击你一下，告诉你一个可
悲的事实——jre永远不会从你这个classpath中去寻找dt.jar。你完全是徒劳的！）就是
把"."搞不见了，搞得是冷水一盆盆的往自己身上泼，脑袋一点点的涨大。

不要设！classpath没有你想象中那么普适和强大，它只是命令行的简化替代品。
你不设的话它就是"."。


回到sourcepath，sourcepath指的是你源代码树的存放地点。
为什么是源代码树？而不是一个目录的平板源代码呢？
请大家将原本脑海中C的编译过程完全砸掉！
java完全不同，java没有头文件，每个.java都是要放在源代码树中的。
那么这颗树是怎么组织的呢？
对了，就是package语句。
比如写了package com.lightning;
那么这个.java就必须放在源代码树根\的com\lighting\之下才行。

很多浮躁的初学者被default打包方式宠坏了。自我为中心，以为java就是一套库，自己写
的时候最多import进来就行了，代码从不打包，直接javac，直接java，多么方便。
孰不知自己写的这个.java也不过是java大平台的其中一个小单元而已。如果不打包，
我写一个Point，你写一个Point，甚至更有甚者敢于给自己的类起名叫String等等。
全部都在平板式的目录中，那jre该选哪一个？

一旦要使用package语句，就要使用代码树结构，当然，你要直接javac，也行。
不过javac出来的这个class要放在符合package结构的目录中才能发挥正常作用，否则就是
废物一坨。
而且，如果你这个.java还用到其它自己写的有package语句的.java，那这个方法就回天乏
术了。

按照sun的想象，应该是写好的.java放在符合package结构的目录中，package语句保证了
正确放置，如果目录位置和package语句中指示的不同，则会出错。

所以按照刚才我们的那种package写法，我们必然要将那几个.java文件放入相应的目录中
才能让javac顺利找到他们来make。

有人说javac可不可以像java那样 java aaa.bbb.c...java？
不可以
javac中的那个.java文件参数必须是一个文件系统的路径文件名形式。
然后如果用到其它的.java，javac会根据目前的sourcepath出发寻找目录结构中的那些
java文件。

当然了，既然打了包，在使用的时候。
要么写全名——包名.类名
或者使用import
不得不提的是，import就好比c++的using，它不负责做文件操作，它只是方便你写代码。

另外import语句中的*代表的是类名，不代表包名片断。
你import com.*;
编译器仍然找不到com.lightning中的任何类。
反之亦然。
就好象你告诉编译器，我这里面要用到姓诸葛的人。
那么姓诸的人当然编译器不会认为也包含在内。


所以，如果程序一旦写到一定规模。
就不得不使用ant来管理这些。
或者使用IDE，否则jdk就真的变成了java developer killer。

但是对于初学者，在了解为什么会有ant之类的之前，还是要体会一下使用
jdk的艰辛。

这个和以前在unix上开发的人用gcc命令行到后来使用make之后使用ide
之类的时候的体会是类似的。

最后，javac的-d参数是指示编译出来的class文件放在哪里的，如果你不指定的话，它们
和.java混在一起。当然也符合目录结构。
