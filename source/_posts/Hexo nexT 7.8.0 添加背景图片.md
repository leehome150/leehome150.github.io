---
title: Hexo nexT 7.8.0 添加背景图片
abbrlink: b56cb502
date: 2020-11-27 16:54:22
tags: hexo
categories: hexo
---

版本信息

```
"hexo": "^5.0.0",
"NexT": "7.8.0",
```

<strong>由于 next 版本较新，next 主题更新至 7.0+版本后取消了\_custom 文件夹以及 custom.styl 文件。</strong>

在这里提醒下<strong>不要打开</strong>themes/next/\_config.yml 配置文件搜索 custom 以下代码里的 style 的 注释（博主踩过坑）

```
custom_file_path:
  #head: source/_data/head.swig
  #header: source/_data/header.swig
  #sidebar: source/_data/sidebar.swig
  #postMeta: source/_data/post-meta.swig
  #postBodyEnd: source/_data/post-body-end.swig
  #footer: source/_data/footer.swig
  #bodyEnd: source/_data/body-end.swig
  #variable: source/_data/variables.styl
  #mixin: source/_data/mixins.styl
  # style: source/_data/styles.styl
```

首先在 themes/next/souces/images 添加你喜欢的背景图片，这里我推荐在知乎里面找推荐。

然后在 themes/next/souces 下新建\_data 文件，创建 styles.styl,添加内容

```
body {
    background:url(/images/background.jpg); // 可以是路径也可以是链接
    background-repeat: no-repeat; // 不重复
    background-attachment:fixed; // 固定住背景图片
    background-position:50% 50%; // 图片位置：居中
    background-size: 100% 100%; // 图片长宽扩充为100%
}

```

最后需要在 yourblog/themes/next/source/css 下找到 main.styl 文件，并在最后一行添上如下代码：

```
@import "../_data/styles.styl";
```

最后的最后

```
hexo clean
hexo server
```

就可以看到最终效果了！！！！
