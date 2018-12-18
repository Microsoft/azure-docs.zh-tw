---
title: 擲回的常見 FabricClient 例外狀況 | Microsoft Docs
description: 描述 FabricClient API 可在執行應用程式和叢集管理作業時擲回的常見例外狀況和錯誤。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: bb821313-b221-479f-b08e-36cf07e60a07
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/20/2018
ms.author: ryanwi
ms.openlocfilehash: e854ed42b6af8bc090950e8399e3229e202a2ed0
ms.sourcegitcommit: 1438b7549c2d9bc2ace6a0a3e460ad4206bad423
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36293407"
---
# <a name="common-exceptions-and-errors-when-working-with-the-fabricclient-apis"></a>使用 FabricClient API 時常見的例外狀況和錯誤
[FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient#System_Fabric_FabricClient) API 可讓叢集和應用程式系統管理員對 Service Fabric 應用程式、服務或叢集執行系統管理工作。 例如，應用程式部署、升級和移除、檢查叢集的健康狀態，或測試服務。 應用程式開發人員和叢集系統管理員可以使用 FabricClient API，來開發用於管理 Service Fabric 叢集和應用程式的工具。

使用 FabricClient 可以執行許多不同類型的作業。  每種方法都可能因輸入不正確、執行階段錯誤或暫時性基礎結構問題而擲回錯誤的例外狀況。  請參閱 API 參考文件來尋找特定方法所擲回的例外狀況。 不過，許多不同的 [FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient#System_Fabric_FabricClient) API 可能會擲回一些例外狀況。 下表列出常見的 FabricClient API 例外狀況。

| 例外狀況 | 擲回時機 |
| --- |:--- |
| [System.Fabric.FabricObjectClosedException](https://docs.microsoft.com/dotnet/api/system.fabric.fabricobjectclosedexception#System_Fabric_FabricObjectClosedException) |[FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient#System_Fabric_FabricClient) 物件處於已關閉狀態。 處置正在使用的 [FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient#System_Fabric_FabricClient) 物件，並具現化新的 [FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient#System_Fabric_FabricClient) 物件。 |
| [System.TimeoutException](https://docs.microsoft.com/dotnet/core/api/system.timeoutexception#System_TimeoutException) |作業逾時。作業需要 MaxOperationTimeout 以上的時間才能完成時，會傳回 [OperationTimedOut](https://docs.microsoft.com/dotnet/api/system.fabric.fabricerrorcode#System_Fabric_FabricErrorCode)。 |
| [System.UnauthorizedAccessException](https://docs.microsoft.com/dotnet/core/api/system.unauthorizedaccessexception#System_UnauthorizedAccessException) |作業的存取檢查失敗。 傳回 E_ACCESSDENIED。 |
| [System.Fabric.FabricException](https://docs.microsoft.com/dotnet/api/system.fabric.fabricexception#System_Fabric_FabricException) |執行作業時發生執行階段錯誤。 任何 FabricClient 方法都可能擲回 [FabricException](https://docs.microsoft.com/dotnet/api/system.fabric.fabricexception#System_Fabric_FabricException)，[ErrorCode](https://docs.microsoft.com/dotnet/api/system.fabric.fabricexception#System_Fabric_FabricException_ErrorCode) 屬性指出例外狀況的確切原因。 錯誤碼是在 [FabricErrorCode](https://docs.microsoft.com/dotnet/api/system.fabric.fabricerrorcode#System_Fabric_FabricErrorCode) 列舉中定義。 |
| [System.Fabric.FabricTransientException](https://docs.microsoft.com/dotnet/api/system.fabric.fabrictransientexception#System_Fabric_FabricTransientException) |作業因某種暫時性錯誤狀況而失敗。 例如，作業可能因暫時無法到達複本的仲裁而失敗。 暫時性例外狀況會對應至可重試的失敗作業。 |

[FabricException](https://docs.microsoft.com/dotnet/api/system.fabric.fabricexception#System_Fabric_FabricException) 中傳回的一些常見 [FabricErrorCode](https://docs.microsoft.com/dotnet/api/system.fabric.fabricerrorcode#System_Fabric_FabricErrorCode) 錯誤：

| Error | 條件 |
| --- |:--- |
| CommunicationError |通訊錯誤導致作業失敗，請重試作業。 |
| InvalidCredentialType |認證類型無效。 |
| InvalidX509FindType |X509FindType 無效。 |
| InvalidX509StoreLocation |X509 存放區位置無效。 |
| InvalidX509StoreName |X509 存放區名稱無效。 |
| InvalidX509Thumbprint |X509 憑證指紋字串無效。 |
| InvalidProtectionLevel |保護層級無效。 |
| InvalidX509Store |無法開啟 X509 憑證存放區。 |
| InvalidSubjectName |主體名稱無效。 |
| InvalidAllowedCommonNameList |一般名稱清單字串的格式無效。 它應該是以逗號分隔的清單。 |

