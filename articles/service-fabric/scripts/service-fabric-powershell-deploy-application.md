---
title: Azure PowerShell 指令碼範例 - 將應用程式部署到叢集 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 將應用程式部署到 Service Fabric 叢集。
services: service-fabric
documentationcenter: ''
author: rwike77
manager: timlt
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.devlang: na
ms.topic: sample
ms.date: 01/18/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: c81514fb4b1c1da483ebd55deae149caf22d4b63
ms.sourcegitcommit: 2a70752d0987585d480f374c3e2dba0cd5097880
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2018
ms.locfileid: "27927595"
---
# <a name="deploy-an-application-to-a-service-fabric-cluster"></a>將應用程式部署到 Service Fabric 叢集

這個範例指令碼會將應用程式封裝複製到叢集映像存放區、在叢集中註冊應用程式類型、移除不必要的應用程式套件，並從應用程式類型建立應用程式執行個體。  如果已在目標應用程式類型的應用程式資訊清單中定義任何預設服務，則這些服務也會一併建立。 視需要自訂參數。 

如有需要，可隨同 [Service Fabric SDK](../service-fabric-get-started.md) 一起安裝 Service Fabric PowerShell 模組。 

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/service-fabric/deploy-application/deploy-application.ps1 "Deploy an application to a cluster")]

## <a name="clean-up-deployment"></a>清除部署 

指令碼範例執行之後，[移除應用程式](service-fabric-powershell-remove-application.md)中的指令碼可用來移除應用程式執行個體、取消註冊應用程式類型，並且從映像存放區刪除應用程式封裝。

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
|[Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps)| 建立連接 Service Fabric 叢集的連線。 |
|[Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) | 將應用程式封裝複製到叢集映像存放區。  |
|[Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps)| 在叢集上註冊應用程式類型和版本。 |
|[New-ServiceFabricApplication](/powershell/module/servicefabric/new-servicefabricapplication?view=azureservicefabricps)| 從註冊的應用程式類型建立應用程式。 |
| [Remove-ServiceFabricApplicationPackage](/powershell/module/servicefabric/remove-servicefabricapplicationpackage?view=azureservicefabricps) | 從映像存放區移除 Service Fabric 應用程式封裝。|

## <a name="next-steps"></a>後續步驟

如需有關 Service Fabric PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/service-fabric/?view=azureservicefabricps)。

您可以在 [PowerShell 範例](../service-fabric-powershell-samples.md)中找到適用於 Azure Service Fabric 的其他 Azure PowerShell 範例。
