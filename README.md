## C語言常發生問題

### Buffer overflow Problem

``` c
#include <stdio.h>

int main(int argc, char **argv) {
    // 宣告一個大小為 1 的 unsigned long 陣列
    unsigned long a[1];
    
    // 嘗試訪問陣列越界的位置（第4個元素），這是一個緩衝區溢位
    a[3] = 0x7fffff7b36cebUL;
    
    // 程式結束
    return 0;
}
```
