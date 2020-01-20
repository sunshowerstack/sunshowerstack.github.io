---
title: GitHub Pages博客搭建
date: 2020-01-17 13:30:09
categories:
- other
tags:
- github
- jekyll
- jekyll-theme-next
---


# GitHub Pages博客搭建



## 1. fork仓库

访问https://github.com/Simpleyyt/jekyll-theme-next，点击左上角的**Fork**，再点击**Settings**，重命名这个仓库：

新名字就按照你的用户名.github.io，例如   sample.github.io。



## 2.clone到本地做修改

将这个仓库clone到本地仓库，项目结构如下图所示：



其中，**about.html**是关于的界面，**index.html**是首页，**tags.html**是标签集合页面。用记事本打开，可以修改里面的title和description改成你自己的~

**favicon.ico**和**img文件夹**里的图片是各种背景图和头像图，也可以做相应的修改~

最后，改一下**_config.yml**里的配置文件里的内容，基本都有中文注释的~**注意一定要把百度统计和Google分析改成自己的，如果不用可以直接注释（在前面加个#）！！**



在**_posts文件夹**里的就是你要发布的博客了，

Markdown格式的文件。你可以将我的全部删除改成自己的~文件命令格式2018-01-25-build-home-page-with-wordpress.markdown，前面是日期，后面是名字。里面内容开头格式如下：


后面再跟着Markdown格式的内容就可以了~



## 3. 本地安装 NexT

比较麻烦，需要用ruby， bundle install 安装还会报错。要求不高建议在线修改实验。



确保已安装`Ruby 2.1.0` 或更高版本：

```
$ ruby --version
```

安装Bundler：

```
$ gem install bundler
```



下载 NexT 主题：

```
$ git clone https://github.com/Simpleyyt/jekyll-theme-next.git
$ cd jekyll-theme-next
```


安装依赖：

```
$ bundle install
```


运行 Jekyll：

```
$ bundle exec jekyll server
```


此时即可使用浏览器访问 http://localhost:4000，检查站点是否正确运行。







# 可能遇到的问题

## 1. 错误 bundle install报错

搭建本地测试环境。

~~~
$ bundle install
~~~

> 错误信息
>
> /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk/System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/include/ruby-2.3.0/universal-darwin16/ruby/config.h,
> needed by `arena.o'.  Stop.
>
> make failed, exit code 2
>
> Gem files will remain installed in /var/folders/bk/z8r5nhg551s7b5glz7f4rht40000gn/T/bundler20200117-30554-1bldq3ecommonmarker-0.17.13/gems/commonmarker-0.17.13 for inspection.
> Results logged to /var/folders/bk/z8r5nhg551s7b5glz7f4rht40000gn/T/bundler20200117-30554-1bldq3ecommonmarker-0.17.13/extensions/universal-darwin-16/2.3.0/commonmarker-0.17.13/gem_make.out
>
> An error occurred while installing commonmarker (0.17.13), and Bundler cannot continue.
> Make sure that `gem install commonmarker -v '0.17.13' --source 'https://rubygems.org/'` succeeds before bundling.
> In Gemfile:
>   github-pages was resolved to 193, which depends on
>     jekyll-commonmark-ghpages was resolved to 0.1.5, which depends on
>       jekyll-commonmark was resolved to 1.2.0, which depends on
>         commonmarker





尝试通过brew安装cmake（没用）

~~~shell
brew install cmake
~~~



## 2. 显示摘要。

加一行配置。有用

~~~
excerpt_separator: 。
~~~



尝试enable改为true，修改无用

~~~
auto_excerpt:
  enable: true
  length: 150
~~~



## 3. 绑定域名



### 1. 域名商增加2条CNAME记录

方法一：点击添加记录，需要添加两个记录，两个记录类型都是 CNAME ，第一个主机记录为 @ ，第二个主机记录为 www，记录值都是填你自己的博客地址（比如我的是：tst.github.io），保存之后域名解析就完成了！



### 2. github设定domain

然后打开你的github博客项目，点击settings，拉到下面Custom domain处，填上你自己的域名，保存：



## 3. https



今年 5 月 1 号，GitHub Pages 支持**自定义域名 Enforce HTTPS** 了，并且有原生的 CDN 支持

看了证书也是letsencrypt发行的，默认有效期3个月。待观察github会不会自动更新。



没必要再通过cloudflare来支持https了



步骤很简单

1.在_config.yml里加入带https的domain

~~~
url: https://sunshowerstack.xyz
~~~



2.在github的settings里

**Custom domain**栏里指定sunshowerstack.xyz后save

刚开始底下的*enforce https*是灰掉无法选的。等几分钟，刷新页面后就可以勾选了。

勾选后一会https就会生效





参考

https://io-oi.me/tech/custom-domains-on-github-pages/