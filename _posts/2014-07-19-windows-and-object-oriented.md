---
layout: posts
title: "windows窗口与面向对象"
---

# 译(Making a simple application using the Win32 API)

### <font color="blue">按Wizard默认生成一个标准的win32程序</font>
### <font color="blue">修改资源文件、窗口外观</font>

1. Resource.h添加
<xmp class="prettyprint linenums">
#define IDM_ADD        		110
#define IDM_SUB				111
#define IDM_MULT			112
#define IDM_REM				113
#define IDC_RHS				114
#define IDC_LHS				115
#define IDC_RESULT			116
#define IDC_OPBUTTON		117
</xmp>
如图：<br>
![ID](/images/windows and oo/id.jpg)<br>
2. 右键(“Your Project Name”).rc-->查看代码,IDC_CALC MENU下添加
<xmp class="prettyprint linenums">
	POPUP "计算(&O)"
	BEGIN
	MENUITEM "Add"							IDM_ADD
	MENUITEM "Subtract"						IDM_SUB
	MENUITEM "Multiply"						IDM_MULT
	MENUITEM "Remainder"					IDM_REM
	END
</xmp>
如图：<br>
![MENU](/images/windows and oo/menu.jpg)<br>
3. (“Your Project Name”).cpp中<br><font color="blue">MyRegisterClass()修改窗口背景颜色</font>
<xmp class="prettyprint linenums">
wcex.hbrBackground = CreateSolidBrush(RGB(180, 180, 180));
</xmp>
<font color="blue">MyRegisterClass()修改窗口大小、样式</font>
<xmp class="prettyprint linenums">
    hWnd = CreateWindow(szWindowClass, szTitle, WS_OVERLAPPED | WS_CAPTION | WS_SYSMENU | WS_MINIMIZEBOX,
		CW_USEDEFAULT, 0, 200, 200, NULL, NULL, hInstance, NULL);
</xmp>

### <font color="blue">向窗口中添加控件((“Your Project Name”).cpp)</font>

1. 添加WM_CREATE消息处理
<xmp class="prettyprint linenums">
    case WM_CREATE:
		CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("EDIT"), TEXT("0"),
			WS_CHILD|WS_VISIBLE|ES_AUTOHSCROLL|WS_TABSTOP,
			20, 10, 85, 25, hWnd, (HMENU)IDC_LHS, GetModuleHandle(NULL), NULL);
		CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("EDIT"), TEXT("0"),
			WS_CHILD|WS_VISIBLE|ES_AUTOHSCROLL|WS_TABSTOP,
			20, 45, 85, 25, hWnd, (HMENU)IDC_RHS, GetModuleHandle(NULL), NULL);
		CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("EDIT"), TEXT("0"),
			WS_CHILD|WS_VISIBLE|ES_AUTOHSCROLL|WS_TABSTOP,
			20, 85, 85, 25, hWnd, (HMENU)IDC_RESULT, GetModuleHandle(NULL), NULL);
		CreateWindowEx(NULL, TEXT("BUTTON"), TEXT("+"),
			WS_CHILD|WS_VISIBLE|WS_CHILD|BS_DEFPUSHBUTTON|WS_TABSTOP,
			115, 45, 85, 25, hWnd, (HMENU)IDC_OPBUTTON, GetModuleHandle(NULL), NULL);
		break;
</xmp>
2. 绘制分割线,修改WM_PAINT消息处理
<xmp class="prettyprint linenums">
    	MoveToEx(hdc, 10, 78, NULL);
		LineTo(hdc, 115, 78);
</xmp>
如图<br>
![WM_PAINT](/images/windows and oo/wm_paint.jpg)<br>

### <font color="blue">添加相应函数((“Your Project Name”).cpp)</font>

1. 添加全局变量
<xmp class="prettyprint linenums">
enum operation {ADD,SUB,MULT,REM};
operation op = ADD;
</xmp>
如图<br>
![op](/images/windows and oo/op.jpg)<br>
2. 添加菜单处理函数
<xmp class="prettyprint linenums">
    	case IDM_ADD:
			SetDlgItemText(hWnd, IDC_OPBUTTON, TEXT("+"));
			op = ADD;
			break;
		case IDM_SUB:
			SetDlgItemText(hWnd, IDC_OPBUTTON, TEXT("-"));
			op = SUB;
			break;
		case IDM_MULT:
			SetDlgItemText(hWnd, IDC_OPBUTTON, TEXT("*"));
			op = MULT;
			break;
		case IDM_REM:
			SetDlgItemText(hWnd, IDC_OPBUTTON, TEXT("%"));
			op = REM;
			break;
</xmp>
如图:<br>
![wm_menu](/images/windows and oo/wm_menu.jpg)<br>
3. 添加运算按钮处理函数,不要忘了case下的第一层大括号
<xmp class="prettyprint linenums">
    	case IDC_OPBUTTON:
			{
				BOOL success = false;
				int lhs = GetDlgItemInt(hWnd, IDC_LHS, &success, true);
				if(!success){
					MessageBox(hWnd, TEXT("the first expression is not an integer。"), TEXT("Error"), MB_OK);
					break;
				}
				int rhs = ::GetDlgItemInt(hWnd, IDC_RHS, &success, true);
				if(!success){
					MessageBox(hWnd, TEXT("the second expression is not an integer。"), TEXT("Error"), MB_OK);
					break;
				}
				int result = 0;
				switch(op){
				case ADD:
					result = lhs + rhs;
					break;
				case SUB:
					result = lhs - rhs;
					break;
				case MULT:
					result = lhs * rhs;
					break;
				case REM:
					if(rhs == 0){
						SetDlgItemText(hWnd, IDC_RESULT, TEXT("Underfined"));
					}else {
						result = lhs % rhs;
					}
				}
				SetDlgItemInt(hWnd, IDC_RESULT, result, true);
				break;
			}
</xmp>
如图:<br>
![wm_button](/images/windows and oo/wm_button.jpg)<br>

### <font color="blue">生成工程</font>
### <font color="blue">小结</font>

1. <font color="red">面向对象的角度</font><br>
windows平台的窗口虽然已经有了面向对象的思想，而且把窗口中的空间也看作是特殊的窗口，从上面的程序也可以看出，在访问子控件的内容时（GetDlgItemInt(hWnd, IDC_LHS, &success, true);），并不像java swing那样的纯面向对象（JTextField tf;int n = Integer.parseInt(tf.getText());）。从这个角度看，windows的窗口并不是面向对象，而且不像传统的对象（含有传统的数值成员变量）和swing中的对象（像对象一样访问其内容tf.getText()）
2. <font color="red">编程最基本的问题之数据访问</font><br>
最开始在接受了把窗口看做对象的思想时（特别是MFC之socket编程总结：面向对象文章中，窗口对象中添加了，socket对象），就把窗口中的控件看做对象，在访问时总是面向对象的思路（swing的思想），上面也可以看出来，实际上窗口并没有显示的子控件对象，所以这种思路也不行。<font color="blue">在GUI环境下，访问数据不再只有对象、变量的形式，还有有API决定方式，如上面的使用GetDlgItemInt函数和窗口句柄</font>
3. <font color="red">上面两个角度总结一下</font><br>
拿访问编辑框的内容为例

>1. 通过子窗口句柄，Int GetWindowText（HWND hWnd，LPTSTR lpString，Int nMaxCount），需要在创建子控件的时候保存窗口句柄
2. 通过资源ID，UINT GetDlgItemInt(int nID, BOOL\* lpTrans = NULL, BOOL bSigned = TRUE）
3. 1、2的结合，即创建控件时不保存句柄，而是通过资源ID来获取子控件窗口句柄，HWND GetDlgItem(HWNDhDlg, intnIDDlgItem),网上例子：Win32 Examples: Net Price Calculation。
4. MFC中的窗口，只有顶层的窗口可以添加传统的数值型的成员变量，其子控件的获取也可以通过1、2、3方法得到，当时也可以通过DDX机制实现，即父窗口的数值成员变量与子控件的数据交换，但子控件还是不像java中的纯对象控件。
5. 多窗口应用程序中顶层窗口可以有数值型的成员变量，而且可以通过对象访问。