---
title: 使用 Azure CLI 管理 Azure DNS 中的 DNS 記錄 | Microsoft Docs
description: 將網域裝載於 Azure DNS 時，在 Azure DNS 管理 DNS 記錄集和記錄。
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
ms.assetid: 5356a3a5-8dec-44ac-9709-0c2b707f6cb5
ms.service: dns
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: H1Hack27Feb2017
ms.workload: infrastructure-services
ms.date: 05/15/2018
ms.author: victorh
ms.openlocfilehash: 1f1ee4f69cc1ab656df04ed30cae6f4c3e55bfa7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46963810"
---
# <a name="manage-dns-records-and-recordsets-in-azure-dns-using-the-azure-cli"></a>使用 Azure CLI 管理 Azure DNS 中的 DNS 記錄和記錄集

> [!div class="op_single_selector"]
> * [Azure 入口網站](dns-operations-recordsets-portal.md)
> * [Azure CLI](dns-operations-recordsets-cli.md)
> * [PowerShell](dns-operations-recordsets.md)

本文適用於 Windows、Mac 和 Linux，將會說明如何使用跨平台 Azure CLI 管理 DNS 區域的 DNS 記錄。 您也可以使用 [Azure PowerShell](dns-operations-recordsets.md) 或 [Azure 入口網站](dns-operations-recordsets-portal.md)來管理 DNS 記錄。

此文章中的範例假設您已[安裝 Azure CLI、登入，並建立 DNS 區域](dns-operations-dnszones-cli.md)。

## <a name="introduction"></a>簡介

在 Azure DNS 中建立 DNS 記錄前，您需要先了解 Azure DNS 如何將 DNS 記錄組織成 DNS 記錄集。

[!INCLUDE [dns-about-records-include](../../includes/dns-about-records-include.md)]

如需在 Azure DNS 的 DNS 記錄的詳細資訊，請參閱 [DNS 區域與記錄](dns-zones-records.md)。

## <a name="create-a-dns-record"></a>建立 DNS 記錄

若要建立 DNS 記錄，請使用 `az network dns record-set <record-type> add-record` 命令 (其中 `<record-type>` 是記錄的類型，亦即 a、srv、txt 等等。)如需協助，請參閱 `az network dns record-set --help`。

建立記錄時，您必須指定資源群組名稱、區域名稱、記錄集名稱、記錄類型，以及所建立記錄的詳細資料。 提供的記錄集名稱必須是「相對」名稱，表示它不能包含區域名稱。

如果記錄集不存在，此命令會為您建立。 如果記錄集已經存在，此命令會將您指定的記錄新增至現有的記錄集。

如果建立新的記錄集，則會使用預設存留時間 (TTL) 3600。 如需如何使用不同 TTL 的指示，請參閱[建立 DNS 記錄集](#create-a-dns-record-set)。

下列範例會在 MyResourceGroup 資源群組的 contoso.com 區域中建立稱為 www 的 A 記錄。 A 記錄的 IP 位址是 1.2.3.4。

```azurecli
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 1.2.3.4
```

若要在區域頂點 (在此案例中為 "contoso.com") 建立記錄集，請使用記錄名稱 "\@" (包括引號)：

```azurecli
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --ipv4-address 1.2.3.4
```

## <a name="create-a-dns-record-set"></a>建立 DNS 記錄集

在上述範例中，DNS 記錄不是新增至現有記錄集，就是記錄集是以*隱含方式*建立。 您也可以先*明確地*建立記錄集，再於其中新增記錄。 Azure DNS 支援「空白」記錄集，其可做為預留位置，以在建立 DNS 記錄之前保留 DNS 名稱。 空白記錄集可在 Azure DNS 控制面板中看到，但不會出現在 Azure DNS 名稱伺服器上。

請使用 `az network dns record-set <record-type> create` 命令建立記錄集。 如需協助，請參閱 `az network dns record-set <record-type> create --help`。

明確地建立記錄集可讓您指定記錄集屬性，例如[存留時間 (TTL)](dns-zones-records.md#time-to-live) 和中繼資料。 [記錄集中繼資料](dns-zones-records.md#tags-and-metadata)可用來將應用程式特定資料與每一個資料集產生關聯 (以索引鍵值組的形式)。

下列範例使用 `--ttl` 參數 (簡短形式 `-l`)，建立類型「A」且 60 秒 TTL 的空白記錄集：

```azurecli
az network dns record-set a create --resource-group myresourcegroup --zone-name contoso.com --name www --ttl 60
```

下列範例使用 `--metadata` 參數建立具有兩個中繼資料項目 ("dept=finance" 和 "environment=production") 的記錄集：

```azurecli
az network dns record-set a create --resource-group myresourcegroup --zone-name contoso.com --name www --metadata "dept=finance" "environment=production"
```

建立好空白記錄集之後，可依[建立 DNS 記錄](#create-a-dns-record)所述使用 `azure network dns record-set <record-type> add-record` 新增記錄。

## <a name="create-records-of-other-types"></a>建立其他類型的記錄

參閱如何建立 'A' 記錄的詳細資訊後，下列範例會示範如何建立 Azure DNS 所支援其他記錄類型的記錄。

用來指定記錄資料的參數，根據記錄的類型而所有不同。 例如，對於類型 "A" 的記錄，您可使用參數 `--ipv4-address <IPv4 address>` 指定 IPv4 位址。 每個記錄類型的參數可以使用 `az network dns record-set <record-type> add-record --help` 列出。

在每個案例中，我們會說明如何建立單一記錄。 記錄會新增至現有記錄集，或者會以隱含方式建立記錄集。 如需明確建立記錄集和定義記錄集參數的詳細資訊，請參閱[建立 DNS 記錄集](#create-a-dns-record-set)。

我們沒有提供 SOA 記錄集的建立範例，因為已與每一個 DNS 區域完成 SOA 建立與刪除，且無法個別建立或刪除 SOA。 然而，[可以對 SOA 進行修改，如稍後範例所示](#to-modify-an-SOA-record)。

### <a name="create-an-aaaa-record"></a>建立 AAAA 記錄

```azurecli
az network dns record-set aaaa add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-aaaa --ipv6-address 2607:f8b0:4009:1803::1005
```

### <a name="create-an-caa-record"></a>建立 CAA 記錄

```azurecli
az network dns record-set caa add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-caa --flags 0 --tag "issue" --value "ca1.contoso.com"
```

### <a name="create-a-cname-record"></a>建立 CNAME 記錄

> [!NOTE]
> DNS 標準在區域頂點不允許 CNAME 記錄 (`--Name "@"`)，也不允許包含一個記錄以上的記錄集。
> 
> 如需詳細資訊，請參閱 [CNAME 記錄](dns-zones-records.md#cname-records)。

```azurecli
az network dns record-set cname set-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-cname --cname www.contoso.com
```

### <a name="create-an-mx-record"></a>建立 MX 記錄

此範例會使用記錄集名稱 "\@"，在區域頂點 (在此案例中，"contoso.com") 建立 MX 記錄。

```azurecli
az network dns record-set mx add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --exchange mail.contoso.com --preference 5
```

### <a name="create-an-ns-record"></a>建立 NS 記錄

```azurecli
az network dns record-set ns add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-ns --nsdname ns1.contoso.com
```

### <a name="create-a-ptr-record"></a>建立 PTR 記錄

在此情況下，'my-arpa-zone.com' 代表表示您 IP 範圍的 ARPA 區域。 此區域中的每個 PTR 記錄集都與此 IP 範圍內的一個 IP 位址相對應。  記錄名稱 '10' 是此記錄所代表的這個 IP 範圍內 IP 位址的最後一個八位元。

```azurecli
az network dns record-set ptr add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name my-arpa.zone.com --ptrdname myservice.contoso.com
```

### <a name="create-an-srv-record"></a>建立 SRV 記錄

建立 [SRV 記錄集](dns-zones-records.md#srv-records)時，指定記錄集名稱中的 *\_服務* 和 *\_通訊協定*。 在區域頂點建立一筆 SRV 記錄集時，不需要在記錄集名稱中包含 "\@"。

```azurecli
az network dns record-set srv add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name _sip._tls --priority 10 --weight 5 --port 8080 --target sip.contoso.com
```

### <a name="create-a-txt-record"></a>建立 TXT 記錄

下列範例示範如何建立 TXT 記錄。 如需 TXT 記錄中，所支援字串長度上限的相關資訊，請參閱 [TXT 記錄](dns-zones-records.md#txt-records)。

```azurecli
az network dns record-set txt add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-txt --value "This is a TXT record"
```

## <a name="get-a-record-set"></a>取得記錄集

若要擷取現有的記錄集，使用 `az network dns record-set <record-type> show`。 如需協助，請參閱 `az network dns record-set <record-type> show --help`。

和建立記錄或記錄集時相同，提供的記錄集名稱必須是「相對」名稱，表示它不能包含區域名稱。 您也必須指定記錄類型、包含記錄集的區域，以及包含區域的資源群組。

下列範例會從 MyResourceGroup 資源群組的 contoso.com 區域中，擷取類型為 A 的 www 記錄：

```azurecli
az network dns record-set a show --resource-group myresourcegroup --zone-name contoso.com --name www
```

## <a name="list-record-sets"></a>列出記錄集

您可以使用 `az network dns record-set list` 命令來列出 DNS 區域中的所有記錄。 如需協助，請參閱 `az network dns record-set list --help`。

這個範例會傳回資源群組 MyResourceGroup 之區域 contoso.com 中的所有記錄集，不論其名稱或記錄類型為何︰

```azurecli
az network dns record-set list --resource-group myresourcegroup --zone-name contoso.com
```

此範例會傳回符合指定記錄類型 (此案例中為 'A' 記錄) 的所有記錄集：

```azurecli
az network dns record-set a list --resource-group myresourcegroup --zone-name contoso.com 
```

## <a name="add-a-record-to-an-existing-record-set"></a>將記錄新增至現有的記錄集

您可以使用 `az network dns record-set <record-type> add-record` 在新的記錄集內建立記錄，或用它將記錄新增至現有記錄集。

如需詳細資訊，請參閱上面的[建立 DNS 記錄](#create-a-dns-record)和[建立其他類型的記錄](#create-records-of-other-types)。

## <a name="remove-a-record-from-an-existing-record-set"></a>從現有的記錄集移除記錄。

若要從現有記錄集內移除 DNS 記錄，請使用 `az network dns record-set <record-type> remove-record`。 如需協助，請參閱 `az network dns record-set <record-type> remove-record -h`。

此命令會刪除記錄集內的 DNS 記錄。 如果記錄集內的最後一個記錄遭到刪除，記錄集本身也會遭到刪除。 若要改為保留空白記錄集，請使用 `--keep-empty-record-set` 選項。

您必須使用和使用 `az network dns record-set <record-type> add-record` 建立記錄時相同的參數，指定要刪除的記錄和應從哪個區域中刪除。 這些參數在上面的[建立 DNS 記錄](#create-a-dns-record)和[建立其他類型的記錄](#create-records-of-other-types)中有所說明。

下列範例會在 MyResourceGroup 資源群組的 contoso.com 區域中，刪除名為 www 之記錄集內值為 '1.2.3.4' 的 A 記錄。

```azurecli
az network dns record-set a remove-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "www" --ipv4-address 1.2.3.4
```

## <a name="modify-an-existing-record-set"></a>修改現有記錄集

每個記錄集都包含[存留時間 (TTL)](dns-zones-records.md#time-to-live)、[中繼資料](dns-zones-records.md#tags-and-metadata)和 DNS 記錄。 下列各節說明如何修改每個屬性。

### <a name="to-modify-an-a-aaaa-caa-mx-ns-ptr-srv-or-txt-record"></a>修改 A、AAAA、CAA、MX、NS、PTR、SRV 或 TXT 記錄

若要修改類型為 A、AAAA、CAA、MX、NS、PTR、SRV 或 TXT 的現有記錄，您應該先新增記錄，再刪除現有記錄。 如需如何刪除和新增記錄的詳細指示，請參閱本文稍早的章節。

下列範例說明如何修改 'A' 記錄，從 IP 位址 1.2.3.4 到 IP 位址 5.6.7.8：

```azurecli
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 5.6.7.8
az network dns record-set a remove-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 1.2.3.4
```

您無法在區域頂點 (`--Name "@"` (包含引號)) 在自動建立的 NS 記錄集中新增、移除或修改記錄。 針對此記錄集，修改記錄集 TTL 與中繼資料是唯一允許的變更。

### <a name="to-modify-a-cname-record"></a>修改 CNAME 記錄

不同於大多數的其他記錄類型，CNAME 記錄集只能包含單一記錄。  因此就其他記錄類型而言，您無法透過新增新記錄與移除現有記錄，來取代目前的值。

相反地，若要修改 CNAME 記錄，請使用 `az network dns record-set cname set-record`。 如需說明，請參閱 `az network dns record-set cname set-record --help`

此範例會修改 MyResourceGroup 資源群組之 contoso.com 區域中的 CNAME 記錄集 www，使其指向 'www.fabrikam.net' 而非其現有值︰

```azurecli
az network dns record-set cname set-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-cname --cname www.fabrikam.net
``` 

### <a name="to-modify-an-soa-record"></a>修改 SOA 記錄

不同於大多數的其他記錄類型，CNAME 記錄集只能包含單一記錄。  因此就其他記錄類型而言，您無法透過新增新記錄與移除現有記錄，來取代目前的值。

相反地，若要修改 SOA 記錄，請使用 `az network dns record-set soa update`。 如需協助，請參閱 `az network dns record-set soa update --help`。

下列範例說明如何設定資源群組 MyResourceGroup 之區域 contoso.com 的 SOA 記錄 'email' 屬性：

```azurecli
az network dns record-set soa update --resource-group myresourcegroup --zone-name contoso.com --email admin.contoso.com
```

### <a name="to-modify-ns-records-at-the-zone-apex"></a>在區域頂點修改 NS 記錄

系統會自動使用每個 DNS 區域在區域頂點建立 NS 記錄集。 此記錄集包含指派給區域的 Azure DNS 名稱伺服器的名稱。

您可以將其他名稱伺服器新增至此 NS 記錄集，以支援使用多個 DNS 提供者的共同裝載網域。 您也可以修改此記錄集的 TTL 和中繼資料。 不過，您無法移除或修改預先填入的 Azure DNS 名稱伺服器。

請注意，這只適用於區域頂點的 NS 記錄集。 區域中的其他 NS 記錄集 (如用於委派子區域) 可以修改，沒有任何限制。

下列範例顯示如何將其他的名稱伺服器新增至在區域頂點的 NS 記錄集：

```azurecli
az network dns record-set ns add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --nsdname ns1.myotherdnsprovider.com 
```

### <a name="to-modify-the-ttl-of-an-existing-record-set"></a>修改現有記錄集的 TTL

若要修改現有記錄集的 TTL，請使用 `azure network dns record-set <record-type> update`。 如需協助，請參閱 `azure network dns record-set <record-type> update --help`。

下列範例說明如何修改記錄集 TTL，在此案例中是修改為 60 秒︰

```azurecli
az network dns record-set a update --resource-group myresourcegroup --zone-name contoso.com --name www --set ttl=60
```

### <a name="to-modify-the-metadata-of-an-existing-record-set"></a>修改現有記錄集的中繼資料

[記錄集中繼資料](dns-zones-records.md#tags-and-metadata)可用來將應用程式特定資料與每一個資料集產生關聯 (以索引鍵值組的形式)。 若要修改現有記錄集的中繼資料，請使用 `az network dns record-set <record-type> update`。 如需協助，請參閱 `az network dns record-set <record-type> update --help`。

下列範例示範如何修改具有兩個中繼資料項目 ("dept=finance" 與 "environment=production") 的記錄集。 請注意，任何現有的中繼資料都會由給定值所「取代」。

```azurecli
az network dns record-set a update --resource-group myresourcegroup --zone-name contoso.com --name www --set metadata.dept=finance metadata.environment=production
```

## <a name="delete-a-record-set"></a>刪除記錄集

您可以使用 `az network dns record-set <record-type> delete` 命令來刪除記錄集。 如需協助，請參閱 `azure network dns record-set <record-type> delete --help`。 刪除記錄集時，也會刪除記錄集內的所有記錄。

> [!NOTE]
> 您無法在區域頂點 (`--name "@"`) 刪除 SOA 和 NS 記錄集。  這些項目在區域建立時即會自動建立，且當區域刪除時則會自動刪除。

下列範例會從 MyResourceGroup 資源群組的 contoso.com 區域中，刪除類型為 A 的 www 記錄集：

```azurecli
az network dns record-set a delete --resource-group myresourcegroup --zone-name contoso.com --name www
```

系統會提示您確認刪除作業。 若要抑制此提示，請使用 `--yes` 參數。

## <a name="next-steps"></a>後續步驟

深入了解[ Azure DNS 中的區域和記錄](dns-zones-records.md)。
<br>
了解使用 Azure DNS 時，如何[保護區域和記錄](dns-protect-zones-recordsets.md)。
