---
layout: post
title: Visual assistance system for the blind
class: 電機4B
student ID: (01072114)
name: 張家豪
---

# <span style="font-size:50px;"><b>EdgeAI 微控制應用期末專題實作報告</b></span>

## 一、EdgeAI MCU 系統架構
本系統由 AMB82-mini 為主控核心，搭配多個模組實現人工智慧邊緣運算應用。系統整體架構如下：

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/%E5%9C%961.%20AMB82-mini%E7%B3%BB%E7%B5%B1%E6%9E%B6%E6%A7%8B%E5%9C%96.png?raw=true"></p>
<p align="center">圖1 AMB82-Mini系統架構圖</p>

- AMB82-mini：具備雙核心 CPU 及 Wi-Fi 功能，支援 TensorFlow Lite Micro 與 OpenAI API。

- ILI9341 TFT LCD：負責顯示圖像、辨識結果或系統狀態。

- PAM8403 + 4Ω 3W Speaker：進行語音播放，傳遞 TTS 結果。

- Camera (內建)：每分鐘擷取影像，送至 Gemini Vision 模型。

- MicroSD 卡：儲存辨識結果與對應影像。

- RTC 模組（若接入）：確保時間戳記準確。

模組彼此透過 SPI、PWM 與 GPIO 接腳整合，並透過 Wi-Fi 連網串接生成式 AI 服務。

### 1. AMB82-mini
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/%E5%9C%961.%20AMB82-Mini%E5%AF%A6%E9%AB%94%E5%A4%96%E8%A7%80%E5%9C%96.png?raw=true"></p>
<p align="center">圖2 AMB82-Mini實體外觀圖</p>
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/%E5%9C%963.%20AMB82-Mini%20%E9%96%8B%E7%99%BC%E6%9D%BF%E5%89%8D%E5%BE%8C%E8%A6%96%E5%9C%96%E8%88%87%20GPIO%20%E8%85%B3%E4%BD%8D%E5%8A%9F%E8%83%BD%E5%B0%8D%E6%87%89%E5%9C%96.png?raw=true"></p>
<p align="center">圖3 AMB82-Mini主要介面說明圖</p>
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/%E5%9C%964.%20ILI9341%20TFT%20LCD%E5%AF%A6%E9%AB%94%E5%A4%96%E8%A7%80%E5%9C%96.png?raw=true"></p>
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

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/%E5%9C%964.%20ILI9341%20TFT%20LCD%E5%AF%A6%E9%AB%94%E5%A4%96%E8%A7%80%E5%9C%96.png?raw=true"></p>
<p align="center">圖5 ILI9341 TFT LCD實體外觀圖</p>
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/%E5%9C%965.AMB82%20MINI%20and%20QVGA%20TFT%20LCD%20%E6%8E%A5%E7%B7%9A%E5%9C%96.png?raw=true"></p>
<p align="center">圖6 AMB82 MINI and QVGA TFT LCD 接線圖</p>


<p align="center">表1 ILI9341 TFT LCD 規格</p>

<div align="center">
	
<table>
  <thead>
    <tr>
      <th>項目</th>
      <th>說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>顯示尺寸</td>
      <td>2.4 吋 / 2.8 吋 TFT LCD</td>
    </tr>
    <tr>
      <td>解析度</td>
      <td>240 × 320 pixels</td>
    </tr>
    <tr>
      <td>控制晶片</td>
      <td>ILI9341</td>
    </tr>
    <tr>
      <td>通訊介面</td>
      <td>SPI（支援 4 線序列通訊，亦可設定為 8-bit 並列）</td>
    </tr>
    <tr>
      <td>顯示色彩</td>
      <td>262K（18-bit）真彩顯示</td>
    </tr>
    <tr>
      <td>操作電壓</td>
      <td>3.3V（邏輯電平，部分模組具備 5V 轉接）</td>
    </tr>
    <tr>
      <td>觸控功能</td>
      <td>可選（部分模組具電阻式/電容式觸控，搭配 XPT2046）</td>
    </tr>
    <tr>
      <td>背光模組</td>
      <td>LED 背光，PWM 可調整亮度</td>
    </tr>
  </tbody>
</table>

</div>


<p align="center">表2 ILI9341 TFT LCD 模組引腳定義與功能說明表</p>

<div align="center">
	
<table>
  <thead>
    <tr>
      <th>序號</th>
      <th>引腳標號</th>
      <th>說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>VCC</td>
      <td>5V/3.3V電源輸入</td>
    </tr>
    <tr>
      <td>2</td>
      <td>GND</td>
      <td>接地</td>
    </tr>
    <tr>
      <td>3</td>
      <td>CS</td>
      <td>液晶屏片選信號，低電平使能</td>
    </tr>
    <tr>
      <td>4</td>
      <td>RESET</td>
      <td>液晶屏重定信號，低電平重定</td>
    </tr>
    <tr>
      <td>5</td>
      <td>DC/RS</td>
      <td>液晶屏寄存器/資料選擇信號，低電平：寄存器，高電平：數據</td>
    </tr>
    <tr>
      <td>6</td>
      <td>SDI(MOSI)</td>
      <td>SPI匯流排寫資料信號</td>
    </tr>
    <tr>
      <td>7</td>
      <td>SCK</td>
      <td>SPI匯流排時鐘信號</td>
    </tr>
    <tr>
      <td>8</td>
      <td>LED</td>
      <td>背光控制，高電平點亮，如無需控制則接3.3V常亮</td>
    </tr>
    <tr>
      <td>9</td>
      <td>SDO(MISO)</td>
      <td>SPI匯流排讀數據信號，如無需讀取功能則可不接</td>
    </tr>
    <tr>
      <td></td>
      <td></td>
      <td>(以下為觸摸屏信號線接線，如無需觸摸或者模組本身不帶觸摸功能，可不連接)</td>
    </tr>
    <tr>
      <td>10</td>
      <td>T_CLK</td>
      <td>觸摸SPI匯流排時鐘信號</td>
    </tr>
    <tr>
      <td>11</td>
      <td>T_CS</td>
      <td>觸摸屏片選信號，低電平使能</td>
    </tr>
    <tr>
      <td>12</td>
      <td>T_DIN</td>
      <td>觸摸SPI匯流排輸入</td>
    </tr>
    <tr>
      <td>13</td>
      <td>T_DO</td>
      <td>觸摸SPI匯流排輸出</td>
    </tr>
    <tr>
      <td>14</td>
      <td>T_IRQ</td>
      <td>觸摸屏中斷信號，檢測到觸摸時為低電平</td>
    </tr>
  </tbody>
</table>

</div>

### 3. PAM8403
PAM8403 是一款低功耗、高效率的 D 類音訊放大器晶片，可提供最高 3W + 3W 雙聲道立體聲輸出，適用於驅動如 4Ω/3W 或 8Ω/2W 的小型喇叭。它常被整合為 小型擴大器模組，可直接與微控制器（如 AMB82-Mini）搭配，用於播放語音合成（TTS）或音樂訊號。

<p align="center">表3 PAM8403產品特性</p>

<div align="center">
	
<table>
  <thead>
    <tr>
      <th>項目</th>
      <th>規格說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>工作電壓</td>
      <td>2.5V ~ 5.5V</td>
    </tr>
    <tr>
      <td>最大輸出功率</td>
      <td>3W × 2（於 5V、4Ω 負載）</td>
    </tr>
    <tr>
      <td>音訊輸入</td>
      <td>模擬音訊（L/R 左右聲道輸入）</td>
    </tr>
    <tr>
      <td>控制方式</td>
      <td>無需 MCU 控制，可直接使用 PWM 或 DAC 輸入</td>
    </tr>
    <tr>
      <td>音質表現</td>
      <td>低 THD（總諧波失真）與雜訊，音質清晰</td>
    </tr>
    <tr>
      <td>功耗特性</td>
      <td>高效率（&gt;85%），待機電流極低</td>
    </tr>
    <tr>
      <td>大小</td>
      <td>模組極小（約 2×2 cm），易於整合至小型裝置</td>
    </tr>
  </tbody>
</table>

</div>


<p align="center">表4 PAM8403腳位定義</p>

<div align="center">
	
<table>
  <thead>
    <tr>
      <th>項目</th>
      <th>規格說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>阻抗（Impedance）</td>
      <td>4Ω（歐姆）</td>
    </tr>
    <tr>
      <td>額定功率（Rated Power）</td>
      <td>3W</td>
    </tr>
    <tr>
      <td>響應頻率範圍</td>
      <td>約 200Hz ~ 10kHz（視型號而定）</td>
    </tr>
    <tr>
      <td>音壓靈敏度</td>
      <td>約 85 ~ 90 dB（1W/1m）</td>
    </tr>
    <tr>
      <td>直徑尺寸</td>
      <td>常見尺寸為 36mm / 40mm / 長條型</td>
    </tr>
    <tr>
      <td>結構類型</td>
      <td>有紙盆、塑膠盆、防塵網、磁鐵等結構</td>
    </tr>
    <tr>
      <td>音圈材質</td>
      <td>銅線音圈 / 鋁音圈</td>
    </tr>
  </tbody>
</table>


</div>


### 4. 4ohm 3W speaker
4Ω 3W 喇叭是一款常見於嵌入式系統、智慧裝置與音訊模組中的 小功率聲音輸出元件，具備結構緊湊、阻抗與功率規格標準化、易於搭配音訊擴大器模組（如 PAM8403）等優點。此類喇叭設計用於近距離音訊播放，適合語音提示、簡易音樂與 TTS（Text-to-Speech）語音輸出等應用。

<p align="center">表5 4ohm 3W speaker規格參數</p>

<div align="center">
	
<table>
  <thead>
    <tr>
      <th>項目</th>
      <th>規格說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>阻抗（Impedance）</td>
      <td>4Ω（歐姆）</td>
    </tr>
    <tr>
      <td>額定功率（Rated Power）</td>
      <td>3W</td>
    </tr>
    <tr>
      <td>響應頻率範圍</td>
      <td>約 200Hz ~ 10kHz（視型號而定）</td>
    </tr>
    <tr>
      <td>音壓靈敏度</td>
      <td>約 85 ~ 90 dB（1W/1m）</td>
    </tr>
    <tr>
      <td>直徑尺寸</td>
      <td>常見尺寸為 36mm / 40mm / 長條型</td>
    </tr>
    <tr>
      <td>結構類型</td>
      <td>有紙盆、塑膠盆、防塵網、磁鐵等結構</td>
    </tr>
    <tr>
      <td>音圈材質</td>
      <td>銅線音圈 / 鋁音圈</td>
    </tr>
  </tbody>
</table>

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

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/GenAI%20%E7%B7%A8%E7%A2%BC%E8%A8%AD%E8%A8%88%E6%B5%81%E7%A8%8B.png?raw=true"></p>
<p align="center">圖7 GenAI 編碼設計流程</p>

## 三、程式生成提示語設計

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

## 四、盲人視覺輔助系統專案流程圖

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/%E7%9B%B2%E4%BA%BA%E8%A6%96%E8%A6%BA%E8%BC%94%E5%8A%A9%E7%B3%BB%E7%B5%B1%E5%B0%88%E6%A1%88%E6%B5%81%E7%A8%8B%E5%9C%96.png?raw=true"></p>
<p align="center">圖8 盲人視覺輔助系統專案流程圖</p>

## 五、盲人視覺輔助系統程式碼與說明

### 1.作業目標：
整合以下 4 項功能，建立一個可以進行感測、影像辨識、時間推理、語音互動的智慧系統，使用樣例程式作為參考，完成整合應用程式。

### 2.功能說明：
#### (一)觸控（Touch）功能 — ADC 模擬輸入
使用 類比輸入（Analog Input） 偵測觸控狀態（可用手指接觸金屬片或電阻式觸控感測）。

範例參考程式：

examples > 03. Analog > AnalogInput.ino

用途：透過觸控觸發不同功能模式（例如每觸一次切換功能）。

#### (二)拍照並詢問 Gemini 場景辨識
使用攝影機模組拍照，並把圖片傳送給 Google Gemini Vision API。

從回傳結果中獲得對當前場景的描述。

範例參考程式：

GenAIVision_TTS.ino

用途：辨識你面前的東西，並用文字描述它。

#### (三)傳送 RTC 時間給 Gemini 並生成文字
使用 RTC 模組讀取實際時間資訊（年月日與時間）。

傳送給 Gemini Text API，請它根據時間生成一段有趣的描述或敘述。

範例參考程式：

examples > AmebaRTC > Simple_RTC.ino

用途：像是「現在是幾點鐘」→ Gemini 回答：「早上八點，是個適合喝咖啡的時刻」。

#### (四)錄音後傳送語音給 Gemini 並轉語音播放
使用麥克風錄音。

將音訊檔案傳送給 Gemini Audio API，進行語音辨識轉成文字。

再用 Google TTS（文字轉語音） 播放出來。

範例參考程式：

GenAISpeech.ino

用途：你說一句話 → 系統將語音轉成文字，再唸出來（語音回應）。

### 3.整合應用建議：
你可以設計成 按一次觸控切換一個模式：

第一次觸控 ➜ 啟動 Vision 場景辨識模式

第二次觸控 ➜ 啟動 RTC 時間解釋模式

第三次觸控 ➜ 啟動語音錄音＋轉文字＋TTS 模式

第四次觸控 ➜ 回到初始或輪迴模式

### 4.實作重點：
每個功能模組可以用樣例程式測試完成後再進行整合。

整合時需注意：

函式的呼叫與切換流程

LCD 顯示（若有）、MP3 播放、SD 儲存等額外功能配合使用

各模組之間的資源（如 memory、pin 腳、串流）不可衝突

<b>Code:</b>
```
#include <Arduino.h>
#include <WiFi.h>
#include <WiFiUdp.h>
#include "rtc.h"  // 引入 rtc.h，假設它是 Realtek SDK 中的 RTC 模組
#include "GenAI.h"
#include "AmebaFatFS.h"
#include "MP4Recording.h"

// WiFi 設置
const char* ssid = "hahaha"; 
const char* password = "93034570";

// RTC 設置
RTCClass rtc;  // 使用之前定義的 RTCClass

// GenAI 物件
GenAI gemini;

// 觸控相關設置
int TouchPin = 34;  // 假設使用的觸控引腳

void setup() {
  // 設定序列埠
  Serial.begin(115200);

  // 連接 WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.println("正在連接 WiFi...");
  }
  Serial.println("WiFi 連接成功!");

  // RTC 初始化
  rtc.Init();  // 初始化 RTC

  // 設定觸控引腳為輸入
  pinMode(TouchPin, INPUT);
}

void loop() {
  // 檢查觸控事件
  checkTouch();

  // 捕獲並顯示圖像，並從 Gemini 獲取描述
  captureAndShowImage();

  // 獲取並顯示當前時間，並發送給 Gemini 進行處理
  getCurrentTimeAndSend();

  // 錄音並將錄音轉換為文字，然後用 TTS 朗讀
  recordAndSpeak();
  
  delay(1000);  // 控制主循環延遲
}

// 檢查觸控輸入
void checkTouch() {
  if (digitalRead(TouchPin) == LOW) {
    Serial.println("觸控偵測到! 切換模式...");
    // 你可以在此處添加模式切換代碼
  }
}

// 捕獲圖像並向 Gemini 发送，並顯示描述
void captureAndShowImage() {
  Serial.println("正在捕獲圖像...");
  
  uint8_t* jpgBuffer;
  size_t jpgSize;
  
  // 假設這裡有捕獲圖像的程式碼，並將圖像保存在 jpgBuffer 和 jpgSize 中
  // camera.captureJPEG(&jpgBuffer);  // 根據你的硬體設置進行圖像捕獲
  
  // 向 Gemini 請求場景描述
  getGeminiVisionDescription(jpgBuffer, jpgSize);
}

// 從 Gemini 獲取圖像的場景描述
void getGeminiVisionDescription(uint8_t* jpgBuffer, size_t jpgSize) {
  Serial.println("獲取場景描述...");
  
  // 向 Gemini 發送圖像數據並獲取描述
  String message = "請描述這張圖片";  // 可以根據需求自定義訊息
  String description = gemini.geminivision("AIzaSyCCwbt-JZVF_sdc2Eos6A8KipWZmjupvQk", "模型名稱", message, (uint32_t)jpgBuffer, jpgSize, WiFiClient());
  
  Serial.println("描述: " + description);
  speakDescription(description);  // 朗讀描述
}

// 朗讀描述文字
void speakDescription(String description) {
  Serial.println("正在朗讀描述...");
  gemini.googletts("output.mp3", description, "zh");  // "zh" 表示中文，可以根據需要更改語言代碼
}

// 獲取 RTC 時間並發送給 Gemini
void getCurrentTimeAndSend() {
  // 使用 rtc.h 提供的 rtc_read 函數來獲取時間
  long long current_time = rtc.Read();
  
  // 提取時間組件（這部分根據 RTC 的實際行為進行調整）
  int hour = (current_time / (60 * 60)) % 24;
  int min = (current_time / 60) % 60;
  int sec = current_time % 60;
  
  String timeMessage = "當前時間: " + String(hour) + ":" + String(min) + ":" + String(sec);
  String response = gemini.geminitext("AIzaSyCCwbt-JZVF_sdc2Eos6A8KipWZmjupvQk", "模型名稱", timeMessage, WiFiClient());
  
  Serial.println("時間響應: " + response);
}

// 錄音並轉換為文字，然後用 TTS 語音回應
void recordAndSpeak() {
  // 假設有錄音並轉換為文字的代碼
  String audioMessage = "錄音已完成，開始轉換為文字";
  String textMessage = gemini.whisperaudio("你的API密鑰", "API服務器地址", "api路徑", "whisper模型", "錄音文件名", WiFiClient());
  
  Serial.println("錄音轉文字: " + textMessage);
  gemini.googletts("output_audio.mp3", textMessage, "zh");  // 播放錄音轉換的文本
}

```

## 六、盲人視覺輔助系統成果展示
編譯失敗

<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*
