# 升级到Java10之旅失败之旅，最终回到了Java9

## TL;DR

昨天尝试把项目升级到

* Jdk Java10，
* 构建工具 Gradle 4.9，
* IDE IntelliJ IDEA 2018.1

后来遇到：

* Gradle 4.9 与 lombok:1.18.0 不兼容，需要等待 1.18.1
* Java 10 与 Apache POI 3.7 不兼容，需要等待 4.0

没有成功，最后将环境升级到：

* JDK Java 9,
* 构建工具 Gradle 4.7
* IDE IntelliJ IDEA 2018.1



## 升级到Java 10，听起来是个good idea么？！

上半年在做的一个 JAVA 项目，整体技术栈是 JAVA8 + Spring Boot 1.5 + Gradle 4.0 构建的；最近有点时间，我想着把项目的JDK升级到 JAVA 10，学习下模块化和新的 JAVA 语法，做下技术储备。

### 坑坑坑， Java 10 依然是大坑，by now

为什么是JAVA 10 ？？

虽然从 Java 8 一步到 Java 10，步子有点大，但是既然 Java 官方说了 10 是LTS，总归是可靠的吧… 另外一个原因，我是重度的 homebrew 患者，brew上只能下到 Java8 或者 Java 10。

很随性的理由吧，我就开始了随性的趟坑之旅。

## 趟坑之旅

### 趟坑的第一步——保证环境恢复能力

虽然人暂时不在项目上，但保不齐哪天就会被拉回去板砖了。在升级 Java 10之前，我 spike 了一下 动态切换 Java 环境的方案。

我的工作环境：

* MacBook Pro：macOS High Sierra 10.13.6
* Shell：Zsh
* Java Path：/Library/Java/JavaVirtualMachines
* IDE：IDE IntelliJ IDEA

对于 Terminal 而言，我希望的动态切换就是通过命令在终端里随时在 Java 10 和 Java 8的环境间切换。

这个想法，实现起来也很简单：通过动态设置 JAVA_HOME这个环境变量，指向对应版本的 Jdk 目录就可以了。

具体实现起来：我们可以通过`/usr/libexec/java_home`命令来获得对应版本的Jdk Path；通过 zsh 的 别名来创建切换命令; 把命令写在 .zshrc 里保证加载执行

代码如下：

```bash
# 设置 JDK 8
export JAVA_8_HOME=`/usr/libexec/java_home -v 1.8`
# 设置 JDK 9
export JAVA_9_HOME=`/usr/libexec/java_home -v 9.0`
# 设置 JDK 10
export JAVA_10_HOME=`/usr/libexec/java_home -v 10.0`
# 默认 JDK 9
export JAVA_HOME=$JAVA_9_HOME

# 动态切换版本
alias jdk8="export JAVA_HOME=$JAVA_8_HOME"
alias jdk10="export JAVA_HOME=$JAVA_10_HOME"
alias jdk9="export JAVA_HOME=$JAVA_9_HOME"
```

*P.S 后来发现有 [JEnv](http://www.jenv.be/) 这个工具 可以帮我们完成以上的配置*



对于IDE 而言，我们就需要显示的设置项目的 SDK了，

具体可以看 [Working with SDKs](https://www.jetbrains.com/help/idea/sdk.html) 这个帖子



好，那让我们现在执行` jdk10`，切换到 Java 10 的环境下，走起吧。



### 趟坑的第二步—— 升级到Gradle 支持 Java 10

设置好 JDK，那么下面一步就是华丽丽的跑下`gradle build`了。如果 compile 和 test 跑通了，那就说明项目的大部分功能还是基本可用的。然并卵，我看到的只有华丽丽的报错

```bash
sh gradlew build --stacktrace                                                                                 2018-07-19 13:46

FAILURE: Build failed with an exception.

* What went wrong:
Could not determine java version from '10.0.2'.

* Try:
Run with --info or --debug option to get more log output.

* Exception is:
java.lang.IllegalArgumentException: Could not determine java version from '10.0.2'.
        at org.gradle.api.JavaVersion.toVersion(JavaVersion.java:70)
        at org.gradle.api.JavaVersion.current(JavaVersion.java:80)
        at org.gradle.internal.jvm.UnsupportedJavaRuntimeException.assertUsingVersion(UnsupportedJavaRuntimeException.java:29)
        at org.gradle.launcher.cli.JavaRuntimeValidationAction.execute(JavaRuntimeValidationAction.java:32)
        at org.gradle.launcher.cli.JavaRuntimeValidationAction.execute(JavaRuntimeValidationAction.java:24)
```

what？ gradle 不支持 Java 10…… 官方说，要4.7 才能支持 Java 10。那好我升级到 4.9可以了吧。

### 趟坑的第三步 —— Gradle 4.9 和 lombok的冲突

升级到4.9，Gradle终于正常跑起来了。可是好景不常，又看到华丽丽的报错画面

```bash
> Task :compileJava FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileJava'.
> java.lang.ExceptionInInitializerError

```

这事啥~~ :-(，难道gradle 4.9还不足以征服 Java 10么



加`—stacktrace`打下堆栈看眼，原来是 lombok挂了。

```java
        ... 56 more
Caused by: java.lang.ClassNotFoundException: com.sun.tools.javac.code.TypeTags
        at lombok.launch.ShadowClassLoader.loadClass(ShadowClassLoader.java:422)
        at lombok.javac.JavacTreeMaker$SchroedingerType.getFieldCached(JavacTreeMaker.java:156)
        at lombok.javac.JavacTreeMaker$TypeTag.typeTag(JavacTreeMaker.java:245)
        at lombok.javac.Javac.<clinit>(Javac.java:155)
```

到 lombok 官方看了下 最新版本已经是 1.18.0，我用的还是 1.16.20 （P.S 随着Java的起舞，感觉最近 Java圈的这些 library 都升级得好快哈），升之。

然后，依然是华丽丽的

```java
        ... 56 more
Caused by: java.lang.ClassNotFoundException: com.sun.tools.javac.code.TypeTags
        at lombok.launch.ShadowClassLoader.loadClass(ShadowClassLoader.java:422)
        at lombok.javac.JavacTreeMaker$SchroedingerType.getFieldCached(JavacTreeMaker.java:156)
        at lombok.javac.JavacTreeMaker$TypeTag.typeTag(JavacTreeMaker.java:245)
        at lombok.javac.Javac.<clinit>(Javac.java:155)
```

在 lombok 的 issue 里看了看，发现这是因为当前的 lombok 和 gradle 4.9不兼容造成的，并且没有 workaround的方法。那好我退，降级到 4.7…

### 趟坑的第四步 —— Java 10 那些消失的包

即使降级到 4.7，compile 阶段依然在报错，一片一片的报错；

```java
import javax.xml.ws.ProtocolException;
                   ^
  symbol:   class ProtocolException
  location: package javax.xml.ws
  --------------------------------------
 import javax.xml.bind.JAXBException;
               ^
  (package javax.xml.bind is declared in module java.xml.bind, which is not in the module graph)
```

这个就是 Java 9 中 jigsaw 带来的锅了：为了更好的进行模块化，Java 9 中 属于 J2EE 而不是 J2SE 的一些包，默认不能被系统引用。这些包有：

- *java.activation* with javax.activation package
- *java.corba* with javax.activity, javax.rmi, javax.rmi.CORBA, andorg.omg.* packages
- *java.transaction* with javax.transaction package
- *java.xml.bind* with all javax.xml.bind.* packages
- *java.xml.ws* with javax.jws, javax.jws.soap, javax.xml.soap, and alljavax.xml.ws.* packages
- *java.xml.ws.annotation* with javax.annotation package

具体的信息可以看下[JEP 320: Remove the Java EE and CORBA Modules](http://openjdk.java.net/jeps/320)



解决这个问题，官方给出了两种方案：

1. 在 Java 11 前，可以用`—add-modules java.xml.bind` 来提示 java 把相关的模块引入

2. 在 Java 11 及之后的版本，`add-modules` 因为 以上的模块将彻底从 Java 11 中移除掉，所以`—add-modules java.xml.bind`就不work了，需要我们显示引用 独立出来的相关maven模块。

   例如：

   ```java
   compile 'javax.xml.ws:jaxws-api:2.3.0'
   ```

这里选择了方案2，理由有两个：

1. 向后兼容 Java 11，一次改到底。
2. 如果用了`—add-modules java.xml.bind`，需要编译时候增加这个参数，执行的时候也需要增加这个参数，明显的duplicate。

### 趟坑的第五步 —— POI 3.7 与 Java 10 不兼容

* compile阶段通过；
* check阶段通过
* 开始单元测试了…..单元测试挂了

看了下测试报告，错误

> ```
> org.apache.poi.openxml4j.exceptions.OpenXML4JRuntimeException: Fail to save: an error occurs while saving the package : org.apache.poi.openxml4j.util.ZipSecureFile$ThresholdInputStream cannot be cast to java.base/java.util.zip.ZipFile$ZipFileInputStream
> ```

这个是因为 Apache POI 一直用一个比较hacker的方式来防御`zip-bomb protection`

具体间[Compiling with Java 10 fails with ClassCastException ](https://bz.apache.org/bugzilla/show_bug.cgi?id=62187) 和 [zip-bomb protection](https://lists.apache.org/thread.html/20120c9ec71ab2de8a4f8217a89a0bd4ebd93cd07c0ddd98f6c301f1@%3Cuser.commons.apache.org%3E)

顺带看了下 POI 4.0的发布都延期3个月了….

没招了，降到 Java9吧，发现官网上已经不直接给出 Java9 的下载地址了，下Java9的都给转到Java10

`吐槽一个，现在Apache基金和Java组织到底啥关系哈，赤裸裸的不兼容 Java10 还是被主推了`

[Java9 下载地址](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase9-3934878.html)



### 趟坑的第六步 —— jacoco 与 gradle 的冲突

回到 Java9，世界一起都正常了。

其实有一个 jacoco 的问题，之前一直没有处理。gradle 4.6 以后 和 jacoco 0.7.8以下的版本不兼容，并且 gradle-jacoco-plugin的一些参数名变了classDumpFile->classDumpDir。



### 还有更远的路要走

因为有很多第三方 Vendor 以 Jar 提供的 SDK，并且这些 SDK的运行强依赖于第三方的服务

我们没法在本地完成测试…..后面的坑，觉得更多….









