# VA开发文档 #
本文档主要介绍2部分。第一部分是VA的代码结构介绍，这部分是为了让开发者能掌握VA的代码框架。第二部分是VA的SDK如何使用。
</br></br>  
**下面开始第一部分，代码结构介绍：**

## VA源码目录介绍 ##
可以看到VA源码一共有4个源码目录，各个目录介绍如下：  
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

## VA编译配置文件介绍 ##
**AppConfig.gradle：**
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

**VAConfig.gradle：**
<table >
        <tr>
            <th>配置名称</th>
            <th>作用</th>
        </tr>
        <tr  align="left">
            <th>VA_MAIN_PACKAGE_32BIT</th>
            <th>用于配置VA主包是32位还是64位，true位32位，false位64位</th>
        </tr>
        <tr  align="left">
            <th>VA_ACCESS_PERMISSION_NAME</th>
            <th>用于配置VA中4大组建的权限名称</th>
        </tr>
        <tr  align="left">
            <th>VA_AUTHORITY_PREFIX</th>
            <th>用于配置VA主包中ContentProvider的authorities/th>
        </tr>
        <tr  align="left">
            <th>VA_EXT_AUTHORITY_PREFIX</th>
            <th>用于配置VA插件包中ContentProvider的authorities</th>
        </tr>
        <tr  align="left">
            <th>VA_VERSION</th>
            <th>用于配置VA库版本</th>
        </tr>
        <tr  align="left">
            <th>VA_VERSION_CODE</th>
            <th>用于配置VA库版本代码</th>
        </tr>
</table>  

## VA核心代码解释 ##
1. `com.lody.virtual.client`包下的代码主要用于VA Framework中的APP Hook部分，里面完成了对各个Service的HOOK处理  
2. `com.lody.virtual.server包下的代码主要用于VA Framework中的APP Server部分，实现了对APP的安装处理以及其他不给Android系统处理的APP请求
3. `mirror`包下的代码主要用于对系统隐藏类的引用，较少大量反射代码的编写
4. `cpp`包下的代码主要用于VA Native部分，实现IO重定向和jni函数HOOK。其中：  
	- `substrate`中提供了针对arm32和arm64的hook
	- `vfs.cpp`中实现了VA的虚拟文件系统，用于控制APP对文件的访问限制
	- `syscall_hook.cpp`中实现了对IO的Hook
5. `DelegateApplicationExt.java`用于VA插件包，实现了对主包代码的加载执行

</br></br>
**下面开始第二部分，VA SDK使用介绍：**





