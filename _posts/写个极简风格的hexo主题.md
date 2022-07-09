---
title: 写个极简风格的hexo主题
description: hexo 的主题有很多，选择困难症，决定写个极简风格的主题，了解一下其实现过程。
date: 2022-07-09 00:00
---

hexo 的主题有很多，选择困难症，决定写个极简风格的主题，满足自己对笔记博客的诉求、了解一下其实现过程。
完成了以下个人认为笔记博客必须的内容，整体完成下来对前端来说还是比较简单的。

- [x] **首页** 展示/文章列表分页/回到顶部按钮
- [x] **文章页** 展示/阅读大纲(TOC)/回到顶部按钮
- [x] **关于我页** 展示
- [x] **其他** 文章搜索/样式处理(移动端兼容)

源码地址待补充：xxx

> 参考文章：
> https://cloud.tencent.com/developer/article/1440353
> https://cloud.tencent.com/developer/article/1624646
> 注：许多样式从原文复制粘贴而来、代码块颜色取自于 One Monokai，非商用，若侵犯权益，请联系我整改。

<!-- more -->

# 一、主题实现概览

## 1.1 目录结构

在 themes 目录下，创建自己的主题目录 baozi，并在 baozi 目录下创建如下目录及文件：

- languages 目录：多语言/国际化，暂时没用到
- layout 目录：用来放页面模板，全是 ejs，两种如下情况
  - layout/_partial 目录下：可复用的局部模板，如 head(meta)、header(导航)、footer 等，
  - layout 目录下：layout.ejs 表明每个页面的结构(需要组合哪些ejs，如：head+header+index+footer 组成首页)，其他 ejs 用来表示对应页面主体(如：index.ejs-首页、post.ejs-文章页)
- scripts 目录
- source 目录，顾名思义，一些静态资源依赖，会被打包到全局。各项内容如下：
  - css 目录，存放 .styl 样式文件，两种情况：
    - css/_partial 下：各部分 ejs 对应的 css 样式
    - css 下：style.styl 引入 css/_partial 下的各个样式文件，作为页面 css 加载的最终内容，在 head.ejs 中通过 <%- css('css/style') %> 关联(引入)
  - js 目录，存放一些需要加载的 js，如：加载 jquery - <%- js('js/jquery.js') %>
- _config.yml 文件，这个都很熟悉：主题的配置文件，在这之中的各种配置，ejs 中都可以通过 theme.xxx 来获取到，从而起到配置化渲染的效果

## 1.2 其他的知识

- css 不仅可以使用 styl ，也可以使用其他预处理 css 格式(安装对应插件，前端应该很熟悉)，但 hexo 预装了 styl 相关的插件，建议使用 styl
- 项目的跟配置文件 _config.yml，可以使用 config.xxx 来获取对应的属性
- 主题的配置文件 theme/_config.yml，可以使用 theme.xxx 来获取对应属性
- hexo 内置了一些辅助函数及变量，如：
  - partial()，可以引入其他 ejs 模板文件，如：<%- partial('_partial/head') %>
  - css()，可以引入 css 文件，如：<%- css('css/style.styl') %>。注意，实操中发现这里引入需要去掉 .styl 后缀
  - url_for()，构建跳转链接，如：`<a href="<%- url_for(post.path) %>"></a>`
  - paginator()，插入分页链接，首页文章列表有用到
  - toc()，根据文章内容生成大纲/目录
  - page 变量，比较特殊，在不同的页面/模板是不同的内容，如：
    - 首页 index.ejs 通过 page.posts 遍历获得每个文章的数据
    - 文章详情页 post.ejs 获取文章数据，如：page.title 名称，page.date 日期，page.content 内容(markdown 顶部定义的属性，这里都可以获取到)

# 二、创建首页

- 首页文章列表渲染：page.posts.each
- 首页添加分页器：partial('_partial/paginator')

```html
<!-- layout/index.ejs -->
<section class="posts">
  <% page.posts.each(function (post) { %>
  <article class="post">
    <div class="post-title">
      <a class="post-title-link" href="<%- url_for(post.path) %>"><%= post.title %></a>
    </div>
    <div class="post-content"><%- post.excerpt %></div>
    <div class="post-meta">
      <span class="post-time"><%- date(post.date, "YYYY-MM-DD") %></span>
    </div>
  </article>
  <% }) %>
</section>
<%- partial('_partial/paginator') %>

<!-- layout/_partial/paginator.ejs -->
<% if (page.total > 1){ %>
<nav class="page-nav"><%- paginator({ prev_text: "&laquo; Prev", next_text: "Next &raquo;" }) %></nav>
<% } %>
```

# 三、创建文章页

```html
<!-- layout/post.ejs -->
<article class="post">
  <div class="post-title">
    <h2 class="title"><%= page.title %></h2>
  </div>
  <div class="post-meta">
    <span class="post-time"><%- date(page.date, "YYYY-MM-DD") %></span>
  </div>
  <div class="post-content"><%- page.content %></div>
</article>

```

# 四、样式引入

通过 <%- css('css/style') %> 引入 source/css 下的 style.styl 样式文件，该文件中，引入了其他需要的样式文件

```html
<!-- layout/_partial/head.ejs -->
<head>
  <meta http-equiv="content-type" content="text/html; charset=utf-8" />
  <meta content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" name="viewport" />
  <title><%= config.title %></title>
  <%- css('css/style') %>
</head>
```

# 五、创建自定义的页面-关于我

1. 执行hexo new page about进行手动生成页面 => 在项目根目录 source 下自动创建了 about/index.md (同理，自己创建对应目录和 md 文件也一样)
2. 添加需要的内容
3. 然后就可以通过 /about 来访问关于我页面了

# 六、功能实现

## 6.1 搜索

1. 设置项目 _config.yml - search - path 改为 json 格式，即 search.json
2. 在 source/js 下放入 jquery.js，head.ejs 使用 `<%- js('js/jquery.js') %>` 引入 JQuery，用来实现 scripts/search.js，从 json 中检索数据
3. source/js/search.js 中，我们通过 searchFunc 函数实现 json 搜索，同样需要引入
4. 创建 search.ejs 及其样式，包含相应的输入框，和结果展示，并调用 search.js/searchFunc 进行搜索
5. searchFunc 会注册 input 时间，并将结果渲染到制定的 dom 中。我们通过合理的添加一些样式文件来实现弹窗形式的搜索组件(不再赘述)

```js
// source/js/search.js
var searchFunc = function (path, search_id, content_id) {
  // 'use strict';
  $.ajax({
    url: path,
    dataType: 'json',
    success: function (datas) {
      console.log(datas);
      var $input = document.getElementById(search_id);
      var $resultContent = document.getElementById(content_id);
      $input.addEventListener('input', function () {
        var str = '<ul class="search-result-list">';
        var keywords = this.value
          .trim()
          .toLowerCase()
          .split(/[\s\-]+/);
        $resultContent.innerHTML = '';
        if (this.value.trim().length <= 0) {
          return;
        }
        // perform local searching
        datas.forEach(function (data) {
          var isMatch = true;
          var content_index = [];
          var data_title = data.title.trim().toLowerCase();
          var data_content = data.content
            .trim()
            .replace(/<[^>]+>/g, '')
            .toLowerCase();
          var data_url = data.url;
          var index_title = -1;
          var index_content = -1;
          var first_occur = -1;
          // only match artiles with not empty titles and contents
          if (data_title != '' && data_content != '') {
            keywords.forEach(function (keyword, i) {
              index_title = data_title.indexOf(keyword);
              index_content = data_content.indexOf(keyword);
              if (index_title < 0 && index_content < 0) {
                isMatch = false;
              } else {
                if (index_content < 0) {
                  index_content = 0;
                }
                if (i == 0) {
                  first_occur = index_content;
                }
              }
            });
          }
          // show search results
          if (isMatch) {
            str += "<li><a href='" + data_url + "' class='search-result-title'>" + data_title + '</a>';
            var content = data.content.trim().replace(/<[^>]+>/g, '');
            if (first_occur >= 0) {
              // cut out 100 characters
              var start = first_occur - 20;
              var end = first_occur + 80;
              if (start < 0) {
                start = 0;
              }
              if (start == 0) {
                end = 100;
              }
              if (end > content.length) {
                end = content.length;
              }
              var match_content = content.substr(start, end);
              // highlight all keywords
              keywords.forEach(function (keyword) {
                var regS = new RegExp(keyword, 'gi');
                match_content = match_content.replace(regS, '<em class="search-keyword">' + keyword + '</em>');
              });

              str += '<p class="search-result">' + match_content + '...</p>';
            }
            str += '</li>';
          }
        });
        str += '</ul>';
        $resultContent.innerHTML = str;
      });
    },
  });
};
```

```html
<!-- layout/_partial/search.ejs -->
<button id="btnSearch" class="search-btn">搜索</button>
<div id="searchContainer" class="search-container">
  <div id="searchContent" class="search-content">
    <div class="search-input">
      <input id="searchInput" type="text" placeholder="查找文章" />
    </div>
    <div id="searchResult" class="search-result"></div>
  </div>
</div>
<script>
  // 点击搜索打开搜索弹窗
  btnSearch.addEventListener('click', function () {
    $('.search-container').addClass('show-search');
  });
  // 阻止冒泡，防止点击弹窗内容触发弹窗关闭
  searchContent.addEventListener('click', function (e) {
    e.stopPropagation();
  });
  // 关闭搜索弹窗
  searchContainer.addEventListener('click', function () {
    $('#searchInput').val('');
    $('#searchResult').empty();
    $('.search-container').removeClass('show-search');
  });
  // 注册搜索函数
  searchFunc('search.json', 'searchInput', 'searchResult');
</script>

```

## 6.2 回到顶部

```html
<a id="goTop" class="go-top" type="button">回到顶部</a>

<script>
  goTop.addEventListener('click', function (e) {
    scrollTo(0, 0);
  });
</script>
```

## 6.3 阅读大纲 TOC

```html
<div class="post-toc"><%- toc(page.content, { list_number: false }) %></div>
```
