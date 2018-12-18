---
title: 包含檔案
description: 包含檔案
services: azure-policy
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 09/18/2018
ms.author: dacoulte
ms.custom: include file
ms.openlocfilehash: eadff6f61bd9e5979ae835230de53562ff8542d1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47003692"
---
### <a name="virtual-machines"></a>虛擬機器

|  |  |
|---------|---------|
| [允許來自某個資源群組的自訂 VM 映像](../articles/governance/policy/samples/allow-custom-vm-image.md) |  要求自訂映像必須來自某個已核准的資源群組。 您需指定已核准之資源群組的名稱。 |
| [允許的儲存體帳戶和虛擬機器 SKU](../articles/governance/policy/samples/allowed-skus-storage.md) | 要求儲存體帳戶和虛擬機器使用已核准的 SKU。 使用內建的原則來確保使用已核准的 SKU。 您需指定已核准的虛擬機器 SKU 陣列和已核准的儲存體帳戶 SKU 陣列。 |
| [已核准的 VM 映像](../articles/governance/policy/samples/allowed-custom-images.md) | 要求只能在環境中部署已核准的自訂映像。 您需指定已核准的映像識別碼陣列。 |
| [稽核擴充功能是否不存在](../articles/governance/policy/samples/audit-ext-not-exist.md) | 稽核是否未搭配虛擬機器部署擴充功能。 您需指定擴充功能發行者和類型，以檢查是否已部署該擴充功能。 |
| [不允許的 VM 擴充功能](../articles/governance/policy/samples/not-allowed-vm-ext.md) | 禁止使用指定的擴充功能。 您需指定一個包含所禁止擴充功能類型的陣列。 |