---
title: Azure Service Fabric Mesh 的狀態儲存體選項 | Microsoft Docs
description: 深入了解如何在 Azure Service Fabric Mesh 上執行的 Service Fabric Mesh 應用程式中可靠地儲存狀態。
services: service-fabric-mesh
keywords: ''
author: rwike77
ms.author: ryanwi
ms.date: 07/12/2018
ms.topic: conceptual
ms.service: service-fabric-mesh
manager: timlt
ms.openlocfilehash: 6aa268cf56bfb8be9c27a9e0d9e5c9f4464b0c9d
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39075510"
---
# <a name="state-management-with-service-fabric"></a>使用 Service Fabric 的狀態管理
Service Fabric 支援許多不同的狀態儲存體選項。 如需狀態管理模式和 Service Fabric 的概念性概觀，請參閱 [Service Fabric 概念：狀態](/azure/service-fabric/service-fabric-concepts-state)。 不論您的服務在 Service Fabric Mesh 內部或外部執行，這些相同的概念都適用。 

## <a name="state-storage-options-in-azure-service-fabric-mesh"></a>Azure Service Fabric Mesh 中的狀態儲存體選項
使用 Service Fabric Mesh，您可以輕鬆地部署新應用程式，並將它連線到 Azure 中裝載的現有資料存放區。 除了使用任何遠端資料庫，有數個選項可供儲存資料，其取決於服務需要本機或遠端儲存體。 

* 在本機儲存的已複寫資料
  * 可靠的集合 (預覽版中並未提供)
    * 一個程式庫，可實作要在服務中使用的資料結構，例如佇列和索引鍵值組
    * 這可讓您以最簡單且最快速的方式與資料互動，同時提供結合 Service Fabric Mesh 中智慧型路由的簡單分割區路由
  * Service Fabric 磁碟區驅動程式 (預覽版中並未提供)
    * Docker 磁碟區驅動程式，可將本機磁碟區掛接至容器
    * 這可讓您透任何可支援檔案儲存的 API 在本機儲存資料時，具有最大的彈性。

* 遠端儲存體
  * Azure 檔案服務磁碟區驅動程式
    * Docker 磁碟區驅動程式，可將 Azure 檔案服務共用掛接至容器
    * 透過任何可支援檔案儲存的 API，為您提供較低的效能，但同時是較為便宜且完全富有彈性的可靠資料選項。
    * [操作說明指南：使用 Azure 檔案服務磁碟區部署應用程式](service-fabric-mesh-howto-deploy-app-azurefiles-volume.md)
    
## <a name="next-steps"></a>後續步驟

如需應用程式模型的資訊，請參閱 [Service Fabric 資源](service-fabric-mesh-service-fabric-resources.md)
