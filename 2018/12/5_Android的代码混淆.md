目录：  

一、简介  
二、混淆配置  
三、混淆语法  
- 3.1、常见的混淆命令：
- 3.2、保持元素不参与混淆的规则  
    - [类] 代表类相关的限定条件，它的内容可以使用:
    - [成员] 代表类成员相关的限定条件
    - 常用的自定义混淆规则
    - -keep 和 -keepclassesmember  

四、通用的混淆规则
- 4.1 proguard-android.txt 文件内容
- 4.2 proguard-rules.pro 自定义混淆。以下为一些通用的混淆配置。  
五、检查混淆结果  
六、解出混淆栈


### 一、简介

Android中的"混淆"分为两部分。一部分是Java代码的优化与混淆，依靠proguard混淆器来完成。第二部分是资源压缩，移除项目及依赖库中未被使用的资源。  

这里主要说Java代码的优化与混淆。流程如下图：

![2](img/2)

代码混淆包含了代码压缩，优化，混淆等一系列过程。
1. shrink(压缩) ： Proguard会递归地确定哪些类和类成员被使用，其它则被丢弃；
2. optimize(优化) ： Proguard 会进一步分析和优化方法。比如一些无用的参数会被丢弃，一些方法会被做内联。
3. obfuscate(混淆) ：  进行重命名。吧原来包含注释意义的类名，方法名等进行无意义重命名。
4. preverify(预校验) : 将预校验信息添加到类中。



### 二、混淆配置

一般情况下，app module的 build.gradle 文件默认会有如下结构：


```
android {
    buildTypes {
        release {
            minifyEnabled true   //打开混淆
            shrinkResources true   // 打开资源压缩
            //proguard-android.txt表示默认的混淆规则，在sdk目录/tools/proguard中
            // proguard-rules.pro表示自定义的混淆规则（文件名和后缀可以修改）
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'   
        }
    }
}
```



### 三、混淆语法

#### 3.1、常见的混淆命令：


命令| 作用
---|---
-keep | 防止类或成员被移除或被重命名
-keepnames | 防止类或成员被重命名
-keepclassmembers |  防止成员被移除或者被重命名
-keepnames | 防止成员被重命名
-keepclasseswithmembers |  防止拥有该成员的类和成员被移除或者被重命名
-keepclasseswithmembernames | 防止拥有该成员的类和成员被重命名


#### 3.2、保持元素不参与混淆的规则

格式 ：


```
[保持命令] [类] {
    [成员]
}

```


##### [类] 代表类相关的限定条件，它的内容可以使用:

- 具体的类
- 访问修饰符 (public、protected、private)
- 通配符 * ： 匹配任意长度字符，但不含包名分隔符(.)
- 通配符 ** : 匹配任意长度字符，并且包含包名分隔符(.)
- extends ,即可以指定的基类
- implement ,匹配实现了某接口的类
- $ 内部类


##### [成员] 代表类成员相关的限定条件
- <init> 匹配所有的构造器
- <fields> 匹配所有域
- <methodds> 匹配所有方法
- 通配符 * ： 匹配任意长度字符，但不含包名分隔符(.)
- 通配符 ** : 匹配任意长度字符，并且包含包名分隔符(.)
- ...  ： 匹配任意长度类型参数。 比如 void test(...) 就能匹配任意void test(String a) 或者是 void test(int a, String b) 这些方法。
- 访问修饰符 (public、private、protected)


##### 常用的自定义混淆规则


```
-keep public class name.huihui.example.Test { *; } // 不混淆某个类

-keep class name.huihui.test.** { *; } // 不混淆某个包下所有类

-keep public class * extends name.huihui.example.Test { *; } // 不混淆某个类的子类

-keep public class **.*model*.** {*;} // 不混淆所有类名中包含了*module*的类及其成员

-keep class * implements name.huihui.example.TestInterface { *; } // 不混淆某个接口的实现

-keepclassmembers class name.huihui.example.Test {   // 不混淆某个类的构造方法
    public <init>(); 
}

-keepclassmembers class name.huihui.example.Test {   // 不混淆某个类的特定的方法
    public void test(java.lang.String); 
}

```


##### -keep 和 -keepclassesmember
上面虽然说得很详细了，但是可能有人会对于 keep和keepclassesmember还很懵，此处举个例子

对于 -keepclassesmember 命令 ：

```
-keepclasseswithmember class * {
    native <methods>;
}

```

就是保留所有含有nativie方法的类的类名和native方法名，而如果某个类中没有含有native方法，那就还是会被混淆。

但是如果改成 -keep命令，结果就会完全不一样：

```
-keep class * {
    native <methods>;
}
```

使用keep关键字后，代码中所有类的类名都不会被混淆，因为keep关键字看到class * 就认为应该所有类名进行保留，而不会关心该类中是否含有nativie方法。


### 四、通用的混淆规则 

在gradle配置混淆时有一行如下代码 去定义 混淆规则。其中
proguard-android.txt 为原生自带默认的混淆规则，在sdk目录/tools/proguard中

```
proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

```

proguard-android.txt表示默认的混淆规则，在sdk目录/tools/proguard中  

proguard-rules.pro表示自定义的混淆规则（文件名和后缀可以修改）

   

##### 4.1  proguard-android.txt 文件内容


```
#包名不混合大小写
-dontusemixedcaseclassnames

#不跳过非公共的库的类
-dontskipnonpubliclibraryclasses

#混淆时记录日志
-verbose

#关闭预校验
-dontpreverify

#不优化输入的类文件
-dontoptimize

#保护注解
-keepattributes *Annotation*

#保持所有拥有本地方法的类名及本地方法名
-keepclasseswithmembernames class * {
    native <methods>;
}

#保持自定义View的get和set相关方法
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}

#保持Activity中View及其子类入参的方法
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

#枚举
-keepclassmembers enum * {
    **[] $VALUES;
    public *;
}

#Parcelable
-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}

#R文件的静态成员
-keepclassmembers class **.R$* {
    public static <fields>;
}

-dontwarn android.support.**

#keep相关注解
-keep class android.support.annotation.Keep

-keep @android.support.annotation.Keep class * {*;}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}
```


#### 4.2  proguard-rules.pro 自定义混淆。以下为一些通用的混淆配置。



```
#指定压缩级别
-optimizationpasses 5


#混淆时采用的算法
# 这个过滤器是谷歌推荐的算法，一般不做更改
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*

#把混淆类中的方法名也混淆了
-useuniqueclassmembernames

#优化时允许访问并修改有修饰符的类和类的成员 
-allowaccessmodification

#将文件来源重命名为“SourceFile”字符串
-renamesourcefileattribute SourceFile

#保留行号
-keepattributes SourceFile,LineNumberTable


# 避免混淆泛型
-keepattributes Signature


#保持所有实现 Serializable 接口的类成员
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}


# 保留support下的所有类及其内部类
-keep class android.support.** {*;}

# 保留继承的
-keep public class * extends android.support.v4.**
-keep public class * extends android.support.v7.**
-keep public class * extends android.support.annotation.**


# 保留在Activity中的方法参数是view的方法，
# 这样以来我们在layout中写的onClick就不会被影响
-keepclassmembers class * extends android.app.Activity{
    public void *(android.view.View);
}

# webView处理，项目中没有使用到webView忽略即可
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
    public *;
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.webView, jav.lang.String);
}


```
所有在AndrodManifest.xml涉及到的类已经自动被保持，因此不用特意去添加这块混淆规则。  
proguard-android.txt已经存在一些默认混淆规则，没必要在proguard-rules.pro重复添加。

真正通用的、需要添加的就是上面这些，除此之外，需要每个项目根据自身的需求添加一些混淆规则：

- 第三方库所需的混淆规则。正规的第三方库一般都会在接入文档中写好所需混淆规则，使用时注意添加。
- 在运行时动态改变的代码，例如反射。比较典型的例子就是会与 json 相互转换的实体类。假如项目命名规范要求实体类都要放在model包下的话，可以添加类似这样的代码把所有实体类都保持住：-keep public class ** .*Model*.** {*;}
- JNI中调用的类。
- WebView中JavaScript调用的方法。


### 5、检查混淆结果

混淆过的包必须进行检查，避免因混淆引入的bug。  

一方面，需要从代码层面检查，使用上文配置进行混淆打包后在<module-name>/build/outputs/mapping/release/  | 目录下输出一下文件 ：

- dump.txt : 描述APK文件中所有类的内部结果；
- mapping.txt : 提供混淆前后类、方法、类成员等的对照表；
- seeds.txt ： 列出没有被混淆的类和成员；
- usage.txt ： 列出被移除的代码。

我们可以根据seeds.txt文件检查未被混淆的类和成员是否已包含所有期望保留的，在根据usage.txt文件查看是否有被误移除的代码。

另一方面 ，需要从测试方面检查，将混淆过的包进行全方面测试，检查是否有bug产生。



### 6、解出混淆栈
混淆前后的类、方法名等等难以阅读，对追踪线上crash造成了阻碍。可以使用sdk目录下/tools/proguard/bin中的proguardgui.bat可视化工具进行混淆定位。示例如下图：

![3](img/3)




### 参考 

[写给 Android 开发者的混淆使用手册](https://www.diycode.cc/topics/380)  

[Android修炼之混淆](https://juejin.im/post/5bfdf2f26fb9a049d61d3b29)  

[Android安全攻防战，反编译与混淆技术完全解析（下）](https://blog.csdn.net/guolin_blog/article/details/50451259)


