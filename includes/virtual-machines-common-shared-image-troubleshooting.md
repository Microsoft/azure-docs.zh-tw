---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: axayjo
ms.service: virtual-machines
ms.topic: include
ms.date: 09/20/2018
ms.author: akjosh; cynthn
ms.custom: include file
ms.openlocfilehash: 2902fe53e7f3ffb86dc727ebcdc2200ba02ac203
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47046484"
---
如果您在共用映像資源庫、映像定義及映像版本上執行任何作業時遇到問題，請再次以偵錯模式執行失敗的命令。 透過 CLI 傳遞 **-debug** 參數以及透過 PowerShell 傳遞 **-Debug** 參數，可啟動偵錯模式。 一旦您找到錯誤，請遵循本文件來針對錯誤進行疑難排解。


## <a name="unable-to-create-a-shared-image-gallery"></a>無法建立共用映像資源庫

可能的原因：

資源庫名稱無效。

資源庫名稱允許的字元為大寫或小寫字母、數字、點和句點。 資源庫名稱不能有連字號。 變更資源庫名稱並再試一次。 

資源庫名稱在您的訂用帳戶內不是唯一的。

挑選另一個資源庫名稱並再試一次。


## <a name="unable-to-create-an-image-definition"></a>無法建立映像定義 

可能的原因：

映像定義名稱無效。

映像定義允許的字元為大寫或小寫字母、數字、點、破折號和句點。 變更映像定義名稱並再試一次。

未填入可供建立映像定義的必要屬性。

名稱、發行者、供應項目、sku 和 OS 類型等都是必要屬性。 請確認是否正在傳遞所有屬性。

請確定映像定義的 **OSType** (Linux 或 Windows) 是您用來建立映像版本的相同來源受控映像。 


## <a name="unable-to-create-an-image-version"></a>無法建立映像版本 

可能的原因：

映像版本名稱無效。

映像版本允許的字元是數字及句點。 數字必須在 32 位元整數的範圍內。 格式：*MajorVersion.MinorVersion.Patch*。 變更映像版本名稱並再試一次。

找不到從中建立映像版本的來源受控映像。 

檢查來源映像是否存在且位於與映像版本相同的區域。

受控映像未完成佈建。

請確定來源受控映像的佈建狀態是 [已成功]。

尚未支援來源區域。

使用下表來查看是否支援預定的來源區域：
<br>

| 在「來源地區」中建立資源庫   | 將版本複寫至「目標區域」 |
|----------------------------------------|-------------------------------------------|
| 美國中西部                        | 所有 Azure 公用雲端區域            |
| 美國中南部                       |                                           |
| 美國東部 2                              |                                           |
| 東南亞                         |                                           |
| 西歐                            |                                           |

<br>

目標區域清單必須包含映像版本的來源區域。 確定您已在您希望 Azure 將您的映像版本複寫至的目標區域清單中包含來源區域。

未完成複寫至所有目標區域。

使用 **--expand ReplicationStatus** 旗標，以檢查是否已完成複寫到所有指定的目標區域。 如果尚未完成，最多等待 6 小時讓作業完成。 如果失敗，再次執行命令來建立和複寫映像版本。 如果映像版本要複寫至大量目標區域，請考慮分階段進行複寫。

## <a name="unable-to-create-a-vm-or-a-scale-set"></a>無法建立 VM 或擴展集 

可能的原因：

嘗試建立 VM 或虛擬機器擴展集的使用者沒有映像版本的讀取權限。

請連絡訂用帳戶擁有者，並要求他們透過[角色型存取控制](https://docs.microsoft.com/azure/role-based-access-control/rbac-and-directory-admin-roles)(RBAC) 提供映像版本或父代資源 (像是共用映像資源庫或映像定義) 的讀取權限。 

找不到映像版本。

確認您嘗試在其中建立 VM 或虛擬機器擴展集的區域包含在映像版本的目標區域清單中。 如果此區域已經在目標區域清單中，則確認複寫作業是否已完成。 您可使用 **-ReplicationStatus** 旗標，檢查是否已完成複寫到所有指定的目標區域。 

VM 或虛擬機器擴展集建立花費很長的時間。

確認您嘗試從中建立 VM 或虛擬機器擴展集的映像版本 **OSType**，與您用來建立映像版本的來源受控映像具有相同的 **OSType**。 

## <a name="unable-to-share-resources"></a>無法共用資源

使用[角色型存取控制](https://docs.microsoft.com/azure/role-based-access-control/rbac-and-directory-admin-roles) (RBAC)，可在訂用帳戶間啟用共用映像資源庫、映像定義及映像版本資源的共用。 

## <a name="replication-is-slow"></a>複寫速度很慢

使用 **--expand ReplicationStatus** 旗標，以檢查是否已完成複寫到所有指定的目標區域。 如果尚未完成，最多等待 6 小時讓作業完成。 如果失敗，再次觸發命令來建立和複寫映像版本。 如果映像版本要複寫至大量目標區域，請考慮分階段進行複寫。

## <a name="azure-limits-and-quotas"></a>Azure 限制和配額 

[Azure 限制和配額](https://docs.microsoft.com/azure/azure-subscription-service-limits)適用於所有的共用映像資源庫、映像定義和映像版本資源。 確定您在您訂用帳戶的限制範圍內。 



