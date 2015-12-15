---
layout: post
title:  "自制编译时 Java 注解处理器（二）"
date:   2015-12-12 10:42:00
---

继续探险，开始自制 @InjectExtra

## 0x01 怎么开始

1. 直接修改 dagger 生成的代码文件。简单粗暴，但 [JavaPoet] 似乎不支持修改已有的 java 文件，也没找到其它好的工具。而且这样会要求 APT 顺序必须正确，可能对使用者造成困扰。

2. 借鉴 dagger 完全编写自己的 dagger-compiler，但这样的话代码抄袭量会很多，而且我也不想在项目中看到两个雷同的 Activity_MembersInjector.java。

3. 我选择 Fork 一个[自己的 dagger 分支](https://github.com/lotreal/dagger)。这样做不需要增加一个 APT 处理，编译更快，还可以考虑把 ButterKnife 的注入代码也给自动生成了。

{% highlight bash %}
git clone git@github.com:google/dagger.git
cd dagger

# 修改包名，此处使用了 osx 自带的 sed，参数用法和 gun sed 略有不同
find . -name pom.xml | xargs sed -i "" "s/com.google.dagger/im.lot.dagger/"
mvn install
... 顺利完成
{% endhighlight %}

整合到 [Swain-dagger2-5f1378]：
{% highlight groovy %}
dependencies {
    compile 'im.lot.dagger:dagger:2.1-SNAPSHOT'
    apt 'im.lot.dagger:dagger-compiler:2.1-SNAPSHOT'
}
{% endhighlight %}

现在，修改这份 fork 版 dagger, 生成 im.lot.dagger 包后，就可以在其它项目引用了。

## 0x02 无法接受的 3 分钟

但是，每次 mvn install 耗时 1m30s ，意味着每次 im.lot.dagger 的小改动，想在 Swain 看到效果的话，都必须等 3 分钟！

试着在 Swain 新建了 Java Library 类型 module: dagger-compiler，然后引入相关代码：

> 为方便修改，我在 [Swain/dagger-compiler/src/main/java](https://github.com/lotreal/Swain/blob/e3c7c0ff3f09a3649b252469a1d6974205a1b66e/dagger-compiler/src/main/java) 目录下， ln -s 了 [dagger/compiler/src/main/java/dagger](https://github.com/lotreal/dagger/tree/4d91161f59eb688d19b5aeac0e58eb546086715c/compiler/src/main/java/dagger/) 目录

遇到问题：Java Library Module 的 build.gradle 里，无法使用 provided 类型依赖，无法为 dagger-compiler 里大量使用的 @AutoValue 生成代码。

Google 到[解决方法](http://stackoverflow.com/questions/18738888/how-to-use-provided-scope-for-jar-file-in-gradle-build)，使用 [gradle-propdeps-plugin] 解决了问题。

{% highlight groovy %}
apt project(':dagger-compiler')
{% endhighlight %}

Module 使用成功！现在代码是：[Swain-dagger2-dcb1c5]，修改 dagger-compiler 后，只需 Rebuild Swain , 15 秒后就可以看到修改后的效果了。

## 0x03 断点调试

在 dagger-compiler 里使用 Logger 打印日志调试后，发现如果能断点调试就更好了，参考 [Debug an Android annotation processor with gradle and IntelliJ (or Eclipse)]:

1. Android Studio：在 Run/Debug Configurations 里添加 Remote，配置取名 REMOTE，其它使用默认即可，然后菜单：Run->Debug 'REMOTE'

2. vi ~/.gradle/gradle.properties
{% highlight groovy %}
org.gradle.daemon=true
org.gradle.jvmargs=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
{% endhighlight %}

现在开发起来就愉快多了，插入了自制的简易 @InjectExtra 代码后，项目文件为：[Swain](https://github.com/lotreal/Swain/tree/e3c7c0ff3f09a3649b252469a1d6974205a1b66e) + [dagger](https://github.com/lotreal/dagger/tree/4d91161f59eb688d19b5aeac0e58eb546086715c)

> 注意：Swain/dagger-compiler 里链接使用了 dagger/compiler 里的代码。

{% highlight java %}
// 生成的 Swain/app/build/generated/source/apt/debug/
// im/lot/swain/MainActivity_MembersInjector.java 实例
@Override
public void injectMembers(MainActivity instance) {
  if (instance == null) {
    throw new NullPointerException("Cannot inject members into a null reference");
  }
  instance.config = configAndConfig2Provider.get();
  instance.config2 = configAndConfig2Provider.get();
  instance.textView =
      (android.widget.TextView) instance.getIntent().getSerializableExtra("textView");
}
{% endhighlight %}

## 0x04 相关链接

1. [Android Annotation Processing: POJO string generator]
2. [Debug an Android annotation processor with gradle and IntelliJ (or Eclipse)]

> Android Studio 奇怪问题：有时候无法生成 AutoValue_BindingKey.java 等源文件。此时进入 Swain 目录，切换一下分支，如：git checkout master; git checkout dagger2 后就解决了。

[ANNOTATION PROCESSING 101]: http://hannesdorfmann.com/annotation-processing/annotationprocessing101/
[Java注解处理器]: http://www.race604.com/annotation-processing/
[dagger 2]: https://github.com/google/dagger
[roboguice]: https://github.com/roboguice/roboguice
[Swain]: https://github.com/lotreal/Swain
[Swain-dagger2-cf9190]: https://github.com/lotreal/Swain/commit/cf919091a7ceed27dd12564e285a001bec0a02e2
[JavaPoet]: https://github.com/square/javapoet
[Swain-dagger2-5f1378]: https://github.com/lotreal/Swain/commit/5f157888ea24c1e038d7250a1b7396eb479a2449
[Swain-dagger2-dcb1c5]: https://github.com/lotreal/Swain/tree/dcb1c540eabada9d714bbfe211c0750059504c10
[gradle-propdeps-plugin]: https://github.com/spring-projects/gradle-plugins/tree/master/propdeps-plugin
[Android Annotation Processing: POJO string generator]: http://brianattwell.com/android-annotation-processing-pojo-string-generator/
[Debug an Android annotation processor with gradle and IntelliJ (or Eclipse)]: https://turbomanage.wordpress.com/2014/06/09/debug-an-annotation-processor-with-intellij-and-gradle/
