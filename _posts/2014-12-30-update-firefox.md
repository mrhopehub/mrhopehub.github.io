---
layout: posts
title: "ubuntu10.04升级Firefox"
---
# {{ page.title }}
<xmp class="my_xmp_class">add-apt-repository命令用于添加软件源到/etc/apt/sources.list或者/etc/apt/sources.list.d/
第一种形式：REPOSITORY会被追加到/etc/apt/sources.list
第二种形式： ppa:<user>/<ppa-name>会被追加到/etc/apt/sources.list.d/</xmp>
<xmp style="color: red; font-size: 14px;" class="my_xmp_class">这就是为什么的文章在更新源是重视gedit /etc/apt/sources.list，而更新ppa时总是用命令add-apt-repository</xmp>
<xmp class="my_xmp_class">    首先，说明一下，PPA就是指Personal Package Archives（个人软件包档案）。有很多软件因为种种原因，不能进入官方的 Ubuntu 软件仓库。为了方便 Ubuntu 用户使用，Ubuntu Launchpad网站（https://launchpad.net/）提供了源服务PPA，允许个人用户上传软件源代码，通过Launchpad进行编译并发布为二进制软件包，作为apt/新立得源，供其他用户下载和更新。PPA 也被用来对一些打算进入 Ubuntu 官方仓库的软件，或者某些软件的新版本进行测试。PPA 上面的软件相当丰富，如果在Ubuntu的官方仓库中找不到需要的软件，可以去 PPA 上找找看。在Ubuntu中添加了PPA源之后，软件可以直接在软件中心进行安装并会自动提示升级，这就是Ubuntu带来的方便，现在我们就来看看如何添加PPA软件源。</xmp>
## 查找合适的ppa源
[https://launchpad.net/ubuntu/+ppas](https://launchpad.net/ubuntu/+ppas)搜索合适的源，比如第一个查找结果Firefox Beta - Lucid，点击链接进去，找到Adding this PPA to your system下的源ppa:silverwave/testing6
## 添加并更新源
<xmp class="prettyprint linenums">sudo add-apt-repository ppa:silverwave/testing6
sudo apt-get update</xmp>
## 安装(升级)Firefox
<xmp class="prettyprint linenums">sudo apt-get install firefox</xmp>