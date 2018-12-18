---
title: 在 Azure 容器執行個體中掛接 emptyDir 磁碟區
description: 了解如何在 Azure 容器執行個體中掛接 emptyDir 磁碟區，以在容器群組中的容器之間共用資料
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: article
ms.date: 02/08/2018
ms.author: danlep
ms.openlocfilehash: 98a72123a05fa7d8dc16be7ddb787f2a2cf7e4d1
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48857647"
---
# <a name="mount-an-emptydir-volume-in-azure-container-instances"></a>在 Azure 容器執行個體中掛接 emptyDir 磁碟區

了解如何在 Azure 容器執行個體中掛接 *emptyDir* 磁碟區，以在容器群組中的容器之間共用資料。

> [!NOTE]
> 目前只有 Linux 容器才能掛接 *emptyDir* 磁碟區。 雖然我們致力於將所有功能帶入 Windows 容器，但是您可以在 [Azure 容器執行個體配額和區域可用性](container-instances-quotas.md)中找到目前的平台差異。

## <a name="emptydir-volume"></a>emptyDir 磁碟區

*emptyDir* 磁碟區提供了可供容器群組中每個容器存取的可寫入的目錄。 群組中的容器可以讀取和寫入磁碟區中的相同檔案，且可以在每個容器中使用相同或不同的路徑掛接它。

部分範例會針對 *emptyDir* 磁碟區使用下列項目：

* 塗銷空間
* 長時間執行工作期間的檢查點
* 側車容器所擷取和應用程式容器所提供的儲存資料

*emptyDir* 磁碟區中的資料會透過容器損毀保存。 不過，重新啟動的容器不保證會保存 *emptyDir* 磁碟區中的資料。

## <a name="mount-an-emptydir-volume"></a>掛接 emptyDir 磁碟區

若要在容器執行個體中掛接 emptyDir 磁碟區，您必須使用 [Azure Resource Manager 範本](/azure/templates/microsoft.containerinstance/containergroups)進行部署。

首先，填入範本的容器群組 `properties` 區段中的 `volumes` 陣列。 接下來，針對您想要掛接 *emptyDir* 磁碟區所在容器群組中的每個容器，填入容器定義之 `properties` 區段中的 `volumeMounts` 陣列。

例如，下列 Resource Manager 範本會建立一個由兩個容器 (個別掛接了 *emptyDir* 磁碟區) 組成的容器群組：

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-emptydir.json --> [!code-json[volume-emptydir](~/azure-docs-json-samples/container-instances/aci-deploy-volume-emptydir.json)]

若要使用 Azure Resource Manager 範本來查看容器執行個體部署範例，請參閱[在 Azure 容器執行個體中部署多個容器群組](container-instances-multi-container-group.md)。

## <a name="next-steps"></a>後續步驟

了解如何在 Azure 容器執行個體中掛接其他類型的磁碟區：

* [在 Azure 容器執行個體中掛接 Azure 檔案共用](container-instances-volume-azure-files.md)
* [在 Azure 容器執行個體中掛接 gitRepo 磁碟區](container-instances-volume-gitrepo.md)
* [在 Azure 容器執行個體中掛接秘密磁碟區](container-instances-volume-secret.md)