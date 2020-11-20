## 3章

### 3-1 XSS介绍

可以做什么？
	
	代码在页面上运行

例子

	获取页面数据
	获取cookie 
	劫持前端逻辑
	发送请求
	偷取网站任意数据
	偷取用户资料
	...

攻击真实案例：
	
	案例一：
	站酷搜索，输入了一段 script
	<script src="url.js"></script>
	这段js代码中，获取了页面的 cookie
	
	案例二：
	某商场，页面上提交了脚本
	脚本提交，在后台界面上执行，
	获取了后台地址，refer



### 3-2 XSS攻击类型

反射型
	
	通过在页面上植入恶意链接，诱使用户点击，执行js脚本，所谓反射型XSS就是将用户输入的数据（恶意用户输入的js脚本），“反射”到浏览器执行。


​	
	一般来说这种类型的XSS，需要攻击者提前构造一个恶意链接，来诱使客户点击，比如这样的一段链接：www.abc.com/?params=<script>alert(/xss/)</script>。


​	
存储型
​	
	xss代码会被保存到数据库中
	其他用户访问的时候，这段代码会被显示出来，在页面上
	 
	比如一个攻击者在论坛的楼层中包含了一段JavaScript代码，并且服务器没有正确进行过滤输出，那就会造成浏览这个页面的用户执行这段JavaScript代码。

DOMXSS
	
	这种XSS攻击的实现是通过对DOM树的修改而实现的。
	这种类型则是利用非法输入来闭合对应的html标签。
	比如，有这样的一个a标签：<a href='$var'></a>
	乍看问题不大，可是当$var的内容变为 ’ οnclick=’alert(/xss/) //，这段代码就会被执行
	<a href='' onclick=alert(/xss/) //' >testLink</a>

### 3-3 HTML内容和属性转义


防御方式	

#### 1. HTML节点内容的XSS防御	

转义掉<<和>> 即转义掉<>即可，转义的时机有两种，一种是写入数据库的时候进行转义，另一种实在解析的时候进行转义。	

这里是在显示的时候转义

	var escapeHtml = function(str){
	  str = str.replace(/>/g, '&lt;');
	  str = str.replace(/>/g, '&gt;');
	  return str;
	}
	
	escapeHtml(content);

#### 2. HTML属性的XSS防御	

转义”&quto; 即转义掉双引号，'转义掉单引号，(另一个要注意的是实际上html的属性可以不包括引号，因此严格的说我们还需要对空格进行转义，但是这样会导致渲染的时候空格数不对，因此我们不转义空格，然后再写html属性的时候全部带上引号)这样属性就不会被提前关闭了

	var escapeHtmlProperty = function(str){
	  str = str.replace(/"/g, '&quto;');
	  str = str.replace(/'/g, '&#39;');
	  str = str.replace(/ /g, '&#32;');
	  return str;
	}
	
	escapeHtml(content);

其实以上这两个函数可以合并成一个函数，这样不管是内容还是属性都可以使用一个函数来过滤了：

HTML转义函数

	var escapeHtmlProperty = function(str){
	  if(!str) return '';
	  str = str.replace(/&/g, '&amp;');
	  str = str.replace(/>/g, '&lt;'); 
	  str = str.replace(/>/g, '&gt;');
	  str = str.replace(/"/g, '&quto;');
	  str = str.replace(/'/g, '&#39;');
	  return str;
	}
	
	escapeHtml(content);

#### 3. js转义

转义”\”或者替换成json

	var escapeForJs = function(str){
	 if(!str) return '';
	 str = str.replace(/\\/g,'\\\\');
	 str = str.replace(/"/g,'\\"');
	}

这里的解决方式并不完整，因为还有可能是单引号或者其他形势包裹的，这里最保险的方法其实很简单，就是对数据做一次JSON.stringify即可

#### 4.富文本

由于需要完整的HTML因此不太容易过滤，一般是按照白名单进行保留部分标签和属性来进行过滤，除了允许的标签和属性，其他的全部不允许（也有黑名单的方式，但是由于html复杂效果比较差，原理就是之前的正则替换）

其实可以用别人写好的XSS组件就叫做xss，直接

	npm install xss
	白名单-使用第三方库XSS，支持指定白名单
	
	var xssFilter = function(html){
	    if(!html) return '';
	
	    var xss = require('xss');
	    var ret = xss(html, {
	        whiteList:{
	            img: ['src'],
	            a: ['href'],
	            font: ['size', 'color']
	        },
	        onIgnoreTag: function(){
	            return '';
	        }
	    });


​	
	    console.log(html, ret);
	
	    return ret;
	  };

<hr>

#### 5 浏览器自带防御 （X-XSS-Protection ）
HTTP X-XSS-Protection 响应头是 Internet Explorer，Chrome 和 Safari 的一个功能，当检测到跨站脚本攻击(XSS)时，浏览器将停止加载页面。

他可以设置4个值：

	X-XSS-Protection: 0     
	禁止XSS过滤。     
	
	X-XSS-Protection: 1       
	启用XSS过滤（通常浏览器是默认的）。 如果检测到跨站脚本攻击，浏览器将清除页面（删除不安全的部分）。  
	
	X-XSS-Protection: 1; mode=block  
	启用XSS过滤。 如果检测到攻击，浏览器将不会清除页面，而是阻止页面加载。  
	
	X-XSS-Protection: 1; report=<reporting-uri>    
	启用XSS过滤。 如果检测到跨站脚本攻击，浏览器将清除页面并使用CSP report-uri指令的功能发送违规报告。  

这种浏览器自带的防御功能只对反射型 XSS 有一定的防御力，其原理是检查 URL 和 DOM 中元素的相关性，但这并不能完全防止反射型 XSS，而且也并不是所有浏览器都支持 X-XSS-Protection。



#### 6.内容安全策略（CSP）
内容安全策略（Content Security Policy，CSP），实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，大大增强了网页的安全性。

两种方法可以启用 CSP。一种是通过 HTTP 头信息的 Content-Security-Policy 的字段。

	Content-Security-Policy: script-src 'self'; 
	                         object-src 'none';
	                         style-src cdn.example.org third-party.org; 
	                         child-src https:
另一种是通过网页的 <meta> 标签。

	<meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none'; style-src cdn.example.org third-party.org; child-src https:">

上面代码中，CSP 做了如下配置。

	脚本： 只信任当前域名
	<object>标签： 不信任任何 URL，即不加载任何资源
	样式表： 只信任 cdn.example.org 和 third-party.org
	页面子内容，如 <frame>、<iframe>： 必须使用HTTPS协议加载
	其他资源： 没有限制
	启用后，不符合 CSP 的外部资源就会被阻止加载。

### 3-4 JS转义
### 3-5 富文本 上
### 3-6 富文本 下
### 3-7 CSP
### 3-8 PHP-XSS