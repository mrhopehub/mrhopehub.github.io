---
layout: posts
title: "ubuntu翻墙"
---
# {{ page.title }}
使用VPN Gate+Openvpn
## 安装openvpn软件及openvpn界面配置组件
<xmp class="prettyprint linenums">sudo apt-get install openvpn network-manager-openvpn</xmp>
## 获取VPN配置文件
<xmp class="my_xmp_class">http://www.vpngate.net/cn上下载配置文件</xmp>
## 修改配置文件
<xmp class="my_xmp_class">假如下载好的VPN配置文件是vpngate_140.206.98.102_tcp_1972.ovpn，打开该文件找到ca标签，把ca标签之间的内容剪切出来，粘贴到一个新的文本文件中，然后重命名该这个文本文件为ca.crt</xmp>
## 配置VPN
<xmp class="my_xmp_class">打开VPN配置窗口，添加VPN，选择导入VPN配置，在弹出的窗口中认证方式为密码，并且添加ca.crt</xmp>
## 连接VPN