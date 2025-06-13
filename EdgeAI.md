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
  
GPIO INT（General-Purpose Input/Output with Interrupts）
  
      支援中斷觸發的 GPIO 腳位可用於偵測電壓變化（如上升緣/下降緣）並自動呼叫中斷函數，不需輪詢（polling），節省 CPU 運算資源，實現事件驅動控制。
  應用實例：
  •	按鈕按壓觸發畫面切換
  •	PIR 感測器啟動語音警示
  •	碰撞感測器觸發機械動作
  對應腳位：
  幾乎所有 GPIO 腳位皆支援中斷，包括 PF0 ~ PF15、PE1 ~ PE6、PD14 ~ PD18、PA0 ~ PA3
  	ADC（Analog to Digital Converter）
      ADC 將外部模擬訊號（連續電壓）轉為數位資料，供 MCU 做進一步分析與處理。AMB82-Mini 提供 8 個 ADC 通道，最大解析度 12-bit。
  應用實例：
  •	讀取光敏電阻控制螢幕亮度
  •	偵測土壤濕度控制自動澆水
  •	監控電池電壓防止過放
  對應腳位：
  PF0（A0）、PF1（A1）、PF2（A2）、PF3（*A3）、PA0（A4）、PA1（A5）、PA2（A6）、PA3（A7）
