---
title: Simple Encryptor【HTB】【WriteUp】
tags:
  - Security
  - 初心者
  - writeup
  - HackTheBoxsdd
private: true
updated_at: '2024-10-03T01:13:36+09:00'
id: c0bd3640120c8aba4967
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに
はじめてreversingに挑戦してみました！
備忘録がてら実施内容を記事に起こしてみようと思います。

# 問題

# デコンパイル
バイナリをIDAでデコンパイルして暗号化の内容を見てみます。デコンパイルするとC言語のコードを入手することができました。

### コード
入手したコードです。暗号化の手順を解説していきます。
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

### 関数一覧
- `unsigned __int64 __readfsqword(unsigned long Offset)`
  - FSセグメントの先頭を基準としたオフセットによって指定された場所からメモリを読み取ります。
- `FILE *fopen(const char *filename, const char *mode)`
  - `filename`のファイルを開く関数です。今回用いた`mode`は読み取り用にバイナリー・ファイルをオープンする`rb`と書き込み用に空のバイナリー・ファイルを作成する`wb`です。
- `int fseek(FILE *stream, long offset, int origin)`
  - `stream`と関連付けられているファイルの現在位置を、ファイル内の新しいロケーションに変更します。具体的に`SEEK_SET`、`SEEK_CUR`、`SEEK_END`の3つの`origin`から`offset`分移動した値にポインタを移動させます。
- `long ftell(FILE *stream)`
  - ファイルの現在の位置を返します。
- `void *malloc(size_t size)`
  - 割り当てられた領域への`void`ポインターを返します。このとき少なくとも`size`バイトのメモリブロックを割り当てます。
- `size_t fread(void *buffer, size_t size, size_t count, FILE *stream)`
  - 入力`stream`から`size`長の`count`項目までを読み取り、それらを指定された`buffer`に格納します。
- `int fclose(FILE *stream)`
   - ストリームをフラッシュし、次にそのストリームに関連するファイルをクロ ーズします。その後、関数はそのストリームに関連するバッファーをすべて解放します。
- `time_t time(time_t *timeptr`
  - 現在のカレンダー時間を秒単位で判別します。
- `void srand(unsigned int seed)`
  - 一連の疑似乱数を生成するための開始点を設定します。`srand()`が呼び出されない場合、`rand()`シードは、プログラムの開始時に`srand(1)`が呼び出されたかのように設定されます。それ以外の値が`seed`に設定された場合には、異なる開始点に生成プログラムが設定されます。
- `int rand(void)`
  - `0` ～ `RAND_MAX`(`0x7fff`)の範囲の疑似乱数整数を生成します。`rand()`を呼び出して乱数生成プログラムに`seed`を設定する前には、`srand()`関数を 使用してください。`srand()`を呼び出さない場合、デフォルトのシードは`1`です。
- `size_t fwrite(const void *buffer, size_t size, size_t count, FILE *stream)`
  - 最大数`count`の項目を、それぞれ`size`バイト長まで、`buffer`から出力`stream`に書き込みます。
- `__ROL1__(v1, v2)`
  - `v1`を`v2`だけ左にローテーションさせる

### コード解説
変数についての説明は割愛します。
`for`文の中身以外の処理は基本的なファイル操作で、フラグをファイルから読み取り、処理を施し、シードとともに暗号化したフラグを保存するといった内容です。

1. `__readfsqword(0x28u)`でFSセグメントの先頭から`0x28u`(40バイト先)の場所から64ビットのデータを読み取ります。このデータはスレッド情報を読み取っていると考えられます。（詳しくないのであってないかもしれないです。）
2. `fopen("flag", "rb")`で`flag`ファイルを開きます。この時、返り値はファイルにアクセスするための`FILE`構造体のポインタを返します。
3. `fseek(stream, 0LL, 2)`この処理は`SEEK_END`からオフセット`0`の場所つまりファイルの最後までポインタを移動させます。
4. `ftell(stream)`でファイル末尾のポインタの位置がわかる。つまりファイルサイズを取得することができます。
5. `fseek(stream, 0LL, 0)`でポインタの位置を元にもどします（`SEEK_SET`からオフセット`0`）。
6. `ptr = malloc(size)`ファイルサイズに応じてメモリを確保します。
7. `fread(ptr, size, 1uLL, stream)`ファイルの最初（`stream`）から最後（`stream+size`）までを1カウントで読み込み、`ptr`に格納しています。
8. `fclose(stream)`で`stream`変数で確保していた領域を解放します。
9. `seed[0] = time(0LL)`で現在時刻を取得`0LL`は`NULL`と同値で格納しているメモリを指定していない。
10. `srand(seed[0])`は疑似乱数の開始点に先ほど取得した現在時刻をセットしている。
11. **flagのencrypt処理を1バイトずつ実行する ※ 後に詳しく処理を解説します。**
12. `s = fopen("flag.enc", "wb")`で書き込み用のフォルダをオープンします。
13. `fwrite(seed, 1uLL, 4uLL, s)`で`seed`の内容を4つの項目を1バイトずつ書き込んでいきます。 ※ 4バイト保存なので`seed[0]`のみを保存
14. `fwrite(ptr, 1uLL, size, s)`で同様に`ptr`の`size`個の項目を1バイトずつ書き込んでいきます。
15. `fclose(s)`で`s`変数で確保していた領域を解放します。

つづいて暗号化処理についてどのような処理がされているか解説します。
#### `for`文の中の処理
`i`バイト目の文字を`ci`とすると
1. 1つ目の`rand()`(`r1`)と`ci`のビット単位の排他的論理和の計算(`ci^r1`)
2. `v3`に2つめの`rand()`(`r2`)の格納
3. `v5`に`ci^r1`の格納
4. `seed[1]`に格納してるがその後の計算で特につかわないので無視
5. `__ROL1__(v5, v3 & 7)`の処理をおこない暗号化完了。
  - `__ROL1__(v5, v3 & 7)`：`v5`を`v3 & 7`だけ左にローテーションする

#### ローテーションについて
ここでローテーションについて簡単に説明します。
まず、以下のように格納されている値があるとします。

| アドレス | 7 | 6 | 5 | 4 | 3 | 2 | 1 |
| --- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 値 | 0 | 1 | 0 | 1 | 1 | 1 | 0 |

左に1にローテションするということは、アドレスの値はアドレス2へ、アドレス2の値はアドレス3へ…というふうに移動させ最後にアドレス7の値はアドレス1に移動させます。
最終的に以下の値になります。
| アドレス | 7 | 6 | 5 | 4 | 3 | 2 | 1 |
| --- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 値 | 1 | 0 | 1 | 1 | 1 | 0 | 0 |

### POC
単純に1バイトずつ逆の処理を施すことで、複合することができます。
`r1`と`r2`についてはseedから求めることができます。
暗号化された値1バイト（`d1`）に対して以下の処理を行います。
1. `d1`を`r2 & 7`分右にローテーション
2.  1の値^`r1`を計算する

``` c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
    FILE *stream;
    size_t size;
    char *d;
    int seed;
    int result;
    int r1;
    int r2;
    int key;
    stream = fopen("flag.enc", "rb");
    fseek(stream, 0, 2);
    size = ftell(stream);
    fseek(stream, 0, 0);
    d = malloc(size);
    fread(d, sizeof(char), size, stream);
    fclose(stream);
    memcpy(&seed, d, sizeof(seed));
    printf("seed: %d\n", seed);
    srand(seed);

    for (int i=4; i < size; i++) {
        r1 = rand();
        r2 = rand();
        key = r2 & 7;
        d[i] = (unsigned char)((d[i] >> key) | ((d[i]) << (8 - key)));
        d[i] = d[i] ^ r1; 
    }

    for (int i = 4; i < size; i++) {
        printf("%c", d[i]);
    }
    return 0;
}
```
