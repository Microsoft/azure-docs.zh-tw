---
title: StorSimple Virtual Array Web UI 管理 |Microsoft Docs
description: 描述如何透過 StorSimple Virtual Array Web UI 執行基本的裝置管理工作。
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: ea65b4c7-a478-43e6-83df-1d9ea62916a6
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 12/1/2016
ms.author: alkohli
ms.openlocfilehash: 989e7b697f9b527df549fb32be18edd1d3c8d224
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109956"
---
# <a name="use-the-web-ui-to-administer-your-storsimple-virtual-array"></a>使用 Web UI 來管理 StorSimple Virtual Array
![安裝程序流程](./media/storsimple-ova-web-ui-admin/manage4.png)

## <a name="overview"></a>概觀
本文中的教學課程適用於執行 2016 年 3 月公開上市 (GA) 版的 Microsoft Azure StorSimple Virtual Array (也稱為 StorSimple 內部部署虛擬裝置)。 本文將說明一些可以在 StorSimple Virtual Array 執行的複雜工作流程和管理工作。 您可以使用 StorSimple Manager 服務 UI (又稱為入口網站 UI) 和裝置的本機 Web UI 來管理 StorSimple Virtual Array。 本文著重於可使用 Web UI 執行的工作。

本文包含下列教學課程：

* 取得服務資料加密金鑰
* 疑難排解 Web UI 安裝程式錯誤
* 產生記錄檔封裝
* 關閉或重新啟動您的裝置

## <a name="get-the-service-data-encryption-key"></a>取得服務資料加密金鑰
當您向 StorSimple Manager 服務註冊您的第一個裝置時，會產生服務資料加密金鑰。 這個金鑰需要與服務註冊金鑰搭配使用，以向 StorSimple Manager 服務註冊其他裝置。

如果您弄丟您的服務資料加密金鑰而需要擷取它，請在向您的服務註冊的裝置的本機 Web UI 中執行下列步驟。

#### <a name="to-get-the-service-data-encryption-key"></a>取得服務資料加密金鑰
1. 連線到本機 Web UI。 移至 [組態]  >  [雲端設定]。
2. 按一下頁面底部的 [取得服務資料加密金鑰] 。 金鑰將會顯示。 複製並儲存此金鑰。
   
    ![取得服務資料加密金鑰 1](./media/storsimple-ova-web-ui-admin/image27.png)

## <a name="troubleshoot-web-ui-setup-errors"></a>疑難排解 Web UI 安裝程式錯誤
在某些情況下，當您透過本機 Web UI 設定裝置時，可能會遇到錯誤。 若要診斷和移難排解這類錯誤，您可以執行診斷測試。

#### <a name="to-run-the-diagnostic-tests"></a>執行診斷測試
1. 在本機 Web UI 中，移至 [疑難排解]  >  [診斷測試]。
   
    ![執行診斷 1](./media/storsimple-ova-web-ui-admin/image29.png)
2. 按一下頁面底部的 [執行診斷測試] 。 這樣會起始測試以診斷和您的網路、裝置、Web Proxy、時間或雲端設定相關的任何可能問題。 您將收到裝置正在執行測試的通知。
3. 測試完成後會顯示結果。 下列範例顯示診斷測試的結果。 請注意，此裝置上未設定 Web Proxy 設定，因此未執行 Web Proxy 測試。 網路設定、DNS 伺服器和時間設定的所有其他測試均成功。
   
    ![執行診斷 2](./media/storsimple-ova-web-ui-admin/image30.png)

## <a name="generate-a-log-package"></a>產生記錄檔封裝
記錄檔封裝包含有助於 Microsoft 支援小組疑難排解任何裝置問題的所有相關記錄檔。 在此版本中，可透過本機 Web UI 產生記錄檔封裝。

#### <a name="to-generate-the-log-package"></a>產生記錄檔封裝
1. 在本機 Web UI 中，移至 [疑難排解]  >  [系統記錄檔]。
   
    ![產生記錄檔封裝 1](./media/storsimple-ova-web-ui-admin/image31.png)
2. 按一下頁面底部的 [建立記錄檔封裝] 。 將建立系統記錄檔的封裝。 這需要幾分鐘的時間。
   
    ![產生記錄檔封裝 2](./media/storsimple-ova-web-ui-admin/image32.png)
   
    系統會通知您已成功建立封裝，並會更新頁面以顯示封裝建立的日期與時間。
   
    ![產生記錄檔封裝 3](./media/storsimple-ova-web-ui-admin/image33.png)
3. 按一下 [下載記錄檔封裝] 。 壓縮的封裝將會下載到您的系統上。
   
    ![產生記錄檔封裝 4](./media/storsimple-ova-web-ui-admin/image34.png)
4. 您可以解壓縮下載的記錄檔封裝，並檢視系統記錄檔。

## <a name="shut-down-and-restart-your-device"></a>關閉並重新啟動您的裝置
您可以使用本機 Web UI 關閉或重新啟動您的虛擬裝置。 我們建議您在重新啟動之前，讓主機上的磁碟區或共用離線，然後讓裝置離線。 這樣可以讓資料損毀的可能性降至最低。 

#### <a name="to-shut-down-your-virtual-device"></a>關閉虛擬裝置
1. 在本機 Web UI 中，移至 [維護]  >  [電源設定]。
2. 按一下頁面底部的 [關閉] 。
   
    ![裝置關閉 1](./media/storsimple-ova-web-ui-admin/image36.png)
3. 會出現警告，指出關閉裝置將中斷已在進行的任何 IO，因而導致停機。 按一下核取圖示  ![核取圖示](./media/storsimple-ova-web-ui-admin/image3.png).
   
    ![裝置關閉警告](./media/storsimple-ova-web-ui-admin/image37.png)
   
    系統會通知您已起始關機。
   
    ![已開始關閉裝置](./media/storsimple-ova-web-ui-admin/image38.png)
   
    裝置會立即關閉。 如果您想要啟動您的裝置，您必須透過 Hyper-V 管理員執行。

#### <a name="to-restart-your-virtual-device"></a>重新啟動虛擬裝置
1. 在本機 Web UI 中，移至 [維護]  >  [電源設定]。
2. 按一下頁面底部的 [重新啟動] 。
   
    ![裝置重新啟動](./media/storsimple-ova-web-ui-admin/image36.png)
3. 會出現警告，指出重新啟動裝置將中斷已在進行的任何 IO，因而導致停機。 按一下核取圖示  ![核取圖示](./media/storsimple-ova-web-ui-admin/image3.png).
   
    ![重新啟動警告](./media/storsimple-ova-web-ui-admin/image37.png)
   
    系統會通知您已起始重新啟動。
   
    ![已起始重新啟動](./media/storsimple-ova-web-ui-admin/image39.png)
   
    重新啟動時，將會中斷 UI 連線。 您可以定期重新整理 UI 以監視重新啟動的情況。 或者，您可以透過 Hyper-V 管理員監視裝置重新啟動狀態。

## <a name="next-steps"></a>後續步驟
了解如何 [使用 StorSimple Manager 服務管理裝置](storsimple-virtual-array-manager-service-administration.md)。

