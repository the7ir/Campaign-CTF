# Luck
## Tết gần đến rồi, BTC cũng hiểu nên ra đề ăn tết sớm nè. 
### Nhìn tên của đề bài chắc các bạn cũng ngầm hiểu giống mình, đề đang nói đến vấn đề gì rồi đúng hônggg. :>>

Quay lại với Luck...

Challenge này sẽ cung cấp cho các bạn 69đ, bạn có quyền cược số tiền thấp hơn hoặc bằng số tiền bạn đang có.
* Nếu thắng cược, bạn sẽ được số tiền mà bạn đã đặt cược cộng với số tiền đang có.
* Nếu thua, bạn sẽ mất 10đ. Nếu bạn đặt dưới 10đ thì chắc chắn sẽ lỗ, vì số tiền mất luôn là 10đ.
* Khi bạn có đủ số tiền là `696.969.696.969đ` thì bạn sẽ có thể dùng số tiền này mua flag.

## Thấy luật chơi thì bạn thấy là lời rồi. Bắt đầu thôi.

Nhìn trong source thì thấy 

![image](https://user-images.githubusercontent.com/76993858/104086104-64cab180-5287-11eb-9537-c0c4bca8031a.png)

Ấy chà chà... Mình chỉ có 1/10 phần trăm là win thôi nhỉ? mà điều này cũng hem quan trọng lắm... 
 :>>
 
 # Solution

Vì chúng ta có thể chơi nhiều lượt nên phần trăm may ruổi cũng không quan trọng nữa nhỉ. Thua keo này mình thua thêm keo nữa có sao đâu :3

Chúng ta sẽ tạo 1 file để exploit bài này như sau:
* Cược tất cả những gì mình đang có. Được ăn cả ngã mất 10$. (Chơi là phải khô máu mà :>>)
* mình sẽ code tương tự hàm trên để đặt cược số tiền mà mình đã có.

## File Exploit.py

```sh
import random, time
from randcrack import RandCrack
from pwn import *

rc = RandCrack()
def get_value(n):
    value = 0
    for i in range(len(n)):
        value = 10*value + (ord(n[i]) - 0x30)
    return value

def getRandBits():
     for i in range(624):
         rc.submit(random.getrandbits(1))

def brute():
    s = remote('128.199.234.122',2222)
    money = 69
    count = 0
    while 1:
        bet = str(money)
        sleep(0.35)
        s.sendline('1')
        s.sendline(bet)
        rStr = s.recv()
        print('BET:  \t',bet)
        print('MONEY:\t',str(money))
        s.recvuntil("Money: ")
        money = s.recvuntil("$")
        money = money.replace("$","")
        money = int(money)
        if money > 696969696969:
            print('BUY THE FLAG, NOW!!!')
            s.interactive()
        if money < 10:
            s.close()
            break

while 1:
    brute()

```

### Ở đây mình có dùng sleep(0x35) để tránh tình trạng Race_Condition, tức là khi chương trình chạy quá nhanh trong 1 khoảng thời gian nào đó, thì cùng 1 biến của mình nó sẽ bị tranh giành giữa việc thêm tiền vào và lấy số tiền hiện tại để cược, dẫn đến tình trạng là mình có đủ khoảng như vậy để cược nhưng chương trình báo là Cược quá số tiền hiện có. 

Sau khi chạy thì chúng ta sẽ dừng lại khi số tiền đủ mua flag.
### Nhập số 2 để mua. END GAME. :>>

## Flag

`efiensctf{wh4t_1s_th4t_w31rd_numb3r_FeelsWeirMan}`

