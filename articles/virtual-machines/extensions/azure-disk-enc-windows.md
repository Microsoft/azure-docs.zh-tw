---
title: 適用於 Windows 的 Azure 磁碟加密 | Microsoft Docs
description: 使用虛擬機器擴充功能將 Azure 磁碟加密部署至 Windows 虛擬機器。
services: virtual-machines-windows
documentationcenter: ''
author: ejarvi
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 06/12/2018
ms.author: ejarvi
ms.openlocfilehash: 11ec26729b2239279dddc8cd62f6b658a4f7ed20
ms.sourcegitcommit: 96f498de91984321614f09d796ca88887c4bd2fb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39413785"
---
# <a name="azure-disk-encryption-for-windows-microsoftazuresecurityazurediskencryption"></a>適用於 Windows 的 Azure 磁碟加密 (Microsoft.Azure.Security.AzureDiskEncryption)

## <a name="overview"></a>概觀

Azure 磁碟加密會利用 Bitlocker 在執行 Windows 的 Azure 虛擬機器上提供完整的磁碟加密。  此解決方案與 Azure Key Vault 整合，可讓您管理金鑰保存庫訂用帳戶中的磁碟加密金鑰與祕密。 

## <a name="prerequisites"></a>必要條件

如需先決條件的完整清單，請參閱 [Azure 磁碟加密先決條件](
../../security/azure-security-disk-encryption-prerequisites.md)。

### <a name="operating-system"></a>作業系統

如需目前的 Windows 版本清單，請參閱 [Azure 磁碟加密先決條件](../../security/azure-security-disk-encryption-prerequisites.md)。

### <a name="internet-connectivity"></a>網際網路連線

Azure 磁碟加密需要網際網路連線以存取 Active Directory、Key Vault、儲存體和套件管理端點。  如需網路安全性設定的詳細資訊，請參閱 [Azure 磁碟加密先決條件](
../../security/azure-security-disk-encryption-prerequisites.md)。

## <a name="extension-schema"></a>擴充功能結構描述

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2015-06-15",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientSecret": "[aadClientSecret]",
    },
    "publisher": "Microsoft.Azure.Security",
    "settings": {
      "AADClientID": "[aadClientID]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "KeyVaultURL": "[keyVaultURL]",
      "SequenceVersion": "sequenceVersion]",
      "VolumeType": "[volumeType]"
    },
    "type": "AzureDiskEncryption",
    "typeHandlerVersion": "[extensionVersion]"
  }
}
```

### <a name="property-values"></a>屬性值

| Name | 值 / 範例 | 資料類型 |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | 日期 |
| publisher | Microsoft.Azure.Security | 字串 |
| type | AzureDiskEncryptionForWindows| 字串 |
| typeHandlerVersion | 1.0、2.2 (VMSS) | int |
| (選擇性) AADClientID | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | GUID | 
| (選擇性) AADClientSecret | password | 字串 |
| (選擇性) AADClientCertificate | thumbprint | 字串 |
| EncryptionOperation | EnableEncryption | 字串 | 
| KeyEncryptionAlgorithm | RSA-OAEP | 字串 |
| KeyEncryptionKeyURL | url | 字串 |
| KeyVaultURL | url | 字串 |
| SequenceVersion | uniqueidentifier | 字串 |
| VolumeType | 作業系統、資料、全部 | 字串 |

## <a name="template-deployment"></a>範本部署
如需範本部署的範例，請參閱[從資源庫映像建立新的加密 Windows VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-new-vm-gallery-image)。

## <a name="azure-cli-deployment"></a>Azure CLI 部署

您可以在最新的 [Azure CLI 文件](/cli/azure/vm/encryption?view=azure-cli-latest)中找到相關指示。 

## <a name="troubleshoot-and-support"></a>疑難排解與支援

### <a name="troubleshoot"></a>疑難排解

請參閱 [Azure 磁碟加密疑難排解指南](../../security/azure-security-disk-encryption-tsg.md)。

### <a name="support"></a>支援

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/community/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。

## <a name="next-steps"></a>後續步驟
如需擴充功能的詳細資訊，請參閱[虛擬機器擴充功能和 Windows 功能](features-windows.md)。