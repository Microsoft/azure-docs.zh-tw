---
title: 針對邏輯組織標記 Azure 資源 | Microsoft Docs
description: 示範如何套用標籤以針對計費及管理來組織 Azure 資源。
services: azure-resource-manager
documentationcenter: ''
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 003a78e5-2ff8-4685-93b4-e94d6fb8ed5b
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: AzurePortal
ms.devlang: na
ms.topic: conceptual
ms.date: 08/10/2018
ms.author: tomfitz
ms.openlocfilehash: df9c218c275367852885e67ac2649926ba1d31d3
ms.sourcegitcommit: 17fe5fe119bdd82e011f8235283e599931fa671a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/11/2018
ms.locfileid: "42142081"
---
# <a name="use-tags-to-organize-your-azure-resources"></a>使用標記來組織 Azure 資源

[!INCLUDE [resource-manager-governance-tags](../../includes/resource-manager-governance-tags.md)]

若要將標記套用到資源，使用者必須具有該資源類型的寫入權限。 若要將標記套用到所有資源類型，請使用[參與者](../role-based-access-control/built-in-roles.md#contributor)角色。 若只要將標記套用到一個資源類型，則使用適用於該資源的參與者角色。 例如，若要將標記套用到虛擬機器，可使用[虛擬機器參與者](../role-based-access-control/built-in-roles.md#virtual-machine-contributor)。

[!INCLUDE [Handle personal data](../../includes/gdpr-intro-sentence.md)]

## <a name="powershell"></a>PowerShell

本文的範例需要 Azure PowerShell 6.0 版或更新版本。 如果版本不是 6.0 或更新版本，請[更新您的版本](/powershell/azure/install-azurerm-ps)。

若要查看*資源群組*的現有標籤，請使用：

```powershell
(Get-AzureRmResourceGroup -Name examplegroup).Tags
```

該指令碼會傳回下列格式︰

```powershell
Name                           Value
----                           -----
Dept                           IT
Environment                    Test
```

若要查看「具有指定之資源識別碼的資源」的現有標記，請使用：

```powershell
(Get-AzureRmResource -ResourceId /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Storage/storageAccounts/<storage-name>).Tags
```

或者，若要查看「具有指定之名稱和資源群組的資源」的現有標記，請使用：

```powershell
(Get-AzureRmResource -ResourceName examplevnet -ResourceGroupName examplegroup).Tags
```

若要取得「具有特定標記的資源群組」，請使用：

```powershell
(Get-AzureRmResourceGroup -Tag @{ Dept="Finance" }).ResourceGroupName
```

若要取得「具有特定標記的資源」，請使用：

```powershell
(Get-AzureRmResource -Tag @{ Dept="Finance"}).Name
```

若要取得「具有特定標記名稱的資源」，請使用：

```powershell
(Get-AzureRmResource -TagName Dept).Name
```

每當您將標記套用到資源或資源群組時，即會覆寫該資源或資源群組上現有的標記。 因此，您必須視該資源或資源群組是否具備現有標籤來使用不同的方法。

若要將標籤新增至*不具現有標籤的資源群組*，請使用：

```powershell
Set-AzureRmResourceGroup -Name examplegroup -Tag @{ Dept="IT"; Environment="Test" }
```

若要將標記新增至「具有現有標記的資源群組」，請擷取現有標記、新增標記，然後重新套用標記：

```powershell
$tags = (Get-AzureRmResourceGroup -Name examplegroup).Tags
$tags.Add("Status", "Approved")
Set-AzureRmResourceGroup -Tag $tags -Name examplegroup
```

若要將標籤新增至*不具現有標籤的資源*，請使用：

```powershell
$r = Get-AzureRmResource -ResourceName examplevnet -ResourceGroupName examplegroup
Set-AzureRmResource -Tag @{ Dept="IT"; Environment="Test" } -ResourceId $r.ResourceId -Force
```

若要將標記新增至「沒有現有標記的資源」，請使用：

```powershell
$r = Get-AzureRmResource -ResourceName examplevnet -ResourceGroupName examplegroup
$r.Tags.Add("Status", "Approved") 
Set-AzureRmResource -Tag $r.Tags -ResourceId $r.ResourceId -Force
```

若要將所有標籤從資源群組套用至其資源，而*不在資源上保留現有的標籤*，請使用下列指令碼︰

```powershell
$groups = Get-AzureRmResourceGroup
foreach ($g in $groups)
{
    Get-AzureRmResource -ResourceGroupName $g.ResourceGroupName | ForEach-Object {Set-AzureRmResource -ResourceId $_.ResourceId -Tag $g.Tags -Force }
}
```

若要將所有標籤從資源群組套用至其資源，並*在資源上保留所有非重複的現有標籤*，請使用下列指令碼︰

```powershell
$group = Get-AzureRmResourceGroup "examplegroup"
if ($group.Tags -ne $null) {
    $resources = Get-AzureRmResource -ResourceGroupName $group.ResourceGroupName
    foreach ($r in $resources)
    {
        $resourcetags = (Get-AzureRmResource -ResourceId $r.ResourceId).Tags
        if ($resourcetags)
        {
            foreach ($key in $group.Tags.Keys)
            {
                if (-not($resourcetags.ContainsKey($key)))
                {
                    $resourcetags.Add($key, $group.Tags[$key])
                }
            }
            Set-AzureRmResource -Tag $resourcetags -ResourceId $r.ResourceId -Force
        }
        else
        {
            Set-AzureRmResource -Tag $group.Tags -ResourceId $r.ResourceId -Force
        }
    }
}
```

若要移除所有標記，請傳遞空的雜湊表：

```powershell
Set-AzureRmResourceGroup -Tag @{} -Name examplegroup
```

## <a name="azure-cli"></a>Azure CLI

若要查看*資源群組*的現有標籤，請使用：

```azurecli
az group show -n examplegroup --query tags
```

該指令碼會傳回下列格式︰

```json
{
  "Dept"        : "IT",
  "Environment" : "Test"
}
```

或者，若要查看「具有指定之名稱、類型和資源群組的資源」的現有標籤，請使用：

```azurecli
az resource show -n examplevnet -g examplegroup --resource-type "Microsoft.Network/virtualNetworks" --query tags
```

對資源的集合執行迴圈時，您可能想要依據資源識別碼顯示資源。 本文稍後會示範完整範例。 若要查看「具有指定之資源識別碼的資源」的現有標記，請使用：

```azurecli
az resource show --id <resource-id> --query tags
```

若要取得「具有特定標籤的資源群組」，請使用 `az group list`：

```azurecli
az group list --tag Dept=IT
```

若要取得「具有特定標籤和值的所有資源」，請使用 `az resource list`：

```azurecli
az resource list --tag Dept=Finance
```

每當您將標記套用到資源或資源群組時，即會覆寫該資源或資源群組上現有的標記。 因此，您必須視該資源或資源群組是否具備現有標籤來使用不同的方法。

若要將標籤新增至*不具現有標籤的資源群組*，請使用：

```azurecli
az group update -n examplegroup --set tags.Environment=Test tags.Dept=IT
```

若要將標籤新增至*不具現有標籤的資源*，請使用：

```azurecli
az resource tag --tags Dept=IT Environment=Test -g examplegroup -n examplevnet --resource-type "Microsoft.Network/virtualNetworks"
```

若要將標籤新增至已經具有標籤的資源，請擷取現有的標籤、將該值重新格式化，然後重新套用現有和新的標籤： 

```azurecli
jsonrtag=$(az resource show -g examplegroup -n examplevnet --resource-type "Microsoft.Network/virtualNetworks" --query tags)
rt=$(echo $jsonrtag | tr -d '"{},' | sed 's/: /=/g')
az resource tag --tags $rt Project=Redesign -g examplegroup -n examplevnet --resource-type "Microsoft.Network/virtualNetworks"
```

若要將所有標籤從資源群組套用至其資源，而*不在資源上保留現有的標籤*，請使用下列指令碼︰

```azurecli
groups=$(az group list --query [].name --output tsv)
for rg in $groups
do
  jsontag=$(az group show -n $rg --query tags)
  t=$(echo $jsontag | tr -d '"{},' | sed 's/: /=/g')
  r=$(az resource list -g $rg --query [].id --output tsv)
  for resid in $r
  do
    az resource tag --tags $t --id $resid
  done
done
```

若要將所有標籤從資源群組套用至其資源，並*在資源上保留現有的標籤*，請使用下列指令碼：

```azurecli
groups=$(az group list --query [].name --output tsv)
for rg in $groups
do
  jsontag=$(az group show -n $rg --query tags)
  t=$(echo $jsontag | tr -d '"{},' | sed 's/: /=/g')
  r=$(az resource list -g $rg --query [].id --output tsv)
  for resid in $r
  do
    jsonrtag=$(az resource show --id $resid --query tags)
    rt=$(echo $jsonrtag | tr -d '"{},' | sed 's/: /=/g')
    az resource tag --tags $t$rt --id $resid
  done
done
```

## <a name="templates"></a>範本

[!INCLUDE [resource-manager-tags-in-templates](../../includes/resource-manager-tags-in-templates.md)]

## <a name="portal"></a>入口網站

[!INCLUDE [resource-manager-tag-resource](../../includes/resource-manager-tag-resources.md)]

## <a name="rest-api"></a>REST API

Azure 入口網站和 PowerShell 在幕後都使用 [Resource Manager REST API](https://docs.microsoft.com/rest/api/resources/) 。 如果您需要將標籤整合到另一個環境中，則可以在資源識別碼上利用 **GET** 來取得標籤，並使用 **PATCH** 呼叫來更新一組標籤。

## <a name="tags-and-billing"></a>標記與計費

您可以使用標籤將您的計費資料分組。 例如，如果您執行不同組織的多個 VM，您可以使用標籤根據成本中心將使用情況分組。 您也可以使用標籤來根據執行階段環境將成本分類。例如，在生產環境中執行之 VM 的計費使用量。

您可以透過 [Azure 資源使用狀況和 RateCard API](../billing/billing-usage-rate-card-overview.md) 或使用狀況逗號分隔值 (CSV) 檔案，擷取關於標記的資訊。 您可以從 [Azure 帳戶入口網站](https://account.windowsazure.com/)或 [EA 入口網站](https://ea.azure.com)下載使用量檔案。 如需以程式設計方式存取帳單資訊的詳細資訊，請參閱 [深入了解 Microsoft Azure 資源耗用量](../billing/billing-usage-rate-card-overview.md)。 若為 REST API 作業，請參閱 [Azure 計費 REST API 參考](https://msdn.microsoft.com/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c)。

當您下載服務 (支援附計費的標記) 的使用狀況 CSV 時，標記會出現在 [標記]  資料行。 如需詳細資訊，請參閱[了解 Microsoft Azure 帳單](../billing/billing-understand-your-bill.md)。

![查看計費中的標記](./media/resource-group-using-tags/billing_csv.png)

## <a name="next-steps"></a>後續步驟

* 您可以使用自訂原則，在訂用帳戶內套用限制和慣例。 您所定義的原則可能需要所有資源都具有特定標籤的值。 如需詳細資訊，請參閱[何謂 Azure 原則？](../azure-policy/azure-policy-introduction.md)
* 如需在部署資源時使用 Azure PowerShell 的簡介，請參閱 [搭配使用 Azure PowerShell 與 Azure Resource Manager](powershell-azure-resource-manager.md)。
* 如需在部署資源時使用 Azure CLI 的簡介，請參閱[使用適用於 Mac、Linux 和 Windows 的 Azure CLI 搭配 Azure Resource Manager](xplat-cli-azure-resource-manager.md)。
* 如需使用入口網站的簡介，請參閱[使用 Azure 入口網站來管理您的 Azure 資源](resource-group-portal.md)。  
* 如需關於企業如何使用 Resource Manager 有效地管理訂用帳戶的指引，請參閱 [Azure 企業 Scaffold - 規定的訂用帳戶治理](/azure/architecture/cloud-adoption-guide/subscription-governance)。
