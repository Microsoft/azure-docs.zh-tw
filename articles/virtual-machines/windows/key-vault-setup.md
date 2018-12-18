---
title: 為 Azure Resource Manager 中的 Windows VM 設定 Key Vault | Microsoft Docs
description: 如何設定要與 Azure Resource Manager 虛擬機器搭配使用的金鑰保存庫。
services: virtual-machines-windows
documentationcenter: ''
author: singhkays
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 33a483e2-cfbc-4c62-a588-5d9fd52491e2
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 01/24/2017
ms.author: kasing
ms.openlocfilehash: 599b16f633d9a0de5165bdf5cb3d7b82abca655b
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39597705"
---
# <a name="set-up-key-vault-for-virtual-machines-in-azure-resource-manager"></a>為 Azure Resource Manager 中的虛擬機器設定金鑰保存庫

[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-rm-include.md)]

在 Azure Resource Manager 堆疊中，密碼/憑證會被塑造成「金鑰保存庫資源提供者」所提供的資源。 若要深入了解「金鑰保存庫」，請參閱 [什麼是 Azure 金鑰保存庫？](../../key-vault/key-vault-whatis.md)

> [!NOTE]
> 1. 為了讓「金鑰保存庫」能與 Azure Resource Manager 虛擬機器搭配使用，必須將「金鑰保存庫」上的 **EnabledForDeployment** 屬性設定為 true。 您可以在各種用戶端中執行這項操作。
> 2. 「金鑰保存庫」必須建立在與「虛擬機器」相同的訂用帳戶和位置中。
>
>

## <a name="use-powershell-to-set-up-key-vault"></a>使用 PowerShell 來設定金鑰保存庫
若要使用 PowerShell 來建立「金鑰保存庫」，請參閱 [開始使用 Azure 金鑰保存庫](../../key-vault/key-vault-get-started.md#vault)。

針對新的「金鑰保存庫」，您可以使用下列 PowerShell Cmdlet：

    New-AzureRmKeyVault -VaultName 'ContosoKeyVault' -ResourceGroupName 'ContosoResourceGroup' -Location 'East Asia' -EnabledForDeployment

針對現有的「金鑰保存庫」，您可以使用下列 PowerShell Cmdlet：

    Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -EnabledForDeployment

## <a name="use-cli-to-set-up-key-vault"></a>使用 CLI 來設定金鑰保存庫
若要使用命令列介面 (CLI) 建立金鑰保存庫，請參閱 [使用 CLI 管理金鑰保存庫](../../key-vault/key-vault-manage-with-cli2.md#create-a-key-vault)。

若使用 CLI，您必須在您指派部署原則之前建立金鑰保存庫。 您可以使用下列命令來達成目的：

    azure keyvault set-policy ContosoKeyVault –enabled-for-deployment true

## <a name="use-templates-to-set-up-key-vault"></a>使用範本來設定金鑰保存庫
使用範本時，您需要將「金鑰保存庫」資源的 `enabledForDeployment` 屬性設定為 `true`。

    {
      "type": "Microsoft.KeyVault/vaults",
      "name": "ContosoKeyVault",
      "apiVersion": "2015-06-01",
      "location": "<location-of-key-vault>",
      "properties": {
        "enabledForDeployment": "true",
        ....
        ....
      }
    }

如需使用範本來建立金鑰保存庫時可以設定的其他選項，請參閱 [Create a Key Vault (建立金鑰保存庫)](https://azure.microsoft.com/documentation/templates/101-key-vault-create/)。
