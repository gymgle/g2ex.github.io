title: VC6多线程编程
date: 2014-10-09 
tags:
- C++
permalink: VC6-Multi-Thread-Program
---

## 一、问题的提出

编写一个耗时的单线程程序：在Visual C++ 6.0中新建一个基于对话框的MFC项目，这里取名「Demo」。

![基于对话框的MFC项目][1]

删掉主对话框`IDD_DEMO_DIALOG`上的控件，添加一个按钮和文本框。按钮的ID和标题设置为`IDD_DEMO_DIALOG`和`测试`，文本框的ID设置为`IDC_EDT_OUT`。

双击`测试按钮`为按钮添加响应函数`OnBtnTest()`，用于计算从1累加到100的和，为了看起来耗时，在每次循环中加入0.1秒的延时。

```cpp
void CDemoDlg::OnBtnTest() 
{
	GetDlgItem(IDC_EDT_OUT)->SetWindowText("计算开始...");
	int sum = 0;
	CString str;
	for (int i = 1; i <= 100; ++i)
	{
		sum += i;
		Sleep(100);	// 延时0.1秒
	}
	str.Format("计算结果: %d", sum);
	GetDlgItem(IDC_EDT_OUT)->SetWindowText(str);
}
```

编译并运行应用程序，单击`测试`按钮，会发现在这10秒多期间内程序卡死了，不再响应其它消息。为了更好地处理这种耗时的操作，我们有必要使用多线程编程。

![程序卡死][2]

## 二、如何使用多线程

下面在MFC中使用多线程完成1到100的累加计算。程序运行时，会自动创建一个Demo主线程，点击测试按钮后主线程启动一个计算线程执行`ThreadFunc()`函数，计算线程计算完成后发送`WM_USER_THREAD_FINISHED`消息，最后，主线程使用`OnThreadFinished()`函数处理`WM_USER_THREAD_FINISHED`消息并把结果显示在文本框中。

实现步骤如下：

### 1) 定义参数传递结构体

在`DemoDlg.h`文件中`CDemoDlg`类的外部定义两个结构体，第一个用于主线程创建计算线程时向计算线程传递参数：

```cpp
typedef struct tagTHREADPARMS {
	int BeginNumb;	// 起始数字
	int EndNumb;	// 结束数字
	HWND hWnd;
} THREADPARMS;
```

当计算线程计算完成后向主线程发送消息时，使用下面的结构体：

```cpp
typedef struct tagRESULTPARMS {
	int Sum;	// 计算结果
	HWND hWnd;
} RESULTPARMS;
```

如果线程间不需要参数传递或者传递的参数很少，可以不使用结构体。

### 2) 声明线程函数

在`DemoDlg.h`文件中`CDemoDlg`类的外部添加线程函数声明：

```cpp
UINT ThreadFunc(LPVOID lpParam);
```

### 3) 编写线程函数

在`DemoDlg.cpp`文件中编写计算线程函数。其计算结果是一个整形数字，完全可以用返回值的形式返回。为了应对更复杂的参数传递，这里演示用结构体传递参数，把结果以消息的形式发送给消息处理函数（消息处理函数后续添加）。

```cpp
UINT ThreadFunc(LPVOID lpParam)
{
	// 获取传递过来的参数
	THREADPARMS* ptp = (THREADPARMS*) lpParam;
	HWND hWnd = ptp->hWnd;
	int m = ptp->BeginNumb;
	int n = ptp->EndNumb;
	int sum = 0;

	// 构造发送消息的参数
	RESULTPARMS* prp = new RESULTPARMS;
	prp->hWnd = hWnd;

	for (int i = m; i <= n; ++i)
	{
		sum += i;
		Sleep(100);	// 延时0.1秒
	}
	prp->Sum = sum;

	// 发送 WM_USER_THREAD_FINISHED 消息
	::PostMessage (hWnd, WM_USER_THREAD_FINISHED, (WPARAM) prp, 0);
	
	return 0;
}
```

### 4) 创建线程函数

在按钮事件`OnBtnTest()`中使用`AfxBeginThread()`创建线程：

```cpp
void CDemoDlg::OnBtnTest() 
{
	// 构造启动线程的参数
	THREADPARMS* ptp = new THREADPARMS;
	ptp->hWnd = m_hWnd;
	ptp->BeginNumb = 1;
	ptp->EndNumb = 100;

	// 启动计算线程
	AfxBeginThread (ThreadFunc, ptp);

	GetDlgItem(IDC_EDT_OUT)->SetWindowText("计算开始...");
}
```

### 5) 消息处理函数

计算线程发送结果的`WM_USER_THREAD_FINISHED`消息怎么来处理呢？

首先，在`DemoDlg.h`文件中定义`WM_USER_THREAD_FINISHED`消息：

```cpp
#define WM_USER_THREAD_FINISHED WM_USER+0x100
```

然后，在`DemoDlg.h`文件`CDemoDlg`类的`protected`中添加消息函数的声明，把声明放到`DECLARE_MESSAGE_MAP()`行的上面。

```cpp
afx_msg LONG OnThreadFinished(WPARAM wParam, LPARAM lParam);
```

这时就可以把`WM_USER_THREAD_FINISHED`和`OnThreadFinished()`对应起来了：在`DemoDlg.cpp`中，找到`BEGIN_MESSAGE_MAP(CDemoDlg, CDialog)`，在`END_MESSAGE_MAP()`结束之前加入消息映射：

```cpp
ON_MESSAGE (WM_USER_THREAD_FINISHED, OnThreadFinished)
```

最后，在`DemoDlg.cpp`文件中编写`OnThreadFinished()`函数：

```cpp
void CDemoDlg::OnBtnTest() 
{
	// 构造启动线程的参数
	THREADPARMS* ptp = new THREADPARMS;
	ptp->hWnd = m_hWnd;
	ptp->BeginNumb = 1;
	ptp->EndNumb = 100;

	// 启动计算线程
	AfxBeginThread (ThreadFunc, ptp);

	GetDlgItem(IDC_EDT_OUT)->SetWindowText("测试开始");
}
```

### 6) 多线程运行结果

编译运行，对话框可以随意拖动不再卡了，使用`Process Hacker`查看`Demo.exe`进程，可以发现当点击`测试按钮`时，它会创建一条新线程。试着多点几次测试，会发生什么？

![多线程运行][3]

## 三、参考内容

关于多线程详细的内容，请参考 http://www.cnblogs.com/TenosDoIt/archive/2013/04/15/3022092.html 或者善用搜索引擎。

## 四、源码下载

Demo源码百度网盘下载: http://pan.baidu.com/s/1qW5kwsk 密码: 3u5i


  [1]: https://i.imgur.com/K23Z4Eh.png "创建基于对话框的MFC项目"
  [2]: https://i.imgur.com/l9Dv9Jr.png "程序卡死"
  [3]: https://i.imgur.com/bwmEU6x.png "多线程运行效果"