# Problem 

Đây là Pseudo code của chương tình khi mình bỏ nó vào trình IDA 7.5 xịn xò con bò của anh Hoàng mình xin phép dấu tên share cho mình. :v

![image](https://user-images.githubusercontent.com/76993858/104085152-90e23480-527f-11eb-98a2-6c1ab2649cf8.png)
![image](https://user-images.githubusercontent.com/76993858/104085168-b8390180-527f-11eb-8dd4-e294c15539d7.png)
![image](https://user-images.githubusercontent.com/76993858/104085172-c2f39680-527f-11eb-84ad-8f2d94fb77e5.png)
![image](https://user-images.githubusercontent.com/76993858/104085174-cab33b00-527f-11eb-8446-9e3fdea834f2.png)

Đúng như cái tên của chương trình thì mình phải dùng ROP để by pass được bài này!

Nhưng vấn đề ở đây không chỉ ROP bình thường mà trong hàm flag, vì nó kiểm tra những tham số truyền vào nên chúng ta phải by pass lần lượt từng cái.

**Let's go!!!**

# Sovle

Để có thể in ra được flag, mình sẽ by pass điều kiện trong hàm flag như sau:
![image](https://user-images.githubusercontent.com/76993858/104085557-1adfcc80-5283-11eb-82a8-8414fea4d2d4.png)

vì win1 và win2 phải bằng true tức là bằng 1 nên chúng ta phải by pass luôn 2 hàm win_function 1 và 2 nữa. Vậy là có thể giải quyết được bài này.

![image](https://user-images.githubusercontent.com/76993858/104085564-28955200-5283-11eb-8b72-94a6db51afbb.png)
![image](https://user-images.githubusercontent.com/76993858/104085569-3054f680-5283-11eb-8c02-0abba3b4ee67.png)

Chúng ta sẽ dùng ROP nhưng vì có 3 điều kiện cần và đủ nên chúng ta phải sắp xếp thứ tự ROP để cho chương trình thêm mượt mà nèo. :>>

## Phân tích cách ROP-chain trong bài này
* win_function1 không cần tham số, gọi nó là chúng ta được win1 = 1. Easy-ing
* win_function2 thì cần 1 tham số đúng bằng `0xBAAAAAAD`, ta sẽ được win2 = 1.
* Cuối cùng là flag, cần tham số là 0xDEADBAAD. Vậy là xong.

**Thứ tự sắp xếp là win_func1 + win_func2 + flag + arg(win_func2) + arg(flag)**

Mình quên mất, còn 1 vấn đề nữa là offset từ lúc nhập vào đến ret, chúng ta sẽ lấy địa chỉ của nó như sau:
![image](https://user-images.githubusercontent.com/76993858/104085772-92622b80-5284-11eb-9c76-0453213ed0f7.png)

Sau khi thu thập đủ thông tin cần exploit thì giờ mình sẽ viết 1 file exploit.py đơn giản như sau:

![image](https://user-images.githubusercontent.com/76993858/104085835-00a6ee00-5285-11eb-8d5b-d30a2140097a.png)

Uầy!!! Tính tới thời điểm mình viết write-up đã là 2 tuần hơn rồi nhưng server của những anh chị Efient vẫn còn mở, Cảm ơn mn nhiều, <3 <3 

Tadaaaaaa

![image](https://user-images.githubusercontent.com/76993858/104085810-ce958c00-5284-11eb-9c94-d861e00222af.png)