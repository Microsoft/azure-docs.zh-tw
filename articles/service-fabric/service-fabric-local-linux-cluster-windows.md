---
title: 在 Windows 上設定 Azure Service Fabric Linux 叢集 | Microsoft Docs
description: 本文說明如何設定在 Windows 開發電腦上執行的 Service Fabric Linux 叢集。 這特別適用於跨平台開發。
services: service-fabric
documentationcenter: .net
author: suhuruli
manager: mfussell
editor: ''
ms.assetid: bf84458f-4b87-4de1-9844-19909e368deb
ms.service: service-fabric
ms.devlang: java
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/20/2017
ms.author: suhuruli
ms.openlocfilehash: ad0d383888c173ece5a7fbd3b0de690ed13074f7
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34206288"
---
# <a name="set-up-a-linux-service-fabric-cluster-on-your-windows-developer-machine"></a>在 Windows 開發人員電腦上設定 Linux Service Fabric 叢集

本文件涵蓋如何在 Windows 開發電腦上設定本機 Linux Service Fabric。 設定本機 Linux 叢集，對快速測試以 Linux 叢集為目標但在 Windows 電腦上開發的應用程式而言，會很有用。

## <a name="prerequisites"></a>先決條件
以 Linux 為基礎的 Service Fabric 叢集無法在 Windows 上原生執行。 為執行本機 Service Fabric 叢集，我們提供預先設定的 Docker 容器映像。 開始之前，您需要：

* 至少 4 GB 的 RAM
* 最新版 [Docker](https://store.docker.com/editions/community/docker-ce-desktop-windows)

>[!TIP]
> * 您可以遵循官方 Docker [文件](https://store.docker.com/editions/community/docker-ce-desktop-windows/plans/docker-ce-desktop-windows-tier?tab=instructions)中所述的步驟，在 Windows 上安裝 Docker。 
> * 一旦您完成安裝後，如果遵循[這裡](https://docs.docker.com/docker-for-windows/#check-versions-of-docker-engine-compose-and-machine)所述的步驟正確安裝，就進行驗證


## <a name="create-a-local-container-and-setup-service-fabric"></a>建立本機容器並設定 Service Fabric
若要設定本機的 Docker 容器，並在容器上執行 Service Fabric 叢集，請在 PowerShell 中執行下列步驟：

1. 從 Docker 中樞存放庫中提取映像：

    ```powershell
    docker pull microsoft/service-fabric-onebox
    ```

2. 使用下列內容更新您主機上的 Docker 精靈設定，然後重新啟動 Docker 精靈： 

    ```json
    {
      "ipv6": true,
      "fixed-cidr-v6": "2001:db8:1::/64"
    }
    ```
    建議的更新方法是：移至 [Docker] 圖示 > [設定] > [精靈] > [進階]，並在該處進行更新。 接下來，重新啟動 Docker 精靈以讓變更生效。 

3. 使用映像啟動 Service Fabric One-box 容器執行個體：

    ```powershell
    docker run -itd -p 19080:19080 --name sfonebox microsoft/service-fabric-onebox
    ```
    >[!TIP]
    > * 您可以藉由指定名稱給容器執行個體，以更易於閱讀的方式加以處理。 
    > * 如果您的應用程式正在特定連接埠上接聽，就必須使用額外的 -p 標籤來指定它。 例如，如果您的應用程式正在連接埠 8080 上接聽，請執行 docker run -itd -p 19080:19080 -p 8080:8080 --name sfonebox microsoft/service-fabric-onebox

4. 在互動式 SSH 模式中登入 Docker 容器：

    ```powershell
    docker exec -it sfonebox bash
    ```

5. 執行可擷取必要相依性的設定指令碼，然後在容器上啟動叢集。

    ```bash
    ./setup.sh     # Fetches and installs the dependencies required for Service Fabric to run
    ./run.sh       # Starts the local cluster
    ```

6. 順利完成步驟 5 之後，您可以從 Windows 移至 ``http://localhost:19080``，就能看到 Service Fabric 總管。 此時，您可以使用 Windows 開發人員電腦上的任何工具連線到這個叢集，並部署以 Linux Service Fabric 叢集為目標的應用程式。 

    > [!NOTE]
    > Windows 上目前不支援 Eclipse 外掛程式。 

## <a name="next-steps"></a>後續步驟
* 開始使用 [Eclipse](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started-eclipse)
* 查看其他 [Java 範例](https://github.com/Azure-Samples/service-fabric-java-getting-started)


<!-- Image references -->

[publishdialog]: ./media/service-fabric-manage-multiple-environment-app-configuration/publish-dialog-choose-app-config.png
[app-parameters-solution-explorer]:./media/service-fabric-manage-multiple-environment-app-configuration/app-parameters-in-solution-explorer.png