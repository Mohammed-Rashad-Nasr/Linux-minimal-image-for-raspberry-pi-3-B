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
