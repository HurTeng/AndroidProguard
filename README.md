# AndroidProguard
Android代码混淆，包含了通用混淆配置，以及常用的第三方库混淆配置

# 简介

作为Android开发者，如果你不想开源你的应用，那么在应用发布前，就需要对代码进行混淆处理，从而让我们代码即使被反编译，也难以阅读。混淆概念虽然容易，但很多初学者也只是网上搜一些成型的混淆规则粘贴进自己项目，并没有对混淆有个深入的理解。本篇文章的目的就是让一个初学者在看完后，能在不进行任何帮助的情况下，独立写出适合自己代码的混淆规则。

代码压缩通过 ProGuard 提供，ProGuard 会检测和移除封装应用中未使用的类、字段、方法和属性，包括自带代码库中的未使用项（这使其成为以变通方式解决 64k 引用限制的有用工具）。ProGuard 还可优化字节码，移除未使用的代码指令，以及用短名称混淆其余的类、字段和方法。混淆过的代码可令您的 APK 难以被逆向工程，这在应用使用许可验证等安全敏感性功能时特别有用。

## 混淆介绍

Android中的“混淆”可以分为两部分，一部分是 Java 代码的优化与混淆，依靠 proguard 混淆器来实现；另一部分是资源压缩，将移除项目及依赖的库中未被使用的资源(资源压缩严格意义上跟混淆没啥关系)。

我们通常说的proguard（代码混淆）包括以下四个方面:
1. shrink（压缩）： 检测并移除没有用到的类，变量，方法和属性；
2. optimize（优化）: 分析和优化代码，优化可能会造成一些潜在风险，不能保证在所有版本的Dalvik上都正常运行；
3. obfuscate（混淆）: 把类名、属性名、方法名替换为简短且无意义的名称，增大反编译难度；
4. preverify（预校验）: 预校验是作用在Java平台上的（校验代码是否符合Java1.6+），Android平台上不需要这项功能，去掉之后还可以加快混淆速度。

这四个流程默认开启，在 Android 项目中我们可以选择将“优化”和“预校验”关闭，对应命令是-dontoptimize、-dontpreverify



## 代码混淆

Android Studio集成了Java语言的ProGuard作为压缩，优化和混淆的工具，使用起来很方便。
首先要通过ProGuard启用代码混淆，首先要在app module下的build.gradle文件将“minifyEnabled”属性设置为true，以便开启代码混淆（开启混淆会使编译时间变长，默认关闭）：
``` java
android {
    buildTypes {
        release {
            minifyEnabled true  // 代码混淆(true为打开，开启混淆会使编译时间变长，默认不开启)
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

在上面的“混淆配置”中有这样一行代码
```
proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
```
proguard-android.txt，这是系统默认的混淆文件，具体在../sdk/tools/proguard/ 目录下，其中包含了android最基本的混淆，一般不需要改动;
我们需要配置的是项目中app下的 proguard-rules.pro 文件


ProGuard作用

-dontshrink 关闭压缩
-optimizationpasses n 表示proguard对代码进行迭代优化的次数，Android一般为5
-dontobfuscate 关闭混淆

混淆后默认会在工程目录app/build/outputs/mapping/release下生成一个mapping.txt文件，这就是混淆规则，我们可以根据这个文件把混淆后的代码反推回源本的代码，所以这个文件很重要，注意保护好。原则上，代码混淆后越乱越无规律越好，但有些地方我们是要避免混淆的，否则程序运行就会出错，所以就有了下面我们要教大家的，如何让自己的部分代码避免混淆从而防止出错。


## 混淆规则

1.基本规则

1. 常见混淆命令：

命令	作用
-keep	防止类和成员被移除或者被重命名
-keepnames	防止类和成员被重命名
-keepclassmembers	防止成员被移除或者被重命名
-keepclasseswithmembers	防止拥有该成员的类和成员被移除或者被重命名
-keepclasseswithmembernames	防止拥有该成员的类和成员被重命名

2. 保持元素不参与混淆的规则

语句结构：[保持命令] [类] { [成员] }

“类”表示符合相关限定条件的类，它的内容可以使用：
- 具体的类
- 访问修饰符（public、protected、private）
- 通配符*，匹配任意长度字符，但不含包名分隔符(.)
- 通配符**，匹配任意长度字符，并且包含包名分隔符(.)
- extends，即可以指定类的基类
- implement，匹配实现了某接口的类
- $，内部类

“成员”表示符合相关限定条件的类成员，它的内容可以使用：
- <init> 匹配所有构造器
- <fields> 匹配所有域
- <methods> 匹配所有方法
- 通配符*，匹配任意长度字符，但不含包名分隔符(.)
- 通配符**，匹配任意长度字符，并且包含包名分隔符(.)
- 通配符***，匹配任意参数类型
- …，匹配任意长度的任意类型参数。比如void test(…)就能匹配任意 void test(String a) 或者是 void test(int a, String b) 这些方法。
- 访问修饰符（public、protected、private）

举个例子，假如需要将name.huihui.test包下所有继承Activity的public类及其构造函数都保持住，可以这样写：
-keep public class name.huihui.test.** extends Android.app.Activity {
    <init>
}
3. 常用的自定义混淆规则

不混淆某个类
-keep public class name.huihui.example.Test { *; }

不混淆某个包所有的类
-keep class name.huihui.test.** { *; }

不混淆某个类的子类
-keep public class * extends name.huihui.example.Test { *; }

不混淆所有类名中包含了“model”的类及其成员
-keep public class **.*model*.** {*;}

不混淆某个接口的实现
-keep class * implements name.huihui.example.TestInterface { *; }

不混淆某个类的构造方法
-keepclassmembers class name.huihui.example.Test { 
    public <init>(); 
}

不混淆某个类的特定的方法
-keepclassmembers class name.huihui.example.Test { 
    public void test(java.lang.String); 
}


再者，如果一个类中你不希望保持全部内容不被混淆，而只是希望保护类下的特定内容，就可以使用

<init>;     //匹配所有构造器
<fields>;   //匹配所有域
<methods>;  //匹配所有方法方法
你还可以在<fields>或<methods>前面加上private 、public、native等来进一步指定不被混淆的内容，如

-keep class cn.hadcn.test.One {
    public <methods>;
}
表示One类下的所有public方法都不会被混淆，当然你还可以加入参数，比如以下表示用JSONObject作为入参的构造函数不会被混淆

-keep class cn.hadcn.test.One {
   public <init>(org.json.JSONObject);
}
有时候你是不是还想着，我不需要保持类名，我只需要把该类下的特定方法保持不被混淆就好，那你就不能用keep方法了，keep方法会保持类名，而需要用keepclassmembers ，如此类名就不会被保持，为了便于对这些规则进行理解，官网给出了以下表格

保留	防止被移除或者被重命名	防止被重命名
类和类成员	-keep	-keepnames
仅类成员	-keepclassmembers	-keepclassmembernames
如果拥有某成员，保留类和类成员	-keepclasseswithmembers	-keepclasseswithmembernames

## 注意事项
1. jni方法不可混淆（jni方法必须要跟native方法一致)；
2. 反射用到的类不混淆(否则反射可能出现问题)；
4. 使用了 Gson 之类的工具要使 JavaBean 类即实体类不被混淆（否则无法将JSON解析成对应的对象）；
5. 添加第三方库所需的混淆规则（一般都在文档中会写混淆规则，使用时注意添加）
6. 有用到WebView的JS调用也需要保证写的接口方法不混淆；
7. Parcelable的子类和Creator静态成员变量不混淆（否则会抛出BadParcelableException异常）；
8. enum类型（枚举）不进行混淆

- 所有在 AndroidManifest.xml 涉及到的类已经自动被保持，因此不用特意去添加这块混淆规则。（很多老的混淆文件里会加，现在已经没必要）
- proguard-android.txt 已经存在一些默认混淆规则，没必要在 proguard-rules.pro 重复添加

关于第三方库的混淆规则，github上有相关的库[android-proguard-snippets](https://github.com/krschultz/android-proguard-snippets)，但已经几年没更新了，所以还是希望自己手动添加


3. 检查混淆结果

混淆过的包必须进行检查，避免因混淆引入的bug。

一方面，需要从代码层面检查。使用上文的配置进行混淆打包后在 <module-name>/build/outputs/mapping/release/ 目录下会输出以下文件：
- dump.txt 
描述APK文件中所有类的内部结构
- mapping.txt
提供混淆前后类、方法、类成员等的对照表
- seeds.txt
列出没有被混淆的类和成员
- usage.txt
列出被移除的代码

我们可以根据 seeds.txt 文件检查未被混淆的类和成员中是否已包含所有期望保留的，再根据 usage.txt 文件查看是否有被误移除的代码。

另一方面，需要从测试方面检查。将混淆过的包进行全方面测试，检查是否有 bug 产生。





## 基本的混淆模板
这里提供一份基本的混淆模板，适用于大部分项目
当然第三方库，或者上面提到的地方，需要根据项目的实际需求进行混淆

```
#############################################
#
# 基础混淆配置
#
#############################################


####################基本混淆指令的设置####################

# 代码混淆压缩比，在0~7之间，默认为5，一般不做修改
-optimizationpasses 5

# 混合时不使用大小写混合，混合后的类名为小写
-dontusemixedcaseclassnames

# 优化时允许访问并修改有修饰符的类和类的成员
-allowaccessmodification

# 指定不忽略非公共库的类
-dontskipnonpubliclibraryclasses

# 指定不忽略非公共库的类成员
-dontskipnonpubliclibraryclassmembers

# 记录日志，使我们的项目混淆后产生映射文件（类名->混淆后类名）
-verbose

# 忽略警告，避免打包时某些警告出现，没有这个的话，构建报错
-ignorewarnings

# 不做预校验，preverify是proguard的四个步骤之一，Android不需要preverify，去掉这一步能够加快混淆速度。
-dontpreverify

# 不混淆Annotation(保留注解)
-keepattributes *Annotation*,InnerClasses

# 避免混淆泛型
-keepattributes Signature

# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable

# 指定混淆是采用的算法，后面的参数是一个过滤器
# 这个过滤器是谷歌推荐的算法，一般不做更改
-optimizations !code/simplification/cast,!field/*,!class/merging/*


####################Android开发中需要保留的公共部分####################

# 保留support下的所有类及其内部类
-keep class android.support.** {*;}
# 保留继承的support类
-keep public class * extends android.support.v4.**
-keep public class * extends android.support.v7.**
-keep public class * extends android.support.annotation.**

# 保留R下面的资源
-keep class **.R$* {*;}

# 保留本地native方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保留Activity中参数类型为View的所有方法
-keepclassmembers class * extends android.app.Activity{
    public void *(android.view.View);
}

# 保留枚举类不被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保留Parcelable序列化类不被混淆
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

# 保留Serializable序列化的类不被混淆
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# 保留我们自定义控件（继承自View）不被混淆
-keep public class * extends android.view.View{
    *** get*();
    void set*(***);
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

# 对于带有回调函数的onXXEvent、**On*Listener的，不能被混淆
-keepclassmembers class * {
    void *(**On*Event);
    void *(**On*Listener);
}

# webView的混淆处理
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



#############################################
#
# 自定义的混淆配置（根据项目需求进行定义）
#
#############################################

# 不混淆log
-assumenosideeffects class android.util.Log {
    public static boolean isLoggable(java.lang.String, int);
    public static int v(...);
    public static int i(...);
    public static int w(...);
    public static int d(...);
    public static int e(...);
}



#############################################
#
# 第三方库的混淆配置（根据第三方库官网添加混淆代码）
#
#############################################

# Gson
-keepattributes *Annotation*
-keep class sun.misc.Unsafe { *; }
-keep class com.idea.fifaalarmclock.entity.***
-keep class com.google.gson.stream.** { *; }
-keep class com.你的bean.** { *; }


# OkHttp3
-dontwarn okhttp3.logging.**
-keep class okhttp3.internal.**{*;}
-dontwarn okio.**


# Retrofit
-dontwarn retrofit2.**
-keep class retrofit2.** { *; }
-keepattributes Signature
-keepattributes Exceptions


# RxJava RxAndroid
-dontwarn sun.misc.**
-keepclassmembers class rx.internal.util.unsafe.*ArrayQueue*Field* {
    long producerIndex;
    long consumerIndex;
}
-keepclassmembers class rx.internal.util.unsafe.BaseLinkedQueueProducerNodeRef {
    rx.internal.util.atomic.LinkedQueueNode producerNode;
}
-keepclassmembers class rx.internal.util.unsafe.BaseLinkedQueueConsumerNodeRef {
    rx.internal.util.atomic.LinkedQueueNode consumerNode;
}

# Glide图片库
-keep class com.bumptech.glide.**{*;}

```
