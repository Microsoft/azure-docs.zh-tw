---
title: 為即時串流而設的疑難排解指南 | Microsoft Docs
description: 本主題提供有關如何疑難排解即時資料流問題的建議。
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/15/2018
ms.author: juliako
ms.openlocfilehash: e6b135e14f06ecf4edfbb97913c411f55711854a
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49351447"
---
# <a name="troubleshooting-guide-for-live-streaming"></a>為即時串流而設的疑難排解指南
本文提供有關如何疑難排解某些即時串流問題的建議。

## <a name="issues-related-to-on-premises-encoders"></a>內部部署編碼器的相關問題
本節提供如何疑難排解內部部署編碼器相關問題的建議，而編碼器的設定是傳送單一位元速率資料流到啟用即時編碼的 AMS 通道。

### <a name="problem-would-like-to-see-logs"></a>問題：想要查看記錄檔
* 潛在問題：找不到可能有助於偵錯問題的編碼器記錄檔。
  
  * **Telestream Wirecas**：您通常可以在 C:\Users\{使用者名稱}\AppData\Roaming\Wirecast\ 下找到記錄檔 
  * ︰您可在管理入口網站上找到記錄檔連結。 依序按一下 [統計] 及 [記錄檔]。 在 [記錄檔]  頁面上，您會看到一份所有 LiveEvent 項目的記錄檔清單，請選取符合您目前工作階段的項目。 
  * **Flash Media Live Encoder**︰瀏覽至 [編碼記錄檔] 索引標籤即可找到 [記錄檔目錄...]。

### <a name="problem-there-is-no-option-for-outputting-a-progressive-stream"></a>問題：沒有輸出漸進式資料流的選項
* **可能的問題**：使用的編碼器不會自動進行非交錯處理。 
  
    **疑難排解步驟**：尋找編碼器介面中的非交錯處理選項。 一旦啟用非交錯處理，請再次檢查漸進式的輸出設定。 

### <a name="problem-tried-several-encoder-output-settings-and-still-unable-to-connect"></a>問題：已嘗試數種編碼器輸出設定，但仍然無法連線。
* **可能的問題**：未正確重設 Azure 編碼通道。 
  
    **疑難排解步驟**：確定編碼器不再推播至 AMS，停止並重設通道。 再次執行後，請嘗試使用新的設定連線至您的編碼器。 有時候，通道在經過幾次失敗的嘗試後可能會損毀，如果這樣仍無法更正問題，請嘗試建立新通道。  
* **可能的問題**：GOP 大小或主要畫面設定不佳。 
  
    **疑難排解步驟**：建議的 GOP 大小或主要畫面間隔為兩秒。 某些編碼器以畫面數目計算此設定，某些則使用秒。 例如：輸出 30 fps 時，GOP 大小會是 60 個畫面，相當於 2 秒。  
* **可能的問題**：關閉的連接埠封鎖資料流。 
  
    **疑難排解步驟**：透過 RTMP 串流處理時，請檢查防火牆和/或 Proxy 設定，確認輸出連接埠 1935 和 1936 已開啟。 

> [!NOTE]
> 如果依循下列疑難排解步驟後仍無法順利串流處理，請使用 Azure 入口網站提交支援票證。
> 
> 

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

