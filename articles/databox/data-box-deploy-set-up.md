---
title: 設定您的 Azure 資料箱 | Microsoft Docs
description: 深入了解如何連接 Azure 資料箱的纜線並且連線
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 10/17/2018
ms.author: alkohli
ms.openlocfilehash: c3766713c87dd721f7e4c9a90624192cdb402be9
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49393825"
---
# <a name="tutorial-cable-and-connect-to-your-azure-data-box"></a>教學課程：連接 Azure 資料箱的纜線並且連線

本教學課程說明如何連接 Azure 資料箱的纜線、加以連線並開啟。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 連接資料箱的纜線
> * 連線至資料箱

## <a name="prerequisites"></a>必要條件

在您開始前，請確定：

1. 您已完成[教學課程：訂購 Azure 資料箱](data-box-deploy-ordered.md)。
2. 您已收到資料箱，且入口網站中的訂購狀態為 [已交付]。 黏貼到裝置目前標籤下的透明袋中有出貨標籤。 請妥善保存此標籤，您會在退貨時用到。
3. 您已檢閱[資料箱安全指導方針](data-box-safety.md)。
4. 您已收到一條用於 100 TB 儲存體裝置的接地電源線。
5. 您有一部主機電腦，其中包含要複製到資料箱的資料。 您的主機電腦必須符合下列條件：
    - 執行[支援的作業系統](data-box-system-requirements.md)。
    - 連線至高速網路。 強烈建議您具有至少一個 10 GbE 的連線。 如果無法使用 10 GbE 連線，也可以使用 1 GbE 資料連結，但是複製速度會受到影響。 
6. 您必須能夠進入放置資料箱所在的平坦表面。 如果您想要將裝置放在標準機架上，您的資料中心機架中需要有 7U 插槽。 您可以將裝置平放或垂直放置在機架中。
7. 您已購買下列纜線，以將您的資料箱連線至主機電腦。
    - 兩條 10 GbE SFP+ Twinax 銅纜線 (與 DATA 1、DATA 2 網路介面搭配使用)
    - 一條 RJ-45 CAT 6 網路線 (與 MGMT 網路介面搭配使用)
    - 一條 RJ-45 CAT 6A 或一條 RJ-45 CAT 6 網路線 (與 DATA 3 網路介面搭配使用，分別設定為 10 Gbps 或 1 Gbps)

## <a name="cable-your-device"></a>將裝置接上纜線

請執行下列步驟以連接裝置的纜線。

1. 檢查裝置是否有任何竄改的跡象，或其他任何明顯的損毀。 如果裝置已遭竄改或嚴重損毀，請勿繼續操作。 請立即連絡 Microsoft 支援服務以協助您評估裝置是否可正常運作，以及是否需要寄送替代品給您。
2. 將裝置移至您要加以開啟的位置上。 將裝置放在平面上。 裝置也可以放置在標準機架上。
3. 連接電源線和網路線。 在常用的組態中，連線裝置的後擋板如下所示。 
    
    ![已連接纜線的資料箱裝置後擋板](media/data-box-deploy-set-up/data-box-cabled-dhcp.png)

    1. 將電源線連接到標示的電源輸入位置。 電源線的另一端應連接到配電裝置。
    2. 使用 RJ-45 CAT 6 纜線連接一端的 MGMT 連接埠，以及另一端的膝上型電腦。            
    3. 使用 RJ-45 CAT 6A 纜線連接到一端的 DATA 3 連接埠。 如果您透過 RJ-45 CAT 6A 纜線連線，DATA 3 將會設定為 10 GbE，如果透過 RJ-45 CAT 6 纜線連線，則會設定為 1 GbE。
    4. 使用 10 GbE SFP + Twinax 銅纜線，分別連接 DATA 1 和 DATA 2 連接埠。 
    5. 資料連接埠纜線的另一端會透過 10 GbE 交換器連接到主機電腦。

4. 在裝置的前端操作面板上找出電源按鈕。 開啟裝置。

    ![資料箱電源按鈕](media/data-box-deploy-set-up/data-box-powered-door-open.png)

## <a name="connect-to-your-device"></a>連接到您的裝置

執行下列步驟，以使用本機 Web UI 和入口網站 UI 設定您的裝置。

1. 使用靜態 IP 位址 192.168.100.5 和子網路 255.255.255.0，在您要用來連接到裝置的膝上型電腦上設定乙太網路介面卡。 
2. 連接到裝置的 MGMT 連接埠，並經由 https://192.168.100.10 存取其本機 Web UI。 在開啟裝置後，此作業最多可能需耗時 5 分鐘。
3. 按一下 [詳細資料]，然後按一下 [繼續瀏覽網頁]。

   ![連線至本機 Web UI](media/data-box-deploy-set-up/data-box-connect-local-web-ui.png) 

4. 您會看到本機 Web UI 的 [登入] 頁面。 請確定入口網站 UI 和本機 Web UI 上的裝置序號是相符的。 裝置此時處於鎖定狀態。
5. 登入 [Azure 入口網站](https://portal.azure.com)。
6. 從入口網站下載裝置認證。 移至 [一般] > [裝置詳細資料]。 複製 [裝置密碼]。 裝置密碼會繫結至入口網站中的特定訂單。 

    ![取得裝置認證](media/data-box-deploy-set-up/data-box-device-credentials.png)
    
    
7. 提供您在上一個步驟中從 Azure 入口網站取得的裝置密碼，以登入裝置的本機 Web UI。 按一下 [ **登入**]。
8. 在 [儀表板] 上，確定網路介面已設定。 
    - 如果您的環境中啟用了 DHCP，則會自動設定網路介面。 
    - 如果未啟用 DHCP，請移至 [設定網路介面]，並視需要指派靜態 IP。

    ![裝置儀表板](media/data-box-deploy-set-up/data-box-dashboard-1.png)

設定資料網路介面後，您也可以使用任何 DATA 1 -DATA 3 介面的 IP 位址來存取位於 `https://<IP address of a data network interface>` 的本機 Web UI。 

在裝置設定完成後，您可以連線至裝置共用，並將資料從您的電腦複製到裝置。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解 Azure 資料箱的相關主題，像是：

> [!div class="checklist"]
> * 連接資料箱的纜線
> * 連線至資料箱

請繼續進行下一個教學課程，以了解如何複製資料箱上的資料。

> [!div class="nextstepaction"]
> [將資料複製到 Azure 資料箱](./data-box-deploy-copy-data.md)

