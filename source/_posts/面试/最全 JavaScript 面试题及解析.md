---
title: 最全 JavaScript 面试题及解析
date: 2025-10-30 14:42:34
category: 程序人生
tags: 面试
---

1、怎样辨别一个变量是数组还是字符串？

可以借助typeof操作符。

2、“\==”与“===”的区别？

"=="是相等运算符，会进行隐式类型转换。
"==="是严格运算符，会同时判断值和类型。

3、如何去除字符串中的所有空格？

function trim(str){
return str.replace(/\s|\xA0/g,"");    
}

4、如何获取当前浏览的URL？

window.location.href

5、怎样添加、删除、移动、复制、创建和查找节点？

1）创建新节点

createDocumentFragment() //创建一个DOM片段

createElement() //创建一个具体的元素

createTextNode() //创建一个文本节点

2）添加、移除、替换、插入

appendChild() //添加

removeChild() //移除

replaceChild() //替换

insertBefore() //插入

3）查找

getElementsByTagName() //通过标签名称

getElementsByName() //通过元素的Name属性的值

getElementById() //通过元素Id，具有唯一性

实现一个函数clone，可以对JavaScript中的5种主要的数据类型(包括Number、String、Object、Array、Boolean)进行值复制。

6、什么是闭包？

闭包就是能够读取其他函数内部变量的函数。

7、JS中const、let、var之间的区别

1）const定义的变量不可以修改，而且必须初始化。

2）var定义的变量可以修改，如果不初始化会输出undefined，不会报错。

3）let是块级作用域，函数内部使用let定义后，对函数外部没有影响。

8、什么叫同源策略？有什么用？

同源策略是指域名、协议、端口都相同，只有同源的脚本才会被执行。

9、什么是Ajax？适用于哪些应用场景？

Ajax是一种无需重新加载整个网页，就能更新部分网页的技术。
例如：无刷新判断验证码是否输入正确等。

10、Ajax请求时get和post有什么区别？

1）get参数放在url后面，post放在http body里面
2）有大小限制，get受url长度限制，post受内存限制
3）安全方面，get明文传输安全性较差
4）应用场景不同，get用于获取数据，post用于提交数据

11、谈谈Jsonp的原理？

动态创建script标签，结合回调函数。

12、split()和join()函数的区别？

前者是将字符串切割成数组，后者是将数组转换成字符串。

13、写出3个使用this的典型应用。

1）事件：如onclick，this指向发生事件的对象
2）构造函数：this指向new出来的对象
3）call/apply：改变this的指向

14、怎么使用JS改变元素的class？

object.ClassName = xxxx

15、普通事件与绑定事件有什么区别？

普通事件只支持单个事件处理，而事件绑定可以添加多个事件处理。

16、请描述js里事件的三个阶段。

捕获、处于目标阶段、冒泡阶段(IE8及以下版本只支持冒泡)

17、数组方法pop()、push()、unshift()、shift()都有什么用？

push()：向数组尾部添加元素
pop()：删除数组尾部的元素
unshift()：向数组头部添加元素
shift()：删除数组头部的元素

18、IE和DOM事件流的区别是什么？

1）执行顺序不同
2）参数不同
3）事件是否加on
4）this指向不同

19、call和apply有什么区别？

call与apply的区别在于参数的写法不同:

a.call(b,arg1,arg2,...)
a.apply(b,[arg1,arg2,...])

20、什么是事件委托？

利用事件冒泡的原理，让子元素触发的事件，由其父元素代替执行。

21、JS中的本地对象、内置对象和宿主对象分别有哪些？

本地对象：array、obj、regExp等可以通过new实例化的对象
内置对象：Math等不可以实例化的对象
宿主对象：document、window等浏览器自带的对象

22、document load 和 document ready 的区别是什么？

load必须等到页面内包括图片的所有元素加载完毕后才能执行。ready是DOM结构绘制完成后就执行，不必等到所有资源加载完毕。

23、你怎么理解WebPack、Grunt和Gulp？

该标准从一开始就是针对JavaScript语言制定的，之所以不叫JavaScript，有两个原因。一是商标问题，Java是Sun公司的商标，根据授权协议，只有Netscape公司可以合法使用JavaScript这个名字，且JavaScript本身也已被Netscape公司注册为商标。二是为了体现这门语言的制定者是ECMA，而非Netscape，这样有利于保证语言的开放性和中立性。

因此，ECMAScript和JavaScript的关系是，前者是后者的规格，后者是前者的一种实现（另外的ECMAScript方言还有Jscript和ActionScript）。在日常使用中，这两个词可以互换。

26、ES6有哪些新特性？

箭头操作符；对class的支持（包含constructor构造函数）；不定参数...x；let和const关键字；for
of遍历；模块支持import；promise异步函数处理模式（pending等待中；resolve返回成功，reject返回失败）；

27、typeof都会返回什么数据类型？

Object、number、function、boolean、undefined

28、请例举3种强制类型转换和2种隐式类型转换。

29、如何截取字符串javastack中的java部分？

30、如何规避javascript多人开发时的函数重名问题？

31、检测一个变量是否是String类型有哪几种方式？

1）typeof(obj) == 'string'
2）obj.constructor == String;

32、请说出至少三种减少页面加载时间的方法。

1）压缩css、js文件
2）合并js、css文件，减少http请求
3）将外部js、css文件放在页面底部
4）减少dom操作，尽可能用变量替代不必要的dom操作

33、请详细解释Ajax的工作原理。

1）创建ajax对象（XMLHttpRequest/ActiveXObject(Microsoft.XMLHttp)）
2）判断数据传输方式(GET/POST)
3）打开链接 open()
4）发送 send()
5）当ajax对象完成第四步（onreadystatechange）数据接收后，判断http响应状态（status）在200-300之间或者为304（缓存）时，执行回调函数

34、JS中有哪几种函数？

具名函数（命名函数）和匿名函数。

35、请至少写出3种创建函数的方式。

1）声明函数

function fn1(){}

2）创建匿名函数表达式

var fn1 = function (){}

3）创建具名函数表达式

var fn1 = function javastack(){};

36、什么是跨域？解决跨域的方法有哪些？

由于浏览器的同源策略，只要发送请求的url的协议、域名、端口三者中任意一个与当前页面地址不同，就属于跨域。

解决方案：JSONP、CORS、代理、服务器端处理等。