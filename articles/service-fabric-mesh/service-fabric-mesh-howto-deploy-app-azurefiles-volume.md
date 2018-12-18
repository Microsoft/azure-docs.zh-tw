---
title: 在容器內裝載以 Azure 檔案服務為基礎的磁碟區，以在 Azure Service Fabric Mesh 應用程式中儲存狀態 | Microsoft Docs
description: 了解如何使用 Azure CLI 在容器內裝載以 Azure 檔案服務為基礎的磁碟區，以在 Azure Service Fabric Mesh 應用程式中儲存狀態。
services: service-fabric-mesh
documentationcenter: .net
author: rwike77
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: azure-cli
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/09/2018
ms.author: ryanwi
ms.custom: mvc, devcenter
ms.openlocfilehash: 3b350deff2883761af6a3a2b3c5c9ef22235bde0
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037966"
---
# <a name="store-state-in-an-azure-service-fabric-mesh-application-by-mounting-an-azure-files-based-volume-inside-the-container"></a>在容器內裝載以 Azure 檔案服務為基礎的磁碟區，以在 Azure Service Fabric Mesh 應用程式中儲存狀態

本文會說明如何在 Service Fabric Mesh 應用程式的容器中裝載磁碟區，以儲存 Azure 檔案服務中的狀態。 在此範例中，計數器應用程式會透過 ASP.NET Core 服務建立網頁，以在瀏覽器中顯示計數器的值。 

`counterService` 會定期從檔案讀取計數器的值、增加該值並將其寫回檔案。 儲存檔案的資料夾會裝載在由 Azure 檔案服務共用支援的磁碟區中。

## <a name="prerequisites"></a>必要條件

您可以使用 Azure Cloud Shell 或安裝在本機的 Azure CLI 來完成此工作。 若要依照本文的指示使用 Azure CLI，請確認 `az --version` 至少傳回 `azure-cli (2.0.43)`。  請依照下列[指示](service-fabric-mesh-howto-setup-cli.md)安裝 (或更新) Azure Service Fabric Mesh CLI 擴充功能模組。

## <a name="sign-in-to-azure"></a>登入 Azure

登入 Azure 並設定您的訂用帳戶。

```azurecli-interactive
az login
az account set --subscription "<subscriptionID>"
```

## <a name="create-a-file-share"></a>建立檔案共用

依照這些[指示](/azure/storage/files/storage-how-to-create-file-share)來建立 Azure 檔案共用。 在下列指示中，儲存體帳戶名稱、儲存體帳戶金鑰和檔案共用名稱會以 `<storageAccountName>`、`<storageAccountKey>` 和 `<fileShareName>` 來表示。 您可以在 Azure 入口網站中取得這些值：
* <storageAccountName> - 在 [儲存體帳戶] 下方，會顯示您在建立檔案共用時所使用的儲存體帳戶名稱。
* <storageAccountKey> - 在 [儲存體帳戶] 下方選取您的儲存體帳戶，然後選取 [存取金鑰]，並使用 **key1** 下方的值。
* <fileShareName> - 在 [儲存體帳戶] 下方選取您的儲存體帳戶，然後選取 [檔案]。 要使用的名稱是您剛才建立的檔案共用名稱。

## <a name="create-a-resource-group"></a>建立資源群組

建立要部署應用程式的資源群組。 下列命令會在美國東部的位置中建立名為 `myResourceGroup` 的資源群組。

```azurecli-interactive
az group create --name myResourceGroup --location eastus 
```

## <a name="deploy-the-template"></a>部署範本

請使用下列命令建立應用程式和相關資源，並提供先前在[建立檔案共用](#create-a-file-share)步驟中取得的 `storageAccountName`、`storageAccountKey` 和 `fileShareName` 值。

範本中的 `storageAccountKey` 參數是安全字串。 這不會顯示在部署狀態和 `az mesh service show` 命令中。 請確定已在下列命令中正確指定此參數。

下列命令會使用 [mesh_rp.linux.json 範本](https://sfmeshsamples.blob.core.windows.net/templates/counter/mesh_rp.linux.json)來部署 Linux 應用程式。 若要部署 Windows 應用程式，請使用 [mesh_rp.windows.json 範本](https://sfmeshsamples.blob.core.windows.net/templates/counter/mesh_rp.windows.json)。 請注意，較大的容器映像可能需要較長的部署時間。

```azurecli-interactive
az mesh deployment create --resource-group myResourceGroup --template-uri https://sfmeshsamples.blob.core.windows.net/templates/counter/mesh_rp.linux.json  --parameters "{\"location\": {\"value\": \"eastus\"}, \"fileShareName\": {\"value\": \"<fileShareName>\"}, \"storageAccountName\": {\"value\": \"<storageAccountName>\"}, \"storageAccountKey\": {\"value\": \"<storageAccountKey>\"}}"
```

命令應會在幾分鐘內傳回以下回應：`counterApp has been deployed successfully on counterAppNetwork with public ip address <IP Address>`

## <a name="open-the-application"></a>開啟應用程式

部署命令會傳回服務端點的公用 IP 位址。 在應用程式成功部署後，請取得服務端點的公用 IP 位址，並在瀏覽器中將其開啟。 網頁會隨即出現，其中包含每秒更新的計數器值。

此應用程式的網路資源名稱為 `counterAppNetwork`。 您可以使用下列命令查看應用程式的相關資訊，例如其描述、位置、資源群組等：

```azurecli-interactive
az mesh network show --resource-group myResourceGroup --name counterAppNetwork
```

## <a name="verify-that-the-application-is-able-to-use-the-volume"></a>確認應用程式都可以使用磁碟區

應用程式會在 `counter/counterService` 資料夾的檔案共用內建立名為 `counter.txt` 的檔案。 此檔案的內容就是正在網頁上顯示計的數器值。

您可以下載此檔案，但用來下載的工具須能瀏覽 Azure 檔案服務的檔案共用，例如 [Microsoft Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)。

## <a name="delete-the-resources"></a>刪除資源

請勤加刪除您已不會在 Azure 中使用的資源。 若要刪除此範例的相關資源，請使用下列命令，將資源部署所在的資源群組刪除 (這會刪除所有與該資源群組相關聯的項目)：

```azurecli-interactive
az group delete --resource-group myResourceGroup
```

## <a name="next-steps"></a>後續步驟

- 在 [GitHub](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/counter) 上檢視 Azure 檔案服務磁碟區的範例應用程式。
- 若要深入了解 Service Fabric 資源模型，請參閱 [Service Fabric Mesh 資源模型](service-fabric-mesh-service-fabric-resources.md)。
- 若要深入了解 Service Fabric Mesh，請閱讀 [Service Fabric Mesh 概觀](service-fabric-mesh-overview.md)。