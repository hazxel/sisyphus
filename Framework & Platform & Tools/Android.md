### Commit message

```
改动目的及范围：
修改了 TDialog.onCreate 中提示下载页面的 DecorView 的默认 padding 值，现已完全覆盖屏幕(已更新open_sdk.jar)

自测情况：
sdk，sample正常编译运行
```



修改pb：(参考 pbfile/generatepb.bat)

1. cd ./QQLite/pbfile

2. 修改该目录下的xxx.proto

3. Run:

   > protoc_mqq --javamqq_out=../../Foundation/QQCommon/src/main/java xxx.proto 

4. GatewayVerify.java 会被自动修改

### Quick Android sdk commands:

- Search all local jdk paths：/usr/libexec/java_home -V

- What's current activity: 

  adb shell dumpsys activity activities > /Users/zhouboyan/Desktop/task.txt

- Print stack trace:

  android.util.Log.d("", android.util.Log.getStackTrace(new Throwable()));





### SDK Manager:

1. if using proxy, proxy settings need to be specified:

   ```shell
   sdkmanager --install "build-tools;19.1.0" --verbose --no_https --proxy=http --proxy_host=127.0.0.1 --proxy_port=12639
   ```

2. official link here:

   https://developer.android.com/studio/command-line/sdkmanager



SDK Log Path：

/sdcard/Tencent/msflogs/com/tencent/mobileqq/com.tencent.mobileqq_connectSdk.xxxx
/sdcard/Android/data/包名/files/tencent/mobileqq/opensdk/logs/


### Build Tools

1. java.io.IOException: Cannot run program "/Users/zhouboyan/Library/Android/sdk/build-tools/19.1.0/aapt" (in directory "/Users/zhouboyan/AndroidSDK"): error=86, Bad CPU type in executable:
   - build-tool version is too low
   - if you insist on using this version, try to copy a higher version "aapt" and replace the current one
2. ?



### ProGurad:

1. obfuscating tool & java bytecode optimizing tool
2. java.io.IOException: Can't process class [android/Manifest.class] (Unsupported class version number [52.0] (maximum 51.0, Java 1.7))
   - ProGuard Version is too low
   - Try to upgrade:
     - **http://proguard.sourceforge.net/**
     - or refer to https://www.coder.work/article/137741



### Android system start-up：

1. init process: init is the first Linux User-Mode process

2. Zygote process (app_process): 

   - Zygote process is the first Java process
   - Zygote process is a child process of init process
   - Zygote works as a "Server" of a socket, receiving requests of creating precesses
   - Processes created by Zygote share the same JVM

3. System Server process

   - System Server process is a child process of Zygote process

     ```java
     pid = Zygote.forkSystemServer();
     ```

   - ?

4. Activity Manager Service process

5. Home Activity



#### APP start-up

1. Activity Thread:
   - Activity Thread is the so-called "main thread" or "UI thread"
   - main method of ActivityThread is the entry point of the APP
   - It creates the MainLooper in its main method
2. ?



### Class

1. Activity
2. Window
3. View
4. Dialog
5. Appliation
   - Base class for maintaining global application state. 
   - You can provide your own implementation by creating a subclass and specifying the fully-qualified name of this subclass as the `"android:name"` attribute in your AndroidManifest.xml's `<application>` tag. 
   - The Application class, or your subclass of the Application class, is instantiated before any other class when the process for your application/package is created.
6. Contex
   - Interface to global information about an application environment.
7. PackageManager
   - Class for retrieving global information related to the application packages
   - ApplicationInfo: Information you can retrieve about a particular application. This corresponds to information collected from the AndroidManifest.xml's <application> tag.
   - PackageInfo: Overall information about the contents of a package. This corresponds to all of the information collected from AndroidManifest.xml.
8. Looper
9. Handler
10. Intent
    - Intent
      - Action
      - Component: class name of the activity or service you want to start
      - Data
      - Extras
    - IntentFilter: class used to set up run-time intent filter 
    - <intent-filter>: element specified in AndroidManifest.xml file
      - <action>: Declares the intent action accepted, in the **name** attribute.
      - <data>: Declares the type of data accepted, using one or more attributes that specify various aspects of the data URI (**scheme**, **host**, **port**, **path**) and MIME type.
      - <category>: Declares the intent category accepted, in the **name** attribute.
    - startActivityForResult <--> onActivityResult
11. ?



### UI

1. DecorView:
   - root view of activity window
   - Usually owns a linear view containing:
     - FrameLayout of title bar (optional)
     - FrameLayout content (of certain activity)
2. onCreate() method:
   - calls setConventView() method
     - which calls getwindow().setContentView() method
       - which is PhoneWindow.setContentView() method
         - which calls installDecor() method
           - which creates a DecorView instance
3. Layout
   - FrameLayout: usually displaying single widget
   - LinearLayout: usually used to display multiple widgets
4. UI Automator Viewer
   - located in Android/Sdk/tools/bin
   - heip to visualize the UI hierarchy
5. ？



### Useful codes

1. pause

   ```java
   new Handler(Looper.getMainLooper()).postDelayed(new Runnable() {
      @Override
      public void run() {
         ((ViewGroup)(getWindow().getDecorView())).getChildAt(0).setPadding(0, 0, 0, 0);
      }
   }, 100);
   ```

   safe version:

   ```java
   // 未安装手Q时，为使提示下载的页面完全覆盖整个页面，需要去除 DecorView 下 LinearLayout 的默认 padding
   // 需要使用 Handler.post 延迟执行，否则改动会被覆盖
   new Handler(Looper.getMainLooper()).post(new Runnable() {
       @Override
       public void run() {
           Window dialogWindow = getWindow();
           if (dialogWindow == null) {
               return;
           }
   
           View dectorView = dialogWindow.getDecorView();
           if (dectorView == null) {
               return;
           }
   
           View linearLayout = ((ViewGroup) dectorView).getChildAt(0);
           if (linearLayout == null) {
               return;
           }
   
           linearLayout.setPadding(0, 0, 0, 0);
       }
   });
   ```

2. ?



