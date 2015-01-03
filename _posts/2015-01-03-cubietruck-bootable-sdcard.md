---
layout: posts
title: "ubuntu10.04制作cubietruck可启动SD卡"
---
# {{ page.title }}
## 创建工程目录
<xmp class="prettyprint linenums">cd $HOME
mkdir cubietruck-build
cd cubietruck-build/
mkdir cubietruck-uboot
mkdir cubietruck-kernel
mkdir cubietruck-sdcard</xmp>
cubietruck-build为总目录，cubietruck-uboot用于编译uboot，cubietruck-kernel用于编译内核，cubietruck-sdcard用于将编译结果、根文件系统写入SD卡。
## 安装工具链
<xmp class="prettyprint linenums">sudo apt-get install git-core
sudo add-apt-repository ppa:linaro-maintainers/toolchain
sudo apt-get update
sudo apt-get install gcc-arm-linux-gnueabi

sudo apt-get install libncurses5-dev

sudo apt-get install libusb-1.0-0-dev
export PKG_CONFIG_PATH=/usr/lib/pkgconfig:$PKG_CONFIG_PATH
sudo pkg-config libusb-1.0 --cflags --libs</xmp>
gcc-arm-linux-gnueabi为交叉编译工具，libncurses5-dev为menuconfig需要的库，libusb-1.0-0-dev为编译bin2fex、fex2bin需要的库
## 获取源码
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-uboot
git clone https://github.com/linux-sunxi/u-boot-sunxi.git
cd $HOME/cubietruck-build/cubietruck-kernel
git clone -b sunxi-3.4 https://github.com/linux-sunxi/linux-sunxi.git
git clone https://github.com/cubieboard/cubie_configs
git clone git://github.com/linux-sunxi/sunxi-tools.git</xmp>
## 编译Uboot
修改$HOME/cubietruck-build/cubietruck-uboot/u-boot-sunxi/include/configs/sunxi-common.h文件，去掉以下两块代码
<xmp class="prettyprint linenums">      "fatload $device $partition $scriptaddr ${bootscr}" \
	  " || " \
	  "ext2load $device $partition $scriptaddr boot/${bootscr}" \
	  " ||" \</xmp>
第二块
<xmp class="prettyprint linenums">      "fatload $device $partition $scriptaddr ${bootenv}" \
	  " || " \
	  "ext2load $device $partition $scriptaddr boot/${bootenv}" \
	  " || " \</xmp>
开始编译
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-uboot/u-boot-sunxi
make CROSS_COMPILE=arm-linux-gnueabi- Cubietruck_config
make CROSS_COMPILE=arm-linux-gnueabi-</xmp>
## 编译内核
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-kernel
export PATH=$HOME/cubietruck-build/cubietruck-uboot/u-boot-sunxi/tools:$PATH
cp cubie_configs/kernel-configs/3.4/cubietruck_defconfig linux-sunxi/.config
cd linux-sunxi
make ARCH=arm menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- uImage modules</xmp>
## 编译sunxi-tools
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-kernel/sunxi-tools
make</xmp>
## 构建script.bin
复制$HOME/cubietruck-build/cubietruck-kernel/cubie_configs/sysconfig/linux/目录下的cubietruck.fex到$HOME/cubietruck-build/cubietruck-kernel，修改screen0_output_type为4（VGA显示），还需要添加以下项
<xmp class="prettyprint linenums">[dynamic]
MAC = "028A0AC2353E"</xmp>
构建script.bin
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-kernel
rm script.bin
./sunxi-tools/fex2bin cubietruck.fex script.bin</xmp>
## 构建boot.scr
在$HOME/cubietruck-build/cubietruck-kernel/目录下新建boot.cmd，添加以下内容
<xmp class="prettyprint linenums">setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait panic=10
ext2load mmc 0 0x43000000 script.bin
ext2load mmc 0 0x48000000 uImage
bootm 0x48000000</xmp>
构建boot.scr
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-kernel
export PATH=$HOME/cubietruck-build/cubietruck-uboot/u-boot-sunxi/tools:$PATH
rm boot.scr
mkimage -C none -A arm -T script -d boot.cmd boot.scr</xmp>
## uEnv.txt
在$HOME/cubietruck-build/cubietruck-kernel/目录下新建uEnv.txt，这个文件的作用是存放uboot的环境变量，这里暂时不添加任何内容。
## 制作根文件系统
这里并没有自己编译根文件系统，而是下载别人制作好的根文件系统。可以到[https://snapshots.linaro.org/ubuntu/images](https://snapshots.linaro.org/ubuntu/images)下载，这里选择https://snapshots.linaro.org/ubuntu/images/developer/latest的最新版本为linaro-utopic-developer-20141212-693.tar.gz，下载好之后放在$HOME/cubietruck-build/cubietruck-sdcard目录下，用以下命令解压
<xmp class="prettyprint linenums">export ROOTFS_TARBALL=linaro-utopic-developer-20141212-693.tar.gz
cd $HOME/cubietruck-build/cubietruck-sdcard
sudo rm -rv binary/
sudo tar -zxvf ${ROOTFS_TARBALL}</xmp>
$HOME/cubietruck-build/cubietruck-sdcard/binary目录下为根文件系统
## SD卡分区
SD卡为sdb，中间需要输入分区参数，输入内容为(回车代表回车键)：--》c--》u--》n--》p--》1--》回车--》+16M--》n--》p--》2--》回车--》回车--》w
<xmp class="prettyprint linenums">export card=/dev/sdb
export p=""
sudo umount ${card}?
sudo dd if=/dev/zero of=${card} bs=1M count=1
sudo fdisk ${card}
sudo mkfs.ext2 ${card}${p}1
sudo mkfs.ext4 ${card}${p}2</xmp>
## 写入Uboot
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-uboot
export card=/dev/sdb
export p=""
sudo dd if=/dev/zero of=${card} bs=1k count=1023 seek=1
sudo dd if=u-boot-sunxi/u-boot-sunxi-with-spl.bin of=${card} bs=1024 seek=8</xmp>
## 复制启动分区
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-sdcard
export card=/dev/sdb
export p=""
sudo umount ${card}${p}1
sudo mount ${card}${p}1 /mnt
sudo rm -rv /mnt/*
sudo cp -v ../cubietruck-kernel/uEnv.txt /mnt
sudo cp -v ../cubietruck-kernel/boot.scr /mnt
sudo cp -v ../cubietruck-kernel/script.bin /mnt
sudo cp -v ../cubietruck-kernel/linux-sunxi/arch/arm/boot/uImage /mnt
sudo umount ${card}${p}1</xmp>
## 复制根文件系统
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-sdcard
export card=/dev/sdb
export p=""
sudo umount ${card}${p}2
sudo mount ${card}${p}2 /mnt
sudo rm -rv /mnt/*
sudo cp -rv binary/* /mnt
sudo umount ${card}${p}2</xmp>
## 测试
插入sd卡，连接TTL，一下是串口打印信息
<xmp class="prettyprint linenums">
U-Boot SPL 2014.04-10733-gea1ac32-dirty (Jan 03 2015 - 17:09:46)
Board: Cubietruck
DRAM: 2048 MiB
CPU: 960000000Hz, AXI/AHB/APB: 3/2/2
spl: not an uImage at 1600


U-Boot 2014.04-10733-gea1ac32-dirty (Jan 03 2015 - 17:09:46) Allwinner Technology

CPU:   Allwinner A20 (SUN7I)
Board: Cubietruck
I2C:   ready
DRAM:  2 GiB
MMC:   SUNXI SD/MMC: 0
*** Warning - bad CRC, using default environment

In:    serial
Out:   serial
Err:   serial
Net:   dwmac.1c50000
Hit any key to stop autoboot:  0 
234 bytes read in 55 ms (3.9 KiB/s)
Jumping to boot.scr
## Executing script at 44000000
46332 bytes read in 82 ms (551.8 KiB/s)
4597160 bytes read in 510 ms (8.6 MiB/s)
## Booting kernel from Legacy Image at 48000000 ...
   Image Name:   Linux-3.4.103+
   Image Type:   ARM Linux Kernel Image (uncompressed)
   Data Size:    4597096 Bytes = 4.4 MiB
   Load Address: 40008000
   Entry Point:  40008000
   Verifying Checksum ... OK
   Loading Kernel Image ... OK

Starting kernel ...

<6>Booting Linux on physical CPU 0
<6>Initializing cgroup subsys cpuset
<6>Initializing cgroup subsys cpu
<5>Linux version 3.4.103+ (mrhopehub@mrhopehub-desktop) (gcc version 4.6.2 (Ubuntu/Linaro 4.6.2-14ubuntu2~ppa1) ) #1 SMP PREEMPT Sat Jan 3 17:08:09 CST 2015
CPU: ARMv7 Processor [410fc074] revision 4 (ARMv7), cr=10c5387d
CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
Machine: sun7i
<6>Memory Reserved:
<6>     SYS  : 0x43000000 - 0x4300ffff  (  64 kB)
<6>     VE   : 0x44000000 - 0x48ffffff  (  80 MB)
<6>     G2D  : 0x49000000 - 0x49ffffff  (  16 MB)
<6>     LCD  : 0x4a000000 - 0x4bffffff  (  32 MB)
Memory policy: ECC disabled, Data cache writealloc
<6>sunxi: Allwinner A20 (AW1651/sun7i) detected.
<7>On node 0 totalpages: 524288
<7>free_area_init_node: node 0, pgdat c088aa00, node_mem_map d0000000
<7>  DMA zone: 512 pages used for memmap
<7>  DMA zone: 0 pages reserved
<7>  DMA zone: 65024 pages, LIFO batch:15
<7>  Normal zone: 1008 pages used for memmap
<7>  Normal zone: 128016 pages, LIFO batch:31
<7>  HighMem zone: 2576 pages used for memmap
<7>  HighMem zone: 327152 pages, LIFO batch:31
<6>PERCPU: Embedded 7 pages/cpu @d100e000 s7360 r8192 d13120 u32768
<7>pcpu-alloc: s7360 r8192 d13120 u32768 alloc=8*4096<c>
<7>pcpu-alloc: <c>[0] <c>0 <c>[0] <c>1 <c>
Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 520192
<5>Kernel command line: console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait panic=10
<6>PID hash table entries: 4096 (order: 2, 16384 bytes)
<6>Dentry cache hash table entries: 131072 (order: 7, 524288 bytes)
<6>Inode-cache hash table entries: 65536 (order: 6, 262144 bytes)
<6>allocated 4194304 bytes of page_cgroup
<6>please try 'cgroup_disable=memory' option if you don't want memory cgroups
<6>Memory: 2048MB = 2048MB total
<5>Memory: 1934044k/1934044k available, 163108k reserved, 1318912K highmem
<5>Virtual kernel memory layout:
    vector  : 0xffff0000 - 0xffff1000   (   4 kB)
    fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
    vmalloc : 0xf0000000 - 0xff000000   ( 240 MB)
    lowmem  : 0xc0000000 - 0xef800000   ( 760 MB)
    pkmap   : 0xbfe00000 - 0xc0000000   (   2 MB)
    modules : 0xbf000000 - 0xbfe00000   (  14 MB)
      .text : 0xc0008000 - 0xc07ee74c   (8090 kB)
      .init : 0xc07ef000 - 0xc0824cc0   ( 216 kB)
      .data : 0xc0826000 - 0xc08915e8   ( 430 kB)
       .bss : 0xc089160c - 0xc0a4f838   (1785 kB)
<6>SLUB: Genslabs=11, HWalign=64, Order=0-3, MinObjects=0, CPUs=2, Nodes=1
<6>Preemptible hierarchical RCU implementation.
<6>     RCU dyntick-idle grace-period acceleration is enabled.
<6>     Additional per-CPU info printed with stalls.
<6>NR_IRQS:192
<6>Architected local timer running at 24.00MHz.
<6>sched_clock: 32 bits at 24MHz, resolution 41ns, wraps every 178956ms
<2>start_kernel(): bug: interrupts were enabled early
<6>Console: colour dummy device 80x30
<6>Calibrating delay loop... <c>1915.28 BogoMIPS (lpj=9576448)
<6>pid_max: default: 32768 minimum: 301
<6>Mount-cache hash table entries: 512
<6>Initializing cgroup subsys cpuacct
<6>Initializing cgroup subsys memory
<6>Initializing cgroup subsys devices
<6>Initializing cgroup subsys freezer
<6>Initializing cgroup subsys blkio
<6>Initializing cgroup subsys perf_event
<6>CPU: Testing write buffer coherency: ok
<6>CPU0: thread -1, cpu 0, socket 0, mpidr 80000000
<6>hw perfevents: enabled with ARMv7 Cortex-A7 PMU driver, 5 counters available
<6>Setting up static identity map for 0x40582118 - 0x40582170
CPU1: Booted secondary processor
<6>CPU1: thread -1, cpu 1, socket 0, mpidr 80000001
<6>Brought up 2 CPUs
<6>SMP: Total of 2 processors activated (3830.57 BogoMIPS).
<6>devtmpfs: initialized
<6>dummy: 
<6>NET: Registered protocol family 16
<6>DMA: preallocated 256 KiB pool for atomic coherent allocations
<6>hw-breakpoint: found 5 (+1 reserved) breakpoint and 4 watchpoint registers.
<6>hw-breakpoint: maximum watchpoint size is 8 bytes.
<6>[ccu-inf] aw clock manager init
<6>[ccu-inf] aw_ccu_init
<6>[ccu-inf] script config pll3 to 297MHz
<6>[ccu-inf] script config pll4 to 300MHz
<6>[ccu-inf] script config pll6 to 600MHz
<6>[ccu-inf] script config pll7 to 297MHz
<6>[ccu-inf] script config pll8 to 336MHz
<6>Init eGon pin module V2.0
<6>bio: create slab <bio-0> at 0
<6>sunxi_gpio driver init ver 1.3
<6>gpiochip_add: registered GPIOs 1 to 2 on device: A1X_GPIO
<5>SCSI subsystem initialized
<7>libata version 3.00 loaded.
<6>usbcore: registered new interface driver usbfs
<6>usbcore: registered new interface driver hub
<6>usbcore: registered new device driver usb
<6>Linux media interface: v0.10
<6>Linux video capture interface: v2.00
<6>Advanced Linux Sound Architecture Driver Version 1.0.25.
<6>cfg80211: Calling CRDA to update world regulatory domain
<6>Switching to clocksource arch_sys_counter
<5>FS-Cache: Loaded
<6>CacheFiles: Loaded
<6>sw_hcd_host0 sw_hcd_host0: sw_hcd host driver
<6>sw_hcd_host0 sw_hcd_host0: new USB bus registered, assigned bus number 1
<6>hub 1-0:1.0: USB hub found
<6>hub 1-0:1.0: 1 port detected
<6>NET: Registered protocol family 2
<6>IP route cache hash table entries: 32768 (order: 5, 131072 bytes)
<6>TCP established hash table entries: 131072 (order: 8, 1048576 bytes)
<6>TCP bind hash table entries: 65536 (order: 7, 786432 bytes)
<6>TCP: Hash tables configured (established 131072 bind 65536)
<6>TCP: reno registered
<6>UDP hash table entries: 512 (order: 2, 16384 bytes)
<6>UDP-Lite hash table entries: 512 (order: 2, 16384 bytes)
<6>NET: Registered protocol family 1
<6>RPC: Registered named UNIX socket transport module.
<6>RPC: Registered udp transport module.
<6>RPC: Registered tcp transport module.
<6>RPC: Registered tcp NFSv4.1 backchannel transport module.
<6>audit: initializing netlink socket (disabled)
<5>type=2000 audit(0.540:1): initialized
highmem bounce pool size: 64 pages
<5>VFS: Disk quotas dquot_6.5.2
Dquot-cache hash table entries: 1024 (order 0, 4096 bytes)
<5>NFS: Registering the id_resolver key type
<6>NTFS driver 2.1.30 [Flags: R/W].
<6>fuse init (API version 7.18)
<6>msgmni has been set to 1201
<6>alg: No test for stdrng (krng)
<6>Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
<6>io scheduler noop registered
<6>io scheduler deadline registered
<6>io scheduler cfq registered (default)
<6>sunxi disp driver loaded (/dev/disp api 1.0)
<6>Serial: 8250/16550 driver, 8 ports, IRQ sharing disabled
<6>[uart]: used uart info.: 0x05
<6>[uart]: serial probe 0 irq 33 mapbase 0x01c28000
<6>sunxi-uart.0: ttyS0 at MMIO 0x1c28000 (irq = 33) is a U6_16550A
[    0.000000] Booting Linux on physical CPU 0
[    0.000000] Initializing cgroup subsys cpuset
[    0.000000] Initializing cgroup subsys cpu
[    0.000000] Linux version 3.4.103+ (mrhopehub@mrhopehub-desktop) (gcc version 4.6.2 (Ubuntu/Linaro 4.6.2-14ubuntu2~ppa1) ) #1 SMP PREEMPT Sat Jan 3 17:08:09 CST 2015
[    0.000000] Memory Reserved:
[    0.000000]  SYS  : 0x43000000 - 0x4300ffff  (  64 kB)
[    0.000000]  VE   : 0x44000000 - 0x48ffffff  (  80 MB)
[    0.000000]  G2D  : 0x49000000 - 0x49ffffff  (  16 MB)
[    0.000000]  LCD  : 0x4a000000 - 0x4bffffff  (  32 MB)
[    0.000000] sunxi: Allwinner A20 (AW1651/sun7i) detected.
[    0.000000] PERCPU: Embedded 7 pages/cpu @d100e000 s7360 r8192 d13120 u32768
[    0.000000] Kernel command line: console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait panic=10
[    0.000000] PID hash table entries: 4096 (order: 2, 16384 bytes)
[    0.000000] Dentry cache hash table entries: 131072 (order: 7, 524288 bytes)
[    0.000000] Inode-cache hash table entries: 65536 (order: 6, 262144 bytes)
[    0.000000] allocated 4194304 bytes of page_cgroup
[    0.000000] please try 'cgroup_disable=memory' option if you don't want memory cgroups
[    0.000000] Memory: 2048MB = 2048MB total
[    0.000000] Memory: 1934044k/1934044k available, 163108k reserved, 1318912K highmem
[    0.000000] Virtual kernel memory layout:
[    0.000000]     vector  : 0xffff0000 - 0xffff1000   (   4 kB)
[    0.000000]     fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
[    0.000000]     vmalloc : 0xf0000000 - 0xff000000   ( 240 MB)
[    0.000000]     lowmem  : 0xc0000000 - 0xef800000   ( 760 MB)
[    0.000000]     pkmap   : 0xbfe00000 - 0xc0000000   (   2 MB)
[    0.000000]     modules : 0xbf000000 - 0xbfe00000   (  14 MB)
[    0.000000]       .text : 0xc0008000 - 0xc07ee74c   (8090 kB)
[    0.000000]       .init : 0xc07ef000 - 0xc0824cc0   ( 216 kB)
[    0.000000]       .data : 0xc0826000 - 0xc08915e8   ( 430 kB)
[    0.000000]        .bss : 0xc089160c - 0xc0a4f838   (1785 kB)
[    0.000000] SLUB: Genslabs=11, HWalign=64, Order=0-3, MinObjects=0, CPUs=2, Nodes=1
[    0.000000] Preemptible hierarchical RCU implementation.
[    0.000000]  RCU dyntick-idle grace-period acceleration is enabled.
[    0.000000]  Additional per-CPU info printed with stalls.
[    0.000000] NR_IRQS:192
[    0.000000] Architected local timer running at 24.00MHz.
[    0.000000] sched_clock: 32 bits at 24MHz, resolution 41ns, wraps every 178956ms
[    0.000000] start_kernel(): bug: interrupts were enabled early
[    0.000000] Console: colour dummy device 80x30
[    0.010880] Calibrating delay loop... 1915.28 BogoMIPS (lpj=9576448)
[    0.076016] pid_max: default: 32768 minimum: 301
[    0.079685] Mount-cache hash table entries: 512
[    0.083888] Initializing cgroup subsys cpuacct
[    0.087207] Initializing cgroup subsys memory
[    0.090629] Initializing cgroup subsys devices
[    0.094067] Initializing cgroup subsys freezer
[    0.097292] Initializing cgroup subsys blkio
[    0.100968] Initializing cgroup subsys perf_event
[    0.104546] CPU: Testing write buffer coherency: ok
[    0.109722] CPU0: thread -1, cpu 0, socket 0, mpidr 80000000
[    0.116822] hw perfevents: enabled with ARMv7 Cortex-A7 PMU driver, 5 counters available
[    0.122355] Setting up static identity map for 0x40582118 - 0x40582170
[    0.297216] CPU1: thread -1, cpu 1, socket 0, mpidr 80000001
[    0.299214] Brought up 2 CPUs
[    0.304523] SMP: Total of 2 processors activated (3830.57 BogoMIPS).
[    0.307669] devtmpfs: initialized
[    0.312876] dummy: 
[    0.316549] NET: Registered protocol family 16
[    0.322681] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.330241] hw-breakpoint: found 5 (+1 reserved) breakpoint and 4 watchpoint registers.
[    0.335058] hw-breakpoint: maximum watchpoint size is 8 bytes.
[    0.338210] [ccu-inf] aw clock manager init
[    0.340480] [ccu-inf] aw_ccu_init
[    0.344436] [ccu-inf] script config pll3 to 297MHz
[    0.348187] [ccu-inf] script config pll4 to 300MHz
[    0.351934] [ccu-inf] script config pll6 to 600MHz
[    0.355695] [ccu-inf] script config pll7 to 297MHz
[    0.359442] [ccu-inf] script config pll8 to 336MHz
[    0.362178] Init eGon pin module V2.0
[    0.370267] bio: create slab <bio-0> at 0
[    0.373746] sunxi_gpio driver init ver 1.3
[    0.379385] gpiochip_add: registered GPIOs 1 to 2 on device: A1X_GPIO
[    0.382494] SCSI subsystem initialized
[    0.390001] usbcore: registered new interface driver usbfs
[    0.394322] usbcore: registered new interface driver hub
[    0.398468] usbcore: registered new device driver usb
[    0.401581] Linux media interface: v0.10
[    0.405184] Linux video capture interface: v2.00
[    0.410738] Advanced Linux Sound Architecture Driver Version 1.0.25.
[    0.416875] cfg80211: Calling CRDA to update world regulatory domain
[    0.421270] Switching to clocksource arch_sys_counter
[    0.423377] FS-Cache: Loaded
[    0.425561] CacheFiles: Loaded
[    0.458912] sw_hcd_host0 sw_hcd_host0: sw_hcd host driver
[    0.465620] sw_hcd_host0 sw_hcd_host0: new USB bus registered, assigned bus number 1
[    0.469010] hub 1-0:1.0: USB hub found
[    0.471904] hub 1-0:1.0: 1 port detected
[    0.476425] NET: Registered protocol family 2
[    0.492720] IP route cache hash table entries: 32768 (order: 5, 131072 bytes)
[    0.499619] TCP established hash table entries: 131072 (order: 8, 1048576 bytes)
[    0.507055] TCP bind hash table entries: 65536 (order: 7, 786432 bytes)
[    0.513581] TCP: Hash tables configured (established 131072 bind 65536)
[    0.515783] TCP: reno registered
[    0.520660] UDP hash table entries: 512 (order: 2, 16384 bytes)
[    0.525991] UDP-Lite hash table entries: 512 (order: 2, 16384 bytes)
[    0.529598] NET: Registered protocol family 1
[    0.534871] RPC: Registered named UNIX socket transport module.
[    0.538570] RPC: Registered udp transport module.
[    0.542227] RPC: Registered tcp transport module.
[    0.547629] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.553012] audit: initializing netlink socket (disabled)
[    0.556765] type=2000 audit(0.540:1): initialized
[    0.572088] VFS: Disk quotas dquot_6.5.2
[    0.584279] NFS: Registering the id_resolver key type
[    0.588870] NTFS driver 2.1.30 [Flags: R/W].
[    0.592152] fuse init (API version 7.18)
[    0.595655] msgmni has been set to 1201
[    0.602724] alg: No test for stdrng (krng)
[    0.609434] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
[    0.612321] io scheduler noop registered
[    0.615579] io scheduler deadline registered
[    0.619268] io scheduler cfq registered (default)
[    0.624031] sunxi disp driver loaded (/dev/disp api 1.0)
[    0.629617] Serial: 8250/16550 driver, 8 ports, IRQ sharing disabled
[    0.633973] [uart]: used uart info.: 0x05
[    0.638719] [uart]: serial probe 0 irq 33 mapbase 0x01c28000
[    0.664933] sunxi-uart.0: ttyS0 at MMIO 0x1c28000 (irq = 33) is a U6_16550A
<6>console [ttyS0] enabled
[    1.286018] console [ttyS0] enabled
<6>[uart]: serial probe 2 irq 35 mapbase 0x01c28800
[    1.294352] [uart]: serial probe 2 irq 35 mapbase 0x01c28800
<6>sunxi-uart.2: ttyS1 at MMIO 0x1c28800 (irq = 35) is a U6_16550A
[    1.326194] sunxi-uart.2: ttyS1 at MMIO 0x1c28800 (irq = 35) is a U6_16550A
<7>G2D: drv_g2d_init
<6>G2D: g2dmem: g2d_start=49000000, g2d_size=1000000
[    1.340382] G2D: g2dmem: g2d_start=49000000, g2d_size=1000000
<6>G2D: head:c9000000,tail:ca000000
[    1.349370] G2D: head:c9000000,tail:ca000000
<6>G2D: Module initialized.major:250
[    1.356988] G2D: Module initialized.major:250
<6>brd: module loaded
[    1.364273] brd: module loaded
<6>loop: module loaded
[    1.372865] loop: module loaded
[NAND] nand driver version: 0x2 0x9 
<4>Dev Sunxi softw311 nand magic does not match for MBR 1: 
[    1.447864] Dev Sunxi softw311 nand magic does not match for MBR 1: 
<4>Dev Sunxi softw311 nand magic does not match for MBR 2: 
[    1.460943] Dev Sunxi softw311 nand magic does not match for MBR 2: 
<4>Dev Sunxi softw311 nand magic does not match for MBR 3: 
[    1.473990] Dev Sunxi softw311 nand magic does not match for MBR 3: 
<4>Dev Sunxi softw311 nand magic does not match for MBR 4: 
[    1.487033] Dev Sunxi softw311 nand magic does not match for MBR 4: 
<4>Dev Sunxi softw311 nand header bad for all MBR copies, MBR corrupted or not present.
[    1.501815] Dev Sunxi softw311 nand header bad for all MBR copies, MBR corrupted or not present.
<4>Dev Sunxi softw411 nand magic does not match for MBR 1: 
[    1.516605] Dev Sunxi softw411 nand magic does not match for MBR 1: 
<4>Dev Sunxi softw411 nand magic does not match for MBR 2: 
[    1.531745] Dev Sunxi softw411 nand magic does not match for MBR 2: 
<4>Dev Sunxi softw411 nand magic does not match for MBR 3: 
[    1.546867] Dev Sunxi softw411 nand magic does not match for MBR 3: 
<4>Dev Sunxi softw411 nand magic does not match for MBR 4: 
[    1.561977] Dev Sunxi softw411 nand magic does not match for MBR 4: 
<4>Dev Sunxi softw411 nand header bad for all MBR copies, MBR corrupted or not present.
[    1.576761] Dev Sunxi softw411 nand header bad for all MBR copies, MBR corrupted or not present.
<6> nand: unknown partition table
[    1.588591]  nand: unknown partition table
[NAND]nand driver, ok.
<6>sw_ahci sw_ahci.0: controller can't do PMP, turning off CAP_PMP
[    1.601391] sw_ahci sw_ahci.0: controller can't do PMP, turning off CAP_PMP
<4>sw_ahci sw_ahci.0: forcing PORTS_IMPL to 0x1
[    1.612647] sw_ahci sw_ahci.0: forcing PORTS_IMPL to 0x1
<6>sw_ahci sw_ahci.0: AHCI 0001.0100 32 slots 1 ports 3 Gbps 0x1 impl platform mode
[    1.625395] sw_ahci sw_ahci.0: AHCI 0001.0100 32 slots 1 ports 3 Gbps 0x1 impl platform mode
<6>sw_ahci sw_ahci.0: flags: ncq sntf pm led clo only pio slum part ccc 
[    1.640270] sw_ahci sw_ahci.0: flags: ncq sntf pm led clo only pio slum part ccc 
<6>scsi0 : sw_ahci_platform
[    1.650975] scsi0 : sw_ahci_platform
<6>ata1: SATA max UDMA/133 mmio [mem 0x01c18000-0x01c18fff] port 0x100 irq 88
[    1.661728] ata1: SATA max UDMA/133 mmio [mem 0x01c18000-0x01c18fff] port 0x100 irq 88
<6>PPP generic driver version 2.4.2
[    1.673088] PPP generic driver version 2.4.2
<6>PPP BSD Compression module registered
[    1.681249] PPP BSD Compression module registered
<6>PPP Deflate Compression module registered
[    1.689959] PPP Deflate Compression module registered
<6>PPP MPPE Compression module registered
[    1.699870] PPP MPPE Compression module registered
<6>NET: Registered protocol family 24
[    1.708089] NET: Registered protocol family 24
<6>ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    1.718455] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
<6>ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    1.730210] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[sw-ehci1]: open clock
[sw-ehci1]: Set USB Power ON
<6>sw-ehci sw-ehci.1: SW USB2.0 'Enhanced' Host Controller (EHCI) Driver
[    1.767878] sw-ehci sw-ehci.1: SW USB2.0 'Enhanced' Host Controller (EHCI) Driver
<6>sw-ehci sw-ehci.1: new USB bus registered, assigned bus number 2
[    1.781570] sw-ehci sw-ehci.1: new USB bus registered, assigned bus number 2
<6>sw-ehci sw-ehci.1: irq 71, io mem 0x01c14000
[    1.792999] sw-ehci sw-ehci.1: irq 71, io mem 0x01c14000
<6>sw-ehci sw-ehci.1: USB 2.0 started, EHCI 1.00
[    1.815709] sw-ehci sw-ehci.1: USB 2.0 started, EHCI 1.00
ehci_irq: port change detect
<6>hub 2-0:1.0: USB hub found
[    1.827024] hub 2-0:1.0: USB hub found
<6>hub 2-0:1.0: 1 port detected
[    1.833665] hub 2-0:1.0: 1 port detected
[sw-ohci1]: open clock
<6>sw-ohci sw-ohci.1: SW USB2.0 'Open' Host Controller (OHCI) Driver
[    1.866244] sw-ohci sw-ohci.1: SW USB2.0 'Open' Host Controller (OHCI) Driver
<6>sw-ohci sw-ohci.1: new USB bus registered, assigned bus number 3
[    1.879607] sw-ohci sw-ohci.1: new USB bus registered, assigned bus number 3
<6>sw-ohci sw-ohci.1: irq 96, io mem 0x01c14400
[    1.890962] sw-ohci sw-ohci.1: irq 96, io mem 0x01c14400
<6>hub 3-0:1.0: USB hub found
[    1.958598] hub 3-0:1.0: USB hub found
<6>hub 3-0:1.0: 1 port detected
[    1.965265] hub 3-0:1.0: 1 port detected
[sw-ehci2]: open clock
[sw-ehci2]: Set USB Power ON
<6>sw-ehci sw-ehci.2: SW USB2.0 'Enhanced' Host Controller (EHCI) Driver
[    2.000783] sw-ehci sw-ehci.2: SW USB2.0 'Enhanced' Host Controller (EHCI) Driver
<6>sw-ehci sw-ehci.2: new USB bus registered, assigned bus number 4
<6>ata1: SATA link down (SStatus 0 SControl 300)
[    2.014479] sw-ehci sw-ehci.2: new USB bus registered, assigned bus number 4
[    2.018835] ata1: SATA link down (SStatus 0 SControl 300)
ehci_irq: port change detect
<6>sw-ehci sw-ehci.2: irq 72, io mem 0x01c1c000
[    2.038377] sw-ehci sw-ehci.2: irq 72, io mem 0x01c1c000
The port change to OHCI now!
<6>sw-ehci sw-ehci.2: USB 2.0 started, EHCI 1.00
[    2.115719] sw-ehci sw-ehci.2: USB 2.0 started, EHCI 1.00
<6>hub 4-0:1.0: USB hub found
[    2.124426] hub 4-0:1.0: USB hub found
<6>hub 4-0:1.0: 1 port detected
[    2.131067] hub 4-0:1.0: 1 port detected
[sw-ohci2]: open clock
<6>sw-ohci sw-ohci.2: SW USB2.0 'Open' Host Controller (OHCI) Driver
[    2.163663] sw-ohci sw-ohci.2: SW USB2.0 'Open' Host Controller (OHCI) Driver
<6>sw-ohci sw-ohci.2: new USB bus registered, assigned bus number 5
[    2.177050] sw-ohci sw-ohci.2: new USB bus registered, assigned bus number 5
<6>sw-ohci sw-ohci.2: irq 97, io mem 0x01c1c400
[    2.188411] sw-ohci sw-ohci.2: irq 97, io mem 0x01c1c400
<6>hub 5-0:1.0: USB hub found
[    2.258636] hub 5-0:1.0: USB hub found
<6>hub 5-0:1.0: 1 port detected
[    2.265311] hub 5-0:1.0: 1 port detected
<6>mousedev: PS/2 mouse device common for all mice
[    2.274250] mousedev: PS/2 mouse device common for all mice
<6>input: sunxi-ir as /devices/virtual/input/input0
[    2.284784] input: sunxi-ir as /devices/virtual/input/input0
IR Initial OK
===========================hv_keypad_init=====================
========HV Inital ===================
<3>tkey_fetch_sysconfig_para: tkey_unused. 
<6>usb 3-1: new low-speed USB device number 2 using sw-ohci
[    2.304839] tkey_fetch_sysconfig_para: tkey_unused. 
[    2.310146] usb 3-1: new low-speed USB device number 2 using sw-ohci
hv_keypad_init: after fetch_sysconfig_para:  normal_i2c: 0x0. normal_i2c[1]: 0x0 
<6>sunxi-rtc sunxi-rtc: rtc core: registered rtc as rtc0
[    2.334384] sunxi-rtc sunxi-rtc: rtc core: registered rtc as rtc0
<6>i2c /dev entries driver
[    2.343018] i2c /dev entries driver
config i2c gpio with gpio_config api
<6>axp_mfd 0-0034: AXP (CHIP ID: 0x41) detected
[    2.355704] axp_mfd 0-0034: AXP (CHIP ID: 0x41) detected
<6>axp_mfd 0-0034: AXP internal temperature monitoring enabled
[    2.366807] axp_mfd 0-0034: AXP internal temperature monitoring enabled
[AXP]axp driver uning configuration failed(342)
[AXP]power_start = 0
<4>i2c i2c-0: Invalid probe address 0x00
[    2.384116] i2c i2c-0: Invalid probe address 0x00
<6>I2C: i2c-0: AW16XX I2C adapter
[    2.391889] I2C: i2c-0: AW16XX I2C adapter
[ace_drv] start!!!
[ace_drv] init end!!!
[pa_drv] start!!!
[pa_drv] init end!!!
<6>axp20_ldo1: 1300 mV 
[    2.406305] axp20_ldo1: 1300 mV 
<6>axp20_ldo2: 1800 <--> 3300 mV at 3000 mV 
[    2.414699] axp20_ldo2: 1800 <--> 3300 mV at 3000 mV 
<6>axp20_ldo3: 700 <--> 3500 mV at 2800 mV 
[    2.425019] axp20_ldo3: 700 <--> 3500 mV at 2800 mV 
<6>axp20_ldo4: 1250 <--> 3300 mV at 2800 mV 
[    2.435123] axp20_ldo4: 1250 <--> 3300 mV at 2800 mV 
<6>axp20_buck2: 700 <--> 2275 mV at 1450 mV 
[    2.445522] axp20_buck2: 700 <--> 2275 mV at 1450 mV 
<3>somebody is trying to set dcdc3 range to (1300000, 1300000) uV
[    2.457057] somebody is trying to set dcdc3 range to (1300000, 1300000) uV
<3>but we keep dcdc3 = 1300000 uV from the bootloader
[    2.468998] but we keep dcdc3 = 1300000 uV from the bootloader
<6>axp20_buck3: 700 <--> 3500 mV at 1300 mV 
[    2.479082] axp20_buck3: 700 <--> 3500 mV at 1300 mV 
<6>axp20_ldoio0: 1800 <--> 3300 mV at 2800 mV 
[    2.488979] axp20_ldoio0: 1800 <--> 3300 mV at 2800 mV 
<6>input: axp20-supplyer as /devices/platform/sunxi-i2c.0/i2c-0/0-0034/axp20-supplyer.28/input/input1
[    2.503730] input: axp20-supplyer as /devices/platform/sunxi-i2c.0/i2c-0/0-0034/axp20-supplyer.28/input/input1
<4>axp20_ldo2: Failed to create debugfs directory
[    2.531948] axp20_ldo2: Failed to create debugfs directory
<6>device-mapper: uevent: version 1.0.3
[    2.542384] device-mapper: uevent: version 1.0.3
<6>device-mapper: ioctl: 4.22.0-ioctl (2011-10-19) initialised: dm-devel@redhat.com
[    2.554942] device-mapper: ioctl: 4.22.0-ioctl (2011-10-19) initialised: dm-devel@redhat.com
<6>device-mapper: multipath: version 1.3.2 loaded
[    2.568375] device-mapper: multipath: version 1.3.2 loaded
<6>device-mapper: multipath round-robin: version 1.0.0 loaded
[    2.579376] device-mapper: multipath round-robin: version 1.0.0 loaded
<6>device-mapper: multipath queue-length: version 0.1.0 loaded
[    2.591501] device-mapper: multipath queue-length: version 0.1.0 loaded
<6>device-mapper: multipath service-time: version 0.2.0 loaded
[    2.603694] device-mapper: multipath service-time: version 0.2.0 loaded
<6>cpuidle: using governor ladder
[    2.613909] cpuidle: using governor ladder
<6>cpuidle: using governor menu
[    2.620911] cpuidle: using governor menu
<6>[mmc-msg] sw_mci_init
[    2.627150] [mmc-msg] sw_mci_init
<6>[mmc-msg] MMC host used card: 0x9, boot card: 0x0, io_card 8
[    2.636219] [mmc-msg] MMC host used card: 0x9, boot card: 0x0, io_card 8
<6>[mmc-msg] sdc0 set round clock 400000, src 24000000
[    2.648179] [mmc-msg] sdc0 set round clock 400000, src 24000000
<6>[mmc-msg] sdc0 set ios: clk 0Hz bm OD pm OFF vdd 3.3V width 1 timing LEGACY(SDR12) dt B
[    2.662361] [mmc-msg] sdc0 set ios: clk 0Hz bm OD pm OFF vdd 3.3V width 1 timing LEGACY(SDR12) dt B
<6>[mmc-msg] sdc0 Probe: base:0xf0144000 irq:64 sg_cpu:f0146000(4fc00000) ret 0.
[    2.679613] [mmc-msg] sdc0 Probe: base:0xf0144000 irq:64 sg_cpu:f0146000(4fc00000) ret 0.
<6>[mmc-msg] sdc3 set round clock 400000, src 24000000
[    2.692763] [mmc-msg] sdc3 set round clock 400000, src 24000000
<6>[mmc-msg] sdc3 set ios: clk 0Hz bm OD pm OFF vdd 3.3V width 1 timing LEGACY(SDR12) dt B
[    2.706837] [mmc-msg] sdc3 set ios: clk 0Hz bm OD pm OFF vdd 3.3V width 1 timing LEGACY(SDR12) dt B
<6>[mmc-msg] sdc3 Probe: base:0xf0148000 irq:67 sg_cpu:f014a000(4fc01000) ret 0.
[    2.724053] [mmc-msg] sdc3 Probe: base:0xf0148000 irq:67 sg_cpu:f014a000(4fc01000) ret 0.
[mmc_pm]: failed to fetch sdio card configuration!
<6>sunxi_leds driver init
[    2.739157] sunxi_leds driver init
<7>Registered led device: blue:ph21:led1
<7>Registered led device: orange:ph20:led2
<7>Registered led device: white:ph11:led3
<7>Registered led device: green:ph07:led4
<6>ledtrig-cpu: registered to indicate activity on CPUs
[    2.763171] ledtrig-cpu: registered to indicate activity on CPUs
<6>input: Dell Dell USB Keyboard as /devices/platform/sw-ohci.1/usb3/3-1/3-1:1.0/input/input2
[    2.789570] input: Dell Dell USB Keyboard as /devices/platform/sw-ohci.1/usb3/3-1/3-1:1.0/input/input2
<6>generic-usb 0003:413C:2105.0001: input: USB HID v1.10 Keyboard [Dell Dell USB Keyboard] on usb-sw-ohci-1/input0
[    2.809316] generic-usb 0003:413C:2105.0001: input: USB HID v1.10 Keyboard [Dell Dell USB Keyboard] on usb-sw-ohci-1/input0
<6>usbcore: registered new interface driver usbhid
[    2.825133] usbcore: registered new interface driver usbhid
<6>usbhid: USB HID core driver
[    2.833520] usbhid: USB HID core driver
<6>ashmem: initialized
[    2.839985] ashmem: initialized
<6>logger: created 256K log 'log_main'
[    2.846774] logger: created 256K log 'log_main'
<6>logger: created 256K log 'log_events'
[    2.855078] logger: created 256K log 'log_events'
<6>logger: created 256K log 'log_radio'
[    2.863439] logger: created 256K log 'log_radio'
<6>logger: created 256K log 'log_system'
[    2.871842] logger: created 256K log 'log_system'
<6>IPv4 over IPv4 tunneling driver
[    2.881762] IPv4 over IPv4 tunneling driver
<6>TCP: bic registered
[    2.888714] TCP: bic registered
<6>TCP: cubic registered
[    2.894155] TCP: cubic registered
<6>TCP: westwood registered
[    2.899989] TCP: westwood registered
<6>TCP: highspeed registered
[    2.906183] TCP: highspeed registered
<6>TCP: hybla registered
[    2.912103] TCP: hybla registered
<6>TCP: htcp registered
[    2.917613] TCP: htcp registered
<6>TCP: vegas registered
[    2.923099] TCP: vegas registered
<6>TCP: veno registered
[    2.928597] TCP: veno registered
<6>TCP: scalable registered
[    2.934366] TCP: scalable registered
<6>TCP: lp registered
[    2.939940] TCP: lp registered
<6>TCP: yeah registered
[    2.945178] TCP: yeah registered
<6>TCP: illinois registered
[    2.950924] TCP: illinois registered
<6>Initializing XFRM netlink socket
[    2.957721] Initializing XFRM netlink socket
<6>NET: Registered protocol family 10
[    2.966048] NET: Registered protocol family 10
<6>NET: Registered protocol family 17
[    2.974841] NET: Registered protocol family 17
<6>NET: Registered protocol family 15
[    2.982701] NET: Registered protocol family 15
[mmc_pm]: No sdio card, please check your config !!
<5>Registering the dns_resolver key type
[    2.995502] Registering the dns_resolver key type
<6>VFP support v0.3: [    3.002159] VFP support v0.3: implementor 41 architecture 2 part 30 variant 7 rev 4
implementor 41 architecture 2 part 30 variant 7 rev 4
<5>Registering SWP/SWPB emulation handler
[    3.018385] Registering SWP/SWPB emulation handler
<4>axp20_buck2: Failed to create debugfs directory
[    3.027788] axp20_buck2: Failed to create debugfs directory
<6>[cpu_freq] INF:-------------------V-F Table-------------------
[    3.039537] [cpu_freq] INF:-------------------V-F Table-------------------
<6>[cpu_freq] INF:      voltage = 1450mv        frequency = 1008MHz
[    3.051463] [cpu_freq] INF:  voltage = 1450mv        frequency = 1008MHz
<6>[cpu_freq] INF:      voltage = 1425mv        frequency =  912MHz
[    3.062640] [cpu_freq] INF:  voltage = 1425mv        frequency =  912MHz
<6>[cpu_freq] INF:      voltage = 1350mv        frequency =  864MHz
[    3.073780] [cpu_freq] INF:  voltage = 1350mv        frequency =  864MHz
<6>[cpu_freq] INF:      voltage = 1250mv        frequency =  720MHz
[    3.084913] [cpu_freq] INF:  voltage = 1250mv        frequency =  720MHz
<6>[cpu_freq] INF:      voltage = 1150mv        frequency =  528MHz
[    3.096046] [cpu_freq] INF:  voltage = 1150mv        frequency =  528MHz
<6>[cpu_freq] INF:      voltage = 1100mv        frequency =  312MHz
[    3.107180] [cpu_freq] INF:  voltage = 1100mv        frequency =  312MHz
<6>[cpu_freq] INF:      voltage = 1050mv        frequency =  144MHz
[    3.118312] [cpu_freq] INF:  voltage = 1050mv        frequency =  144MHz
<6>[cpu_freq] INF:      voltage = 1000mv        frequency =    0MHz
[    3.129444] [cpu_freq] INF:  voltage = 1000mv        frequency =    0MHz
<6>[cpu_freq] INF:-----------------------------------------------
[    3.141363] [cpu_freq] INF:-----------------------------------------------
<6>[cpu_freq] INF:sunxi_cpufreq_initcall, get cpu frequency from sysconfig, max freq: 912MHz, min freq: 720MHz
[    3.157967] [cpu_freq] INF:sunxi_cpufreq_initcall, get cpu frequency from sysconfig, max freq: 912MHz, min freq: 720MHz
<6>registered taskstats version 1
[    3.172464] registered taskstats version 1
<6>Console: switching to colour frame buffer device 128x48
[    3.587083] Console: switching to colour frame buffer device 128x48
<6>I2C: i2c-1: HDMI I2C adapter
[    3.611789] I2C: i2c-1: HDMI I2C adapter
<4>axp20_buck3: incomplete constraints, leaving on
[    3.646568] axp20_buck3: incomplete constraints, leaving on
<6>[mmc-msg] mmc 0 detect change, present 1
[    3.661067] [mmc-msg] mmc 0 detect change, present 1
<4>axp20_buck2: incomplete constraints, leaving on
[    3.670812] axp20_buck2: incomplete constraints, leaving on
<4>axp20_ldo4: incomplete constraints, leaving on
[    3.681071] axp20_ldo4: incomplete constraints, leaving on
<4>axp20_ldo3: incomplete constraints, leaving on
[    3.691231] axp20_ldo3: incomplete constraints, leaving on
<4>axp20_ldo2: incomplete constraints, leaving on
[    3.701392] axp20_ldo2: incomplete constraints, leaving on
<4>axp20_ldo1: incomplete constraints, leaving on
[    3.711550] axp20_ldo1: incomplete constraints, leaving on
<6>console [netcon0] enabled
[    3.719959] console [netcon0] enabled
<6>netconsole: network logging started
[    3.727124] netconsole: network logging started
<6>sunxi-rtc sunxi-rtc: setting system clock to 2015-01-03 12:01:46 UTC (1420286506)
[    3.739181] sunxi-rtc sunxi-rtc: setting system clock to 2015-01-03 12:01:46 UTC (1420286506)
<6>ALSA device list:
[    3.749735] ALSA device list:
<6>  #0: sunxi-CODEC  Audio Codec
[    3.755761]   #0: sunxi-CODEC  Audio Codec
<6>Waiting for root device /dev/mmcblk0p2...
[    3.764395] Waiting for root device /dev/mmcblk0p2...
<6>[mmc-msg] sdc0 set ios: clk 0Hz bm PP pm UP vdd 3.3V width 1 timing LEGACY(SDR12) dt B
[    4.169277] [mmc-msg] sdc0 set ios: clk 0Hz bm PP pm UP vdd 3.3V width 1 timing LEGACY(SDR12) dt B
<6>[mmc-msg] sdc0 power on
[    4.180714] [mmc-msg] sdc0 power on
<6>[mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
[    4.209716] [mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
<6>[mmc-msg] sdc0 set round clock 400000, src 24000000
[    4.224012] [mmc-msg] sdc0 set round clock 400000, src 24000000
<3>[mmc-err] smc 0 err, cmd 52,  RTO
[    4.306992] [mmc-err] smc 0 err, cmd 52,  RTO
<3>[mmc-err] smc 0 err, cmd 52,  RTO
[    4.315504] [mmc-err] smc 0 err, cmd 52,  RTO
<6>[mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
[    4.328203] [mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
<6>[mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
[    4.348318] [mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
<3>[mmc-err] smc 0 err, cmd 5,  RTO
[    4.363060] [mmc-err] smc 0 err, cmd 5,  RTO
<3>[mmc-err] smc 0 err, cmd 5,  RTO
[    4.371364] [mmc-err] smc 0 err, cmd 5,  RTO
<3>[mmc-err] smc 0 err, cmd 5,  RTO
[    4.379710] [mmc-err] smc 0 err, cmd 5,  RTO
<3>[mmc-err] smc 0 err, cmd 5,  RTO
[    4.388020] [mmc-err] smc 0 err, cmd 5,  RTO
<6>[mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
[    4.401274] [mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
<6>[mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
[    4.419013] [mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
<6>[mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
[    4.439153] [mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing LEGACY(SDR12) dt B
<6>[mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing SD-HS(SDR25) dt B
[    4.478797] [mmc-msg] sdc0 set ios: clk 400000Hz bm PP pm ON vdd 3.3V width 1 timing SD-HS(SDR25) dt B
<6>[mmc-msg] sdc0 set ios: clk 50000000Hz bm PP pm ON vdd 3.3V width 1 timing SD-HS(SDR25) dt B
[    4.496533] [mmc-msg] sdc0 set ios: clk 50000000Hz bm PP pm ON vdd 3.3V width 1 timing SD-HS(SDR25) dt B
<6>[mmc-msg] sdc0 set round clock 42857143, src 600000000
[    4.511155] [mmc-msg] sdc0 set round clock 42857143, src 600000000
<6>[mmc-msg] sdc0 set ios: clk 50000000Hz bm PP pm ON vdd 3.3V width 4 timing SD-HS(SDR25) dt B
[    4.579847] [mmc-msg] sdc0 set ios: clk 50000000Hz bm PP pm ON vdd 3.3V width 4 timing SD-HS(SDR25) dt B
<6>mmc0: new high speed SD card at address 0002
[    4.593605] mmc0: new high speed SD card at address 0002
<6>mmcblk0: mmc0:0002 SD    1.87 GiB 
[    4.602829] mmcblk0: mmc0:0002 SD    1.87 GiB 
<6> mmcblk0: p1 p2
[    4.610451]  mmcblk0: p1 p2
<3>EXT3-fs (mmcblk0p2): error: couldn't mount because of unsupported optional features (240)
[    4.650404] EXT3-fs (mmcblk0p2): error: couldn't mount because of unsupported optional features (240)
<3>EXT2-fs (mmcblk0p2): error: couldn't mount because of unsupported optional features (240)
[    4.680848] EXT2-fs (mmcblk0p2): error: couldn't mount because of unsupported optional features (240)
<6>EXT4-fs (mmcblk0p2): mounted filesystem with ordered data mode. Opts: (null)
[    4.728075] EXT4-fs (mmcblk0p2): mounted filesystem with ordered data mode. Opts: (null)
<6>VFS: Mounted root (ext4 filesystem) on device 179:2.
[    4.741231] VFS: Mounted root (ext4 filesystem) on device 179:2.
<6>devtmpfs: mounted
[    4.750311] devtmpfs: mounted
<6>Freeing init memory: 212K
[    4.756184] Freeing init memory: 212K
<6>Write protecting the kernel text section c0008000 - c07c2000
[    4.765496] Write protecting the kernel text section c0008000 - c07c2000
<4>init: plymouth-upstart-bridge main process (68) terminated with status 1
[    5.205488] init: plymouth-upstart-bridge main process (68) terminated with status 1
<4>init: plymouth-upstart-bridge main process ended, respawning
[    5.219143] init: plymouth-upstart-bridge main process ended, respawning
<4>init: ureadahead main process (70) terminated with status 5
[    5.235523] init: ureadahead main process (70) terminated with status 5
<30>systemd-udevd[197]: starting version 208
<4>init: gator-daemon main process (210) terminated with status 1
 * Stopping Send an event to indicate plymouth is up[ OK ]
 * Starting Mount filesystems on boot[ OK ]
 * Starting Signal sysvinit that the rootfs is mounted[ OK ]
 * Starting Fix-up sensitive /proc filesystem entries[ OK ]
 * Starting Populate /dev filesystem[ OK ]
 * Stopping Fix-up sensitive /proc filesystem entries[ OK ]
 * Stopping Populate /dev filesystem[ OK ]
 * Starting Fix-up /sys/kernel/debug filesystem[ OK ]
 * Starting Clean /tmp directory[ OK ]
 * Stopping Fix-up /sys/kernel/debug filesystem[ OK ]
 * Starting Populate and link to /run filesystem[ OK ]
 * Stopping Populate and link to /run filesystem[ OK ]
 * Stopping Clean /tmp directory[ OK ]
 * Stopping Track if upstart is running in a container[ OK ]
 * Starting Initialize or finalize resolvconf[ OK ]
 * Starting set console keymap[ OK ]
 * Starting Signal sysvinit that virtual filesystems are mounted[ OK ]
 * Starting Signal sysvinit that virtual filesystems are mounted[ OK ]
 * Starting Bridge udev events into upstart[ OK ]
 * Stopping set console keymap[ OK ]
 * Starting Signal sysvinit that local filesystems are mounted[ OK ]
 * Starting Signal sysvinit that remote filesystems are mounted[ OK ]
 * Stopping Mount filesystems on boot[ OK ]
 * Starting device node and kernel event manager[ OK ]
 * Starting flush early job output to logs[ OK ]
 * Starting Load gator driver module and launch gator daemon[ OK ]
 * Starting Load gator driver module and launch gator daemon[fail]
 * Starting load modules from /etc/modules[ OK ]
 * Starting log initial device creation[ OK ]
 * Starting cold plug devices[ OK ]
 * Stopping flush early job output to logs[ OK ]
 * Stopping load modules from /etc/modules[ OK ]
 * Starting load fallback graphics devices[ OK ]
 * Starting set console font[ OK ]
 * Stopping load fallback graphics devices[ OK ]
 * Starting system logging daemon[ OK ]
 * Stopping set console font[ OK ]
 * Starting userspace bootsplash[ OK ]
 * Stopping cold plug devices[ OK ]
 * Stopping log initial device creation[ OK ]
 * Stopping userspace bootsplash[ OK ]
 * Starting Send an event to indicate plymouth is up[ OK ]
 * Starting configure network device security[ OK ]
 * Starting save udev log and update rules[ OK ]
 * Stopping Send an event to indicate plymouth is up[ OK ]
 * Starting configure network device security[ OK ]
 * Starting configure network device security[ OK ]
 * Starting configure network device[ OK ]
 * Starting Bridge file events into upstart[ OK ]
 * Starting configure network device[ OK ]
 * Starting Mount network filesystems[ OK ]
 * Starting Failsafe Boot Delay[ OK ]
 * Stopping Mount network filesystems[ OK ]
<4>init: failsafe main process (353) killed by TERM signal
 * Stopping Failsafe Boot Delay[ OK ]
 * Starting System V initialisation compatibility[ OK ]
 * Starting configure virtual network devices[ OK ]
 * Starting Bridge socket events into upstart[ OK ]
[ OK ]ting sensors limits        
 * Stopping System V initialisation compatibility[ OK ]
 * Starting System V runlevel compatibility[ OK ]
 * Starting save kernel messages[ OK ]
 * Starting regular background program processing daemon[ OK ]
 * Stopping Restore Sound Card State[ OK ]
 * Starting OpenSSH server[ OK ]
 * Stopping save kernel messages[ OK ]
 * Stopping Restore Sound Card State[ OK ]
[ OK ]ding cpufreq kernel modules...        
 * Not running within Xen or no compatible utils
[ OK ]0...         * CPU1...         * CPUFreq Utilities: Setting ondemand CPUFreq governor...        
 * No usable Xen toolstack selected
 * Stopping System V runlevel compatibility[ OK ]

Last login: Sat Jan  3 11:44:34 UTC 2015 on tty1
Welcome to Ubuntu 14.10 (GNU/Linux 3.4.103+ armv7l)

 * Documentation:  http://www.linaro.org
<4>init: plymouth-upstart-bridge main process ended, respawning
root@linaro-developer:~# <4>init: tty1 main process (759) killed by TERM signal
</xmp>
VGA显示器内容如下
![VGA](/images/cubietruck-bootable-sdcard/VGA.jpg)