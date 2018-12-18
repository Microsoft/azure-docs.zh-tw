---
title: 無法使用的 SKU 系列 |Microsoft Docs
description: 對於選取的訂用帳戶，此區域中某些 SKU 系列無法使用。
services: Azure Supportability
documentationcenter: ''
author: stevendotwang
manager: rajatk
editor: ''
ms.service: azure-supportability
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/09/2017
ms.author: xingwan
ms.openlocfilehash: 62964d0c5d75168226a35b25e5c256a1b57f3f81
ms.sourcegitcommit: 7d107bb9768b7f32ec5d93ae6ede40899cbaa894
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/16/2017
ms.locfileid: "24870573"
---
# <a name="region-or-sku-unavailable"></a>無法使用區域或 SKU
本文說明如何解決無法存取區域或 VM SKU 的 Azure 訂用帳戶問題。

## <a name="symptoms"></a>徵兆

### <a name="when-deploying-a-virtual-machine-you-receive-one-of-the-following-error-messages"></a>在部署虛擬機器時，您可能會收到下列其中一個錯誤碼︰
```
Code: SkuNotAvailable
Message: The requested size for resource '<resource>' is currently not available in location 
'<location>' zones '<zone>' for subscription '<subscriptionID>'. Please try another size or 
deploy to a different location or zones. See https://aka.ms/azureskunotavailable for details.
```

```
Message: Your subscription doesn’t support virtual machine creation in <location>. Choose a 
different location. Supported locations are <list of locations>
```

```
Code: NotAvailableForSubscription
Message: This size is currently unavailable in this location for this subscription
```

### <a name="when-purchasing-reserved-virtual-machine-instances-you-receive-one-of-the-following-error-messages"></a>在購買保留的虛擬機器執行個體時，您可能會收到下列其中一則錯誤訊息：

```
Message: Your subscription doesn’t support virtual machine reservation in <location>. Choose a 
different location. Supported locations are: <list of locations>  
```

```
Message: This size is currently unavailable in this location for this subscription
```

### <a name="when-creating-a-support-request-to-increase-compute-core-quota-a-region-or-a-sku-family-is-not-available-for-selection"></a>在建立支援要求以增加計算核心配額時，無法選取區域或 SKU 系列。

## <a name="solution"></a>解決方法
我們先建議您考慮符合您的業務需求的替代地區或 SKU。 如果您找不到適合的區域或 SKU，請遵循下列步驟來建立「訂用帳戶管理」[支援要求](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)：


- 在 [基本] 頁面上，選取 [訂用帳戶管理] 作為 [問題類型]、選取 [訂用帳戶]，然後按 [下一步]。

![基本概念刀鋒視窗](./media/SKU-series-unavailable/BasicsSubMgmt.png)


-   在 [問題] 頁面上，選取 [其他一般問題] 作為 [問題類型]。
- 在 [詳細資料] 區段中：
  - 請指出您期望部署虛擬機器，或購買保留的虛擬機器執行個體
  - 請指定區域、SKU 和您計劃要部署或購買的虛擬機器執行個體數目


![問題](./media/SKU-series-unavailable/ProblemSubMgmt.png)

-   輸入聯絡人詳細資訊，然後按一下 [建立]。

![連絡人資訊](./media/SKU-series-unavailable/ContactInformation.png)

## <a name="feedback"></a>意見反應
我們歡迎意見反應和建議！ 請將您的 [建議](https://feedback.azure.com/forums/266794-support-feedback)傳送給我們。 此外，您可以透過 [Twitter](https://twitter.com/azuresupport) 或 [MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure)與我們連絡。

## <a name="learn-more"></a>深入了解
[Azure 支援常見問題集](https://azure.microsoft.com/support/faq)

