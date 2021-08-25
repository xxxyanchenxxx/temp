# VA开发文档 #
本文档主要介绍2部分。  
第一部分是VA的源码结构介绍，这部分是为了让开发者能快速了解掌握VA源码框架。  
第二部分是VA的SDK如何使用。  
</br>
**下面开始第一部分，VA源码结构介绍：**

## 1. VA源码目录介绍 ##
下图是VA源码根目录：  
![](https://github.com/xxxyanchenxxx/temp/blob/master/dev/1.png)  
可以看到VA一共有4个源码目录，各个目录介绍如下：
<table >
        <tr>
            <th>目录名称</th>
            <th>作用</th>
        </tr>
        <tr  align="left">
            <th>app</th>
            <th>VA Demo主包源码所在目录</th>
        </tr>
        <tr  align="left">
            <th>app-ext</th>
            <th>VA Demo插件包源码所在目录</th>
        </tr>
        <tr  align="left">
            <th>lib</th>
            <th>VA库源码所在目录</th>
        </tr>
        <tr  align="left">
            <th>lib-ext</th>
            <th>VA插件库源码所在目录</th>
        </tr>
</table>  

## 2. VA编译配置文件介绍 ##
VA有2个配置文件，第一个配置文件是AppConfig.gradle：  
![](https://github.com/xxxyanchenxxx/temp/blob/master/dev/2_1.png)  
配置解释：  
<table >
        <tr>
            <th>配置名称</th>
            <th>作用</th>
        </tr>
        <tr  align="left">
            <th>PACKAGE_NAME</th>
            <th>用于配置VA主包的包名</th>
        </tr>
        <tr  align="left">
            <th>EXT_PACKAGE_NAME</th>
            <th>用于配置VA插件包的包名</th>
        </tr>
</table>  

第二个配置文件是VAConfig.gradle：
![](https://github.com/xxxyanchenxxx/temp/blob/master/dev/2_2.png)  
配置解释：
<table >
        <tr>
            <th>配置名称</th>
            <th>作用</th>
        </tr>
        <tr  align="left">
            <th>VA_MAIN_PACKAGE_32BIT</th>
            <th>用于配置VA主包是32位还是64位，true为32位，false为64位</th>
        </tr>
        <tr  align="left">
            <th>VA_ACCESS_PERMISSION_NAME </th>
            <th>用于配置VA中4大组建的权限名称 </th>
        </tr>
        <tr  align="left">
            <th>VA_AUTHORITY_PREFIX </th>
            <th>用于配置VA主包中ContentProvider的authorities </th>
        </tr>
        <tr  align="left">
            <th>VA_EXT_AUTHORITY_PREFIX </th>
            <th>用于配置VA插件包中ContentProvider的authorities </th>
        </tr>
        <tr  align="left">
            <th>VA_VERSION</th>
            <th>用于配置VA库版本，开发者一般不需要关心</th>
        </tr>
        <tr  align="left">
            <th>VA_VERSION_CODE</th>
            <th>用于配置VA库版本代码，开发者一般不需要关心</th>
        </tr>
</table>  

## 3. VA核心代码解释 ##
1. `com.lody.virtual.client`包下的代码运行在VAPP Client进程中，主要用于VA Framework中的APP Hook部分，完成对各个Service的HOOK处理  
![](https://github.com/xxxyanchenxxx/temp/blob/master/dev/3_1.png)  
2. `com.lody.virtual.server`包下的代码运行在VA Server进程中，代码主要用于VA Framework中的APP Server部分，实现处理APP安装以及其他不给Android系统处理的APP请求  
![](https://github.com/xxxyanchenxxx/temp/blob/master/dev/3_2.png)
3. `mirror`包下的代码主要用于对系统隐藏类的引用，属于工具类，减少大量反射代码的编写  
![](https://github.com/xxxyanchenxxx/temp/blob/master/dev/3_3.png)
4. `cpp`包下的代码进行在VAPP Client进程中，主要用于VA Native部分，实现IO重定向和jni函数HOOK。其中：  
	- `substrate`中实现了针对arm32和arm64的hook  
	- `vfs.cpp`中实现了VA的虚拟文件系统，用于控制APP文件访问限制  
	- `syscall_hook.cpp`中实现了对IO的Hook  
![](https://github.com/xxxyanchenxxx/temp/blob/master/dev/3_4.png)  
5. `DelegateApplicationExt.java`运行在VA Host Plugin进程中，用于VA插件包，实现了对主包代码的加载执行  
![](https://github.com/xxxyanchenxxx/temp/blob/master/dev/3_5.png)  

</br></br>
**下面开始第二部分，VA SDK使用介绍：**

## 1. 安装APP ##
## VA 2.1 开始，引入了新的安装API:
```java
VirtualCore.java

 public VAppInstallerResult installPackage(Uri uri, VAppInstallerParams params);
```
## Uri是什么?
Uri决定了**需要安装的apk**的来源，目前支持 package 和 file 协议。
### Package Uri 示例:
```java
Uri packageUri = Uri.parse("package:com.hello.world");
```
### File Uri 示例:
```java
File apkFile = new File("/sdcard/test.apk"); 
Uri packageUri = Uri.fromFile(apkFile);
```

## 两种Uri安装app有何区别?
**package协议** 安装app，只需要传入包名，不需要具体的APK路径，所以以这种协议安装的app，**相当于双开**。

app会随外部版本的升级而自动升级，随外部版本的卸载而自动卸载。`PackageSetting` 中的 `dynamic` 为 `true`。

**file协议** 则是内部安装，apk会被复制到容器内部，与外部版本完全独立. `PackageSetting` 中的 `dynamic` 为 `false`。

## 安装参数 VAppInstallerParams

### 安装标志 installFlags

FLAG | 说明
--- | ---
FLAG_INSTALL_OVERRIDE_NO_CHECK | 允许覆盖安装
FLAG_INSTALL_OVERRIDE_FORBIDDEN | 禁止覆盖安装
FLAG_INSTALL_OVERRIDE_DONT_KILL_APP | 覆盖安装不kill已经启动的APP

### 安装模式 mode

FLAG | 说明
--- | ---
MODE_FULL_INSTALL | 完整安装
MODE_INHERIT_EXISTING | 已安装的的安装模式。预留

预留参数，暂时未使用。目前不管设置哪种都一样。

### cpuAbiOverride

指定app的abi。特殊需求下，可以强制指定app在指定abi下运行。不指定的情况下默认根据`系统规则`来决定运行的abi。

可选参数：
* armeabi
* armeabi-v7a
* arm64-v8a

### 双开app实例代码：
```java
VAppInstallerParams params = new VAppInstallerParams(VAppInstallerParams.FLAG_INSTALL_OVERRIDE_NO_CHECK);
VAppInstallerResult result = VirtualCore.get().installPackage(Uri.parse("package:com.tencent.mobileqq"), params);
if (result.status == VAppInstallerResult.STATUS_SUCCESS) {
    Log.e("test", "install apk success.");
}
```

### 从sd卡安装apk实例代码：
```java
VAppInstallerParams params = new VAppInstallerParams(VAppInstallerParams.FLAG_INSTALL_OVERRIDE_NO_CHECK);
VAppInstallerResult result = VirtualCore.get().installPackage(Uri.fromFile(new File("/sdcard/test.apk")), params);
if (result.status == VAppInstallerResult.STATUS_SUCCESS) {
    Log.e("test", "install apk success.");
}
```

### 安装Split apk
先安装base包，然后再安装所有split包即可。
```java
File dir = new File("/sdcard/YouTube_XAPK_Unzip/");
VAppInstallerParams params = new VAppInstallerParams(VAppInstallerParams.FLAG_INSTALL_OVERRIDE_NO_CHECK);
VAppInstallerResult result = VirtualCore.get().installPackage(
        Uri.fromFile(new File(dir,"com.google.android.youtube.apk")), params);
for (File file : dir.listFiles()) {
    String name = file.getName();
    if (name.startsWith("config.") && name.endsWith(".apk")) {
        result = VirtualCore.get().installPackage(
            Uri.fromFile(file), params);
    }
}

```




