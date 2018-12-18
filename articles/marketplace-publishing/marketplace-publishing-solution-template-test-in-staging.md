---
title: 針對 Marketplace 測試您的解決方案範本供應項目 | Microsoft Docs
description: 了解如何針對 Azure Marketplace 測試您的解決方案範本供應項目。
services: marketplace-publishing
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: ef8f9b5e-b98c-49f3-913f-cdf772c14c12
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/04/2015
ms.author: hascipio; v-divte
ms.openlocfilehash: da1fc4713fd1d832c7ba91226f72cbef63b241bc
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39714820"
---
# <a name="test-your-solution-template-offer-in-staging"></a>在預備環境中測試您的解決方案範本供應項目
預備環境代表將您的供應項目部署在私人的「沙箱」中，您可以在推送到生產環境之前在沙箱中測試與驗證其功能。 供應項目會出現在預備環境中，就如同客戶已部署該項目一樣。 您的供應項目必須經過認證才能推送至預備環境。

預備供應項目之後，您可以在 [Azure 入口網站](https://portal.azure.com/)中檢視並測試供應項目。

請遵循下列步驟，將您的供應項目推送至預備環境並在 [Azure 入口網站](https://portal.azure.com/)中進行測試：

1. 瀏覽至[發佈入口網站](https://publish.windowsazure.com) > [解決方案範本] 索引標籤 > 您的供應項目 > [發佈] > [推送至預備環境]。
2. 提供您將用來預覽和測試供應項目的 Azure 訂用帳戶清單。
3. 使用上一個步驟中使用的訂用帳戶 ID，登入 Azure Preview 入口網站。
4. 在 Azure Preview 入口網站中，將以下提到的重點至少執行一回合測試：
   * 請確定該行銷內容可在 Azure Marketplace 中正確顯示。
   * 拓撲的端對端部署。
   * 執行效能測試和壓力測試。
   * 確定您的拓撲遵守最佳作法。

## <a name="next-steps"></a>後續步驟
如果您很滿意結果，則可以繼續進行最後的供應項目發佈階段，也就是 **步驟 4**：[將您的供應項目部署至 Marketplace](marketplace-publishing-push-to-production.md)。 否則，請在您的供應項目中進行變更並再次要求憑證。

> [!NOTE]
> 若為行銷內容變更，則不需要憑證。
> 
> 

如需所有發行者工作的指南，請參閱 [快速入門：如何將供應項目發佈至 Azure Marketplace](marketplace-publishing-getting-started.md) 。

