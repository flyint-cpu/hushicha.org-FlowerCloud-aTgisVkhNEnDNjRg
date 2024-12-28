
## 首先编写一个android测试程序


功能：校验用户名和注册码，成功则弹出注册成功提示


以下仅给出关键部分的代码


**res/layout/activity\_main.xml**



```
xml version="1.0" encoding="utf-8"?
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/textView1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:text="@string/info"
        android:textSize="20dp" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:orientation="horizontal">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/username" />

        <EditText
            android:id="@+id/edit_username"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginLeft="10dp"
            android:layout_marginRight="10dp"
            android:layout_weight="1"
            android:ems="10"
            android:hint="@string/hint_username">EditText>
    LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:orientation="horizontal">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/sn" />

        <EditText
            android:id="@+id/edit_sn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginLeft="10dp"
            android:layout_marginRight="10dp"
            android:layout_weight="1"
            android:ems="10"
            android:hint="@string/hint_sn">EditText>
    LinearLayout>

    <Button
        android:id="@+id/button_register"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:layout_marginRight="10dp"
        android:text="@string/register" />

LinearLayout>

```

**java/com/example/myapplication/MainActivity.java**



```
package com.example.myapplication;

import android.app.Activity;
import android.os.Bundle;
import android.view.Menu;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class MainActivity extends Activity {
    private EditText edit_userName;
    private EditText edit_sn;
    private Button btn_register;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        setTitle(R.string.unregister);  //模拟程序未注册
        edit_userName = (EditText) findViewById(R.id.edit_username);
        edit_sn = (EditText) findViewById(R.id.edit_sn);
        btn_register = (Button) findViewById(R.id.button_register);
        btn_register.setOnClickListener(new OnClickListener() {

            public void onClick(View v) {
                if (!checkSN(edit_userName.getText().toString().trim(),
                        edit_sn.getText().toString().trim())) {
                    Toast.makeText(MainActivity.this,       //弹出无效用户名或注册码提示
                            R.string.unsuccessed, Toast.LENGTH_SHORT).show();
                } else {
                    Toast.makeText(MainActivity.this,       //弹出注册成功提示
                            R.string.successed, Toast.LENGTH_SHORT).show();
                    btn_register.setEnabled(false);
                    setTitle(R.string.registered);  //模拟程序已注册
                }
            }
        });
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.activity_main, menu);
        return true;
    }

    private boolean checkSN(String userName, String sn) {
        try {
            if ((userName == null) || (userName.length() == 0))
                return false;
            if ((sn == null) || (sn.length() != 16))
                return false;
            MessageDigest digest = MessageDigest.getInstance("MD5");
            digest.reset();
            digest.update(userName.getBytes());
            byte[] bytes = digest.digest();     //采用MD5对用户名进行Hash
            String hexstr = toHexString(bytes, ""); //将计算结果转化成字符串
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < hexstr.length(); i += 2) {
                sb.append(hexstr.charAt(i));
            }
            String userSN = sb.toString(); //计算出的SN
            //Log.d("crackme", hexstr);
            //Log.d("crackme", userSN);
            if (!userSN.equalsIgnoreCase(sn))   //比较注册码是否正确
                return false;
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }

    private static String toHexString(byte[] bytes, String separator) {
        StringBuilder hexString = new StringBuilder();
        for (byte b : bytes) {
            String hex = Integer.toHexString(0xFF & b);
            if (hex.length() == 1) {
                hexString.append('0');
            }
            hexString.append(hex).append(separator);
        }
        return hexString.toString();
    }

}


```

![2024-07-22T06:35:25.png](https://img2024.cnblogs.com/blog/2747555/202412/2747555-20241227134933479-1673202419.png)


运行没有问题之后，通过`AndroidStudio`编译成apk文件


## 开始破解程序


破解 Android 程序通常的方法是将 apk 文件利用 ApkTool 反编译，生成 Smali 格式的反汇编代码，然后阅读 Smali 文件的代码来理解程序的运行机制，找到程序的突破口进行修改，最后使用 ApkTool 重新编译生成 apk 文件并签名，最后运行测试，如此循环，直至程序被成功破解。


使用`apk-tool`反编译apk程序


下载地址：[https://down.52pojie.cn/Tools/Android\_Tools/apktool\_2\.9\.3\.jar](https://github.com)


执行



```
java -jar apktool_2.9.3.jar d -f app-debug.apk -o output

```

![2024-07-22T06:45:41.png](https://img2024.cnblogs.com/blog/2747555/202412/2747555-20241227134933467-1472106580.png)


`smail`目录下存放了程序的所有反汇编代码，`res`目录下则是程序的所有资源文件


先通过`res\values\string.xml`定位程序的错误信息



```
<resources>
...
    <string name="unsuccessed">无效用户名或注册码string>
...
resources>

```

再通过同目录下的`public.xml`找到`name="unsuccessed"`的id



```
<public type="string" name="unsuccessed" id="0x7f1000a5" />

```

通过该id可以到`smail`目录搜索，在`output\smali_classes3\com\example\myapplication\MainActivity$1.smali`搜索到一处结果



```
   91      iget-object v0, p0, Lcom/example/myapplication/MainActivity$1;->this$0:Lcom/example/myapplication/MainActivity;
   92  
   93:     const v2, 0x7f1000a5
   94  
   95      invoke-static {v0, v2, v1}, Landroid/widget/Toast;->makeText(Landroid/content/Context;II)Landroid/widget/Toast;


```

onclick方法



```

# virtual methods
.method public onClick(Landroid/view/View;)V
    .locals 3
    .param p1, "v"    # Landroid/view/View;

    .line 31
    iget-object v0, p0, Lcom/example/myapplication/MainActivity$1;->this$0:Lcom/example/myapplication/MainActivity;

    invoke-static {v0}, Lcom/example/myapplication/MainActivity;->access$000(Lcom/example/myapplication/MainActivity;)Landroid/widget/EditText;

    move-result-object v1

    invoke-virtual {v1}, Landroid/widget/EditText;->getText()Landroid/text/Editable;

    move-result-object v1

    invoke-virtual {v1}, Ljava/lang/Object;->toString()Ljava/lang/String;

    move-result-object v1

    invoke-virtual {v1}, Ljava/lang/String;->trim()Ljava/lang/String;

    move-result-object v1

    iget-object v2, p0, Lcom/example/myapplication/MainActivity$1;->this$0:Lcom/example/myapplication/MainActivity;

    .line 32
    invoke-static {v2}, Lcom/example/myapplication/MainActivity;->access$100(Lcom/example/myapplication/MainActivity;)Landroid/widget/EditText;

    move-result-object v2

    invoke-virtual {v2}, Landroid/widget/EditText;->getText()Landroid/text/Editable;

    move-result-object v2

    invoke-virtual {v2}, Ljava/lang/Object;->toString()Ljava/lang/String;

    move-result-object v2

    invoke-virtual {v2}, Ljava/lang/String;->trim()Ljava/lang/String;

    move-result-object v2

    .line 31
    invoke-static {v0, v1, v2}, Lcom/example/myapplication/MainActivity;->access$200(Lcom/example/myapplication/MainActivity;Ljava/lang/String;Ljava/lang/String;)Z

    move-result v0

    const/4 v1, 0x0

    if-nez v0, :cond_0 # 关键条件判断

    .line 33
    iget-object v0, p0, Lcom/example/myapplication/MainActivity$1;->this$0:Lcom/example/myapplication/MainActivity;

    const v2, 0x7f1000a5

    invoke-static {v0, v2, v1}, Landroid/widget/Toast;->makeText(Landroid/content/Context;II)Landroid/widget/Toast;

    move-result-object v0

    .line 34
    invoke-virtual {v0}, Landroid/widget/Toast;->show()V

    goto :goto_0

    .line 36
    :cond_0
    iget-object v0, p0, Lcom/example/myapplication/MainActivity$1;->this$0:Lcom/example/myapplication/MainActivity;

    const v2, 0x7f1000a2

    invoke-static {v0, v2, v1}, Landroid/widget/Toast;->makeText(Landroid/content/Context;II)Landroid/widget/Toast;

    move-result-object v0

    .line 37
    invoke-virtual {v0}, Landroid/widget/Toast;->show()V

    .line 38
    iget-object v0, p0, Lcom/example/myapplication/MainActivity$1;->this$0:Lcom/example/myapplication/MainActivity;

    invoke-static {v0}, Lcom/example/myapplication/MainActivity;->access$300(Lcom/example/myapplication/MainActivity;)Landroid/widget/Button;

    move-result-object v0

    invoke-virtual {v0, v1}, Landroid/widget/Button;->setEnabled(Z)V

    .line 39
    iget-object v0, p0, Lcom/example/myapplication/MainActivity$1;->this$0:Lcom/example/myapplication/MainActivity;

    const v1, 0x7f100099

    invoke-virtual {v0, v1}, Lcom/example/myapplication/MainActivity;->setTitle(I)V

    .line 41
    :goto_0
    return-void
.end method

```

经过分析，`if-nez v0, :cond_0`判断决定了校验是否通过，这句代码的意思是：如果v0不为0则跳转到cond\_0，也就是注册失败的分支。


破解方法：将`if-nez`改为`if-eqz`也就是等于则为真


修改后保存，执行如下代码重新编译



```
java -jar apktool_2.9.3.jar b output

```

编译后需要对apk进行签名，这里我本地生成了一个测试签名



```
keytool -genkey -alias testalias -keyalg RSA -keysize 2048 -validity 36500 -keystore test.keystore

```

然后用360加固助手的工具包进行签名。


接下来可以通过adb命令安装和启动



```
adb install app-debug_sign.apk
adb shell am start -n com.example.myapplication/.MainActivity

```

![2024-07-22T08:00:54.png](https://img2024.cnblogs.com/blog/2747555/202412/2747555-20241227134933491-1252555501.png)


至此破解就算完成了。


## 使用IDA pro破解


因为使用apktool每次都需要重新编译，很花时间，idapro提供了快速测试的方法。


下载地址：[https://down.52pojie.cn/Tools/Disassemblers/IDA\_Pro\_v8\.3\_Portable.zip](https://github.com):[wgetCloud机场](https://longdu.org)


用压缩包工具打开`app-debug.apk`，提取出`classes.dex`文件，通过ida pro打开



> 注意：如果`classes.dex`没有可能在`classes3.dex`


按照先找错误信息的思路，按`alt+t`打开文本搜索功能，搜索：0x7f1000a5


![2024-07-22T08:35:50.png](https://img2024.cnblogs.com/blog/2747555/202412/2747555-20241227134933963-1356348784.png)


定位到关键代码（按空格可以切换视图）


![2024-07-23T01:31:23.png](https://img2024.cnblogs.com/blog/2747555/202412/2747555-20241227134933503-774747428.png)


可以看到分支判断位于`CODE:000006BE`，将光标放在`if-nez`处，点击`hex-view`，修改`if-nez`的字节码


`39 00 0F 00`改为`38 00 0F 00`


然后关闭`ida pro`，不需要保存到database


用`c32asm`打开`classes3.dex`定位到`000006BE`，将39改为38，保存退出


![2024-07-23T01:41:13.png](https://img2024.cnblogs.com/blog/2747555/202412/2747555-20241227134933490-924636897.png)


这里由于修改了dex文件，会导致dex文件在验证计算checksum会失败，从而导致程序安装失败，因此需要重新计算checksum值


用Dexfixer将classes3\.dex文件checksum值修复


![2024-07-23T01:51:10.png](https://img2024.cnblogs.com/blog/2747555/202412/2747555-20241227134933488-748917198.png)


将修复好的`classes3.dex`，重新拉入apk



```
aapt r app-debug.apk classes3.dex
aapt a app-debug.apk classes3.dex

```

删除META\-INT（可使用winrar工具），并重新签名apk即可破解成功。


![image](https://img2024.cnblogs.com/blog/2747555/202412/2747555-20241218222717143-1042054688.png)


参考：《Android软件安全与逆向分析》


