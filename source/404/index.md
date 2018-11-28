---
title: 404 Not Found
date: 2018-05-12 09:56:25
permalink: /404
---

<html>
<head>
<meta charset="utf-8">
<title>404页面</title>
<style>
*{margin:0;padding:0;outline:none;font-family:\5FAE\8F6F\96C5\9ED1,宋体;-webkit-user-select:none;-moz-user-select:none;-ms-user-select:none;-khtml-user-select:none;user-select:none;cursor:default;font-weight:lighter;}
.center{margin:0 auto;}
.whole{width:100%;height:100%;line-height:100%;position:fixed;bottom:0;left:0;z-index:-1000;overflow:hidden;}
.whole img{width:100%;height:100%;}
.mask{width:100%;height:100%;position:absolute;top:0;left:0;background:#FFFFFF;opacity:0.6;filter:alpha(opacity=60);}
.b{width:100%;text-align:center;height:400px;position:absolute;top:50%;margin-top:-230px}.a{width:150px;height:50px;margin-top:30px}.a a{display:block;float:left;width:150px;height:50px;background:#fff;text-align:center;line-height:50px;font-size:18px;border-radius:25px;color:#333}.a a:hover{color:#000;box-shadow:#fff 0 0 20px}
p{color:#000;margin-top:40px;font-size:24px;}
#num{font-weight:bold;color:#FF0000;}
</style>
<script type="text/javascript">
	var num=6;
	function redirect(){
		num--;
		document.getElementById("num").innerHTML=num;
		if(num<0){
			document.getElementById("num").innerHTML=0;
			location.href="https://lyl873825813.github.io/";
			}
		}
	setInterval("redirect()", 1000);
</script>
</head>

<body onload="redirect();">
<div class="whole">
	
    <div class="mask"></div>
</div>
<div class="b">
		<h2 class="post-title" itemprop="name headline">非常抱歉~您访问的页面不存在喔！</h2>
		<img src="http://localhost:4000/medias/page/404.jpg">
		<p><span id="num"></span> 秒后自动跳转到主页</p>
	</div>

</body>
</html>  
