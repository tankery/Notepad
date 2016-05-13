---
layout: post
title: 记一次混淆App导致的崩溃
description: 一般的问题都是某反射方法找不到，或者序列化失败等等，但这次，居然涉及到ART特有的机制，和 ProGuard 的bug，也是离奇了。
categories: development
tags:
  - Proguard
  - 混淆
  - ART
  - AbstractMethodError
  - Before Android 4.1
comments: true
mathjax: null
featured: true
published: true
---

自测没有问题的App，在测试那里出现了必现的Crash。检查 CallStack 后，觉得可能是混淆问题，然而名称虽然被混淆，但并没有被删除，应该是没有问题的。

<!-- more -->

CallStack 如下：

```
FATAL EXCEPTION: main
Process: com.mobvoi.ticwear.dialer, PID: 2541
java.lang.AbstractMethodError: abstract method "android.view.View dialer.TS.z(java.util.List)"
	at dialer.TS.a(HeaderScrollingViewBehavior.java:57)
	at ticwear.design.widget.AppBarLayout$ScrollingViewBehavior.a(AppBarLayout.java:1416)
	at ticwear.design.widget.CoordinatorLayout.onMeasure(CoordinatorLayout.java:778)
	at android.view.View.measure(View.java:17668)
```

查看 `HeaderScrollingViewBehavior.java` 得知是一个 abstract method 的问题。

HeaderScrollingViewBehavior 的实现如下：

``` java
abstract class HeaderScrollingViewBehavior extends ViewOffsetBehavior<View> {

    ...

    @Override
    public boolean onMeasureChild(CoordinatorLayout parent, View child, ...) {

        ...

        final View header = findFirstDependency(dependencies);

        ...

    }

    abstract View findFirstDependency(List<View> views);
}
```

其实现子类实现如下：

``` java
public static class ScrollingViewBehavior extends HeaderScrollingViewBehavior {
    @Override
    View findFirstDependency(List<View> views) {
        ...
    }
}
```

`ScrollingViewBehavior` 继承 `HeaderScrollingViewBehavior`， 最终继承了 `CoordinatorLayout.Behavior`。

ProGuard 有这样的规则：

``` ProGuard
-keep public class * extends ticwear.design.widget.CoordinatorLayout$Behavior {
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>();
}
```

查看 ProGuard 的 mapping 文件，发现 `ScrollingViewBehavior` 没混淆，而 `HeaderScrollingViewBehavior` 被混淆了。

这也是正常的，因为根据 ProGuard 规则，确实应该这样。而且 `findFirstDependency` 混淆后的签名一致，仍然符合正确的继承关系，不应该出现 `AbstractMethodError` 的问题。

直到发现 Crash 的 App 有这样的 warning log：

> Before Android 4.1, method "..." would have incorrectly overridden the package-private method in "..."

查看源码，发现是函数的包访问权限出错了：

``` cpp
if (klass->CanAccessMember(super_method->GetDeclaringClass(),
                           super_method->GetAccessFlags())) {
  if (super_method->IsFinal()) {
    ThrowLinkageError(klass.Get(), "Method %s overrides final method in class %s",
                      PrettyMethod(virtual_method).c_str(),
                      super_method->GetDeclaringClassDescriptor());
    return false;
  }
  vtable->SetWithoutChecks<false>(j, virtual_method);
  virtual_method->SetMethodIndex(j);
} else {
  LOG(WARNING) << "Before Android 4.1, method " << PrettyMethod(virtual_method)
               << " would have incorrectly overridden the package-private method in "
               << PrettyDescriptor(super_method->GetDeclaringClassDescriptor());
}
```

于是再次查看 mapping 文件，有这么两个转换：

```
ticwear.design.widget.HeaderScrollingViewBehavior -> dialer.TS
ticwear.design.widget.AppBarLayout$ScrollingViewBehavior -> ticwear.design.widget.AppBarLayout$ScrollingViewBehavior
```

也就是说， `ScrollingViewBehavior` 由于被 keep，保持了原有的包名，而 `HeaderScrollingViewBehavior` 的包名被混淆了。

这个包名的混淆，是我们添加的一个 ProGuard 规则的作用，目的是更好的防止反编译，和减小包体积：

``` ProGuard
-repackageclasses 'dialer'
-allowaccessmodification
```

父类的包名被修改，而继承的又是包访问域的方法。导致这个 crash。

需要说明的是，这个问题只有在打开ART的机器上才会导致crash，应该是ART做了更严格的检查。老的VM是不会有crash的。

搜索到的一些结果，也说跟 ART 有关系，解决方案就是不做混淆。比如[这篇文章](https://github.com/JodaOrg/joda-time/issues/207)。

但还有另一个情况。我们的构建系统有两套。一套是应用层的构建，使用 Gradle；另一套是系统的构建，使用 Android Make。实测发现只有 Make 构建出来的 APK 会 crash。事情变得更蹊跷了。

我们对比了 make 使用的系统默认 ProGuard 规则，和 gradle 使用的 Android SDK 的 ProGuard 规则，发现只有些许细微差别，而且使用相同规则构建出来的两个 APK，也还是同样只在 Make 上会 crash。

于是尝试了反编译两者构建出来的 APK。终于发现了一个区别：

- Gradle 生成的 APK，其父类 `HeaderScrollingViewBehavior` 的包名虽然被修改，但访问权限也随之修改为 public，使得不同包名的子类仍然可以正确的继承。
- Make 生成的 APK，访问权限没有修改，也就是说，`-allowaccessmodification` 貌似没有生效。

到这里，大致就可以确定是 Android Make 所使用的 ProGuard 存在bug，也许升级就能解决问题了。

不过这个改动比较大，目前来看，还是禁用 `-repackageclasses` 规则比较保险。
