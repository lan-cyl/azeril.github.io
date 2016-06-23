---
layout: post
title: Introduction of my blog's source code
categories: [blog ]
tags: [Jekyll, ]
description: 看一看本博的代码
---


## \_config.yml

网站的配置信息，通过`site.xxx`获得，部分内容如下

````
# Site settings
title: "生生不息"
header-img: "img/home-bg-01.jpg"
tagline: ""
description: "源码之前 了无秘密"
baseurl: ""
#url: "http://azeril.me"
#url: "http://localhost:4000"

# About/contact
owner:
  name: "lan_cyl"
  email: caiyongle91@gmail.com
  bio: "源码之前 了无秘密"

# Data
gavatar: img/my.png
favicon: img/favicon.ico
````

## \_layouts/default.html

默认的页面样式，代码如下：

````
<!DOCTYPE html>
<html lang="en">

{% include head.html %}

<body ontouchstart="">

    {% include nav.html %}

    {{ content }}

    {% include footer.html %}

</body>

</html>

````

可以看出，头部内容存放在“\_includes/head.html”，包含文章标题、引用的样式等。

“\_includes/nav.html”中存放网站的导航栏，"Home/About Me/Tags"。

`{ { content } }`表示文件的内容，一般是我们的post文件，page文件的内容，例如index.html的内容、\_post/2016-06-23-Introduction-of-my-blog-source-code.md的内容等。

“footer.html”中存放网站的底部栏，外链、copyright。

head/nav/footer都没什么好说的，重点看下‘{ { content } }’部分。

## \_layouts/page.html

“layout: page”表示的文件，都会经过本文件的包装，之后整体作为content，再被default.html包装起来，来看下包装成啥样子了：

````
---
layout: default
description: single page's title and content, like About page.
---

<!-- Page Header -->
<header class="intro-header" style="background-image: url('{ { site.baseurl } }/{ % if page.header-img % }{ { page.header-img } }{ % else % }{ { site.header-img } }{ % endif % }')">
    <div class="container">
        <div class="row">
            <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-offset-1">
                <div class="site-heading">
                    <h1>{ % if page.title % }{ { page.title } }{ % else % }{ { site.title } }{ % endif % }</h1>
                    <hr class="small"/>
                    <span class="subheading">{ { page.description } }</span>
                </div>
            </div>
        </div>
    </div>
</header>

<!-- Main Content -->
<div class="container">
	<div class="row">
		<div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-offset-1">
			{ { content } }
		</div>
	</div>
</div>

<hr>

````

首先，设置头部图片，使用文件指定的herder-img图片，或者网站的默认图片。

然后，设置头部标题，使用文章指定的page.title，或者网站的默认标题。

最后，正文部分，Main Content，显示文件的内容。

## \_layouts/post.html

“layout: post”标示的文件，都会经过本文件的包装，之后整体作为content，再被default.html包装起来，来看下包装成啥样子了：

````
---
layout: default
description: post page, contains title, content, license, discuss, et al.
---

<!-- Post Header -->
<style type="text/css">
    header.intro-header{
        background-image: url('{{ site.baseurl }}/{% if page.header-img %}{{ page.header-img }}{% else %}{{ site.header-img }}{% endif %}')
    }
</style>
<header class="intro-header" >
    <div class="container">
        <div class="row">
            <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-offset-1">
                <div class="post-heading">
                    <div class="Tags">
                        {% for tag in page.tags %}
                        <a class="tag" href="/Tags/#{{ tag }}" title="{{ tag }}">{{ tag }}</a>
                        {% endfor %}
                    </div>
                    <h1>{{ page.title }}</h1>
                    {% if page.subtitle %}
                    <h2 class="subheading">{{ page.subtitle }}</h2>
                    {% endif %}
                    <span class="meta">Posted by {% if page.author %}{{ page.author }}{% else %}{{ site.title }}{% endif %} on {{ page.date | date: "%B %-d, %Y" }}</span>
                </div>
            </div>
        </div>
    </div>
</header>

<!-- Post Content -->
<article>
    <div class="container">
        <div class="row">
            <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-offset-1 post-container">

                {{ content }}

                <hr>

                <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
                  <img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" />
                </a><br />
                This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">CC A-S 4.0 International License</a>.

                <hr>

                <ul class="pager">
                    {% if page.previous.url %}
                    <li class="previous">
                        <a href="{{ page.previous.url | prepend: site.baseurl | replace: '//', '/' }}" data-toggle="tooltip" data-placement="top" title="{{page.previous.title}}">&larr; Previous Post</a>
                    </li>
                    {% endif %}
                    {% if page.next.url %}
                    <li class="next">
                        <a href="{{ page.next.url | prepend: site.baseurl | replace: '//', '/' }}" data-toggle="tooltip" data-placement="top" title="{{page.next.title}}">Next Post &rarr;</a>
                    </li>
                    {% endif %}
                </ul>

                <!-- Duoshuo Share start -->
                <style>
                    .ds-share{
                        text-align: right;
                    }

                    @media only screen and (max-width: 700px) {
                        .ds-share {

                        }
                    }
                </style>

                <div class="ds-share"
                    data-thread-key="{{page.id}}" data-title="{{page.title}}"
                    data-images="{{ site.url }}/{% if page.header-img %}{{ page.header-img }}{% else %}{{ site.header-img }}{% endif %}"
                    data-content="{{ content | strip_html | truncate:80 }} | Lancyl's blog"
                    data-url="{{site.url}}{{page.url}}">
                    <div class="ds-share-inline">
                      <ul  class="ds-share-icons-16">

                        <li data-toggle="ds-share-icons-more"><a class="ds-more" href="#">分享到：</a></li>
                        <li><a class="ds-wechat" href="javascript:void(0);" data-service="wechat">微信</a></li>
                        <li><a class="ds-weibo" href="javascript:void(0);" data-service="weibo">微博</a></li>
                        <li><a class="ds-douban" href="javascript:void(0);" data-service="douban">豆瓣</a></li>
                      </ul>
                      <div class="ds-share-icons-more">
                      </div>
                    </div>
                <hr>
                </div>
                <!-- Duoshuo Share end-->

                <!-- 多说评论框 start -->
                <div class="comment">
                    <div class="ds-thread" data-thread-key="{{page.id}}" data-title="{{page.title}}" data-url="{{site.url}}{{page.url}}"></div>
                </div>
                <!-- 多说评论框 end -->
            </div>
        </div>
    </div>
</article>

````

首先，头部图片、Tags标签、标题、Post by信息。

然后，正文部分，post内容显示、版权信息、上下一页按钮、多说分享、多说评论。


## 参考
* [打造你的 GitHub Pages 专属博客](http://azeril.me/blog/Build-Your-First-GitHub-Pages-Blog.html)
* [Jekyll 博客主题精选](http://azeril.me/blog/Selected-Collection-of-Jekyll-Themes.html)
* [Git 教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)  
* [Jekyll 语法简单笔记](http://github.tiankonguse.com/blog/2014/11/10/jekyll-study.html)
* [Jekyll/Liquid API 语法文档](http://alfred-sun.github.io/blog/2015/01/10/jekyll-liquid-syntax-documentation/)
