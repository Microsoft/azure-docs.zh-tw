---
title: Azure Service Fabric 資源模型簡介 |Microsoft Docs
description: 了解 Service Fabric 資源模型，一種定義 Service Fabric Mesh 應用程式的簡化方法。
services: service-fabric-mesh
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/12/2018
ms.author: vturecek
ms.custom: mvc, devcenter
ms.openlocfilehash: 6a96995a46cd2ad0a1afe4f4afe49d62409f2b68
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39075146"
---
# <a name="introduction-to-service-fabric-resource-model"></a>Azure Service Fabric 資源模型簡介

Service Fabric 資源模型描述一種簡單的方法，可定義組成 Service Fabric Mesh 應用程式的資源。 以下是這個模型目前支援的資源類型：

- [應用程式]
- 服務
- 磁碟區
- 網路

描述每個資源時，會將它們視為宣告的資源檔，也就是簡單的 YAML 或 JSON 文件來描述 Mesh 應用程式，而且是由 Service Fabric 平台佈建的。

## <a name="resource-overview"></a>資源概觀

### <a name="applications-and-services"></a>應用程式與服務

應用程式資源是 Mesh 應用程式部署、版本控制以及存留期的單位。 它包含的一個或多個代表微服務的服務資源。 而每個服務資源會包含一個或多個程式碼封裝，這些封裝會描述執行程式碼套件相關容器映像時所需的內容，包括：

- 容器名稱、版本和登錄
- 每個容器所需的 CPU 和記憶體資源
- 網路端點
- 容器中要掛接的磁碟區，參考不同的磁碟區資源。

所有被定義為服務資源的程式碼封裝，會以整體的形式加以部署和啟動。 服務資源也會描述要執行的服務執行個體數目，也會參考它依賴的其他資源 (例如，網路資源)。

如果 Mesh 應用程式是由多個服務所組成，不保證它們會在相同節點上同時執行。 此外，應用程式升級時，某一個服務升級失敗會導致所有服務要回復成以前的版本。



```yaml
    services:
      - name: helloWorldService
        properties:
          description: Hello world service.
          osType: linux
          codePackages:
            - name: helloworld
              image: myapp:1.0-alpine
              resources:
                requests:
                  cpu: 2
                  memoryInGB: 2
              endpoints:
                - name: helloWorldEndpoint
                  port: 8080
    replicaCount: 3
    networkRefs:
      - name: mynetwork
```

如前文所述，每個應用程式執行個體的生命週期都可以獨立管理。 例如，應用程式執行個體可以獨立升級。 通常，您會儘可能將最少量的服務放入應用程式中，因為放入應用程式的服務越多，就越不容易管理每個服務。

### <a name="networks"></a>網路

網路資源是個別部署的資源，與那些會參考它的應用程式或服務資源 (相依項) 無關。 它用來建立應用程式的私人網路。 不同應用程式的多個服務可以是屬於同一個網路。

> [!NOTE]
> 目前的預覽版本僅支援應用程式和網路之間的一對一對應

### <a name="volumes"></a>磁碟區

磁碟區是容器裡面掛接的目錄，可用來保存狀態。 磁碟區資源是一種宣告方式，可描述目錄的掛接辺式以及其支援儲存體。

## <a name="programming-models"></a>程式設計模型
服務資源只需要一個容器映像即可執行，而且資源相關的程式碼套件會參照該映像。 您可以在容器裡面使用任何架構，不需要知道或使用 Service Fabric Mesh 專用的 API，即可執行以任何語言撰寫的程式碼。 

您的應用程式程式碼即便是在 Service Fabric Mesh 外面，仍然可移植。而且您的應用程式部署會保持一致，無論實作服務時所採用的語言或架構為何。 無論您的應用程式是 ASP.NET Core、 Go 或只是一組處理程序和指令碼，Service Fabric Mesh 資源部署模型保持不變。 

## <a name="deployment"></a>部署

部署到 Service Fabric Mesh 的時候，會透過 HTTP 或 Azure CLI，將資源會部署為 Azure 的 Azure Resource Manager 範本。 


## <a name="next-steps"></a>後續步驟 
若要深入了解 Service Fabric Mesh，請閱讀下列概觀：
- [Service Fabric Mesh 概觀](service-fabric-mesh-overview.md)
