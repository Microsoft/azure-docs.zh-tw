---
title: 更換儲存體存取金鑰之後更新媒體服務 | Microsoft Docs
description: 本文章會教您如何在更換儲存體存取金鑰之後更新媒體服務。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: a892ebb0-0ea0-4fc8-b715-60347cc5c95b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/30/2018
ms.author: milanga;cenkdin;juliako
ms.openlocfilehash: e8d8067fcf30b16dd3dbc7f6cf50129d837aa3a5
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306851"
---
# <a name="update-media-services-after-rolling-storage-access-keys"></a>更換儲存體存取金鑰之後更新媒體服務

在建立新的 Azure 媒體服務 (AMS) 帳戶時，您需要選取用來儲存媒體內容的 Azure 儲存體帳戶。 您可以在媒體服務帳戶新增一個以上的儲存體帳戶。 本文說明如何更換儲存體金鑰。 其中也示範如何在媒體帳戶新增儲存體帳戶。 

若要執行本文中描述的動作，您應該使用 [Azure Resource Manager API](https://docs.microsoft.com/rest/api/media/mediaservice) 和 [Powershell](https://docs.microsoft.com/powershell/module/azurerm.media)。  如需詳細資訊，請參閱[如何使用 PowerShell 和 Resource Manager 來管理 Azure 資源](../../azure-resource-manager/powershell-azure-resource-manager.md)。

## <a name="overview"></a>概觀

建立新的儲存體帳戶時，Azure 會產生兩個 512 位元儲存體存取金鑰，用來驗證儲存體帳戶的存取權。 為了讓儲存體連線更加安全，建議您定期重新產生並更換儲存體存取金鑰。 您會收到兩個存取金鑰(主要和次要)，這樣當您重新產生其他存取金鑰時，您就可以使用其中一個存取金鑰保持儲存體連線不中斷。 此程序也稱為 「 更換存取金鑰 」。

媒體服務取決於所提供的儲存體金鑰。 具體而言，定位器是要串流傳送或下載您的資產，這會根據指定的儲存體存取金鑰而定。 建立 AMS 帳戶時，它預設會相依於主要儲存體存取金鑰，但身為使用者，您可以更新 AMS 所擁有的儲存體金鑰。 您必須確定會讓媒體服務知道下列本文所述步驟所要使用的金鑰。  

>[!NOTE]
> 如果您有多個儲存體帳戶，請針對每一個儲存體帳戶執行此程序。 更換儲存體金鑰無固定順序。 您可以先更換次要金鑰再更換主要金鑰，反之亦然。
>
> 在生產帳戶上執行本文描述的步驟之前，請事先在測試用帳戶上進行測試。
>

## <a name="steps-to-rotate-storage-keys"></a>更換儲存體金鑰的步驟 
 
 1. 透過 PowerShell Cmdlet 或 [Azure](https://portal.azure.com/) 入口網站來變更儲存體帳戶主要金鑰。
 2. 搭配適當的參數呼叫 Sync-AzureRmMediaServiceStorageKeys Cmdlet，以強制媒體帳戶接收儲存體帳戶金鑰
 
    以下範例示範如何將金鑰同步到儲存體帳戶。
  
         Sync-AzureRmMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId
  
 3. 請等候約一小時。 確認資料流案例運作無誤。
 4. 透過 PowerShell Cmdlet 或 Azure 入口網站變更儲存體帳戶次要金鑰。
 5. 搭配適當的參數呼叫 Sync-AzureRmMediaServiceStorageKeys PowerShell，以強制媒體帳戶接收儲存體帳戶金鑰。 
 6. 請等候約一小時。 確認資料流案例運作無誤。
 
### <a name="a-powershell-cmdlet-example"></a>PowerShell Cmdlet 範例 

以下範例示範如何取得儲存體帳戶並與 AMS 帳戶同步。

    $regionName = "West US"
    $resourceGroupName = "SkyMedia-USWest-App"
    $mediaAccountName = "sky"
    $storageAccountName = "skystorage"
    $storageAccountId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$storageAccountName"

    Sync-AzureRmMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId

 
## <a name="steps-to-add-storage-accounts-to-your-ams-account"></a>將儲存體帳戶新增到 AMS 帳戶的步驟

下列文章說明如何將儲存體帳戶新增到 AMS 帳戶：[將多個儲存體帳戶附加到媒體服務帳戶](meda-services-managing-multiple-storage-accounts.md)。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

### <a name="acknowledgments"></a>通知
我們想要向下列為建立此文件貢獻心力的人員致謝：Cenk Dingiloglu、Milan Gada、Seva Titov。
