---
date: 2015-09-15
draft: false
#slug: "slug"
tags: [IDE]
categories: [Android]
title: "加速Android开发"
weight: 10
---

## IDEA 技巧

#### 使用 Action

快捷键为 `cmd+shift+a`，在弹出的 Action 对话框中可以输入任意 Action，比如说开启自动导入包的功能 `auto import`。

#### 搜索

- `cmd+n` 搜索所有类文件
- `shift+cmd+n` 搜索所有文件
- `shift+shift` 在任何地方进行搜索
- `cmd+f12` 大纲

#### 替换

当一个对象调用一个方法时（如 a.foo()），如果你需要替换当前方法的话，通常做法是在 `.` 后根据提示追加新方法（如 a.bfoo()），此时按空格的话提示会直接追加到当前位置(a.barfoo())，而使用 tab 键则会进行替换（a.bar()）。

#### IDEA Live Template

Live Template 可以减少键入的字母数量，比如说可以只键入 `logd` 就表示输入 `Log.d(TAG, String)`。

![][01]

定义适合 Android 的 Live Template 可能会花费不少时间。所幸的是 Github 上现在已经有创建好的适合 Android 开发的 Live Template。

你所要做的就是访问这个链接 [idea-live-templates](https://github.com/keyboardsurfer/idea-live-templates)，将所有模板下载下来，放到对应平台的 templates 文件夹下，重启 IDEA 就可以了。

对应平台的 templates 目录路径

- **Windows:** `<your home directory>\.<product name><version number>\config\templates`
- **Linux:** `~/.<product name><version number>/config/templates`
- **OS X:** `~/Library/Preferences/<product name><version number>/templates`

比如说我使用的是 Mac 平台的 IDEA 14，那么我的 templates 目录就在 `/Users/sidneyxu/Library/Preferences/IdeaIC14/templates` 下。

IDEA LiveTemplate 不仅可以直接用于代码补完，也可以作用在代码上，比如说要遍历一个列表，正确的做法应该是输入 `列表名.` 后按下 `cmd+j` 后输入 `for` 即可自动生成 `for-each` 形式的代码块。

默认的几个 Live Template

- for 循环：`obj.for`, `obj.fori`
- 检查空值：`obj.null`, `obj.nn`

#### 查看模块依赖

在界面左下角的 `Build Variants` 中点击感叹号就能查看所选模块依赖了哪些模块

#### 删除无用的资源

选择工程，右键 `Refactor` -> `Remove Unused Resources...` 可以将无用的资源进行删除

## PID Cat

普通的 `adb logcat` 命令打印出的 log 信息全是白色的一堆，很难从中找到有用的信息。PID Cat 可以为 log 信息进行着色，以便于查看。

安装教程见 [pidcat](https://github.com/JakeWharton/pidcat)

效果如下

![][02]

## Peco

Peco 是一个使用 Go 编写的命令行工具，它可以用于进行各种交互式的过滤操作。通过使用它我们可以提高 `adb` 命令的使用效率。

Peco 的安装可以参照 Github 上的说明：[peco](https://github.com/peco/peco)

#### 快速卸载指定应用

在命令行中输入以下内容

``` bash
alias uninstallapp='adb shell pm list package | sed -e s/package:// | peco | xargs adb uninstall'
```

之后只要执行 `uninstallapp` 就可以选择卸载指定的应用

![][03]

#### 快速安装应用

在命令行中输入以下内容

``` bash
alias installapp="find ./ -name '*.apk' | peco | xargs adb install"
```

之后执行 `installapp` 就会列出当前电脑中所有的 apk 文件，可以选择需要的进行安装

![][04]

## 截图

在命令行中输入以下内容

``` bash
alias screenshot='screenshot2 $TMPDIR/screenshot.png; open $TMPDIR/screenshot.png'
```

之后执行 `screenshot` 就会在 `$TMPDIR` 目录生成截图文件并打开

## nimbledroid

[nimbledroid](https://nimbledroid.com/) 是一个在线 APK 分析网站，可以分析 APK 中的各种文件的大小，方法数量，占用内存大小等等。不过因为是在线网站，所以只能为了安全起见，不要上传还在开发中的应用。

## Stetho

Stetho 可以用于通过 Chrome 查看当前应用布局，使用文档见 [http://facebook.github.io/stetho/](http://facebook.github.io/stetho/)

使用

在 `build.gradle` 中添加依赖

```groovy
compile 'com.facebook.stetho:stetho:1.4.1'
compile 'com.facebook.stetho:stetho-okhttp3:1.4.1'
```

在 Application 中进行初始化

```java
Stetho.initializeWithDefaults(this)
```

## 方法统计

#### 统计 APK 方法数量

访问 [APK method count](http://inloop.github.io/apk-method-count/)，直接将apk 文件拖到中间区域就可以直接显示对应的代码行数。

![][05]

#### 统计依赖的方法数量

安装 `IDEA` 或 `Android Studio` 插件 `Android Methods Count` ([官网](http://www.methodscount.com/))后就可以在 `build.gradle` 文件中直接查看方法数。

![][07]

### 动画预览

访问 [interpolator](http://inloop.github.io/interpolator/) 可以直接修改对应的值来预览对应的动画效果。

![][06]

### 窗口投影

[Droid](http://droid-at-screen.org/download.html) 和 [android-screen-monitor](https://github.com/adakoda/android-screen-monitor) 都可以将当前手机屏幕投影到电脑屏幕上，尽管效果上会有一些延迟，不过已经基本满足做手机应用简单演示的需求了。



## WebView 的 Debug

WebView 通过 Chrome 进行 Debug，要求手机系统在 Android 4.4 以上，并且 Chrome 升级到最新版本（如果无法看到 WebView 的内容通常都是 Chrome 版本太低）。

首先需要在代码中插入以下内容

``` java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
   webView.setWebContentsDebuggingEnabled(true);
}
```

然后在电脑上的 Chrome 地址栏输入 `chrome://inspect` ，接着就能像调试普通的 Web 页面一样调试 WebView 里的页面了。

### UI 的 Debug

开源项目 [Scalpel](https://github.com/JakeWharton/scalpel) 可以以 3D 的方式显示界面的层次，方便进行 UI 的 Debug。

使用时需要将自己的 layout 包裹在 `ScalpelFrameLayout` 中

```xml
<com.jakewharton.scalpel.ScalpelFrameLayout
    android:id="@+id/scalpel"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    ...
</com.jakewharton.scalpel.ScalpelFrameLayout>
```

然后在代码中调用以下代码显示 3D 效果

```java
scalpelFrameLayout.setLayerInteractionEnabled(true);
scalpelFrameLayout.setDrawViews(true);
scalpelFrameLayout.setDrawIds(true);
```


[01]: http://7xlqqp.com1.z0.glb.clouddn.com/livetemplate.png
[02]: http://7xlqqp.com1.z0.glb.clouddn.com/pidcat.png
[03]: http://7xlqqp.com1.z0.glb.clouddn.com/uninstallapp.png
[04]: http://7xlqqp.com1.z0.glb.clouddn.com/installapp.png
[05]: http://7xlqqp.com1.z0.glb.clouddn.com/methodcount.png
[06]: http://7xlqqp.com1.z0.glb.clouddn.com/interpolator.png
[07]: http://www.methodscount.com/images/methods-count-plugin-1.png
