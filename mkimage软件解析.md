## mkimage软件解析

前面了解了mkimage 64字节的数据组成，现在进行解析

数据格式如下：

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

我们使用的是STM32，在app中已经附加了这64字节的数据，但这里的数据是大端格式，我们要转换成小段格式处理

## 伪代码

```c
/*大端字节序转小端字节序*/
uint32_t my_ntohl(uint32_t netlong) {
    return ((netlong & 0xFF000000) >> 24) |
           ((netlong & 0x00FF0000) >> 8) |
           ((netlong & 0x0000FF00) << 8) |
           ((netlong & 0x000000FF) << 24);
}



/*打印头部信息*/
void printf_app_head(image_header_t *pimage_header_t , char *name){
	
    if (strcmp(name, APP1_FILE_NAME) == 0) {
        printf("| app1 head information     \r\n");
    } else if (strcmp(name, APP2_FILE_NAME) == 0) {
        printf("| app2 head information     \r\n");
    }

	# if(0)   //用于调试
   printf("ih_magic in hexadecimal: 0x%x\r\n", my_ntohl(pimage_header_t->ih_magic));
	 printf("ih_hcrc in hexadecimal: 0x%x\r\n", my_ntohl(pimage_header_t->ih_hcrc));
	 printf("ih_time in hexadecimal: 0x%x\r\n", my_ntohl(pimage_header_t->ih_time));
	 printf("ih_size in hexadecimal: 0x%x\r\n", my_ntohl(pimage_header_t->ih_size));
	 printf("ih_load in hexadecimal: 0x%x\r\n", my_ntohl(pimage_header_t->ih_load));
	 printf("ih_ep in hexadecimal: 0x%x\r\n", my_ntohl(pimage_header_t->ih_ep));
	 printf("ih_dcrc in hexadecimal: 0x%x\r\n", my_ntohl(pimage_header_t->ih_dcrc));
	
	 printf("ih_os in hexadecimal: 0x%x\r\n",   pimage_header_t->ih_os);
	 printf("ih_arch in hexadecimal: 0x%x\r\n", pimage_header_t->ih_arch);
	 printf("ih_type in hexadecimal: 0x%x\r\n", pimage_header_t->ih_type);
	 printf("ih_comp in hexadecimal: 0x%x\r\n", pimage_header_t->ih_comp);
	 printf("| ih_name is: ");
   for (int i = 0; i < IH_NMLEN; i++) 
	 printf("%c", (unsigned char)pimage_header_t->ih_name[i]);  printf("\r\n");
	#else
	 printf("| ih_name   |");
   for (int i = 0; i < IH_NMLEN; i++) 
	 printf("%c", (unsigned char)pimage_header_t->ih_name[i]);  printf("\r\n");
	
	 printf("| ih_size   |  %u Byte    \r\n", my_ntohl(pimage_header_t->ih_size));
	 printf("| ih_load   |  0x%x       \r\n", my_ntohl(pimage_header_t->ih_load));
	 printf("| ih_ep     |  0x%x       \r\n", my_ntohl(pimage_header_t->ih_ep));
	 printf("| ih_hcrc   |  0x%x       \r\n", my_ntohl(pimage_header_t->ih_hcrc)); //头部crc
	 printf("| ih_dcrc   |  0x%x       \r\n", my_ntohl(pimage_header_t->ih_dcrc)); //app crc32
	 printf("\r\n");
	#endif
 
}
	


void get_app_head(image_header_t *app_image_header_t, char *app_name) {
    int bytesRead = 0;
    //读取头部64字节
    bytesRead = readFileContent(app_name, app_image_header_t, sizeof(image_header_t), 0);

    if (bytesRead = sizeof(image_header_t)) {
	    //打印该头部结构体信息
         printf_app_head(app_image_header_t, app_name);
			
    }
}


//使用如下
image_header_t app1_image_header_t;
image_header_t app2_image_header_t;

get_app_head(&app1_image_header_t,APP1_FILE_NAME);
get_app_head(&app2_image_header_t,APP2_FILE_NAME);
```

