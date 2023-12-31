# Linux minimal image for raspberry pi 3 B

In this project we will go through step by step tutorial to generate our custom linux image built for raspberry pi 3 model B. We will cover three methods : 
* manual method 
* using buildroot
* using yocto 



## Linux system components

Our minimal linux image consists of four essential components in order to work and run your applications. The image below showes 3 of these components interacting together during booting and running conditions.  

![linux components](https://vocal.com/wp-content/uploads/2021/10/Linux-System-Components.png)

### Boot Loader
Booting sequence varries from device to another but they all share the same concept. Booting sequence uses multi stage boot loader as the first steps after device power on. Multi stages boot loader means that you may have first boot loader which initializes some basic HW components like memory drivers and then loads the second boot loader which actually loads linux kernel. Some devices may also have more than two steps each step is suitable for particular device state and each one loads the following one until they reach the kernel. As we are talking about raspberry pi here is the booting sequence of raspberry pi :

![rpi boot sequence](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRtreHICSPMRMclNKxiUyD5XeOA1nE6DnU8kw&usqp=CAU)

In raspberry pi GPU turns on first before CPU so there’re some steps using files provided by the manufacturer like bootcode.bin and start.elf and they are not related to CPU. Until we reach the last step where start.elf reads config.txt which contains system configuration parameters and loads the kernel. This file is editable you can edit it to hack the sequence and put your boot loader and then your custom image starts from here !

### linux kernel
Kernel is the core of the operating system which manages everything for you and abstracts user space from HW level. It interfaces with hardware , manages resources and provides APIs used by user space to keep it abstracted.

![linux kernel](https://www.engineersgarage.com/wp-content/uploads/2016/07/ArticleImage-12104-1.png)

The image above shows the important rule of kernel in user space abstraction. User space applications run in low privilege level so they can not access hardware directly. Fortunately there is an interface between these applications and kernel level which is C libraries which translate  user level functions into kernel system call which is something like interrupt or trap generated to enter kernel space and access specific resources defined by kernel.

### root filesystem
In linux everything is a file , these files are organized in root filesystem which contains userspace applications , devices , processes , etc.The image below shows basic root filesystem hirarchy:

![linux rootfs](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcToIUFOmx85-uEaei6qho7xxS7Z5cOViwLlRg&usqp=CAU)

* /bin:	Programs essential for all users
* /dev:	Device nodes and other special files
* /etc:	System configuration files
* /lib:	Essential shared libraries,	for	example, those that	make up	the	C-library
* /proc: The proc filesystem
* /sbin: Programs essential	to the system administrator
* /sys: The sysfs filesystem
* /tmp:	A place to put temporary or volatile files
* /usr:	Additional programs, libraries,	and	system administrator utilities,	in the directories /usr/bin, /usr/lib and /usr/sbin, respectively
* /var:	A hierarchy	of files and directories that may be modified at runtime, for example, log  messages, some of which must be retained after boot

We are going to create minimal root filesystem which will give us init process , basic shell and some basic apps.

### Toolchain
This component should be the first one , it is not visible in the image above which shows linux components but it is the creator of all the components we have mentioned. Each target has its own toolchain used to compile applications and make them ready to run on this target. This image shows the components of the toolchain:

![toolchain components](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQTTbvGL_Pzxv2NwQvnVWkOh31cm20aBF997Q&usqp=CAU)

* Binutils:	A set of binary	utilities including	the	assembler and the linker. It is available at http://www.gnu.org/software/binutils.
* GNU	Compiler	Collection	(GCC):	These are the compilers	for	C and other	languages	which,	depending	on	the version	of	GCC,	include	C++, Objective-C, Objective-C++, Java, Fortran, Ada,	and	Go.	They	all	use	a common backend which produces assembler code, which is	fed	to the GNU assembler. It is	available at http://gcc.gnu.org/.
* C	library:	A	standardized	application	program	interface	(API)	based	on
the	POSIX	specification,	which	is	the	main	interface	to	the	operating
system	kernel	for	applications.

![cross compiler](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ4TPEE8xuhZduAW-v92WF_0bCJgNicznb_sg&usqp=CAU)

In our case we use cross compiler which means the host device and the target don't have the same architecture for example we will use linux os running on x86 computer to develope our application then we will compile the application and move the resulting executable to run on raspberry pi which have ARM architecture. 




Now we know the 4 components of the system we are going to create so let's start creating them by different methods.
