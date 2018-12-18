---
title: Azure Service Fabric 環境變數 | Microsoft Docs
description: Service Fabric 環境變數的參考文件
documentationcenter: .net
author: mikkelhegn
manager: msfussell
editor: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: reference
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 12/07/2017
ms.author: mikhegn
ms.openlocfilehash: f7c36fec7ff58c225e41899e8264ca1dde95ce7c
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34213243"
---
# <a name="service-fabric-environment-variables"></a>Service Fabric 環境變數

Service Fabric 具有針對每個服務執行個體所設定的內建環境變數。 環境變數的完整清單如下：

| 環境變數                         | 說明                                                            | 範例                                                              |
|----------------------------------------------|------------------------------------------------------------------------|----------------------------------------------------------------------|
| Fabric_ApplicationName                       | 應用程式的網狀架構 URI 名稱                                 | fabric:/MyApplication                                                |
| Fabric_CodePackageName                       | 處理序所屬的程式碼套件名稱              | 代碼                                                                 |
| Fabric_Endpoint\_IPOrFQDN\_*ServiceEndpointName*     | IP 位址或端點的 FQDN                                 | 10.0.0.1                                                     |
| Fabric\_Endpoint\_*ServiceEndpointName*              | 端點的連接埠號碼                                  | 8234                                                                 |
| Fabric_Folder_App_Log                        | 記錄資料夾                                                             | C:\\\\Data\\\\_App\\\\_Node_0\\\\MyApplicationType_App12\\\\log      |
| Fabric_Folder_App_Temp                       | 暫存資料夾                                                            | C:\\\\Data\\\\_App\\\\_Node_0\\\\MyApplicationType_App12\\\\temp     |
| Fabric_Folder_App_Work                       | 工作資料夾                                                            | C:\\\\Data\\\\_App\\\\_Node_0\\\\MyApplicationType_App12\\\\work     |
| Fabric_Folder_Application                    | 應用程式的主資料夾                                           | C:\\\\Data\\\\_App\\\\_Node_0\\\\MyApplicationType_App12             |
| Fabric_IsContainerHost                       | 指定處理序是否為容器的 bool                   | false                                                                |
| Fabric_NodeId                                | 節點識別碼 (執行處理序的節點)                            | bf865279ba277deb864a976fbf4c200e                                     |
| Fabric_NodeIPOrFQDN                          | IP 或節點的 FQDN，如叢集資訊清單檔所指定。 | localhost 或 10.0.0.1                                                |
| Fabric_NodeName                              | 節點名稱 (執行處理序的節點)                          | _Node_0                                                              |
| Fabric_ServiceName                           | 服務名稱 (如果服務裝載於 ExclusiveProcess 模式中)。 只有在您使用 ServicePackageActivationMode ExclusiveProcess 建立服務時，才能使用此變數值。  | MyService                                               |
| Fabric_ServicePackageActivationId            | ServicePackageActivationId                                         | GUID                                                               |
| Fabric_ServicePackageName                    | 處理序所屬的服務套件名稱                     | Web1Pkg                                                              |

Service Fabric 執行階段所使用的內部環境變數：

- Fabric_ApplicationHostId
- Fabric_ApplicationHostType
- Fabric_ApplicationId
- Fabric_CodePackageInstanceId
- Fabric_CodePackageInstanceSeqNum
- Fabric_RuntimeConnectionAddress
- Fabric_ServicePackageActivationGuid
- Fabric_ServicePackageInstanceId
- Fabric_ServicePackageInstanceSeqNum
- Fabric_ServicePackageVersionInstance
- FabricActivatorAddress
- FabricPackageFileName
- HostedServiceName