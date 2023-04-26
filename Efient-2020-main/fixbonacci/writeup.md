# PROBLEM

Bài này sẽ Check xem dãy fibonacci đưa ra có đúng không, sau đó sẽ là 2 lượt nhập số nguyên %d trong source, và tồn tại 1 hàm getFlag luôn.

# Solution

Ý tưởng để mình giải bài này là mình redirection vủa Ret_addr về địa chỉ của biến getFlag, vậy là xong, để làm được điều này. Mình sẽ có những điều kiện như sau:

## Điều kiện:
 * lấy được địa chỉ của hàm getFlag
 * lấy được off set của ret_add để đặt getFlag_addr vào

 Để lấy được địa chỉ của getFlag mình sẽ dùng GDB để xem như sau:

![image](https://user-images.githubusercontent.com/76993858/104084235-02b68000-5278-11eb-81d4-3dedeb3334a1.png)

còn lại là mình sẽ kiểm tra xem off set là bao nữa coi như solve đc bài này, địa chỉ của ret_addr

![image](https://user-images.githubusercontent.com/76993858/104084554-a7d25800-527a-11eb-9b03-56a9a5508854.png)

mình sẽ kiểm tra offset nào sẽ làm thay đổi được địa chỉ ret_addr bằng 1 cách... kiểm thử.
Sau khi nhập 18 và 1234 thì kết quả sẽ như này
![image](https://user-images.githubusercontent.com/76993858/104084818-3b0c8d00-527d-11eb-8665-7ced7aa5b2ce.png)

Yeahh vậy là chúng ta đã biết được off set tiến hành nhâp `18` và địa chỉ `getFlag` đã được chuyển thành số int vì nó đã được đặc tả thành `%d` mà địa chỉ của getFlag dưới dạng hexa.

![Exploit](https://user-images.githubusercontent.com/76993858/104084850-8c1c8100-527d-11eb-8fbc-ee16b7aba113.png)

Taddaaaaaaaaaaaaaaaaaaaa.
## Chúc các bạn mạnh khỏe!!!