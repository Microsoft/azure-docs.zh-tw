---
title: Azure Service Fabric 應用程式模型 | Microsoft Docs
description: 如何在 Service Fabric 中建立模型和描述應用程式與服務。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: mani-ramaswamy
ms.assetid: 17a99380-5ed8-4ed9-b884-e9b827431b02
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: ryanwi
ms.openlocfilehash: dd453f393620923041eb8fa07b551f4945f48235
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34204629"
---
# <a name="model-an-application-in-service-fabric"></a>在 Service Fabric 中模型化應用程式
本文提供 Azure Service Fabric 應用程式模型的概觀，以及如何透過資訊清單檔案定義應用程式和服務。

## <a name="understand-the-application-model"></a>了解應用程式模型
應用程式是組成服務的集合，這些服務會執行特定函數。 服務會執行完整且獨立的函式，並且可獨立於其他服務啟動和執行。  服務是由程式碼、組態和資料所組成。 針對每個服務，程式碼由可執行檔二進位檔組成、組態由可在執行階段載入的服務設定組成，而資料由讓服務使用的任意靜態資料組成。 此階層應用程式模型中的每個元件都可以獨立建立版本和升級。

![Service Fabric 應用程式模型][appmodel-diagram]

應用程式類型是應用程式的分類，由服務類型的組合所組成。 服務類型是一項服務分類。 分類可以有不同的設定和組態，但是核心功能保持不變。 服務的執行個體是相同服務類型的不同服務組態變形。  

應用程式和服務的類別 (或「類型」) 是透過 XML 檔案 (應用程式資訊清單和服務資訊清單) 來說明。  資訊清單可描述應用程式與服務，並且是應用程式可以從叢集的映像存放區具現化的範本。  [應用程式及服務資訊清單](service-fabric-application-and-service-manifests.md)中詳細說明資訊清單。 ServiceManifest.xml 和 ApplicationManifest.xml 檔案的結構描述定義是和 Service Fabric SDK 及工具一起安裝在 C:\Program Files\Microsoft SDKs\Service Fabric\schemas\ServiceFabricServiceModel.xsd。 XML 結構描述則記載在 [ServiceFabricServiceModel.xsd 結構描述文件](service-fabric-service-model-schema.md)中。

不同應用程式執行個體的程式碼會以個別程序的形式執行，即使是由相同的 Service Fabric 節點所裝載。 此外，每個應用程式執行個體的生命週期可以獨立進行管理 (例如，升級)。 下圖顯示應用程式類型如何由服務類型組成，依序分別為程式碼、組態和資料套件的組成。 為了簡化此圖，只會顯示 `ServiceType4` 的程式碼/組態/資料套件，但每個服務類型都包含這其中部分或所有的套件類型。

![Service Fabric 應用程式類型和服務類型][cluster-imagestore-apptypes]

叢集中可以有一或多個使用中的服務類型執行個體。 例如，可設定狀態的服務執行個體或複本，藉由複寫叢集中不同節點上複本之間的狀態達到高可靠性。 即使叢集中有一個節點失敗，複寫基本上會提供備援讓服務可供使用。 [分割服務](service-fabric-concepts-partitioning.md) 進一步在叢集中的節點之間分割其狀態 (並且存取該狀態的模式)。

下圖顯示應用程式和服務執行個體、分割和複本之間的關聯性。

![服務內的分割和複本][cluster-application-instances]

> [!TIP]
> 您可以使用 Service Fabric Explorer 工具，在叢集中檢視應用程式的配置，該工具可以在 http://&lt;yourclusteraddress&gt;:19080/Explorer 上取得。 如需詳細資訊，請參閱[使用 Service Fabric Explorer 視覺化叢集](service-fabric-visualizing-your-cluster.md)。
> 
> 


## <a name="next-steps"></a>後續步驟
- 深入了解[應用程式延展性](service-fabric-concepts-scalability.md)。
- 了解服務[狀態](service-fabric-concepts-state.md)、[分割](service-fabric-concepts-partitioning.md)和[可用性](service-fabric-availability-services.md)。
- 閱讀[應用程式及服務資訊清單](service-fabric-application-and-service-manifests.md)，以了解應用程式和服務的定義方式。
- [應用程式主控模型](service-fabric-hosting-model.md)說明已部署之服務的複本 (或執行個體) 與服務主機處理序之間的關聯性。

<!--Image references-->
[appmodel-diagram]: ./media/service-fabric-application-model/application-model.png
[cluster-imagestore-apptypes]: ./media/service-fabric-application-model/cluster-imagestore-apptypes.png
[cluster-application-instances]: media/service-fabric-application-model/cluster-application-instances.png


