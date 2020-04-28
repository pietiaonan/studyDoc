## 一、maven依赖原则

-  **依赖最短路径优先原则**

一个项目Demo依赖了两个jar包，其中A-B-C-X(1.0) ， A-D-X(2.0)。由于X(2.0)路径最短，所以项目使用的是X(2.0)。

- **pom文件中申明顺序优先**

 如果A-B-X(1.0) ，A-C-X(2.0) 这样的路径长度一样怎么办呢？这样的情况下，maven会根据pom文件声明的顺序加载，如果先声明了B，后声明了C，那就最后的依赖就会是X(1.0)。

- **覆写优先**

子pom内声明的优先于父pom中的依赖。

## 二、如何解决jar冲突

遇到冲突的时候第一步要找到maven加载的到时是什么版本的jar包，通过们`mvn dependency:tree`查看依赖树，通过maven的依赖原则来调整坐标在pom文件的申明顺序是最好的办法。

## 三、scope的依赖传递

\* compile，缺省值，适用于所有阶段，会随着项目一起发布。

\* provided，类似compile，期望JDK、容器或使用者会提供这个依赖。如servlet.jar。

\* runtime，只在运行时使用，如JDBC驱动，适用运行和测试阶段。

\* test，只在测试时使用，用于编译和运行测试代码。不会随项目发布。

\* system，类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。

![微信图片_20200318095846](..\img\maven.jpg)

A–>B–>C。当前项目为A，A依赖于B，B依赖于C。知道B在A项目中的scope，那么怎么知道C在A中的scope呢？答案是：

当C是test或者provided时，C直接被丢弃，A不依赖C；

否则A依赖C，C的scope继承于B的scope。

## 四、<optional>true</optional>属性

在添加依赖项时，我们可以使用optional熟悉，或将scope设置为“provided”。在这两种情况下，依赖关系都将在声明它们的模块的classpath中，但是使用将它们定义为依赖关系的模块不会在其他项目中传递它们，即不会形成依赖传递

例如上面的例子，在SpringBoot官网文件中你可以得到解释就是，<optional>true</optional>的话，其他项目依赖此项目也不会进行传递，只能本项目使用。