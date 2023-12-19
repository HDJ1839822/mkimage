# mkimage介绍以及使用

![image-20231219224427585](https://newbie-typora.oss-cn-shenzhen.aliyuncs.com/TyporaJPG/image-20231219224427585.png)

## 基本概况

mkimage是uboot下的一个工具，用于在编译好的uboot.bin文件前附加信息，crc以及文件大小和入口地址……   默认附加信息占64个字节， 这里使用该工具给app的bin文件加上头部信息，配合bootloader加载app时候可先识别这一段信息作为认证以及校验

这里介绍win下的使用方法

参考教程：

韦东山从0写bootloaer  第24节、25节有介绍该工具的使用，由于版本差异，不同的版本的工具该附加的信息可能不一样，以当下使用的工具为准

https://video.100ask.net/p/t_pc/course_pc_detail/video/v_62c7a9a7e4b050af23994763?product_id=p_62b98507e4b00a4f371ef52c&content_app_id=&type=6

## 使用方法

- 该工具无法直接在cmd命令窗口使用，可使用git bash  here运行

```
./mkimage.exe
```

![image-20231219205825052](https://newbie-typora.oss-cn-shenzhen.aliyuncs.com/TyporaJPG/image-20231219205825052.png)

解释：

- `-l`：显示镜像文件头信息。
- `-A arch`：设置架构（architecture）为 'arch'。
- `-O os`：设置操作系统（operating system）为 'os'。
- `-T type`：设置镜像类型（image type）为 'type'。
- `-C comp`：设置压缩类型（compression type）为 'comp'。
- `-a addr`：设置加载地址（load address）为 'addr'（十六进制）。
- `-e ep`：设置入口点（entry point）为 'ep'（十六进制）。
- `-n name`：设置镜像名称（image name）为 'name'。
- `-d data_file[:data_file...]`：使用来自 'data_file' 的镜像数据。可以指定多个文件。
- `-D dtc_options`：设置设备树编译器（device tree compiler）的选项。
- `-f fit-image.its|-f auto|-F`：指定 FIT 源文件，可以是 `.its` 文件，`auto` 表示自动生成，`-F` 表示从 stdin 读取。
- `-b <dtb>`：指定设备树二进制文件（DTB）。可以指定多个。
- `-i <ramdisk.cpio.gz>`：指定 RAM Disk 文件。
- `-V`：打印版本信息并退出。
- 使用 `-x` 选项可以设置 XIP（execute in place）。

先使用该工具生成一个附加头文件的文件

```
./mkimage.exe -n "application_name" -A arm -a 0x240008f8 -e 0x900002b1  -d app.bin app_head.bin
```

![image-20231219212044327](https://newbie-typora.oss-cn-shenzhen.aliyuncs.com/TyporaJPG/image-20231219212044327.png)

附加的64字节头文件信息如下

```c
#define IH_NMLEN	32	/* Image Name Length		*/
typedef struct image_header {
	__be32		ih_magic;	/* Image Header Magic Number	*/
	__be32		ih_hcrc;	/* Image Header CRC Checksum	*/
	__be32		ih_time;	/* Image Creation Timestamp	*/
	__be32		ih_size;	/* Image Data Size		*/
	__be32		ih_load;	/* Data	 Load  Address		*/
	__be32		ih_ep;		/* Entry Point Address		*/
	__be32		ih_dcrc;	/* Image Data CRC Checksum	*/
	uint8_t		ih_os;		/* Operating System		*/
	uint8_t		ih_arch;	/* CPU architecture		*/
	uint8_t		ih_type;	/* Image Type			*/
	uint8_t		ih_comp;	/* Compression Type		*/
	uint8_t		ih_name[IH_NMLEN];	/* Image Name		*/
} image_header_t;
```



## 附加前后文件对比

原bin文件（小端）

![image-20231219203430559](https://newbie-typora.oss-cn-shenzhen.aliyuncs.com/TyporaJPG/image-20231219203430559.png)

附加后bin文件（附加64字节为大端）

![image-20231219212217853](https://newbie-typora.oss-cn-shenzhen.aliyuncs.com/TyporaJPG/image-20231219212217853.png)

附加信息如下（注：附加信息为大端）

```c
	__be32		ih_magic;	0x27051956         /* Image Header Magic Number	*/     
	__be32		ih_hcrc;	0X0051CE40         /* Image Header CRC Checksum	*/
	__be32		ih_time;	0X6581989C         /* Image Creation Timestamp	*/
	__be32		ih_size;	0X00003238         /* Image Data Size		*/
	__be32		ih_load;	0X240008F8         /* Data	Load  Address	*/
	__be32		ih_ep;		0X900002B1         /* Entry Point Address	*/
	__be32		ih_dcrc;	0XA82B3BB6         /* Image Data CRC Checksum	*/
	uint8_t		ih_os;		0X05         /* Operating System		*/
	uint8_t		ih_arch;	0X02         /* CPU architecture		*/
	uint8_t		ih_type;	0X02         /* Image Type			*/
	uint8_t		ih_comp;	0X01         /* Compression Type		*/
	uint8_t		ih_name[IH_NMLEN];       /* Image Name		max 32 char*/  
```

这里我们注重以下几个信息：

- ih_dcrc   数据crc校验（原bin文件校验，使用crc32）
- ih_load  加载地址（sp地址）
- ih_ep    入口地址（pc地址）
- ih_size  文件大小 （原bin文件大小）
- ih_name  名字 （最长32个字符） 如果需要增加字符需要更改源码  #define IH_NMLEN	32	/* Image Name Length		*/

使用hex查看器分析，得出mkimage使用的是crc32校验，  

![image-20231219220236071](https://newbie-typora.oss-cn-shenzhen.aliyuncs.com/TyporaJPG/image-20231219220236071.png)

-   如果要比较高的安全性，头部ih_hcrc  也可加上



## 扩展

可以使用name区域的32个字节来确定app的名字以及版本号

- 由于名字的长度可变，存储的数据为版本号+名字  较为合理
- 如下版本号占10个字节(算上空格)，我们将app的名字控制在22个字符以内即可

```
./mkimage.exe -n "V02.10.88 application_name" -A arm -a 0x240008f8 -e 0x900002b1  -d app.bin app_head.bin
```

- V02.10.88 版本解释：  02：主版本号  10：次版本号  88：修订版本号(补丁)

![image-20231219222245474](https://newbie-typora.oss-cn-shenzhen.aliyuncs.com/TyporaJPG/image-20231219222245474.png)

由此我们只要在软件上解析头部的64个字节中的一部分即可。















