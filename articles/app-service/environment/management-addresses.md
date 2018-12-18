---
title: Azure App Service Environment 管理位址
description: 列出用於命令 App Service Environment 的管理位址
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: a7738a24-89ef-43d3-bff1-77f43d5a3952
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/29/2018
ms.author: ccompy
ms.openlocfilehash: 590679daff20f9c469fb8fcfcc0fbbad77f91b5b
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162845"
---
# <a name="app-service-environment-management-addresses"></a>App Service Environment 管理位址

App Service Environment (ASE) 是 Azure App Service 到 Azure 虛擬網路 (VNet) 中之子網路的部署。  必須能夠從 Azure App Service 使用的管理介面來存取 ASE。  此 ASE 管理流量會周遊使用者控制的網路。 如果封鎖或誤傳此流量，ASE 會暫止。 如需 ASE 網路服務相依性的詳細資訊，請閱讀[網路考量和 App Service Environment][networking]。 如需 ASE 的一般資訊，您可以從 [App Service Environment 簡介][intro]開始。

本文件針對流向 ASE 的管理流量列出 App Service 來源位址，並提供兩個重要用途。  

1. 您可以使用這些位址來建立網路安全性群組，以鎖定連入流量。  
2. 您可以使用這些位址建立路由，以支援強制通道部署。 如需深入了解如何設定 ASE，以便在由內部部署軟體傳送輸出流量的環境中運作，請參閱[為您的 ASE 設定強制通道][forcedtunnel]

所有 ASE 都有可讓管理流量進入的公用 VIP。 這些位址的連入管理流量會進入 ASE 公用 VIP 上的連接埠 454 和 455。  

## <a name="list-of-management-addresses"></a>管理位址的清單 ##

| 區域 | 位址 |
|--------|-----------|
| 所有公用區域 | 70.37.57.58, 157.55.208.185, 52.174.22.21, 13.94.149.179, 13.94.143.126, 13.94.141.115, 52.178.195.197, 52.178.190.65, 52.178.184.149, 52.178.177.147, 13.75.127.117, 40.83.125.161, 40.83.121.56, 40.83.120.64, 52.187.56.50, 52.187.63.37, 52.187.59.251, 52.187.63.19, 52.165.158.140, 52.165.152.214, 52.165.154.193, 52.165.153.122, 104.44.129.255, 104.44.134.255, 104.44.129.243, 104.44.129.141, 23.102.188.65, 191.236.154.88, 13.64.115.203, 65.52.193.203, 70.37.89.222, 52.224.105.172, 23.102.135.246 |
| Microsoft Azure Government (Fairfax 或 MAG) | 23.97.29.209, 13.72.53.37, 13.72.180.105, 23.97.0.17, 23.97.16.184 |

## <a name="get-your-management-addresses-from-api"></a>從 API 取得管理位址 ##

您可以使用下列 API 呼叫來列出符合您 ASE 的管理位址。

    get /subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Web/hostingEnvironments/<ASE Name>/inboundnetworkdependenciesendpoints?api-version=2016-09-01

此 API 會傳回一份 JSON 文件，其中包含適用於 ASE 的所有連入位址。 位址清單會包含管理位址、您 ASE 所使用的 VIP，以及 ASE 子網路位址範圍本身。  

若要使用 [armclient](http://github.com/projectkudu/ARMClient) 呼叫 API，請使用下列命令，但必須以您的值取代訂用帳戶識別碼、資源群組與 ASE 名稱。  

    armclient login
    armclient get /subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Web/hostingEnvironments/<ASE Name>/inboundnetworkdependenciesendpoints?api-version=2016-09-01


<!-- LINKS -->
[networking]: ./network-info.md
[intro]: ./intro.md
[forcedtunnel]: ./forced-tunnel-support.md
