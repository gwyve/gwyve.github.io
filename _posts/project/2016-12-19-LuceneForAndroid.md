---
layout: post
title: LuceneForAndroid
category: project
description: Lucene5.5.0在Android上面的适配
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：[VE] 
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    

## 项目路径
[LuceneforAndroid](https://github.com/gwyve/LuceneForAndroid)

## 背景介绍
Lucene是个开源的检索引擎（具体功能和实现我也不太懂）。这个项目的主要目的是在将[Lucene5.5.0](http://archive.apache.org/dist/lucene/java/5.5.0/)这个版本移植到Android平台使用。其中通过测试替换为本项目jar包后的[LuceneSearchDemo-Android](https://github.com/lukhnos/LuceneSearchDemo-Android)，来判定本项目是否成功。
本项目主要参考[mobilelucene](https://github.com/lukhnos/mobilelucene/tree/master/lucene）。这里特别感谢[Lukhnos Liu](https://github.com/lukhnos)

## 环境说明
- Ubuntu 14.04 64bit
- openjdk-1.8
- ant-1.9.2

## 具体步骤

### 环境搭建

这个问题自行google吧，只要有openjdk、ant、ivy，应该就可。这里注意，一定要用openjdk而不是jdk

### 代码修改
__说明：这里将下载下来的lucene5.5.0源码称为Lucene，将从Lukhnos所clone下来的代码称为LuceneK（没有原因，只是想不出叫什么名字而已）__

- 将Lucene5.5.0的源码解压
- 将LuceneK中core/src/java/org/lukhons中的源码拷贝到Lucene中core/src/java/org/中
- 将LuceneK中translate_common.py和transform.py放到lucene的根目录下，并运行transform.py
- 参考LuceneK在根目录下的build.xml，在lucene的跟目录下的build.xml中添加

```xml
   <target name="mobile-build-modules-without-test" depends="jar-core"
          description="Builds all mobile additional modules without their tests">
    <mobile-modules-crawl target="default"/>
    <mobile-modules-crawl target="mobile-copy-jar"/>
  </target>
```

- 参考LuceneK根目录下的common-build.xml，在Lucene的根目录下的common-build.xml添加

```xml
<macrodef name="mobile-modules-crawl">
    <attribute name="target" default=""/>
    <attribute name="failonerror" default="true"/>
    <sequential>
      <subant target="@{target}" failonerror="@{failonerror}" inheritall="false">
        <propertyset refid="uptodate.and.compiled.properties"/>
        <fileset dir=".">
          <include name="analysis/common/build.xml" />
          <include name="core/build.xml" />
          <include name="highlighter/build.xml" />
          <include name="join/build.xml" />
          <include name="memory/build.xml" />
          <include name="misc/build.xml" />
          <include name="queries/build.xml" />
          <include name="queryparser/build.xml" />
          <include name="sandbox/build.xml" />
          <include name="suggest/build.xml" />
          <exclude name="build/**" />
          <!-- <exclude name="core/**" /> -->
          <exclude name="test-framework/**" />
          <exclude name="tools/**" />
        </fileset>
      </subant>
    </sequential>
  </macrodef>

  <property name="mobilejars" value="" />

  <target name="mobile-copy-jar">
    <fail message="Property mobilejars has no value">
      <condition>
        <equals arg1="${mobilejars}" arg2=""/>
      </condition>
    </fail>

    <copy todir="${mobilejars}" verbose="yes" flatten="yes" failonerror="no">
     <fileset dir="${build.dir}">
        <include name="*.jar" />
     </fileset>
    </copy>
  </target>
```

- 在core中修改org.apache.lucene.util.OfflinerSorter.java、org.apahce.lucene.util.IOUtils.java、org.apache.store.NativeFSLockFactory.java、org.apache.store.NIOFSDirectory.java、org.apache.store.MMapDirectory.java、org.apache.store.FSDirectory.java具体修改参见该项目的标记“Changed by VEVEVE”相关代码

- 在analysis/common中修改org.tartarus.snowball.Among.java具体想修改参见该项目的标记“Changed by VEVEVE”相关代码。__这里并没有考虑5.5.0在这里修改之后的具体功能，按照作者目前的修改可能会造成不可知的某种错误而造成闪退。该功能理论上在Android平台应该无法使用__

- 在spatial3d中修改org.apache.lucene.bkdtree3d.OfflineWriter.java、org.apache.lucene.bkdtree3d.BKD3DTreeWriter.java、org.apache.lucene.bkdtree3d.OfflineReader.java具体想修改参见该项目的标记“Changed by VEVEVE”相关代码。__作者认为如果在translate_common.py中添加spatial3d的相关目录，应该可以省略此步骤，作者并未验证__

- 在core中修改org.apache.lucene.util.AttributeSource.java、org.apache.lucene.util.Factory.java具体想修改参见该项目的标记“Changed by VEVEVE”相关代码。__这一步比较难以发现，主要因为这一步是在android平台运行之后总是报NoClassDefFoundError，报错的行数跟我们编译前源码的行数不对应，这步骤比较难以发现，因为是运行错误。__

- 编译：在Lucene跟目录下执行

```bush
ant -Dmobilejars=../build/out/ clean mobile-build-modules-without-test 
```

在backward-codes/下执行

```bush
ant
```

- 将[LuceneSearchDemo-Android](https://github.com/lukhnos/LuceneSearchDemo-Android)中的[lib](https://github.com/lukhnos/lucenestudy/tree/f992866738da1d15c45db663c4ef3eb074adb65e/libs)更换为本项目新编译的jar:lucene-analyzers-common.jar、lucene-core.jar、lucene-highlighter.jar、lucene-join.jar、lucene-memory、lucene-misc.jar、lucene-misc.jar、lucene-queries.jar、lucene-queryparser.jar、lucene-sandbox.jar、lucene-spatial3d.jar、lucene-suggest.jar、lucene-backward-codecs.jar

- 编译该app可在android运行使用

## 若干问题说明

### ant和ivy
ant是java的一个编译工具，具体使用网络有相关教程。ivy为ant中处理相关依赖的一个工具，起初，试图使用LuceneK的ivy文件，但是，因为5.5.0跟5.3.0在依赖上发生改变（例如5.5.0sandbox依赖spatial3d，在5.3.0中依赖spatial），所以本项目继续使用5.5.0的ivy相关配置文件

### jdk openjdk AndroidSdk
简单而言前两者者有交际同时又不同，Lucene的使用是在oracle开发的jdk下进行使用的，但是Lucene所涉及的相关包在Openjdk中并不存在，这就是本项目存在意义。

在openjdk先编译成功并不意味着在Android上可以运行成功，仍然会报NoClassDefFoundError。

### Android Studio
在AS上面开发，已经习惯于在error提示的代码行上进行bug查找。而AS上显示的代码行号并非实际源码，针对AS的错误提示，如果是引入jar的问题，有的时候行号没有任何意义。

### 自己的思考
[Oracle终于要向Java的非付费用户开枪了](https://news.cnblogs.com/n/559240)

如果收费的话，那只能依赖openjdk，Lukhnos这个大神确实给了一个很好的解决方案。有很大的参考意义