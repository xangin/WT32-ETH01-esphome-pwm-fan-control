# WT32-ETH01 智慧風扇控制器 (ESPHome)

這是一個基於 **WT32-ETH01** (ESP32 乙太網路開發板) 的 DIY 風扇控制器專案。

本專案整合了 PWM 風扇控制、轉速監測 (RPM)、WS2812 燈效控制以及環境溫度監測，並透過有線網路 (Ethernet) 與 Home Assistant 進行穩定連線。

<img src="img/pcb1.png" width="600">

## 🚀 功能特色

* **有線連網**：利用板載 LAN8720 晶片+ESP32，解決 Wi-Fi 訊號不穩的問題。
* **雙區 PWM 控速**：
    * 群組 1：同步控制風扇 1 與 2
    * 群組 2：同步控制風扇 3 與 4
* **獨立轉速監控**：雖然控速是分組的，但 4 顆風扇的 **RPM 轉速皆可獨立讀取**。
* **總電源管理**：透過 Relay 控制，可在待機時完全切斷風扇電源。
* **RGB 燈條整合**：可外接 WS2812 燈條，支援從 Home Assistant 調控燈光。
* **環境溫度監控**：支援 DS18B20 溫度感測器。
* **Home Assistant 原生支援**：使用 ESPhome 連接HA。

## 🛠 硬體需求自備清單

欲使用此模組需自備以下硬體:

* 風扇：4x 12V 4Pin PWM 電腦風扇 
* 燈條：**SM2.54 3Pin** WS2812 (或相容規格) 5V LED 燈條，燈數超過60顆需要修改YAML再重新編譯
* 電源：**5525圓頭規格** 12V 1A以上 DC電源，如果燈數較多，燈條需外接5V電源

## 🔌 腳位配置圖 

| 元件功能 | 腳位 | 備註 |
| :--- | :--- | :--- |
| **Ethernet** | 0, 16, 18, 23 | 內部 LAN8720 專用，不可挪用 |
| **DS18B20 溫度** | GPIO 15 | 需接 4.7kΩ 上拉電阻 |
| **Relay 繼電器** | GPIO 14 | 風扇總電源開關 |
| **WS2812 燈條** | GPIO 5 | 使用 RMT 硬體驅動 |
| **PWM 群組 1** | GPIO 4 | 控制風扇 1 & 2 轉速 |
| **PWM 群組 2** | GPIO 33 | 控制風扇 3 & 4 轉速 |
| **TACH 轉速 1** | **GPIO 35** | **僅限輸入 (Input Only)，需外接 10kΩ 上拉** |
| **TACH 轉速 2** | **GPIO 36** | **僅限輸入 (Input Only)，需外接 10kΩ 上拉** |
| **TACH 轉速 3** | **GPIO 39** | **僅限輸入 (Input Only)，需外接 10kΩ 上拉** |
| **TACH 轉速 4** | GPIO 32 | 使用 ESP32 內部上拉 (Internal Pull-up) |

> **⚠️ 硬體製作重要注意事項：**
> ESP32 的 **GPIO 35, 36, 39** 是「僅限輸入 (Input Only)」腳位，晶片內部**沒有**上拉電阻電路。
>
> 由於電腦風扇的 TACH 轉速訊號通常是 Open-Collector (開集極) 輸出，**您必須在電路板上自行焊接 10kΩ 電阻將這三個腳位上拉至 3.3V**，否則讀不到任何轉速數值。

## ⚙️ 專案設定

YAML 檔案頂部設有變數替換 (Substitutions)，方便您根據實際情況修改：

```yaml
substitutions:
  led_count: 60  # 請修改為您燈條上實際的 LED 顆數
```

## 🧩 程式碼運作原理

1. 風扇控制邏輯

- 為了讓 Home Assistant 的 Fan UI 能完美對應硬體 PWM，程式碼使用了「代理 (Proxy)」架構：

   * fan 元件：負責處理 HA 介面的開關與滑桿 (0-100%)。

   * Template Output：作為中間層，接收 HA 的數值並轉換為浮點數 (0.0 - 1.0)。

   * LEDC Output：底層硬體驅動，產生 25kHz 的 PWM 訊號給風扇。

2. 轉速計算

一般 PC 風扇每轉一圈會輸出 2 個脈衝 (Pulses)。ESPHome 的 pulse_counter 預設計算的是「每分鐘脈衝數」。為了得到正確的 RPM (每分鐘轉速)，我們使用 Filter 進行換算：

```yaml
filters:
  - multiply: 0.5  # 2 pulses/rev -> 乘以 0.5 還原為真實 RPM
```

## 📦 安裝說明

-  安裝 ESPHome 編譯環境。

    下載本專案的 YAML 檔案。

    編譯並上傳至 WT32-ETH01。

>  初次燒錄提示：WT32-ETH01 需透過 USB-TTL 模組連接 (TX0/RX0)，並將 GPIO0 接地 (GND) 後上電才能進入燒錄模式。

## 📄 授權條款

[MIT License](LICENSE)
