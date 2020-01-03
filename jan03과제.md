# 과제 제출
## 6번 led 점등하기

## [ 커널 모듈(드라이버)]

1. 커널모듈(드라이버) 작성
```c
pi@raspberrypi:~/development/drivers/sample_driver $ nano sample_driver.c
```
```c
#include <linux/module.h>
#include <linux/io.h>
#include <linux/delay.h>

volatile unsigned int* map;

static int __init mygpio_init(void)
{
    map = (volatile unsigned int*) ioremap(0x3F200000, 180);

	// map : 0x7E200000(GPFSEL0) 6번 20-18 001	// 000001000 00000000 00000000 00000000
    *map = 0x00040000; 					// 000000000 00000100 00000000 00000000
	// 0x7E200000 = 0x00040000;
	// map+7 : 0x7E20001c(GPSET0)
    *(map + 7) = 0x00000040;					// 00000000 00000000 00000000 01000000

	// ======================= 
	// 1. sheet에서 GPFSEL0 / GPSET0 / GPCLR0 세개의 주소통한 GPIO 설정방법
	// 0x7E20001c = 0x00000200;  0x7E200000 + 28(4byte * 7) = 0x00000200, 28을 16진수로 표현하면 1c
	// 포인터형 타입은 size가 4byte(int*, char*), 차이는 연산에서는 char*은 1byte
	// =======================


    msleep(2000);

	// map+10 : 0x7E200028(GPCLR0)
    *(map + 10) = 0x00000040;					// 00000000 00000000 00000000 01000000

    printk(KERN_INFO "map : %p\n", map);
    printk(KERN_INFO "call mygpio_init\n");

    return 0;
}

static void __exit mygpio_exit(void)
{
    iounmap(map);
    printk(KERN_INFO "call mygpio_exit\n");
}

module_init(mygpio_init);
module_exit(mygpio_exit);

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

1. dummy_app.c소스 작성
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

	// GPIO_6 LED ON
	buffer[0] = 1;
	write(fd, buffer, 1);

	sleep(2);

	//GPIO_6 LED OFF
	buffer[0] = 0;
	write(fd, buffer, 1);
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
