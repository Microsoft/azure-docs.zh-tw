---
title: 快速入門 - 使用 Azure CLI 建立 Azure DNS 區域和記錄
description: 快速入門 - 了解如何在 Azure DNS 中建立 DNS 區域和記錄。 這份逐步指南將引導您使用 Azure CLI 建立和管理第一個 DNS 區域和記錄。
services: dns
author: vhorne
ms.service: dns
ms.topic: quickstart
ms.date: 7/16/2018
ms.author: victorh
ms.openlocfilehash: f80488f555cfa3b7be6f35b9f23ea0a501a27fd9
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48831592"
---
# <a name="quickstart-create-an-azure-dns-zone-and-record-using-azure-cli"></a>快速入門：使用 Azure CLI 建立 Azure DNS 區域和第一筆記錄

本文將逐步引導您使用 Azure CLI (適用於 Windows、Mac 和 Linux)，建立第一個 DNS 區域和記錄。 您也可以使用 [Azure 入口網站](dns-getstarted-portal.md)或 [Azure PowerShell](dns-getstarted-powershell.md) 來執行這些步驟。

DNS 區域用來裝載特定網域的 DNS 記錄。 若要開始將網域裝載到 Azure DNS 中，您必須建立該網域名稱的 DNS 區域。 接著在此 DNS 區域內，建立網域的每筆 DNS 記錄。 最後，若要將 DNS 區域發佈至網際網路，您需要設定網域的名稱伺服器。 以下說明上述各步驟。

Azure DNS 現在也支援私人 DNS 區域 (目前處於公開預覽狀態)。 若要深入了解私人 DNS 區域，請參閱[使用 Azure DNS 私人網域](private-dns-overview.md)。 如需如何建立私人 DNS 區域的範例，請參閱[利用 CLI 開始使用 Azure DNS 私人區域](./private-dns-getstarted-cli.md)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="create-the-resource-group"></a>建立資源群組

建立 DNS 區域之前，請先建立包含 DNS 區域的資源群組：

```azurecli
az group create --name MyResourceGroup --location "East US"
```

## <a name="create-a-dns-zone"></a>建立 DNS 區域

使用 `az network dns zone create` 命令建立 DNS 區域。 若要查看此命令的說明，請輸入 `az network dns zone create -h`。

下列範例會在 MyResourceGroup 資源群組中建立稱為 contoso.com 的 DNS 區域。 使用範例來建立 DNS 區域，並將值替換為您自己的值。

```azurecli
az network dns zone create -g MyResourceGroup -n contoso.com
```

## <a name="create-a-dns-record"></a>建立 DNS 記錄

若要建立 DNS 記錄，請使用 `az network dns record-set [record type] add-record` 命令。 如需 A 記錄的說明，請參閱 `azure network dns record-set A add-record -h`。

下列範例會在資源群組 "MyResourceGroup" 中的 DNS 區域 "contoso.com" 中，建立具有相對名稱 "www" 的記錄。 記錄集的完整名稱是 "www.contoso.com"。 記錄類型為 "A"，IP 位址是 "1.2.3.4"，預設 TTL 為 3600 秒 (1 小時)。

```azurecli
az network dns record-set a add-record -g MyResourceGroup -z contoso.com -n www -a 1.2.3.4
```

## <a name="view-records"></a>檢視記錄

若要列出區域中的 DNS 記錄，請執行︰

```azurecli
az network dns record-set list -g MyResourceGroup -z contoso.com
```

## <a name="update-name-servers"></a>更新名稱伺服器

當您滿意 DNS 區域且已正確設定記錄之後，您必須設定網域名稱來使用 Azure DNS 名稱伺服器，讓網際網路上的其他使用者能找到您的 DNS 記錄。

`az network dns zone show` 命令可顯示您的區域的名稱伺服器。 若要查看名稱伺服器的名稱，請使用 JSON 輸出，如下列範例所示。

```azurecli
az network dns zone show -g MyResourceGroup -n contoso.com -o json

{
  "etag": "00000003-0000-0000-b40d-0996b97ed101",
  "id": "/subscriptions/a385a691-bd93-41b0-8084-8213ebc5bff7/resourceGroups/myresourcegroup/providers/Microsoft.Network/dnszones/contoso.com",
  "location": "global",
  "maxNumberOfRecordSets": 5000,
  "name": "contoso.com",
  "nameServers": [
    "ns1-01.azure-dns.com.",
    "ns2-01.azure-dns.net.",
    "ns3-01.azure-dns.org.",
    "ns4-01.azure-dns.info."
  ],
  "numberOfRecordSets": 3,
  "resourceGroup": "myresourcegroup",
  "tags": {},
  "type": "Microsoft.Network/dnszones"
}
```

這些名稱伺服器應該向網域名稱註冊機構 (您購買網域名稱的來源) 設定。 您的註冊機構會提供選項來設定網域的名稱伺服器。 如需詳細資訊，請參閱[將網域委派給 Azure DNS](dns-domain-delegation.md)。

## <a name="delete-all-resources"></a>刪除所有資源
 
不再需要時，您可以藉由刪除資源群組，刪除在本快速入門中建立的所有資源：

```azurecli
az group delete --name MyResourceGroup
```

## <a name="next-steps"></a>後續步驟

既然您已經使用 Azure CLI 建立第一個 DNS 區域和第一筆記錄，就可以在自訂網域中為 Web 應用程式建立記錄。

> [!div class="nextstepaction"]
> [在自訂網域中建立 Web 應用程式的 DNS 記錄](./dns-web-sites-custom-domain.md)
