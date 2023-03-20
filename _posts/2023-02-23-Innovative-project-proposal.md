---
layout: post
title: Innovative Project Proposal
author: [Mkyzzzzz]
category: [Lecture]
tags: [jekyll, ai]
---

This homework is to propose an innovative project and describe the key features, list all Design Considerations and the required technologies, then draw the System Block Diagram.

---
## Homework Report Format
**Contents:**<br>
* **應用與功能說明**
  - Specify the future home application, and Describe the key features
  - Describe the key features which may be applied to the home space (kitchen, living room, play room, study room, bed room)
* **設計考量與所需相關技術**
  - List all design considerations and the required technologies
* **系統方塊圖**
  - Draw a System Block Diagram

---
## 遠端智能衣物&學用品搭配系統
![](https://github.com/rkuo2023/MCU-project/blob/main/images/ESP32_RoboCar.jpg?raw=true)

### 應用功能說明
1. 智能衣櫃：每件衣服有專屬的吊掛位置，並依每件衣服的款式登錄到相對應的吊掛位置，並以手機程式控制，在手機上以表單呈現每個衣架所屬的衣服圖片及編號，手機觸發所選的衣服後衣架上的夾子將張    開使衣服掉落至衣櫃底部的托盤，並自動將托盤推出，而鞋子之部分也是以自動托盤，推出相對應隻鞋子。
2. 智能書櫃：如同智能衣櫃一樣，以每本書有相對應之書擋，並由手機中的程式觸發其指定之書擋放書，並從書架滑下與衣服由同一托盤一併推出，不同的是程式中可設定每天固定選取哪幾本課本，使使用者    無須每天選取以及於書擋上放置感測器，若書不在位置上則無法選取此書，方便使用者得知書是否遺失。

### 設計考量與相關技術
**系統設計考量：**<br>
1. 操作方式: 手機程式
2. 移動方式: 自由活動之托盤
3. 供電方式: 一般家用電源
4. 聯網方式: WiFi

**所需相關技術：**
1. 軟式夾具
2. 感測器
3. 手機程式設計

### 系統方塊圖
![](https://github.com/rkuo2000/MCU-course/blob/main/images/FutureHome_kitchen_robot.png?raw=true)

---
### 參考範例
<iframe width="954" height="537" src="https://www.youtube.com/embed/d7NcoepWlyU" title="Real time reinforcement learning demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*


