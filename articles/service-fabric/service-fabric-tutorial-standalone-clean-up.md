---
title: 教學課程：清除 Service Fabric 獨立叢集 - Azure Service Fabric | Microsoft Docs
description: 在本教學課程中，您將了解如何清除獨立叢集
services: service-fabric
documentationcenter: .net
author: david-stanford
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/11/2018
ms.author: dastanfo
ms.custom: mvc
ms.openlocfilehash: d92712200920f135e362e15e1d9536dc2342ebaa
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34211084"
---
# <a name="tutorial-clean-up-your-standalone-cluster"></a>本教學課程：清除獨立叢集

Service Fabric 獨立叢集讓您能夠選擇自己的環境，並且在 Service Fabric 所採用的「任何 OS、任何雲端」方法中建立叢集。 在此教學課程系列中，您會建立裝載於 AWS 的獨立叢集，並在其中安裝應用程式。

本教學課程是一個系列的第四部分。 此部分的教學課程將說明如何清除您先前建立用以裝載 Service Fabric 叢集的 AWS 資源。

在本系列的第一部分中，您了解如何：

> [!div class="checklist"]
> * 清除 Service Fabric 叢集
> * 清除 AWS 資源

## <a name="clean-up-service-fabric-cluster"></a>清除 Service Fabric 叢集

* 透過 RDP 進入您對已安裝的 Service Fabric 使用的 EC2 執行個體
* 開啟 PowerShell
* 將目錄切換至第二個教學課程中的解壓縮資料夾。
* 執行下列命令以移除 Service Fabric 叢集：

```powershell
.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json
```

* 出現 `Y` 提示時，如果作業已順利完成，輸出將會如下所示，並且替換為您自己的 IP 位址：

```powershell
Best Practices Analyzer completed successfully.
Removing configuration from machine 172.31.21.141
Removing configuration from machine 172.31.27.1
Removing configuration from machine 172.31.20.163
Configuration removed from machine 172.31.21.141
Configuration removed from machine 172.31.27.1
Configuration removed from machine 172.31.20.163
The cluster is successfully removed.
```

## <a name="clean-up-aws-resources"></a>清除 AWS 資源

* 登入您的 AWS 帳戶
* 移至 EC2 主控台。
* 選取您在教學課程的第一個單元中建立的三個節點。
* 按一下 [動作] > [執行個體狀態] > [終止]

## <a name="next-steps"></a>後續步驟

在此系列的第四個單元中，您已了解如何清除您在先前的步驟中建立的資源。

> [!div class="checklist"]
> * 清除資源

> [!div class="nextstepaction"]
> [回到開頭處](service-fabric-tutorial-standalone-create-infrastructure.md)