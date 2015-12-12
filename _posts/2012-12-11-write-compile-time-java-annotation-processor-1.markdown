---
layout: post
title:  "自制编译时 Java 注解处理器（一）"
date:   2015-12-10 20:52:00
---

## 背景

想用 [dagger 2] 替换掉项目中的 [roboguice]，但 [dagger 2] 又没有提供 @InjectExtra 这样的便利，就想自制一个，像 [dagger 2] 那样在编译时生成相关代码。

但我刚接触 Android 开发 1 个多月，Java 多年没用，maven 都不会。这将是一个有意思的挑战，于是有了这篇入门文。

## 开始

首先很幸运的 Google 到了教程 [ANNOTATION PROCESSING 101] 及其译文 [Java注解处理器]，开始旅程：

### 一、编译 sockeqwe/annotationprocessing101

{% highlight bash %}
# 安装 maven
brew install maven

git clone git@github.com:sockeqwe/annotationprocessing101.git

cd annotationprocessing101/factory/annotation
mvn install
# ... 顺利完成

annotationprocessing101/factory/processor
mvn install
# [ERROR] Failed to execute goal on project processor: Could not resolve dependencies for project com.hannesdorfmann.annotationprocessing101.factorypattern:processor:jar:1.0: Failed to collect dependencies at com.hannesdorfmann.annotationprocessing101.factorypattern:annotation:jar:1.0: Failed to read artifact des
# criptor for com.hannesdorfmann.annotationprocessing101.factorypattern:annotation:jar:1.0: Failure to find com.hannesdorfmann.annotationprocessing101.factory pattern:parent:pom:1.0 in https://repo.maven.apache.org/maven2 was cached in the local repository, resolution will not be reattempted until the update interval of central has elapsed or updates are forced -> [Help 1]
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

### 二、在 Swain 项目里使用

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

[Swain] 是个人用来测试 Android 开发的实验项目，把上面的代码，和 annotationprocessing101 里提供的 [示例代码](https://github.com/sockeqwe/annotationprocessing101/tree/master/factory-sample/pizzastore) 加入后 ，Build->Rebuild Project ，就成功生成了代码：

> Swain/app/build/generated/source/apt/debug/im/lot/swain/pizza/MealFactory.java

完整可运行代码，修改文件记录，可见 Swain 项目 dagger2 分支下的 [Swain-dagger2-cf9190]。

[ANNOTATION PROCESSING 101]: http://hannesdorfmann.com/annotation-processing/annotationprocessing101/
[Java注解处理器]: http://www.race604.com/annotation-processing/
[dagger 2]: https://github.com/google/dagger
[roboguice]: https://github.com/roboguice/roboguice
[Swain]: https://github.com/lotreal/Swain
[Swain-dagger2-cf9190]: https://github.com/lotreal/Swain/commit/cf919091a7ceed27dd12564e285a001bec0a02e2
