# 과제 제출


## [ 커널 모듈(드라이버)]

1. 커널모듈(드라이버) 작성
```c
pi@raspberrypi:~/development/drivers/sample_driver $ nano sample_driver.c
```
```c
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/io.h>
#include <linux/delay.h>

#define SAMPLE_MAJOR 220
#define SAMPLE_MINOR 0
#define DEVICE_NAME "sample"

volatile unsigned int* map;

static int sample_open(struct inode *inodep, struct file *filep)
{
    printk(KERN_INFO "call sample_open\n");
    return 0;
}

static ssize_t sample_read(struct file *filep, char *buffer, size_t len, loff_t *offset)
{
    printk(KERN_INFO "call sample_read\n");
    return len;
}
static ssize_t sample_write(struct file *filep, const char *buffer, size_t len, loff_t *offset)
{

	int i = 0;
	for (i=0; i < len; i++)
	{
		printk(KERN_INFO "ch:%c,", buffer[i]);
	}
    printk(KERN_INFO "call sample_write\n");
    return len;
}

static int sample_release(struct inode *inodep, struct file *filep)
{                                                                                                                                       

    printk(KERN_INFO "call sample_write\n");
    return 0;
}

static struct file_operations fops =
{
        .owner = THIS_MODULE,
        .open = sample_open,
        .read = sample_read,
        .write = sample_write,
        .release = sample_release,
};

static int __init sample_init(void)
{
    register_chrdev(SAMPLE_MAJOR, DEVICE_NAME, &fops);
    
    map = (volatile unsigned int*) ioremap(0x3F200000, 180);
	
	// map : 0x7E200000(GPFSEL0) 6번 20-18 001	// 000001000 00000000 00000000 00000000
    *map = 0x00040000; 					// 000000000 00000100 00000000 00000000
	// 0x7E200000 = 0x00040000;
	
	// map+7 : 0x7E20001c(GPSET0)
    *(map + 7) = 0x00000040;					// 00000000 00000000 00000000 01000000
	// 0x7E20001c = 0x00000200;  0x7E200000 + 28(4byte * 7) = 0x00000200, 28을 16진수로 표현하면 1c
	
    msleep(2000);
	
	// map+10 : 0x7E200028(GPCLR0)
    *(map + 10) = 0x00000040;					// 00000000 00000000 00000000 01000000		

    printk(KERN_INFO "map : %p\n", map);
    printk(KERN_INFO "call sample_init\n");

    return 0;
}

static void __exit sample_exit(void)
{
    unregister_chrdev(SAMPLE_MAJOR, DEVICE_NAME);
    printk(KERN_INFO "call sample_exit\n");
}

module_init(sample_init);
module_exit(sample_exit);

MODULE_LICENSE("GPL");
```
2. 커널모듈(드라이버) Makefile 작성
```
pi@raspberrypi:~/development/drivers/sample_driver $ nano Makefile
```
```
obj-m+=sample_driver.o

all:
    make -C /lib/modules/$(shell uname -r)/build/ M=$(PWD) modules
clean:
    make -C /lib/modules/$(shell uname -r)/build/ M=$(PWD) clean
```
3. 커널모듈(드라이버) 빌드(컴파일)
```
pi@raspberrypi:~/development/drivers/sample_driver $ make
```
4. 커널올리기
```
pi@raspberrypi:~/development/drivers/sample_driver $ sudo insmod sample_driver.ko
```
5. 커널 로그 확인하기
```
pi@raspberrypi:~/development/drivers/sample_driver $ tail /var/log/kern.log
```


## [디바이스 파일]

1. 디바이스 파일 생성
```
pi@raspberrypi:~/development/drivers/sample_driver $ sudo mknod /dev/sample c 220 0 
pi@raspberrypi:~/development/drivers/sample_driver $ ls -l /dev/sample
```

2. 디바이스 파일 권한 변경
```
pi@raspberrypi:~/development/drivers/sample_driver $ sudo chmod 666 /dev/sample
pi@raspberrypi:~/development/drivers/sample_driver $ ls -l /dev/sample
```


## [테스트 어플 작성]

1. sample_app.c소스 작성
```
pi@raspberrypi:~/development/drivers/sample_driver $ nano sample_app.c
```
```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>

int main(void)
{
    char buffer[1024] = {0, };

    int fd = open("/dev/sample", O_RDWR);
    write(fd, buffer, 1);
    read(fd, buffer, 1);
    close(fd);

    return 0;
}
```
2. 컴파일 하기
```
pi@raspberrypi:~/development/drivers/sample_driver $ gcc -o sample_app sample_app.c
```

3. 실행하기
```
pi@raspberrypi:~/development/drivers/sample_driver $ ./sample_app
```

4. 커널로그 확인하기
```
pi@raspberrypi:~/development/drivers/sample_driver $ tail /var/log/kern.log
```
