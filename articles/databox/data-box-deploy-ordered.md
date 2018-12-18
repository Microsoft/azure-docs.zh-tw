---
title: 訂購 Microsoft Azure 資料箱 | Microsoft Docs
description: 深入了解部署必要條件以及如何訂購 Azure 資料箱
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 10/10/2018
ms.author: alkohli
ms.openlocfilehash: d02557ecd84ea14db297ee07f7055a08304e7fcd
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49091210"
---
# <a name="tutorial-order-azure-data-box"></a>教學課程：訂購 Azure 資料箱

Azure 資料箱是一項混合式解決方案，可讓您以快速、簡便而可靠的方式，將內部部署資料匯入 Azure 中。 您可將資料傳輸至 Microsoft 提供的 80 TB (可用容量) 儲存裝置，然後將裝置寄回。 這項資料隨後會上傳至 Azure。

本教學課程說明如何訂購 Azure 資料箱。 在本教學課程中，您會了解：

> [!div class="checklist"]
> * 註冊資料箱
> * 訂購資料箱
> * 追蹤訂單狀態
> * 取消訂單

## <a name="prerequisites"></a>必要條件

在您部署裝置之前，請完成下列資料箱服務與裝置的組態必要條件。

### <a name="for-service"></a>針對服務

在您開始前，請確定：
- 您擁有的 Microsoft Azure 儲存體帳戶具有存取認證。
- 請確定您用於資料箱服務的訂用帳戶是下列其中一種類型：
    - Microsoft Enterprise 合約 (EA)。 深入了解 [EA 訂用帳戶](https://azure.microsoft.com/pricing/enterprise-agreement/)。
    - 雲端解決方案提供者 (CSP)。 深入了解 [Azure CSP 方案](https://docs.microsoft.com/azure/cloud-solution-provider/overview/azure-csp-overview)。
    - 使用量 - 隨用隨付。 關於 Azure [隨用隨付訂用帳戶](https://azure.microsoft.com/offers/ms-azr-0003p/)的詳細資訊。

- 請確定您有訂用帳戶的擁有者或參與者存取權，才能建立資料箱訂單。

### <a name="for-device"></a>針對裝置

在您開始前，請確定：
- 您已將主機電腦連線到資料中心網路。 資料箱會從這部電腦複製資料。 您的主機電腦必須執行支援的作業系統，如 [Azure 資料箱系統需求](data-box-system-requirements.md)中所述。
- 您的資料中心必須有高速網路。 我們強烈建議您具有至少一個 10 GbE 的連線。 如果無法使用 10 GbE 連線，也可以使用 1 GbE 資料連結，但是複製速度會受到影響。


## <a name="order-data-box"></a>訂購資料箱

請在 Azure 入口網站中執行下列步驟，以訂購裝置。

1. 使用您的 Microsoft Azure 認證在以下 URL 登入：[https://portal.azure.com](https://portal.azure.com)。
2. 按一下 [+ 建立資源]，然後搜尋「Azure 資料箱」。 按一下 [Azure 資料箱]。
    
   [![搜尋 Azure 資料箱 1](media/data-box-deploy-ordered/search-azure-data-box1.png)](media/data-box-deploy-ordered/search-azure-data-box1.png#lightbox)

3. 按一下頁面底部的 [新增] 。

4. 確認您的區域是否適用資料箱服務。 輸入或選取下列資訊，然後按一下 [套用]。 
    |設定  |值  |
    |---------|---------|
    |訂用帳戶     | 為資料箱服務選取 EA、CSP 或隨用隨付訂用帳戶。 <br> 訂用帳戶會連結到您的帳單帳戶。       |
    |傳輸類型     | 選取 [匯入至 Azure]。        |
    |來源國家/地區     |   選取您的資料目前所在的國家/地區。         |
    |目的地 Azure 區域     |     選取要傳輸資料的 Azure 區域。        |

5. 選取 [資料箱]。 單一訂單的解決方案最大容量是 80 TB。 您可以建立多份訂單以訂購更大的資料大小。

      [![選取資料箱選項 1](media/data-box-deploy-ordered/select-data-box-option1.png)](media/data-box-deploy-ordered/select-data-box-option1.png#lightbox)

6. 在 [訂單] 中，指定 [訂單詳細資料]。 輸入或選取下列資訊，然後按 [下一步]。
    
    |設定  |值  |
    |---------|---------|
    |名稱     |  提供用來追蹤訂單的易記名稱。 <br> 名稱長度可介於 3 到 24 個字元之間，且可以是字母、數字和連字號。 <br> 名稱必須以字母或數字為開頭或結尾。      |
    |資源群組     |   使用現有的群組或建立新群組。 <br> 資源群組是適用於資源而可一併管理或部署的邏輯容器：         |
    |目的地 Azure 區域     | 選取儲存體帳戶的區域。 <br> 如需詳細資訊，請移至[區域可用性](data-box-overview.md#region-availability)。        |
    |儲存體帳戶     | 根據指定的 Azure 區域，從現有儲存體帳戶的篩選清單中選取一或多個儲存體帳戶。 資料箱可以與最多 10 個儲存體帳戶連結。 <br> 您也可以建立新的**一般用途 v1**、**一般用途 v2** 或 **Blob 儲存體帳戶**。        |
    
7. 在 [交貨地址] 中，提供您的姓名、公司的名稱和郵寄地址，以及有效的電話號碼。 按一下 [驗證地址]。 服務會驗證交貨地址以確認服務可用性。 如果服務可提供至指定的交貨地址，您將會收到該項通知。 按 [下一步] 。

8. 在 [通知詳細資料] 中，指定電子郵件地址。 服務會將關於任何訂單狀態更新的電子郵件通知傳送至指定的電子郵件地址。

    建議您使用群組電子郵件，以便在群組中的管理員離開時繼續接收通知。

9. 檢閱與訂單、連絡人、通知和隱私權條款有關的資訊**摘要**。 請勾選隱私權條款合約的對應方塊。

10. 按一下 [訂單]。 建立訂單需要幾分鐘的時間。 


## <a name="track-the-order"></a>追蹤訂單狀態

在您下訂單之後，可以從 Azure 入口網站來追蹤訂單狀態。 請移至您的訂單，然後移至 [概觀] 以檢視狀態。 入口網站會顯示處於 [已訂購] 狀態的訂單。

如果裝置無法使用，您會收到通知。 如果可使用服務，Microsoft 會識別要寄送的裝置，並準備出貨。 在裝置準備期間，會執行下列動作：

- 系統會針對與裝置相關聯的每個儲存體帳戶建立 SMB 共用。 
- 針對每個共用，會產生例如使用者名稱和密碼的存取認證。
- 也會產生可協助將裝置解除鎖定的裝置密碼。 
- 資料箱會鎖定，防止任何對裝置的未經授權存取。

裝置準備完成後，入口網站會顯示訂單處於 [已處理] 狀態。

![資料箱訂單已處理](media/data-box-overview/data-box-order-status-processed.png)

接著，Microsoft 會準備裝置，並透過區域貨運公司派送給您。 在裝置送達後，您會收到追蹤號碼。 入口網站會顯示訂單處於 [已分派] 狀態。

![資料箱訂單分派](media/data-box-overview/data-box-order-status-dispatched.png)

## <a name="cancel-the-order"></a>取消訂單

若要取消此訂單，請在 Azure 入口網站中移至 [概觀]，然後從命令列按一下 [取消]。

下訂單之後，您可以在訂單狀態標示為已處理之前的任何時間點，取消訂單。
 
若要刪除已取消的訂單，請移至 [概觀]，然後從命令列按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解 Azure 資料箱的相關主題，像是：

> [!div class="checklist"]
> * 部署資料箱的必要條件
> * 訂購資料箱
> * 追蹤訂單狀態
> * 取消訂單

請繼續進行下一個教學課程，以了解如何設定資料箱。

> [!div class="nextstepaction"]
> [設定您的 Azure 資料箱](./data-box-deploy-set-up.md)


