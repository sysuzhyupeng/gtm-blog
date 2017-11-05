Google Analytics介绍
-----
在了解gtm之前，需要先了解Google Analytics，简称为ga统计，ga统计主要解决这几个问题：

 * 集中统计页面的访问ip、页面访问量、访问设备等
 * 集中统计页面上用户的操作，比如点击、提交表单等事件
 * 其他运营需要的数据整合

有关ga的更详细介绍可以参考[ga中文文档](https://developers.google.com/analytics/devguides/collection/analyticsjs/ "ga中文文档") 

开始ga统计
-----
开始使用ga数据需要在页面加载一段脚本
```javascript
    <!-- Google Analytics -->
    <script>
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
    m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
    })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

    ga('create', 'UA-XXXXX-Y', 'auto');
    ga('send', 'pageview');
    </script>
    <!-- End Google Analytics -->
```
以上代码完成了这样四件事：

 * 创建了一个 < script> 元素，并开始从 https://www.google-analytics.com/analytics.js 异步下载 analytics.js JavaScript 库。
 * 初始化了一个全局函数 ga（也称为 ga() 命令队列），可以通过该函数来安排要在 analytics.js 库加载完毕可供使用时执行的命令。
 * 在 ga() 命令队列中添加一条命令，为通过 ‘UA-XXXXX-Y’ 参数指定的媒体资源创建一个新的跟踪器对象。
 * 在 ga() 命令队列中添加另一条命令，为当前页面向 Google Analytics服务器发送网页浏览数据。

简单来说创建了一个ga的方法让我们可以直接调用。
在加载脚本之后，可以通过这样的编码来创建事件统计：
```javascript
    //ga的方法之前已经通过加载脚本定义
    ga('send', 'event', [eventCategory], [eventAction], [eventLabel], [eventValue], [fieldsObject]);
``` 
那么我们如果统计下载按钮的点击次数，需要这样来做：
```javascript
    var btn = document.getElementById('btn');
    btn.onclick = function(){
        ga('send', 'click', '下载按钮', '点击', 'feb页面');
    }
```
这时在点击之后，ga就会向统计后台发送一个点击事件的请求。
其他相关的ga用法就不举例了，`因为我们的重点是gtm`。

Google Tag Manager
-----
我们从上述ga的方案中可以发现，ga有一个问题：
在前端代码中需要对手动对ga事件的发送进行硬编码，ga相关的代码嵌入我们的代码后，会产生这样两个问题：

 * ga代码一定要由前端人员维护
 * 当想修改ga代码的时候，要走一遍发布上线流程(因为修改到前端代码)，非常麻烦

那么Google Tag Manager，这里简称gtm，就是为了解决我们这个问题。gtm将我们的事件绑定迁移到后台，可以实时对统计进行修改。这样就可以做到：

 * 运营人员可以维护统计
 * 修改统计直接修改后台，不用处理前端代码

gtm脚本
-----
gtm的基础脚本由运营这里提供，类似于以下的脚本，要嵌入到页面代码中
```javascript
    //将此代码粘贴到页面的 <head> 部分中，并使其尽可能靠近顶部位置： 
    <!-- Google Tag Manager -->
    <script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
    new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
    j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
    'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
    })(window,document,'script','dataLayer','GTM-5C6N5G4');</script>
    <!-- End Google Tag Manager -->


    //此外，请将此代码粘帖到紧跟起始 <body> 标记之后的位置： 
    <!-- Google Tag Manager (noscript) -->
    <noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-5C6N5G4"
    height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
    <!-- End Google Tag Manager (noscript) -->

```

gtm使用
-----
gtm的使用主要由三部分组成：

 * 代码：添加到页面的代码
 * 触发器（Trigger）：决定哪些代码能触发
 * 变量（Variable）：用于接收和存储数据，被代码和Trigger使用

分别对应gtm管理页面的三个标签的内容：
    
![Alt text](/img/1.png)

在我们之前下载按钮的例子里面，我们进一步分解下原本在代码中的实现：我们为下载按钮绑定了一个回调函数，在它被点击的时候发送统计请求。

那么这个下载按钮被点击的条件可以称之为触发器，统计的代码可以称之为代码。

触发器是一个condition，当它为true的时候，触发绑定在触发器上的代码。变量用来保存重复出现的值。

也就是说，代码和触发器组成了我们的统计事件绑定，同时触发器和代码中会用变量来保存一些重复值，这三个封装起来的功能让我们在使用的时候不需要进行编码。这样说可能太抽象，下面演示下具体操作：

建立gtm事件统计
-----
首先我们在内置变量中要进行勾选(后面的触发器会用到)，在变量栏把这两个变量勾上

![Alt text](/img/5.png)

因为gtm有内置的点击监听，我们可以直接使用触发器对点击进行条件筛选。

我们在触发器栏新建一个触发器

![Alt text](/img/2.png)


然后填写名称和筛选条件，这里元素可以选择仅链接(a标签)或者所有元素，因为我们以元素类名(class)作为筛选，所以这里先选所有元素

![Alt text](/img/3.png)

然后选某些点击

![Alt text](/img/4.png)

这样我们就设定了一个触发器，当页面点击的时候可以筛选除类名为btn的按钮，接下来我们需要去触发统计代码。

新建一个代码，关于统计的字段(类别、操作等)由运营这里提供，将触发器设置为我们刚才新建的触发器。

![Alt text](/img/6.png)

接下来看到成功之后，保存

![Alt text](/img/7.png)

发布与调试
-----
全部保存之后就可以发布到本地或者线上调试了

![Alt text](/img/8.png)

点击调试即可直接在线上调试，如果确定更改就点发布。

现在只要我们页面上有类名为btn的元素，就会不断触发统计了，可以在f12中看到调用栈。到这里统计就完成了！

![Alt text](/img/9.png)

可以通过chrome浏览器的扩展程序Google Tag Assistant对ga安装和gtm安装进行监控。


后续
-----
上面的过程完全可以由运营操作。前端只需要定义类名为btn的元素即可。

除了gtm自带的监听，我们也可以通过自定义事件进行硬编码处理复杂逻辑。

facebook在gtm上的集成 
-
首先复制由运营提供的facebook pixel参数，这些原本应该被放置在html的script标签中，我们把它集成到gtm中就不需要进行编码
```javascript
    <!-- Facebook Pixel Code -->
    <script>
      !function(f,b,e,v,n,t,s)
      {if(f.fbq)return;n=f.fbq=function(){n.callMethod?
      n.callMethod.apply(n,arguments):n.queue.push(arguments)};
      if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
      n.queue=[];t=b.createElement(e);t.async=!0;
      ....
    <!-- End Facebook Pixel Code -->
```
然后在gtm中新建一个代码(英文翻译是tag)，命名为FB - Base Pixel。设置类型为`Custom HTML`，并把代码粘贴上去。

![Alt text](/img/10.png)

然后点击高级设置中的`每次网页加载触发一次`(英文为Once per page)，设置触发条件为`所有页面`

![Alt text](/img/11.png)

接下按下保存即可，这时每个引入gtm的页面都会触发facebook的pixel脚本加载。

使用gtm预览之后，可以添加chrome浏览器的插件扩展程序Facebook Pixel Helper对facebook统计脚本进行监控。

facebook事件统计
-
还是以下载事件的统计为例，运营发给我四个统计代码，用来区分4个下载按钮(这样区分按钮可能值得讨论)。事件代码如下：
```javascript
    1）<script>
      fbq('track', 'Purchase');
    </script>

    2）<script>
      fbq('track', 'Lead');
    </script>

    3）<script>
      fbq('track', 'CompleteRegistration');
    </script>

    4）<script>
      fbq('track', 'AddPaymentInfo');
    </script>
```
这里的Purchase，Lead，CompleteRegistration都是facebook统计的不同事件，可以参考链接：https://www.facebook.com/business/help/402791146561655

接下来我们为facebook统计添加事件统计。我们新建一个代码，用来统计第一个按钮，可以命名为FB-下载按钮1，并将运营发过来的统计代码粘贴上去。

![Alt text](/img/12.png)

这里我们还是使用gtm内置的点击事件，新建一个触发器，设置如下：

![Alt text](/img/13.png)

再将刚才代码的触发器设置为这个触发器，这时html中类名为download1的链接被点击时候将触发相应的代码。

保存之后，使用预览就可以看到事件统计了。其他按钮和这个操作相似，通过建立触发器和相应的facebook代码来完成统计。


