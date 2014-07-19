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
</xmp>
2. 绘制分割线,修改WM_PAINT消息处理
<xmp class="prettyprint linenums">
    	MoveToEx(hdc, 10, 78, NULL);
		LineTo(hdc, 115, 78);
</xmp>
如图<br>
![WM_PAINT](/images/windows and oo/wm_paint.jpg)<br>

### <font color="blue">添加相应函数((“Your Project Name”).cpp)</font>

>1. 添加菜单处理函数
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
2. 添加运算按钮处理函数,不要忘了case下的第一层大括号
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