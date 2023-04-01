---
title: Android-Create
tags: Android
categories: Android
top: false
---



- 安卓开发

------

### 主界面

```xml
<activity
    android:name=".MainActivity"                                    
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

### AndroidManifest.xml常见参数

```xml
android:name=""                 //设置activity所属application
android:exported="true"         //允许外部组件启动这个Activity
android:debuggable="true"       //设定apk可调试
```

### 夜神模拟器无法adb识别

- adb版本不一致，CV

### 隐藏标题栏失效

- 继承的是 Activity

```xml
Java:
requestWindowFeature(Window.FEATURE_NO_TITLE);
//写在setContentView()方法的前面

AndroidManifest.xml:
<application android:theme="@android:style/Theme.NoTitleBar">
```

- 继承的是 AppCompatActivity

```xml
Java:
getSupportActionBar().hide();
//写在setContentView()方法的后面

AndroidManifest.xml:
<application android:theme="@style/Theme.AppCompat.NoActionBar">
```

## Java问题

- trim函数，去掉字符串头尾空白字符