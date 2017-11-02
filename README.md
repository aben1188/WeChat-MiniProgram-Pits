# WeChat-MiniProgram-Pits
## 微信小程序开发过程中的那些坑

<br>

- #### **WebSocket URL中不能带有端口**

	用于创建WebSocket连接的wx.connectSocket API中的URL不能带有端口，比如下面的写法是错误的：

		wx.connectSocket({
			url: 'wss://xxx.abc.com:4343'
		})

	这是一个巨坑！坑就坑在，开发者工具上是允许指定端口的，不会报错，连个警告性的提示都没有，而到了真机上就报错“url not in domain list”，而这个错误信息又不知所云，很难让人想到是跟WebSocket相关的错误。
而更坑的还在于，为了找到上述只在真机上报错的原因，在真机上打开vConsole调试功能，却又不报错了。这下怎么调试呢？只能通过采取弹modal的“笨办法”，逐步排查，最终才发现是创建WebSocket时报的错。
（这里提示一下，出现不知所云的错误，首先应该用错误信息到微信小程序官方论坛上搜索，看是否有解决方案。我当时头脑一时短路，忘了先到官方论坛上查一下，结果浪费不少时间）

	2017/10/16

- #### **腾讯位置服务(即QQ地图服务)中的两个坑**

	1、开发者工具报错：“合法域名校验出错”。合法域名不是设了吗，怎么还报错？原来，微信公众平台中设置合法域名列表，包括request、socket、uploadFile、downloadFile四类，每一类都允许设置多个（我原来以为每一类只能设一个）。其中，request中，必须将https://apis.map.qq.com 也加进去（都是你们鹅厂一家的，咋就不直接列入白名单中呢，还要设置啥合法域名呐~~~）。

	2、上一个域名校验出错的问题解决后，又报“请求来源未被授权”的错误。截止到今天(2017/10/16)，这个错误提示，官方文档中没说，在腾讯位置服务官方论坛和微信小程序官方论坛上也都搜索不到真正有用的答案，百度、Google也没用，某种程度上比上述WebSocket的坑还坑！
其实，只要将域名“servicewechat.com”添加到腾讯位置服务的“密钥(key)管理-应用类型(选“浏览器”)-授权域名”中就可以了，而不是添加你自己小程序后台的访问域名，因为小程序前台是托管在微信服务器上的（但如果是后台访问QQ地图服务，则必须将小程序后台的访问域名添加到授权域名中）。（这里同样还是忍不住要吐槽一下，都是你们鹅厂一家的，咋就不直接列入白名单中呢，还要设置授权域名干嘛呢~~~）。

	2017/10/16

- #### **.wpy文件中注释掉一个view不生效（基于wepy框架开发遇到的问题）**

	参见wepy仓库中的这个[ issue ](https://github.com/wepyjs/wepy/issues/418)。不过据issue中wepy开发者Gcaufy的回复，这个问题他一直无法复现，很可能是特定于IDE的问题。目前的解决方案是一旦出现这个问题，只能重新手动wepy build一次。
	
	2017/10/16

- #### **请求URL中的中文必须先编码**

	请求URL中的中文必须先用encodeURI()或encodeURIComponent()编码，否则在安卓Android真机上发出的请求URL中的中文在服务端收到的是乱码，奇怪的是开发者工具和iOS真机上则无此问题。此问题在官方文档和官方论坛上均找不到答案。
	
	2017/10/17

- #### **安卓版toast显示不换行**

	安卓真机上toast超过7个中文字符就会显示不全，超出部分被隐藏了，而不是像iOS上自动换行。
	
	2017/10/17

- #### **showToast、showLoading、hideToast、hideLoading会互相影响**

	以前微信小程序没有loading效果的API，只能通过toast来实现。自基础库1.1.0开始多了wx.showLoading()和wx.hideLoading()。但是注意，loading本质上和toast是一回事，可认为loading是toast的封装版，适用于专门针对需要显示数据加载提示的场景。

	因此：

		wx.showToast({title: "成功"}); 
		wx.hideLoading();

	toast会一闪而过，这是因为它刚显示出来就被hideLoading()隐藏掉了。

	因为二者本质上是同一个控件。所以：
	
		后面的 hideLoading 会隐藏前面的 showToast；
		后面的 hideToast 会隐藏前面的 showLoading；
		后面的 showLoading 会覆盖前面的 showToast，从而显示 Loading；
		后面的 showToast 会覆盖前面的 showLoading，从而显示 Toast。
	
	2017/10/24

- #### **引入自定义模块文件时，必须使用相对路径（基于wepy框架开发遇到的问题）**

	引入自定义模块时，千万不能只写模块名称，即便被引入的模块文件与所引入它的那个文件在同一个目录下，也必须在其前面写上一般可省略不写的表示当前目录的“./”，否则可能导致依赖于node环境的wepy在编译时错误引入node的同名核心模块或其他目录下的同名第三方模块(比如node_modules目录中的同名模块)，从而报错：
	```
	[Error] Error: 找不到模块: crypto
	被依赖于: E:\LypData\MyDev\zxq\node_modules\request\lib\helpers.js。
	请尝试手动执行 npm install crypto 进行安装。
	```
	![image](https://github.com/aben1188/WeChat-MiniProgram-Pits/blob/master/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20171101205428.png)
	
	这是由于某个文件需要引入同目录下的request文件，于是按照一般的写法习惯性地省略了表示当前目录的“./”，写作了：
	```
	import request from 'request'
	```
	而在node环境下，不能省略表示当前目录的“./”，必须写作：
	```
	import request from './request'
	```
	具体可参考：[《require()源码解读（阮一峰）》](http://www.ruanyifeng.com/blog/2015/05/require.html)一文的第一节“一、require() 的基本用法”。

	
- #### **使用了wepy-plugin-autoprefixer后build时报错（基于wepy框架开发遇到的问题）**

	使用了wepy-plugin-autoprefixer之后，将会导致src目录下的.wxss文件在wepy build时出错：TypeError: Cannot read property 'type' of null：
	
	![image](https://github.com/aben1188/WeChat-MiniProgram-Pits/blob/master/images/QQ%E6%88%AA%E5%9B%BE20171102162656.jpg)
	
	可参见wepy-plugin-autoprefixer仓库中的这个[issue](https://github.com/li-xianfeng/wepy-plugin-a)。
	
    这是由src目录下的.wxss文件所导致，具体原因不明。目前的解决办法是，将src目录下的.wxss文件改为.scss文件（由于scss的语法兼容css语法，而wxss实际就是css，因此只需改文件后缀即可，无需修改文件内容）

	2017/11/02

### （未完待续，欢迎pr！）
