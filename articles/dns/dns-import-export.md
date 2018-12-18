---
title: 使用 Azure CLI 將網域區域檔案匯入和匯出至 Azure DNS | Microsoft Docs
description: 了解如何使用 Azure CLI 匯入和匯出 DNS 區域檔案至 Azure DNS
services: dns
documentationcenter: na
author: vhorne
manager: timlt
ms.assetid: f5797782-3005-4663-a488-ac0089809010
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/30/2018
ms.author: victorh
ms.openlocfilehash: 5afb607f0410b428d8e67fdff043a4e376dd60a5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956348"
---
# <a name="import-and-export-a-dns-zone-file-using-the-azure-cli"></a>使用 Azure CLI 匯入及匯出 DNS 區域檔案 

本文會引導您了解如何使用 Azure CLI 來匯入和匯出適用於 Azure DNS 的 DNS 區域檔案。

## <a name="introduction-to-dns-zone-migration"></a>DNS 區域移轉簡介

DNS 區域檔案是一個文字檔，其中包含區域中每筆網域名稱系統 (DNS) 記錄的詳細資料。 它會遵循標準格式，使其適合於在 DNS 系統之間傳送 DNS 記錄。 使用區域檔案是從 Azure DNS 移入或移出 DNS 區域的快速、可靠又方便的方法。

Azure DNS 支援使用 Azure 命令列介面 (CLI) 匯入和匯出區域檔案。 目前並「不」支援透過 Azure PowerShell 或 Azure 入口網站進行區域檔案匯入。

Azure CLI 是用來管理 Azure 服務的跨平台命令列工具。 它可從 [Azure 下載頁面](https://azure.microsoft.com/downloads/)取得，且適用於 Windows、Mac 及 Linux 平台。 跨平台支援對於匯入和匯出區域檔案十分重要，因為最常見的名稱伺服器軟體 [BIND](https://www.isc.org/downloads/bind/) 通常會在 Linux 上執行。


## <a name="obtain-your-existing-dns-zone-file"></a>取得現有的 DNS 區域檔案

將 DNS 區域檔案匯入 Azure DNS 之前，您必須取得區域檔案的複本。 此檔案的來源取決於目前裝載 DNS 區域的位置。

* 如果 DNS 區域是由合作夥伴服務 (例如網域註冊機構、專用 DNS 主機服務提供者或其他雲端提供者) 託管，該服務應提供下載 DNS 區域檔案的能力。
* 如果 DNS 區域託管在 Windows DNS 上，區域檔案的預設資料夾是 **%systemroot%\system32\dns**。 DNS 主控台的 [一般] 索引標籤上，也會顯示每個區域檔案的完整路徑。
* 如果使用 BIND 裝載 DNS 區域，則 BIND 組態檔 **named.conf**中會指定每個區域的區域檔案位置。

> [!NOTE]
> 下載自 GoDaddy 的區域檔案會有些微非標準格式。 您必須先修正格式，再將這些區域檔案匯入 Azure DNS。
>
> 每個 DNS 記錄 RDATA 中的 DNS 名稱會指定為完整名稱，但是結尾沒有「.」。這表示其他 DNS 系統會將這些名稱解譯為相對名稱。 將區域檔案匯入 Azure DNS 之前，您必須先加以編輯，以將結尾的 "." 附加至這些名稱。
>
> 例如，CNAME 記錄「www 3600 IN CNAME contoso.com」應變更為「www 3600 IN CNAME contoso.com.」。
> (結尾附加「.」)。

## <a name="import-a-dns-zone-file-into-azure-dns"></a>將 DNS 區域檔案匯入 Azure DNS

匯入區域檔案會在 Azure DNS 中建立新區域 (如果區域不存在)。 如果區域已經存在，則區域檔案中的記錄集必須與現有的記錄集合併。

### <a name="merge-behavior"></a>合併行為

* 預設會合併現有和新的記錄集。 合併的資料錄集內相同的記錄會進行重複資料刪除。
* 合併記錄集時，會使用預先存在之記錄集的存留時間 (TTL)。
* 起始授權 (SOA) 參數 (`host` 除外) 一律取自匯入的區域檔案。 同樣地，對於位於區域頂點的名稱伺服器記錄集，TTL 一律取自匯入的區域檔案。
* 匯入的 CNAME 記錄不會取代具有相同名稱的現有 CNAME 記錄。  
* 當 CNAME 記錄與另一筆同名但不同類型的記錄 (不論何者是現有或新的記錄) 之間發生衝突時，會保留現有的記錄。 

### <a name="additional-information-about-importing"></a>匯入的其他資訊

下列幾點提供有關區域匯入程序的其他技術詳細資訊。

* `$TTL` 指示詞為選擇性並受到支援。 若未提供 `$TTL` 指示詞，會匯入沒有明確 TTL 的記錄，並設定為預設 TTL 3600 秒。 如果相同資料錄集中有兩筆記錄指定不同的 TTL，則會使用較低的值。
* `$ORIGIN` 指示詞為選擇性並受到支援。 若未設定 `$ORIGIN` ，則使用的預設值是在命令列上指定的區域名稱 (加上結尾的 ".")。
* `$INCLUDE` 和 `$GENERATE` 指示詞不受支援。
* 支援這些記錄類型：A、AAAA、CNAME、MX、NS、SOA、SRV 和 TXT。
* Azure DNS 會在建立區域時，自動建立 SOA 記錄。 當您匯入區域檔案時，所有 SOA 參數都會取自該區域檔案，但 `host` 參數*除外*。 這個參數會使用 Azure DNS 所提供的值。 這是因為此參數必須參照 Azure DNS 所提供的主要名稱伺服器。
* Azure DNS 也會在建立區域時，自動建立位於區域頂點的名稱伺服器記錄集。 只會匯入此記錄集的 TTL。 這些記錄包含 Azure DNS 所提供的名稱伺服器名稱。 所匯入區域檔案中包含的值不會覆寫記錄資料。
* 在公開預覽期間，Azure DNS 僅支援單一字串 TXT 記錄。 Multistring TXT 記錄會串連起來並截斷為 255 個字元。

### <a name="cli-format-and-values"></a>CLI 格式和值

用來匯入 DNS 區域的 Azure CLI 命令格式為：

```azurecli
az network dns zone import -g <resource group> -n <zone name> -f <zone file name>
```

值：

* `<resource group>` 是 Azure DNS 中區域的資源群組名稱。
* `<zone name>` 是區域的名稱。
* `<zone file name>` 是要匯入之區域檔案的路徑/名稱。

如果資源群組中不存在具有此名稱的區域，則會為您建立。 如果區域已經存在，則匯入的記錄集會與現有的記錄集合併。 


### <a name="step-1-import-a-zone-file"></a>步驟 1. 匯入區域檔案

匯入 **contoso.com**區域的區域檔案。

1. 如果您還沒有 Resource Manager 資源群組，則必須建立一個。

    ```azurecli
    az group create --group myresourcegroup -l westeurope
    ```

2. 為了將 **contoso.com.txt** 檔案中的 **contoso.com** 區域匯入至 **myresourcegroup** 資源群組中的新 DNS 區域，您將會執行命令 `az network dns zone import`。<BR>此命令會載入並剖析該區域檔案。 此命令會在 Azure DNS 服務上執行一系列的命令，以建立區域和區域中的所有記錄集。 此命令會在主控台視窗中報告進度，以及任何的錯誤或警告。 由於記錄集是以系列方式建立，可能需要幾分鐘的時間來匯入大型的區域檔案。

    ```azurecli
    az network dns zone import -g myresourcegroup -n contoso.com -f contoso.com.txt
    ```

### <a name="step-2-verify-the-zone"></a>步驟 2. 確認區域

若要在匯入檔案之後確認 DNS 區域，您可以使用下列任何一個方法︰

* 您可以使用下列 Azure CLI 命令來列出記錄：

    ```azurecli
    az network dns record-set list -g myresourcegroup -z contoso.com
    ```

* 您可以使用 PowerShell Cmdlet `Get-AzureRmDnsRecordSet`來列出記錄。
* 您可以使用 `nslookup` 來驗證記錄的名稱解析。 因為尚未委派區域，所以您必須明確指定正確的 Azure DNS 名稱伺服器。 下列範例顯示如何擷取已指派給區域的名稱伺服器名稱。 這也會示範如何使用 `nslookup` 來查詢 "www" 記錄。

    ```azurecli
    az network dns record-set ns list -g myresourcegroup -z  --output json 
    ```

    ```json
    [
      {
       .......
       "name": "@",
        "nsRecords": [
          {
            "additionalProperties": {},
            "nsdname": "ns1-03.azure-dns.com."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns2-03.azure-dns.net."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns3-03.azure-dns.org."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns4-03.azure-dns.info."
          }
        ],
        "resourceGroup": "myresourcegroup",
        "ttl": 86400,
        "type": "Microsoft.Network/dnszones/NS"
      }
    ]
    ```

    ```cmd
    nslookup www.contoso.com ns1-03.azure-dns.com

        Server: ns1-01.azure-dns.com
        Address:  40.90.4.1

        Name:www.contoso.com
        Addresses:  134.170.185.46
        134.170.188.221
    ```

### <a name="step-3-update-dns-delegation"></a>步驟 3. 更新 DNS 委派

確認已正確匯入區域之後，必須更新 DNS 委派以指向 Azure DNS 名稱伺服器。 如需詳細資訊，請參閱 [更新 DNS 委派](dns-domain-delegation.md)。

## <a name="export-a-dns-zone-file-from-azure-dns"></a>從 Azure DNS 匯出 DNS 區域檔案

用來匯入 DNS 區域的 Azure CLI 命令格式為：

```azurecli
az network dns zone export -g <resource group> -n <zone name> -f <zone file name>
```

值：

* `<resource group>` 是 Azure DNS 中區域的資源群組名稱。
* `<zone name>` 是區域的名稱。
* `<zone file name>` 是要匯出之區域檔案的路徑/名稱。

和區域匯入時一樣，您必須先登入，選擇訂用帳戶，然後設定 Azure CLI 以使用資源管理員模式。

### <a name="to-export-a-zone-file"></a>匯出區域檔案

若要將 **myresourcegroup** 資源群組中的現有 Azure DNS 區域 **contoso.com** 匯出至 **contoso.com.txt** 檔案 (在目前資料夾中)，請執行 `azure network dns zone export`。 此命令會呼叫 Azure DNS 服務，以列舉區域中的記錄集，並將結果匯出至 BIND 相容的區域檔案。

    ```
    az network dns zone export -g myresourcegroup -n contoso.com -f contoso.com.txt
    ```
