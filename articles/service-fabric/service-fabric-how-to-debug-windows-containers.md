---
title: 使用 Service Fabric 和 VS 對 Windows 容器進行偵錯 | Microsoft Docs
description: 了解如何使用 Visual Studio 2017 對 Azure Service Fabric 中的 Windows 容器進行偵錯。
services: service-fabric
documentationcenter: .net
author: mikkelhegn
manager: msfussell
editor: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/14/2018
ms.author: mikhegn
ms.openlocfilehash: 180bd3709cc9ffefb17f78e337e6f6995024fdcf
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39523422"
---
# <a name="how-to-debug-windows-containers-in-azure-service-fabric-using-visual-studio-2017"></a>如何：使用 Visual Studio 2017 對 Azure Service Fabric 中的 Windows 容器進行偵錯

使用 Visual Studio 2017 Update 7 (15.7) 時，您可以在作為 Service Fabric 服務的容器中，對 .NET 應用程式進行偵錯。 本文說明如何設定環境，然後在執行於本機 Service Fabric 叢集的容器中，對 .NET 應用程式進行偵錯。

## <a name="prerequisites"></a>必要條件

* 在 Windows 10 上，請遵循本快速入門來[設定 Windows 10 以執行 Windows 容器](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10)
* 在 Windows Server 2016 上，請遵循本快速入門來[設定 Windows 2016 以執行 Windows 容器](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-server)
* 遵循[在 Windows 上準備您的開發環境](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started)，以設定本機 Service Fabric 環境

## <a name="configure-your-developer-environment-to-debug-containers"></a>設定開發人員環境以對容器進行偵錯

1. 先確定適用於 Windows 服務的 Docker 有在執行，再繼續進行下一個步驟。

1. 為了在容器之間支援 DNS 解析，您必須使用機器名稱來設定本機開發叢集。 如果您想要透過反向 Proxy 處理服務，則也需要執行這些步驟。
    1. 以系統管理員身分開啟 PowerShell
    2. 瀏覽至 SDK 叢集的安裝程式資料夾，通常是 `C:\Program Files\Microsoft SDKs\Service Fabric\ClusterSetup`。
    3. 使用 `-UseMachineName` 參數執行 `DevClusterSetup.ps1` 指令碼

       ``` PowerShell
         C:\Program Files\Microsoft SDKs\Service Fabric\ClusterSetup\DevClusterSetup.ps1 -UseMachineName
       ```

    > [!NOTE]
    > 您可以使用 `-CreateOneNodeCluster` 來設定單節點叢集。 依預設會建立五節點的本機叢集。
    >

    若要深入了解 Service Fabric 中的 DNS 服務，請參閱 [Azure Service Fabric 中的 DNS 服務](https://docs.microsoft.com/azure/service-fabric/service-fabric-dnsservice)。 若要深入了解如何從在容器中執行的服務使用 Service Fabric 反向 Proxy，請參閱[針對在容器中執行的服務進行反向 Proxy 特殊處理](service-fabric-reverseproxy.md#special-handling-for-services-running-in-containers)。

### <a name="known-limitations-when-debugging-containers-in-service-fabric"></a>在對 Service Fabric 中的容器進行偵錯時的已知限制

以下是對 Service Fabric 中的容器進行偵錯時的已知限制和可能的解決方式清單：

* 對 ClusterFQDNorIP 使用 localhost 不會在容器中支援 DNS 解析。
    * 解決方式：使用機器名稱設定本機叢集 (請參閱上文)
* 在虛擬機器中執行 Windows10 不會讓 DNS 回覆返回容器。
    * 解決方式：對虛擬機器 NIC 上的 IPv4 停用 UDP 總和檢查碼卸載
    * 請注意，這會降低機器的網路效能。
    * https://github.com/Azure/service-fabric-issues/issues/1061
* 如果使用了 Docker Compose 來部署應用程式，則無法在 Windows10 上使用 DNS 服務名稱來解析相同應用程式中的服務
    * 解決方式：使用 servicename.applicationname 來解析服務端點
    * https://github.com/Azure/service-fabric-issues/issues/1062
* 如果對 ClusterFQDNorIP 使用 IP-address，則在主機上變更主要 IP 會破壞 DNS 功能。
    * 解決方式：使用主機上的新主要 IP 或使用機器名稱，來重新建立叢集。 原先的設計就是如此。
* 如果無法在網路上解析用來建立叢集的 FQDN，則 DNS 會失敗。
    * 解決方式：使用主機的主要 IP 來重新建立本機叢集。 原先的設計就是如此。
* 在對容器進行偵錯時，Docker 記錄只能透過 Visual Studio 輸出視窗取得，而無法透過 Service Fabric API (包括 Service Fabric Explorer) 取得

## <a name="debug-a-net-application-running-in-docker-containers-on-service-fabric"></a>針對在 Service Fabric 上的 Docker 容器中執行的 .NET 應用程式進行偵錯

1. 以系統管理員身分執行 Visual Studio。

1. 開啟現有的 .NET 應用程式，或新建一個。

1. 以滑鼠右鍵按一下專案，然後選取 [新增] -> [容器協調器支援] -> [Service Fabric]

1. 按 **F5** 開始對應用程式進行偵錯。

    Visual Studio 支援適用於 .NET 和 .NET Core 的主控台與 ASP.NET 專案類型。

## <a name="next-steps"></a>後續步驟
若要深入了解 Service Fabric 和容器的功能，請跟隨此連結：[Service Fabric 容器概觀](service-fabric-containers-overview.md)。
