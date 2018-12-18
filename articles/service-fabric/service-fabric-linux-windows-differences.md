---
title: Linux 與 Windows 之間的 Azure Service Fabric 差異 | Microsoft Docs
description: Linux 上的 Azure Service Fabric 與 Windows 上的 Azure Service Fabric 之間的差異。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: d552c8cd-67d1-45e8-91dc-871853f44fc6
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: cb4f738fd63b8b013f674557a0c5f5639c26391e
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43126879"
---
# <a name="differences-between-service-fabric-on-linux-and-windows"></a>Linux 與 Windows 上的 Service Fabric 差異

Windows 上支援的一些功能，在 Linux 上尚未提供支援。 最終，功能集會處於同等地位，而隨著每次發行，此功能差距將會縮小。 可用的最新版本之間存在下列差異。

* Envoy (Reverse Proxy) 在 Linux 上處於預覽狀態
* 適用於 Linux 的獨立安裝程式還無法在 Linux 上使用
* 主控台重新導向 (Linux 或 Windows 生產叢集中不支援)
* Linux 上的錯誤分析服務 (FAS)
* Service Fabric 服務的 DNS 服務 (Linux 上的容器支援 DNS 服務)
* 特定 Powershell 命令的 CLI 命令對等項目 (在下面的清單中，大多數項目只適用於獨立叢集)

## <a name="powershell-cmdlets-that-do-not-work-against-a-linux-service-fabric-cluster"></a>無法針對 Linux Service Fabric 叢集運作的 Powershell Cmdlet

* Invoke-ServiceFabricChaosTestScenario
* Invoke-ServiceFabricFailoverTestScenario
* Invoke-ServiceFabricPartitionDataLoss
* Invoke-ServiceFabricPartitionQuorumLoss
* Restart-ServiceFabricPartition
* Start-ServiceFabricNode
* Stop-ServiceFabricNode
* Get-ServiceFabricImageStoreContent
* Get-ServiceFabricChaosReport
* Get-ServiceFabricNodeTransitionProgress
* Get-ServiceFabricPartitionDataLossProgress
* Get-ServiceFabricPartitionQuorumLossProgress
* Get-ServiceFabricPartitionRestartProgress
* Get-ServiceFabricTestCommandStatusList
* Remove-ServiceFabricTestState
* Start-ServiceFabricChaos
* Start-ServiceFabricNodeTransition
* Start-ServiceFabricPartitionDataLoss
* Start-ServiceFabricPartitionQuorumLoss
* Start-ServiceFabricPartitionRestart
* Stop-ServiceFabricChaos
* Stop-ServiceFabricTestCommand
* Get-ServiceFabricNodeConfiguration
* Get-ServiceFabricClusterConfiguration
* Get-ServiceFabricClusterConfigurationUpgradeStatus
* Get-ServiceFabricPackageDebugParameters
* New-ServiceFabricPackageDebugParameter
* New-ServiceFabricPackageSharingPolicy
* Add-ServiceFabricNode
* Copy-ServiceFabricClusterPackage
* Get-ServiceFabricRuntimeSupportedVersion
* Get-ServiceFabricRuntimeUpgradeVersion
* New-ServiceFabricCluster
* New-ServiceFabricNodeConfiguration
* Remove-ServiceFabricCluster
* Remove-ServiceFabricClusterPackage
* Remove-ServiceFabricNodeConfiguration
* Test-ServiceFabricClusterManifest
* Test-ServiceFabricConfiguration
* Update-ServiceFabricNodeConfiguration
* Approve-ServiceFabricRepairTask
* Complete-ServiceFabricRepairTask
* Get-ServiceFabricRepairTask
* Invoke-ServiceFabricDecryptText
* Invoke-ServiceFabricEncryptSecret
* Invoke-ServiceFabricEncryptText
* Invoke-ServiceFabricInfrastructureCommand
* Invoke-ServiceFabricInfrastructureQuery
* Remove-ServiceFabricRepairTask
* Start-ServiceFabricRepairTask
* Stop-ServiceFabricRepairTask
* Update-ServiceFabricRepairTaskHealthPolicy



## <a name="next-steps"></a>後續步驟
* [在 Linux 上準備您的開發環境](service-fabric-get-started-linux.md)
* [在 OSX 上準備您的開發環境](service-fabric-get-started-mac.md)
* [使用 Yeoman 在 Linux 上建立和部署第一個 Service Fabric Java 應用程式](service-fabric-create-your-first-linux-application-with-java.md)
* [在 Linux 上使用適用於 Eclipse 的 Service Fabric 外掛程式建立和部署第一個 Service Fabric Java 應用程式](service-fabric-get-started-eclipse.md)
* [在 Linux 上建立第一個 CSharp 應用程式](service-fabric-create-your-first-linux-application-with-csharp.md)
* [使用 Service Fabric CLI 管理應用程式](service-fabric-application-lifecycle-sfctl.md)
