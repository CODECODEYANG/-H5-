# -H5-
微信小程序和H5间的通信

1.h5向小程序发送消息，根据官方文档（https://developers.weixin.qq.com/miniprogram/dev/component/web-view.html），网页向小程序 postMessage 时，会在特定时机（小程序后退、组件销毁、分享）触发并收到消息。小程序页面通过 bindmessage 绑定的函数读取 post 信息。

2.微信小程序怎么向H5发送消息呢？

目前常用的方法是通过设置webview指向网页的链接（url）拼接参数，然后H5页面截取url中的参数的方式来通信。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

基于以上两种相互间的通信和传参，现在要解决H5页面A，跳转到H5页面B，在页面B中做了一些操作（这些操作会改变了页面A中的一些数据），然后返回到页面A，希望将这些改变反馈到页面A。

如果页面A跳转到页面B是使用的location.href，操作完成后完返回上一页（页面A）用的是window.history.go(-1)。这时是不会触发visibilitychange事件，可以使用pageshow事件
window.addEventListener('pageshow', (event) => {

    if (event.persisted) { // 是否从缓存读取

    }

})

2.第二种方式是使用hashchange来处理的，具体操作如下：

页面B中发送postMessage，增加标识符test，值为test_时间戳，用来告知webview，返回上一页面的时在页面A的url后拼接#test_。（建议使用时间戳，如果是固定值，hashchange之后执行一次）

webview中接收页面B发送的消息postMessage，判断是否存在test，如果有则将test的值存入全局变量。在webview的onShow中判断全局变量中是否有test，如果有，修改webview的src，增加hash参数#test_，如果没有则不增加。清除全局变量test。由于只修改了hash部分，页面不会重新刷新。

页面A中绑定hashchange事件，hashchange事件执行自定义逻辑方法，读取hash参数，调用window.history.go(-1)，恢复history。
