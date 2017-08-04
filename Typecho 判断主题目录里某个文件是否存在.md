最近需要实现一个功能是在网站里做一个热门标签栏目，并在该页面显示热门标签的对应图片，效果见：

![https://github.com/Moidea/moidea.github.io/blob/master/%E6%A0%B7%E5%BC%8F.jpg](样式)

首先想到的就是将 tag 命名的统一格式的图片放置在主题目录里，用 PHP 函数进行判断该图片是否存在来显示标签图片，不过该方法有以下几个问题：

### 1、标签对应图片格式问题

为了统一我在主题目录里建立了一个 tags 的目录来存放标签对应图片，并且要求 tag 对应的图片必须统一为 png 格式图片。

### 2、主题目录的图片命名问题

用 tag 标签的 slug 来为图片命名，但是此操作必须要设置 slug ，然而 typecho 新建文章时设置标签并不支持同时设置标签的 slug ，用户想要设置 tag 对应的 slug 需要到标签菜单单独去设置，考虑到大部分人可能并不会去设置，所以此方案暂时不采用。然后考虑用 tag 标签的 name 来为图片命名，但是这样同样产生了问题，标签名为中文的问题，图片直接设置成标签名的中文名称。

### 3、标签对应的图片不存在的问题

因为有时候发布的文章太多，势必会导致可能有些 tag 太多了，忘了在 tags 目录放置某些 tag 对应的图片，这时就需要先判断 tag 对应图片是否存在，如果存在则调用显示，不存在则调用默认的图片来占位。

中间踩了几个坑，这里也记录一下，各位看官可以注意一下：

**1、使用 http://www.moidea.info/RedType/usr/themes/RedType/tags/test.png  判断图片是否存在， is_file() 函数输出始终为 false ，不过可以通过 fopen() 函数进行判断，可能带上 HTTP 协议的链接被 PHP 理解为远程文件的缘故，必须通过管道数据流判断，解决方法就是将 HTTP 协议图片改成本地图片相对路径 `usr/themes/RedType/tags/test.png`**

**2、使用 `usr/themes/RedType/tags/中文标签名.png` 中文标签名图片，if_file() 函数仍然始终输出为 false ，打印文件地址发现并没有错误，猜测是因为文件是 utf-8 格式，才导致判断错误，解决方法是将 utf-8 格式的文件地址转成 gbk2312 格式**

### 4、代码实现过程

上面分析结束我们直接上代码：

**1、新建页面模板**

```php+HTML
<?php
/**
* 热门精选页面
*
* @package custom
*/
$this->need('header.php');?>

<div class="page-tags">
<?php
$this->widget('Widget_Metas_Tag_Cloud', 'sort=count&ignoreZeroCount=1&desc=1&limit=20')->to($tags); // 调用标签20个
if($tags->have()):
	while ($tags->next()): ?>
	<div class="tag-box">
      <?php $url = 'usr/themes/RedType/tags/'.$tags->name.'.png';
						$url = iconv('UTF-8','GB2312',$url);
						if( is_file($url) ){ ?>
							<img src="<?php $this->options->themeUrl(); ?>tags/<?php $tags->name(); ?>.png" title="<?php $tags->name(); ?>" alt="<?php $tags->name(); ?>">
						<?php } else { ?>
							<img src="<?php $this->options->themeUrl(); ?>tags/default.png" title="<?php $tags->name(); ?>" alt="<?php $tags->name(); ?>">
						<?php } ?>
      </div>
  <?php endwhile; 
endif; ?>    
</div>

<?php $this->need('footer.php'); ?>  
```

**2、后台新建页面选择上述模板**

----

### 4、尾记

本文参考了 qqdie 的文章 《[php判断文件是否存在](https://qqdie.com/archives/php-to-determine-whether-the-file-exists.html)》 ，可以看到我也在 qqdie 跌倒的坑里跌了一跤，不过 qqdie 文中是使用的 file_exits() 函数，在文件不存在时 is_file() 要比 file_exits() 函数稍微慢一点，除此之外，is_file() 都要比 file_exits() 函数快，所以我这边还是建议大家使用 is_file() 函数。

关于 is_file() 函数和 file_exits() 函数速度测试，请看下面内容：

```php
<?php
$start_time = get_microtime();
for($i=0;$i<10000;$i++)//默认1万次，可手动修改
{
if(is_file('test.txt')) {
//do nothing;
}
}
echo 'is_file-->'.(get_microtime() - $start_time).'<br>';
$start_time = get_microtime();
for($i=0;$i<10000;$i++)//默认1万次，可手动修改
{
if(file_exists('test.txt')) {
 //do nothing;
}
}
echo 'file_exits-->'.(get_microtime() - $start_time).'<br>';
function get_microtime()//时间
{
list($usec, $sec) = explode(' ', microtime());
return ((float)$usec + (float)$sec);
}
?>
```

is_file() 和 file_exists() 效率比较，结果当文件存在时，is_file() 函数比 file_exists() 函数速度快14倍，当文件不存在时，两者速度相当。同理，当文件目录存在时，is_dir() 比 file_exists() 快18倍。不存在时两者效率相当。PHP 的 file_exists = is_dir + is_file。

* 如果要判断目录是否存在，请优先考虑函数 is_dir(directory)
* 如果要判断文件是否存在，请优先考虑函数 is_file(filepath)

### 5、引用

1. https://qqdie.com/archives/php-to-determine-whether-the-file-exists.html
2. http://www.cnblogs.com/xuan52rock/p/4548635.html
