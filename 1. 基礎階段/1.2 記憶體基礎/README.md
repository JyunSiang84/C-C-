# 記憶體基礎
## 1. 基本數據類型與記憶體大小
### 1.1 Arduino 中最常用的數據類型：
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
針對float vs double 的差異：
```cpp
// 標準 C/C++ 中：
float  // 單精度浮點數：4 bytes，約 7 位數的精確度
double // 雙精度浮點數：8 bytes，約 15-17 位數的精確度

// 但在 Arduino 中：
float  // 4 bytes
double // 4 bytes（實際上被編譯器轉換為 float）
```
為什麼 Arduino 還保留兩種類型？這是為了程式碼的可移植性和相容性。許多程式碼可能是從標準 C/C++ 移植過來的，如果 Arduino 完全移除 double 類型，這些程式碼就需要大量修改。通過保留兩種類型但讓它們在底層相同，Arduino 可以：
- 保持與標準 C/C++ 的語法相容性
- 節省 AVR 微控制器有限的記憶體資源
- 確保程式碼可以在不同平台間更容易移植
- 資料類型範圍檢查：

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
### 1.2 溢位處理：

```cpp
void setup() {
    Serial.begin(9600);
    
    // 方法一：使用範圍檢查
    int rawTemperature = -10;
    if (rawTemperature >= 0 && rawTemperature <= 255) {
        byte safeTemperature = rawTemperature;
        Serial.println("溫度在有效範圍內");
    } else {
        Serial.println("警告：溫度超出 byte 的有效範圍！");
    }
    
    // 方法二：使用 static_cast 進行明確的類型轉換
    int originalValue = -10;
    
    // static_cast 會讓程式設計師更清楚地意識到類型轉換
    byte convertedValue = static_cast<byte>(originalValue);
    
    // 方法三：使用 constrain 函數確保值在正確範圍內
    byte constrainedValue = constrain(originalValue, 0, 255);
    
    Serial.print("Original: ");
    Serial.println(originalValue);
    Serial.print("Converted: ");
    Serial.println(convertedValue);
    Serial.print("Constrained: ");
    Serial.println(constrainedValue);
}
```

## 2. 全域變數 vs 區域變數
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

## 3. 變數儲存方式
### 3.1 Arduino UNO 的記憶體架構：
#### 3.1.1  Flash (程式記憶體)：
- 32 KB(容量大)
- 只能在寫程式時更改內容
- 斷電後資料保留，當你關掉電源（闔上書），內容還在那裡（斷電後資料保持）
- 當你要更換內容時，需要重新印刷整本書（上傳新程式）
- 主要存放程式碼和常數

#### 3.1.2 SRAM (RAM)：
- 2 KB(容量小)
- 可以快速讀寫
- 斷電後資料消失
- 存放運行時的變數

#### 3.1.3 EEPROM：
- 1 KB(容量更小)
- 可以在程式運行時更改
- 斷電後資料保留
- 適合存放設定值或重要數據

#### 3.1.4 應用範例
這裡是一個展示不同記憶體使用的例子：
```cpp
// 儲存在 Flash 中的常數字串，靜態儲存（編譯時確定）
const char* flashString PROGMEM = "這個字串存在 Flash 中";

// 儲存在 RAM 中的一般字串，態儲存（運行時使用）
String ramString = "這個字串存在 RAM 中";

void setup() {
    Serial.begin(9600);
    
    // 顯示兩種字串的使用方式
    Serial.println(flashString);  // 需要特殊處理才能讀取
    Serial.println(ramString);    // 可以直接使用但佔用寶貴的 RAM
}

// EEPROM：儲存設定值
int savedThreshold = EEPROM.read(0);  // 從 EEPROM 讀取溫度閾值
```

### 3.2 Flash vs RAM
1. 持久性：
- Flash：斷電後資料保持 
- RAM：斷電後資料消失

2. 讀寫速度：
- Flash：讀取較慢，寫入更慢
- RAM：讀寫都很快

3. 使用壽命：
- Flash：有寫入次數限制（約 10,000 次）
- RAM：無寫入次數限制

4. 存取方式：
- Flash：需要特殊指令讀取（如使用 PROGMEM）
- RAM：可直接存取
```cpp
// 靜態儲存（編譯時決定）固定的大型數據放在 Flash
const int SERVO_PIN = 9;  // 儲存在 Flash 記憶體
const char* FLASH_STRING PROGMEM = "Hello";  // 明確指定儲存在 Flash
const char* menuItems[] PROGMEM = {
    "選項 1",
    "選項 2",
    "選項 3"
};  // 選單文字不會改變，佔用空間大

// 動態儲存（運行時使用）經常變動的數據放在 RAM
int ramVariable = 42;                 // 可變，儲存在 RAM
const int RAM_CONSTANT = 42;          // 不可變，但仍在 RAM
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

### 3.3 應用範例
當資料存儲在 Flash 中時，需要使用特殊函數讀取。這裡是一個完整的例子：
```cpp
#include <avr/pgmspace.h>

// 在 Flash 中存儲一個簡單的字串
const char welcomeMessage[] PROGMEM = "歡迎使用 Arduino";

void setup() {
    Serial.begin(9600);
    
    // 建立一個緩衝區來存放從 Flash 讀取的字串
    char buffer[32];  // 確保緩衝區足夠大
    
    // 使用 strcpy_P 從 Flash 複製字串到 RAM
    strcpy_P(buffer, welcomeMessage);
    
    // 現在我們可以使用這個字串了
    Serial.println(buffer);
}
```
讓我們看一個更複雜的例子，展示不同的使用情境：
```cpp
#include <avr/pgmspace.h>  // 必須包含這個標頭檔才能使用 PROGMEM

// 在 Flash 中存儲多個訊息 正確方式：使用 PROGMEM 將字串存在 Flash 中
const char message1[] PROGMEM = "溫度過高警告！";
const char message2[] PROGMEM = "系統運行正常";
const char message3[] PROGMEM = "請檢查感應器";

// 錯誤方式：沒有使用 PROGMEM，字串會存在 RAM 中
const char message_RAM[] = "這個字串會存在 RAM 中";

// 建立一個訊息表格，也存儲在 Flash 中
const char* const messages[] PROGMEM = {
    message1,
    message2,
    message3
};

// 在 Flash 中存儲一個長字串
const char longText[] PROGMEM = 
    "這是一段很長的說明文字，"
    "如果存儲在 RAM 中會佔用太多空間，"
    "所以我們將它存儲在 Flash 中。";

void setup() {
    Serial.begin(9600);
    char buffer[100];  // 確保在 RAM 中建立暫存區緩衝區夠大
    
    // 情境 1：讀取單一訊息
    // 使用 strcpy_P 從 Flash 讀取字串
    strcpy_P(buffer, message1);
    Serial.println(buffer);
    
    // 情境 2：從訊息表格中讀取
    // 首先要取得字串的位址
    PGM_P messagePtr = (PGM_P)pgm_read_word(&messages[1]);
    strcpy_P(buffer, messagePtr);
    Serial.println(buffer);
    
    // 情境 3：讀取長文字
    strcpy_P(buffer, longText);
    Serial.println(buffer);

    // 直接使用 RAM 中的字串
    // strcpy_P讀取message_RAM 這可能導致未定義的行為
    Serial.println(message_RAM);    // 從 RAM 讀取

}

// 使用函數來顯示特定訊息
void showMessage(uint8_t messageIndex) {
    char buffer[100];
    
    if (messageIndex < 3) {  // 確保索引有效
        // 獲取字串位址並複製到緩衝區
        PGM_P messagePtr = (PGM_P)pgm_read_word(&messages[messageIndex]);
        strcpy_P(buffer, messagePtr);
        Serial.println(buffer);
    }
}
```
- 記得包含 <avr/pgmspace.h> 標頭檔
- PROGMEM 關鍵字告訴編譯器將資料存在 Flash 中，源字串必須使用 PROGMEM 儲存在 Flash 中
- pgm_read_word() 用於從 Flash 讀取資料
- strcpy_P() 用於將 Flash 中的字串複製到 RAM 中的緩衝區，但是總是要確保目標緩衝區夠大
這樣的特殊處理是必要的，因為 Flash 的存取方式與 RAM 不同，需要特定的指令來讀取資料。



## 4. EEPROM
Flash 記憶體就像是一本存放在 Arduino 內部的書。當我們斷電時，雖然這本書還在那裡，但我們需要特定的工具和程序才能讀取它的內容。
在 Arduino 中，程式是以二進制形式儲存在 Flash 記憶體中。當我們斷電後，要讀取這些內容需要：
- AVR ISP 程式燒錄器
-  Arduino 作為 ISP（In-System Programmer）
-  其他 AVR 微控制器程式讀取工具
這就像是要讀取一本特殊格式的電子書，我們需要正確的閱讀器才能開啟它。並不像讀取一般檔案那麼簡單，這就是為什麼在實際應用中，如果我們需要在斷電後保存和讀取資料，我們通常會選擇使用 EEPROM：
  
EEPROM（Electrically Erasable Programmable Read-Only Memory）是一種可以在電源關閉後仍保留資料的記憶體。以下是一個具體的例子：
```cpp
#include <EEPROM.h>

void setup() {
    Serial.begin(9600);
    
    // 寫入資料到 EEPROM
    int addressToWrite = 0;          // EEPROM 的地址從 0 開始
    byte valueToWrite = 123;         // 要儲存的值
    EEPROM.write(addressToWrite, valueToWrite);
    Serial.println("已寫入資料到 EEPROM");
    
    // 讀取 EEPROM 中的資料
    delay(1000);
    byte valueRead = EEPROM.read(addressToWrite);
    Serial.print("從 EEPROM 讀取的值: ");
    Serial.println(valueRead);
    
    // 寫入和讀取較大的數據（如 int）
    int bigNumber = 12345;
    // 將 int 分成兩個 byte 儲存
    EEPROM.write(1, bigNumber & 0xFF);          // 儲存低位元組
    EEPROM.write(2, (bigNumber >> 8) & 0xFF);   // 儲存高位元組
}

void loop() {
    // 空的
}
```
```cpp
#include <EEPROM.h>

void setup() {
    Serial.begin(9600);
    
    // 將訊息寫入 EEPROM
    String message = "這是要保存的訊息";
    for (int i = 0; i < message.length(); i++) {
        EEPROM.write(i, message[i]);
    }
    EEPROM.write(message.length(), '\0');  // 結束符號
}

void loop() {
    // 從 EEPROM 讀取訊息
    String storedMessage = "";
    int i = 0;
    char c;
    while ((c = EEPROM.read(i)) != '\0') {
        storedMessage += c;
        i++;
    }
    
    Serial.println(storedMessage);
    delay(1000);
}
```
這樣的設計更適合需要在斷電後讀取的資料，因為：
- EEPROM 專門設計用於資料存儲
- 可以通過程式輕鬆讀寫
- 不需要特殊設備

所以，雖然 Flash 中的內容確實會在斷電後保留，但如果要在實際應用中保存和讀取資料，使用 EEPROM 是更好的選擇。這就是為什麼 Arduino 提供了這兩種不同類型的記憶體，各自服務於不同的用途。

## 5. Arduino 記憶體架構基礎
Arduino 的記憶體分為幾個主要區段：
```cpp
// 1. 程式區段 (Flash)
const int MAGIC_NUMBER = 42;  // 常數存在 Flash

// 2. 資料區段 (RAM)
int globalCounter = 0;        // 全域變數

void setup() {
    // 3. 堆疊區段 (RAM)
    int localVar = 100;       // 區域變數
    
    // 4. 堆積區段 (RAM)
    String* dynamicString = new String("動態配置的字串");
    
    // 記得釋放動態配置的記憶體
    delete dynamicString;
}
```
要有效管理 Arduino 的記憶體，請記住：
- 盡可能使用合適大小的數據類型
- 注意全域變數的使用，它們會持續佔用 RAM
- 大型常數資料最好存在 Flash 中
- 注意字串的使用，它們可能佔用大量 RAM

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
