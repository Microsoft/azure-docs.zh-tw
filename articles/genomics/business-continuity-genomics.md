---
title: 商務持續性 - Microsoft Genomics | Microsoft Docs
titleSuffix: Azure
description: 本概觀說明 Microsoft Genomics 針對商務持續性和災害復原所提供的功能。 深入了解可使用哪些選項，從可能造成資料遺失的干擾性事件 (例如 Azure 區域中斷) 復原。
keywords: 商務持續性，災害復原
services: genomics
author: grhuynh
manager: cgronlun
ms.author: grhuynh
ms.service: genomics
ms.topic: article
ms.date: 04/06/2018
ms.openlocfilehash: be678648ab93fcbdfd0a0baa1b01dcb273060ce2
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45729447"
---
# <a name="overview-of-business-continuity-with-microsoft-genomics"></a>使用 Microsoft Genomics 的商務持續性概觀
本概觀說明 Microsoft Genomics 針對商務持續性和災害復原所提供的功能。 深入了解可使用哪些選項，從可能造成資料遺失的干擾性事件 (例如 Azure 區域中斷) 復原。 


## <a name="microsoft-genomics-features-that-support-business-continuity"></a>支援商務持續性的 Microsoft Genomics 功能 
雖然罕見，但是 Azure 資料中心也有可能發生中斷，造成可能持續數分鐘到數小時的業務中斷。 當發生中斷時，資料中心目前正在執行的所有作業都會失敗，而 Microsoft Genomics 服務不會自動將作業重新提交到次要地區。 

* 其中一個選項，是在資料中心中斷結束後等待資料中心重新上線。 如果中斷時間很短，Microsoft Genomics 服務會自動偵測失敗的作業，且工作流程會自動重新啟動。

* 另一個選項是在其他資料區域中主動提交工作流程。 Microsoft Genomics 會在數個 [Azure 區域](https://azure.microsoft.com/regions/services/)中部署執行個體，而且每個區域專屬執行個體都是獨立的。 如果其中一個 Microsoft Genomics 執行個體發生區域性的本機失敗，其他執行 Microsoft Genomics 執行個體的區域會繼續處理作業。 這個轉換到替代區域的作業可由使用者控制，而且隨時可以進行。


### <a name="manually-failover-microsoft-genomics-workflows-to-another-region"></a>手動將 Microsoft Genomics 工作流程容錯移轉到另一個區域
如果區域性資料中心發生中斷情形，您可以根據您的個別資料主權和商務持續性需求，選擇在次要地區中提交 Microsoft Genomics 工作流程。 若要手動容錯移轉 Microsoft Genomics 工作流程，您要使用不同的區域專屬  Genomics 帳戶，並且使用適當的區域專屬 Genomics 和儲存體帳戶認證來提交作業。

具體而言，您需要：
* 使用 Azure 入口網站，在次要地區建立 Genomics 帳戶。 
* 將您的輸入資料遷移至次要地區中的儲存體帳戶，並且在次要地區中設定輸出資料夾。
* 在次要地區中提交工作流程。

還原原始區域時，Microsoft Genomics 服務不會從次要地區將資料遷移回到原始區域。 您可以選擇從次要地區將輸入和輸出檔案移回到原始區域。  如果您選擇移動其資料，由於這不在 Genomics 的服務範圍內，因此與資料移動相關的所有費用需由您負責。 

### <a name="preparing-for-a-possible-region-specific-outage"></a>為可能發生的區域性中斷做好準備
如果您希望在發生資料中心中斷時可更快速復原，您可以採取幾個步驟來減少手動將 Microsoft Genomics 工作流程重新提交到次要地區的時間：

* 找出適當的次要地區，在該區域主動建立 Genomics 帳戶
* 在主要和次要地區複製您的資料，讓您的資料在次要地區立即可供取用。 這個操作可以手動完成，或是使用 Azure 儲存體中可用的 [geo-redundant storage](https://docs.microsoft.com/azure/storage/common/storage-redundancy) 功能來完成。 

## <a name="next-steps"></a>後續步驟
在本文中，您已了解使用 Microsoft Genomics 服務時的商務持續性和災害復原選項。 如需 Azure 中一般商務持續性和災害復原的詳細資訊，請參閱 [Azure 復原技術指引](https://docs.microsoft.com/azure/architecture/resiliency/recovery-loss-azure-region)。 