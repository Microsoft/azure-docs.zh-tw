---
title: 設定 Service Fabric 應用程式的升級 | Microsoft Docs
description: 了解如何使用 Microsoft Visual Studio 來設定升級 Service Fabric 應用程式的設定。
services: service-fabric
documentationcenter: na
author: mikkelhegn
manager: mfussell
editor: tglee
ms.assetid: 1757ba85-0b7b-4f16-8a23-2ddaa61c86c6
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 06/29/2017
ms.author: mikkelhegn
ms.openlocfilehash: 79120371ca2a62e5ef9f2bf38476635db12e9fcc
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037968"
---
# <a name="configure-the-upgrade-of-a-service-fabric-application-in-visual-studio"></a>在 Visual Studio 中設定 Service Fabric 應用程式的升級
Azure Service Fabric 的 Visual Studio 工具提供發佈至本機或遠端叢集的升級支援。 在進行測試和偵錯時，有三種情況您會想要將應用程式升級成較新的版本，而不是取代應用程式：

* 應用程式資料不會在升級期間遺失。
* 可用性仍然很高，因此，如果有足夠的服務執行個體分散到升級網域，則在升級期間不會有任何服務中斷。
* 在應用程式進行升級時，可以對該應用程式進行測試。

## <a name="parameters-needed-to-upgrade"></a>升級所需的參數
您可以選擇的部署類型有兩種：一般或升級。 一般部署會將叢集上所有先前的部署資訊和資料都清除，而升級部署則會將其保留。 當您在 Visual Studio 中升級 Service Fabric 應用程式時，您需要提供應用程式升級參數和健康情況檢查原則。 應用程式升級參數可協助控制升級，而健康狀態檢查原則則可判斷升級是否成功。 如需詳細資訊，請參閱 [Service Fabric 應用程式升級：升級參數](service-fabric-application-upgrade-parameters.md) 。

有三種升級模式：Monitored、UnmonitoredAuto 及 UnmonitoredManual。

* Monitored 升級會自動進行升級和應用程式健康狀態檢查。
* UnmonitoredAuto 升級會自動進行升級，但會略過應用程式健康狀態檢查。
* 執行 UnmonitoredManual 升級時，您必須手動升級每個升級網域。

每一種升級模式都需要一組不同的參數。 若要深入了解可用的升級選項，請參閱 [應用程式升級參數](service-fabric-application-upgrade-parameters.md) 。

## <a name="upgrade-a-service-fabric-application-in-visual-studio"></a>在 Visual Studio 中升級 Service Fabric 應用程式
如果您要使用 Visual Studio Service Fabric 工具升級 Service Fabric 應用程式，則您可以核取 [升級應用程式]  核取方塊，將發佈程序指定為升級而非一般部署。

### <a name="to-configure-the-upgrade-parameters"></a>設定升級參數
1. 按一下核取方塊旁邊的 [設定]  按鈕。 將會顯示 [編輯升級參數]  對話方塊。 [編輯升級參數]  對話方塊支援 Monitored、UnmonitoredAuto 及 UnmonitoredManual 升級模式。
2. 選取您想要使用的升級模式，然後填寫參數方格。

    每個參數都有預設值。 選用參數 *DefaultServiceTypeHealthPolicy* 會接受雜湊表輸入。 以下是 *DefaultServiceTypeHealthPolicy*的雜湊表輸入格式範例：

    ```
    @{ ConsiderWarningAsError = "false"; MaxPercentUnhealthyDeployedApplications = 0; MaxPercentUnhealthyServices = 0; MaxPercentUnhealthyPartitionsPerService = 0; MaxPercentUnhealthyReplicasPerPartition = 0 }
    ```

    *ServiceTypeHealthPolicyMap* 是另一個會接受雜湊表輸入 (格式如下) 的選擇性參數：

    ```    
    @ {"ServiceTypeName" : "MaxPercentUnhealthyPartitionsPerService,MaxPercentUnhealthyReplicasPerPartition,MaxPercentUnhealthyServices"}
    ```

    以下是一個真實範例：

    ```
    @{ "ServiceTypeName01" = "5,10,5"; "ServiceTypeName02" = "5,5,5" }
    ```
3. 如果您選取 UnmonitoredManual 升級模式，您必須手動啟動 PowerShell 主控台，才能繼續並完成升級程序。 若要了解手動升級如何運作，請參閱 [Service Fabric 應用程式升級：進階主題](service-fabric-application-upgrade-advanced.md) 。

## <a name="upgrade-an-application-by-using-powershell"></a>使用 PowerShell 升級應用程式
您可以使用 PowerShell Cmdlet 來升級 Service Fabric 應用程式。 如需詳細資訊，請參閱 [Service Fabric 應用程式升級教學課程](service-fabric-application-upgrade-tutorial.md)和 [Start-ServiceFabricApplicationUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricapplicationupgrade)。

## <a name="specify-a-health-check-policy-in-the-application-manifest-file"></a>在應用程式資訊清單檔案中指定健康情況狀態檢查原則
Service Fabric 應用程式中的每個服務都可以有自己的健康情況原則參數來覆寫預設值。 您可以在應用程式資訊清單檔案中提供這些參數值。

下列範例示範如何在應用程式清單中套用每個服務的唯一健康情況檢查原則。

```xml
<Policies>
    <HealthPolicy ConsiderWarningAsError="false" MaxPercentUnhealthyDeployedApplications="20">
        <DefaultServiceTypeHealthPolicy MaxPercentUnhealthyServices="20"               
                MaxPercentUnhealthyPartitionsPerService="20"
                MaxPercentUnhealthyReplicasPerPartition="20" />
        <ServiceTypeHealthPolicy ServiceTypeName="ServiceTypeName1"
                MaxPercentUnhealthyServices="20"
                MaxPercentUnhealthyPartitionsPerService="20"
                MaxPercentUnhealthyReplicasPerPartition="20" />      
    </HealthPolicy>
</Policies>
```
## <a name="next-steps"></a>後續步驟
如需有關升級應用程式的詳細資訊，請參閱[使用 Visual Studio 升級應用程式](service-fabric-application-upgrade-tutorial.md)。
