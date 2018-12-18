---
title: 快速入門 - 使用 Azure PowerShell 建立 Azure DNS 區域和記錄
description: 了解如何在 Azure DNS 中建立 DNS 區域和記錄。 這份逐步快速入門將引導您使用 Azure PowerShell 來建立和管理第一個 DNS 區域和第一筆記錄。
services: dns
author: vhorne
ms.service: dns
ms.topic: quickstart
ms.date: 07/16/2018
ms.author: victorh
ms.openlocfilehash: e5801e9ed512a32d793f7b4b71be86174f656ab0
ms.sourcegitcommit: e32ea47d9d8158747eaf8fee6ebdd238d3ba01f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39089973"
---
# <a name="quickstart-create-an-azure-dns-zone-and-record-using-azure-powershell"></a>快速入門：使用 Azure PowerShell 建立 Azure DNS 區域和記錄

在本快速入門中，您會使用 Azure PowerShell 建立第一個 DNS 區域和第一筆記錄。 您也可以使用 [Azure 入口網站](dns-getstarted-portal.md)或 [Azure CLI](dns-getstarted-cli.md) 來執行這些步驟。 

DNS 區域用來裝載特定網域的 DNS 記錄。 若要開始將網域裝載到 Azure DNS 中，您必須建立該網域名稱的 DNS 區域。 接著在此 DNS 區域內，建立網域的每筆 DNS 記錄。 最後，若要將 DNS 區域發佈至網際網路，您需要設定網域的名稱伺服器。 以下說明上述各步驟。

Azure DNS 也支援建立私人網域。 如需有關如何建立第一個私人 DNS 區域與記錄的逐步指示，請參閱[透過 PowerShell 開始使用 Azure DNS 私人區域 ](private-dns-getstarted-powershell.md)。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="create-the-resource-group"></a>建立資源群組

建立 DNS 區域之前，請先建立包含 DNS 區域的資源群組：

```powershell
New-AzureRMResourceGroup -name MyResourceGroup -location "eastus"
```

## <a name="create-a-dns-zone"></a>建立 DNS 區域

使用 `New-AzureRmDnsZone` Cmdlet 建立 DNS 區域。 下列範例會在稱為 MyResourceGroup 的資源群組中建立稱為 contoso.com 的 DNS 區域。 使用範例來建立 DNS 區域，並將值替換為您自己的值。

```powershell
New-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyResourceGroup
```

## <a name="create-a-dns-record"></a>建立 DNS 記錄

您可以使用 `New-AzureRmDnsRecordSet` Cmdlet 來建立記錄集。 下列範例會在資源群組 "MyResourceGroup" 中的 DNS 區域 "contoso.com" 中，建立具有相對名稱 "www" 的記錄。 記錄集的完整名稱是 "www.contoso.com"。 記錄類型為 'A'，IP 位址是 "1.2.3.4"，TTL 為 3600 秒。

```powershell
New-AzureRmDnsRecordSet -Name www -RecordType A -ZoneName contoso.com -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -IPv4Address "1.2.3.4")
```

## <a name="view-records"></a>檢視記錄

若要列出區域中的 DNS 記錄，請使用︰

```powershell
Get-AzureRmDnsRecordSet -ZoneName contoso.com -ResourceGroupName MyResourceGroup
```


## <a name="update-name-servers"></a>更新名稱伺服器

當您滿意 DNS 區域且已正確設定記錄之後，您必須設定網域名稱來使用 Azure DNS 名稱伺服器。 這可讓網際網路上的其他使用者找到您的 DNS 記錄。

`Get-AzureRmDnsZone` Cmdlet 可顯示您的區域的名稱伺服器：

```powershell
Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyResourceGroup

Name                  : contoso.com
ResourceGroupName     : myresourcegroup
Etag                  : 00000003-0000-0000-b40d-0996b97ed101
Tags                  : {}
NameServers           : {ns1-01.azure-dns.com., ns2-01.azure-dns.net., ns3-01.azure-dns.org., ns4-01.azure-dns.info.}
NumberOfRecordSets    : 3
MaxNumberOfRecordSets : 5000
```

這些名稱伺服器應該向網域名稱註冊機構 (您購買網域名稱的來源) 設定。 您的註冊機構會提供選項來設定網域的名稱伺服器。 如需詳細資訊，請參閱[將網域委派給 Azure DNS](dns-domain-delegation.md)。

## <a name="delete-all-resources"></a>刪除所有資源

不再需要時，您可以藉由刪除資源群組，刪除在本快速入門中建立的所有資源：

```powershell
Remove-AzureRMResourceGroup -Name MyResourceGroup
```

## <a name="next-steps"></a>後續步驟

既然您已經使用 Azure PowerShell 建立第一個 DNS 區域和第一筆記錄，就可以在自訂網域中為 Web 應用程式建立記錄。

> [!div class="nextstepaction"]
> [在自訂網域中建立 Web 應用程式的 DNS 記錄](./dns-web-sites-custom-domain.md)

