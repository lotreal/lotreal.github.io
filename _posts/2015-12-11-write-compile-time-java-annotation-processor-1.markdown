---
layout: post
title:  "自制编译时 Java 注解处理器（一）"
date:   2015-12-11 21:58:00
---

在 DI（依赖注入）工具中，[Dagger 2] 不用反射，而是在编译时完整地生成所需代码。比起 [Roboguice] 等，不但避免了反射的性能损耗，更重要的是使得代码可读性更强，易于调试。

所以想用 Dagger 替换掉公司项目中的 Roboguice。但 Dagger 里没有提供 @InjectExtra 这样的便利。就想自制一个，并像 Dagger 那样在编译时生成相关代码。

> 注意：这是一篇入门文，是一次新手的探险。因为作者刚接触安卓开发 1 个多月，Java 多年没用，maven 不会 gradle 不熟。

## 准备

1. Android Studio 1.5.1
2. [Swain] : 个人试探 Android 开发的实验项目，提供完整、可运行的实例。

## 初探

首先很幸运的 Google 到了教程 [ANNOTATION PROCESSING 101] 及其译文 [Java注解处理器]。

今天的目标是把文中的示例代码跑起来，开始旅程：

### 0x01 编译 sockeqwe/annotationprocessing101

{% highlight bash %}
# OSX 下安装 maven
brew install maven

git clone git@github.com:sockeqwe/annotationprocessing101.git

cd annotationprocessing101/factory/annotation
mvn install
# ... 顺利完成

annotationprocessing101/factory/processor
mvn install
# [ERROR] Failed to execute goal on project processor: Could not resolve dependencies for project com.hannesdorfmann.annotationprocessing101.factorypattern:processor:jar:1.0:
# Failed to collect dependencies at com.hannesdorfmann.annotationprocessing101.factorypattern:annotation:jar:1.0:
# Failed to read artifact descriptor for com.hannesdorfmann.annotationprocessing101.factorypattern:annotation:jar:1.0:
# Failure to find com.hannesdorfmann.annotationprocessing101.factory pattern:parent:pom:1.0 in https://repo.maven.apache.org/maven2 was cached in the
# local repository, resolution will not be reattempted until the update interval of central has elapsed or updates are forced -> [Help 1]
{% endhighlight %}

Google 到[解决方法](http://stackoverflow.com/questions/6642146/maven-failed-to-read-artifact-descriptor)：

> This problem can also occur if you have some child projects that refer to a parent pom and you have not installed from the parent pom directory (run mvn > install from the parent directory). One of the child projects may depend on a sibling project and when it goes to read the pom of the sibling, it will > > fail with the error mentioned in the question unless you have installed from the parent pom directory at least once.
>
> In addition, when running mvn install on parent you can also add -N for non-recursive operation. This will cause maven to skip all the modules (including one that fails) and just do install goal for the parent. – Jacek Prucia

{% highlight bash %}
cd annotationprocessing101/factory
mvn install
# ... 顺利完成
{% endhighlight %}

现在我们已将该项目打包并安装到了 maven 的本地仓库里：

> ~/.m2/repository/com/hannesdorfmann/annotationprocessing101/factorypattern/

### 0x02 在 Swain 项目里使用

{% highlight groovy %}
repositories {
    jcenter()
    mavenLocal()
}

dependencies {
    compile 'com.hannesdorfmann.annotationprocessing101.factorypattern:annotation:1.0'
    apt 'com.hannesdorfmann.annotationprocessing101.factorypattern:processor:1.0'
}
{% endhighlight %}

在 [Swain] 里加入上面的代码，以及 annotationprocessing101 里提供的 [示例代码](https://github.com/sockeqwe/annotationprocessing101/tree/master/factory-sample/pizzastore) 后 ，Build->Rebuild Project ，就成功生成了代码：

> Swain/app/build/generated/source/apt/debug/im/lot/swain/pizza/MealFactory.java
>
> 完整代码可见 Swain 项目 dagger2 分支下的 [Swain-dagger2-cf9190]。

现在，我们成功地运行了一个编译时代码生成器。

### 0x03 相关链接

1. [ANNOTATION PROCESSING 101]

[ANNOTATION PROCESSING 101]: http://hannesdorfmann.com/annotation-processing/annotationprocessing101/
[Java注解处理器]: http://www.race604.com/annotation-processing/
[Dagger 2]: https://github.com/google/dagger
[Roboguice]: https://github.com/roboguice/roboguice
[Swain]: https://github.com/lotreal/Swain
[Swain-dagger2-cf9190]: https://github.com/lotreal/Swain/tree/cf919091a7ceed27dd12564e285a001bec0a02e2
