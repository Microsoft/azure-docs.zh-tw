---
title: BizTalk 服務的管理與開發工作清單 | Microsoft Docs
description: 部署 Azure BizTalk 服務的規劃和作業協助。
services: biztalk-services
documentationcenter: ''
author: msftman
manager: erikre
editor: ''
ms.assetid: 0ab70b5b-1a88-4ba5-b329-ec51b785010e
ms.service: biztalk-services
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2016
ms.author: deonhe
ms.openlocfilehash: 9edd7261ca62f505ffb4854e3132fae916768f67
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/11/2017
ms.locfileid: "26736669"
---
# <a name="administration-and-development-task-list-in-biztalk-services"></a>BizTalk 服務的管理與開發工作清單

> [!INCLUDE [BizTalk Services is being retired, and replaced with Azure Logic Apps](../../includes/biztalk-services-retirement.md)]

> [!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]

## <a name="getting-started"></a>開始使用
使用「Microsoft Azure BizTalk 服務」時，應考量數項內部部署與雲端架構元件。 開始操作前，請考量下列處理流程：  

| 步驟 | 負責人 | Task | 相關連結 |
| --- | --- | --- | --- |
| 1. |系統管理員 |使用 Microsoft 帳戶或組織帳戶建立 Microsoft Azure 訂用帳戶 |[Azure 入口網站](https://portal.azure.com) |
| 2. |系統管理員 |建立或佈建「BizTalk 服務」。 |[建立 BizTalk 服務](https://msdn.microsoft.com/library/azure/dn232347.aspx) |
| 3. |系統管理員 |註冊您或貴公司的「BizTalk 服務」部署 |[在 BizTalk 服務入口網站註冊和更新 BizTalk 服務部署](https://msdn.microsoft.com/library/azure/hh689837.aspx) |
| 4. |系統管理員 |適用於應用程式採用「BizTalk 配接器服務」連線至內部部署的「企業營運」(LOB) 系統，或是使用「佇列」或「主題目的地」。  建立 Azure 服務匯流排命名空間。 向開發人員提供此命名空間、「服務匯流排簽發者名稱」和「服務匯流排簽發者金鑰」值。 |[作法：建立或修改服務匯流排服務命名空間](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md)與[取得簽發者名稱和簽發者金鑰值](biztalk-issuer-name-issuer-key.md) |
| 5. |開發人員 |在 Visual Studio 中安裝 SDK 並建立「BizTalk 服務」專案。 |[安裝 Azure BizTalk 服務 SDK](https://msdn.microsoft.com/library/azure/hh689760.aspx) 與[在 Azure 上建立豐富傳訊端點](https://msdn.microsoft.com/library/azure/hh689766.aspx) |
| 6. |開發人員 |將「BizTalk 服務」專案部署至您於 Azure 託管的「BizTalk 服務」。 |[部署和重新整理 BizTalk 服務專案](https://msdn.microsoft.com/library/azure/hh689881.aspx) |
| 7. |系統管理員 |適用於使用 EDI。  您可在「Microsoft Azure BizTalk 服務入口網站」上新增「合作夥伴」和建立「合約」。 當您建立「合約」時，可將橋接器和/或開發人員建立的「轉換」新增至「合約」設定。 |[在 BizTalk 服務入口網站上設定 EDI、 AS2 和 EDIFACT](https://msdn.microsoft.com/library/azure/hh689853.aspx) |
| 8. |系統管理員 |使用 [REST](https://msdn.microsoft.com/library/azure/dn232347.aspx) \(英文\)，監視包括效能度量在內的 BizTalk 服務健康情況。 |[BizTalk 服務：儀表板、監視和調整索引標籤](http://go.microsoft.com/fwlink/p/?LinkID=302281) |
| 9. |系統管理員 |使用 Microsoft Azure BizTalk 服務入口網站來管理 BizTalk 服務使用的構件，並追蹤橋接器檔案處理的訊息。 |[使用 BizTalk 服務入口網站](https://msdn.microsoft.com/library/azure/dn874043.aspx) |
| 10. |系統管理員 |建立備份計劃以備份「BizTalk 服務」。 |[業務持續性和 BizTalk 服務的災害復原](https://msdn.microsoft.com/library/azure/dn509557.aspx) |

## <a name="next-steps"></a>後續步驟
[教學課程和範例](https://msdn.microsoft.com/library/azure/hh689895.aspx)

[在 Visual Studio 中建立專案](https://msdn.microsoft.com/library/azure/hh689811.aspx)

[安裝 Azure BizTalk 服務 SDK](https://msdn.microsoft.com/library/azure/hh689760.aspx)

## <a name="concepts"></a>概念
[在 Visual Studio 中建立專案](https://msdn.microsoft.com/library/azure/hh689811.aspx)  
[EDI、AS2 和 EDIFACT 傳訊 (企業對企業) 中建立專案](https://msdn.microsoft.com/library/azure/hh689898.aspx)  

## <a name="other-resources"></a>其他資源
[新增來源、目的地和橋接器傳訊端點](https://msdn.microsoft.com/library/azure/hh689877.aspx)  
[學習和建立訊息對應與轉換](https://msdn.microsoft.com/library/azure/hh689905.aspx)  
[使用 BizTalk 配接器服務 (BAS)](https://msdn.microsoft.com/library/azure/hh689889.aspx)  
[Azure BizTalk 服務](http://go.microsoft.com/fwlink/p/?LinkID=303664)

