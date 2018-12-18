---
title: Azure 服務的反向 DNS | Microsoft Docs
description: 了解如何設定 Azure 託管服務的反向 DNS 對應
services: dns
documentationcenter: na
author: vhorne
manager: timlt
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/29/2017
ms.author: victorh
ms.openlocfilehash: cbd1a7a3a797cc20be92583bbb5ac163333729fc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46969796"
---
# <a name="configure-reverse-dns-for-services-hosted-in-azure"></a>設定 Azure 託管服務的反向 DNS

本文說明如何設定 Azure 託管服務的反向 DNS 對應。

Azure 中的服務會使用由 Azure 指派並由 Microsoft 所擁有的 IP 位址。 這些反向 DNS 記錄 (PTR 記錄) 必須建立在對應的 Microsoft 擁有之反向 DNS 對應區域中。 本文說明如何執行此作業。

請勿混淆這種情況以及[為在 Azure DNS 中指派的 IP 範圍託管反向 DNS 對應區域](dns-reverse-dns-hosting.md)的功能。 本例中，反向對應區域所代表的 IP 範圍必須指派給您的組織，通常由您的 ISP 指派。

在閱讀本文之前，您應該先熟悉這篇 [Azure 反向 DNS 和支援概觀](dns-reverse-dns-overview.md)。

在 Azure DNS 中，是透過 PublicIpAddress 資源公開計算資源 (例如虛擬機器、虛擬機器擴展集或 Service Fabric 叢集)。 反向 DNS 對應是使用 PublicIpAddress 的 'ReverseFqdn' 屬性設定。


目前不支援 Azure App Service 反向 DNS。

## <a name="validation-of-reverse-dns-records"></a>反向 DNS 記錄的驗證

協力廠商應該無法為對應至您 DNS 網域的 Azure 服務建立反向 DNS 記錄。 為避免這個問題，Azure 只允許建立這種條件下的反向 DNS 記錄：反向 DNS 記錄中指定的網域名稱能解析為相同 Azure 訂用帳戶中之 PublicIpAddress 或雲端服務的 DNS 名稱或 IP 位址，或和它們相同。

只有設定或修改反向 DNS 記錄時，才會執行這項驗證。 不會執行定期的重新驗證。

例如：假設 PublicIpAddress 資源的 DNS 名稱為 contosoapp1.northus.cloudapp.azure.com，且 IP 位址為 23.96.52.53。 則可將 PublicIpAddress 的 ReverseFqdn 指定為：
* PublicIpAddress 的 DNS 名稱為 contosoapp1.northus.cloudapp.azure.com。
* 相同訂用帳戶中不同 PublicIpAddress 的 DNS 名稱，例如 contosoapp2.westus.cloudapp.azure.com。
* 虛名 DNS 名稱，例如 app1.contoso.com，只要這個名稱是「第一次」設定為 contosoapp1.northus.cloudapp.azure.com 的 CNAME，或相同訂用帳戶中不同 PublicIpAddress 的 CNAME。
* 虛名 DNS 名稱，例如 app1.contoso.com，只要這個名稱是「第一次」設定為 IP 位址 23.96.52.53 的 A 記錄，或相同訂用帳戶中不同 PublicIpAddress 之 IP 位址的 A 記錄。

相同的條件約束適用於雲端服務的反向 DNS。


## <a name="reverse-dns-for-publicipaddress-resources"></a>PublicIpAddress 資源的反向 DNS

本節會詳細說明如何在資源管理員部署模型中，使用 Azure PowerShell、Azure 傳統 CLI 或 Azure CLI 為 PublicIpAddress 資源設定反向 DNS。 目前不支援透過 Azure 入口網站為 PublicIpAddress 資源設定反向 DNS。

Azure 目前只支援 IPv4 PublicIpAddress 資源的反向 DNS。 其不支援 IPv6。

### <a name="add-reverse-dns-to-an-existing-publicipaddresses"></a>將反向 DNS 新增至現有的 PublicIpAddresses

#### <a name="powershell"></a>PowerShell

將反向 DNS 新增至現有的 PublicIpAddresses：

```powershell
$pip = Get-AzureRmPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
$pip.DnsSettings.ReverseFqdn = "contosoapp1.westus.cloudapp.azure.com."
Set-AzureRmPublicIpAddress -PublicIpAddress $pip
```

若要將反向 DNS 新增至尚未有 DNS 名稱的現有 PublicIpAddress，您也必須指定 DNS 名稱：

```powershell
$pip = Get-AzureRmPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
$pip.DnsSettings = New-Object -TypeName "Microsoft.Azure.Commands.Network.Models.PSPublicIpAddressDnsSettings"
$pip.DnsSettings.DomainNameLabel = "contosoapp1"
$pip.DnsSettings.ReverseFqdn = "contosoapp1.westus.cloudapp.azure.com."
Set-AzureRmPublicIpAddress -PublicIpAddress $pip
```

#### <a name="azure-classic-cli"></a>Azure 傳統 CLI

將反向 DNS 新增至現有的 PublicIpAddresses：

```azurecli
azure network public-ip set -n PublicIp -g MyResourceGroup -f contosoapp1.westus.cloudapp.azure.com.
```

若要將反向 DNS 新增至尚未有 DNS 名稱的現有 PublicIpAddress，您也必須指定 DNS 名稱：

```azurecli
azure network public-ip set -n PublicIp -g MyResourceGroup -d contosoapp1 -f contosoapp1.westus.cloudapp.azure.com.
```

#### <a name="azure-cli"></a>Azure CLI

將反向 DNS 新增至現有的 PublicIpAddresses：

```azurecli
az network public-ip update --resource-group MyResourceGroup --name PublicIp --reverse-fqdn contosoapp1.westus.cloudapp.azure.com.
```

若要將反向 DNS 新增至尚未有 DNS 名稱的現有 PublicIpAddress，您也必須指定 DNS 名稱：

```azurecli
az network public-ip update --resource-group MyResourceGroup --name PublicIp --reverse-fqdn contosoapp1.westus.cloudapp.azure.com --dns-name contosoapp1
```

### <a name="create-a-public-ip-address-with-reverse-dns"></a>建立具有反向 DNS 的公用 IP 位址

建立已指定反向 DNS 屬性的新 PublicIpAddress：

#### <a name="powershell"></a>PowerShell

```powershell
New-AzureRmPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup" -Location "WestUS" -AllocationMethod Dynamic -DomainNameLabel "contosoapp2" -ReverseFqdn "contosoapp2.westus.cloudapp.azure.com."
```

#### <a name="azure-classic-cli"></a>Azure 傳統 CLI

```azurecli
azure network public-ip create -n PublicIp -g MyResourceGroup -l westus -d contosoapp3 -f contosoapp3.westus.cloudapp.azure.com.
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network public-ip create --name PublicIp --resource-group MyResourceGroup --location westcentralus --dns-name contosoapp1 --reverse-fqdn contosoapp1.westcentralus.cloudapp.azure.com
```

### <a name="view-reverse-dns-for-an-existing-publicipaddress"></a>檢視現有 PublicIpAddresses 的反向 DNS

檢視現有 PublicIpAddress 已設定的值：

#### <a name="powershell"></a>PowerShell

```powershell
Get-AzureRmPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
```

#### <a name="azure-classic-cli"></a>Azure 傳統 CLI

```azurecli
azure network public-ip show -n PublicIp -g MyResourceGroup
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network public-ip show --name PublicIp --resource-group MyResourceGroup
```

### <a name="remove-reverse-dns-from-existing-public-ip-addresses"></a>移除現有公用 IP 位址的反向 DNS

從現有的 PublicIpAddress 移除反向 DNS 屬性：

#### <a name="powershell"></a>PowerShell

```powershell
$pip = Get-AzureRmPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
$pip.DnsSettings.ReverseFqdn = ""
Set-AzureRmPublicIpAddress -PublicIpAddress $pip
```

#### <a name="azure-classic-cli"></a>Azure 傳統 CLI

```azurecli
azure network public-ip set -n PublicIp -g MyResourceGroup –f ""
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network public-ip update --resource-group MyResourceGroup --name PublicIp --reverse-fqdn ""
```


## <a name="configure-reverse-dns-for-cloud-services"></a>設定雲端服務的反向 DNS

本節會詳細說明如何在傳統部署模型中，使用 Azure PowerShell 為雲端服務設定反向 DNS。 不支援透過 Azure 入口網站、Azure 傳統 CLI 或 Azure CLI，為雲端服務設定反向 DNS。

### <a name="add-reverse-dns-to-existing-cloud-services"></a>將反向 DNS 新增至現有的雲端服務

將反向 DNS 記錄新增至雲端服務：

```powershell
Set-AzureService –ServiceName "contosoapp1" –Description "App1 with Reverse DNS" –ReverseDnsFqdn "contosoapp1.cloudapp.net."
```

### <a name="create-a-cloud-service-with-reverse-dns"></a>建立具有反向 DNS 的雲端服務

建立已指定反向 DNS 屬性的新雲端服務：

```powershell
New-AzureService –ServiceName "contosoapp1" –Location "West US" –Description "App1 with Reverse DNS" –ReverseDnsFqdn "contosoapp1.cloudapp.net."
```

### <a name="view-reverse-dns-for-existing-cloud-services"></a>檢視現有雲端服務的反向 DNS

檢視現有雲端服務的反向 DNS 屬性：

```powershell
Get-AzureService "contosoapp1"
```

### <a name="remove-reverse-dns-from-existing-cloud-services"></a>移除現有雲端服務的反向 DNS

從現有的雲端服務移除反向 DNS 屬性：

```powershell
Set-AzureService –ServiceName "contosoapp1" –Description "App1 with Reverse DNS" –ReverseDnsFqdn ""
```

## <a name="faq"></a>常見問題集

### <a name="how-much-do-reverse-dns-records-cost"></a>反向 DNS 記錄的成本為何？

完全免費！  反向 DNS 記錄或查詢不需要額外成本。

### <a name="will-my-reverse-dns-records-resolve-from-the-internet"></a>我的反向 DNS 記錄會從網際網路解析嗎？

是。 在您為 Azure 服務設定反向 DNS 屬性之後，Azure 會管理所有必要的 DNS 委派和 DNS 區域，以確保反向 DNS 記錄可以為所有網際網路使用者解析。

### <a name="are-default-reverse-dns-records-created-for-my-azure-services"></a>我的 Azure 服務有沒有建立預設的反向 DNS 記錄？

否。 反向 DNS 是選用的功能。 如果您選擇不設定，則不會建立任何預設反向 DNS 記錄。

### <a name="what-is-the-format-for-the-fully-qualified-domain-name-fqdn"></a>完整網域名稱 (FQDN) 的格式為何？

FQDN 是以正向順序指定，且必須以點結束 (例如，"app1.contoso.com")。

### <a name="what-happens-if-the-validation-check-for-the-reverse-dns-ive-specified-fails"></a>如果我指定的反向 DNS 驗證檢查失敗，會發生什麼事？

當反向 DNS 驗證檢查失敗時，設定反向 DNS 記錄的作業也會失敗。 請依要求更正反向 DNS 值，然後再試一次。

### <a name="can-i-configure-reverse-dns-for-azure-app-service"></a>Azure App Service 可以設定反向 DNS 嗎？

否。 目前不支援 Azure App Service 反向 DNS。

### <a name="can-i-configure-multiple-reverse-dns-records-for-my-azure-service"></a>Azure 服務可以設定多個反向 DNS 記錄嗎？

否。 Azure 雲端服務或 PublicIpAddress，Azure 只支援一筆反向 DNS 記錄。

### <a name="can-i-configure-reverse-dns-for-ipv6-publicipaddress-resources"></a>IPv6 PublicIpAddress 資源可以設定反向 DNS嗎？

否。 Azure 目前只支援 IPv4 PublicIpAddress 資源和雲端服務的反向 DNS。

### <a name="can-i-send-emails-to-external-domains-from-my-azure-compute-services"></a>我可以從我的 Azure 計算服務將電子郵件傳送至外部網域嗎？

從 Azure 部署直接傳送電子郵件的技術功能，須視訂用帳戶的類型而定。 不論訂用帳戶的類型為何，Microsoft 均建議您使用信任的郵件轉送服務來傳送外寄郵件。 如需詳細資訊，請參閱 [Enhanced Azure Security for sending Emails – November 2017 Update (增強 Azure 傳送電子郵件的安全性 – 2017 年 11 月更新)](https://blogs.msdn.microsoft.com/mast/2017/11/15/enhanced-azure-security-for-sending-emails-november-2017-update/)。

## <a name="next-steps"></a>後續步驟

如需反向 DNS 的詳細資訊，請參閱維基百科的 [reverse DNS lookup](http://en.wikipedia.org/wiki/Reverse_DNS_lookup) (反向 DNS 對應)。
<br>
了解如何[在 Azure DNS 中為您 ISP 指派的 IP 範圍託管反向對應區域](dns-reverse-dns-for-azure-services.md)。

