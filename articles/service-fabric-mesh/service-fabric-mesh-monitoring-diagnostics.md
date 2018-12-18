---
title: Azure Service Fabric Mesh 應用程式的監視和診斷 | Microsoft Docs
description: 了解如何在 Azure 上監視和診斷 Service Fabric Mesh 應用程式。
services: service-fabric-mesh
documentationcenter: .net
author: srrengar
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/22/2018
ms.author: srrengar
ms.custom: mvc, devcenter
ms.openlocfilehash: 9f857c23b5500bc7790a0ff7fcf81eaec51f37c9
ms.sourcegitcommit: 99a6a439886568c7ff65b9f73245d96a80a26d68
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39358263"
---
# <a name="monitoring-and-diagnostics"></a>監視和診斷
Azure Service Fabric Mesh 是一個受到完整管理的服務，讓開發人員能夠部署微服務應用程式，而不需管理虛擬機器、儲存體或網路功能。 Service Fabric Mesh 的監視和診斷可歸類為三種主要診斷資料類型：

- 應用程式記錄 - 根據您檢測應用程式的方式，這些會定義為來自您的容器化應用程式的記錄 (例如 Docker 記錄)
- 平台事件 - 來自 Mesh 平台並與您的容器作業相關的事件，目前包括容器啟用、停用及終止。
- 容器計量 - 您的容器適用的資源使用率和效能計量 (Docker 統計資料)

本文討論最新預覽版本可用的監視和診斷選項。

## <a name="application-logs"></a>應用程式記錄檔

您可以從您已部署的容器，依照容器檢視 Docker 記錄。 在 Service Fabric Mesh 應用程式模型中，每個容器都是您應用程式中的程式碼封裝。 若要查看相關聯的記錄與程式碼封裝，請使用下列命令：

```cli
az mesh code-package-log get --resource-group <nameOfRG> --app-name <nameOfApp> --service-name <nameOfService> --replica-name <nameOfReplica> --code-package-name <nameOfCodePackage>
```

> [!NOTE]
> 您可以使用 "az mesh service-replica" 命令來取得複本名稱。 複本名稱是從 0 遞增的數字。*

從投票應用程式的 VotingWeb.Code 容器查看記錄時，其外觀如下所示：

```cli
az mesh code-package-log get --resource-group <nameOfRG> --application-name SbzVoting --service-name VotingWeb --replica-name 0 --code-package-name VotingWeb.Code
```

## <a name="next-steps"></a>後續步驟
若要深入了解 Service Fabric Mesh，請閱讀下列概觀：
- [Service Fabric Mesh 概觀](service-fabric-mesh-overview.md)
