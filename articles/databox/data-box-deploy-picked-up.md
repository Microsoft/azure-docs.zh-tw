---
title: 寄回 Microsoft Azure 資料箱 | Microsoft Docs
description: 了解如何將您的 Azure 資料箱寄給 Microsoft
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 10/03/2018
ms.author: alkohli
ms.openlocfilehash: 7676360d71dab4da58693221645517c69b56dff8
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49090683"
---
# <a name="tutorial-return-azure-data-box-and-verify-data-upload-to-azure"></a>教學課程：送回 Azure 資料箱，並確認資料上傳至 Azure

本教學課程將說明如何送回 Azure 資料箱，並確認資料已上傳至 Azure。

在本教學課程中，您將了解如下所列的主題：

> [!div class="checklist"]
> * 將資料箱寄送給 Microsoft
> * 確認資料上傳至 Azure
> * 清除資料箱的資料

## <a name="prerequisites"></a>必要條件

在開始之前，請確定您已完成[教學課程：將資料複製到 Azure 資料箱並確認](data-box-deploy-copy-data.md)。

## <a name="ship-data-box-back"></a>寄回資料箱

1. 請確認裝置的電源已關閉，然後移除纜線。 纏繞裝置隨附的電源線，並將其安全地放在裝置背後。
2. 如果裝置是在美國運送，請確定出貨標籤已顯示在電子墨水顯示器上，並安排貨運業者取貨。 如果標籤損毀、遺失或未顯示在電子筆墨顯示器上，請從 Azure 入口網站下載出貨標籤，然後黏貼在裝置上。 移至 [概觀 > 下載出貨標籤]。 

    如果裝置是在歐洲運送，電子墨水顯示器不會顯示出貨標籤。 相反地，回寄出貨標籤會在正面出貨標籤底下的透明袋中。 移除舊的出貨標籤，並確定出貨標籤清楚可見。
    
3. 如果是在美國辦理裝置退貨，請排程由 UPS 取貨。 如果是在歐洲經由 DHL 辦理裝置退貨，請造訪 DHL 的網站並且指定空運提單號碼，要求他們取貨。 移至該國家/地區的 DHL Express 網站，然後選擇 [Book a Courier Collection > eReturn Shipment] \(預約快遞收貨 > eReturn 出貨\)。 

    指定運貨單號碼，然後按一下 [Schedule Pickup] (排程取貨) 以安排取貨。

4. 一旦貨運業者收取資料箱並進行掃描，入口網站的訂單狀態會更新為 [已取貨]。 此外，也會顯示追蹤識別碼。

## <a name="verify-data-upload-to-azure"></a>確認資料上傳至 Azure

當 Microsoft 收到並掃描裝置之後，訂單狀態會更新為 [已收到]。 然後，裝置會進行實體驗證，以檢查是否有損毀或遭到竄改的跡象。 

完成驗證之後，資料箱會連線到 Azure 資料中心的網路。 資料複製會自動啟動。 視資料大小而定，複製作業可能需要數小時到數天才能完成。 您可以在入口網站中監視複製作業進度。

一旦複製完成，訂單狀態會更新為 [已完成]。

請先確認您的資料位於儲存體帳戶中，再從來源予以刪除。 

## <a name="erasure-of-data-from-data-box"></a>清除資料箱的資料
 
 完成上傳到 Azure 之後，資料箱會根據 [NIST SP 800-88 修訂 1 指導方針](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi)清除磁碟中的資料。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解 Azure 資料箱的相關主題，像是：

> [!div class="checklist"]
> * 將資料箱寄送給 Microsoft
> * 確認資料上傳至 Azure
> * 清除資料箱的資料

請前往下列文章，了解如何透過本機 Web UI 管理資料箱。

> [!div class="nextstepaction"]
> [使用本機 Web UI 來管理 Azure 資料箱](./data-box-local-web-ui-admin.md)


