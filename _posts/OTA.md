---
layout: post
title: OTA作業
author: [張家豪]
category: [Lecture]
tags: [jekyll, ai]
---


## 程式
```
#include <Arduino.h>
#include <WiFi.h>
#include <AsyncTCP.h>
#include <ESPAsyncWebSrv.h>
#include <AsyncElegantOTA.h>

const char* ssid = "A18";
const char* password = "18181818";

AsyncWebServer server(80);

void setup(void) {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  Serial.println("");

  // Wait for connection
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Connected to ");
  Serial.println(ssid);
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  server.on("/", HTTP_GET, [](AsyncWebServerRequest *request) {
    request->send(200, "text/plain", "Hi! I am ESP32.");
  });

  AsyncElegantOTA.begin(&server);    // Start ElegantOTA
  server.begin();
  Serial.println("HTTP server started");
}

void loop(void) {
}
```
### 網頁展示
![](https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/IMG_4426.PNG)

### 設計考量與相關技術
**系統設計考量：**<br>
1. 操作方式:WebUI
2. 移動方式:兩輪 
3. 供電方式:鋰電池 3.7V x2
4. 聯網方式:藍牙

**所需相關技術：**
1. WiFi 
2. Arduino程式設計

**所需相關套件:**
![](https://image.ruten.com.tw/g2/8/d4/16/21440347657238_872.jpg)

### 系統方塊圖
![](https://github.com/Mkyzzzzz/MCU-project/blob/main/WebUI_car.jpg)

### 成果展示
![](https://github.com/Mkyzzzzz/MCU-project/blob/main/forward_stop.gif)

![](https://github.com/Mkyzzzzz/MCU-project/blob/main/back.gif)

![](https://github.com/Mkyzzzzz/MCU-project/blob/main/left.gif)

![](https://github.com/Mkyzzzzz/MCU-project/blob/main/right.gif)

![](https://github.com/Mkyzzzzz/MCU-project/blob/main/RobotCar)
