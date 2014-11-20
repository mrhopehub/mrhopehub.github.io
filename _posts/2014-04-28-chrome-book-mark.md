---
layout: posts
title: "修改Host，让Chrome快速同步书签"
---

# {{ page.title }}
<xmp style="white-space: pre-wrap; word-wrap: break-word;">
不少Chrome用户反应，Chrome的同步功能越来越不给力了，平均要花费10多分钟来同步，有时候的有些地区的用户还直接同步不了。
我就单独计算过，在云南大理的网络，一个全新的操作系统，一个全新的稳定版Chrome，我同步Chrome一共花了18分钟。
所以，同步是非常非常麻烦的，至于原因我在这里就不多说了，在这里我们使用修改Hosts的方法来解决这个问题。
首先，进入这个地址：C:\Windows\System32\drivers\etc，找到host文件，然后用笔记本程序打开。在这笔记本的最后，加入下面这一段乱七八糟的代码，然后点击保存，即可。
203.208.46.22 talkgadget.google.com
2404:6800:8005::71 profiles.google.com
2404:6800:8005::65 plusone.google.com
2404:6800:8005::8a plus.google.com
2404:6800:8005::62 talkgadget.google.com
203.208.46.180 lh6.googleusercontent.com
203.208.46.180 lh5.googleusercontent.com
203.208.46.180 lh3.googleusercontent.com
203.208.46.180 lh2.googleusercontent.com
203.208.46.180 lh1.googleusercontent.com
203.208.46.180 lh4.googleusercontent.com
203.208.46.180 webcache.googleusercontent.com
203.208.46.180 mail.google.com
203.208.46.180 www.google.com.hk
203.208.46.180 www.google.com
203.208.46.180 picasaweb.google.com
203.208.46.180 www.googlelabs.com
203.208.46.180 docs.google.com
203.208.46.180 plus.google.com
203.208.46.180 plus.google.com.hk
203.208.46.180 profiles.google.com
203.208.46.180 services.google.com
203.208.46.180 clients4.google.com
203.208.46.180 clients2.google.com
203.208.46.180 chrome.google.com
203.208.46.180 tools.google.com
当然，做到这个步骤后，你再用Chrome同步试试，是不是比以前快了很多？
唯一需要注意的是，在“数字安全卫士”检测的时候，会提示你“域名解释发现异常，会让您访问假冒的网站”，你只要在设置中，“加入信任”即可。以前介绍过相类似的文章，不过，貌似那不能用了。
请各位Chrome迷低调宣传，本文只做技术交流之用。目的只有一个，为了更好的让Chrome迷们使用Chrome。
</xmp>