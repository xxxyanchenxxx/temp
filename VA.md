[![VA banner](https://raw.githubusercontent.com/asLody/VirtualApp/master/Logo.png)](https://github.com/asLody/VirtualApp)

## VA是什么? ##
VirtualApp(简称：VA)是一款运行于Android系统的沙盒产品，可以理解为轻量级的“Android虚拟机”。其产品形态为高可扩展，可定制的集成SDK，您可以基于VA或者使用VA定制开发各种看似不可能完成的项目。VA目前被广泛应用于APP多开、手游境外加速、手游租号、手游手柄免激活、区块链、移动办公安全、军队政府数据隔离、手机模拟信息、脚本自动化、插件化开发、无感知热更新、云控等技术领域。

## VA中的术语 ##
<table >
        <tr>
            <th>术语</th>
            <th>解释</th>
        </tr>
        <tr  align="left">
            <th>宿主</th>
            <th>集成VirtualApp类库（lib）的App叫做宿主</th>
        </tr>
        <tr  align="left">
            <th>宿主插件</th>
            <th> 用于在同一个手机,运行另一种ABI的宿主包,又称做插件包,扩展包,宿主插件包,宿主扩展包</th>
        </tr>
        <tr  align="left">
            <th>虚拟App/VApp</th>
            <th>VA的虚拟环境多开的app</th>
        </tr>
        <tr  align="left">
            <th>外部App</th>
            <th>手机真实环境安装的app</th>
        </tr>
</table>  

## VA技术架构 ##
![](https://github.com/xxxyanchenxxx/temp/blob/master/va1.jpg)  
可以看到，VA技术一共涉及到了Android的APP层，Framework层，以及Native层。  
为什么要分为3层呢？ 这也是必不可少的。  
因为一个APP想要在Android系统上运行，必须要安装了后系统才会接纳你。可是安装到VA内部的APP实际上并没有安装到系统中，所以正常情况下是无法运行的。那如何才能让它运行呢？  
答：那就只有“欺骗”系统，让系统认为已经安装好了。而这个“欺骗”过程就是VA Framework的核心工作内容，也是整个VA的核心技术原理。    
**下面介绍下在这3个层次分别做了什么事情：**</br>
<table >
        <tr>
            <th>层次</th>
            <th>主要工作</th>
        </tr>
        <tr  align="left">
            <th>VA Space</th>
            <th>由VA提供了一个内部的空间，用于安装要在其内部运行的APP，这个空间是系统隔离的。</th>
        </tr>
        <tr  align="left">
            <th>VA Framework</th>
            <th>这一层主要给Android Framework和VAPP做代理，这也是VA的核心。VA提供了一套自己的VA Framework，处于Android Framework与VA APP之间。对于VAPP，其访问的所有系统Service均已被App Hook模块拦截住，它会修改VAPP的请求参数，将其中与VAPP安装信息相关的全部参数修改为宿主的参数之后发送给Android Framework（有部分请求会发送给自己的VA Server直接处理而不再发送给Android系统）。这样Android Framework收到VAPP请求后检查参数就会认为没有问题，待Android系统对该请求处理完成返回结果时，VA Framework同样也会拦截住该返回结果，此时再将原来修改过的参数全部还原为VAPP请求时发送的。这样VAPP与Android系统的交互也就能走通了。</th>
        </tr>
        <tr  align="left">
            <th>VA Native</th>
            <th>在这一层主要为了完成2个工作，IO重定向和VA APP与Android系统交互的请求修改。IO重定向是因为可能有部分APP会通过写死的绝对路径访问，但是如果APP没有安装到系统，这个路径是不存在的，通过IO重定向，则将其转向VA内部安装的路径。另外有部分jni函数在VA Framework中无法hook的，所以需要在native层来做hook。</th>  
        </tr>
</table> 
</br>
**总结：**</br>
通过上面的技术架构可以看到，VA内部的APP实际是跑在VA自己的VA Framework之上。  
VA已将其内部APP的全部系统请求都拦截住，通过这项技术也能对APP进行全面的控制，而不仅仅只是多开。并且为了方便开发者，VA还提供了SDK以及Hook SDK。


## VA进程架构 #
![](https://github.com/xxxyanchenxxx/temp/blob/master/va_process.jpg)    
可以看到，VA运行时有5类进程：CHILD进程，VA Host Main进程，VA Host Plugin进程，VAPP Client进程，VAServer进程。  
VA为了同时支持32位APP与64位APP，需要安装2个包：一个主包，一个插件包(在本文档中主包是32位，插件包是64位)。  
2个包也是必须的，因为一个包只能运行在一种模式下，要么32位，要么64位。所以对于32位的APP，VA使用32位的主包去运行，对于64位的APP，VA则使用64位的插件包去运行。  
在主包中含了VA的所有代码，插件包中只有一段加载主包代码执行的代码，无其他代码。所以插件包几乎不用更新，只需要更新主包即可。  
另外主包是选择用32位还是64位，可以在配置文件中修改(比如对于要上GooglePlay的用户，会修改为主包64位，插件包32位)。  

**各类进程的作用与解释如下：**</br>
<table >
        <tr>
            <th>进程类型</th>
            <th>作用</th>
        </tr>
        <tr  align="left">
            <th>CHILD</th>
            <th>由VA Host集成的其他进程，比如：保活进程，推送进程等。</th>
        </tr>
        <tr  align="left">
            <th>VA Host Main</th>
            <th>VA主包的UI主界面所在的进程。默认主包是32位，插件包是64位，可在配置文件中修改切换。</th>
        </tr>
        <tr  align="left">
            <th>VA Host Plugin</th>
            <th>支持64位APP的插件包所在进程。默认主包是32位，插件包是64位，可在配置文件中修改切换。</th>
        </tr>
        <tr  align="left">
            <th>VAPP Client</th>
            <th>安装到VA中的APP启动后产生的进程，在运行时会将io.busniess.va:pxxx进程名修改VAPP的真实进程名。</th>  
        </tr>
        <tr  align="left">
            <th>VAServer</th>
            <th>VA Server的所在的进程，用于处理VA中不交予系统处理的请求。比如APP的安装处理。</th>  
        </tr>
</table>  


## VA能满足您的一切需求##
通过上面的技术架构，我们可以了解到VA可以对APP进行全面的控制，并且提供了Hook SDK，几乎能满足您在各个领域的一切需求：  
1. 可以满足您的**双开/多开**需求    
VA可以让您在同一部手机上安装多个微信/QQ/WhatsApp/Facebook等APP，实现一部手机，多个账号同时登录。  

2. 可以满足您的**移动安全**需求  
VA提供了一整套内部与外部的隔离机制，包括但不限于(文件隔离/组件隔离/进程通讯隔离)，简单的说VA内部就是一个“完全独立的空间”。
通过VA可将工作事务与个人事务安全的隔离，互不干扰。稍作定制即可实现应用行为审计、数据加密、数据采集、数据防泄漏、防攻击泄密等移动安全相关的需求。    
    **2.1 应用行为审计**  
通过VA提供的HOOK能力可以实现实时监测用户使用行为，将违规信息上传到服务器；并能轻易实现诸如时间围栏(在某个时间段内能否使用应用的某个功能)、地理围栏(在某个区域内能否使用应用的某个功能)、敏感关键字过滤拦截等功能需求。  
	**2.2 数据加密**  
通过VA提供的HOOK能力可以实现对应用的全部数据/文件加密，保证数据/文件落地安全。   
	**2.3 数据采集**  
通过VA提供的HOOK能力可以实现应用数据的实时无感上传需求，如聊天记录、转账记录等，防止事后删除无法追溯。  
	**2.4 数据防泄漏**  
通过VA提供的HOOK能力可以实现应用防复制/粘贴、防截屏/录屏、防分享/转发、水印溯源等需求。   
	**2.5 防攻击泄密**  
通过VA提供的应用管控能力可以将APP获取短信/通讯录/通话记录/后台录音/后台拍照/浏览历史/位置信息等隐私相关的行为完全控制在沙盒中，防止木马/恶意APP获取到用户真实的隐私数据，造成泄密等严重后果。  

3. 可以满足您的**免ROOT HOOK**需求  
VA提供了Java与Native的Hook能力，通过VA，您可以轻易实现诸如虚拟定位、改机、APP监控管理、移动安全等各种场景需要的功能。  

4. 可以满足您的**APP静默安装**需求  
VA提供了APP静默安装，静默升级，静默卸载的能力。如应用商店或游戏中心在集成VA后可以避免需要用户手动点击确认安装的操作，做到下载后立即安装到VA内，给用户带来“小程序”搬的体验，彻底避免了应用不易被用户安装上的问题。  

5. 可以满足您的**APP管控**需求  
通过VA提供的应用管控能力，您可以轻易控制VA中APP的运行界面或运行流程，实现如脚本自动化、特定界面插屏弹窗、修改APP界面内容，修改APP与服务器交互内容等需求。  

6. 可以满足您的**海外市场**需求  
VA实现了对Google服务的支持，以支持海外的App运行，比如Twitter、Messenger、WhatsApp、Instagram、FaceBook、Youtube等。  

7. 可以满足您**几乎一切能想到**的需求  
VA对于内部的App具有完全的监管和控制能力，几乎能满足您的一切需求！

8. 同时VA也是该技术领域__唯一一款__对外商业授权的产品    
截止目前已有**上百家**授权客户在付费使用VirtualApp商业版代码，集成VirtualApp代码的APP__日启动__次数__超过2亿次__，众多安卓工程师向我们提供不同场景下的用户反馈，通过我们技术团队不断优化迭代，不断提升产品性能与兼容性！

## VA的集成 ##
第1步：在您的Application中调用VA接口```VirtualCore.get().startup()```来启动VA引擎  
第2步:调用VA接口```VirtualCore.get().installPackageAsUser()```将目标APP安装到VA中  
第3步:调用VA接口```VirtualCore.get().installPackageAsUser()```启动APP    
**仅通过以上3个API就完成了基础使用，VA已屏蔽了复杂的技术细节，并提供了接口API，让您的开发变得很简单！**

## VA与其他技术方案对比 ##
针对APP的管控部分，以下是所有可能的技术方案：  
<table >
        <tr>
            <th>技术方案</th>
            <th>原理简介</th>
            <th>点评</th>
            <th>运行性能</th>
            <th>兼容稳定性</th>
            <th>项目维护成本</th>
        </tr>
        <tr  align="left">
            <th>二次打包</th>
            <th>通过反编译目标APP，加入自己的控制代码，重新打包</th>
            <th>1.现在的APP几乎都有加固或防篡改保护，重打包已是一件非常困难的事</br> 2.手机系统也会检测APP是否被重打包，如果重打包，会直接提示用户存在安全风险，甚至不让安装</br>3.针对每一个APP，甚至每一个版本都要深入去逆向分析，耗时耗力，难于维护</th>
            <th>优秀</th>
            <th>差</th>
            <th>非常大</th>
        </tr>
        <tr  align="left">
            <th>定制ROM</th>
            <th>通过定制系统源码，编译刷到指定手机</th>
            <th>只能针对指定的内部手机，无法扩展</br></th>
            <th>优秀</th>	
            <th>优秀</th>
            <th>大</th>
        </tr>
        <tr  align="left">
            <th>ROOT手机</th>
            <th>通过ROOT手机，刷入xposed等类似框架</th>
            <th>1.现在ROOT手机本身已是一件很难的事</br>2.现实中也很难让用户能去ROOT自己的手机</br></br></br></th>
            <th>优秀</th>
            <th>一般</th>
            <th>一般</th>
        </tr>
        <tr  align="left">
            <th>Android虚拟机</th>
            <th>重量级虚拟机，在Android系统上再跑一个定制的Android系统</th>
            <th>1.太过重量级，对手机性能要求很高<br> 2.因为要自己做硬件访问，所以对设备访问不可避免天然存在诸多兼容性问题，也很难解决<br>3.需要维护一套系统源码，并在虚拟机，设备硬件之间做适配</br></br></br></th>
            <th>差</th>
            <th>差</th>
            <th>大</th>
        </tr>
        <tr  align="left">
            <th>VA</th>
            <th>轻量级虚拟机，速度快，设备要求低</th>
            <th>无以上风险点，但是如果未被商业授权使用，将给公司带来法律风险。</br></br></br></th>
            <th>优秀，与原生应用几乎一样的速度</th>
            <th>优秀，有上百家企业在同时测试反馈</th>
            <th>低，VA提供了API并有专业的技术团队保障项目稳定运行</th>
        </tr>
    </table>


## VA的兼容稳定性##
VA已被**上百家**企业进行了广泛测试，包含**数十家上市公司高标准**的测试及反馈，几乎涵盖了海内外的各种机型设备和场景！
为您的稳定运行提供了充分的保障！  

截止目前，支持的系统版本:

<table>
        <tr>
            <th>5.0</th>
            <th>支持</th>
        </tr>
        <tr>
            <th>5.1</th>
            <th>支持</th>
        </tr>
        <tr>
            <th>6.0</th>
            <th>支持</th>
        </tr>
        <tr>
            <th>7.0</th>
            <th>支持</th>
        </tr>
        <tr>
            <th>8.0</th>
            <th>支持</th>
        </tr>
        <tr>
            <th>9.0</th>
            <th>支持</th>
        </tr>
        <tr>
            <th>10.0</th>
            <th>支持</th>
        </tr>
        <tr>
            <th>11.0</th>
            <th>支持</th>
        </tr>
        <tr>
            <th>12.0</th>
            <th>该系统还Google还未发布，已在支持准备中</th>
        </tr>
    </table>

支持的APP类型:
<table>
        <tr>
            <th>APP类型</th>
            <th>是否支持</th>
        </tr>
        <tr>
            <th>32位APP</th>
            <th>支持</th>
        </tr>
        <tr>
            <th>64位APP</th>
            <th>支持</th>
        </tr>
    </table>

支持的HOOK类型:
<table>
        <tr>
            <th>Hook类型</th>
            <th>是否支持</th>
        </tr>
        <tr>
            <th>Java Hook</th>
            <th>支持</th>
        </tr>
        <tr>
            <th>Native Hook</th>
            <th>支持</th>
        </tr>
    </table>

## 集成VA遇到问题如何反馈？ ##
购买授权后，会有专门的技术问题反馈群。我们有多名技术工程师随时在线接受问题反馈，会在第一时间处理！

## VA开发文档 ##
VA的详细开发文档请参考：[https://github.com/asLody/VirtualApp-Priv/wiki](https://github.com/asLody/VirtualApp-Priv/wiki)


