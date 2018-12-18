---
title: Azure Service Fabric 映像存放區連接字串 | Microsoft Docs
description: 了解映像存放區連接字串
services: service-fabric
documentationcenter: .net
author: alexwun
manager: timlt
editor: ''
ms.assetid: 00f8059d-9d53-4cb8-b44a-b25149de3030
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/27/2018
ms.author: alexwun
ms.openlocfilehash: 8a11f9c9ebc2dd0b0eabf7a34d5ef38ae4e29309
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44719065"
---
# <a name="understand-the-imagestoreconnectionstring-setting"></a>了解 ImageStoreConnectionString 設定

在某些文件中，我們簡單介紹 "ImageStoreConnectionString" 參數存在而不描述它真正的意義。 在瀏覽過[使用 PowerShell 部署和移除應用程式][10]類似的文章後，看起來您所要做的是複製/貼上值，如目標叢集的叢集資訊清單中所示。 所以每個叢集的設定必須是可設定，但當您透過 [Azure 入口網站][11]建立叢集時，沒有設定此設定的選項，而一律是 "fabric:ImageStore"。 那麼此設定的目的是什麼？

![叢集資訊清單][img_cm]

Service Fabric 由許多不同的小組以內部 Microsoft 耗用量的平台開始，所以其某些層面可以靈活自訂 -「映像存放區」就是這類項目。 基本上，映像存放區是可儲存應用程式套件隨的插即用存放庫。 當您的應用程式部署到叢集中的節點時，該節點會從映像存放區下載應用程式套件的內容。 ImageStoreConnectionString 是包含所有用戶端與節點之必要資訊的設定，供指定叢集找到正確的映像存放區。

目前有三種可能的映像存放區提供者，其對應的連接字串如下︰

1. 映像存放服務："fabric:ImageStore"

2. 檔案系統："file:[file system path]"

3. Azure 儲存體："xstore:DefaultEndpointsProtocol=https;AccountName=[...];AccountKey=[...];Container=[...]"

生產環境中使用的提供者類型是映像存放服務，這是您可以從 Service Fabric Explorer 看到的可設定狀態持續性系統服務。 

![映像存放服務][img_is]

在叢集本身的系統服務中裝載映像存放可排除套件存放庫的外部相依性，並讓我們更充分掌控儲存體位置。 有關映像存放的未來增強功能應該會先將目標放在映像存放區提供者 (如果不是專用)。 映像存放區服務提供者的連接字串沒有任何唯一的資訊，因為用戶端已連接至目標叢集。 用戶端只需要知道，應該使用目標為系統服務的通訊協定。

在開發期間會針對本機一整體叢集使用檔案系統提供者而非映像存放區服務，以稍微加速啟動叢集。 差別通常很小，但它在開發期間對於大部分人是有用的最佳化。 可以使用其他存放裝置提供者類型部署本機一整體叢集，但通常沒有理由這麼做，因為開發/測試工作流程將保持不變，不論提供者為何。 Azure 儲存體提供者的存在，只是要為在映像存放區服務提供者導入之前部署的舊叢集提供舊版支援。

此外，檔案系統提供者或 Azure 儲存體提供者均不應作為在多個叢集之間共用映像存放區的方法 - 這會導致叢集組態資料損毀，因為每個叢集都可以將衝突的資料寫入至映像存放區。 若要在多個叢集之間共用已佈建的應用程式套件，請改用 [sfpkg][12] 檔案，此檔案可透過下載 URI 上傳至任何外部存放區。

因此，當 ImageStoreConnectionString 可設定時，您只要使用預設設定即可。 透過 Visual Studio 發佈至 Azure 時，參數會自動針對您進行相對應的設定。 針對以程式設計方式部署至 Azure 中裝載的叢集，連接字串一律是 "fabric:ImageStore"。 但是當有疑問時，其值一律能夠透過 [PowerShell](https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricclustermanifest)[.NET](https://msdn.microsoft.com/library/azure/mt161375.aspx) 或 [REST](https://docs.microsoft.com/rest/api/servicefabric/get-a-cluster-manifest) 擷取叢集資訊清單來驗證。 內部部署測試與生產叢集也應該一律設定為使用映像存放區服務提供者。

### <a name="next-steps"></a>後續步驟
[使用 PowerShell 部署與移除應用程式][10]

<!--Image references-->
[img_is]: ./media/service-fabric-image-store-connection-string/image_store_service.png
[img_cm]: ./media/service-fabric-image-store-connection-string/cluster_manifest.png

[10]: service-fabric-deploy-remove-applications.md
[11]: service-fabric-cluster-creation-via-portal.md
[12]: service-fabric-package-apps.md#create-an-sfpkg
