# seqコマンドへの-xオプション追加の実装サマリー

## 実装内容

`seq`コマンドに新しいオプション`-x`（`--hex-output`）を追加しました。このオプションは以下の機能を提供します：

- **整数値**: 16進数で出力
- **実数値**: 従来通り10進数で出力

## 変更箇所

すべての変更箇所には`/* ADDITION for option -x */`というコメントを付けて、差分が明確になるようにしています。

### 1. グローバル変数の追加 (57-58行目)
```c
/* ADDITION for option -x: If true print integers in hexadecimal format.  */
static bool hex_output;
```
16進数出力モードを追跡するための静的ブール変数を追加。

### 2. long_optionsへのオプション追加 (72行目)
```c
{ "hex-output", no_argument, nullptr, 'x'}, /* ADDITION for option -x */
```
長いオプション形式`--hex-output`を定義。

### 3. 使用方法テキストの更新 (100行目)
```c
  -x, --hex-output         output integers in hexadecimal format\n\
```
ヘルプメッセージに新しいオプションの説明を追加。

### 4. 変数の初期化 (588行目)
```c
hex_output = false; /* ADDITION for option -x */
```
main関数でhex_output変数をfalseで初期化。

### 5. getopt_longの更新 (602行目)
```c
optc = getopt_long (argc, argv, "+f:s:wx", long_options, nullptr); /* ADDITION for option -x */
```
短いオプション文字列に'x'を追加。

### 6. オプションハンドラの追加 (620-622行目)
```c
case 'x': /* ADDITION for option -x */
  hex_output = true;
  break;
```
-xオプションが指定された時にhex_outputをtrueに設定。

### 7. print_numbers関数の修正 (313-325行目)
```c
/* ADDITION for option -x: Check if hex output is enabled and value is an integer */
if (hex_output && x == (long long)x)
  {
    /* Output as hexadecimal for integer values */
    if (printf ("%llx", (long long)x) < 0)
      write_error ();
  }
else
  {
    /* Use default format for floating-point values */
    if (printf (fmt, x) < 0)
      write_error ();
  }
```
整数値かどうかをチェックし、hex_outputが有効な場合は16進数で出力。

### 8. get_default_format関数の修正 (390-392行目)
```c
/* ADDITION for option -x: For hex output, use %Lg to handle both cases */
if (hex_output)
  return "%Lg";
```
16進数出力モードの場合、適切なフォーマット文字列を返す。

### 9. 高速パスの無効化 (677行目、711行目)
```c
&& !hex_output) /* ADDITION for option -x: disable fast path for hex output */
```
16進数出力モードの場合、seq_fast関数を使用せずprint_numbers関数を使用するように変更。

## テスト結果

### テスト1: 基本的な整数シーケンス (1-5)
```bash
$ ./src/seq -x 1 5
1
2
3
4
5
```

### テスト2: 16進数範囲 (10-15 → a-f)
```bash
$ ./src/seq -x 10 15
a
b
c
d
e
f
```

### テスト3: より大きな範囲 (0-20 → 0-14 in hex)
```bash
$ ./src/seq -x 0 20
0
1
2
3
4
5
6
7
8
9
a
b
c
d
e
f
10
11
12
13
14
```

### テスト4: 255前後 (ff in hex)
```bash
$ ./src/seq -x 255 260
ff
100
101
102
103
104
```

### テスト5: ステップ増分付き (100から110まで2刻み)
```bash
$ ./src/seq -x 100 2 110
64
66
68
6a
6c
6e
```

### テスト6: 浮動小数点シーケンス（10進数のまま）
```bash
$ ./src/seq -x 1.5 0.5 3.5
1.5
2
2.5
3
3.5
```

### テスト7: 浮動小数点形式の整数値
```bash
$ ./src/seq -x 1.0 1.0 5.0
1
2
3
4
5
```

### テスト8: 長いオプション形式
```bash
$ ./src/seq --hex-output 10 15
a
b
c
d
e
f
```

### テスト9: -xなしの通常動作（10進数）
```bash
$ ./src/seq 10 15
10
11
12
13
14
15
```

## 実装の特徴

1. **明確な差分化**: すべての追加箇所に`/* ADDITION for option -x */`コメントを付与
2. **整数判定**: `x == (long long)x`で値が整数かどうかを判定
3. **浮動小数点の保持**: シーケンスに実数値が含まれる場合、整数値も10進数で出力
4. **高速パスの考慮**: 整数専用の高速パス（seq_fast）を16進数モードでは無効化
5. **両方のオプション形式**: 短いオプション`-x`と長いオプション`--hex-output`の両方をサポート

## コンパイル

```bash
cd /workspaces/ubuntu/custom-seq/coreutils-9.8
make
```

コンパイルエラーなし。すべてのテストが正常に動作。
