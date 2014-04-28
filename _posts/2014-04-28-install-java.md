---
layout: posts
title: "ubuntu下JDK安装方法总结"
---

# ubuntu下JDK安装方法总结
<xmp style="white-space: pre-wrap; word-wrap: break-word;">
.bin安装方法
1)首先，在Oracle的官网上下载JDK。
http://www.oracle.com/technetwork/java/javase/downloads/jdk-6u31-download-1501634.html
这里下载的是jdk-6u31-linux-i586.bin。

2)进入jdk-6u31-linux-i586.bin所在的目录,更改文件权限
sudo chmod u+x jdk-6u31-linux-i586.bin

3）安装JDK
sudo -s ./jdk-6u31-linux-i586.bin
接下来一直按回车，输入yes即可。

4)配置环境变量
执行命令：sudo gedit /etc/profile
打开profile文件，在profile文件的最下面输入如下的内容并保存。

#set java environment 
export JAVA_HOME=/usr/jdk1.6.0_31
export JRE_HOME=/usr/jdk1.6.0_31/jre
export CLASSPATH=".:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH"
export PATH="$JAVA_HOME/bin:$JRE_HOME/bin:$PATH"

5)source /etc/profile

6)3.2 设置java和javac
sudo update-alternatives --install /usr/bin/java java /home/gg/jdk1.6.0_33/bin/java 300  
sudo update-alternatives --install /usr/bin/javac javac /home/gg/jdk1.6.0_33/bin/javac 300  
通过 这一步将我们安装的JDK加入java选单
7)配置默认的jdk
执行代码：sudo update-alternatives --config java
根据提示设置系统默认的JDK。
8)验证默认java版本 java -version

PS：6、7两步在只有一个java版本的时候不需要

.tar.gz安装方法
如果你现在去Oracle官网去看一下的话，会发现都变成.tar.gz的压缩文件了

1. 首先你需要到oracle官网下载最新版本的JDK。跑到oracle官网，自己到Download下面找找吧
随便给个网址：http://www.oracle.com/technetwork/java/javase/downloads/jdk7u7-downloads-1836413.html

2.转到下载路径，对下载后的文件解压缩，比如我下载的文件名为jdk-7u7-linux-i586.tar.gz
cd xxx(你的下载路径)
sudo tar zxvf jdk-7u7-linux-i586.tar.gz

3.要将解压缩出来的文件夹拷贝到/usr/lib/jdk中，假设我解压出来的文件夹为jdk1.7.0_07
sudo cp -r jdk1.7.0_07 /usr/lib/jdk
注意：如果/usr/lib/jdk不存在，就自己手动建一个，名字叫jdk或者jvm啥的都可以
sudo mkdir /usr/lib/jdk

4.修改环境变量，或者用gedit随你
gedit ~/.bashrc或者sudo gedit /etc/environment

最下面添加下面几行，注意红色字部分要根据你下载解压得到的东西修改。
export JAVA_HOME=/usr/lib/jdk1.7.0_07
export JRE_HOME=${JAVA_HOME}/jre   
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib   
export PATH=${JAVA_HOME}/bin:$PATH   
保存退出，输入:
source ~/.bashrc或者source /etc/.environment（要与前面对应）

6.可以通过输入
java -version
查看版本号
当系统中如果还安装有其他的jdk版本还要修改默认版本

JDK还有其他安装方法，如PPA
在ubuntu系统中，和OpenJDK比起来，如果你更偏爱Oracle JDK（前 Sun JDK），我推荐一种很简便的方法给你。通过一个PPA仓库，你可以很容易的进行安装Oracle JDK（包括JRE）并始终保持最新的版本。
Oracle JDK7本身并不存在于该PPA中，这是因为新的Java授权许可证并不允许这么做（这也是Oracle JDK7从ubuntu官方软件仓库中移除的原因）。PPA中的软件程序自动从Oracle官方网站下载Oracle Java JDK7并把它安装到你的电脑中，就像flashplugin-installer软件包那样。
需要注意的是，该软件包当前还是alpha版本，可能在有些情况下不能正常工作！该软件包支持代理，但是如果你的ISP或者路由器封禁了一些非标准端口，这可能导致安装失败，这是由于Oracle在Java7二进制安装包的下载链接中使用了许多重定向！如果因此而导致下载失败，亦或你的电脑在防火墙保护之下，你就需要手动安装 Oracle Java 7了。

安装Oracle Java 7
该软件包提供安装Oracle Java JDK 7 (包括 Java JDK, JRE 和 the Java 浏览器插件)，如果你只需要安装Oracle JRE，请不要使用该PPA。
运行下述命令，即可完成添加PPA、安装最新版本的Oracle Java 7（支持Ubuntu 12.04, 11.10, 11.04 and 10.04）:
 
1
2
3    sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java7-installer	 
安装完成之后，如果你想看看是否真的安装成功了，你只需运行下面的命令：
 
1	java -version	 
命令输出应该包含和下面类似的内容：
java version "1.7.0_04" Java(TM) SE Runtime Environment (build 1.7.0_04-b20) Java HotSpot(TM) Server VM (build 23.0-b21, mixed mode)
注：版本号中的”_04″部分可能会与你的不同，这是由于该PPA总是安装最新的Oracle Java 7版本。
如果由于一些其它的原因，当前的Java版本不是1.7.0，你可以尝试运行下面的命令：
 
1	sudo update-java-alternatives -s java-7-oracle	 
卸载 Oracle Java 7
如果你不再想使用Oracle Java (JDK) 7，想回归OpenJDK了，你只需卸载Oracle JDK7 Installer，这样OpenJDK就又变成当前使用的java了：
 
1	sudo apt-get remove oracle-java7-installer	 
</xmp>