---
title: 針對 Azure 虛擬網路閘道和連線進行疑難排解 - Azure CLI | Microsoft Docs
description: 此頁面說明如何使用 Azure 網路監看員來針對 Azure CLI 進行疑難排解
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 2838bc61-b182-4da8-8533-27db8fdbd177
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/19/2017
ms.author: jdial
ms.openlocfilehash: 5a8ac32f43ecadb8908582fa853fc3f2bec7ac55
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970986"
---
# <a name="troubleshoot-virtual-network-gateway-and-connections-using-azure-network-watcher-azure-cli"></a>使用 Azure 網路監看員 Azure CLI 來針對虛擬網路閘道和連線進行疑難排解

> [!div class="op_single_selector"]
> - [入口網站](diagnose-communication-problem-between-networks.md)
> - [PowerShell](network-watcher-troubleshoot-manage-powershell.md)
> - [Azure CLI](network-watcher-troubleshoot-manage-cli.md)
> - [REST API](network-watcher-troubleshoot-manage-rest.md)

網路監看員提供了許多功能，因為它的作用就是為了讓您了解您在 Azure 中的網路資源。 這些功能的其中之一便是資源疑難排解。 您可以透過入口網站、PowerShell、CLI 或 REST API 呼叫資源疑難排解。 一經呼叫，網路監看員就會檢查虛擬網路閘道或連線的健全狀況，並傳回其調查結果。

若要執行此文章中的步驟，您必須[安裝適用於 Mac、Linux 和 Windows 的 Azure 命令列介面 (CLI)](/cli/azure/install-azure-cli)。

## <a name="before-you-begin"></a>開始之前

此案例假設您已依照[建立網路監看員](network-watcher-create.md)中的步驟建立網路監看員。

如需支援的閘道類型清單，請瀏覽[支援的閘道類型](network-watcher-troubleshoot-overview.md#supported-gateway-types)。

## <a name="overview"></a>概觀

資源疑難排解可讓您針對虛擬網路閘道和連線所發生的問題進行疑難排解。 在要求進行資源疑難排解後，便會查詢並檢查記錄。 檢查完成時，就會傳回結果。 資源疑難排解要求是執行時間很長的要求，可能需要幾分鐘的時間才會傳回結果。 疑難排解記錄會儲存在指定儲存體帳戶的容器中。

## <a name="retrieve-a-virtual-network-gateway-connection"></a>擷取虛擬網路閘道連線

在此範例中，系統會對連線執行資源疑難排解。 您也可以將它傳遞給虛擬網路閘道。 下列 Cmdlet 會列出資源群組中的 VPN 連線。

```azurecli
az network vpn-connection list --resource-group resourceGroupName
```

在取得連線的名稱之後，您可以執行此命令來取得其資源識別碼︰

```azurecli
az network vpn-connection show --resource-group resourceGroupName --ids vpnConnectionIds
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶

資源疑難排解會傳回資源健全狀況的相關資料，它也會將記錄儲存到儲存體帳戶以供檢閱。 在此步驟中，我們會建立儲存體帳戶，如果已有現有的儲存體帳戶，您也可以使用它。

1. 建立儲存體帳戶

    ```azurecli
    az storage account create --name storageAccountName --location westcentralus --resource-group resourceGroupName --sku Standard_LRS
    ```

1. 取得儲存體帳戶金鑰

    ```azurecli
    az storage account keys list --resource-group resourcegroupName --account-name storageAccountName
    ```

1. 建立容器

    ```azurecli
    az storage container create --account-name storageAccountName --account-key {storageAccountKey} --name logs
    ```

## <a name="run-network-watcher-resource-troubleshooting"></a>執行網路監看員資源疑難排解

您可以使用 `az network watcher troubleshooting` Cmdlet 對資源進行疑難排解。 我們會將資源群組、網路監看員名稱、連線識別碼、儲存體帳戶識別碼，以及用以儲存疑難排解結果之 Blob 的路徑傳遞給此 Cmdlet。

```azurecli
az network watcher troubleshooting start --resource-group resourceGroupName --resource resourceName --resource-type {vnetGateway/vpnConnection} --storage-account storageAccountName  --storage-path https://{storageAccountName}.blob.core.windows.net/{containerName}
```

在執行此 Cmdlet 後，網路監看員會檢閱資源以驗證其健康狀態。 它會將結果傳回殼層，並將結果的記錄儲存在指定的儲存體帳戶中。

## <a name="understanding-the-results"></a>了解結果

動作文字會提供有關如何解決問題的一般指引。 如果問題有可行動作，則會提供附有其他指引的連結。 如果沒有其他指引，回應中會提供 URL 以供您開啟支援案例。  如需回應屬性和所含內容的詳細資訊，請瀏覽[網路監看員疑難排解概觀](network-watcher-troubleshoot-overview.md)

如需從 Azure 儲存體帳戶下載檔案的指示，請參閱[以 .NET 開始使用 Azure Blob 儲存體](../storage/blobs/storage-dotnet-how-to-use-blobs.md)。 另一項可用工具為儲存體總管。 有關儲存體總管的詳細資訊可以在下列連結找到︰[儲存體總管](http://storageexplorer.com/)

## <a name="next-steps"></a>後續步驟

如果設定已變更而停止了 VPN 連線，請參閱[管理網路安全性群組](../virtual-network/manage-network-security-group.md)以追蹤可能有問題的網路安全性群組和安全性規則。
