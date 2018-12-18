---
title: 建立 Marketplace 供應項目所需的各種入口網站概觀 | Microsoft Docs
description: 建立 Marketplace 供應項目所需的各種入口網站概觀
services: marketplace-publishing
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: 89ce82b3-c28a-4b0d-b37a-db3112160a4e
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/27/2016
ms.author: hascipio
ms.openlocfilehash: 093d3ee3ecce89c86a12e7e216072ece57f82f9d
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39715908"
---
# <a name="portals-you-will-need"></a>您需要的入口網站
在您開始發行供應項目程序之前，讓我們簡介您所需的各種入口網站。 以下是有關入口網站的簡短摘要，包括：開發人員中心、Azure 發佈入口網站和 Azure 入口網站 (依您將與之互動的順序顯示)。                                                                            

## <a name="developer-center"></a>開發人員中心
[http://dev.windows.com/registration?accountprogram=azure](http://dev.windows.com/registration?accountprogram=azure)

### <a name="description"></a>說明
建立您的 Microsoft 開發人員中心帳戶是一次性的工作。 當您嘗試建立開發人員中心帳戶之前，請確認公司確實未曾擁有開發人員中心帳戶。 在這個程序期間，我們會收集銀行帳戶資訊、稅務資訊和公司地址資訊。

> [!NOTE]
> 如果您只要發佈免費的供應項目 (或自備授權)，即不需要稅捐和銀行資訊。
> 
> 

### <a name="identityaccount-used"></a>使用的身分識別/帳戶
在理想情況下，這是通訊群組清單或安全性群組 (例如 azurepublishing@partnercompany.com)。 此通訊群組清單或安全性群組 **必須** 已註冊為 Microsoft 帳戶。

> [!TIP]
> 建議使用通訊群組清單或安全性群組，因為它會移除任何個別帳戶的相依性；雖然也會使用個別帳戶。
> 
> 

## <a name="publishing-portal"></a>發佈入口網站
[https://publish.windowsazure.com](https://publish.windowsazure.com)

### <a name="description"></a>說明
此入口網站可讓您處理供應項目並加以發佈 (不論是行銷、價格、發佈憑證等)。

### <a name="identityaccount-used"></a>使用的身分識別/帳戶
第一次登入發佈入口網站時必須使用上述的通訊群組清單或安全性群組。 稍後，即可將其他使用者新增為共同管理員。 這就是對應至開發人員中心註冊資料的方式。

## <a name="azure-portal"></a>Azure 入口網站
[https://portal.azure.com](https://portal.azure.com)

### <a name="description"></a>說明
在此入口網站中，您可於 Azure Marketplace 中檢視您預備與發佈的供應項目 (適用於 VM、方案範本和以 Azure 資源管理員為基礎的開發人員服務)。

### <a name="identityaccount-used"></a>使用的身分識別/帳戶
從發佈入口網站預備供應項目時，必須將訂用帳戶識別碼列入允許清單。 登入此入口網站來測試預備的供應項目時，必須使用相同的訂用帳戶 (具有相關聯的使用者名稱和密碼)。

## <a name="see-also"></a>另請參閱
* [使用者入門：如何將供應項目發佈至 Azure Marketplace](marketplace-publishing-getting-started.md)

