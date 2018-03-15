#### 通信方式

1. android原生调用js：通过loadUrl("url")和evaluateJavascript("url", callback)
    * loadUrl：加入调用一个js里面的 call(var test) 方法，客户端调用方式  loadUrl("javascript:call(" + "test" + ")");
    * evaluateJavascript("url", callback)：因为该方法的执行不会使页面刷新，而第一种方法（loadUrl ）的执行则会。Android 4.4 后才可使用。假如调用一个js里面的 call(var test) 方法，客户端调用方式  evaluateJavascript("javascript:call(" + "test" + ")", callback);通过callback将js的执行结果回调回来。
2. js调用android：通过addJavaScriptInterface()进行映射，或者通过shouldOverrideUrlLoading()方法回调拦截js发送过来的url数据。WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）方法数据拦截。必须设置setJavaScriptEnabled(true)，打开跟js交互的权限。
    * addJavaScriptInterface：继承object，随便定义一个方法然后使用 @JavascriptInterface注释需要调用的方法，通过webView.addJavaScriptInterface()方法进行映射。