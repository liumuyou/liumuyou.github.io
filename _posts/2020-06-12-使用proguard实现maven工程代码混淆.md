---
layout:     post
title:      通过JAVA自动生成数据字典
subtitle:   JAVA 数据字典
date:       2020-06-11
author:     MY
header-img: img/post-bg-data.jpg
catalog: true
tags:
    - JAVA
    - 数据字典
---

proguard简单来说是为了防止反编译，更准确的说，是使得代码易读性变差。

### maven pom配置中加入以下：
```
<!-- ProGuard混淆插件-->
<plugin>
    <groupId>com.github.wvengen</groupId>
    <artifactId>proguard-maven-plugin</artifactId>
    <version>2.1.0</version>
    <executions>
        <execution>
            <!-- 混淆时刻，这里是打包的时候混淆-->
            <phase>package</phase>
            <goals>
                <!-- 指定使用插件的混淆功能 -->
                <goal>proguard</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!-- 是否将生成的PG文件安装部署-->
        <attach>true</attach>
        <!-- 是否混淆-->
        <obfuscate>true</obfuscate>
        <!-- 指定生成文件分类 -->
        <attachArtifactClassifier>pg</attachArtifactClassifier>
        <proguardInclude>${basedir}/proguard.conf</proguardInclude>
        <libs>
            <lib>${java.home}/lib/rt.jar</lib>
            <lib>${java.home}/lib/jce.jar</lib>
        </libs>
        <!-- 对什么东西进行加载，这里仅有classes成功，不可能对配置文件及JSP混淆吧-->
        <injar>classes</injar>
        <outjar>${project.build.finalName}-pg.jar</outjar>
        <!-- 输出目录-->
        <outputDirectory>${project.build.directory}</outputDirectory>
    </configuration>
</plugin>
```
### 在pom同级目录下添加proguard.conf文件

```
# 忽略所有警告，否则有警告的时候混淆会停止
-ignorewarnings

# JDK目标版本1.8
-target 1.8

# 不做收缩（删除注释、未被引用代码）
-dontshrink

# 不做优化（变更代码实现逻辑）
-dontoptimize

# 不路过非公用类文件及成员
-dontskipnonpubliclibraryclasses
-dontskipnonpubliclibraryclassmembers

# 优化时允许访问并修改有修饰符的类和类的成员
-allowaccessmodification

# 确定统一的混淆类的成员名称来增加混淆
-useuniqueclassmembernames

# 不混淆所有包名，本人测试混淆后WEB项目问题实在太多，毕竟Spring配置中有大量固定写法的包名
-keeppackagenames

# 不混淆局部变量名
-keepparameternames

# 不混淆所有特殊的类 LocalVariable*Table,
-keepattributes Exceptions,InnerClasses,Signature,Deprecated,SourceFile,LineNumberTable,*Annotation*,Synthetic,EnclosingMethod

# 不混淆包下的所有类名
-keep class weg.base.** { <methods>; }
-keep class weg.service.** { <methods>; }
-keep class weg.dao.** { <methods>; }
-keep class weg.util.** { <methods>; }

# 不混淆quartz包下的所有类名，且类中的方法也不混淆
-keep class weg.quartz.** { <methods>; }

# 不混淆model包中的所有类以及类的属性及方法，实体包，混淆了会导致ORM框架及前端无法识别
-keep class weg.model.** {*;}

# 不混淆所有的set/get方法，毕竟项目中使用的部分第三方框架（例如Shiro）会用到大量的set/get映射
-keepclassmembers public class * {void set*(***);*** get*();}

# 保持类protected不被混淆
-keep public class * { public protected <fields>;public protected <methods>; }
```

### 执行maven命令：

```
clean package -DskipTests

执行上述代码后，在target目录下会生成3个文件：
· classes-pg.jar 混淆后的classes文件，里面包含完整的项目结构
· proguard_map.txt 混淆内容的映射
· proguard_seed.txt 参与混淆的类
```

注：也可以使用[proGuard工具](https://sourceforge.net/projects/proguard/)生成配置