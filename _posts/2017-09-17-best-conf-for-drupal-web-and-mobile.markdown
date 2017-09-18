---
layout: post
title:  "Drupal站点 单数据库+手机端&WEB端 跳转配置"
date:   2017-09-15 07:55:40 +0800
categories: Drupal8
---

随着移动端开发的工作逐渐流行，越来越多的网站需要同时提供手机端和web端。Drupal8作为著名的CMF系统，自然也不例外要同时进行web端和手机端的配置。本文目的是提供nginx进行服务器端的判断和跳转，同时包括Drupal系统内两端访问的配置说明。

1.nginx配置

{% highlight ruby %}

server {
  listen 80;
  server_name www.example.com;
  
  set $mobile_rewrite do_not_perform;  
    
  if ($http_user_agent ~* "(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino") {  
      set $mobile_rewrite perform;  
  }  
    
  if ($http_user_agent ~* "^(1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-)") {  
      set $mobile_rewrite perform;  
  }  
    
  if ($http_cookie ~ 'gotoweb=true') {  
      set $mobile_rewrite do_not_perform;  
  }  

  add_header Access-Control-Allow-Origin *;

  root /Users/bluers/git/code/example;
  index index.html index.htm index.php;

  location / {
    try_files  $uri  $uri/  /index.php?$query_string;
    index index.php;
    if ($mobile_rewrite = perform){
      rewrite ^(.*)$ http://m.example.com$1;
    }
  }

  location ~ \.php$ {
    fastcgi_index   index.php;
    fastcgi_pass    127.0.0.1:9000;
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
    include mime.types; 
  }
}
server {
  listen 80;
  server_name m.example.com;
  
  root /Users/bluers/git/code/example;
  index index.html index.htm index.php;

  location / {
    try_files  $uri  $uri/  /index.php?$query_string;
    index index.php;
  }

  location ~ \.php$ {
    fastcgi_index   index.php;
    fastcgi_pass    127.0.0.1:9000;
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
    include mime.types; 
  }
}

{% endhighlight %}

2.Drupal8系统内配置

经过笔者测试，目前市面上并没有可用的基于host进行不同theme调用的模块。作为抛砖引玉的素材，本文希望能给读者有所启发。

2.1 现可用模块
经过搜索，发现可用的模块包括：domain switch theme、themekey等，其描述是作为一个drupal系统两个theme的模块提供的。但是themekey目前只有drupal7版本，而domain switch theme虽然支持drupal8，但在笔者反复加载和测试后没有生效，看其更新频率也够快，经过再三考虑，确定放弃了这种思路。

2.2 一端使用drupal，另一端使用接口获取数据
使用drupal提供基于www.example.com的web端访问，然后在drupal的目录中建立m文件夹，用于更新静态文件提供给手机端访问。其中手机端的数据来自于web端提供的restful接口。
经过测试，这种方式是可行的。其优势在于可以更方便的控制手机端的文件大小和性能，缺点在于在实际的产出中，会需要依据项目的规模提供给手机端比较多的接口。

2.3 两端均基于drupal，利用css的媒体查询来使用不同的样式。
手机端的宽度一般都是800PX以下，因此自建theme中同时调用web端的样式文件以及M端的样式文件。不同的是，web端样式文件最上方增加媒体查询，如下：
{% highlight css %}
@media all and (min-width: 800px){
}
{% endhighlight %}
而M端则使用另一种查询：
{% highlight css %}
@media all and (max-width: 800px){
}
{% endhighlight %}
这样虽然用户在手机端和WEB端都加载了同样大小的资源文件，但笔者测试后发现，通过优化文件内容和合并压缩，drupal可以将这类文件优化到300K以内。

3. 判断设备后跳转

跳转时通常有三种思路，一种是通过服务器跳转，另一种是通过PHP或Drupal跳转，第三种是通过javaScript跳转。通过笔者测试，这三种均可实现，相比起来在服务器端跳转是最为方便可靠的。

3.1 服务器跳转
跳转代码参见1。这种是从服务器直接302跳转，没有加载无用的代码，且性能最高。

3.2 通过PHP或Drupal跳转
通常可以将这部分判断写到自制的Theme中比较方便，或是加载到自定义block中，前者执行更快。参考代码如下：

{% highlight php %}

	function isPhone() {
	    if (isset($_SERVER['HTTP_X_WAP_PROFILE'])) {
	        return true;
	    }
	    if (isset($_SERVER['HTTP_VIA'])) {
	        //找不到为flase,否则为true
	        if(stristr($_SERVER['HTTP_VIA'], "wap"))
	        {
	            return true;
	        }
	    }
	    if (isset($_SERVER['HTTP_USER_AGENT'])) {
	        $clientkeywords = array (
	            'nokia',
	            'sony',
	            'ericsson',
	            'mot',
	            'samsung',
	            'htc',
	            'sgh',
	            'lg',
	            'sharp',
	            'sie-',
	            'philips',
	            'panasonic',
	            'alcatel',
	            'lenovo',
	            'iphone',
	            'ipod',
	            'blackberry',
	            'meizu',
	            'android',
	            'netfront',
	            'symbian',
	            'ucweb',
	            'windowsce',
	            'palm',
	            'operamini',
	            'operamobi',
	            'openwave',
	            'nexusone',
	            'cldc',
	            'midp',
	            'wap',
	            'mobile',
	            'phone',
	        );
	        // 从HTTP_USER_AGENT中查找手机浏览器的关键字
	        if (preg_match("/(" . implode('|', $clientkeywords) . ")/i", strtolower($_SERVER['HTTP_USER_AGENT']))) {
	            return true;
	        }
	    }
	    //协议法，因为有可能不准确，放到最后判断
	    if (isset($_SERVER['HTTP_ACCEPT'])) {
	        // 如果只支持wml并且不支持html那一定是移动设备
	        // 如果支持wml和html但是wml在html之前则是移动设备
	        if ((strpos($_SERVER['HTTP_ACCEPT'], 'vnd.wap.wml') !== false) && (strpos($_SERVER['HTTP_ACCEPT'], 'text/html') === false || (strpos($_SERVER['HTTP_ACCEPT'], 'vnd.wap.wml') < strpos($_SERVER['HTTP_ACCEPT'], 'text/html')))) {
	            return true;
	        }
	    }
	    return false;
	}
	function exampletheme_preprocess_page(&$variables) {
		if(isPhone() == true){
			header("Location: http://m.example.com");
			exit();
		}
	}

{% endhighlight %}

3.3 通过javaScript跳转

代码部分很简单：

{% highlight javascript%}
var ua = navigator.userAgent;
if(/mobile/.test(ua)){
	window.location.href = "m.example.com";
}

{% endhighlight %}

这种方式虽然也可以实现，代码可以写在html.html.twig中，也可以写在theme加载的js文件中。但都有几个缺点。例如从www.example.com跳转到m.example.com，两者都会加载一定大小的中间数据，并且从js文件中跳转会加载几乎全部的web端文件。

综合以上方法，笔者选择了服务器302跳转、CSS媒体查询，同时提供手机端和WEB端的服务。








