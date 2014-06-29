---
layout: posts
title: "一次不愉快的GITHUB经历"
---

### 一次不愉快的GITHUB经历
重装了系统，一个多月没有使用github了，由于上次用的是免安装的git版本，所以重新建立github系统不是问题，可是当中遇到了一个莫名其妙的问题，所以记录一下如何使用git、github。
#### <font color="blue">安装git</font>
1. 下载PortableGit-1.9.0-preview20140217.7z
2. 解压到C盘根目录
3. 添加环境变量
<blockquote>
1. 新建系统变量env_git=C:\PortableGit-1.9.0-preview20140217<br>
<font color="red">使用变量名：GIT_DIR将导致所有的git命令的目录在C:\PortableGit-1.9.0-preview20140217，一直提示“fatal: Not a git repository: 'C:\PortableGit-1.9.0-preview20140217'”</font><br>
2. path中添加%env_git%\cmd
</blockquote>

#### <font color="blue">配置git</font>
<blockquote>
1. 产生密钥并测试<br>
打开C:\PortableGit-1.9.0-preview20140217目录下git-bash.bat，
<xmp class="prettyprint linenums">
ssh-keygen -t rsa -C "your_email@youremail.com"
</xmp>
-t是type选项多缩写，-C是comment的缩写，后面的your_email@youremail.com改为你的邮箱，之后会要求确认路径和输入密码，我们这使用默认的一路回车就行。成功的话会在~/下生成.ssh文件夹，进去，打开id_rsa.pub，复制里面的key。回到github，进入Account Settings，左边选择SSH Keys，Add SSH Key,title随便填，粘贴key。为了验证是否成功，在git bash下输入：
<xmp class="prettyprint linenums">
ssh -T git@github.com
</xmp>
如果是第一次的会提示是否continue，输入yes就会看到：You’ve successfully authenticated, but GitHub does not provide shell access 。这就表示已成功连上github。<br>
<font color="red">注意是打开git-bash.bat，而不是windows的运行窗口（git ssh-keygen会提示git没有这个选项），</font>
2. 设置用户名、邮箱<br>
<xmp class="prettyprint linenums">
git config --global user.name "your name"
git config --global user.email "your_email@youremail.com"
</xmp>
3. 添加远程仓库
<xmp class="prettyprint linenums">
git remote add origin git@github.com:yourName/yourRepo.git
</xmp>
4. 提交、推送
<xmp class="prettyprint linenums">
git add ./*
git commit -a -m 'comment'
git push origin master
</xmp>
</blockequtoe>