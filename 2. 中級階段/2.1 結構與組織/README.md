# 1. struct (結構體)
主要優點：
- 組織相關數據：把相關的變數組織在一起
- 程式碼結構化：使程式更容易理解和維護
- 封裝功能：可以把數據和相關的操作放在一起
- 重複使用：可以建立多個相同結構的物件

使用時機：
- 當你有多個相關的變數需要一起處理
- 當你需要在多個地方重複使用相同的數據結構
- 當你想要把相關的功能組織在一起
- 當你需要建立複雜的數據結構

```cpp
// 基本 struct 語法
struct 結構名稱 {
    資料型態1 變數名稱1;
    資料型態2 變數名稱2;
    ...
};
```

## 1.1 基本的 struct:
```cpp
// 定義一個表示點的結構
struct Point {
    int x;    // x 座標
    int y;    // y 座標
};

void setup() {
    // 建立結構變數
    Point p1;           // 方法1: 宣告結構變數
    p1.x = 10;         // 設定 x 座標
    p1.y = 20;         // 設定 y 座標
    
    // 方法2: 宣告時直接初始化
    Point p2 = {30, 40};  // x=30, y=40
}
```
## 1.2 包含函數的 struct:
結構內的函數是定義在結構中的一般函數，用於操作該結構的數據，用於執行各種操作和功能，需要明確呼叫才會執行，其可以使用任何有效的函數名稱
```cpp
struct Counter {
    int value;         // 計數值
    
    // 結構內的函數
    void increment() {
        value++;
    }
    
    void reset() {
        value = 0;
    }
};

void setup() {
    Counter c;
    c.value = 0;      // 設定初始值
    c.increment();     // value 變成 1
    c.increment();     // value 變成 2
    c.reset();        // value 變回 0
}
```

## 1.3 巢狀 struct:
```cpp
struct Time {
    int hour;
    int minute;
    int second;
};

struct Event {
    String name;        // 事件名稱
    Time startTime;     // 開始時間
    Time endTime;       // 結束時間
};

void setup() {
    Event meeting;
    meeting.name = "工作會議";
    meeting.startTime = {14, 30, 0};  // 14:30:00
    meeting.endTime = {16, 0, 0};     // 16:00:00
    
    // 存取巢狀結構的資料
    Serial.print(meeting.startTime.hour);   // 印出 14
}
```
## 1.4 帶有建構函數的 struct:
建構函數是一個特殊的函數，它在物件被創建時自動執行，用於初始化物件，在物件被創建時自動執行，命名必須與結構名稱相同
```cpp
struct Sensor {
    int pin;
    int value;
    
    // 建構函數 - 物件創建時自動執行
    Sensor(int p) {
        pin = p;
        value = 0;
        pinMode(pin, INPUT);
    }
    
    // 讀取感測器值
    void read() {
        value = analogRead(pin);
    }
};

void setup() {
    Sensor lightSensor(A0);    // 建立感測器物件
    lightSensor.read();        // 讀取感測器值
}
```
專門用於初始化物件的數據

```cpp
struct Counter {
    int value;
    
    // 建構函數 - 物件創建時自動執行
    Counter() {
        value = 0;    // 自動初始化 value 為 0
    }
    
    // 帶參數的建構函數
    Counter(int initialValue) {
        value = initialValue;    // 使用指定的值初始化
    }
    
    // 一般函數
    void increment() {
        value++;
    }
};

void setup() {
    Counter c1;          // 使用無參數建構函數，value 自動設為 0
    Counter c2(5);       // 使用帶參數建構函數，value 自動設為 5
    
    c1.increment();      // value 變成 1
    c2.increment();      // value 變成 6
}
```
回傳值：
- 建構函數：不能有回傳值
- 一般函數：可以有回傳值
```cpp
struct LED {
    int pin;            // LED 腳位
    bool state;         // LED 狀態
    
    // 建構函數：初始化 LED 負責完成所有初始化工作
    LED(int ledPin) {
        pin = ledPin;
        state = false;
        pinMode(pin, OUTPUT);    // 設置腳位模式
        digitalWrite(pin, LOW);  // 初始狀態為關閉
    }
    
    // 一般函數：控制 LED
    void turnOn() {
        state = true;
        digitalWrite(pin, HIGH);
    }
    
    void turnOff() {  // 負責日常操作
        state = false;
        digitalWrite(pin, LOW);
    }
    
    void toggle() {  // 負責日常操作
        state = !state;
        digitalWrite(pin, state);
    }
    
    bool getState() {  // 負責日常操作
        return state;
    }
};

void setup() {
    LED statusLed(13);    // 建構函數自動執行：設置腳位和初始狀態
    
    statusLed.turnOn();   // 呼叫一般函數開啟 LED
    delay(1000);
    statusLed.turnOff();  // 呼叫一般函數關閉 LED
}
```
上述程式碼：
- 更容易使用（自動初始化）
- 更不容易出錯（確保正確初始化）
- 更容易維護（初始化邏輯集中在一處）

## 1.5 結構創建變數
### 1.5.1 LED 控制系統
```cpp
// 方法 1：分開寫
struct LEDState {
    bool isOn;
    uint8_t brightness;
    unsigned long lastChangeTime;
};

class LEDController {
private:
    LEDState ledState;  // 在需要時宣告變數
    int ledPin;
    
public:
    LEDController(int pin) {
        ledPin = pin;
        ledState.isOn = false;
        ledState.brightness = 0;
        ledState.lastChangeTime = 0;
    }
};

// 方法 2：合併寫
class LEDController {
private:
    struct LEDState {
        bool isOn = false;
        uint8_t brightness = 0;
        unsigned long lastChangeTime = 0;
    } ledState;  // 直接創建變數並使用預設值
    int ledPin;
    
public:
    LEDController(int pin) : ledPin(pin) { }  // 結構已經自動初始化
};
```
### 1.5.2 溫度感測器系統
```cpp
// 方法 1：分開寫 - 適合需要多個感測器的情況
struct SensorData {
    float temperature;
    float humidity;
    unsigned long readTime;
    bool isValid;
};

class TemperatureMeter {
private:
    SensorData sensor1;
    SensorData sensor2;  // 可以輕易創建多個實例
    SensorData sensor3;
};

// 方法 2：合併寫 - 適合只需要單一感測器的情況
class TemperatureMeter {
private:
    struct SensorData {
        float temperature = 0.0f;
        float humidity = 0.0f;
        unsigned long readTime = 0;
        bool isValid = false;
    } sensorData;  // 直接創建單一實例
};
```
### 1.5.3 計時器系統
```cpp
// 方法 1：分開寫 - 結構可以在不同地方重用
struct Timer {
    unsigned long startTime;
    unsigned long duration;
    bool isRunning;
    bool isComplete;
};

// 可以在不同的類中使用
class StopWatch {
    Timer timer;
};

class Countdown {
    Timer timer;
};

// 方法 2：合併寫 - 結構與特定用途緊密綁定
class StopWatch {
private:
    struct StopWatchTimer {
        unsigned long startTime = 0;
        unsigned long duration = 0;
        bool isRunning = false;
        bool isComplete = false;
        bool isPaused = false;  // 可以加入特定於這個用途的額外屬性
    } timer;
};
```
### 1.5.4 優缺點比較：
#### 分開寫
優點：
1. 結構定義更清晰，易於閱讀
2. 可以在多個地方重用同一個結構
3. 更容易進行文檔註解
4. 便於創建多個實例
5. 程式碼組織更有彈性

缺點：
1. 需要額外的初始化步驟
2. 程式碼稍微冗長一些
3. 可能需要在多處維護預設值

### 合併寫
優點：
1. 程式碼更緊湊
2. 可以直接設定預設值
3. 變數宣告和結構定義在一起，關係明確
4. 適合只需要單一實例的情況
5. 可以直接在結構中加入特定用途的成員

缺點：
1. 結構難以重用
2. 可能較難閱讀和理解
3. 不適合需要多個實例的情況
4. 文檔註解可能較難組織

#### 使用建議：
1. 當需要在多個地方使用相同的數據結構時，需要重用性，選擇分開寫方法
```cpp
struct Point {
    int x;
    int y;
};
// 可以在多處使用
Point startPoint;
Point endPoint;
Point controlPoints[10];
```
2. 當結構僅用於特定類別的內部實現時，需要封裝性，選擇合併寫方法
```cpp
class GameCharacter {
private:
    struct Stats {
        int health = 100;
        int mana = 100;
        int level = 1;
    } characterStats;
};
```
3. 根據初始化需求選擇：
```cpp
// 如果需要動態初始化，使用方法 1
struct ConfigData {
    int value;
    String name;
};
ConfigData config;
config.value = readFromStorage();
config.name = getUserInput();

// 如果有固定的預設值，使用方法 2
struct {
    int value = 42;
    String name = "default";
} config;
```

# 2. enum 列舉
# 3. 函數設計
# 4. 作用域和生命週期
作用域和生命週期的重要性：
1. 記憶體管理：
- 區域變數在不需要時自動釋放記憶體
- 避免不必要的全域變數佔用記憶體
2. 程式可靠性：
- 適當的作用域可以防止變數被意外修改
- 清晰的生命週期有助於避免記憶體洩漏
3. 代碼組織：
- 有助於編寫更容易維護的程式
- 減少變數名稱衝突

## 4.1 作用域 (Scope)
想像你正在一棟有多個房間的房子裡。每個房間就像程式中的不同作用域，而變數就像房間裡的物品。
```cpp
int globalLight = 100;  // 全域變數，像是房子的大門口的燈

void setup() {
    int roomLight = 50;  // 區域變數，只在 setup 這個"房間"裡可見
    
    if (roomLight > 30) {
        int nightLight = 10;  // 更小的作用域，只在這個 if 區塊內可見
        globalLight = nightLight;  // 可以改變全域變數
    }
    // 這裡無法使用 nightLight
    
    Serial.println(roomLight);    // 可以使用
    Serial.println(globalLight);  // 可以使用
    // Serial.println(nightLight);  // 錯誤！超出作用域
}

void loop() {
    // Serial.println(roomLight);    // 錯誤！無法看到 setup 裡的變數
    Serial.println(globalLight);  // 可以使用全域變數
}
```
### 4.1.1 全域作用域：
```cpp
int globalVar = 0;  // 全域變數，整個程式都可以使用

void function1() {
    globalVar = 1;  // 可以使用
}

void function2() {
    globalVar = 2;  // 也可以使用
}
```
### 4.1.2 函數作用域：
```cpp
void someFunction() {
    int localVar = 10;  // 區域變數，只在這個函數內有效
    
    for (int i = 0; i < 5; i++) {
        localVar++;  // 可以使用函數內的變數
    }
}
// localVar 在這裡已經不存在了
```
### 4.1.3 區塊作用域：
```cpp
void checkTemperature() {
    int temp = 25;
    
    if (temp > 20) {
        int warning = 1;  // 只在 if 區塊內有效
        Serial.println(warning);
    }
    // warning 在這裡已經不能使用了
    
    Serial.println(temp);  // 仍然可以使用
}
```

## 4.2 生命週期 (Lifetime)
生命週期就像物品在房間裡存在的時間。讓我們看看不同類型變數的生命週期：
### 4.2.1 全域變數的生命週期：
```cpp
int systemUptime = 0;  // 整個程式執行期間都存在

void setup() {
    systemUptime = 1;  // 可以使用
}

void loop() {
    systemUptime++;    // 一直都存在
}
```
### 4.2.2 區域變數的生命週期：
```cpp
void countDown() {
    int counter = 10;  // 每次函數呼叫時建立
    
    while (counter > 0) {
        Serial.println(counter);
        counter--;
    }
}   // counter 的生命在這裡結束
```
### 4.2.3 靜態區域變數：
```cpp
void countButton() {
    static int pressCount = 0;  // 只初始化一次，但保持值
    
    pressCount++;  // 值會保存到下次函數呼叫
    Serial.println(pressCount);
}
```
### 4.2.4 實際應用例子
```cpp
class ButtonHandler {
private:
    static const int DEBOUNCE_TIME = 50;  // 靜態常數
    int buttonPin;                        // 類別成員變數
    
    void handlePress() {
        static unsigned long lastPressTime = 0;  // 靜態區域變數
        unsigned long currentTime = millis();    // 區域變數
        
        if (currentTime - lastPressTime > DEBOUNCE_TIME) {
            // 處理按鈕事件
            lastPressTime = currentTime;
        }
    }
};
```
