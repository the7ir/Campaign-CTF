# Problem 

Bài này chúng ta sẽ gặp 1 lỗi cũng nguy hiểm không kém cạnh Buffer Overflow, đó chính là lỗi Format String.
Nói 1 chút về format string, thì nó là lỗi do người lập trình có thói quen xấu khi code, hoặc khi người lập trình sử dụng các hàm print, printf.... mà không dùng những đặc tả như: %d %s %c .... Thay vào đó, người lập trình chỉ dùng:
```sh
print(username);
```
Điều này hết sức nguy hiểm, vì lỗi này có thể leak được những dữ liệu ` nhạy cảm ` từ stack khi chương trình đang chạy.
