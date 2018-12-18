---
title: 教學課程 - 使用 Azure PowerShell 建立 Azure DNS 私人區域
description: 在本教學課程中，您可以建立並測試私人 DNS 區域，並記錄在 Azure DNS 中。 這份逐步指南將引導您使用 Azure PowerShell，來建立和管理第一個私人 DNS 區域和第一筆記錄。
services: dns
author: vhorne
ms.service: dns
ms.topic: tutorial
ms.date: 7/24/2018
ms.author: victorh
ms.openlocfilehash: 872227e0521bd54e6bf7fdbe3626dfca34170863
ms.sourcegitcommit: c2c64fc9c24a1f7bd7c6c91be4ba9d64b1543231
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39257720"
---
# <a name="tutorial-create-an-azure-dns-private-zone-using-azure-powershell"></a>教學課程：使用 Azure PowerShell 建立 Azure DNS 私人區域

本教學課程將逐步引導您使用 Azure PowerShell 來建立第一個私人 DNS 區域和第一筆記錄。

[!INCLUDE [private-dns-public-preview-notice](../../includes/private-dns-public-preview-notice.md)]

DNS 區域用來裝載特定網域的 DNS 記錄。 若要開始將網域裝載到 Azure DNS 中，您必須建立該網域名稱的 DNS 區域。 接著在此 DNS 區域內，建立網域的每筆 DNS 記錄。 若要將私人 DNS 區域發佈至虛擬網路，指定可以在區域內解析記錄的虛擬網路清單。  這稱為「解析虛擬網路」。 您也可以指定一個虛擬網路，每當 VM 建立、變更 IP 或刪除時，Azure DNS 就會維護其主機名稱記錄。  這稱為「註冊虛擬網路」。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 DNS 私人區域
> * 建立測試虛擬機器
> * 建立其他的 DNS 記錄
> * 測試私人區域

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

如果您想要，您可以使用 [Azure CLI](private-dns-getstarted-cli.md) 完成本教學課程。

<!--- ## Get the Preview PowerShell modules
These instructions assume you have already installed and signed in to Azure PowerShell, including ensuring you have the required modules for the Private Zone feature. -->

<!---[!INCLUDE [dns-powershell-setup](../../includes/dns-powershell-setup-include.md)] -->

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

## <a name="create-the-resource-group"></a>建立資源群組

首先，請建立包含 DNS 區域的資源群組： 

```azurepowershell
New-AzureRMResourceGroup -name MyAzureResourceGroup -location "eastus"
```

## <a name="create-a-dns-private-zone"></a>建立 DNS 私人區域

使用 `New-AzureRmDnsZone` Cmdlet 搭配值為 "Private" 的 ZoneType 參數來建立 DNS 區域。 下列範例會在 **MyAzureResourceGroup** 資源群組中建立 **contoso.local** DNS 區域，並使 **MyAzureVnet** 虛擬網路能夠使用此 DNS 區域。

如果省略 **ZoneType** 參數，則會將區域建立為公用區域，因此，這就是建立私人區域的必要參數。 

```azurepowershell
$backendSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name backendSubnet -AddressPrefix "10.2.0.0/24"
$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName MyAzureResourceGroup `
  -Location eastus `
  -Name myAzureVNet `
  -AddressPrefix 10.2.0.0/16 `
  -Subnet $backendSubnet

New-AzureRmDnsZone -Name contoso.local -ResourceGroupName MyAzureResourceGroup `
   -ZoneType Private `
   -RegistrationVirtualNetworkId @($vnet.Id)
```

如果您只想要建立名稱解析的區域 (沒有自動建立主機名稱)，可以使用 ResolutionVirtualNetworkId 參數，而不是 RegistrationVirtualNetworkId 參數。

> [!NOTE]
> 您將無法看到自動建立的主機名稱記錄。 但稍後，您將測試以確保它們存在。

### <a name="list-dns-private-zones"></a>列出 DNS 私人區域

您可以從 `Get-AzureRmDnsZone` 中省略區域名稱，以列舉資源群組中的所有區域： 此作業會傳回一系列的區域物件。

```azurepowershell
Get-AzureRmDnsZone -ResourceGroupName MyAzureResourceGroup
```

您可以從 `Get-AzureRmDnsZone` 中省略區域名稱和資源群組名稱，以列舉 Azure 訂用帳戶中的所有區域。

```azurepowershell
Get-AzureRmDnsZone
```

## <a name="create-the-test-virtual-machines"></a>建立測試虛擬機器

現在，請建立兩部虛擬機器，讓您可以測試私人 DNS 區域：

```azurepowershell
New-AzureRmVm `
    -ResourceGroupName "myAzureResourceGroup" `
    -Name "myVM01" `
    -Location "East US" `
    -subnetname backendSubnet `
    -VirtualNetworkName "myAzureVnet" `
    -addressprefix 10.2.0.0/24 `
    -OpenPorts 3389

New-AzureRmVm `
    -ResourceGroupName "myAzureResourceGroup" `
    -Name "myVM02" `
    -Location "East US" `
    -subnetname backendSubnet `
    -VirtualNetworkName "myAzureVnet" `
    -addressprefix 10.2.0.0/24 `
    -OpenPorts 3389
```

這需要幾分鐘才能完成。

## <a name="create-an-additional-dns-record"></a>建立其他的 DNS 記錄

您可以使用 `New-AzureRmDnsRecordSet` Cmdlet 來建立記錄集。 下列範例會在資源群組 **MyAzureResourceGroup** 中的 DNS 區域 **contoso.local** 中，建立具有相對名稱 **db** 的記錄。 記錄集的完整名稱是 **db.contoso.local**。 記錄類型為 "A"，IP 位址是 "10.2.0.4"，TTL 為 3600 秒。

```azurepowershell
New-AzureRmDnsRecordSet -Name db -RecordType A -ZoneName contoso.local `
   -ResourceGroupName MyAzureResourceGroup -Ttl 3600 `
   -DnsRecords (New-AzureRmDnsRecordConfig -IPv4Address "10.2.0.4")
```

### <a name="view-dns-records"></a>檢視 DNS 記錄

若要列出區域中的 DNS 記錄，請執行︰

```azurepowershell
Get-AzureRmDnsRecordSet -ZoneName contoso.local -ResourceGroupName MyAzureResourceGroup
```
請記住，您不會看到針對兩部測試虛擬機器自動建立的 A 記錄。

## <a name="test-the-private-zone"></a>測試私人區域

現在您可以測試 **contoso.local** 私人區域的名稱解析。

### <a name="configure-vms-to-allow-inbound-icmp"></a>設定 VM，以允許輸入 ICMP

您可以使用 Ping 命令來測試名稱解析。 因此，請在兩部虛擬機器上設定防火牆，以允許輸入的 ICMP 封包。

1. 連線至 myVM01，然後以系統管理員權限開啟 Windows PowerShell 視窗。
2. 執行以下命令：

   ```powershell
   New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
   ```

針對 myVM02 重複。

### <a name="ping-the-vms-by-name"></a>依照名稱 Ping VM

1. 從 myVM02 Windows PowerShell 命令提示字元中，使用自動註冊的主機名稱 Ping myVM01：
   ```
   ping myVM01.contoso.local
   ```
   您應該會看到如下所示的輸出：
   ```
   PS C:\> ping myvm01.contoso.local

   Pinging myvm01.contoso.local [10.2.0.4] with 32 bytes of data:
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time=1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128

   Ping statistics for 10.2.0.4:
       Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
   Approximate round trip times in milli-seconds:
       Minimum = 0ms, Maximum = 1ms, Average = 0ms
   PS C:\>
   ```
2. 現在 Ping 您先前建立的 **db** 名稱：
   ```
   ping db.contoso.local
   ```
   您應該會看到如下所示的輸出：
   ```
   PS C:\> ping db.contoso.local

   Pinging db.contoso.local [10.2.0.4] with 32 bytes of data:
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128

   Ping statistics for 10.2.0.4:
       Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
   Approximate round trip times in milli-seconds:
       Minimum = 0ms, Maximum = 0ms, Average = 0ms
   PS C:\>
   ```

## <a name="delete-all-resources"></a>刪除所有資源

當不再需要時，請刪除 **MyAzureResourceGroup** 資源群組，來刪除本教學課程中建立的資源。

```azurepowershell
Remove-AzureRMResourceGroup -Name MyAzureResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已部署私人 DNS 區域、建立 DNS 記錄以及測試區域。
接下來，您可以深入了解私人 DNS 區域。

> [!div class="nextstepaction"]
> [將 Azure DNS 用於私人網域](private-dns-overview.md)




