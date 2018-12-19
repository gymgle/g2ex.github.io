title: "Android App与Flask通信实例：POST数据的发送与处理"
date: 2015-11-19 23:00:00
tags:
- Flask
- Android
- POST
- Python
permalink: Post-from-Android-App-to-Flask-Server
---

## 一、说明

在Android开发中，当应用需要向服务器提交数据数据时，可以使用POST方式向服务器提交form表单。服务器端框架有多种选择，快速开发可以使用Python的Flask框架。本文介绍了在服务器端如何使用Flask框架处理客户端发送的POST请求，以及在Android App中如何向服务器发送POST请求。

> 实验环境
服务器：Ubuntu 14.04 x64, IP地址: 10.10.10.128
Android IDE：Android Studio 1.4.1
辅助测试工具：Chrome应用`Postman`

本文中，Android客户端使用POST的方式向服务器提交form表单。我们假设需要提交的form表单只有两个字段，分别是`uid="001"`和`name="g2ex"`。

接下来，我们先编写服务器端Flask应用，接收客户端发送的POST数据，解析出这两个参数的内容，并向客户端返回确认信息`OK.`。然后再在Android Studio中编写客户端App实现POST提交数据的过程。

## 二、服务器端开发

本文中，服务器端使用Python的Flask框架。想要快速了解Flask，可以参考[Flask中文开发文档][1]。Flask的安装请参考[Flask安装说明][2]。

### 1) 编写一个简单的Flask应用

使用Flask处理客户端的POST请求非常简单，我们创建一个项目目录`flaskr`，在目录下创建一个名为`myapp.py`的文件，写入如下代码：

```python
from flask import Flask, request

# create the application object
app = Flask(__name__)

@app.route('/', methods=['POST'])
def post():
    uid = request.form['uid']
    name =  request.form['name']
    print 'uid: %s, name: %s' % (uid, name)
    return 'OK.'

if __name__ == "__main__":
    app.run(host='0.0.0.0', debug=True)
```

使用下面的命令启动Flask，默认在本机的`5000`端口上监听来自所有网络的POST请求。

```bash
python myapp.py
```

### 2) 简单测试Flask

在Chrome浏览器`chrome://apps`中打开`Postman`应用。类型选择`POST`，服务器地址中输入`http://10.10.10.128:5000`，`Body`中勾选`x-www-form-urlencoded`，并在下面的`Key-Value`中输入我们要提交的内容。点击`Send`可以看到服务器返回的`OK.`，说明我们编写的Flask应用可以处理POST请求。如下图所示。

![Postman POST测试][3]

此时，在服务器端的Terminal中，可以看到打印出来的记录：

```bash
uid: 001, name: g2ex
10.10.10.1 - - [18/Nov/2015 22:52:25] "POST / HTTP/1.1" 200 -
```

## 三、Android客户端开发

本文中，Android客户端使用POST的方式向服务器提交form数据，可以编写`HttpUtils`类实现这个功能，`HttpUtils`类中使用了标准Java接口`HttpURLConnection`。

首先，使用Android Studio创建一个名为`PostTest`的项目，在布局文件`activity_main.xml`中创建一个id为`button_send`的按钮。

在`MainActivity`中的`onCreate()`中添加按钮监听事件，按钮点击后调用`postData()`，`postData()`启动线程使用`HttpUtils`类的静态方法`submitPostData()`向服务器POST数据。

```java
/* MainActivity.java */

private Button mButton_send;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    mButton_send = (Button)findViewById(R.id.button_send);
    mButton_send.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            postData();
        }
    });
}

private void postData() {
    new Thread(new Runnable() {
        @Override
        public void run() {
            Map<String, String> params = new HashMap<String, String>();
            String post_result = null;
            params.put("uid", "001");
            params.put("name", "g2ex");
            try {
                post_result = HttpUtils.submitPostData(params, "utf-8");
                Log.i("POST_RESULT", post_result);
            } catch (MalformedURLException e) {
                e.printStackTrace();
            }
        }
    }).start();
}
```

`HttpUtils`类有三个静态方法：

* `submitPostData()`方法发送POST请求到服务器并返回服务器信息
* `getRequestData()`方法用于封装请求体
* `dealResponseResult()`方法处理服务器的响应结果

```java
/* HttpUtils.java */

public class HttpUtils {
    public static String submitPostData(Map<String, String> params, String encode) throws MalformedURLException {
        /**
         * 发送POST请求到服务器并返回服务器信息
         * @param params 请求体内容
         * @param encode 编码格式
         * @return 服务器返回信息
         */
        byte[] data = getRequestData(params, encode).toString().getBytes();
        URL url = new URL("http://10.10.10.128:5000/");
        HttpURLConnection httpURLConnection = null;
        try{
            httpURLConnection = (HttpURLConnection)url.openConnection();
            httpURLConnection.setConnectTimeout(3000);  // 设置连接超时时间
            httpURLConnection.setDoInput(true);         // 打开输入流，以便从服务器获取数据
            httpURLConnection.setDoOutput(true);        // 打开输出流，以便向服务器提交数据
            httpURLConnection.setRequestMethod("POST"); // 设置以POST方式提交数据
            httpURLConnection.setUseCaches(false);      // 使用POST方式不能使用缓存
            // 设置请求体的类型是文本类型
            httpURLConnection.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
            // 设置请求体的长度
            httpURLConnection.setRequestProperty("Content-Length", String.valueOf(data.length));
            // 获得输入流，向服务器写入数据
            OutputStream outputStream = new BufferedOutputStream(httpURLConnection.getOutputStream());
            outputStream.write(data);
            outputStream.flush();                       // 重要！flush()之后才会写入

            int response = httpURLConnection.getResponseCode();     // 获得服务器响应码
            if (response == HttpURLConnection.HTTP_OK) {
                InputStream inputStream = httpURLConnection.getInputStream();
                return dealResponseResult(inputStream);             // 处理服务器响应结果
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            httpURLConnection.disconnect();
        }

        return "";
    }

    /**
     * 封装请求体信息
     * @param params 请求体内容
     * @param encode 编码格式
     * @return 请求体信息
     */
    public static StringBuffer getRequestData(Map<String, String> params, String encode) {
        StringBuffer stringBuffer = new StringBuffer();            //存储封装好的请求体信息
        try {
            for (Map.Entry<String, String> entry : params.entrySet()) {
                stringBuffer.append(entry.getKey())
                        .append("=")
                        .append(URLEncoder.encode(entry.getValue(), encode))
                        .append("&");
            }
            stringBuffer.deleteCharAt(stringBuffer.length() - 1);   // 删除最后一个"&"
        } catch (Exception e) {
            e.printStackTrace();
        }
        return stringBuffer;
    }

    /**
     * 处理服务器的响应结果（将输入流转换成字符串)
     * @param inputStream 服务器的响应输入流
     * @return 服务器响应结果字符串
     */
    public static String dealResponseResult(InputStream inputStream) {
        String resultData = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        byte[] data = new byte[1024];
        int len = 0;
        try {
            while ((len = inputStream.read(data)) != -1) {
                byteArrayOutputStream.write(data, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        resultData = new String(byteArrayOutputStream.toByteArray());
        return resultData;
    }
}
```

最后，记得在`AndroidManifest.xml`中加入网络权限：

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

运行应用，在Android Studio logcat中可以看到输出的日志`I/POST_RESULT: OK.`，服务器终端中也可以看到打印的内容和访问记录。

> 源码下载：http://pan.baidu.com/s/13G7LW 密码: 4ww5

## 四、参考内容

1. http://docs.jinkan.org/docs/flask/ 《Flask中文开发文档》
2. http://www.cnblogs.com/menlsh/archive/2013/05/22/3091983.html 《Android学习笔记46：使用Post方式提交数据》。本文中的`HttpUtils`引用自该文章，文章中服务器端使用的是Apache Tomcat。


  [1]: http://docs.jinkan.org/docs/flask/ "Flask中文开发文档"
  [2]: http://docs.jinkan.org/docs/flask/installation.html "安装Flask"
  [3]: https://i.imgur.com/EWOsEWF.png "Postman POST测试"