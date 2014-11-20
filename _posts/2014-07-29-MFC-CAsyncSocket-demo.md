---
layout: posts
title: "MFC CAsyncSocket demo(客户端)"
---

# MFC CAsyncSocket demo(客户端)
手把手式，学习MFC创建类的过程、CAsyncSocket、面向对象思想。
### <font color="blue">创建支持Windows套接字的MFC工程</font>

1. 创建MFC工程，工程名：MFC CAsyncSocket demo<br>
![mfc](/images/socket demo/wizard mfc.jpg)<br>
2. 基于对话框<br>
![dialog](/images/socket demo/wizard dialog.jpg)<br>
3. 添加windows套接字支持<br>
![socket](/images/socket demo/wizard socket.jpg)<br>

### <font color="blue">UI界面</font>

1. 删除"TODO: 在此放置对话框控件。"这个控件
2. 添加控件、修改Caption属性、修改布局<br>
![ui](/images/socket demo/ui.jpg)<br>
3. 设置控件其他属性，主要是提示消息文本框的可换行，Want Return、Multiline两个属性设置为True

### <font color="blue">添加CAsyncSocket子类</font>

1. 切换到类试图，右击“MFC CAsyncSocket demo”-->添加类-->MFC 类,选择基类CAsyncSocket、填写类名MySocket<br>
![class view](/images/socket demo/class view.jpg)<br>
![add class](/images/socket demo/add class.jpg)<br>
2. 添加重载函数,还是类视图，右击“MySocket”类-->类向导，分别选择OnAccept、OnConnect、OnClose、OnReceive、OnSend、OnOutOfBandData，添加到已重写的虚函数。<font color="red">这一步虽然添加了六个函数，但是不是所有的函数都要做实质性的重载。</font><br>
![class wizard](/images/socket demo/class wizard.jpg)<br>

### <font color="blue">修改类</font>
<font color="red">添加了MySocket类之后，整个工程的类框架搭建完毕，下一步要做的就是修改这些类，满足工程需要。</font><br>

1. 修改CMFCCAsyncSocketdemoDlg类定义（重点在h文件的修改,对应的cpp文件并没有太多的修改）

>* 添加控件变量
<xmp class="prettyprint linenums">
private:
    CIPAddressCtrl m_ipcontrol;
	CEdit m_portcontrol;
	CButton m_btn_con;
	CButton m_btn_discon;
	CEdit m_messagecontrol;
</xmp>
>注意是私有、Control类型变量<br>
>![dialog变量](/images/socket demo/dialog变量.jpg)<br>

>*  添加普通变量<br>
<xmp class="prettyprint linenums">
    MySocket m_socket;
	CString	m_recieveddata;
	CString m_address;
	int m_port;
</xmp>
>不要忘记包含MySocket.h，另外注意是私有变量。<br>

>* 添加处理socket的public方法
<xmp class="prettyprint linenums">
    void OnReceive();
	void OnClose();
	void OnConnect();
</xmp>

2. 修改MySocket类定义（同样重点在h文件的修改,对应的cpp文件并没有太多的修改）<br>

>* 添加private成员变量
<xmp class="prettyprint linenums">
private:
    CDialogEx *m_dialog;
</xmp>
>* 添加public成员函数
<xmp class="prettyprint linenums">
    void set_dialog(CDialogEx *dialog);
</xmp>

3. 实现MySocket类
>MySocket.cpp包含MFC CAsyncSocket demoDlg.h<br>

>* OnClose函数
<xmp class="prettyprint linenums">
void MySocket::OnClose(int nErrorCode)
{
    // TODO: 在此添加专用代码和/或调用基类
	if(nErrorCode==0)
	{
		((CMFCCAsyncSocketdemoDlg*)m_dialog)->OnClose(); 
	}	
	CAsyncSocket::OnClose(nErrorCode);
}
</xmp>
>* OnConnect函数
<xmp class="prettyprint linenums">
void MySocket::OnConnect(int nErrorCode)
{
    // TODO: 在此添加专用代码和/或调用基类
	if(nErrorCode==0)
	{
		((CMFCCAsyncSocketdemoDlg*)m_dialog)->OnConnect(); 
	}
	CAsyncSocket::OnConnect(nErrorCode);
}
</xmp>
>* OnReceive函数
<xmp class="prettyprint linenums">
void MySocket::OnReceive(int nErrorCode)
{
    // TODO: 在此添加专用代码和/或调用基类
	if(nErrorCode==0)
	{
		((CMFCCAsyncSocketdemoDlg*)m_dialog)->OnReceive(); 
	}
	CAsyncSocket::OnReceive(nErrorCode);
}
</xmp>
>* set_dialog函数
<xmp class="prettyprint linenums">
void MySocket::set_dialog(CDialogEx *dialog)
{
    this->m_dialog = dialog;
}
</xmp>

4. 实现CMFCCAsyncSocketdemoDlg类

>* OnInitDialog函数
<xmp class="prettyprint linenums">
    // TODO: 在此添加额外的初始化代码
	this->m_socket.set_dialog(this);
	//按钮相关设置
	this->m_btn_con.EnableWindow(true);
	this->m_btn_discon.EnableWindow(false);
	//ip地址与端口相关设置
	this->m_address = _T("192.168.1.150");
	this->m_ipcontrol.SetAddress(192,168,1,150);
	this->m_port = 2000;
	this->m_portcontrol.SetWindowText(_T("2000"));
	//接收数据相关设置
	this->m_recieveddata = _T("");
	this->m_messagecontrol.SetWindowText(_T(""));
	return TRUE;  // 除非将焦点设置到控件，否则返回 TRUE
</xmp>
>* OnReceive函数
<xmp class="prettyprint linenums">
void CMFCCAsyncSocketdemoDlg::OnReceive()
{
    char *pBuf = new char [1025];
	CString strData;
	int iLen;
	iLen=this->m_socket.Receive(pBuf,1024);
	if(iLen==SOCKET_ERROR)
	{
		::MessageBox(GetSafeHwnd(), _T("无法接受数据!"), _T("错误提示！"), MB_OK);
	}
	else
	{
		pBuf[iLen] = NULL;
		strData = pBuf;
		this->m_recieveddata.Insert(this->m_recieveddata.GetLength(),strData + _T("\r\n"));   //display in server
		this->m_messagecontrol.SetWindowText(this->m_recieveddata);
		delete pBuf;
	}
}
</xmp>
>* OnConnect函数
<xmp class="prettyprint linenums">
void CMFCCAsyncSocketdemoDlg::OnConnect()
{
    this->m_recieveddata.Insert(this->m_recieveddata.GetLength(),_T("连接服务器成功!\r\n"));
	this->m_messagecontrol.SetWindowText(this->m_recieveddata);
	this->m_btn_con.EnableWindow(false);
	this->m_btn_discon.EnableWindow(true);
}
</xmp>
>* OnClose函数<font color="red">实际上完全可以调用OnBnClickedButton2函数（不需要提示“服务器主动断开连接”情况下），</font>
<xmp class="prettyprint linenums">
void CMFCCAsyncSocketdemoDlg::OnClose()
{
    this->m_socket.Close();
	this->m_recieveddata.Insert(this->m_recieveddata.GetLength(),_T("服务器主动断开连接!\r\n"));
	this->m_messagecontrol.SetWindowText(this->m_recieveddata);
	this->m_btn_con.EnableWindow(true);
	this->m_btn_discon.EnableWindow(false);
}
</xmp>
>* “连接服务器”按钮响应函数,<font color="red">注意dwAddress与m_address的转换，CString与int的转换，另外m_socket.Create();是不可缺少的</font>
<xmp class="prettyprint linenums">
void CMFCCAsyncSocketdemoDlg::OnBnClickedButton1()
{
    // TODO: 在此添加控件通知处理程序代码
	DWORD dwAddress;
	this->m_ipcontrol.GetAddress(dwAddress);
	this->m_address.Format(_T("%d.%d.%d.%d"),(0xFF000000&dwAddress)>>24,(0xFF0000&dwAddress)>>16,(0xFF00&dwAddress)>>8,0xFF&dwAddress);
	CString csport;
	this->m_portcontrol.GetWindowText(csport);
	this->m_port = _ttoi(csport);	
	//重要的地方:Create()
	this->m_socket.Create();
	this->m_socket.Connect(m_address,m_port);
	this->m_btn_con.EnableWindow(false);
	this->m_recieveddata = _T("");
	this->m_recieveddata.Insert(this->m_recieveddata.GetLength(),_T("正在建立连接!!服务器IP:") + this->m_address + _T("\r\n"));
	this->m_messagecontrol.SetWindowText(this->m_recieveddata);
}
</xmp>
>* “断开服务器”按钮响应函数
<xmp class="prettyprint linenums">
void CMFCCAsyncSocketdemoDlg::OnBnClickedButton2()
{
    // TODO: 在此添加控件通知处理程序代码
	this->m_socket.Close();
	this->m_recieveddata.Insert(this->m_recieveddata.GetLength(),_T("本客户端主动断开连接!\r\n"));
	this->m_messagecontrol.SetWindowText(this->m_recieveddata);
	this->m_btn_con.EnableWindow(true);
	this->m_btn_discon.EnableWindow(false);
}
</xmp>

### <font color="blue">小结</font>

1. <font color="red">关于create函数：在调用MySocket的构造函数之后，系统中并没有实际的socket句柄，而create函数的作用就是创建实际的socket句柄并关联本MySocket对象。这个create相当于传统socket编程的socket函数、bind的结合。再说一下socket函数相当于TCP层，bind函数相当于关联IP层。</font>
2. CAsyncSocket通信过程，如本例<br>
客户端：create-->connect-->onconnect-->onreceive-->receive-->close(onclose)<br>
服务端：create-->listen-->onaccept-->accept-->send-->onsend-->onclose(close)<br>
<font color="red">注意：如果一方主动close，那么对方处理onclose，本方并没有onclose。</font>
3. 本例的“确定”、“取消”的响应函数并没有检查m_socket的连接是否close，实际上是需要处理的。
4. 并没有检测端口的有效性。
5. [github](https://github.com/mrhopehub/MFC-CAsyncSocket-demo)