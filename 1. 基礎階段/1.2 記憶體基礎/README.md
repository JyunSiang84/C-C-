# 記憶體基礎

## C語言常發生問題

### A. Buffer overflow Problem
    - 永遠檢查陣列邊界
    - 使用安全的函式
    - 適當的錯誤處理
    - 謹慎處理使用者輸入
    - 避免直接操作記憶體位址
    - 定期更新和修補安全漏洞
    - 保持程式碼簡潔和可維護性
``` c
#include <stdio.h>
#include <windows.h>

int main(int argc, char **argv) {
  // 設置控制台輸出編碼為 UTF-8
  SetConsoleOutputCP(65001);

  // 宣告一個大小為 1 的 unsigned long 陣列
  unsigned long a[1];
  printf("陣列 a 的大小為: %d\n", sizeof(a) / sizeof(unsigned long));
  printf("正在嘗試訪問陣列索引 3 (超出範圍)\n");

  // 在寫入前顯示記憶體內容
  printf("寫入前的記憶體內容:\n");
  for (int i = 0; i < 4; i++) {
    printf("記憶體位置 a[%d]: %lu\n", i, *((unsigned long *)a + i));
  }

  // 緩衝區溢位操作
  printf("\n執行緩衝區溢位寫入...\n");
  a[3] = 0x7fffff7b36cebUL;

  // 在寫入後顯示記憶體內容
  printf("\n寫入後的記憶體內容:\n");
  for (int i = 0; i < 4; i++) {
    printf("記憶體位置 a[%d]: %lu\n", i, *((unsigned long *)a + i));
  }

  printf("\n警告：這個程式存在緩衝區溢位風險！\n");
  printf("陣列大小為 1，但試圖訪問索引 3\n");

  return 0;
}

}
```
#### A1.使用安全的陣列大小和邊界檢查：
``` c
#include <stdio.h>
#define ARRAY_SIZE 4

int main(int argc, char **argv) {
    unsigned long long a[ARRAY_SIZE];
    int index = 3;
    
    // 檢查陣列邊界
    if (index >= 0 && index < ARRAY_SIZE) {
        a[index] = 0x7fffff7b36cebUL;
    } else {
        printf("錯誤：陣列索引超出範圍！\n");
        return -1;
    }
    
    return 0;
}
```
#### A2. 使用標準函式庫的安全版本：
``` c
#include <stdio.h>
#include <string.h>

int main() {
    char buffer[10];
    // 使用 strncpy 而不是 strcpy
    strncpy(buffer, "測試字串", sizeof(buffer) - 1);
    buffer[sizeof(buffer) - 1] = '\0';
    
    return 0;
}
```
#### A3. 使用動態記憶體時進行檢查：
``` c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *array;
    int size = 10;
    
    // 檢查記憶體分配
    array = (int *)malloc(size * sizeof(int));
    if (array == NULL) {
        printf("記憶體分配失敗！\n");
        return -1;
    }
    
    // 使用完釋放記憶體
    free(array);
    return 0;
}
```
#### A4. 善用編譯器的安全檢查功能：
``` c
// 編譯時加入安全檢查選項
// gcc -Wall -Wextra -Werror -fstack-protector-all program.c
```
