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

# 2. enum 列舉
# 3. 函數設計
# 4. 作用域和生命週期
