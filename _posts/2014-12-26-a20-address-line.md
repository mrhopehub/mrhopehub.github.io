---
layout: posts
title: "A20 地址线"
---
# {{ page.title }}
[转载：http://www.mouseos.com/arch/002.html](http://www.mouseos.com/arch/002.html)<br>
## 2.1 先看看 real mode 的寻址方法
<xmp class="my_xmp_class">8086/8088 的地址线有 20 条：A0 ~ A19，意味着 processor 可以将 20 位地址放上这 20 条地址线上，它的寻址能力是 1M (00000 ~ FFFFF），它的寻址方法是：segment:offset，这是一种被称为 logic address（逻辑地址）表示法，它需要转化为 processor 的 linear address（线性地址）表示：</xmp>
<table style="BORDER-BOTTOM: #cccccc 1px dotted; BORDER-LEFT: #cccccc 1px dotted; TABLE-LAYOUT: fixed; BORDER-TOP: #cccccc 1px dotted; BORDER-RIGHT: #cccccc 1px dotted" border="0" cellspacing="0" cellpadding="6" width="95%" align="center">
  <tr>
    <td style="WORD-WRAP: break-word" bgcolor="#fdfddf"><p>segment:offset ---------&gt; segment&nbsp;&lt;&lt; 4 + offset</p></td>
  </tr>
</table>
<xmp class="my_xmp_class">如：F000:FFFF = F0000 + FFFF = FFFFF，这是 8086/8088 所能访问的最高地址。这种表示方法是 Intel 为了在 16 位 real mode 下能够访问 20 位地址空间所想设计出来的计算方式。

因此，8086/8088 的寻址范围是可以表示为：从 0000:0000 ~ 0000:FFFF 开始到 F000:0000 ~ F000:FFFF</xmp>
## 2.2　访问 extended memory
<xmp class="my_xmp_class">在后续的 80286 上，Intel 实现了 24 位的 Address bus，那么在 real mode 下 80286 能够访问到的最高地址是 10FFEF，这个地址值是由下面的方法而来：</xmp>
<table style="BORDER-BOTTOM: #cccccc 1px dotted; BORDER-LEFT: #cccccc 1px dotted; TABLE-LAYOUT: fixed; BORDER-TOP: #cccccc 1px dotted; BORDER-RIGHT: #cccccc 1px dotted" border="0" cellspacing="0" cellpadding="6" width="95%" align="center">
  <tr>
    <td style="WORD-WRAP: break-word" bgcolor="#fdfddf"><p><strong>FFFF:FFFF = FFFF0 + FFFF = 10FFEFh</strong></p></td>
  </tr>
</table>
<xmp class="my_xmp_class">这已经是 logic address 所能表达的极限范围了。100000h 以上的内存被称为 extend memory，从 100000h ~ 10FFEFh 这片内存区域在 DOS 下被称为 High Memory（高端内存）。高端内存是 80286 在 real mode 所能访问到的区域，而 8086/8088 所不能访问到的。</xmp>
## 2.3　wraparound 现象
<xmp class="my_xmp_class">当在 8086/8088 下执行 FFFF:FFFF 这个内存寻址时，会产生什么结果呢？</xmp>
<table width="940" height="40" border="1">
  <tr>
    <td width="33"  style="border-style:hidden"><div align="center"></div></td>
    <td width="33" style="border-style:hidden"><div align="center"></div></td>
    <td width="33" style="border-style:hidden"><div align="center"></div></td>
    <td width="33" style="border-style:hidden"><div align="center"></div></td>
    <td width="33"><div align="center">19</div></td>
    <td width="33"><div align="center">18</div></td>
    <td width="33"><div align="center">17</div></td>
    <td width="33"><div align="center">16</div></td>
    <td width="33"><div align="center">15</div></td>
    <td width="33"><div align="center">14</div></td>
    <td width="33"><div align="center">13</div></td>
    <td width="33"><div align="center">12</div></td>
    <td width="33"><div align="center">11</div></td>
    <td width="33"><div align="center">10</div></td>
    <td width="33"><div align="center">9</div></td>
    <td width="33"><div align="center">8</div></td>
    <td width="33"><div align="center">7</div></td>
    <td width="33"><div align="center">6</div></td>
    <td width="33"><div align="center">5</div></td>
    <td width="33"><div align="center">4</div></td>
    <td width="33"><div align="center">3</div></td>
    <td width="33"><div align="center">2</div></td>
    <td width="33"><div align="center">1</div></td>
    <td width="33"><div align="center">0</div></td>
  </tr>
  <tr>
    <td style="border-style:hidden"><div align="center">0</div></td>
    <td style="border-style:hidden"><div align="center">0</div></td>
    <td style="border-style:hidden"><div align="center">0</div></td>
    <td style="border-style:hidden"><div align="center">1</div></td>
    <td><div align="center">0</div></td>
    <td><div align="center">0</div></td>
    <td><div align="center">0</div></td>
    <td><div align="center">0</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">0</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
    <td><div align="center">1</div></td>
  </tr>
</table>
<xmp class="my_xmp_class">结果很明显：由于 8086/8088 只有 20 条 address bus，地址 10FFEF 的高 4 位会被抛弃，实际上送上 address bus 的只有 0FFEFh 值，所以访问 FFFF:FFFF 地址结果只能访问到 1M 以内的地址。这就是 wraparound 现象：访问 1M 以上地址都会回绕到 1M 内的模值。

那么，当 80286 下访问 FFFF:FFFF 地址时，又会产生什么果呢？ 由于 80286 具有 24 条 address bus，对于 FFFF:FFFF 地址的访问，会正确得到访问。

SO，访问 FFFF:FFFF 内存，使得 8086/8088 下产生 wraparound 现象，变相访问 0FFEF 地址内存。而在 80286 下得到正确的的 10FFEF 地址，不存在 wraparound 现象。因此：wraparound 现象在 8086/8088 才会产生。这样产生的问题是：访问高端内存时，80286 在 real mode 下和 8086/8088 的行为不一致！</xmp>
## 2.4　引入 A20 Gate
<xmp class="my_xmp_class">为了使用 80286 和 8086/8088 在 real mode 下的行为一致，即：在 80286 下也产生 wraparound 现象。IBM 想出了古怪方法：当 80286 运行在 real mode 时，将 A20 地址线（第 21 条 address bus）置为 0 ，这样使得 80286 在 real mode 下第 21 条 address line 无效，从而人为造成了 wraparound 现象。

具体实现方法是：</xmp>
<table style="BORDER-BOTTOM: #cccccc 1px dotted; BORDER-LEFT: #cccccc 1px dotted; TABLE-LAYOUT: fixed; BORDER-TOP: #cccccc 1px dotted; BORDER-RIGHT: #cccccc 1px dotted" border="0" cellspacing="0" cellpadding="6" width="95%" align="center">
  <tr>
    <td style="WORD-WRAP: break-word" bgcolor="#fdfddf"><p>设立一个 <strong>AND Gate</strong>（与门电路），<strong>AND gate</strong> 的 <strong>IN</strong> 输入端中一端接 A20 line 上，另一端接在 keyboard control 8042 上，而 AND gate 的 <strong>OUT</strong> 输出端接在 A20 line 上。<span class="STYLE9">只有两个 IN 端都为 1 时，OUT 端才为 1</span></p></td>
  </tr>
</table>
<div align=center><img src="/images/a20-address-line/a20_gate.png" alt=""></div>
<xmp class="my_xmp_class">A20 line 一直处于 1 状态（High 电平），而 8042 内的 A20 gate 一直处于 0（Low 电平），因此：必须使 Keyboard controller 8042 内的 A20 Gate 处于 high 时，A20 line 输出才有效。 A20 gate 也被称为 A20 MASK#。

SO，Keyboard Controller 8042 增加了一组命令去控制 A20 Gate 的开/关，给 8042 发送命令 0xDF 置 A20 gate 有效，给 8042 送命令 0xDD 置 A20 gate 无效。

现在的 system 中，南桥芯片的 A20 MASK# 缺省都是 MASK 状态，即：A20 gate 缺省都是开的。</xmp>
## 2.5　打开 A20 gate 的必要性
<xmp class="my_xmp_class">打开 A20 gate 是为了在 80286/286+ 以后的 processor 上使用 protected mode 来访问完全的 24/24+ 位地址空间，如：在 32 位 protected mode 下，在不打开 A20 gate 的情况下，Bit20 为 0，导致 Bit20 留下一个空位。</xmp>
<table width="1039" height="26" border="1">
  <tr>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center" style="background-color: #00FFFF"><strong>0</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
    <td><div align="center"><strong>1</strong></div></td>
  </tr>
</table>