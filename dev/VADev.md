# VA开发文档 #
本文档主要介绍2部分。  
第一部分是VA的代码结构介绍，这部分是为了让开发者能了解VA源码框架。  
第二部分是VA的SDK如何使用。  
</br>
**下面开始第一部分，代码结构介绍：**

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


详见：[https://github.com/asLody/VirtualApp-Priv/wiki](https://github.com/asLody/VirtualApp-Priv/wiki)




