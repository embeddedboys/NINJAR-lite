Linux i2c-dev驱动 用户空间操作示例
---------------------------------

测试芯片为at24c16


打开i2c总线节点
```c
file = open("/dev/i2c-0", O_RDWR);
if(file < 0){
	fprintf(stderr, "failed to open i2c bus\n");
	return -1;
}
```

设置器件地址
> I2C_SLAVE_FORCE  表示即使该地址的i2c设备已经有驱动程序，也强制其可用
```c
ret = ioctl(file, force?I2C_SLAVE_FORCE:I2C_SLAVE, 0x50);
if(ret < 0){
	fprintf(stderr, "failed to set slave addr\n");
	return -1;
}
```

### EEPROM AT24C02

以EEPROM AT24C02字节写为例， 流程如下
`S -> Daddr -> Waddr -> data -> T`
起始 -> 器件地址 -> 目标地址 -> 数据 -> 结束
符合SMBus如下写序列
![image](https://img2022.cnblogs.com/blog/2605173/202202/2605173-20220228093544279-1578358120.png)

使用`I2C_SMBUS_WRITE`写2字节，详见于`linux/i2c.h`中
```c
__s32 smbus_access(int file, char read_write, __u8 Waddr, int size, union i2c_smbus_data *data)
{
    struct i2c_smbus_ioctl_data msgs;

    msgs.read_write = read_write;
    msgs.command = Waddr;
    msgs.size = size;
    msgs.data = data;

    if(ioctl(file, I2C_SMBUS, &msgs) < 0){
        perror("error, failed to access smbus");
        return -errno;
    }
	
	return 0;
}

__s32 write_byte_data(int file, __u32 Waddr, __u32 value)
{
    union i2c_smbus_data data;
    data.byte = value;
    return smbus_access(file,I2C_SMBUS_WRITE, Waddr, I2C_SMBUS_BYTE_DATA, &data);
}
```

at24cxx每次写后需要max 10ms时间处理内部写循环
```c
int waiting_write_cycle()
{
	int ret;
	struct timespec ts;
	/* waiting for at24cxx internal write cycle. 10ms max */
	ts.tv_sec =0;
	ts.tv_nsec = 10 * 1000 * 1000;
	ret = nanosleep(&ts, NULL);
	if(ret < 0){
		fprintf(stderr, "cannot sleep.\n");
		perror("ERRNO: ");
		return -errno;
	}
	
	return 0;
}
```

### BH1750 光强传感器

以 `BH1750` 光强传感器为例

写命令
```c
int bh1750_write_cmd(int fd, __u8 cmd)
{
    return smbus_access(fd, I2C_SMBUS_WRITE, cmd, I2C_SMBUS_BYTE, NULL);
}
```

读双字
```c
int bh1750_read_word(int fd)
{
    union i2c_smbus_data data;
    struct i2c_smbus_ioctl_data msg;

    msg.read_write = I2C_SMBUS_READ;
    msg.size = I2C_SMBUS_WORD_DATA;
    msg.data = &data;

    if(ioctl(fd, I2C_SMBUS, &msg) < 0){
        perror("error, failed to access smbus");
        return -errno;
    }
    printf("raw data: %d\n", data.word);

    return data.word;
}
```

低分辨率模式光强转换
参考手册如下部分
![image](https://img2022.cnblogs.com/blog/2605173/202205/2605173-20220529152442996-345380276.png)

可知，单次低分辨率模式下，直接将双字除以1.2即可
```c
raw_data = bh1750_read_word(fd);

/* convert raw data to lux */
lux = raw_data / 1.2;
printf("lux: %f\n", lux);
```
![image](https://img2022.cnblogs.com/blog/2605173/202205/2605173-20220529153136770-1384660734.png)


包含如下头文件
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/ioctl.h>

#include <linux/types.h>
#include <linux/i2c.h>
#include <linux/i2c-dev.h>

#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <time.h>
```