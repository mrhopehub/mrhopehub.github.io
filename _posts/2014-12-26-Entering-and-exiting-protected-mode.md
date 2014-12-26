---
layout: posts
title: "Entering and exiting protected mode"
---
# {{ page.title }}
摘自：Protected mode(Wikipedia)
<xmp class="my_xmp_class">Until the release of the 386, protected mode did not offer a direct method to switch back into real mode once protected mode was entered. IBM devised a workaround (implemented in the IBM AT) which involved resetting the CPU via the keyboard controller and saving the system registers, stack pointer and often the interrupt mask in the real-time clock chip's RAM. This allowed the BIOS to restore the CPU to a similar state and begin executing code before the reset.[clarification needed] Later, a triple fault was used to reset the 286 CPU, which was a lot faster and cleaner than the keyboard controller method (and does not depend on IBM AT-compatible hardware, but will work on any 80286 CPU in any system).

To enter protected mode, the Global Descriptor Table (GDT) must first be created with a minimum of three entries: a null descriptor, a code segment descriptor and data segment descriptor. In an IBM-compatible machine, the A20 line (21st address line) also must be enabled to allow the use of all the address lines so that the CPU can access beyond 1 megabyte of memory (Only the first 20 are allowed to be used after power-up, to guarantee compatibility with older software written for the Intel 8088-based IBM PC and PC/XT models). After performing those two steps, the PE bit must be set in the CR0 register and a far jump must be made to clear the prefetch input queue.
</xmp>
<xmp class="prettyprint linenums">; set PE bit
mov eax, cr0
or eax, 1
mov cr0, eax
 
; far jump (cs = selector of code segment)
jmp cs:@pm
 
@pm:
; Now we are in PM.</xmp>
<xmp class="my_xmp_class">With the release of the 386, protected mode could be exited by loading the segment registers with real mode values, disabling the A20 line and clearing the PE bit in the CR0 register, without the need to perform the initial setup steps required with the 286.
</xmp>
## 学而不思则罔
在使能PE位之后，需要执行far jmp，CS寄存器才会解释为段选择子，否则还是段基地址。