---
title: 使用 Azure PowerShell 管理 Azure DNS 中的 DNS 記錄 | Microsoft Docs
description: 將網域裝載於 Azure DNS 時，在 Azure DNS 管理 DNS 記錄集和記錄。 對記錄集和記錄執行作業的所有 PowerShell 命令。
services: dns
documentationcenter: na
author: vhorne
manager: timlt
ms.assetid: 7136a373-0682-471c-9c28-9e00d2add9c2
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: H1Hack27Feb2017
ms.workload: infrastructure-services
ms.date: 12/21/2016
ms.author: victorh
ms.openlocfilehash: b89b7885989a5e93d3d292e5cdcff733fed657af
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990173"
---
# <a name="manage-dns-records-and-recordsets-in-azure-dns-using-azure-powershell"></a>使用 Azure PowerShell 管理 Azure DNS 中的 DNS 記錄和記錄集

> [!div class="op_single_selector"]
> * [Azure 入口網站](dns-operations-recordsets-portal.md)
> * [Azure 傳統 CLI](dns-operations-recordsets-cli-nodejs.md)
> * [Azure CLI](dns-operations-recordsets-cli.md)
> * [PowerShell](dns-operations-recordsets.md)

本文說明如何使用 Azure PowerShell 來管理 DNS 區域的 DNS 記錄。 也可以使用跨平台 [Azure CLI](dns-operations-recordsets-cli.md) 或 [Azure 入口網站](dns-operations-recordsets-portal.md)來管理 DNS 記錄。

此文章中的範例假設您已[安裝 Azure PowerShell、登入，並建立 DNS 區域](dns-operations-dnszones.md)。

## <a name="introduction"></a>簡介

在 Azure DNS 中建立 DNS 記錄前，您需要先了解 Azure DNS 如何將 DNS 記錄組織成 DNS 記錄集。

[!INCLUDE [dns-about-records-include](../../includes/dns-about-records-include.md)]

如需在 Azure DNS 的 DNS 記錄的詳細資訊，請參閱 [DNS 區域與記錄](dns-zones-records.md)。


## <a name="create-a-new-dns-record"></a>建立新的 DNS 區域

如果新記錄的名稱和類型與現有記錄相同，您需要[將它新增至現有的記錄集](#add-a-record-to-an-existing-record-set)。 否則，如果新記錄的名稱和類型與現有記錄不同，則必須建立新的記錄集。 

### <a name="create-a-records-in-a-new-record-set"></a>在新的記錄集中建立 'A' 記錄

您可以使用 `New-AzureRmDnsRecordSet` Cmdlet 來建立記錄集。 建立記錄集時，您必須指定記錄集名稱、區域、存留時間 (TTL) 和記錄類型，與要建立的記錄。

將記錄加入至記錄集的參數，根據記錄集的類型而所有不同。 例如，使用 'A' 類型的記錄集時，您需要使用參數 `-IPv4Address` 來指定 IP 位址。 若是其他記錄類型，則使用其他變數。 請參閱[其他記錄類型範例](#additional-record-type-examples)，以了解詳細資訊。

下列範例會在 DNS 區域 'contoso.com' 中建立具有相對名稱 'www' 的記錄集。 記錄集的完整名稱是 'www.contoso.com'。 記錄類型為 'A'，且 TTL 為 3600 秒。 此記錄集都會包含一筆記錄，其中 IP 位址為 '1.2.3.4'。

```powershell
New-AzureRmDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -IPv4Address "1.2.3.4") 
```

若要在區域頂點 (在此案例中為 'contoso.com') 建立記錄集，請使用記錄集名稱 '\@' (不包括引號)：

```powershell
New-AzureRmDnsRecordSet -Name "@" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -IPv4Address "1.2.3.4") 
```

如果您需要建立包含多筆記錄的記錄集，請先建立本機陣列並新增記錄，然後將陣列傳遞給 `New-AzureRmDnsRecordSet`，如下所示：

```powershell
$aRecords = @()
$aRecords += New-AzureRmDnsRecordConfig -IPv4Address "1.2.3.4"
$aRecords += New-AzureRmDnsRecordConfig -IPv4Address "2.3.4.5"
New-AzureRmDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName MyResourceGroup -Ttl 3600 -RecordType A -DnsRecords $aRecords
```

[記錄集中繼資料](dns-zones-records.md#tags-and-metadata)可用來將應用程式特定資料與每一個資料集產生關聯 (以索引鍵值組的形式)。 下列範例示範如何建立具有兩個中繼資料項目 ('dept=finance' 與 'environment=production') 的記錄集。

```powershell
New-AzureRmDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -IPv4Address "1.2.3.4") -Metadata @{ dept="finance"; environment="production" } 
```

Azure DNS 也支援 'empty' 記錄集，此可作為建立 DNS 記錄前保留 DNS 名稱的預留位置。 空記錄集會顯示在 Azure DNS 控制面板中，但不會出現在 Azure DNS 名稱伺服器上。 下列範例會建立空記錄集：

```powershell
New-AzureRmDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords @()
```

## <a name="create-records-of-other-types"></a>建立其他類型的記錄

參閱如何建立 'A' 記錄的詳細資訊後，下列範例會示範如何建立 Azure DNS 所支援其他記錄類型的記錄。

在每個案例中，我們會說明如何建立包含單一記錄的記錄集。 可將 'A' 記錄的先前範例加以調整，即可建立含多個記錄的其他類型記錄集、具中繼資料的其他類型記錄集，或建立空的記錄集。

我們沒有提供 SOA 記錄集的建立範例，因為已與每一個 DNS 區域完成 SOA 建立與刪除，且無法個別建立或刪除 SOA。 然而，[可以對 SOA 進行修改，如稍後範例所示](#to-modify-an-SOA-record)。

### <a name="create-an-aaaa-record-set-with-a-single-record"></a>建立含有單一記錄的 AAAA 記錄集

```powershell
New-AzureRmDnsRecordSet -Name "test-aaaa" -RecordType AAAA -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -Ipv6Address "2607:f8b0:4009:1803::1005") 
```

### <a name="create-a-caa-record-set-with-a-single-record"></a>建立含有單一記錄的 CAA 記錄集

```powershell
New-AzureRmDnsRecordSet -Name "test-caa" -RecordType CAA -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -Caaflags 0 -CaaTag "issue" -CaaValue "ca1.contoso.com") 
```

### <a name="create-a-cname-record-set-with-a-single-record"></a>建立含有單一記錄的 CNAME 記錄集

> [!NOTE]
> DNS 標準在區域頂點不允許 CNAME 記錄 (`-Name '@'`)，也不允許包含一個記錄以上的記錄集。
> 
> 如需詳細資訊，請參閱 [CNAME 記錄](dns-zones-records.md#cname-records)。


```powershell
New-AzureRmDnsRecordSet -Name "test-cname" -RecordType CNAME -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -Cname "www.contoso.com") 
```

### <a name="create-an-mx-record-set-with-a-single-record"></a>建立含有單一記錄的 MX 記錄集

在此範例中，我們使用記錄集名稱 '\@' 在區域頂點建立 MX 記錄 (在此案例中為 'contoso.com')。


```powershell
New-AzureRmDnsRecordSet -Name "@" -RecordType MX -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -Exchange "mail.contoso.com" -Preference 5) 
```

### <a name="create-an-ns-record-set-with-a-single-record"></a>建立含有單一記錄的 NS 記錄集

```powershell
New-AzureRmDnsRecordSet -Name "test-ns" -RecordType NS -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -Nsdname "ns1.contoso.com") 
```

### <a name="create-a-ptr-record-set-with-a-single-record"></a>建立含有單一記錄的 PTR 記錄集

在此情況下，'my-arpa-zone.com' 代表表示您 IP 範圍的 ARPA 反向對應區域。 此區域中的每個 PTR 記錄集都與此 IP 範圍內的一個 IP 位址相對應。 記錄名稱 '10' 是此記錄所代表的這個 IP 範圍內 IP 位址的最後一個八位元。

```powershell
New-AzureRmDnsRecordSet -Name 10 -RecordType PTR -ZoneName "my-arpa-zone.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -Ptrdname "myservice.contoso.com") 
```

### <a name="create-an-srv-record-set-with-a-single-record"></a>建立含有單一記錄的 SRV 記錄集

建立 [SRV 記錄集](dns-zones-records.md#srv-records)時，指定記錄集名稱中的 *\_服務* 和 *\_通訊協定*。 在區域頂點建立一筆 SRV 記錄集時，不需要在記錄集名稱中包含 '\@'。

```powershell
New-AzureRmDnsRecordSet -Name "_sip._tls" -RecordType SRV -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -Priority 0 -Weight 5 -Port 8080 -Target "sip.contoso.com") 
```


### <a name="create-a-txt-record-set-with-a-single-record"></a>建立含有單一記錄的 TXT 記錄集

下列範例示範如何建立 TXT 記錄。 如需 TXT 記錄中，所支援字串長度上限的相關資訊，請參閱 [TXT 記錄](dns-zones-records.md#txt-records)。

```powershell
New-AzureRmDnsRecordSet -Name "test-txt" -RecordType TXT -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -Value "This is a TXT record") 
```


## <a name="get-a-record-set"></a>取得記錄集

若要擷取現有的記錄集，使用 `Get-AzureRmDnsRecordSet`。 此 cmdlet 會傳回 Azure DNS 中代表記錄集的本機物件。

如同 `New-AzureRmDnsRecordSet`，提供的記錄集名稱必須是*相對*名稱，表示它不能包含區域名稱。 您也必須指定記錄類型，以及包含記錄集的區域。

下列範例示範如何擷取記錄集。 在此範例中，使用 `-ZoneName` 和 `-ResourceGroupName` 參數來指定區域。

```powershell
$rs = Get-AzureRmDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

或者，您也可以使用 `-Zone` 參數傳遞的區域物件來指定區域。

```powershell
$zone = Get-AzureRmDnsZone -Name "contoso.com" -ResourceGroupName "MyResourceGroup"
$rs = Get-AzureRmDnsRecordSet -Name "www" -RecordType A -Zone $zone
```

## <a name="list-record-sets"></a>列出記錄集

您也可以使用 `Get-AzureRmDnsZone`，透過省略 `-Name` 及/或 `-RecordType` 參數，在區域中列出記錄集。

下列範例會傳回區域中的所有記錄集︰

```powershell
$recordsets = Get-AzureRmDnsRecordSet -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

下列範例示範將記錄集名稱省略時，如何透過指定記錄類型來擷取指定類型的所有記錄集：

```powershell
$recordsets = Get-AzureRmDnsRecordSet -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

若要擷取所有記錄類型中，具有指定名稱的所有記錄集，您必須擷取所有記錄集，然後篩選結果︰

```powershell
$recordsets = Get-AzureRmDnsRecordSet -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" | where {$_.Name.Equals("www")}
```

在以上所有範例中，您可以使用 `-ZoneName` 和 `-ResourceGroupName` 參數來指定區域 (如上所示)，或透過指定區域物件來指定：

```powershell
$zone = Get-AzureRmDnsZone -Name "contoso.com" -ResourceGroupName "MyResourceGroup"
$recordsets = Get-AzureRmDnsRecordSet -Zone $zone
```

## <a name="add-a-record-to-an-existing-record-set"></a>將記錄新增至現有的記錄集

若要將記錄新增至現有的記錄集，請遵循下列三個步驟︰

1. 取得現有記錄集

    ```powershell
    $rs = Get-AzureRmDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A
    ```

2. 將新記錄新增至本機記錄集。 這是一種離線作業。

    ```powershell
    Add-AzureRmDnsRecordConfig -RecordSet $rs -Ipv4Address "5.6.7.8"
    ```

3. 將對變更的認可傳回給 Azure DNS 服務。 

    ```powershell
    Set-AzureRmDnsRecordSet -RecordSet $rs
    ```

使用 `Set-AzureRmDnsRecordSet`，用指定的記錄集*取代* Azure DNS 中現有的記錄集 (與其中所包含的所有記錄)。 [Etag 檢查 ](dns-zones-records.md#etags) 是用來確保並行變更不會遭到覆寫。 您可以使用選擇性的 `-Overwrite` 參數來停用這些檢查。

作業的此序列也可以「經由管道輸送」，亦即使用管道傳遞記錄集物件，而不是以參數進行傳遞。

```powershell
Get-AzureRmDnsRecordSet -Name "www" –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A | Add-AzureRmDnsRecordConfig -Ipv4Address "5.6.7.8" | Set-AzureRmDnsRecordSet
```

上述範例顯示如何將 'A' 記錄新增至類型 'A' 的現有記錄集。 作業的類似序列也可用來將記錄新增至其他類型的記錄集，可透過每一個記錄類型專屬的其他變數取代 `Add-AzureRmDnsRecordConfig` 的 `-Ipv4Address` 變數。 每個記錄類型的參數對 `New-AzureRmDnsRecordConfig`cmdlet 而言是相同的，如以上[其他記錄類型範例](#additional-record-type-examples)所示。

類型 'CNAME' 或 'SOA' 的記錄集不能包含多個記錄。 這個條件是起因於 DNS 標準。 而非 Azure DNS 的限制。

## <a name="remove-a-record-from-an-existing-record-set"></a>從現有的記錄集移除記錄

從記錄集移除記錄的程序，與將記錄新增至現有記錄集的程序類似︰

1. 取得現有記錄集

    ```powershell
    $rs = Get-AzureRmDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A
    ```

2. 從本機記錄集物件中移除記錄。 這是一種離線作業。 要移除的記錄必須完全符合現有的資料錄，包括所有參數。

    ```powershell
    Remove-AzureRmDnsRecordConfig -RecordSet $rs -Ipv4Address "5.6.7.8"
    ```

3. 將對變更的認可傳回給 Azure DNS 服務。 使用選擇性 `-Overwrite` 參數來停止對並行變更的 [Etag 檢查](dns-zones-records.md#etags)。

    ```powershell
    Set-AzureRmDnsRecordSet -RecordSet $Rs
    ```

使用上述序列，將最後一筆記錄從記錄集中移除並不會刪除記錄集，而是會留下空的記錄集。 若要完全移除記錄集，請參閱[刪除記錄集](#delete-a-record-set)。

與將記錄新增至記錄集的方式類似，移除記錄集的作業序列也可以管道輸送︰

```powershell
Get-AzureRmDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A | Remove-AzureRmDnsRecordConfig -Ipv4Address "5.6.7.8" | Set-AzureRmDnsRecordSet
```

藉由將特定類型的適當參數傳遞至 `Remove-AzureRmDnsRecordSet`，以支援不同的記錄類型。 每個記錄類型的參數對 `New-AzureRmDnsRecordConfig`cmdlet 而言是相同的，如以上[其他記錄類型範例](#additional-record-type-examples)所示。


## <a name="modify-an-existing-record-set"></a>修改現有記錄集

修改現有記錄集的步驟，與從記錄集新增或移除記錄時所採取的步驟相類似：

1. 使用 `Get-AzureRmDnsRecordSet`擷取現有的記錄集。
2. 透過下列方式修改本機記錄集︰
    * 新增或移除記錄
    * 變更現有記錄的參數
    * 變更記錄集中繼資料和存留時間 (TTL)
3. 使用 `Set-AzureRmDnsRecordSet` Cmdlet 來認可您所做的變更。 這會以指定的記錄集*取代* Azure DNS 中現有的記錄集。

使用 `Set-AzureRmDnsRecordSet` 時，會使用 [Etag 檢查](dns-zones-records.md#etags) 來確保不會覆寫並行變更。 您可以使用選擇性的 `-Overwrite` 參數來停用這些檢查。

### <a name="to-update-a-record-in-an-existing-record-set"></a>更新現有記錄集中的記錄

在此範例中，我們會變更現有 'A' 記錄的 IP 位址：

```powershell
$rs = Get-AzureRmDnsRecordSet -name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
$rs.Records[0].Ipv4Address = "9.8.7.6"
Set-AzureRmDnsRecordSet -RecordSet $rs
```

### <a name="to-modify-an-soa-record"></a>修改 SOA 記錄

您無法在區域頂點 (`-Name "@"` (包含引號)) 從自動建立的 SOA 記錄集中新增或移除記錄。 不過，您可以修改 SOA 記錄 (「主機」除外) 和記錄集 TTL 內的任何參數。

下列範例示範如何變更 SOA 記錄的 *Email* 屬性：

```powershell
$rs = Get-AzureRmDnsRecordSet -Name "@" -RecordType SOA -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
$rs.Records[0].Email = "admin.contoso.com"
Set-AzureRmDnsRecordSet -RecordSet $rs
```

### <a name="to-modify-ns-records-at-the-zone-apex"></a>在區域頂點修改 NS 記錄

系統會自動使用每個 DNS 區域在區域頂點建立 NS 記錄集。 此記錄集包含指派給區域的 Azure DNS 名稱伺服器的名稱。

您可以將其他名稱伺服器新增至此 NS 記錄集，以支援使用多個 DNS 提供者的共同裝載網域。 您也可以修改此記錄集的 TTL 和中繼資料。 不過，您無法移除或修改預先填入的 Azure DNS 名稱伺服器。

請注意，這只適用於區域頂點的 NS 記錄集。 區域中的其他 NS 記錄集 (如用於委派子區域) 可以修改，沒有任何限制。

下列範例顯示如何將其他的名稱伺服器新增至在區域頂點的 NS 記錄集：

```powershell
$rs = Get-AzureRmDnsRecordSet -Name "@" -RecordType NS -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
Add-AzureRmDnsRecordConfig -RecordSet $rs -Nsdname ns1.myotherdnsprovider.com
Set-AzureRmDnsRecordSet -RecordSet $rs
```

### <a name="to-modify-record-set-metadata"></a>若要修改記錄集中繼資料

[記錄集中繼資料](dns-zones-records.md#tags-and-metadata)可用來將應用程式特定資料與每一個資料集產生關聯 (以索引鍵值組的形式)。

下列範例示範如何修改現有記錄集的中繼資料︰

```powershell
# Get the record set
$rs = Get-AzureRmDnsRecordSet -Name www -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"

# Add 'dept=finance' name-value pair
$rs.Metadata.Add('dept', 'finance') 

# Remove metadata item named 'environment'
$rs.Metadata.Remove('environment')  

# Commit changes
Set-AzureRmDnsRecordSet -RecordSet $rs
```


## <a name="delete-a-record-set"></a>刪除記錄集

您可以使用 `Remove-AzureRmDnsRecordSet` Cmdlet 來刪除記錄集。 刪除記錄集時，也會刪除記錄集內的所有記錄。

> [!NOTE]
> 您無法在區域頂點 (`-Name '@'`) 刪除 SOA 和 NS 記錄集。  Azure DNS 在區域建立時就自動建立了這些項目，並且會在區域刪除時自動刪除這些項目。

下列範例說明如何刪除記錄集。 在此範例中，記錄集名稱、記錄集類型、區域名稱和資源群組是個別明確指定。

```powershell
Remove-AzureRmDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

或者，可以透過名稱、類型，和使用物件指定的區域，來指定記錄集︰

```powershell
$zone = Get-AzureRmDnsZone -Name "contoso.com" -ResourceGroupName "MyResourceGroup"
Remove-AzureRmDnsRecordSet -Name "www" -RecordType A -Zone $zone
```

至於第三個選項，記錄集本身即可透過記錄集物件進行指定︰

```powershell
$rs = Get-AzureRmDnsRecordSet -Name www -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
Remove-AzureRmDnsRecordSet -RecordSet $rs
```

將記錄集的刪除方式指定為使用記錄集物件時，則會使用 [Etag 檢查](dns-zones-records.md#etags)，來確保不會刪除並行變更。 您可以使用選擇性的 `-Overwrite` 參數來停用這些檢查。

記錄集物件也可以經由管道輸送，而不是當做參數傳遞：

```powershell
Get-AzureRmDnsRecordSet -Name www -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" | Remove-AzureRmDnsRecordSet
```

## <a name="confirmation-prompts"></a>確認提示

`New-AzureRmDnsRecordSet`、`Set-AzureRmDnsRecordSet` 和 `Remove-AzureRmDnsRecordSet` cmdlets 皆支援確認提示。

如果 `$ConfirmPreference`PowerShell 喜好設定變數的值為 `Medium` 或更低，則每個 cmdlet 會提示確認。 因為 `$ConfirmPreference` 的預設值為 `High`，使用預設 PowerShell 設定時不會出現這些提示。

您可以使用 `-Confirm` 參數覆寫目前 `$ConfirmPreference` 設定。 如果您指定 `-Confirm` 或 `-Confirm:$True`，此 cmdlet 在執行前會提示您進行確認。 如果您指定 `-Confirm:$False`，此 cmdlet 不會提示您進行確認。 

如需 `-Confirm` 和 `$ConfirmPreference` 的詳細資訊，請參閱[有關喜好設定變數](https://msdn.microsoft.com/powershell/reference/5.1/Microsoft.PowerShell.Core/about/about_Preference_Variables)。

## <a name="next-steps"></a>後續步驟

深入了解[ Azure DNS 中的區域和記錄](dns-zones-records.md)。
<br>
了解使用 Azure DNS 時，如何[保護區域和記錄](dns-protect-zones-recordsets.md)。
<br>
檢閱 [Azure DNS PowerShell 參考文件](/powershell/module/azurerm.dns)。
