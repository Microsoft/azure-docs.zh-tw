---
title: 針對本機 Azure Service Fabric 叢集設定進行疑難排解 | Microsoft Azure
description: 本文涵蓋了一組疑難排解本機開發叢集的建議
services: service-fabric
documentationcenter: .net
author: mikkelhegn
manager: timlt
editor: ''
ms.assetid: 97f4feaa-bba0-47af-8fdd-07f811fe2202
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/23/2018
ms.author: mikhegn
ms.openlocfilehash: a7f58914fd6e498e717e19bfea11c9e3fcfc0399
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34212012"
---
# <a name="troubleshoot-your-local-development-cluster-setup"></a>疑難排解本機開發叢集設定
如果您在與本機 Azure Service Fabric 開發叢集互動時遇到問題，請檢閱下列建議，以尋求可能的解決方法。

## <a name="cluster-setup-failures"></a>叢集設定失敗
### <a name="cannot-clean-up-service-fabric-logs"></a>無法清除 Service Fabric 記錄檔
#### <a name="problem"></a>問題
執行 DevClusterSetup 指令碼時，看到以下錯誤：

    Cannot clean up C:\SfDevCluster\Log fully as references are likely being held to items in it. Please remove those and run this script again.
    At line:1 char:1 + .\DevClusterSetup.ps1
    + ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo : NotSpecified: (:) [Write-Error], WriteErrorException
    + FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException,DevClusterSetup.ps1


#### <a name="solution"></a>解決方法
關閉目前的 Powershell 視窗，並以系統管理員身分開啟新的 Powershell 視窗。 您現在可以成功地執行指令碼。

## <a name="cluster-connection-failures"></a>叢集連接失敗

### <a name="type-initialization-exception"></a>類型初始化例外狀況
#### <a name="problem"></a>問題
在 PowerShell 中連線到叢集時，看到 System.Fabric.Common.AppTrace 的 TypeInitializationException 錯誤。

#### <a name="solution"></a>解決方法
在安裝期間未正確設定路徑變數。 登出 Windows，再重新登入。 這會重新整理您的路徑。

### <a name="cluster-connection-fails-with-object-is-closed"></a>叢集連接失敗，且出現「物件已關閉」
#### <a name="problem"></a>問題
呼叫 Connect-ServiceFabricCluster 失敗，且出現類似以下錯誤：

    Connect-ServiceFabricCluster : The object is closed.
    At line:1 char:1
    + Connect-ServiceFabricCluster
    + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo : InvalidOperation: (:) [Connect-ServiceFabricCluster], FabricObjectClosedException
    + FullyQualifiedErrorId : CreateClusterConnectionErrorId,Microsoft.ServiceFabric.Powershell.ConnectCluster

#### <a name="solution"></a>解決方法
關閉目前的 Powershell 視窗，並以系統管理員身分開啟新的 Powershell 視窗。

### <a name="fabric-connection-denied-exception"></a>拒絕網狀架構連線例外狀況
#### <a name="problem"></a>問題
從 Visual Studio 進行偵錯時，看見 FabricConnectionDeniedException 錯誤。

#### <a name="solution"></a>解決方法
當您嘗試手動啟動服務主機處理序時，通常會發生這個錯誤。

請確定您的解決方法中沒有任何設定為啟始專案的服務專案。 只有 Service Fabric 應用程式專案才可設為啟始專案。

> [!TIP]
> 如果在安裝之後，您的本機叢集開始行為異常，您可以使用本機叢集管理員系統匣應用程式將其重設。 這麼做會移除現有叢集，並設定新的叢集。 請注意，所有已部署的應用程式和相關聯的資料都會被移除。
> 
> 

## <a name="next-steps"></a>後續步驟
* [透過系統健康情況報告了解及疑難排解您的叢集](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)
* [使用 Service Fabric 總管視覺化叢集](service-fabric-visualizing-your-cluster.md)

