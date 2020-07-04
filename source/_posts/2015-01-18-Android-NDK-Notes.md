title: Android NDK开发笔记
date: 2015-01-18
tags:
- Android
- NDK
- JNI
permalink: Android-Ndk-Notes
---

在Android开发中需要实现一个加解密功能，为了增加被逆向的难度，决定使用JNI实现该功能。使用C/C++编写代码，编译链接成`.so`动态库，提供给Java调用。本文简单总结了Eclipse中Android NDK环境的搭建过程，编写了一个JNI的小例子，记录了一些开发中遇到的问题。

## 一、Android NDK与JNI简介

Android NDK（Native Development Kit ）是一套工具集合，允许用C/C++语言实现应用程序的一部分功能。

JNI（Java Native Interface）是Java的一种机制，允许使用其他语言（汇编、C/C++等）编写代码，并封装成动态库被Java调用。也就是说JNI提供了一个在Java平台上调用C/C++的一种途径，NDK是Android对JNI实现的一种支持。

关于JNI的理解，请参考[《JAVA基础之理解JNI原理》][1]，文中的图片介绍简单直观：
![JNI][2]

## 二、Eclipse NDK环境搭建

准备工作：
* 下载 [Eclipse][3]、[Eclipse CDT插件][4]
* 下载 [Android SDK][5]、[Android NDK][6]、[Android ADT插件][7]

1. 解压Eclipse，同时安装Android SDK和NDK；
2. 安装CDT：Eclipse `help`->`Instlal New Software...`，选择已下载的CDT；
3. 安装ADT：Eclipse `help`->`Instlal New Software...`，选择已下载的ADT，安装过程中确认选中了`NDK Plugin`；
4. 配置SDK/NDK：Eclipse `Windows`->`Preferences`->`Android`，配置SDK路径，在`Android`的`NDK`标签下配置NDK路径。

## 三、JNI开发示例

开发环境：Windows 7 SP1 x64，Eclipse LUNA，JDK 1.7，CDT 8.5.0，ADT 23.0.4

### 1) 新建Android项目

1. Eclipse新建Android项目，这里命名为`jnitest`。
![New Android Application][8]

2. 我们使用`TextView`显示JNI的调用结果，修改`res`->`layout`目录下的`activity_main.xml`，给默认的`TextView`添加一个id：`tvTest`，方便以后在`MainActivity`中调用。
```xml
<TextView
    android:id="@+id/tvTest"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/hello_world" />
```
### 2) 添加Native支持

`jnitest`项目右键`Android Tool`->`Add Native Support...`，这里为动态库命名为`doit`，然后在Android工程目录中会生成一个`jni`目录，并生成`Android.mk`和`doit.cpp`文件。
![Add Android Native Support][9]

`Android.mk`文件的功能类似GNU Makefile，详细了解请参考[《Android.mk的用法和基础》][10]。打开`Android.mk`，从下面两行代码可以看出，我们要编译的模块名字是`doit`，编译使用的源文件是`doit.cpp`：
```cpp
LOCAL_MODULE    := doit
LOCAL_SRC_FILES := doit.cpp
```

如果添加了更多的`.cpp`源文件，需要在`Android.mk`中`LOCAL_SRC_FILES`后添加对应的源文件。

另外需要注意的地方，就是动态库的名字。在给动态库命名时我们只写了`doit`，Eclipse默认给添加了`lib`和`.so`，也就是说动态库的名字应该是`libdoit.so`。这是因为linux的动态库命名必须以`lib`开头，即`libXXX.so`。`XXX`是动态库的名字，我们在Java代码中加载动态库的时候指定名字是`XXX`而不是`libXXX`。

当我们编写完C/C++代码并编译链接后，会在项目的`libs\armeabi`目录下生成`libdoit.so`。

### 3) 添加JNI类

为Android项目添加一个Java类，这里命名为`JNITest`：
![New Java Class][11]

`JNITest`类指定我们要加载的`libdoit.so`动态库并声明Native方法。这个Native方法就是我们接下来要使用C/C++实现的方法。

JNITest类的实现如下：
```java
package com.example.jnitest;

public class JNITest {
	static {
		System.loadLibrary("doit");
	}

	public native String DoIt(String s);
}

```

第5行使用`System.loadLibrary("doit");`指定加载`libdoit.so`，第8行声明了`DoIt()`的Native方法，参数和返回值都是字符串类型。

如果只是简单地演示，完全可以在`MainActivity.java`里加载动态库和声明Native方法。这里编写了一个`JNITest`类，是为了更好地扩展，以便在其他项目里也能使用这个类来调用生成的动态库。

### 4) 实现C/C++方法

打开`jni`下的`doit.cpp`，添加如下代码：
```cpp
extern "C" {
    JNIEXPORT jstring Java_com_example_jnitest_JNITest_DoIt(JNIEnv* env, jobject obj, jstring s);
}

JNIEXPORT jstring Java_com_example_jnitest_JNITest_DoIt(JNIEnv* env, jobject obj, jstring s)
{
    // 具体实现
    return env->NewStringUTF("Good Job!");
}
```

函数说明：

* 返回值：`jstring`是函数的返回值类型，如果没有返回值，写`void`，如果返回值是`int`型，写`jint`，即在C/C++类型前加`j`。
* 函数名`Java_com_example_jnitest_JNITest_DoIt`：函数名要以`_`连接，`Java`是必需的，`com_example_jnitest`是`JNITest`类的包名，也就是我们创建Android项目时命名的包名，`JNITest`是声明Native方法的类名，`DoIt`是Native方法名。
* 函数的参数：`JNIEnv* env`和`jobject obj`是实现Native方法必需的两个参数，我们在`JNITest`类中声明的Native方法只有一个`String`类型的参数，在这里要添加到这两个参数后面，使用`jstring`类型。
* 因此，C/C++实现JNI方法时函数的命名规则是：
```cpp
JNIEXPORT 返回值类型 Java_包名_类名_方法名(JNIEnv* env, jobject obj, 自定义参数...);
```

### 5) 新建C/C++ Builder

为了更方便地编译C/C++，我们新建一个Builder：

1. `jnitest`项目右键`Properties`->`Builders`->`New`，选择`Program`，点击`OK`，我们把它命名为`NDK_Builder`。
2. Main选项卡：
在`Location:`中点击`Browse File System...`选择Android NDK目录下的`ndk-build.cmd`；
在`Working Directory:`中输入`${workspace_loc}\jnitest`，或者通过`Browse File System...`选择项目所在目录。
![Builder Main Option][12]
3. Build Options选项卡：
默认勾选了`Allocate Console (necessary for input)`、`After a "Clean"`、`During manual builds`；
勾选`During auto builds`和`During a "Clean"`；
也可以勾选`Specify working set of relevant resources`并点击`Specify Resources…`指定项目。
![Builder Optoins][13]
4. 点击`OK`完成配置。

这样，以后每次点击`Project`->`clean...`就直接编译链接C/C++了，在项目的`libs\armeabi`下会自动生成`libdoit.so`动态库。

### 6) Android代码实现

接下来我们在Android的`MainActivity.java`中实例化`JNITest`类并把`DoIt()`方法返回的字符串显示到Android的`TextView`中：
```java
// MainActivity.java

package com.example.jnitest;

import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.TextView;

public class MainActivity extends ActionBarActivity {

	private TextView tv;                 // TextView
	private String s = "Just Do It";     // DoIt()的参数
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		tv = (TextView) findViewById(R.id.tvTest);

		JNITest jnitest = new JNITest(); // 实例化JNITest类
		tv.setText(jnitest.DoIt(s));     // 调用Native方法
	}

    // 其他代码
}
```

`jnitest`项目右键`Run As`->`Android Application`，使用模拟器或者手机来测试一下吧。
![模拟器运行结果][14]

### 7) 更复杂一点

上面的JNI小程序已经结束了。但在实际开发中可不会实现这么简单的功能，复杂的功能可能需要添加多个C/C++源文件和头文件。这里，我们演示JNI中添加`.cpp`和`.h`，在Android的SDCard上创建文件。

1.项目右键`New`->`Header File`，新建C/C++头文件`other.h`，在头文件中声明函数：
```cpp
int createfile(const char *filename);
```

2.项目右键`New`->`Source File`，新建C++源文件`other.cpp`，编写代码：

```cpp
#include "other.h"
#include <stdio.h>

int createfile(const char *filename)
{
	FILE *file;
	if ((file = fopen(filename, "wb")) == NULL)
		return -1;
	fclose(file);
	return 0;
}
```

3.在`doit.cpp`中引用`other.h`，调用`createfile()`：
```cpp
#include <jni.h>
#include "other.h"

extern "C" {
    JNIEXPORT jstring Java_com_example_jnitest_JNITest_DoIt(JNIEnv* env, jobject obj, jstring s);
}

JNIEXPORT jstring Java_com_example_jnitest_JNITest_DoIt(JNIEnv* env, jobject obj, jstring s)
{
	// 把jstring转换为char *
	const char *filename = env->GetStringUTFChars(s, 0);
    createfile(filename);
    return env->NewStringUTF("Good Job!");
}
```

4.非常重要的一步：在`Android.mk`中添加`other.cpp`，否则编译链接的时候会出现` error: undefined reference to xxx`的错误：
```cpp
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE    := doit
LOCAL_SRC_FILES := doit.cpp \
                   other.cpp

include $(BUILD_SHARED_LIBRARY)
```

5.`Project`->`Clean...`编译链接生成动态库。

6.这里涉及到操作Android SDCard，需要在项目的`AndroidManifest.xml`中添加相应权限：
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
```
另外，修改`MainActivity.java`，给Native方法`DoIt(String s)`传递参数指定创建文件的名字：
```java
public class MainActivity extends ActionBarActivity {

	private TextView tv;
	private String s = "/sdcard/JustDoIt.txt";
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		tv = (TextView) findViewById(R.id.tvTest);

		JNITest jnitest = new JNITest();
		tv.setText(jnitest.DoIt(s));
	}
```

项目右键`Run As`->`Android Application`，启动模拟器或连接手机，查看SD卡上是否成功创建了文件。
![模拟器运行结果][15]

### 8) 示例程序下载

百度网盘: http://pan.baidu.com/s/1dD2DNhR 密码: zgvz

## 三、JNI技巧

### 技巧1. JNI实现函数中LOG打印

在Java环境下使用JNI时可以方便的使用printf函数打印信息，在Eclipse控制台Console视图可以方便的观察到，可在Android环境下使用JNI的话，printf函数就无效了，LogCat视图和Console视图里看不到任何输出。但我们可以使用Android本身的log方法，其实现步骤如下：

1.在JNI的实现代码文件（.c或者.cpp）中加入包含LOG头文件的如下代码：
```cpp
#include <android/log.h>
```

2.在需要打印的方法中添加打印代码，例如：
```cpp
__android_log_print(ANDROID_LOG_INFO, "jnitag", "Content");
```

`ANDROID_LOG_INFO`是日志级别；
`jnitag`是要过滤的标签，可以在LogCat视图中过滤；
`Content `是实际的日志内容。

3.在Android工程的Android.mk文件中添加如下内容：
```cpp
LOCAL_LDLIBS += -L$(SYSROOT)/usr/lib -llog
```

4.OK，现在就可以打印信息了。

### 技巧2. Converting jstring to char *

以下是C语言实现：
```cpp
JNIEXPORT void JNICALL Java_ClassName_MethodName(JNIEnv *env, jobject obj, jstring javaString)  
{
   const char *nativeString = (*env)->GetStringUTFChars(env, javaString, 0);
 
   // use your string
 
   (*env)->ReleaseStringUTFChars(env, javaString, nativeString);
}
```

如果使用C++，应该这样（env不加`*`，`GetStringUTFChars()`等方法少一个参数`env`）：
```cpp
JNIEXPORT void JNICALL Java_ClassName_MethodName(JNIEnv *env, jobject obj, jstring javaString)  
{
   const char *nativeString = env->GetStringUTFChars(javaString, 0);
 
   // use your string
 
   env->ReleaseStringUTFChars(javaString, nativeString);
}
```

## 四、JNI问题

### 问题1. 导入Android工程时提示“XXX overlaps the location of another project: 'XXX'”

导入时不要选择导入android工程，而是General工程。

### 问题2. 删除Eclipse的`.metadata`文件夹NDK工程提示`Program "sh" not found in PATH`

重新设置一下Eclipse：`Windows`->`Preferences`->`Android`->`NDK`，然后重启Eclipse。

### 问题3. 更新eclipse或者删除eclipse的`.metadata`文件夹导致找不到`jni.h`等头文件

参考：http://stackoverflow.com/questions/23122934/eclipse-adt-unresolved-inclusion-jni-h

**Removing the C nature:**

1.Close the Eclipse project (e.g. by quitting Eclipse).

2.Open the `.project` file in a text or xml editor. There will be at least 2 `<buildCommand>` nodes that need to be removed. Remove the `<buildCommand>` node with name` org.eclipse.cdt.managedbuilder.core.genmakebuilder` and all its children, and the `<buildCommand>` node with name `org.eclipse.cdt.managedbuilder.core.ScannerConfigBuilder` and its children. Finally, remove the lines:
```xml
<nature>org.eclipse.cdt.core.cnature</nature>
<nature>org.eclipse.cdt.core.ccnature</nature>
<nature>org.eclipse.cdt.managedbuilder.core.managedBuildNature</nature>
<nature>org.eclipse.cdt.managedbuilder.core.ScannerConfigNature</nature>
```

3.Completely remove the `.cproject` file.

**Adding back the Android Native nature**

Reopen the project in Eclipse. Then right-click on the project in the Project Explorer, and from the "Android Tools" contextual menu, choose "Add Native Support...".

### 问题4. Type 'jint' could not be resolved, and JNIEnv, jclass

参考：http://stackoverflow.com/questions/11666711/type-jint-could-not-be-resolved-and-jnienv-jclass

其实还是找不到`jni.h`，删除项目`.project`文件中的`<nature>org.eclipse.cdt.core.ccnature</nature>`，重启Eclipse。

## 五、参考文章

http://www.cnblogs.com/mandroid/archive/2011/06/15/2081093.html
http://blog.csdn.net/zhandoushi1982/article/details/5316669
http://blog.csdn.net/zhangjg_blog/article/details/15505781
http://weizhilizhiwei.iteye.com/blog/2157773


  [1]: http://www.cnblogs.com/mandroid/archive/2011/06/15/2081093.html
  [2]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-01-17_170900.webp "理解JNI原理"
  [3]: https://eclipse.org/downloads/
  [4]: https://eclipse.org/cdt/
  [5]: http://developer.android.com/sdk/index.html#Other
  [6]: http://developer.android.com/tools/sdk/ndk/index.html
  [7]: http://developer.android.com/tools/sdk/eclipse-adt.html
  [8]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-01-17_213510.webp "New Android Application"
  [9]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-01-17_224538.webp "Add Android Native Support"
  [10]: http://blog.csdn.net/zhandoushi1982/article/details/5316669
  [11]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-01-17_230000.webp "New Java Class"
  [12]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-01-17_233948.webp "Builder Main Option"
  [13]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-01-17_234157.webp "Builder Option"
  [14]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-01-18_001951.webp "模拟器运行结果"
  [15]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-01-18_115549.webp "模拟器运行结果"
