### 1

```c
/* If the number just past LAST prints as a value equal to LAST, and prints differently from the previous number, then print the number.  This avoids problems with rounding.  For example, with the x86 it causes "seq 0 0.000001 0.000003" to print 0.000003 instead of stopping at 0.000002.  */

bool print_extra_number = false;
long double x_val;
char *x_str;
int x_strlen;
if (locale_ok)
setlocale (LC_NUMERIC, "C");
x_strlen = asprintf (&x_str, fmt, x);
if (locale_ok)
setlocale (LC_NUMERIC, "");
if (x_strlen < 0)
xalloc_die ();
x_str[x_strlen - layout.suffix_len] = '\0';

if (xstrtold (x_str + layout.prefix_len, nullptr,
            &x_val, cl_strtold)
    && x_val == last)
{
    char *x0_str = nullptr;
    int x0_strlen = asprintf (&x0_str, fmt, x0);
    if (x0_strlen < 0)
    xalloc_die ();
    x0_str[x0_strlen - layout.suffix_len] = '\0';
    print_extra_number = !streq (x0_str, x_str);
    free (x0_str);
}

free (x_str);
if (! print_extra_number)
break;
```
---

翻訳：もしLASTを超えた直後の数値がLASTと等しく表示され、かつ前の数値とは異なる表示になる場合、その数値を出力します。これは丸め誤差による問題を回避するためです。例えば、x86アーキテクチャでは「seq 0 0.000001 0.000003」とした場合、0.000002で停止する代わりに0.000003が出力されることがあります。

### 2
### 3
