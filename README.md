# LiYongTing Blog
================================

> 没想到这个博客模板会这么受欢迎。

![](http://huangxuan.me/img/blog-desktop.jpg)


[使用手册 👉](_doc/Manual.md)
--------------------------------------------------

### 快速开始

1. 你需要安装 [Ruby](https://www.ruby-lang.org/zh_cn/) 和 [Bundler](https://bundler.io/) 来使用 [Jekyll](https://jekyllrb.com/)。请参考 [Using Jekyll with Bundler](https://jekyllrb.com/tutorials/using-jekyll-with-bundler/) 完成环境配置。

2. 安装 `Gemfile` 中的依赖：

`sh
$ bundle install
`

3. 启动本地服务器（默认地址为 `localhost:4000`）：

`sh
$ bundle exec jekyll serve  # 或者使用 npm start
`

### 开发（从源码构建）

如需修改主题样式，你需要安装 [Grunt](https://gruntjs.com/)。`Gruntfile.js` 中定义了多个任务，包括压缩 JavaScript、将 `.less` 编译为 `.css`、添加 Apache 2.0 许可证声明、监听文件变化等。

这些构建流程较为老旧，没有模块化和转译等现代化配置，但功能完整。

Jekyll 相关的核心代码位于 `_includes/` 和 `_layouts/` 目录下，大多数为 [Liquid](https://github.com/Shopify/liquid/wiki) 模板。

本主题使用 Jekyll 默认的代码高亮器 [Rouge](http://rouge.jneen.net/)，与 Pygments 主题兼容，可从 [这里](http://jwarby.github.io/jekyll-pygments-themes/languages/javascript.html) 选择一个 Pygments 主题 CSS，替换 `highlight.less` 的内容即可。


### 想了解更多？查阅[完整使用手册](_doc/Manual.md)！


其他资源
---------------

移植版本
- [**Hexo**](https://github.com/Kaijun/hexo-theme-huxblog) by @kaijun
- [**React-SSR**](https://github.com/LucasIcarus/huxpro.github.io/tree/ssr) by @LucasIcarus

[起始模板 / Boilerplate](https://github.com/huxpro/huxblog-boilerplate)
- 已有些过时，欢迎贡献代码使其与主仓库保持同步。

文档翻译
- [🇨🇳  中文文档（有点过时）](https://github.com/Huxpro/huxpro.github.io/blob/master/_doc/README.zh.md)


许可证
-------

Apache License 2.0  
Copyright (c) 2015-present Huxpro

本博客主题基于 [Clean Blog Jekyll Theme（MIT 许可证）](https://github.com/BlackrockDigital/startbootstrap-clean-blog-jekyll/) 修改开发  
Copyright (c) 2013-2016 Blackrock Digital LLC.