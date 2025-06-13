# <p align="center"><p align="center"><span style="font-size:50px;"><b>AI 微控制應用期末專題實作報告</b></span></p>

## 一、Edge AI MCU System Diagram
本系統由 AMB82-mini 為主控核心，搭配多個模組實現人工智慧邊緣運算應用。系統整體架構如下：

- AMB82-mini：具備雙核心 CPU 及 Wi-Fi 功能，支援 TensorFlow Lite Micro 與 OpenAI API。

- ILI9341 TFT LCD：負責顯示圖像、辨識結果或系統狀態。

- PAM8403 + 4Ω 3W Speaker：進行語音播放，傳遞 TTS 結果。

- Camera (內建)：每分鐘擷取影像，送至 Gemini Vision 模型。

- MicroSD 卡：儲存辨識結果與對應影像。

- RTC 模組（若接入）：確保時間戳記準確。

模組彼此透過 SPI、PWM 與 GPIO 接腳整合，並透過 Wi-Fi 連網串接生成式 AI 服務。

### 1. AMB82-mini
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/%E5%9C%961.%20AMB82-Mini%E5%AF%A6%E9%AB%94%E5%A4%96%E8%A7%80%E5%9C%96.png"></p>
<p align="center">圖1. AMB82-Mini實體外觀圖</p>
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/%E5%9C%962.%20AMB82-Mini%E4%B8%BB%E8%A6%81%E4%BB%8B%E9%9D%A2%E8%AA%AA%E6%98%8E%E5%9C%96.png"></p>
<p align="center">圖2. AMB82-Mini主要介面說明圖</p>
<p align="center"><img src="https://github.com/Mkyzzzzz/MCU-project/blob/main/%E5%9C%963.%20AMB82-Mini%20%E9%96%8B%E7%99%BC%E6%9D%BF%E5%89%8D%E5%BE%8C%E8%A6%96%E5%9C%96%E8%88%87%20GPIO%20%E8%85%B3%E4%BD%8D%E5%8A%9F%E8%83%BD%E5%B0%8D%E6%87%89%E5%9C%96.png"></p>
<p align="center">圖3. AMB82-Mini 開發板前後視圖與 GPIO 腳位功能對應圖</p>

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
- 
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
