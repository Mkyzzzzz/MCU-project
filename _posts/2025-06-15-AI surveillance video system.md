---
layout: post
title: AI surveillance video system
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
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/%E5%9C%963.%20AMB82-Mini%20%E9%96%8B%E7%99%BC%E6%9D%BF%E5%89%8D%E5%BE%8C%E8%A6%96%E5%9C%96%E8%88%87%20GPIO%20%E8%85%B3%E4%BD%8D%E5%8A%9F%E8%83%BD%E5%B0%8D%E6%87%89%E5%9C%96.png?raw=true">圖3 AMB82-Mini主要介面說明圖</p>
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

## 四、AI監視錄影系統專案流程圖

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/AI%E7%9B%A3%E8%A6%96%E9%8C%84%E5%BD%B1%E7%B3%BB%E7%B5%B1%E5%B0%88%E6%A1%88%E6%B5%81%E7%A8%8B%E5%9C%96.png?raw=true"></p>
<p align="center">圖8 AI監視錄影系統專案流程圖</p>

## 五、AI監視錄影系統程式碼與說明

### 1.作業目標(Objective):

使用 AMB82-mini 開發板，每分鐘自動拍照一次，將照片送給 Gemini Vision 進行場景描述。如果與上一次的場景描述不同，則將該照片與描述儲存起來（使用日期與時間作為檔案名稱）。若與上次相同，則不儲存，節省空間。

### 2.開發板與功能（Board & Function）
Board: AMB82-mini（Realtek RTL8735B）

👉 支援攝影機拍照、Wi-Fi 上傳、SD 卡儲存、RTC 實時時鐘功能。
### 3.功能流程說明（Function Flow）
#### (一)每分鐘自動拍照一次
使用 RTC（實時時鐘） 或 millis() 計時器，每 60 秒觸發一次攝影機拍照。

#### (二)照片送出給 Gemini Vision 做場景辨識
拍下來的影像上傳給 Google Gemini Vision，得到一段文字描述（例如："A park with people walking."）

#### (三)比對新回覆與上一次的文字是否相同
如果相同 → 忽略，不存圖也不存文字
如果不同 → 儲存該張 JPG 圖片與文字檔，並使用 RTC 的日期與時間命名

### d.參考範例程式碼（Sample Codes）

<div align="center">
	
<table>
  <thead>
    <tr>
      <th>範例檔名</th>
      <th>說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GenAIVision.ino</td>
      <td>拍照並將影像送給 Gemini Vision，取得場景描述文字</td>
    </tr>
    <tr>
      <td>CaptureJPG_SDcard.ino</td>
      <td>將攝影機擷取的 JPG 圖片儲存在 SD 卡</td>
    </tr>
    <tr>
      <td>examples &gt; AmebaRTC &gt; Simple_RTC.ino</td>
      <td>使用內建 RTC 時鐘模組，獲得目前的時間與日期，用於檔案命名與定時執行</td>
    </tr>
  </tbody>
</table>

</div>

<b>Code:</b>
```
/*
  This sketch captures an image every minute, sends it to Gemini Vision,
  and stores the image and description on an SD card only if the description changes.
  Filenames will include the date and time.
  Using 'gemini-1.5-flash' model as 'gemini-1.0-pro-vision' is deprecated.

  Credit : ChungYi Fu (Kaohsiung, Taiwan) - Original example codes
*/

#include <WiFi.h>
#include <NTPClient.h> // For getting time from NTP server
#include <WiFiUdp.h>   // Required for NTPClient

#include "GenAI.h"
#include "VideoStream.h"
#include "AmebaFatFS.h"

String Gemini_key = "AIzaSyAzAQRnNDlBXiac4E5SZcLSua-luXpbC3E"; // Paste your generated Gemini API key here
char wifi_ssid[] = "hahaha";       // Your network SSID (name)
char wifi_pass[] = "93034570";   // Your network password

WiFiSSLClient client;
GenAI llm;
VideoSetting config(768, 768, CAM_FPS, VIDEO_JPEG, 1); // 可根據需求調整解析度
#define CHANNEL 0

uint32_t img_addr = 0;
uint32_t img_len = 0;

String prompt_msg = "Please provide a brief summary of the image, including any text if visible."; // Simplified prompt
String previous_gemini_text = ""; // 儲存上一次 Gemini 的回應，用於比較

AmebaFatFS fs;
File logFile; // 用於儲存文字描述

// NTP Client setup
WiFiUDP ntpUDP;
// NTP 伺服器, GMT+8 時區偏移 (台灣時間)
// 請根據您的實際時區調整 8 * 3600
NTPClient timeClient(ntpUDP, "pool.ntp.org", 8 * 3600);

unsigned long lastCaptureTime = 0;
const unsigned long captureInterval = 60000; // 1 分鐘 (毫秒)

void initWiFi() {
  Serial.println("\n正在連接到 WiFi...");
  WiFi.begin(wifi_ssid, wifi_pass);

  uint32_t StartTime = millis();
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    if ((StartTime + 15000) < millis()) { // 增加 WiFi 連線超時時間
      Serial.println("連接 WiFi 失敗。正在重試...");
      StartTime = millis(); // 重設計時器以重試
    }
  }

  Serial.println("WiFi 連線成功。");
  Serial.print("IP 位址: ");
  Serial.println(WiFi.localIP());
}

String formatDateTime(unsigned long epochTime) {
  time_t rawtime = epochTime;
  struct tm * ti;
  ti = localtime(&rawtime);

  // 格式: YYYYMMDD_HHMMSS
  char buffer[20];
  sprintf(buffer, "%04d%02d%02d_%02d%02d%02d",
          ti->tm_year + 1900, ti->tm_mon + 1, ti->tm_mday,
          ti->tm_hour, ti->tm_min, ti->tm_sec);
  return String(buffer);
}

void setup() {
  Serial.begin(115200);

  initWiFi();

  // 初始化 NTP 客戶端
  timeClient.begin();
  Serial.println("正在從 NTP 伺服器更新時間...");
  while(!timeClient.update()) { // 等待時間更新完成
    Serial.print(".");
    delay(1000);
  }
  Serial.println("\n時間已更新。");
  Serial.print("當前時間: ");
  Serial.println(timeClient.getFormattedTime());

  config.setRotation(0); // 如果相機方向不同，請調整旋轉角度
  Camera.configVideoChannel(CHANNEL, config);
  Camera.videoInit();
  Camera.channelBegin(CHANNEL);
  Camera.printInfo();

  pinMode(LED_BUILTIN, OUTPUT);
  pinMode(LED_G, OUTPUT);

  // 初始化 SD 卡
  if (!fs.begin()) {
    Serial.println("錯誤: 無法初始化 SD 卡！");
    while (true); // 如果 SD 卡失敗則停止
  }
  Serial.println("SD 卡初始化成功。");

  lastCaptureTime = millis() - captureInterval; // 讓第一次捕捉立即發生
}

void loop() {
  timeClient.update(); // 定期更新時間

  // 檢查是否到了捕捉圖像的時間
  if (millis() - lastCaptureTime >= captureInterval) {
    lastCaptureTime = millis(); // 更新上次捕捉時間

    digitalWrite(LED_BUILTIN, HIGH); // 指示相機正在活動
    Serial.println("\n正在捕捉圖像...");
    Camera.getImage(0, &img_addr, &img_len);
    digitalWrite(LED_BUILTIN, LOW); // 關閉指示燈

    Serial.println("正在將圖像發送到 Gemini Vision (gemini-1.5-flash)...");
    // ****** 關鍵更改：將模型名稱從 "gemini-pro-vision" 更改為 "gemini-1.5-flash" ******
    String current_gemini_text = llm.geminivision(Gemini_key, "gemini-1.5-flash", prompt_msg, img_addr, img_len, client);

    // Simplify the Gemini response by trimming it to a certain length (e.g., 200 characters)
    if (current_gemini_text.length() > 200) {
      current_gemini_text = current_gemini_text.substring(0, 200) + "...";
    }

    Serial.println("\nGemini 回應:");
    Serial.println(current_gemini_text);

    // 與之前的文字進行比較
    // 檢查回應文字是否與上次不同，並且回應長度大於 0 (避免儲存空回應)
    if (current_gemini_text != previous_gemini_text && current_gemini_text.length() > 0) {
      Serial.println("偵測到場景變化或新的有效描述。正在儲存圖像和文字。");

      // 取得當前格式化的日期和時間作為檔名
      String dateTimeString = formatDateTime(timeClient.getEpochTime());

      // 儲存圖像
      String imageFileName = "/" + dateTimeString + ".jpg";
      File imageFile = fs.open(imageFileName);
      if (imageFile) {
        imageFile.write((uint8_t *)img_addr, img_len);
        imageFile.close();
        Serial.print("圖像已儲存為: ");
        Serial.println(imageFileName);
      } else {
        Serial.println("錯誤: 無法打開圖像文件進行寫入。");
      }

      // 儲存文字描述
      String logFileName = "/" + dateTimeString + ".txt";
      logFile = fs.open(logFileName);
      if (logFile) {
        logFile.print(current_gemini_text);
        logFile.close();
        Serial.print("描述已儲存為: ");
        Serial.println(logFileName);
      } else {
        Serial.println("錯誤: 無法打開描述文件進行寫入。");
      }

      previous_gemini_text = current_gemini_text; // 更新先前的文字
    } else {
      Serial.println("未偵測到明顯的場景變化或回應為空。不儲存。");
    }
  }

  // 為了避免迴圈執行過快，添加一個小延遲
  delay(100);
}

```

## 六、AI監視錄影系統成果展示

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/AI%20surveillance%20video%20system_1.png?raw=true"></p>
<p align="center">圖9 第一次擷取照片</p>

<b>照片所對應的AI產生文字敘述:</b>

The image shows a close-up, upside-down view of a person's face and upper body. 


The person has dark hair, and their face is largely visible, although the features are somewhat distorted due to the angle. Their skin tone is light to medium. They are wearing a dark-colored, possibly black, t-shirt with the word "PINETI" printed on it in white, upside down from the viewer's perspective. The background is a plain, light-grey or off-white wall or ceiling.  The overall impression is a casual, informal self-portrait taken from an unusual angle.

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/AI%20surveillance%20video%20system_2.png?raw=true"></p>
<p align="center">圖10 第二次擷取照片</p>

<b>照片所對應的AI產生文字敘述:</b>

The image shows a close-up, upside-down view of a person's head and neck.  The individual has dark hair, and their face is partially visible.  Their eyes are closed, and their mouth is slightly open. The skin tone appears light to medium.  The person is wearing a dark-colored shirt. The background is a plain, light-grey ceiling with some faintly visible light fixtures.

There is no prominent text in the image.

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/AI%20surveillance%20video%20system_3.png?raw=true"></p>
<p align="center">圖11 第三次擷取照片</p>

<b>照片所對應的AI產生文字敘述:</b>

Here's a description of the image:

The image is a low-angle, fisheye shot taken from the perspective of someone sitting in a chair. The viewpoint is significantly tilted upwards.  The scene appears to be a classroom or lecture hall with white walls and ceiling. 

The immediate foreground shows the legs and lower body of the person taking the picture, dressed in dark clothing. Part of a chair, appearing light grey or off-white and made of plastic or similar material, is visible.  A partially visible item with a marijuana leaf pattern is situated on the chair.

In the background, there's a whiteboard, which appears largely blank except for some faint markings. A person is seen sitting at a desk in front of this whiteboard, appearing to be giving a presentation or lecture.  The person's face is partially visible. 

The ceiling features three long, rectangular fluorescent light fixtures. There are also various electrical fixtures and wires visible on the wall. A water cooler or large water bottle is partially visible in the upper right corner. The overall setting seems somewhat basic and functional. There is no prominent text present besides what might be on the whiteboard, which is unreadable in the image.

<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/AI%20surveillance%20video%20system_4.png"></p>
<p align="center">圖12 SD卡內部存取狀況</p>

<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*
