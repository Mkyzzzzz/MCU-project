---
layout: post
title: AI-assisted recycling sorting system
class: 電機4B
student ID: 01072114
name: 張家豪
---

# <p align="center"><p align="center"><span style="font-size:50px;"><b>EdgeAI 微控制應用期末專題實作報告</b></span></p>

## 一、EdgeAI MCU 系統架構
本系統由 AMB82-mini 為主控核心，搭配多個模組實現人工智慧邊緣運算應用。系統整體架構如下：

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/%E5%9C%961.%20AMB82-mini%E7%B3%BB%E7%B5%B1%E6%9E%B6%E6%A7%8B%E5%9C%96.png"></p>
<p align="center">圖1 AMB82-Mini系統架構圖</p>

- AMB82-mini：具備雙核心 CPU 及 Wi-Fi 功能，支援 TensorFlow Lite Micro 與 OpenAI API。

- ILI9341 TFT LCD：負責顯示圖像、辨識結果或系統狀態。

- PAM8403 + 4Ω 3W Speaker：進行語音播放，傳遞 TTS 結果。

- Camera (內建)：每分鐘擷取影像，送至 Gemini Vision 模型。

- MicroSD 卡：儲存辨識結果與對應影像。
- 
- RTC 模組（若接入）：確保時間戳記準確。

模組彼此透過 SPI、PWM 與 GPIO 接腳整合，並透過 Wi-Fi 連網串接生成式 AI 服務。

### 1. AMB82-mini
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/%E5%9C%961.%20AMB82-Mini%E5%AF%A6%E9%AB%94%E5%A4%96%E8%A7%80%E5%9C%96.png"></p>
<p align="center">圖2 AMB82-Mini實體外觀圖</p>
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/%E5%9C%962.%20AMB82-Mini%E4%B8%BB%E8%A6%81%E4%BB%8B%E9%9D%A2%E8%AA%AA%E6%98%8E%E5%9C%96.png"></p>
<p align="center">圖3 AMB82-Mini主要介面說明圖</p>
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/%E5%9C%963.%20AMB82-Mini%20%E9%96%8B%E7%99%BC%E6%9D%BF%E5%89%8D%E5%BE%8C%E8%A6%96%E5%9C%96%E8%88%87%20GPIO%20%E8%85%B3%E4%BD%8D%E5%8A%9F%E8%83%BD%E5%B0%8D%E6%87%89%E5%9C%96.png"></p>
<p align="center">圖4 AMB82-Mini 開發板前後視圖與 GPIO 腳位功能對應圖</p>

圖1展示了 AMB82-Mini 的實體外觀，其中左側為搭載鏡頭的主開發板，右側為擴充模組，兩者皆具備豐富的 GPIO 排針與無線通訊模組。該圖有助於辨識實體接腳位置、連接模組佈局與實體安裝方向，是開發初期理解板子配置的第一步。鏡頭模組的存在也預示了該開發板具備即時影像擷取與傳送的能力，對於後續進行圖像辨識等 AI 應用至關重要。
      
圖2進一步聚焦於 AMB82-Mini 主板下方的三個 USB 介面與一個 RESET 按鈕，分別對應 UART_DOWNLOAD、USB OTG 與 Micro USB 功能，是韌體燒錄、USB 裝置掛載與序列埠通訊的關鍵接口。這張圖對於軟體開發者和系統整合者來說特別重要，因為錯誤的連接方式可能導致無法下載程式、裝置無法啟動或電力不穩。
      
圖3則提供最為關鍵的 GPIO 腳位分佈與功能對應說明，包含前後兩面的腳位排列，並以顏色分類標示各腳位用途，例如 SPI、I2C、UART、PWM、ADC、GPIO 等。開發者可依據此圖完成模組整合與接腳設計，例如將 ILI9341 TFT LCD 透過 SPI 腳位接入，或將麥克風與喇叭透過 I2S、PWM 控制端連接，此外也能依據腳位分配規劃按鈕輸入或感測器模組串接。

#### §  GPIO INT（General-Purpose Input/Output with Interrupts）
支援中斷觸發的 GPIO 腳位可用於偵測電壓變化（如上升緣/下降緣）並自動呼叫中斷函數，不需輪詢（polling），節省 CPU 運算資源，實現事件驅動控制。

應用實例：

- 按鈕按壓觸發畫面切換
- PIR 感測器啟動語音警示
- 碰撞感測器觸發機械動作

對應腳位：

幾乎所有 GPIO 腳位皆支援中斷，包括 PF0 ~ PF15、PE1 ~ PE6、PD14 ~ PD18、PA0 ~ PA3

#### §  ADC（Analog to Digital Converter）

ADC 將外部模擬訊號（連續電壓）轉為數位資料，供 MCU 做進一步分析與處理。AMB82-Mini 提供 8 個 ADC 通道，最大解析度 12-bit。

應用實例：

- 讀取光敏電阻控制螢幕亮度
- 偵測土壤濕度控制自動澆水
- 監控電池電壓防止過放

對應腳位：
PF0（A0）、PF1（A1）、PF2（A2）、PF3（*A3）、PA0（A4）、PA1（A5）、PA2（A6）、PA3（A7）

#### §  PWM（Pulse Width Modulation）

以數位方式產生變化的電壓波形，用以模擬類比輸出，適合控制亮度、音訊輸出、馬達速度等。PWM 透過改變佔空比調整輸出功率。
    
應用實例：

- LED 呼吸燈效果
- 喇叭播放合成語音訊號
- 伺服馬達轉角控制

對應腳位：

PF6、PF7、PF8、PF9（LED_B）、PF12、PF13

#### §  UART（Universal Asynchronous Receiver/Transmitter）
UART 為非同步序列通訊，提供開發板與外部裝置（如藍牙模組、GPS、PC）之間的資料傳輸。AMB82-Mini 支援多組 UART（Serial1–3、LOG）。
    
應用實例：

- 與電腦透過 LOG_TX/RX 傳送 debug 資訊
- 與 HC-05 藍牙模組資料傳輸
- 接收 GPS 定位資訊

對應腳位：

PA2（SERIAL1_TX）、PA3（SERIAL1_RX）、PD15（SERIAL2_TX）、PD16（SERIAL2_RX）、PE1（SERIAL3_TX）、PE2（SERIAL3_RX）、PF4（LOG_TX）、PF3（LOG_RX) 

#### §  SPI（Serial Peripheral Interface）
SPI 為同步序列通訊，速度快，適用於 TFT 顯示器、SD 卡、Flash 等高速裝置。AMB82-Mini 提供兩組 SPI，包含 SPI 與 SPI1。

應用實例：

- 控制 ILI9341 TFT LCD 顯示畫面
- 儲存影像資料至 MicroSD 卡模組
- 外接 Flash 記憶體儲存語音模型

對應腳位：

PF5（SPI1_MISO）、PF6（SPI1_SCLK）、PF7（SPI1_MOSI）、PF8（SPI1_SS）、PE3（SPI_MOSI）、PE2（SPI_MISO）、PE1（SPI_SCLK）、PE4（SPI_SS）
#### §  I2C（Inter-Integrated Circuit）
I2C 為雙線式同步通訊協定，可連接多個從屬裝置如感測器、OLED 等。AMB82-Mini 提供兩組 I2C 介面。
    
應用實例：

- 連接 MPU6050 姿態感測器
- 讀取 SHT31 溫濕度感測值
- 多感測模組串接（透過 TCA9548 擴展）

對應腳位：

PF2（I2C1_SDA）、PF1（I2C1_SCL）、PA1（I2C2_SDA）、PA0（I2C2_SCL）、PE4（I2C_SDA）、PE3（I2C_SCL）

#### §  SWD（Serial Wire Debug）

SWD 為 ARM Cortex-M 的單線除錯介面，用於實現斷點、變數觀察與即時燒錄。適合進行嵌入式除錯與韌體開發。
    
應用實例：

- 使用 J-Link 進行即時中斷除錯
- 使用 OpenOCD 觀察變數、設定斷點
- 韌體 OTA 更新前進行 low-level debug

對應腳位：

PA0（SWD_DATA）、PA1（SWD_CLK）

⚠ 注意：若啟用 SWD 模式，A4/A5 與 I2C2 將無法同時使用。

#### § LED（On-Board LED Control）

板載 LED 可作為系統執行狀態、網路連線、錯誤警告等視覺提示，亦可當作普通 GPIO 控制。

應用實例：

- 網路連線成功後綠燈閃爍
- 辨識結果語音播放時藍燈同步閃爍
- 錯誤發生時 LED 閃爍提示

對應腳位：

PF9：LED_BUILTIN / LED_B（藍燈）、PE6：LED_G（綠燈）

### 2. ILI9341 TFT LCD
ILI9341 是一款廣泛應用於嵌入式系統的 2.4 吋/2.8 吋彩色 TFT LCD 顯示模組，搭載 240×320 像素解析度 與 SPI 通訊介面。該模組內建 ILI9341 顯示驅動 IC，支援 262K 色彩顯示與圖形加速功能，能夠顯示影像、文字、圖形介面，常見於智慧手持裝置、嵌入式儀表與 IoT 應用。

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/%E5%9C%964.%20ILI9341%20TFT%20LCD%E5%AF%A6%E9%AB%94%E5%A4%96%E8%A7%80%E5%9C%96.png"></p>
<p align="center">圖5 ILI9341 TFT LCD實體外觀圖</p>
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/%E5%9C%965.AMB82%20MINI%20and%20QVGA%20TFT%20LCD%20%E6%8E%A5%E7%B7%9A%E5%9C%96.png"></p>
<p align="center">圖6 AMB82 MINI and QVGA TFT LCD 接線圖</p>


<p align="center">表1 ILI9341 TFT LCD 規格</p>

<div align="center">
	
| 項目    | 說明                                          |
| --------|----------------------------------------------|
| 顯示尺寸 | 2.4 吋 / 2.8 吋 TFT LCD                      |
| 解析度   | 240 × 320 pixels                             |
| 控制晶片 | ILI9341                                      |
| 通訊介面 | SPI（支援 4 線序列通訊，亦可設定為 8-bit 並列） |
| 顯示色彩 | 262K（18-bit）真彩顯示                        |
| 操作電壓 | 3.3V（邏輯電平，部分模組具備 5V 轉接）          |
| 觸控功能 | 可選（部分模組具電阻式/電容式觸控，搭配 XPT2046）|
| 背光模組 | LED 背光，PWM 可調整亮度                       |

</div>


<p align="center">表2 ILI9341 TFT LCD 模組引腳定義與功能說明表</p>

<div align="center">
	
|序號	|引腳標號|	說明|
|-------|-------|-----------|
|1	|VCC	|5V/3.3V電源輸入|
|2	|GND	|接地|
|3	|CS	|液晶屏片選信號，低電平使能|
|4	|RESET	|液晶屏重定信號，低電平重定|
|5	|DC/RS	|液晶屏寄存器/資料選擇信號，低電平：寄存器，高電平：數據|
|6	|SDI(MOSI)|	SPI匯流排寫資料信號|
|7	|SCK	|SPI匯流排時鐘信號|
|8	|LED	|背光控制，高電平點亮，如無需控制則接3.3V常亮|
|9	|SDO(MISO)	|SPI匯流排讀數據信號，如無需讀取功能則可不接|
|	|	|(以下為觸摸屏信號線接線，如無需觸摸或者模組本身不帶觸摸功能，可不連接)|
|10	|T_CLK	|觸摸SPI匯流排時鐘信號|
|11	|T_CS	|觸摸屏片選信號，低電平使能|
|12	|T_DIN	|觸摸SPI匯流排輸入|
|13	|T_DO	|觸摸SPI匯流排輸出|
|14	|T_IRQ	|觸摸屏中斷信號，檢測到觸摸時為低電平|

</div>

### 3. PAM8403
PAM8403 是一款低功耗、高效率的 D 類音訊放大器晶片，可提供最高 3W + 3W 雙聲道立體聲輸出，適用於驅動如 4Ω/3W 或 8Ω/2W 的小型喇叭。它常被整合為 小型擴大器模組，可直接與微控制器（如 AMB82-Mini）搭配，用於播放語音合成（TTS）或音樂訊號。

<p align="center">表3 PAM8403產品特性</p>

<div align="center">
	
|項目	|規格說明|
|-------|-------|
|工作電壓	|2.5V ~ 5.5V|
|最大輸出功率	|3W × 2（於 5V、4Ω 負載）|
|音訊輸入	|模擬音訊（L/R 左右聲道輸入）|
|控制方式	|無需 MCU 控制，可直接使用 PWM 或 DAC 輸入|
|音質表現	|低 THD（總諧波失真）與雜訊，音質清晰|
|功耗特性	|高效率（>85%），待機電流極低|
|大小	|模組極小（約 2×2 cm），易於整合至小型裝置|

</div>


<p align="center">表4 PAM8403腳位定義</p>

<div align="center">
	
|腳位名稱|	功能說明|
|-------|--------------|
|VCC	|電源正極（建議供應 5V）|
|GND	|電源地|
|L_IN	|左聲道音訊輸入（模擬/PWM）|
|R_IN	|右聲道音訊輸入（模擬/PWM）|
|L_OUT+ / L_OUT−	|左聲道喇叭輸出（差動）|
|R_OUT+ / R_OUT−	|右聲道喇叭輸出（差動）|
|EN（可選）	|啟用腳，低電平時模組進入待機模式（部分版本）|

</div>


### 4. 4ohm 3W speaker
4Ω 3W 喇叭是一款常見於嵌入式系統、智慧裝置與音訊模組中的 小功率聲音輸出元件，具備結構緊湊、阻抗與功率規格標準化、易於搭配音訊擴大器模組（如 PAM8403）等優點。此類喇叭設計用於近距離音訊播放，適合語音提示、簡易音樂與 TTS（Text-to-Speech）語音輸出等應用。

<p align="center">表5 4ohm 3W speaker規格參數</p>

<div align="center">
	
|項目	|規格說明|
|-------|--------|
|阻抗（Impedance）|	4Ω（歐姆）|
|額定功率（Rated Power）|	3W|
|響應頻率範圍	|約 200Hz ~ 10kHz（視型號而定）|
|音壓靈敏度	|約 85 ~ 90 dB（1W/1m）|
|直徑尺寸	|常見尺寸為 36mm / 40mm / 長條型|
|結構類型	|有紙盆、塑膠盆、防塵網、磁鐵等結構|
|音圈材質	|銅線音圈 / 鋁音圈|

</div>

## 二、GenAI 程式設計流程
在使用 GenAI 函式庫（特別針對 AMB82-MINI 開發板）進行程式設計時，整體流程可分為幾個明確的階段，根據功能需求（圖像辨識、語音辨識、文字生成、文字轉語音等）進行模組搭配。以下是 GenAI 程式設計的完整流程說明：

### 1.初始化階段

步驟:
- 引入必要的頭檔（依功能需求）
- 初始化 WiFi（連接網路）
- 初始化 GenAI 函式庫與 API Key
- 初始化設備（如 Camera、Microphone、RTC、LCD）

### 2.資料擷取階段(視應用)

根據應用目標，從不同來源擷取資料:

圖像擷取(Camera):
```
GenAI_Camera camera;
camera.begin();
camera.capture();  // 擷取一張照片
```

語音錄音(Microphone）：
```
GenAI_Audio audio;
audio.begin();
audio.record(5);  // 錄 5 秒音
```

取得時間(RTC):
```
RTC.getTimeString();  // 取得目前時間字串
```

觸控輸入(ADC）：
```
int touchValue = analogRead(TOUCH_PIN);  // 讀取觸控
```

### 3.呼叫AI模型推論

依需求選擇使用Gemini Vision、Text、Audio:

圖像辨識(Vision):
```
String result = GenAI.visionDescription(camera.getImage());
```

語音辨識:(Whisper):
```
String transcript = GenAI.transcribe(audio.getWavData());
```

指令生成、故事生成(Text):
```
String prompt = "請用圖片寫一個童話故事";
String story = GenAI.generateText(prompt);
```

搭配RTC:
```
String prompt = "現在時間是 " + RTC.getTimeString() + "，請描述天氣狀況";
String result = GenAI.generateText(prompt);
```

### 4.輸出結果(視需求)

可使用多種方式顯示/播放AI結果:

顯示在LCD:
```
lcd.print(result);
```

播放語音(TTS):
```
TTS.speak(result);  // 使用 Google TTS 朗讀
```

儲存到SD卡(選用):
```
file = SD.open("/result.txt", FILE_WRITE);
file.println(result);
file.close();
```

### 5. 控制流程與互動
可以使用:

- 按鈕 / 觸控切換模式
- 定時器 / RTC 控制週期性觸發
- 檢查 AI 回傳是否與前次不同，決定是否更新畫面/播放

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/GenAI%20%E7%B7%A8%E7%A2%BC%E8%A8%AD%E8%A8%88%E6%B5%81%E7%A8%8B.png"></p>
<p align="center">圖7 GenAI 編碼設計流程</p>

## 四、程式生成提示語設計（Prompts for Code Generation）

程式生成提示語設計（Prompts for Code Generation）是一門設計如何清楚、有效地向 AI 模型（如 GPT、Gemini、Copilot 等）描述你想要產生的程式碼的技巧。良好的提示語可以幫助你獲得準確、可執行、易維護的程式碼。

<b>為什麼提示語（Prompt）很重要？</b>

AI 是根據你提供的文字提示來推論程式碼。提示設計得越清楚，輸出的程式碼越貼近你的需求。

<b>設計程式生成提示語的 5 大原則:</b>
1.明確說明目標功能
2.指定語言與平台
3.指出輸入 / 輸出規格
4.補充使用限制 / 框架 / API
5.使用範例（很關鍵！）

<b>小總結:</b>

寫好提示語的重點是：「清楚、具體、有結構」。

- 告訴 AI 你要什麼（功能、語言、框架）
- 限制不想要的東西
- 加入範例與邏輯限制
- 越明確，越好用

## 五、AI輔助回收分類系統程式碼與說明
<b>Code:</b>

<b> AI-assisted Recycle System（AI 輔助回收系統）</b>
### 1.作業目標(Objective):

AI-assisted Recycle System

👉 使用人工智慧輔助的回收分類系統。主要功能是透過按鈕拍照，AI 辨識影像內容並語音播報，幫助使用者判斷垃圾屬於哪一類。

### 2.硬體設備(Hardware):

Development Board: AMB82-mini（MCU: Realtek RTL8735B）

👉 使用 Realtek AMB82-mini 開發板，它是一款內建攝影機、支援 Wi-Fi、具備 AI 應用能力的微控制器。

### 3.功能說明(Features):

#### (一)按下按鈕拍照
使用板上的按鈕觸發攝影機拍照。

#### (二)送出照片到 Google Gemini（Vision 模型）分析內容
利用 Google Gemini Vision AI 判斷照片裡的東西，例如「這是一個寶特瓶」或「這是一張紙」。

#### (三)把 AI 分析出來的內容，透過 Google TTS 轉成語音並播放
使用 Google Text-to-Speech (TTS) 將文字說出來，例如「這是一個可以回收的寶特瓶」。

### 4.整合專案範例（Project Examples):

這些是將多個功能整合起來的進階範例：

<div align="center">
	
|專案名稱|說明|
|-------|-----------|
|GenAIVision_TTS.ino|	拍照 → 使用 Vision AI 分析 → 使用 TTS 播放語音|
|GenAIVision_TTS_LCD.ino|	拍照 → 使用 Vision AI 分析 → 在 LCD 顯示分析結果 → 用 TTS 播放語音|

</div>

## 六、AI輔助回收分類系統成果展示

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/AI-assisted%20recycling%20sorting%20system_1.png"></p>
<p align="center">圖1 Arduino執行結果</p>
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/AI-assisted%20recycling%20sorting%20system_2.png"></p>
<p align="center">圖2 AI判定結果</p>
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/AI-assisted%20recycling%20sorting%20system_3.png"></p>
<p align="center">圖3 測試所使用的照片</p>

<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*
