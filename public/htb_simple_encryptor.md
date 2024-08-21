---
title: Simple Encryptor やってみた【HTB】【WriteUp】
tags:
  - Security
  - 初心者
  - writeup
  - HackTheBox
private: true
updated_at: '2024-08-21T22:18:02+09:00'
id: c0bd3640120c8aba4967
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに
はじめてreversingに挑戦してみました！ \
備忘録がてら実施内容を記事に起こしてみようと思います。

# 問題

# デコンパイル
バイナリをIDAでデコンパイルして暗号化の内容を見てみます。デコンパイルするとC言語のコード

### コード

``` c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char v3; // al
  char v5; // [rsp+7h] [rbp-39h]
  unsigned int seed[2]; // [rsp+8h] [rbp-38h] BYREF
  __int64 i; // [rsp+10h] [rbp-30h]
  FILE *stream; // [rsp+18h] [rbp-28h]
  size_t size; // [rsp+20h] [rbp-20h]
  void *ptr; // [rsp+28h] [rbp-18h]
  FILE *s; // [rsp+30h] [rbp-10h]
  unsigned __int64 v12; // [rsp+38h] [rbp-8h]

  v12 = __readfsqword(0x28u);
  stream = fopen("flag", "rb");
  fseek(stream, 0LL, 2);
  size = ftell(stream);
  fseek(stream, 0LL, 0);
  ptr = malloc(size);
  fread(ptr, size, 1uLL, stream);
  fclose(stream);
  seed[0] = time(0LL);
  srand(seed[0]);
  for ( i = 0LL; i < (__int64)size; ++i )
  {
    *((_BYTE *)ptr + i) ^= rand();
    v3 = rand();
    v5 = *((_BYTE *)ptr + i);
    seed[1] = v3 & 7;
    *((_BYTE *)ptr + i) = __ROL1__(v5, v3 & 7);
  }
  s = fopen("flag.enc", "wb");
  fwrite(seed, 1uLL, 4uLL, s);
  fwrite(ptr, 1uLL, size, s);
  fclose(s);
  return 0;
}
```

### 実行内容


