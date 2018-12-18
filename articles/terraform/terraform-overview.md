---
title: 搭配 Azure 使用 Terraform
description: 使用 Terraform 設定 Azure 基礎結構版本和部署的簡介。
services: terraform
ms.service: terraform
keywords: Terraform, DevOps, 概觀, 計畫, 套用, 自動化
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 08/31/2018
ms.openlocfilehash: dd340238f8a70c1dd0cfc172976bef6b1ad282b1
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43665624"
---
# <a name="terraform-with-azure"></a>搭配 Azure 使用 Terraform

[Hashicorp Terraform](https://www.terraform.io/) (英文) 是開放原始碼工具，可用來佈建和管理雲端基礎結構。 它會在設定檔中制訂基礎結構，以描述雲端資源的拓撲，例如虛擬機器、儲存體帳戶和網路介面。 Terraform 的命令列介面 (CLI) 提供簡單的機制，以針對 Azure 或任何其他支援雲端的設定檔來部署和設定版本。

本文說明使用 Terraform 來管理 Azure 基礎結構的優點。

## <a name="automate-infrastructure-management"></a>將基礎結構管理自動化。

Terraform 的範本型設定檔讓您能夠以可重複且可預測的方式定義、佈建和設定 Azure 資源。 自動化基礎結構有幾項優點：

- 部署和管理基礎結構時，降低發生人為錯誤的可能性。
- 可多次部署同樣的範本，以建立完全相同的開發、測試和生產環境。
- 可依需求建立開發和測試環境，降低建立這些環境的成本。

## <a name="understand-infrastructure-changes-before-they-are-applied"></a>可先了解基礎結構的變更，然後再套用變更 

隨著資源拓撲變得更為複雜，了解基礎結構變更的意義和影響也會變得困難。

Terraform 提供命令列介面 (CLI)，讓使用者能夠在部署基礎結構之前先驗證和預覽其變更。 以安全有效率的方式預覽基礎結構的變更有幾個好處：
- 小組成員可快速地了解預計的變更和其影響，因而更有效地進行共同作業。
- 在開發過程中能提早攔截非預期的變更


## <a name="deploy-infrastructure-to-multiple-clouds"></a>將基礎結構部署到多個雲端

Terraform 是針對多重雲端案例的常用工具，可以將類似的基礎結構部署到 Azure 和其他的雲端提供者，或是內部部署資料中心。 它可讓開發人員使用相同的工具和設定檔，以管理多個雲端提供者上的基礎結構。

## <a name="next-steps"></a>後續步驟

既然您已了解 Terraform 的概觀和其優點，以下是建議的後續步驟：

- 了解如何[安裝 Terraform 並設定它以使用 Azure](https://docs.microsoft.com/azure/virtual-machines/linux/terraform-install-configure)。
- [使用 Terraform 建立 Azure 虛擬機器](https://docs.microsoft.com/azure/virtual-machines/linux/terraform-create-complete-vm)
- 探索[適用於 Terraform 的 Azure Resource Manager 模組](https://www.terraform.io/docs/providers/azurerm/) \(英文\) 