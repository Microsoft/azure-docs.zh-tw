---
title: 教學課程 - 清除 Azure Service Fabric Mesh 資源 | Microsoft Docs
description: 了解如何移除 Azure Service Fabric Mesh 資源，讓您無需為不再使用的資源支付費用。
services: service-fabric-mesh
documentationcenter: .net
author: TylerMSFT
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/18/2018
ms.author: twhitney
ms.custom: mvc, devcenter
ms.openlocfilehash: fb7a444c54a57e7f2c38d941eb99f2fea7eebcef
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993352"
---
# <a name="tutorial-remove-azure-resources"></a>教學課程：移除 Azure 資源

本教學課程是系列課程中的第五個部分，說明如何刪除應用程式及其資源，以免支付其費用。

在本教學課程中，您將了解如何：
> [!div class="checklist"]
> * 清除應用程式所使用的資源，以免產生相關費用。

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [在 Visual Studio 中建立 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * [偵錯在本機開發叢集中執行的 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [部署 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * [升級 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-upgrade.md)
> * 清除 Service Fabric Mesh 資源

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您尚未部署待辦事項應用程式，請依照[發佈 Service Fabric Mesh Web 應用程式](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)中的指示進行操作。

## <a name="clean-up-resources"></a>清除資源

本教學課程即將結束。 在您所建立的資源使用完畢後，請加以刪除，如此您即無需為不再使用的資源支付費用。 這一點非常重要，因為 Mesh 是以秒計費的無伺服器服務。 若要深入了解 Mesh 定價，請參閱 https://aka.ms/sfmeshpricing。

Azure 所提供的便利措施之一，是在您建立與特定資源群組相關聯的資源後，刪除該資源群組即可刪除所有相關聯的資源。 如此，您就無須逐一加以刪除。

由於您已建立新的資源群組來建立待辦事項應用程式，您可以安全地刪除此資源群組，而這也會刪除所有相關聯的資源。

```azurecli
az group delete --resource-group sfmeshTutorial1RG
```

```powershell
Remove-AzureRmResourceGroup -Name sfmeshTutorial1RG
```

或者，您可以[從入口網站](../azure-resource-manager/resource-group-portal.md#delete-resource-group-or-resources)刪除 **sfmeshTutorial1RG** 資源群組。 

## <a name="next-steps"></a>後續步驟

現在您已將 Service Fabric Mesh 應用程式發佈至 Azure，接著請嘗試下列作業：

* 若要檢視服務對服務通訊的另一個範例，請瀏覽[投票應用程式範例](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/votingapp)
* 若要深入了解 Service Fabric 資源模型，請參閱 [Service Fabric Mesh 資源模型](service-fabric-mesh-service-fabric-resources.md)。
* 若要深入了解 Service Fabric Mesh，請閱讀 [Service Fabric Mesh 概觀](service-fabric-mesh-overview.md)。
* 閱讀 [Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview) 相關資訊