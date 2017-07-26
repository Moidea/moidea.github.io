## 网络调用 API 的跨域访问解决方案

**问题发生日期** 

2017/07/26

**问题基本简述** 

使用 https://www.btc123.com/api 访问接口数据时出现跨域问题

**解决方案详细描述**

Chrome 报错提示为：`XMLHttpRequest cannot load https://www.btc123.com/api/getTicker?symbol=chbtcetccny. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://www.moidea.info' is therefore not allowed access.`

因为第一次遇见这个问题，咨询了QQ群里的一些小伙伴，得知这是JS的跨域问题，那么什么是跨域？

## 一、什么是跨域

首先我们分析一下域名地址的组成：

> http:// www . google : 8080 / script/jquery.js
>
> 　　http:// （协议号）
>
> 　　www  （子域名）
>
> 　　google （主域名）
>
> 　　 8080 （端口号）
>
> 　　script/jquery.js （请求的地址）
>

**当协议、子域名、主域名、端口号中任意一各不相同时，都算不同的“域”。**

**不同的域之间相互请求资源，就叫“跨域”。**

## 二、出现跨域问题的情况

由于在工作中需要使用AJAX请求其他域名下的请求，但是会出现拒绝访问的情况，这是因为基于安全的考虑，AJAX只能访问本地的资源，而不能跨域访问。

比如说你的网站域名是aaa.com，想要通过AJAX请求bbb.com域名中的内容，浏览器就会认为是不安全的，所以拒绝访问。

会出现跨域问题的几种情况：

| 编号   | url                                      | 说明           | 是否允许通信 |
| ---- | ---------------------------------------- | ------------ | ------ |
| 1    | http://www.a.com/a.js  http://www.a.com/b.js | 同一域名下        | 允许     |
| 2    | http://www.a.com/a/a.js  http://www.a.com/b/b.js | 同一域名不同文件夹    | 允许     |
| 3    | http://www.a.com:8080/a.js  http://www.a.com:9090/a.js | 同一域名不同端口号    | 不允许    |
| 4    | http://www.a.com/a.js  https://www.a.com/b.js | 同一域名不同协议     | 不允许    |
| 5    | http://www.a.com/a.js  http://192.168.4.158/b.js | 域名与域名对应的ip地址 | 不允许    |
| 6    | http://www.a.com/a.js  http://wwww.a.com/b.js | 主域名相同，子域名不同  | 不允许    |
| 7    | http://www.a.com/a.js  http://www.b.com/b.js | 不同域名         | 不允许    |

比如：http://www.a.com/index.html 请求 http://www.b.com/index.html 的数据 

## **三、处理跨域的方法1 -- 代理**

这种方式是通过后台(ASP、PHP、JAVA、ASP.NET)获取其他域名下的内容，然后再把获得内容返回到前端，这样因为在同一个域名下，所以就不会出现跨域的问题。

比如在A（[www.a.com/sever.php](http://www.a.com/sever.php)）和B（[www.b.com/sever.php](http://www.b.com/sever.php)）各有一个服务器，A的后端（[www.a.com/sever.php](http://www.a.com/sever.php)）直接访问B的服务，然后把获取的响应值返回给前端。也就是A的服务在后台做了一个代理，前端只需要访问A的服务器也就相当与访问了B的服务器。这种代理属于后台的技术，所以不展开叙述。

这个也是推荐大家使用的方法，因为很多时候我们外调第三方开放的接口都是没有接口后台处理权限的，在没有权限的情况下只能在本地通过动态语言来获取原接口数据，返回给前端调用，因为这里调用的 API 后端没有使用 JSONP ，所以只能自己在本地通过 PHP curl 去取接口数据返回给自己使用。我在本地写了一个 phpserver.php ，PHP 代码如下：

```php
<?php $connomains = array(
"https://www.btc123.com/api/getTicker?symbol=huobibtccny",
"https://www.btc123.com/api/getTicker?symbol=huobiethcny",  
"https://www.btc123.com/api/getTicker?symbol=huobibtccny"
);

$mh = curl_multi_init();

foreach ($connomains as $i => $url) {
     $conn[$i]=curl_init($url);
      curl_setopt($conn[$i],CURLOPT_RETURNTRANSFER,1);
      curl_setopt($conn[$i], CURLOPT_SSL_VERIFYPEER, false); //https支持
      curl_setopt($conn[$i], CURLOPT_SSL_VERIFYHOST, false); //https支持
      curl_multi_add_handle ($mh,$conn[$i]);
      
}

do { $n=curl_multi_exec($mh,$active); } while ($active);

foreach ($connomains as $i => $url) {
      $res[$i]=curl_multi_getcontent($conn[$i]);
      curl_close($conn[$i]);
}

print_r($res);?>
```

## **四、处理跨域的方法2 -- JSONP**

假设在 [http://www.a.com/index.php](http://www.a.com/index.php) 这个页面中向 [http://www.b.com/getinfo.php](http://www.b.com/getinfo.php) 提交GET请求，那么我们在 [www.a.com](http://www.a.com/) 页面中添加如下代码：

```javascript
//创建一个script元素

var  Scr = document.reateElement('script');

//声明类型

Scr.type='text/javascript';

//添加src属性，引入跨域访问的url

Scr.src='http://www.b.com/gerinfo.php';

//在页面中添加新创建的script元素

document.getElementsByTagName('head')[0].appendChild(Scr)
```

当GET请求从 [http://www.b.com/getinfo.php](http://www.b.com/getinfo.php) 返回时，可以返回一段JavaScript代码，这段代码会自动执行，可以用来负责调用 [http://www.a.com/index.php](http://www.a.com/index.php) 页面中的一个callback函数。看下面一个列子：

```javascript
<script>

　　alert('hello  我是b');

</script>
```

在 [www.b.com](http://www.b.com/) 页面中：

注意：JSONP只支持 “GET” 请求，但不支持 “POST” 请求。

## **五、处理跨域的方法3 -- XHR2**

“XHR2” 全称 “XMLHttpRequest Level2” 是HTML5提供的方法，对跨域访问提供了很好的支持，并且还有一些新的功能。

\* IE10以下的版本都不支持

\* 只需要在服务器端头部加上下面两句代码：

header( "Access-Control-Allow-Origin:*" );

header( "Access-Control-Allow-Methods:POST,GET" );

使用該方法只能處理服務端，也就是接口開發方，下面為簡潔代碼：[在接口返回數據時設置header內容]

```php
<?php  
$ret = array(  
    'name' => isset($_POST['name'])? $_POST['name'] : '',  
    'gender' => isset($_POST['gender'])? $_POST['gender'] : ''  
);  
  
header('content-type:application:json;charset=utf8');  
  
$origin = isset($_SERVER['HTTP_ORIGIN'])? $_SERVER['HTTP_ORIGIN'] : '';  
  
$allow_origin = array(  
    'http://www.client.com',  
    'http://www.client2.com'  
);  
  
if(in_array($origin, $allow_origin)){  
    header('Access-Control-Allow-Origin:'.$origin);  
    header('Access-Control-Allow-Methods:POST');  
    header('Access-Control-Allow-Headers:x-requested-with,content-type');  
}  
  
echo json_encode($ret);  
?>  
```

因為本次使用的接口屬於第三方提供的，所以沒有後端的控制權限，無法實現方法二中的 JSONP 和 方法三中提到的 XHR2 。

------

**引用文章或网址，以提供更多细节或相关信息**

暫無
