---
layout: post
title: 树莓派安装jekyll，与github搭建blog
category: Raspberry Pi
keywords: 树莓派,Raspberry Pi,jekyll,github
---

之前的blog是使用wordpress，空间用的是东京的linode。由于目前的资源利用比较低，一年也得200多刀，不打算继续使用，转而使用树莓派+github上搭建自己的blog。已经在树莓派上成功的搭建了动态域名解析，可以查看我的上一篇文章。但是由于树莓派的性能有限，将wordpress放上去打开浏览器访问查看top，cup使用率前几秒100%。这样就不可取，然后想到使用静态页面。这里就需要自己选择了使用哪个语言哪个静态系统：go下有gor，ruby下有jekyll等，之前尝试使用了jekyll在github上搭建过blog，再则目前还没接触过go，所以选择了jekyll。  

**以下是wordpress迁移到树莓派下安装搭建jekyll的安装步骤:**  

1.**安装jekyll**  
<pre>
sudo apt-get install jekyll
</pre>  


2.**创建基本目录结构**  
<pre>
|-- _config.yml
|-- _includes
|-- _layouts
|   |-- default.html
|   `-- post.html
|-- _posts
|   |-- 2013-05-01-hello-world.html
|-- _site
`-- index.html
</pre>
_config.yml是网站的一些配置信息，如你的site信息  

_includes目录放着复用的文件，比如网站的头部，底部文件，这样就可以在页面使用include包含进来  

_layouts是模板文件存放目录  

_posts是blog需要发布的文章内容存放的目录，其中的文件必须符合YAML Front Matter标准，
头部需要---开头，---结尾。如：
	---
	layout: post
	title: Blogging Like a Hacker
	--- 
 _site是静态文件生成的目录，即运行jekyll命令后产生的  
index.html即blog的首页，格式必须符合符合YAML Front Matter标准，指定使用的模板


3.**注册disqus，添加评论功能**  
注册dispus挺简单的，填写相关的信息之后，使用Universal Code生成自己网站的评论代码。

	<div  id="disqus_thread"></div>
	<script type="text/javascript">
	    /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
	    var disqus_shortname = 'hisenking'; // required: replace example with your forum shortname

	    /* * * DON'T EDIT BELOW THIS LINE * * */
	    (function() {
	        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
	        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
	        (document.getElementsByTagName('head')[0] 
	        || document.getElementsByTagName('body')[0]).appendChild(dsq);
	    })();
	</script>
	<noscript>
		Please enable JavaScript to view the 
		<a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a>
	</noscript>
	<a href="http://disqus.com" class="dsq-brlink">
		comments powered by <span class="logo-disqus">Disqus</span>
	</a>  

然后将这段代码添加到_layouts下的post.html


4.**迁移wordpress内容**  
官方有教程，地址：https://github.com/mojombo/jekyll/wiki/blog-migrations
由于不想在树莓派上安装过多没用的东西，所以自己尝试写了一个php的脚本，对文章进行生成静态的html文件。
这里最好将文件名以及分类改成英文的，不然可能报编码错误。脚本代码如下：

	<?php
	    $connect = mysql_connect("localhost","root","123456") or die("cannot link");
	    mysql_select_db("wordpress",$connect);
	    mysql_query("set names utf8");

	    $sql = "SELECT ID,post_title,post_name,post_content,post_date FROM wp_posts WHERE post_type='post'";
	    $query = mysql_query($sql);
		$posts = array();
	    while($result = mysql_fetch_array($query)){
	        if(empty($result) || empty($result['post_content'])){
	            continue;
	        }
	        $title = $result['post_title'];
	        $posts[$title] = $result;
	    }

	    if(!empty($posts)){
	        foreach($posts as $post){
	            $name = $post['post_name'];
	            $title = $post['post_title'];
	            $date = $post['post_date'];
	            $content = $post['post_content'];

	            $perfix = array(
	                    '---',
	                    'layout: post',
	                    'title: ' . $title,
	                    '---'
	            );
	            $fileName = date("Y-m-d", strtotime($date)) . '-' . urldecode($name) . '.html';
	            $fileContent = implode("\n", $perfix) . '\n' . $content;
	            file_put_contents('./' . $fileName, $fileContent);
	         }
	    }  
	    echo "ok";


5.**生成静态文件,并查看效果**  
<pre>
jekyll
jekyll --server
</pre>  
其中jekyll，默认会在当前目录生成_site目录，jekyll --server默认会使用4000端口，可以之前在浏览器打开预览
这里还可以修改默认生成的静态文件目录，比如之前在你的web根目录下生成，这样就可以直接使用域名访问查看了。还有就是在YAML头部可以添加published，设置为false，默认为tue，控制这篇文章是否生成在站点中
这里还需要添加一个404页面，内容可以自己指定

6.**注册github，创建自己的github page**  
如我的id是hisen则需要创建hisen.github.com这个仓库，在这个目录下放静态文件，则可以使用hisen.github.com直接在浏览器打开。
如果想使用自己的域名可以在目录下创建CNAME文件，文件内容写上自己的域名即可。如我要是用laoniangke.com的二级域名，则写入github.laoniangke.com
再使用DNSPOD创建一条CNAME记录，将域名指向github.laoniangke.com。之后偶访问hisen.github.com就会跳转到这个域名了

7.**将本地测试的代码提交到github中**  
当然你也可以使gitbub发布文章，再从github同步到你的树莓派中


**总结**  
最后推荐sublime的安装Markdown Preview，进行编辑预览相当方面。  
使用github还得知道基本的操作命令  
在本地克隆版本库  
<pre>
git clone https://github.com/hisen/hisen.github.com.git
cd hisen.github.com
</pre>

新增一个文件  
<pre>touch index.html</pre>

添加内容  
<pre>echo "hello world" > index.html</pre>

告知git  
<pre>git add index.html</pre>
如果修改了很多则可以直接使用
<pre>git add .</pre>

删除文件  
<pre>git rm index.html</pre>

提交  
<pre>git commit -m "update website"</pre>

推送到github  
<pre>git push origin master</pre>



参考资料：  
https://github.com/mojombo/jekyll/wiki/Template-Data  
http://wowubuntu.com/markdown/  
https://github.com/mojombo/jekyll/wiki/Sites  
https://help.github.com/articles/my-custom-domain-isn-t-working  
http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html