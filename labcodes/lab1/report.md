#Lab1 Report
-----
## Ex1
1. 操作系统镜像文件ucore.img是如何一步一步生成的？（需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果）
	> 
	```
	前言：各指令各参数意义在最后，正文注重逻辑梳理。

	生成ucore.img的直接指令如下
	$(UCOREIMG): $(kernel) $(bootblock)
		$(V)dd if=/dev/zero of=$@ count=10000
		$(V)dd if=$(bootblock) of=$@ conv=notrunc
		$(V)dd if=$(kernel) of=$@ seek=1 conv=notrunc
	可看出两个依赖为bootblock和kernel。

	生成bootblock
		$(bootblock): $(call toobj,$(bootfiles)) | $(call totarget,sign)
			@echo + ld $@
			$(V)$(LD) $(LDFLAGS) -N -e start -Ttext 0x7C00 $^ -o $(call toobj,bootblock)
			@$(OBJDUMP) -S $(call objfile,bootblock) > $(call asmfile,bootblock)
			@$(OBJCOPY) -S -O binary $(call objfile,bootblock) $(call outfile,bootblock)
			@$(call totarget,sign) $(call outfile,bootblock) $(bootblock)
		其中bootfiles为/boot下的三个文件，sign为tools/sign.c的目标文件。

		生成obj/boot/bootasm.o和obj/boot/bootmain.o
			bootfiles = $(call listf_cc,boot)
			$(foreach f,$(bootfiles),$(call cc_compile,$(f),$(CC),$(CFLAGS) -Os -nostdinc))
			生成bootasm.o
				gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o
				需要boot/bootasm.S
			生成bootmain.o
				gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootmain.c -o obj/boot/bootmain.o
				需要bootmain.c
		
		生成sign
			gcc -Itools/ -g -Wall -O2 -c tools/sign.c -o obj/sign/tools/sign.o
			gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign
			需要tools/sign.c
		
		ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
		连接生成bootblock.o。
		objdump -S obj/bootblock.o > obj/bootblock.asm
		反汇编bootblock.o到bootblock.asm。
		objcopy -S -O binary obj/bootblock.o obj/bootblock.out
		拷贝二进制代码到bootblock.out。
		bin/sign obj/bootblock.out bin/bootblock
		使用刚生成的sign，构建硬盘主引导扇区。

	生成kernel
		$(call add_files_cc,$(call listf_cc,$(KSRCDIR)),kernel,$(KCFLAGS))
		$(call add_files_cc,$(call listf_cc,$(LIBDIR)),libs,)
		KOBJS	= $(call read_packet,kernel libs)
		$(kernel): tools/kernel.ld
		$(kernel): $(KOBJS)
			@echo + ld $@
			$(V)$(LD) $(LDFLAGS) -T tools/kernel.ld -o $@ $(KOBJS)
			@$(OBJDUMP) -S $@ > $(call asmfile,kernel)
			@$(OBJDUMP) -t $@ | $(SED) '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' > $(call symfile,kernel)
		
		KOBJS为kern和lib下各.c文件编译出来的.o文件
			init.o readline.o stdio.o kdebug.o kmonitor.o panic.o clock.o console.o intr.o picirq.o trap.o trapentry.o vectors.o pmm.o  printfmt.o string.o
		生成命令举例
			gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o
		然后通过ld连接各文件得到kernel.o

	生成含10000个0填充512bytes大小的块文件
	dd if=/dev/zero of=bin/ucore.img count=10000
	不截断把bootblock写入第一个块
	dd if=bin/bootblock of=bin/ucore.img conv=notrunc
	不截断把kernel写入第二个块
	dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc
	最后得到ucore.img

	gcc各参数意义
		-I<dir>  添加搜索头文件的路径
		-m32  生成适用于32位环境的代码
		-ggdb 生成gdb调试信息
		-gstabs 生成stabs调试信息
		-fno-builtin  除非用__builtin_前缀，否则不进行builtin函数的优化
		-nostdinc 不使用标准库
		-fno-stack-protector 不检测缓冲区溢出
		-Os 控制代码大小
	ld各参数意义
		-m elf_i385 模拟i386连接器
		-nostdlib 不使用标准库
		-N 代码段和数据段均可读写
		-e 指定入口
		-Ttext 指定代码段开始位置
	objdump各参数意义
		-S 输出C源代码和反汇编出来的指令对照的格式
	objcopy各参数意义
		-S  移除所有符号和重定位信息
		-O <bfdname>  指定输出格式
	```
1. 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？
	> * 观察sign.c，要求读入文件长度为510字节，写入0x55和0xAA后变为512字节。
	> * 长度为512字节
	> * 倒数第二字节为0x55
	> * 最后一个字节为0xAA

## Ex2
1. 从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行。

	> * 删去tools/gdbinit最后一行，避免qemu在gdb连接后马上启动。
	> * 修改makefile。不希望X11来捣乱，qemu启动时置后台，gdb结束即kill掉进程。
	```
	debug-nox: $(UCOREIMG)
		$(V)$(QEMU) -S -s -d in_asm -D $(BINDIR)/q.log -serial mon:stdio -hda $< -nographic &
		$(V)sleep 2
		$(V)$(TERMINAL) -e "gdb -q -x tools/gdbinit"
		$(V)$(TERMINAL) -e "pkill qemu-system-i38"
	```
	> * 运行如下命令，si即可单步调试。
	```
	make debug-nox
	```
