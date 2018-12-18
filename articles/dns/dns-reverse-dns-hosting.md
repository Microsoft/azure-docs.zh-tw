---
title: 在 Azure DNS 中託管反向 DNS 對應區域 | Microsoft Docs
description: 了解如何使用 Azure DNS 為您的 IP 範圍託管反向 DNS 對應區域
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/29/2017
ms.author: victorh
ms.openlocfilehash: e45b3bde0d5077a5d18369236e81bcd467527940
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990139"
---
# <a name="host-reverse-dns-lookup-zones-in-azure-dns"></a>在 Azure DNS 中託管反向 DNS 對應區域

本文說明如何在 Azure DNS 中為指派的 IP 範圍託管反向 DNS 對應區域。 必須將反向對應區域所代表的 IP 範圍指派給貴組織，通常是由您的 ISP 指派。

若要針對 Azure 所擁有且已派給您 Azure 服務的 IP 位址設定反向 DNS，請參閱[設定 Azure 託管服務的反向 DNS](dns-reverse-dns-for-azure-services.md)。

在閱讀本文之前，您應該先熟悉 [Azure 反向 DNS 和支援概觀](dns-reverse-dns-overview.md)。

此文章會逐步引導您使用 Azure 入口網站、Azure PowerShell、Azure 傳統 CLI 或 Azure CLI，建立第一個反向對應 DNS 區域和記錄。

## <a name="create-a-reverse-lookup-dns-zone"></a>建立反向對應 DNS 區域

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 在 [中樞] 功能表上，選取 [新增] > [網路]，然後選取 [DNS 區域]。

   ![「DNS 區域」選取項目](./media/dns-reverse-dns-hosting/figure1.png)

1. 在 [建立 DNS 區域] 窗格中，命名 DNS 區域。 區域名稱針對 IPv4 和 IPv6 前置詞的製作方式不同。 請遵循 [IPv4](#ipv4) 或 [IPv6](#ipv6) 的指示來命名區域。 當您完成時，選取 [建立] 來建立區域。

### <a name="ipv4"></a>IPv4

IPv4 反向對應區域的名稱是以其所代表的 IP 範圍為基礎。 格式應如下：`<IPv4 network prefix in reverse order>.in-addr.arpa`。 如需範例，請參閱 [Azure 反向 DNS 和支援概觀](dns-reverse-dns-overview.md#ipv4)。

> [!NOTE]
> 在 Azure DNS 中建立無類別反向 DNS 對應區域時，區域名稱中必須使用連字號 (`-`) 而不是正斜線 (`/`)。
>
> 例如，IP 範圍 192.0.2.128/26 必須使用 `128-26.2.0.192.in-addr.arpa` 作為區域名稱，而不是 `128/26.2.0.192.in-addr.arpa`。
>
> 雖然 DNS 標準支援這兩種方法，但 Azure DNS 並不支援包含正斜線 (`/`) 字元的 DNS 區域名稱。

下例示範如何透過 Azure 入口網站，在 Azure DNS 中建立名為 `2.0.192.in-addr.arpa` 的「類別 C」反向 DNS 區域：

 ![[建立 DNS 區域] 窗格，並填滿方塊](./media/dns-reverse-dns-hosting/figure2.png)

[資源群組位置] 會定義資源群組的位置。 它對於 DNS 區域沒有任何影響。 DNS 區域一定是「全域」位置，且不會顯示出來。

下列範例示範如何使用 Azure PowerShell 和 Azure CLI 完成這項工作。

#### <a name="powershell"></a>PowerShell

```powershell
New-AzureRmDnsZone -Name 2.0.192.in-addr.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Azure 傳統 CLI

```azurecli
azure network dns zone create MyResourceGroup 2.0.192.in-addr.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network dns zone create -g MyResourceGroup -n 2.0.192.in-addr.arpa
```

### <a name="ipv6"></a>IPv6

IPv6 反向對應區域的名稱格式應該如下：`<IPv6 network prefix in reverse order>.ip6.arpa`。  如需範例，請參閱 [Azure 反向 DNS 和支援概觀](dns-reverse-dns-overview.md#ipv6)。


下例示範如何透過 Azure 入口網站，在 Azure DNS 中建立名為 `0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa` 的 IPv6 反向 DNS 對應區域：

 ![[建立 DNS 區域] 窗格，並填滿方塊](./media/dns-reverse-dns-hosting/figure3.png)

[資源群組位置] 會定義資源群組的位置。 它對於 DNS 區域沒有任何影響。 DNS 區域一定是「全域」位置，且不會顯示出來。

下列範例示範如何使用 Azure PowerShell 和 Azure CLI 完成這項工作。

#### <a name="powershell"></a>PowerShell

```powershell
New-AzureRmDnsZone -Name 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Azure 傳統 CLI

```azurecli
azure network dns zone create MyResourceGroup 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network dns zone create -g MyResourceGroup -n 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

## <a name="delegate-a-reverse-dns-lookup-zone"></a>委派反向 DNS 對應區域

現在您已建立反向 DNS 對應區域，必須確定該區域是從父區域委派。 DNS 委派可讓 DNS 名稱解析程序尋找託管反向 DNS 對應區域的名稱伺服器。 然後，這些名稱伺服器可接聽您位址範圍中的 IP 位址 DNS 反向查詢。

至於正向對應區域，委派 DNS 區域的程序請參閱[將您的網域委派給 Azure DNS](dns-delegate-domain-azure-dns.md)。 反向對應區域的委派運作方式相同。 唯一的差別是您必須以提供 IP 範圍的 ISP ，而不是您的網域名稱註冊機構來設定名稱伺服器。

## <a name="create-a-dns-ptr-record"></a>建立 DNS PTR 記錄

### <a name="ipv4"></a>IPv4

下例會逐步解說在 Azure DNS 的反向 DNS 區域中建立 PTR 記錄的程序。 關於其他記錄類型和修改現有的記錄，請參閱[使用 Azure 入口網站管理 DNS 記錄和記錄集](dns-operations-recordsets-portal.md)。

1. 在 [DNS 區域] 窗格頂端，選取 [+ 記錄集] 以開啟 [新增記錄集] 窗格。

   ![建立記錄集的按鈕](./media/dns-reverse-dns-hosting/figure4.png)

1. PTR 記錄的記錄集名稱必須是反向順序的其餘 IPv4 位址。 

   在本例中，已填入前三個八位元當作區域名稱的一部分 (.2.0.192)。 因此，[名稱] 方塊中只會填入最後一個八位元。 例如，您可以將 IP 位址是 192.0.2.15 的資源記錄集命名為 **15**。  
1. 在 [類型] 中，選取 [PTR]。  
1. 在 [網域名稱] 中，輸入使用 IP 之資源的完整網域名稱 (FQDN)。
1. 選取窗格底部的 [確定]，建立 DNS 記錄。

 ![[新增記錄集] 窗格，並填滿方塊](./media/dns-reverse-dns-hosting/figure5.png)

下列範例示範如何使用 PowerShell 或 Azure CLI 完成這項工作。

#### <a name="powershell"></a>PowerShell

```powershell
New-AzureRmDnsRecordSet -Name 15 -RecordType PTR -ZoneName 2.0.192.in-addr.arpa -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -Ptrdname "dc1.contoso.com")
```
#### <a name="azure-classic-cli"></a>Azure 傳統 CLI

```azurecli
azure network dns record-set add-record MyResourceGroup 2.0.192.in-addr.arpa 15 PTR --ptrdname dc1.contoso.com  
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
    az network dns record-set ptr add-record -g MyResourceGroup -z 2.0.192.in-addr.arpa -n 15 --ptrdname dc1.contoso.com
```

### <a name="ipv6"></a>IPv6

下列範例會逐步引導您完成建立新 PTR 記錄的程序。 關於其他記錄類型和修改現有的記錄，請參閱[使用 Azure 入口網站管理 DNS 記錄和記錄集](dns-operations-recordsets-portal.md)。

1. 在 [DNS 區域] 窗格頂端，選取 [+ 記錄集] 以開啟 [新增記錄集] 窗格。

   ![建立記錄集的按鈕](./media/dns-reverse-dns-hosting/figure6.png)

2. PTR 記錄的記錄集名稱必須是反向順序的其餘 IPv6 位址。 它絕不能包含任何零壓縮。 

   在本例中，已填入 IPv6 的第一組 64 位元當作區域名稱的一部分 (0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa)。 因此，[名稱] 方塊中只會填入最後 64 個位元。 IP 位址的最後 64 個位元是以反向順序輸入，每個十六進位數字之間會使用句點作為分隔符號。 例如，您可以將 IP 位址是 2001:0db8:abdc:0000:f524:10bc:1af9:405e 的資源記錄集命名為 **e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f**。  
3. 在 [類型] 中，選取 [PTR]。  
4. 在 [網域名稱] 中，輸入使用 IP 之資源的 FQDN。
5. 選取窗格底部的 [確定]，建立 DNS 記錄。

![[新增記錄集] 窗格，並填滿方塊](./media/dns-reverse-dns-hosting/figure7.png)

下列範例示範如何使用 PowerShell 或 Azure CLI 完成這項工作。

#### <a name="powershell"></a>PowerShell

```powershell
New-AzureRmDnsRecordSet -Name "e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f" -RecordType PTR -ZoneName 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -Ptrdname "dc2.contoso.com")
```

#### <a name="azure-classic-cli"></a>Azure 傳統 CLI

```
azure network dns record-set add-record MyResourceGroup 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f PTR --ptrdname dc2.contoso.com 
```
 
#### <a name="azure-cli"></a>Azure CLI

```azurecli
    az network dns record-set ptr add-record -g MyResourceGroup -z 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -n e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f --ptrdname dc2.contoso.com
```

## <a name="view-records"></a>檢視記錄

若要檢視您所建立的記錄，請瀏覽至您在 Azure 入口網站中的 DNS 區域。 在 [DNS 區域] 窗格的下半部，您可以看到 DNS 區域的記錄。 您應該會看到預設 NS 與 SOA 記錄，以及您已建立的任何新記錄。 每個區域中都會建立 NS 與 SOA 記錄。 

### <a name="ipv4"></a>IPv4

[DNS 區域] 窗格會顯示 IPv4 PTR 記錄：

![包含 IPv4 記錄的 [DNS 區域] 窗格](./media/dns-reverse-dns-hosting/figure8.png)

下列範例示範如何使用 PowerShell 或 Azure CLI 來檢視 PTR 記錄。

#### <a name="powershell"></a>PowerShell

```powershell
Get-AzureRmDnsRecordSet -ZoneName 2.0.192.in-addr.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Azure 傳統 CLI

```azurecli
    azure network dns record-set list MyResourceGroup 2.0.192.in-addr.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
    azure network dns record-set list -g MyResourceGroup -z 2.0.192.in-addr.arpa
```

### <a name="ipv6"></a>IPv6

[DNS 區域] 窗格會顯示 IPv6 PTR 記錄：

![包含 IPv6 記錄的 [DNS 區域] 窗格](./media/dns-reverse-dns-hosting/figure9.png)

下列範例示範如何使用 PowerShell 或 Azure CLI 檢視記錄。

#### <a name="powershell"></a>PowerShell

```powershell
Get-AzureRmDnsRecordSet -ZoneName 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Azure 傳統 CLI

```azurecli
    azure network dns record-set list MyResourceGroup 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
    azure network dns record-set list -g MyResourceGroup -z 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

## <a name="faq"></a>常見問題集

### <a name="can-i-host-reverse-dns-lookup-zones-for-my-isp-assigned-ip-blocks-on-azure-dns"></a>我可以在 Azure DNS 上，為 ISP 指派的 IP 區塊託管反向 DNS 對應區域嗎？

是。 完全支援在 Azure DNS 中為您自己的 IP 範圍託管反向對應 (ARPA) 區域。

依本文所述，在 Azure DNS 中建立反向對應區域，然後使用您的 ISP [委派區域](dns-domain-delegation.md)。 接著，您就可以使用與其他記錄類型相同的方式來管理每個反向對應的 PTR 記錄。

### <a name="how-much-does-hosting-my-reverse-dns-lookup-zone-cost"></a>託管反向 DNS 對應區域的費用如何？

在 Azure DNS 中為 ISP 指派的 IP 區塊託管反向 DNS 對應區域，會依[標準 Azure DNS 費率](https://azure.microsoft.com/pricing/details/dns/)計價。

### <a name="can-i-host-reverse-dns-lookup-zones-for-both-ipv4-and-ipv6-addresses-in-azure-dns"></a>我可以同時在 Azure DNS 中為 IPv4 和 IPv6 位址託管反向 DNS 對應區域嗎？

是。 本文說明如何在 Azure DNS 中同時建立 IPv4 和 IPv6 反向 DNS 對應區域。

### <a name="can-i-import-an-existing-reverse-dns-lookup-zone"></a>可以匯入現有的反向 DNS 對應區域嗎？

是。 您可以使用 Azure CLI 將現有的 DNS 區域匯入 Azure DNS 中。 這個方法適用於正向對應區域和反向對應區域。

如需詳細資訊，請參閱[使用 Azure CLI 匯入及匯出 DNS 區域檔案](dns-import-export.md)。

## <a name="next-steps"></a>後續步驟

如需反向 DNS 的詳細資訊，請參閱維基百科的 [reverse DNS lookup](http://en.wikipedia.org/wiki/Reverse_DNS_lookup) (反向 DNS 對應)。
<br>
了解如何[管理 Azure 服務的反向 DNS 記錄](dns-reverse-dns-for-azure-services.md)。
