---
title: Publish-WebApplicationVM | Microsoft Docs
description: 了解如何將 Web 應用程式部署到虛擬機器。 此指令碼會在您的 Azure 訂用帳戶中建立所需的資源 (如果它們不存在)。
services: visual-studio-online
author: ghogen
manager: douge
assetId: de4cec95-f73f-44d9-babd-9f47f2633cdb
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 11/11/2016
ms.author: ghogen
ms.openlocfilehash: c2dc6057eeb4eba1306309785e13192674bc43c6
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42144821"
---
# <a name="publish-webapplicationvm-windows-powershell-script"></a>Publish-WebApplicationVM (Windows PowerShell 指令碼)
將 Web 應用程式部署到虛擬機器。 指令碼會在您的 Azure 訂用帳戶中建立所需的資源 (如果它們不存在)。

```
Publish-WebApplicationVM
–Configuration <configuration>
-SubscriptionName <subscriptionName>
-WebDeployPackage <packageName>
-VMPassword @{Name = "name"; Password = "password")
-DatabaseServerPassword @{Name = "name"; Password = "password"}
-SendHostMessagesToOutput
-Verbose
```

### <a name="configuration"></a>組態
描述部署詳細資訊的 JSON 組態檔路徑。

| 別名 | None |
| --- | --- |
| 必要？ |true |
| 位置 |已命名 |
| 預設值 |None |
| 接受管線輸入？ |false |
| 接受萬用字元？ |false |

### <a name="subscriptionname"></a>SubscriptionName
要建立虛擬機器的 Azure 訂用帳戶名稱。

| 別名 | None |
| --- | --- |
| 必要？ |false |
| 位置 |已命名 |
| 預設值 |使用訂用帳戶檔案中的第一個訂用帳戶 |
| 接受管線輸入？ |false |
| 接受萬用字元？ |false |

### <a name="webdeploypackage"></a>WebDeployPackage
要發佈至虛擬機器的 Web 部署封裝路徑。 您可以使用 Visual Studio 的 [發佈 Web] 精靈來建立此封裝。 請參閱 [如何：在 Visual Studio 中建立 Web 部署封裝](https://msdn.microsoft.com/library/dd465323.aspx)。

| 別名 | None |
| --- | --- |
| 必要？ |false |
| 位置 |已命名 |
| 預設值 |None |
| 接受管線輸入？ |false |
| 接受萬用字元？ |false |

### <a name="allowuntrusted"></a>AllowUntrusted
如果為 true，允許使用未受信任的根授權單位所簽署的憑證。

| 別名 | None |
| --- | --- |
| 必要？ |false |
| 位置 |已命名 |
| 預設值 |false |
| 接受管線輸入？ |false |
| 接受萬用字元？ |false |

### <a name="vmpassword"></a>VMPassword
虛擬機器帳戶的認證。 範例：-VMPassword @{Name = "admin"; Password = "password"}

| 別名 | None |
| --- | --- |
| 必要？ |false |
| 位置 |已命名 |
| 預設值 |None |
| 接受管線輸入？ |false |
| 接受萬用字元？ |false |

### <a name="databaseserverpassword"></a>DatabaseServerPassword
Azure 中的 SQL 資料庫認證。 範例：-DatabaseServerPassword @{Name = "admin"; Password = "password"}

| 別名 | None |
| --- | --- |
| 必要？ |false |
| 位置 |已命名 |
| 預設值 |None |
| 接受管線輸入？ |false |
| 接受萬用字元？ |false |

### <a name="sendhostmessagestooutput"></a>SendHostMessagesToOutput
如果為 true，將訊息從指令碼列印至輸出資料流。

| 別名 | None |
| --- | --- |
| 必要？ |false |
| 位置 |已命名 |
| 預設值 |false |
| 接受管線輸入？ |false |
| 接受萬用字元？ |false |

## <a name="remarks"></a>備註
如需如何使用指令碼來建立開發和測試環境的完整說明，請參閱 [使用 Windows PowerShell 指令碼來發佈至開發和測試環境](vs-azure-tools-publishing-using-powershell-scripts.md)。

JSON 組態檔會指定待部署項目的詳細資料。 它包含您在建立專案時所指定的資訊，例如名稱、同質群組、VHD 映像和虛擬機器的大小。 它也包含虛擬機器上的端點、要佈建的資料庫 (如有)，以及 Web 部署參數。 下列程式碼片段將顯示一個 JSON 組態檔範例：

```
{
    "environmentSettings": {
        "cloudService": {
            "name": "myvmname",
            "affinityGroup": "",
            "location": "West US",
            "virtualNetwork": "",
            "subnet": "",
            "availabilitySet": "",
            "virtualMachine": {
                "name": "myvmname",
                "vhdImage": "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-201404.01-en.us-127GB.vhd",
                "size": "Small",
                "user": "vmuser1",
                "password": "",
                "enableWebDeployExtension": true,
                "endpoints": [
                    {
                        "name": "Http",
                        "protocol": "TCP",
                        "publicPort": "80",
                        "privatePort": "80"
                    },
                    {
                        "name": "Https",
                        "protocol": "TCP",
                        "publicPort": "443",
                        "privatePort": "443"
                    },
                    {
                        "name": "WebDeploy",
                        "protocol": "TCP",
                        "publicPort": "8172",
                        "privatePort": "8172"
                    },
                    {
                        "name": "Remote Desktop",
                        "protocol": "TCP",
                        "publicPort": "3389",
                        "privatePort": "3389"
                    },
                    {
                        "name": "Powershell",
                        "protocol": "TCP",
                        "publicPort": "5986",
                        "privatePort": "5986"
                    }
                ]
            }
        },
        "databases": [
            {
                "connectionStringName": "",
                "databaseName": "",
                "serverName": "",
                "user": "",
                "password": ""
            }
        ],
        "webDeployParameters": {
            "iisWebApplicationName": "Default Web Site"
        }
    }
}
```

您可以編輯 JSON 組態檔來變更佈建項目。 虛擬機器和雲端服務是必要的，但資料庫區段是選擇性的。

