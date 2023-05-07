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

### 成果展示
 ![](https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/IMG_4427.PNG)
 
 ![](https://github.com/Mkyzzzzz/MCU-project/blob/main/_posts/IMG_4429.jpg)
