# 이력서
![트와이스]('C:\Users\한국아이티교육원\Desktop\Winter\twice_v1.jpg')

## 학력란

### 고등학교
* 대건고등학교
  - 입학년도
  - 졸업년도 : 1988년 2월  
  
### 대학교
  - 입학년도 : 
  
## 경력란

## 자기소개서
```
천국에서 살아온 지난 날들이 
아련한 추억이 되고,,
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
	buffer[0] = 1
	write(fd, buffer, 1);
	
	sleep(2)
	
	//GPIO_6 LED OFF
	buffer[0] = 0
	write(fd, buffer, 1);    
    close(fd);

    return 0;
}


```
