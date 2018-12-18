---
title: 透過 Azure 入口網站調整串流端點 | Microsoft Docs
description: 本教學課程將逐步引導您完成使用 Azure 入口網站調整串流端點的步驟。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 1008b3a3-2fa1-4146-85bd-2cf43cd1e00e
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/10/2017
ms.author: juliako
ms.openlocfilehash: f2a14f2622d78a4222a8518172eb1ce8ed9e6637
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785637"
---
# <a name="scale-streaming-endpoints-with-the-azure-portal"></a>透過 Azure 入口網站調整串流端點
## <a name="overview"></a>概觀

> [!NOTE]
> 若要完成此教學課程，您需要 Azure 帳戶。 如需詳細資料，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。 
> 
> 

**進階**串流端點適合進階工作，提供專用並能靈活調整的頻寬容量。 擁有**進階**串流端點的客戶預設會取得一個串流單位 (SU)。 藉由新增 SU 可以調整串流端點。 每個 SU 可為應用程式提供額外的頻寬容量。 如需有關串流端點類型和 CDN 組態的詳細資訊，請參閱[串流端點概觀](media-services-streaming-endpoints-overview.md)主題。
 
本主題說明如何調整串流端點。

如需定價詳細資料的相關資訊，請參閱＜ [媒體服務定價詳細資料](http://go.microsoft.com/fwlink/?LinkId=275107)＞。

## <a name="scale-streaming-endpoints"></a>調整串流端點

若要變更串流端點的數目，請執行下列操作：

1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您的 Azure 媒體服務帳戶。
2. 在 [設定] 視窗中，選取 [串流端點]。
3. 按一下您要調整的串流端點。 

    > [!NOTE] 
    > 您只能調整 [Premium] \(進階\) 串流端點。

4. 移動滑桿以指定資料流處理單位的數目。

    ![串流端點](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints3.png)

## <a name="next-steps"></a>後續步驟
檢閱媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

