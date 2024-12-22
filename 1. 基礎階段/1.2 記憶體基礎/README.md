# 記憶體基礎
## 1. 基本數據類型與記憶體大小
Arduino 中最常用的數據類型：
```cpp
// 整數類型
boolean    // 1 byte (8 bits)  範圍: true/false
byte       // 1 byte (8 bits)  範圍: 0-255
char       // 1 byte (8 bits)  範圍: -128 到 127
int        // 2 bytes (16 bits) 範圍: -32,768 到 32,767
long       // 4 bytes (32 bits) 範圍: -2,147,483,648 到 2,147,483,647

// 浮點數類型
float      // 4 bytes (32 bits) 範圍: -3.4028235E+38 到 3.4028235E+38
double     // 4 bytes (32 bits) 在 Arduino 中與 float 相同

// 字串類型
String     // 動態大小，基本overhead為 6-7 bytes，加上實際字串長度
```
為了更好理解這些數據類型，讓我們看一個實際例子：
```cpp
void setup() {
    Serial.begin(9600);
    
    // 讓我們看看不同數據類型如何使用記憶體
    byte age = 25;            // 只用 1 byte 就足夠儲存年齡
    int temperature = -10;    // 需要 2 bytes 因為可能有負值
    long population = 23000000; // 需要 4 bytes 存儲大數字
    
    // 印出各變數的值與大小
    Serial.print("age（1 byte）: ");
    Serial.println(age);
    Serial.print("temperature（2 bytes）: ");
    Serial.println(temperature);
    Serial.print("population（4 bytes）: ");
    Serial.println(population);
}

void loop() {
}
```

## 2. 變數儲存方式
Arduino 使用兩種主要的記憶體儲存方式：
```cpp
// 靜態儲存（編譯時決定）
const int SERVO_PIN = 9;  // 儲存在 Flash 記憶體

// 動態儲存（運行時使用）
int sensorValue;         // 儲存在 RAM
```
這裡有個更詳細的例子：
```cpp
// 全域變數（存在 RAM 的資料段）
int globalVar = 100;

void setup() {
    Serial.begin(9600);
    
    // 區域變數（存在 RAM 的堆疊段）
    int localVar = 50;
    
    // 常數（存在 Flash）
    const float PI = 3.14159;
    
    Serial.println("記憶體位置示範：");
    Serial.print("globalVar 位置：");
    Serial.println((int)&globalVar);
    Serial.print("localVar 位置：");
    Serial.println((int)&localVar);
}
```
## 3. 全域變數 vs 區域變數
讓我們透過一個實際例子來理解差異：
```cpp
// 全域變數：整個程式都可以訪問
int globalTemp = 25;  // 存在於 RAM 中直到程式結束

void showTemperature() {
    // 可以訪問全域變數
    Serial.print("在另一個函數中的全域溫度：");
    Serial.println(globalTemp);
    
    // 無法訪問 localTemp，因為它是 setup 的區域變數
    // 這行會造成編譯錯誤：
    // Serial.println(localTemp);
}

void setup() {
    Serial.begin(9600);
    
    // 區域變數：只在 setup 函數中有效
    int localTemp = 30;  // 當 setup 執行完就會被釋放
    
    Serial.print("全域溫度：");
    Serial.println(globalTemp);
    
    Serial.print("區域溫度：");
    Serial.println(localTemp);
    
    // 呼叫函數來展示變數範圍
    showTemperature();
}



void loop() {
}
```
## 4. RAM vs Flash 基本概念
## 5. Arduino 記憶體架構基礎


## C語言常發生問題
```cpp
```
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
