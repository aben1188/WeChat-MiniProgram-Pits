# WeChat-MiniProgram-Pits
微信小程序开发过程中所踩过的那些坑......

- **WebSocket URL中不能带有端口**

	用于创建WebSocket连接的wx.connectSocket API中的URL不能带有端口，比如下面的写法是错误的：
		wx.connectSocket({
			url: 'wss://xxx.abc.com:4343'
		})

	这是一个巨坑！坑就坑在，开发者工具上是允许指定端口的，不会报错，连个警告性的提示都没有，而到了真机上就报错“url not in domain list”，而这个错误信息又不知所云，很难让人想到是跟WebSocket相关的错误。而更坑的还在于，为了找到上述只在真机上报错的原因，在真机上打开vConsole调试功能，却又不报错了。这下怎么调试呢？只能通过采取弹modal的“笨办法”，逐步排查，最终才发现是创建WebSocket时报的错。
