---
title: 了解 Azure VM 擴展集 VM 的執行個體識別碼 | Microsoft Docs
description: 了解 Azure VM 擴展集 VM 的執行個體識別碼
services: virtual-machine-scale-sets
documentationcenter: ''
author: gatneil
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: e229664e-ee4e-4f12-9d2e-a4f456989e5d
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/22/2018
ms.author: negat
ms.openlocfilehash: 3a43dc86f1fb53dfde4bce3938faaa30e18f5a6d
ms.sourcegitcommit: 782d5955e1bec50a17d9366a8e2bf583559dca9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/02/2018
ms.locfileid: "29742889"
---
# <a name="understand-instance-ids-for-azure-vm-scale-set-vms"></a>了解 Azure VM 擴展集 VM 的執行個體識別碼
本文說明擴展集的執行個體識別碼和它們呈現的各種方式。

## <a name="scale-set-instance-ids"></a>擴展集執行個體識別碼

擴展集中的每個 VM 都會取得唯一識別本身的執行個體識別碼。 這個執行個體識別碼使用於擴展集 API 中，以在擴展集中的特定 VM 上執行作業。 例如，當您使用重新安裝映像 API 時，可以指定要重新安裝映像的特定執行個體識別碼：

REST API：`POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/reimage?api-version={apiVersion}` (如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesets/reimage) \(英文\))

Powershell：`Set-AzureRmVmssVM -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName} -InstanceId {instanceId} -Reimage` (如需詳細資訊，請參閱 [Powershell 文件](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmssvm) \(英文\))

CLI：`az vmss reimage -g {resourceGroupName} -n {vmScaleSetName} --instance-id {instanceId}` (如需詳細資訊，請參閱 [CLI 文件](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az_vmss_reimage) \(英文\))。

您可以列出擴展集中的所有執行個體，以取得執行個體識別碼的清單：

REST API：`GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/virtualMachines?api-version={apiVersion}` (如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesetvms/list) \(英文\))

Powershell：`Get-AzureRmVmssVM -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName}` (如需詳細資訊，請參閱 [Powershell 文件](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmssvm) \(英文\))

CLI：`az vmss list-instances -g {resourceGroupName} -n {vmScaleSetName}` (如需詳細資訊，請參閱 [CLI 文件](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az_vmss_list_instances) \(英文\))。

您也可以使用 [resources.azure.com](https://resources.azure.com) 或 [Azure SDK](https://azure.microsoft.com/downloads/) 來列出擴展集中的 VM。

確切的輸出呈現內容取決於您提供給命令的選項，但以下有一些來自 CLI 的範例輸出：

```
$ az vmss show -g {resourceGroupName} -n {vmScaleSetName}
[
  {
    "instanceId": "85",
    "latestModelApplied": true,
    "location": "westus",
    "name": "nsgvmss_85",
    .
    .
    .
```

如您所見，"instanceId" 屬性是十進位數字。 一旦將舊的執行個體刪除後，執行個體識別碼便可重複用於新的執行個體。

>[!NOTE]
> **不保證**將執行個體識別碼指派給擴展集中 VM 的方式。 它們可能有時看似循序遞增，但並不一定都是如此。 請勿依賴將執行個體識別碼指派給 VM 的特定方式。

## <a name="scale-set-vm-names"></a>擴展集 VM 名稱

在上述範例輸出中，還有 VM 的 "name"。 此名稱採用 "{scale-set-name}_{instance-id}" 形式。 當您列出擴展集中的執行個體時，您就會在 Azure 入口網站中看到這個名稱：

![](./media/virtual-machine-scale-sets-instance-ids/vmssInstances.png)

名稱的 {instance-id} 部分與先前所討論的 "instanceId" 屬性是相同的十進位數字。

## <a name="instance-metadata-vm-name"></a>執行個體中繼 VM 服務

如果您從擴展集 VM 內查詢[執行個體中繼資料](../virtual-machines/windows/instance-metadata-service.md)，您會在輸出中看到 "name":

```
{
  "compute": {
    "location": "westus",
    "name": "nsgvmss_85",
    .
    .
    .
```

這個名稱與先前所討論的名稱相同。

## <a name="scale-set-vm-computer-name"></a>擴展集 VM 電腦名稱

擴展集中的每部 VM 也會取得指派給它的電腦名稱。 此電腦名稱是[虛擬網路內 Azure 提供的 DNS 名稱解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)中的 VM 主機名稱。 此電腦名稱採用 "{computer-name-prefix}{base-36-instance-id}" 形式。

{base-36-instance-id} 採用 [Base 36](https://en.wikipedia.org/wiki/Base36)，且長度一律為六位數。 如果數字的 Base 36 表示法少於六位數，則 {base-36-instance-id} 會以零填補，使其長度成為六位數。 例如，包含 {computer-name-prefix} "nsgvmss" 和執行個體識別碼 85 之執行個體的電腦名稱會是 "nsgvmss00002D"。

>[!NOTE]
> 電腦名稱前置詞是您可設定之擴展集模型的屬性，因此它可能與擴展集名稱本身不同。