---
layout: posts
title: "The 3 methods for enabling the A20 Gate"
---
# {{ page.title }}
[转载：http://kernelx.weebly.com/a20-address-line.html](http://kernelx.weebly.com/a20-address-line.html)<br>

<xmp class="my_xmp_class">When IBM PC AT System was introduced ,the new Intel 286 processor was not compatible with the old x86 processor.The older x86 micro-processors(Intel 8086) had address bus of 20bits which would total and give access up to 1megabyte of memory.The Intel 386 and above had address bus up to 32 bits allowing 4 Gigabytes of memory.But the old 8086 processors did not have such a big address bus.To keep in compatible with the older processors and solve the problem the Intel introduced a logical OR gate at the 20 bit of the address bus which could be enabled a or disabled.So, to keep in compatible with the older processors and programs the A20 is disabled at the startup. 

NOTE:BIOS actually enables the A20 for counting and testing the available memory and then disables it before booting again to keep in compatible with older processors.

The A20 gate is an electronic OR gate which can be disabled and enabled and placed at the 20th bit of the address bus.It is connected through a P21 line of the keyboard controller which made it possible for the keyboard controller to enable or disable the A20 Gate.

In the current generations, there is a need to have memory lot more than just 1MB.The applications, games,etc need a lot of memory.Even a operating system kernel might eat up the whole 1MB.So it is something like impossible to run modern programs in 1MB of memory.Looks like the A20 is an important feature for good functional of the operating system.

To enable the A20 gate there are 3 methods or you can skip this step by using the high memory managers such as HIMEM.sys or using bootloaders such as GRUB(GRUB will set up you up with protected mode with A20 enabled)

The 3 methods for enabling the A20 Gate are
1. Keyboard Controller
2. BIOS Function
3. System Port

===============================================================================================

Keyboard Controller:
This is the most common method of enabling A20 Gate.The keyboard micro-controller provides functions for disabling and enabling A20.Before enabling A20 we need to disable interrupts to prevent our kernel from getting messed up.The port 0x64 is used to send the command byte.

Command Bytes and ports
0xDD Enable A20 Address Line
0xDF Disable A20 Address Line 

0x64  Port of the 8042 micro-controller for sending commands

Using  the keyboard to enable A20:

EnableA20_KB:
cli                ;Disables interrupts
push	ax         ;Saves AX
mov	al, 0xdd  ;Look at the command list 
out	0x64, al   ;Command Register 
pop	ax          ;Restore's AX
sti                ;Enables interrupts
ret 



Using the BIOS functions to enable the A20 Gate:
The INT 15 2400,2401,2402 are used to disable,enable,return status of the A20 Gate respectively.

Return status of the commands 2400 and 2401(Disabling,Enabling)
CF = clear if success
AH = 0
CF = set on error
AH = status (01=keyboard controller is in secure mode, 0x86=function not supported)

 Return Status of the command 2402
CF = clear if success
AH = status (01: keyboard controller is in secure mode; 0x86: function not supported)
AL = current state (00: disabled, 01: enabled)
CX = set to 0xffff is keyboard controller is no ready in 0xc000 read attempts
CF = set on error 

Disabling the A20

push ax
mov ax, 0x2400 
int 0x15 
pop ax

Enabling the A20 
push ax
mov ax, 0x2401 
int 0x15 
pop ax

Checking A20
push ax
push cx
mov ax, 0x2402 
int 0x15 
pop cx
pop ax

=====================================================================================

Using System Port 0x92
This method is quite dangerous because it may cause conflicts with some hardware devices forcing the system to halt.

Port 0x92 Bits</xmp>
<ul style=""><li style=""><strong style="">Bit 0</strong>&nbsp;- Setting to 1 causes a fast reset&nbsp;</li><li style=""><strong style="">Bit 1</strong>&nbsp;- 0: disable A20, 1: enable A20</li><li style=""><strong style="">Bit 2</strong>&nbsp;- Manufacturer defined</li><li style=""><strong style="">Bit 3</strong>&nbsp;- power on password bytes. 0: accessible, 1: inaccessible</li><li style=""><strong style="">Bits 4-5</strong>&nbsp;- Manufacturer defined</li><li style=""><strong style="">Bits 6-7</strong>&nbsp;- 00: HDD activity LED off, 01 or any value is "on"</li></ul>
<xmp class="my_xmp_class">Code to enable A20 through port 0x92
push ax
mov	al, 2 out	0x92, al pop ax</xmp>