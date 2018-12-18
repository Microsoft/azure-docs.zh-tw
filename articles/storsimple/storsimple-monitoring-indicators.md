---
title: StorSimple 監視指示器 | Microsoft Docs
description: 描述用來監視 StorSimple 裝置狀態的發光二極體 (LED 燈) 與警報器。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 59dee7b9-ca6d-4fd9-96e6-a0071e8d248e
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: alkohli
ms.openlocfilehash: 9ae0caec211dc1199f0abd2ce9bc0c7ad11c02ec
ms.sourcegitcommit: 0930aabc3ede63240f60c2c61baa88ac6576c508
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/07/2017
ms.locfileid: "24030352"
---
# <a name="use-storsimple-monitoring-indicators-to-manage-your-device"></a>使用 StorSimple 監視指示器來管理您的裝置


## <a name="overview"></a>概觀
StorSimple 裝置包括發光二極體 (LED) 與警示，您可以用來監控模組和 StorSimple 裝置上的整體狀態。 監控指示器位於裝置主要機箱與 EBOD 機箱的硬體元件上。 監控指示器可以是 LED 或有聲警報器。

有三種 LED 狀態可用來指出模組的狀態：綠色、綠色不停閃爍到變紅琥珀色，或紅琥珀色。  

* 綠色 LED 代表運作狀態良好  
* 不停閃爍綠色到變紅琥珀色 LED 代表可能發生需要使用者介入的非嚴重狀況。  
* 紅琥珀色 LED 指出模組內發生嚴重錯誤。  

這篇文章的其餘部分將說明各種監控 LED 指示燈、 LED 在 StorSimple 裝置上的位置，根據 LED 狀態的裝置狀態，以及任何相關聯的有聲警報器。

## <a name="front-panel-indicator-leds"></a>前端面板 LED 指示燈
前端面板，也稱為「操作面板」或「操作面板」，其顯示系統中所有模組的彙總狀態。 StorSimple 主要機箱與 EBOD 機箱上的前端面板完全相同，如下圖所示。  

   ![裝置正面面板][1]

前端面板包含下列指示器：  

1. 靜音按鈕
2. 電源 LED 指示燈 (綠色/紅琥珀色)
3. 模組錯誤 LED 指示燈 (開時為紅琥珀色/關)
4. 邏輯錯誤 LED 指示燈 (開時為紅琥珀色/關)
5. 單元識別碼顯示  

裝置前端面板 LED 與 EBOD 機箱的前端面板 LED 的主要差異在於 LED 顯示上顯示的「系統單元識別碼」。 裝置上顯示的預設單位識別碼是 **00**，而 EBOD 機箱上顯示的預設單元識別碼是 **01**。 這可讓您在裝置開啟時，快速區分裝置與 EBOD 機箱。 如果您的裝置已關閉，請使用[啟動新的裝置](storsimple-turn-device-on-or-off.md#turn-on-a-new-device)所提供的資訊來區分裝置與 EBOD 機箱。  

## <a name="front-panel-led-status"></a>前端面板 LED 狀態
您可以使用下表，利用裝置前端面板或 EBOD 機箱前端面板上的 LED 來識別所指示的狀態。  

| 系統電源 | 模組錯誤 | 邏輯錯誤 | 警示 | 狀態 |
| --- | --- | --- | --- | --- |
| 紅琥珀色 |關 |關 |N/A |AC 電源中斷、以備用電源運作，或 AC 電源已開但已移除控制器模組。 |
| 綠色 |開啟 |開啟 |N/A |操作面板電源開啟 (5 秒) 測試狀態 |
| 綠色 |關 |關 |N/A |電源開啟、所有功能正常 |
| 綠色 |開啟 |N/A |PCM 故障 LED、風扇故障 LED |任何 PCM 故障、 風扇故障、 溫度過高或過低 |
| 綠色 |開啟 |N/A |I/O 模組 LED |任何控制器模組錯誤 |
| 綠色 |開啟 |N/A |N/A |機箱邏輯錯誤 |
| 綠色 |閃爍 |N/A |控制器模組上的模組狀態 LED。 PCM 故障 LED、風扇故障 LED |安裝的控制器模組類型不明、I2C 匯流排故障、控制器模組重要產品資料 (VPD) 組態錯誤 |

## <a name="power-cooling-module-pcm-indicator-leds"></a>電源冷卻模組 (PCM) LED 指示燈
電源冷卻模組 (PCM) LED 指示燈位於每個 PCM 模組上主要機箱或 EBOD 機箱的背面。 本主題討論如何使用下列 LED 來監控 StorSimple 裝置的狀態。  

* 主要機箱的 PCM LED
* EBOD 機箱的 PCM LED

## <a name="pcm-leds-for-the-primary-enclosure"></a>主要機箱的 PCM LED
StorSimple 裝置有 764W PCM 模組與額外電池。 下圖顯示裝置的 LED 面板。  

   ![主要機箱上的 PCM LED 燈][2]

LED 燈圖例：

1. AC 電源故障
2. 風扇故障
3. 電池故障
4. PCM 正常
5. DC 故障
6. 電池良好  

LED 面板上會指出 PCM 的狀態。 裝置 PCM LED 面板有六個 LED。 其中四個 LED 顯示電源供應器與風扇的狀態。 其餘兩個 LED 會指出 PCM 中備份電池模組的狀態。 您可以使用下列表格來判斷 PCM 的狀態。  

### <a name="pcm-indicator-leds-for-power-supply-and-fan"></a>電源供應器與風扇的 PCM LED 指示燈
| 狀態 | PCM 正常 (綠色) | AC 故障 (琥珀色) | 風扇故障 (琥珀色) | DC 故障 (琥珀色) |
| --- | --- | --- | --- | --- |
| 沒有 AC 電源 (機箱) |關 |關 |關 |關 |
| 沒有 AC 電源 (只有此 PCM) |關 |開啟 |關 |開啟 |
| 有 AC 且 PCM 開啟 - 正常 |開啟 |關 |關 |關 |
| PCM 故障 (風扇故障) |關 |關 |開啟 |N/A |
| PCM 故障 (安培、 電壓、電流過高) |關 |開啟 |開啟 |開啟 |
| PCM (風扇超出容許範圍) |開啟 |關 |關 |開啟 |
| 待命模式 |不停閃爍 |關 |關 |關 |
| PCM 韌體下載 |關 |不停閃爍 |不停閃爍 |不停閃爍 |

### <a name="pcm-indicator-leds-for-the-backup-battery"></a>備份電池的 PCM LED 指示燈
| 狀態 | 電池良好 (綠色) | 電池故障 (琥珀色) |
| --- | --- | --- |
| 沒有電池 |關 |關 |
| 有電池且已充電 |開啟 |關 |
| 電池充電中或維護放電 |不停閃爍 |關 |
| 電池「軟性」故障 (可修復) |關 |不停閃爍 |
| 電池「硬性」故障 (不可修復) |關 |開啟 |
| 已卸除電池 |不停閃爍 |關 |

## <a name="pcm-leds-for-the-ebod-enclosure"></a>EBOD 機箱的 PCM LED
EBOD 機箱有一個 580W PCM，沒有額外電池。 EBOD 機箱的 PCM 面板 LED 指示燈僅適用於電源供應器與風扇。 下圖顯示這些 LED。

   ![EBOD 機箱上的 PCM LED 燈][3] 

您可以使用下表來判斷 PCM 的狀態。  

| 狀態 | PCM 正常 (綠色) | AC 故障 (琥珀色) | 風扇故障 (琥珀色) | DC 故障 (琥珀色) |
| --- | --- | --- | --- | --- |
| 沒有 AC 電源 (機箱) |關 |關 |關 |關 |
| 沒有 AC 電源 (只有此 PCM) |關 |開啟 |關 |開啟 |
| 有 AC 且 PCM 為開 - 正常 |開啟 |關 |關 |關 |
| PCM 故障 (風扇故障) |關 |關 |開啟 |X |
| PCM 故障 (安培、 電壓、電流過高) |關 |開啟 |開啟 |開啟 |
| PCM (風扇超出容許範圍) |開啟 |關 |關 |開啟 |
| 待命模式 |不停閃爍 |關 |關 |關 |
| PCM 韌體下載 |關 |不停閃爍 |不停閃爍 |不停閃爍 |

## <a name="controller-module-indicator-leds"></a>控制器模組 LED 指示燈
StorSimple 裝置包含主要控制器的 LED 與 EBOD 控制器模組的 LED。   

### <a name="monitoring-leds-for-the-primary-controller"></a>監控主要控制器的 LED
下圖可協助您識別主要控制器上的 LED。 (所有元件列出如下，以協助定位)。  

   ![監視 LED 燈 - 主要控制器][4]

使用下表來判斷控制器模組是否運作正常。  

### <a name="controller-indicator-leds"></a>控制器 LED 指示燈
| LED | 說明 |
| --- | --- |
| ID LED (藍色) |指出已找到此模組。 如果主動控制器上閃爍著藍色 LED，即表示該控制器處於作用中，而另一控制器則處於待命中。 如需詳細資訊，請參閱 [識別裝置上的作用中控制器](storsimple-8000-controller-replacement.md#identify-the-active-controller-on-your-device)。 |
| 錯誤 LED (琥珀色) |指出控制器中發生錯誤。 |
| 正常 LED (綠色) |持續亮著綠色表示控制器正常。 不停閃爍綠色表示控制器 VPD 組態錯誤。 |
| SAS 活動 LED (綠色) |持續亮著綠色表示連線目前沒有活動。 不停閃爍綠色表示連線正在進行活動。 |
| 乙太網路狀態 LED |右側代表連結/網路活動：連結作用中 (持續亮著綠色)、有網路活動 (不停閃爍綠色)。 左側代表網路速度：1000 MB/秒 (黃色)、100 MB/秒 (綠色) 及 10 MB/秒 (關)。 根據元件模型，即使未啟用網路介面，指示燈也可能會閃爍。 |
| POST LED |可在開啟控制器時，指出開機進度。 如果 StorSimple 裝置無法開機，這個 LED 可幫助 Microsoft 支援找出開機程序中發生失敗之處。 |

> [!IMPORTANT]
> 錯誤 LED 亮起時，重新啟動控制器可能會解決控制器模組的問題。 如果重新啟動控制器仍無法解決這個問題，請連絡 Microsoft 支援。  
> 
> 

### <a name="monitoring-leds-for-the-ebod-ebod-enclosure"></a>EBOD 的監控 LED (EBOD 機箱) 
每個 6 GB/s SAS EBOD 控制器都有各自的 LED指出其狀態，如下圖所示。  

  ![監視 LED 燈 - EBOD 機相][5]

使用下表來判斷 EBOD 控制器模組是否運作正常。  

### <a name="ebod-controller-module-indicator-leds"></a>EBOD 控制器模組 LED 指示燈
| 狀態 | I/O 模組正常 (綠色) | I/O 模組故障 (琥珀色) | 主機連接埠活動 (綠色) |
| --- | --- | --- | --- |
| 控制器模組確定 |開啟 |關 |- |
| 控制器模組錯誤 |關 |開啟 |- |
| 沒有外部主機連接埠連接 |- |- |關 |
| 外部主機連接埠連接 – 沒有活動 |- |- |開啟 |
| 外部主機連接埠連接 – 活動 |- |- |不停閃爍 |
| 控制器模組中繼資料錯誤 |不停閃爍 |- |- |

## <a name="disk-drive-indicator-leds-for-the-primary-enclosure-and-ebod-enclosure"></a>主要機箱與 EBOD 機箱的磁碟機 LED 指示燈
StorSimple 裝置的磁碟機可以位於主要機箱與 EBOD 機箱。 每個磁碟機都有各自的監控 LED 指示燈，如本節中所述。 

磁碟機的磁碟機狀態可由每個磁碟機載具模組正面掛接的綠色 LED 與紅琥珀色 LED 指出。 下圖顯示這些 LED。

  ![磁碟機 LED 燈][6]

您可以使用下表來判斷每個磁碟機的狀態，整體的前端面板 LED 狀態會受這些狀態影響。  

### <a name="disk-drive-indicator-leds-for-the-ebod-enclosure"></a>EBOD 機箱的磁碟機 LED 指示燈
| 狀態 | 活動正常 LED (綠色) | 錯誤 LED (紅琥珀色) | 相關聯操作面板 LED |
| --- | --- | --- | --- |
| 未安裝磁碟機 |關 |關 |None |
| 磁碟機已安裝且可運作 |有活動時會不停開/關地閃爍 |X |None |
| SCSI 機箱服務 (SES) 裝置身分識別已設定 |開啟 |每 2 秒閃爍 1 次 |None |
| SES 裝置錯誤位元已設定 |開啟 |開啟 |邏輯錯誤 (紅色) |
| 電源控制電路故障 |關 |開啟 |模組錯誤 (紅色) |

## <a name="audible-alarms"></a>有聲警報器
StorSimple 裝置包含和主要機箱與 EBOD 機箱相關聯的有聲警報器。 有聲警報器位於這兩個機箱的前端面板 (也稱為操作面板)。 有聲警報器會指出發生錯誤狀況。 下列狀況將啟動警報器：  

* 風扇錯誤或故障
* 電壓超出範圍
* 超過或低於溫度條件
* 熱超限
* 系統錯誤
* 邏輯錯誤
* 電源供應器故障
* 卸下電源冷卻模組 (PCM)  

下表說明各種警報器狀態。  

### <a name="alarm-states"></a>警報器狀態
| 警報器狀態 | 動作 | 已按靜音按鈕時的動作 |
| --- | --- | --- |
| S0 |標準模式：靜音 |嗶兩聲 |
| S1 |錯誤模式：每 2 秒嗶 1 聲 |轉換至 S2 或 S3 (請參閱注意事項) |
| S2 |提醒模式：間歇嗶聲 |None |
| S3 |靜音模式：靜音 |None |
| S4 |重大錯誤模式：連續警示 |無法使用：未啟用靜音 |

> [!NOTE]
> * 處於警示狀態 S1 時，如果您未能在 2 分鐘內按下靜音，狀態會自動轉換至 S2 或 S3。  
> * 警示狀態 S1 至 S4 都會在錯誤狀況清除之後，回到 S0。  
> * 任何其他狀態都能進入重大錯誤狀態 S4。  


您可以按下操作面板上的 [靜音] 按鈕，讓有聲警報器變靜音。 如果不手動操作靜音開關，便會在兩分鐘後自動靜音。 警報器靜音時，將會繼續間歇發出短嗶聲，代表問題依然存在。 清除所有問題之後，警報器才會安靜下來。

下表說明各種警示狀況。

### <a name="alarm-conditions"></a>警示狀況
| 狀態 | 嚴重性 | 警示 | 操作面板 LED |
| --- | --- | --- | --- |
| PCM 警示 – 失去單一 PCM 提供的 DC 電源 |錯誤 – 未失去備援 |S1 |模組錯誤 |
| PCM 警示 – 失去單一 PCM 提供的 DC 電源 |錯誤 – 失去備援 |S1 |模組錯誤 |
| PCM 風扇故障 |錯誤 – 失去備援 |S1 |模組錯誤 |
| SBB 模組偵測到 PCM 錯誤 |錯誤 |S1 |模組錯誤 |
| 已卸下 PCM |組態錯誤 |None |模組錯誤 |
| 機箱組態錯誤 |錯誤 – 重大 |S1 |模組錯誤 |
| 警告低溫警示 |警告 |S1 |模組錯誤 |
| 警告高溫警示 |警告 |S1 |模組錯誤 |
| 溫度過高警示 |錯誤 – 重大 |S1 |模組錯誤 |
| I2C 匯流排故障 |錯誤 – 失去備援 |S1 |模組錯誤 |
| 操作面板通訊錯誤 (I2C) |錯誤 – 重大 |S1 |模組錯誤 |
| 控制器錯誤 |錯誤 – 重大 |S1 |模組錯誤 |
| SBB 介面模組錯誤 |錯誤 – 重大 |S1 |模組錯誤 |
| SBB 介面模組錯誤 – 沒有運作中的模組 |錯誤 – 重大 |S4 |模組錯誤 |
| 已卸下 SBB 介面模組 |警告 |None |模組錯誤 |
| 磁碟機電源控制故障 |警告 – 未失去磁碟機電源 |S1 |模組錯誤 |
| 磁碟機電源控制故障 |錯誤 – 重大；失去磁碟機電源 |S1 |模組錯誤 |
| 已卸下磁碟機 |警告 |None |模組錯誤 |
| 沒有足夠的可用電源 |警告 |None |模組錯誤 |

## <a name="next-steps"></a>後續步驟
深入了解 [StorSimple 硬體元件與狀態](storsimple-8000-monitor-hardware-status.md)。

[1]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE01.png
[2]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE02.png
[3]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE03.png
[4]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE04.png
[5]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE05.png
[6]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE06.png


