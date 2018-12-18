---
title: 建立 Marketplace 解決方案範本的指南 | Microsoft Docs
description: 如何建立、認證和部署一個多 VM 映像解決方案範本供他人在 Azure Marketplace 上購買的詳細指示。
services: marketplace-publishing
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: e14e05f2-2385-4ce0-b351-0747cb74ba19
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/27/2016
ms.author: hascipio; v-divte
ms.openlocfilehash: 0e1c049596b5231ad72590fc17a73cd5aa457e4d
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "40246403"
---
# <a name="guide-to-create-a-solution-template-for-azure-marketplace"></a>建立 Azure Marketplace 解決方案範本的指南
完成步驟 1 [帳戶建立和註冊][link-acct-creation]之後，我們引導您依照[建立解決方案範本的技術性必要條件](marketplace-publishing-solution-template-creation-prerequisites.md)建立與 Azure 相容的解決方案範本。 現在，我們將逐步引導您在 Azure Marketplace 的[發佈入口網站][link-pubportal]上建立一個多 VM 的解決方案範本。

## <a name="create-your-solution-template-offer-in-the-publishing-portal"></a>在發佈入口網站中建立解決方案範本供應項目
移至 [https://publish.windowsazure.com](http://publish.windowsazure.com)。 當第一次登入 [發佈入口網站](https://publish.windowsazure.com/)時，請使用與您公司的賣方設定檔註冊時相同的帳戶。 稍後您可以在發佈入口網站中將您公司的任何員工新增為共同管理員。

### <a name="1-select-solution-templates"></a>1.選取 [解決方案範本]
  ![繪圖][img-pubportal-menu-sol-templ]

### <a name="2-create-a-new-solution-template"></a>2.建立新的解決方案範本
  ![繪圖][img-pubportal-sol-templ-new]

### <a name="3-start-with-topologies"></a>3.開始使用拓樸
解決方案範本是所有其拓撲的「父項」。 您可以在一個供應項目/解決方案範本中定義多個拓撲。 當供應項目進入預備環境時，它的所有拓撲也會一起進入。 請遵循下列步驟來定義您的供應項目：     

* 建立拓撲：「拓樸識別項」通常是解決方案範本的拓撲名稱。 拓撲識別項用於 URL，如下所示：

  Azure Marketplace： http://azure.microsoft.com/marketplace/partners/{PublisherNamespace}/{OfferIdentifier}{TopologyIdentifier}

  Azure 入口網站： https://portal.azure.com/#gallery/{PublisherNamespace}.{OfferIdentifier}{TopologyIdentifier}
* 加入新的版本。

### <a name="4-get-your-topology-versions-certified"></a>4.讓您的拓撲版本取得認證
上傳一個 zip 檔案，其包含佈建該特定拓撲版本所需的所有檔案。 此 zip 檔案必須包含下列項目：

* *mainTemplate.json* 和 *createUiDefinition.json* 檔案在其根目錄上。
* 任何連結的範本和所有必要的指令碼。

  > [!TIP]
  > 在您的開發人員建立解決方案範本拓撲和取得認證的同時，貴公司的業務、行銷，和/或法律部門就可以處理行銷和法律內容。
  >
  >

## <a name="next-steps"></a>後續步驟
在建立方案範本並上傳 ZIP 檔案後，請先遵循 [Marketplace 行銷內容指南](marketplace-publishing-push-to-staging.md) 中的指引進行，然後再將供應項目推送至預備環境。 或者，若要查看完整的 Marketplace 發佈文章集合，請瀏覽 [快速入門：如何將供應項目發佈至 Azure Marketplace](marketplace-publishing-getting-started.md)。

您也可能對以下相關文章有興趣：

* VM 映像： [關於 Azure 中的虛擬機器映像](https://msdn.microsoft.com/library/azure/dn790290.aspx)
* VM 延伸模組：[Azure VM 延伸模組與功能](../virtual-machines/extensions/features-windows.md)
* Azure Resource Manager：[撰寫 Azure Resource Manager 範本 (英文)](../azure-resource-manager/resource-group-authoring-templates.md)和[簡單範本範例](https://github.com/rjmax/ArmExamples)
* 儲存體帳戶節流：[如何監視儲存體帳戶節流](http://blogs.msdn.com/b/mast/archive/2014/08/02/how-to-monitor-for-storage-account-throttling.aspx)和[進階儲存體](../virtual-machines/windows/premium-storage.md#scalability-and-performance-targets)

[img-pubportal-menu-sol-templ]:media/marketplace-publishing-solution-template-creation/pubportal-menu-solution-templates.png
[img-pubportal-sol-templ-new]:media/marketplace-publishing-solution-template-creation/pubportal-solution-template-new.png
[link-acct-creation]:marketplace-publishing-accounts-creation-registration.md
[link-pubportal]:https://publish.windowsazure.com
