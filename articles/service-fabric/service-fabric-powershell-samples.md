---
title: Azure PowerShell 範例 - Service Fabric | Microsoft Docs
description: Azure PowerShell 範例 - Service Fabric
services: service-fabric
documentationcenter: service-fabric
author: rwike77
manager: timlt
editor: ''
tags: ''
ms.assetid: b48d1137-8c04-46e0-b430-101e07d7e470
ms.service: service-fabric
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: service-fabric
ms.date: 04/09/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 65f6252ecfa4e5850824fc7aa2e6eca2bdb88692
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34205751"
---
# <a name="azure-powershell-samples"></a>Azure PowerShell 範例

下表包含 PowerShell 指令碼範例的連結，這個範例建立和管理 Service Fabric 叢集、應用程式和服務。

[!INCLUDE [links to azure cli and service fabric cli](../../includes/service-fabric-powershell.md)]

| | |
|-|-|
| **建立叢集** ||
| [建立叢集 (Azure)](./scripts/service-fabric-powershell-create-secure-cluster-cert.md)| 建立 Azure Service Fabric 叢集。 |
|[建立測試叢集 (Azure)](./scripts/service-fabric-powershell-create-test-cluster.md)| 在 Azure 上建立三節點測試 Service Fabric 叢集。|
| **管理叢集、節點和基礎結構** ||
| [新增應用程式憑證](./scripts/service-fabric-powershell-add-application-certificate.md)| 將應用程式的 X.509 憑證加入到叢集中的所有節點。 |
| [更新叢集 VM 上的 RDP 連接埠範圍](./scripts/service-fabric-powershell-change-rdp-port-range.md)|對已部署之叢集中的叢集節點 VM 變更 RDP 連接埠範圍。|
| [更新叢集節點 VM 的管理員使用者和密碼](./scripts/service-fabric-powershell-change-rdp-user-and-pw.md) | 更新叢集節點 VM 的管理員使用者名稱和密碼。 |
| [在負載平衡器中開啟連接埠](./scripts/service-fabric-powershell-open-port-in-load-balancer.md) | 在 Azure 負載平衡器中開啟應用程式連接埠，以在特定連接埠上允許輸入流量。 |
| [建立輸入網路安全性群組規則](./scripts/service-fabric-powershell-add-nsg-rule.md) | 建立輸入網路安全性群組規則，以在特定連接埠上允許叢集的輸入流量。 |
| **管理應用程式** ||
| [部署應用程式](./scripts/service-fabric-powershell-deploy-application.md)| 將應用程式部署到叢集。|
| [升級應用程式](./scripts/service-fabric-powershell-upgrade-application.md)| 升級應用程式。|
| [移除應用程式](./scripts/service-fabric-powershell-remove-application.md)| 從叢集移除應用程式。|
