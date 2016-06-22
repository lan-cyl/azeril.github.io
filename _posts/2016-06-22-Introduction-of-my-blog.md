---
layout: post
title: Introduction of my blog
categories: [blog ]
tags: [Jekyll, Blog, ]
description: 使用github pages打造一个得心应手的blog
---


## Process of publish

#### Use github and git

1.登陆[github官网](www.github.com)，注册账号，记下账号名、邮箱、密码

2.安装git，ubuntu下直接安装即可

``` shell
$ sudo apt-get install git
```

3.配置git的用户名、邮箱

``` shell
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

4.创建ssh-key，用于同github通信

```` shell
$ ssh-keygen -t rsa -C "youremail@example.com"
````

5.github中添加 New SSH key

复制.ssh/id_rsa.pub公钥给github->Personal settings->SSH and GPG keys->New SSH key

(为什么GitHub需要SSH Key呢？因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。)

git的使用可以参看[廖雪峰的教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

到此，可以使用git操作github了～～～

#### Fork a blog theme

1.挑选一个自己喜欢的主题[参看azeril挑选的主题](http://azeril.me/blog/Selected-Collection-of-Jekyll-Themes.html)

2.fork到自己的github，然后在Settings里设置Repository name为"Your Name.github.io"

3.克隆到本地

```` shell
$ git clone https://github.com/lan-cyl/lan-cyl.github.io.git
````

好了，自己改改_post文件夹，就可以发布blog了，当然其他的文件也可以看着改一下～～～

## Jekyll 语法简单笔记

### 文件介绍

#### \_config.yml文件

jekyll的全局配置文件，包含 网站名、域名、链接格式等等信息。

#### \_includes文件夹

对于网站的头部, 底部, 侧栏等公共部分, 为了维护方便, 单独存放在该文件夹内, 使用的时候包含进去即可。

引入语法：`{ % include filename% }`

#### \_layouts文件夹

对于网站的布局,我们一般会写成模板的形式,这样对于写实质性的内容时,比如文章,只需要专心写文章的内容, 然后加个标签指定用哪个模板即可。对于内容,指定模板了模板后,我们可以称内容是模板的儿子。

在模板中, 引入儿子的内容：`{ { content } }`

一般包含default.html(默认页面样式), page.html(页面的样式), post.html(发布的内容的显示样式)。

在儿子中，指定父节点模板，例如 指定使用post模板：

```
---
layout: post
---
```

#### \_post文件夹

下的内容，比如博客放在这里

#### index.html文件

主页文件，后缀有时也用index、.md，根据自己的需要来写。

#### img文件夹

存放图片，然后就可以相对引用

### 模板语法

#### 头部定义

头部定义主要用于指定模板(layout)和定义一些变量, 比如 标题(title), 描述(description), 分类(category/categories), tags, 是否发布(published), 自定义变量。

```
---
layout: post
title: Introduction of my blog
categories: [blog ]
tags: [Jekyll, Blog, ]
description: 使用github pages打造一个得心应手的blog
---
```

写个自己的博客到此也就OK了，但是你还想折腾折腾样式啥的，好吧，语法定义在此～～～

具体样例可以参看\_includes、\_layouts等文件夹下的模板文件

#### 模板语法

##### 使用变量

全局根节点有：

- site \_config.yml中配置的信息
  - site.time 运行jekyll的时间
  - site.pages 所有页面
  - site.posts 所有文章
  - site.related_posts 类似的10篇文章
  - site.static_files
  - site.html_pages 所有的html页面
  - site.collections
  - site.data \_data目录下的所有数据
  - site.documents
  - site.categories 所有的分类
  - site.tags 所有的tag
  - site.[CONFIGURATION_DATA] 自定义变量
- page 页面的配置信息
  - page.content 页面内容
  - page.title 标题
  - page.excerpt 摘要
  - page.url 链接
  - page.date 时间
  - page.id 唯一标示
  - page.categories 分类
  - page.tags 标签
  - page.path 源代码位置
  - page.next 下一篇文章
  - page.previous 上一篇文章
- content 模板中，用于引入子节点的内容
- paginator 分页信息
  - paginator.per_page 每一页的数量
  - paginator.posts 这一页的文章
  - paginator.total_posts 所有文章的数量
  - paginator.total_pages 总的页数
  - paginator.page 当前页数
  - paginator.previous_page 上一页的页数
  - paginator.previous_page_path 上一页的路径
  - paginator.next_page 下一页的页数
  - paginator.next_page_path 下一页的路径

##### 字符转义

有时候想输出 \{ 了,怎么办,使用 \\ 转义即可。`\{ => }`

##### 输出变量

直接使用两个大括号来输出变量：`{ { page.title } }` `{ { content } }`

##### 循环

和解释性语言很像(以下代码摘自 index.html)：

```
<!-- 文章摘要显示 -->
{% for post in paginator.posts %}
<div class="post-preview">
    <a href="{{ post.url | prepend: site.baseurl }}">
        <h2 class="post-title">
            {{ post.title }}
        </h2>
        {% if post.subtitle %}
        <h3 class="post-subtitle">
            {{ post.subtitle }}
        </h3>
        {% endif %}
        <div class="post-content-preview">
            {{ post.content | strip_html | truncate:150 }}
        </div>
    </a>
    <p class="post-meta">Posted by {% if post.author %}{{ post.author }}{% else %}{{ site.title }}{% endif %} on {{ post.date | date: "%B %-d, %Y" }}</p>
</div>

<hr>
{% endfor %}
```

##### 删除指定文本

remove可以删除变量中的指定内容：

```
{{ post.url | remove: 'http' }}
```

##### 删除html标签

在摘要中很有用，例如上面代码中的

```
{{ post.content | strip_html | truncate:150 }}
```

##### 代码高亮

```
{ % highlight ruby linenos % }
\# some ruby code
{ % endhighlight % }
```

##### 数组的大小

```
{{ array | size }}
```

##### 赋值

```
{% assign index = 1 %}
```

##### 格式化时间

```
{{ site.time | date_to_xmlschema }} 2008-11-07T13:07:54-08:00
{{ site.time | date_to_rfc822 }} Mon, 07 Nov 2008 13:07:54 -0800
{{ site.time | date_to_string }} 07 Nov 2008
{{ site.time | date_to_long_string }} 07 November 2008
```

##### 搜索指定key

```
# Select all the objects in an array where the key has the given value.
{ { site.members | where:"graduation_year","2014" } }
```

##### 排序

```
{ { site.pages | sort: 'title', 'last' } }
```

##### to json

```
{ { site.data.projects | jsonify } }
```

##### 序列化

一个对象序列化为一个字符串

```
{ { page.tags | array_to_sentence_string } }
```

##### 单词的个数

```
{ { page.content | number_of_words } }
```

##### 指定个数

得到数组指定范围的结果集

```
{ { for post in site.posts limit:20 } }
```

#### 命名规则

对于博客,名字必须是 YEAR-MONTH-DAY-title.MARKUP 的格式

````
2014-11-06-memcached-code.md
2014-11-06-memcached-lib.md
2014-11-06-sphinx-config-and-use.md
2014-11-07-memcached-hash-table.md
2014-11-07-memcached-string-hash.md
````

## 参考
* [打造你的 GitHub Pages 专属博客](http://azeril.me/blog/Build-Your-First-GitHub-Pages-Blog.html)
* [Jekyll 博客主题精选](http://azeril.me/blog/Selected-Collection-of-Jekyll-Themes.html)
* [Git 教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)  
* [Jekyll 语法简单笔记](http://github.tiankonguse.com/blog/2014/11/10/jekyll-study.html)
