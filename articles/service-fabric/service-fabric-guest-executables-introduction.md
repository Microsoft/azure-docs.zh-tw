---
title: 將現有的可執行檔部署至 Azure Service Fabric | Microsoft Docs
description: 了解如何將現有應用程式封裝為來賓可執行檔，使其可以部署至 Service Fabric 叢集。
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: ''
ms.assetid: d799c1c6-75eb-4b8a-9f94-bf4f3dadf4c3
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 03/15/2018
ms.author: mfussell
ms.openlocfilehash: cdaf3dae12c2c9da1f6bcbebbff560b98e62bade
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34212835"
---
# <a name="deploy-an-existing-executable-to-service-fabric"></a>將現有可執行檔部署至 Service Fabric
您可以在 Azure Service Fabric 中將任何類型的程式碼 (例如 Node.js、Java 或 C++) 當作服務來執行。 Service Fabric 將這些類型的服務稱為客體可執行檔。

Service Fabric 將來賓可執行檔視為無狀態服務。 因此會根據可用性和其他度量將它們放在叢集的節點上。 本文說明如何使用 Visual Studio 或命令列公用程式，封裝來賓執行檔並部署至 Service Fabric 叢集。

## <a name="benefits-of-running-a-guest-executable-in-service-fabric"></a>在 Service Fabric 中執行來賓可執行檔的優點
在 Service Fabric 叢集中執行來賓執行檔有幾個優點：

* 高可用性。 在 Service Fabric 中執行的應用程式都會具有高可用性。 Service Fabric 會確保應用程式執行個體正在執行。
* 健康狀況監視。 Service Fabric 健全狀況監控會偵測應用程式是否正在執行，如果發生失敗情況，則會提供診斷資訊。   
* 應用程式生命週期管理。 除了提供無需停機的升級，如果升級期間回報健全狀況不良事件，Service Fabric 也支援回復到舊版。    
* 密度。 您可以在叢集中執行多個應用程式，每個應用程式不必在自己的硬體上執行。
* 可探索性：藉由使用 REST，您可以呼叫 Service Fabric 命名服務來尋找叢集中的其他服務。 

## <a name="samples"></a>範例
* [封裝和部署來賓可執行檔的範例](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [兩個客體可執行檔 (C# 和 nodejs) 使用 REST 透過命名服務進行通訊的範例](https://github.com/Azure-Samples/service-fabric-dotnet-containers)

## <a name="overview-of-application-and-service-manifest-files"></a>應用程式和服務資訊清單檔案的概觀
在部署來賓執行檔的過程中，最好先了解 Service Fabric 封裝和部署模型，如[應用程式模型](service-fabric-application-model.md)中所述。 Service Fabric 封裝模型依賴兩個 XML 檔案：應用程式和服務資訊清單。 ApplicationManifest.xml 和 ServiceManifest.xml 檔案的結構描述定義是和 Service Fabric SDK 一起安裝在 C:\Program Files\Microsoft SDKs\Service Fabric\schemas\ServiceFabricServiceModel.xsd。

* **應用程式資訊清單**：應用程式資訊清單用來描述應用程式。 其中列出組成該應用程式的服務，以及用來定義應如何部署一或多個服務的其他參數，例如執行個體數目。

  在 Service Fabric 中，應用程式是部署和升級的單位。 應用程式可以當作單一單位來升級，而得以掌控可能的失敗和可能需要的回復。 Service Fabric 可保證升級程序一定成功，萬一升級失敗，也不會讓應用程式處於不明或不穩定的狀態。
* **服務資訊清單**：服務資訊清單描述服務的元件。 它包含資料，例如服務的名稱和類型、其程式碼、組態。 服務資訊清單還包含一些在服務部署後可用來設定服務的額外參數。

## <a name="application-package-file-structure"></a>應用程式套件檔案結構
為了將應用程式部署至 Service Fabric，應用程式應遵循預先定義的目錄結構。 以下是該結構的範例。

```
|-- ApplicationPackageRoot
    |-- GuestService1Pkg
        |-- Code
            |-- existingapp.exe
        |-- Config
            |-- Settings.xml
        |-- Data
        |-- ServiceManifest.xml
    |-- ApplicationManifest.xml
```

ApplicationPackageRoot 包含可定義應用程式的 ApplicationManifest.xml 檔案。 對於應用程式包含的每個服務，都有一個子目錄用來包含服務所需的所有構件。 這些子目錄是 ServiceManifest.xml，通常如下︰

* *Code*。 此目錄包含服務程式碼。
* *Config*。此目錄包含服務可在執行階段存取的 settings.xml 檔案 (和其他必要檔案)，以擷取特定組態設定。
* *Data*。 這是額外的目錄，用來儲存服務可能需要的其他本機資料。 資料應該只用來儲存短期資料。 如果必須重新定位服務 (例如在容錯移轉期間)，Service Fabric 不會將變更複製/複寫到資料目錄。

> [!NOTE]
> 如果不需要 `config` 和 `data` 目錄，則不必建立。
>
>

## <a name="next-steps"></a>後續步驟
請參閱下列文章以了解相關資訊和工作。
* [部署來賓可執行檔](service-fabric-deploy-existing-app.md)
* [部署多個來賓可執行檔](service-fabric-deploy-multiple-apps.md)
* [使用 Visual Studio 建立第一個來賓可執行檔應用程式](quickstart-guest-app.md)
* [封裝和部署來賓可執行檔的範例](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)，包括封裝工具預先發行版本的連結
* [兩個客體可執行檔 (C# 和 nodejs) 使用 REST 透過命名服務進行通訊的範例](https://github.com/Azure-Samples/service-fabric-containers)

