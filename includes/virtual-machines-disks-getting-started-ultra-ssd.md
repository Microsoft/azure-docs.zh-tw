---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 09/24/2018
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: b28830ac2634ad2238d834238de83c9184bcd6f0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47010386"
---
# <a name="enabling-azure-ultra-ssds"></a>啟用 Azure Ultra SSD

Azure Ultra SSD 可為 Azure IaaS VM 提供高輸送量、高 IOPS 以及一致的低延遲磁碟儲存體。 這個新的供應項目可提供絕佳的效能，同時保有我們現有磁碟供應項目的相同可用性層級。 Ultra SSD 的其他優點包括能夠以動態方式變更磁碟的效能和您的工作負載，而不需要重新啟動虛擬機器。 Ultra SSD 適用於資料密集的工作負載 (例如 SAP HANA)、最上層資料庫，以及高交易量的工作負載。

目前，Ultra SSD 處於預覽狀態，您必須[註冊](https://aka.ms/UltraSSDPreviewSignUp)預覽版才能存取它們。

一旦經過核准之後，請執行下列其中一個命令，以判斷要在美國東部 2 的哪一個區域中部署您的 Ultra SSD：

PowerShell：`Get-AzureRmComputeResourceSku | where {$_.ResourceType -eq "disks" -and $_.Name -eq "UltraSSD_LRS" }`

CLI：`az vm list-skus --resource-type disks --query “[?name==’UltraSSD_LRS’]”`

回應將會類似於下面的形式，其中 X 是要用於在美國東部 2 中部署的區域。 X 可能是 1、2 或 3。

|ResourceType  |名稱  |位置  |區域  |限制  |功能  |值  |
|---------|---------|---------|---------|---------|---------|---------|
|磁碟     |UltraSSD_LRS         |eastus2         |X         |         |         |         |

如果命令沒有回應，表示仍在等待您註冊此功能，或註冊尚未核准。

既然您已經知道要部署至哪一個區域，請依照此文章中的部署步驟，使用 Ultra SSD 磁碟部署您的第一部 VM。

## <a name="deploying-an-ultra-ssd"></a>部署 Ultra SSD

首先，判斷要部署的 VM 大小。 在此預覽版本中，只支援 DsV3 與 EsV3 VM 系列。 如需有關這些 VM 大小的其他詳細資訊，請參閱此[部落格](https://azure.microsoft.com/blog/introducing-the-new-dv3-and-ev3-vm-sizes/) \(英文\) 的第二個表格。
另請參閱[使用多個 Ultra SSD 磁碟建立 VM](https://aka.ms/UltraSSDTemplate)，其中示範如何使用多個 Ultra SSD 磁碟建立 VM。

以下說明新的/修改過的 Resource Manager 範本變更：`Microsoft.Compute/virtualMachines` 與 `Microsoft.Compute/Disks` 的 **apiVersion** 必須設定為 `2018-06-01` (或更新版本)。

指定磁碟 SKU UltraSSD_LRS、磁碟容量、IOPS 與輸送量 (以每秒 MB 數為單位)，來建立 Ultra SSD 磁碟。 下列範例會建立一個磁碟，其具有 1,024 GiB (GiB = 2^30 個位元組)、80,000 IOPS 與 1,200 MBps (每秒 MB 數 = 每秒 10^6 個位元組)：

```json
"properties": {  
    "creationData": {  
    "createOption": "Empty"  
},  
"diskSizeGB": 1024,  
"diskIOPSReadWrite": 80000,  
"diskMBpsReadWrite": 1200,  
}
```

在 VM 屬性上新增額外容量，以指出它的 Ultra SSD 已啟用 (如需完整的 Resource Manager 範本，請參閱[範例](https://aka.ms/UltraSSDTemplate))：

```json
{
    "apiVersion": "2018-06-01",
    "type": "Microsoft.Compute/virtualMachines",
    "properties": {
                    "hardwareProfile": {},
                    "additionalCapabilities" : {
                                    "ultraSSDEnabled" : "true"
                    },
                    "osProfile": {},
                    "storageProfile": {},
                    "networkProfile": {}
    }
}
```

一旦佈建 VM，您就可以將資料磁碟分割和格式化，並針對你的工作負載設定它們。

## <a name="additional-ultra-ssd-scenarios"></a>其他 Ultra SSD 案例

- 在 VM 建立期間，也能夠以隱含方式建立 Ultra SSD。 不過，這些磁碟的 IOPS 和輸送量將會收到預設值 (前者為 500，後者為每秒 8 MiB)。
- 額外的 Ultra SSD 可以連接到與 Ultra SSD 相容的 VM。
- Ultra SSD 支援在工作階段調整磁碟效能屬性 (IOPS 與輸送量)，而不必將磁碟從虛擬機器中斷連結。 一旦向磁碟發出磁碟效能調整作業，最多可能需要一個小時，變更才會實際生效。
- 若要增加磁碟容量，必須解除配置虛擬機器。

# <a name="next-steps"></a>後續步驟

如果您想要嘗試新的磁碟類型，但尚未註冊使用預覽版，[請透過此問卷要求存取權](https://aka.ms/UltraSSDPreviewSignUp)。