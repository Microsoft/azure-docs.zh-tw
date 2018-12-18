---
title: 管理 Azure DNS 中的 DNS 區域 - Azure CLI | Microsoft Docs
description: 您可以使用 Azure CLI 管理 DNS 區域。 本文說明如何在 Azure DNS 上更新、刪除及建立 DNS 區域。
services: dns
documentationcenter: na
author: vhorne
manager: timlt
ms.assetid: 8ab63bc4-5135-4ed8-8c0b-5f0712b9afed
ms.service: dns
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/27/2017
ms.author: victorh
ms.openlocfilehash: 7f132c513354a87b4f1bd37db4c6c6ed8e0fd041
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46949956"
---
# <a name="how-to-manage-dns-zones-in-azure-dns-using-the-azure-cli"></a>如何使用 Azure CLI 管理 Azure DNS 中的 DNS 區域

> [!div class="op_single_selector"]
> * [入口網站](dns-operations-dnszones-portal.md)
> * [PowerShell](dns-operations-dnszones.md)
> * [Azure CLI](dns-operations-dnszones-cli.md)


本指南說明如何使用適用於 Windows、Mac 和 Linux 的跨平台 Azure CLI 來管理 DNS 區域。 您也可以使用 [Azure PowerShell](dns-operations-dnszones.md) 或 Azure 入口網站來管理 DNS 區域。

本指南特別說明公用 DNS 區域。 如需在 Azure DNS 中使用 Azure CLI 管理私人區域的資訊，請參閱[開始使用 Azure CLI 處理 Azure DNS 私人區域](private-dns-getstarted-cli.md)。

## <a name="introduction"></a>簡介

[!INCLUDE [dns-create-zone-about](../../includes/dns-create-zone-about-include.md)]

## <a name="set-up-azure-cli-for-azure-dns"></a>設定適用於 Azure DNS 的 Azure CLI

### <a name="before-you-begin"></a>開始之前

在開始設定之前，請確認您具備下列項目。

* Azure 訂用帳戶。 如果您還沒有 Azure 訂用帳戶，則可以啟用 [MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。

* 安裝最新版的 Azure CLI，該 CLI 適用於 Windows、Linux 或 MAC。 您可以在 [安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-az-cli2)中取得詳細資訊。

### <a name="sign-in-to-your-azure-account"></a>登入您的 Azure 帳戶

開啟主控台視窗，並驗證您的認證。 如需詳細資訊，請參閱[從 Azure CLI 登入 Azure](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest)

```
az login
```

### <a name="select-the-subscription"></a>選取訂用帳戶

檢查帳戶的訂用帳戶。

```
az account list
```

選擇其中一個要使用的 Azure 訂用帳戶。

```azurecli
az account set --subscription "subscription name"
```

### <a name="optional-to-installuse-azure-dns-private-zones-feature-public-preview"></a>選擇性：安裝/使用 Azure DNS 私人區域功能 (公開預覽)
Azure DNS 私人區域功能是透過 Azure CLI 的擴充功能，以公開預覽版本發行。 安裝 “dns” Azure CLI 擴充功能 
```
az extension add --name dns
``` 

### <a name="create-a-resource-group"></a>建立資源群組

Azure 資源管理員需要所有的資源群組指定一個位置。 這用來作為該資源群組中資源的預設位置。 然而，因為所有 DNS 資源是全球性，而非區域性，資源群組位置的選擇不會對 Azure DNS 造成影響。

如果您使用現有的資源群組，則可略過此步驟。

```azurecli
az group create --name myresourcegroup --location "West US"
```

## <a name="getting-help"></a>取得說明

與 Azure DNS 相關的所有 Azure CLI 命令都會以 `az network dns` 開頭。 使用 `--help` 選項 (簡短形式為 `-h`) 即可取得每個命令的說明。  例如︰

```azurecli
az network dns --help
az network dns zone --help
az network dns zone create --help
```

## <a name="create-a-dns-zone"></a>建立 DNS 區域

使用 `az network dns zone create` 命令建立 DNS 區域。 如需協助，請參閱 `az network dns zone create -h`。

下列範例會在稱為 *MyResourceGroup* 的資源群組中建立稱為 *contoso.com* 的 DNS 區域：

```azurecli
az network dns zone create --resource-group MyResourceGroup --name contoso.com
```

### <a name="to-create-a-dns-zone-with-tags"></a>使用標籤建立 DNS 區域

下列範例示範如何使用 `--tags` 參數 (簡短形式為 `-t`)，利用 *project = demo* 和 *env = test* 這兩個 [Azure Resource Manager 標籤](dns-zones-records.md#tags)，建立 DNS 區域：

```azurecli
az network dns zone create --resource-group MyResourceGroup --name contoso.com --tags "project=demo" "env=test"
```

## <a name="get-a-dns-zone"></a>取得 DNS 區域

若要擷取 DNS 區域，請使用 `az network dns zone show`。 如需協助，請參閱 `az network dns zone show --help`。

下列範例會從資源群組 MyResourceGroup 傳回 DNS 區域 contoso.com 及其相關聯的資料。 

```azurecli
az network dns zone show --resource-group myresourcegroup --name contoso.com
```

以下是回應範例。

```json
{
  "etag": "00000002-0000-0000-3d4d-64aa3689d201",
  "id": "/subscriptions/147a22e9-2356-4e56-b3de-1f5842ae4a3b/resourceGroups/myresourcegroup/providers/Microsoft.Network/dnszones/contoso.com",
  "location": "global",
  "maxNumberOfRecordSets": 5000,
  "name": "contoso.com",
  "nameServers": [
    "ns1-04.azure-dns.com.",
    "ns2-04.azure-dns.net.",
    "ns3-04.azure-dns.org.",
    "ns4-04.azure-dns.info."
  ],
  "numberOfRecordSets": 4,
  "resourceGroup": "myresourcegroup",
  "tags": {},
  "type": "Microsoft.Network/dnszones"
}
```

請注意，`az network dns zone show` 不會傳回 DNS 記錄。 若要列出 DNS 記錄，請使用 `az network dns record-set list`。


## <a name="list-dns-zones"></a>列出 DNS 區域

若要列舉 DNS 區域，請使用 `az network dns zone list`。 如需協助，請參閱 `az network dns zone list --help`。

指定資源群組只會列出資源群組內的區域︰

```azurecli
az network dns zone list --resource-group MyResourceGroup
```

省略資源群組則會列出訂用帳戶中的所有區域︰

```azurecli
az network dns zone list 
```

## <a name="update-a-dns-zone"></a>更新 DNS 區域

您可以使用 `az network dns zone update`變更 DNS 區域資源。 如需協助，請參閱 `az network dns zone update --help`。

此命令不會更新區域內的任何 DNS 記錄集 (請參閱[如何管理 DNS 記錄](dns-operations-recordsets-cli.md))。 它只用來更新區域資源本身的屬性。 這些屬性目前僅限於區域資源的 [Azure Resource Manager「標籤」](dns-zones-records.md#tags)。

下列範例示範如何更新 DNS 區域上的標籤。 現有標籤會由指定值取代。

```azurecli
az network dns zone update --resource-group myresourcegroup --name contoso.com --set tags.team=support
```

## <a name="delete-a-dns-zone"></a>刪除 DNS 區域

可以使用 `az network dns zone delete`刪除 DNS 區域。 如需協助，請參閱 `az network dns zone delete --help`。

> [!NOTE]
> 刪除 DNS 區域也會刪除該區域內的所有 DNS 記錄。 此作業無法復原。 如果 DNS 區域正在使用中，當該區域遭到刪除時，使用該區域的服務將會失敗。
>
>若要防止區域意外遭到刪除，請參閱[如何保護 DNS 區域和記錄](dns-protect-zones-recordsets.md)。

此命令會提示您確認。 選擇性的 `--yes` 參數會隱藏這個提示。

下列範例示範如何從資源群組 MyResourceGroup 刪除區域 contoso.com。

```azurecli
az network dns zone delete --resource-group myresourcegroup --name contoso.com
```

## <a name="next-steps"></a>後續步驟

了解如何在 DNS 區域中[管理記錄集和記錄](dns-getstarted-create-recordset-cli.md)。

了解如何[將您的網域委派給 Azure DNS](dns-domain-delegation.md)。

