title: "Android NFC 开发简介与示例"
date: 2016-02-18 18:00:00
tags: 
- Android
- NFC
permalink: Android-NFC-Program
---

本文总结Android NFC开发的一般过程，最开始介绍几个概念，然后介绍NFC事件的调度过程，最后通过示例代码说明如何进行NFC开发。至于Android NFC P2P模式开发、模拟卡开发请阅读赵波编著的《Android NFC开发实战详解》。

## 一、Android NFC 开发中的几个类

`NfcManager`：用来管理Android设备中所有NFC Adapter。因为大部分Android设备只支持一个NFC Adapter，因此可以直接使用getDefaultAdapter来获取系统的NfcAdapter。

`NfcAdapter`：手机的NFC硬件设备，更专业的称呼是NFC适配器。该类可以用来定义一个Intent使系统在检测到NFC Tag时通知用户定义的Activity，并提供用来注册Foreground Tag消息发送的方法等。如果不清楚`Tag`和`Foreground`是什么，请继续往下看。

`Tag`：代表一个被动式NFC对象，比如一张公交卡，Tag可以理解成能被手机NFC读写的对象。当Android检测到一个Tag时，会创建一个Tag对象，将其放在Intent对象中，然后发送到相应的Activity。

`IsoDep`：该类位于`android.nfc.tech.IsoDep`包中，是支持ISO-DEP（ISO 14443-4）协议Tag的操作类。NFC协议众多，其他的协议就要用到其他类了。常见的还有：

* `MifareClassic`类：支持Mifare Classic协议
* `MifareUltralight`类：支持Mifare Ultralight协议
* `Ndef`类和`NdefFormatable`类：支持NDEF格式
* `NfcA`类：支持NFC-A（ISO-14443-3A）协议
* `NfcB`类：支持NFC-B（ISO-14443-3B）协议
* `NfcF`类：支持NFC-F（JIS 6319-4）协议
* `NfcV`类：支持NFC-V（ISO 15693）协议
* 等等

## 二、NFC调度系统

NFC调度是指手机检测到NFC对象后如何处理，调度系统分为前台调度系统（Foreground Dispatch System）和标签调度系统（NFC Tag Dispatch System）。

### 1) 前台调度系统

NFC前台调度系统是一种用于在运行的程序中（前台呈现的Activity）处理Tag的技术，即前台调度系统允许Activity拦截Intent对象，并且声明该Activity的优先级比其他的处理Intent对象的Activity高。前台调度系统在一些涉及需要在前台呈现的页面中直接获取或推送NFC信息时十分方便。本文的示例就是使用前台调度。

前台调度的使用方法如下：

```java
// 创建一个PendingIntent对象，以便Android系统能够在扫描到NFC标签时，用它来封装NFC标签的详细信息
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, new Intent(this, getClass()).addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP), 0);

// 创建Intent过滤器
IntentFilter iso = new IntentFilter(NfcAdapter.ACTION_TECH_DISCOVERED);

// 创建一个处理NFC标签技术的数组
String[][] techLists = new String[][]{new String[]{IsoDep.class.getName()}};

// 在主线程中调用enableForegroundDispatch()方法，一旦NFC标签接触到手机，这个方法就会被激活
adapter.enableForegroundDispatch(this, pendingIntent, new IntentFilter[]{iso}, techLists);
```

### 2) 标签调度系统

NFC标签调度系统是一种通过预先定义好的Tag或NDEF消息来启动应用程序的机制，当扫描到一个NFC Tag时，如果Intent中注册对应的App，那么在处理该Tag信息时就会启动该App。当存在多个可以处理该Tag信息的Apps时，系统会弹出一个Activity Choose，供用户选择开启哪个应用。标签调度系统定义了3种Intent对象，按照优先级由高到低分别为`ACTION_NDEF_DISCOVERED`、`ACTION_TECH_DISCOVERED`、`ACTION_TAB_DISCOVERED`。

标签调度的基本工作方法如下：

* 用解析NFC标签时由标签调度系统创建的Intent对象（`ACTION_NDEF_DISCOVERED`）来尝试启动Activity。
* 如果没有对应的处理Intent的Activity，就会尝试使用下一个优先级的Intent（`ACTION_TECH_DISCOVERED`，继而`ACTION_TAG_DISCOVERED`）来启动Activity，直到有对应的App来处理这个Intent，或者是直接标签调度系统尝试了所有可能的Intent。
* 如果没有应用程序来处理任何类型的Intent，就不做任何事情。

## 三、Android NFC 编程示例

一般的非接触智能卡如公交卡都支持取随机数命令，该示例实现通过NFC获取随机数的功能。操作智能卡的NFC协议使用`IsoDep`。

示例主要有两个类，与NFC对象的交互类`NFC.java`，处理NFC调度的`MainActivity.java`。源码在文章最后可以下载。

### 1) 编写与NFC对象的交互类

最开始，我们需要定义自己的`NFC类`，在该类中定义获取随机数的APDU命令，在构造函数中初始化ISO-DEP协议的Tag操作类实例，在`send()`方法中发送和接收APDU。`NFC类`实现如下：

```java
public class NFC {
    // 获取随机数的APDU命令
    private static final byte[] GET_RANDOM = {0x00, (byte)0x84, 0x00, 0x00, 0x08};

    // 声明ISO-DEP协议的Tag操作实例
    private final IsoDep tag;

    public NFC(IsoDep tag) throws IOException {
        // 初始化ISO-DEP协议的Tag操作类实例
        this.tag = tag;
        tag.setTimeout(5000);
        tag.connect();
    }

    /**
     * 向Tag发送获取随机数的APDU并返回Tag响应
     * @return 十六进制随机数字符串
     * @throws IOException
     * @throws APDUError
     */
    public String send() throws IOException, APDUError {
        // 发送APDU命令
        byte[] resp = tag.transceive(GET_RANDOM);
        String strResp =  new String(Hex.encodeHex(resp));
        Log.d("REQ ", new String(Hex.encodeHex(GET_RANDOM)));
        Log.d("RESP", new String(Hex.encodeHex(resp)));
        // 获取NFC Tag返回的状态值
        int status = ((0xff & resp[resp.length - 2]) << 8) | (0xff & resp[resp.length - 1]);
        if (status != 0x9000) {
            throw new APDUError(status);
        }

        return strResp;
    }

}
```

### 2) NFC调度的实现

在MainActivity中编写以下代码。

① 获取NfcAdapter，可以在onCreate()方法中实现。
```java
adapter = NfcAdapter.getDefaultAdapter(getApplicationContext());
```

② 构造PendingInent对象封装NFC标签信息，②-⑤步骤都可以在onResume()方法中实现。
```java
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0,
                new Intent(this, getClass()).addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP), 0);
```

③ 声明Intent对象的过滤器。
```java
IntentFilter iso = new IntentFilter(NfcAdapter.ACTION_TECH_DISCOVERED);
```

④ 建立一个处理NFC标签技术的数组。
```java
String[][] techLists = new String[][]{new String[]{IsoDep.class.getName()}};
```

⑤ 调用enableForegroundDispatch()方法。
```java
adapter.enableForegroundDispatch(this, pendingIntent, new IntentFilter[]{iso}, techLists);
```

⑥ 在onPause()方法中调用disableForegroundDispatch()方法当Activity挂起时禁用前台调用。
```java
adapter.disableForegroundDispatch(this);
```

⑦ 最后在onNewIntent()方法中处理Intent回调给我们的信息。
```java
@Override
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);
    Tag tag = intent.getParcelableExtra(NfcAdapter.EXTRA_TAG);
    if(tag != null) {
        try {
            NFC nfc = new NFC(IsoDep.get(tag));
            // 发送取随机数APDU命令
            String resp = nfc.send();
            // TextView中显示APDU响应
            ((TextView) findViewById(R.id.result)).setText(resp);
        } catch (APDUError apduError) {
            Toast.makeText(getApplicationContext(), apduError.getMessage(), Toast.LENGTH_LONG).show();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            findViewById(R.id.progressBar).setVisibility(View.INVISIBLE);
        }
    }
}
```

## 四、源码下载

程序在Android Studio 1.5.1中编写并测试通过。源码链接： http://pan.baidu.com/s/1hrjprZQ 密码: 29tb

## 五、参考内容

1. 《Android NFC开发实战详解》赵波 第四章
2. http://www.jianshu.com/p/7991dc02b0d8

