# 软件CRC32计算公式

## 计算公式

```c
#define CRC32_POLYNOMIAL 0xEDB88320u
//传入参数：                   数据首地址     数据大小
uint32_t calculate_crc32(const void *data, size_t size) {
    const uint8_t *byteData = (const uint8_t *)data;
    uint32_t crc = 0xFFFFFFFFu;

    for (size_t i = 0; i < size; ++i) {
        crc ^= byteData[i];
        for (int j = 0; j < 8; ++j) {
            crc = (crc >> 1) ^ ((crc & 1) ? CRC32_POLYNOMIAL : 0);
        }
    }

    return ~crc;
}

```

## 测试如下：

使用14k大小数据测试  

![image-20231220213423701](https://newbie-typora.oss-cn-shenzhen.aliyuncs.com/TyporaJPG/image-20231220213423701.png)



![image-20231220213814868](https://newbie-typora.oss-cn-shenzhen.aliyuncs.com/TyporaJPG/image-20231220213814868.png)

打印输出如下：

![image-20231220214046941](https://newbie-typora.oss-cn-shenzhen.aliyuncs.com/TyporaJPG/image-20231220214046941.png)

- 计算结果一致