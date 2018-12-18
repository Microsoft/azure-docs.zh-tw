---
title: 啟用 Azure 防火牆公開預覽
description: 使用 Azure PowerShell 來啟用 Azure 防火牆公開預覽
author: vhorne
ms.service: firewall
services: firewall
ms.topic: article
ms.date: 7/11/2018
ms.author: victorh
ms.openlocfilehash: 263b16a419b5ff20a9b6d62860385f92c2a18f9c
ms.sourcegitcommit: df50934d52b0b227d7d796e2522f1fd7c6393478
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38992196"
---
# <a name="enable-the-azure-firewall-public-preview"></a>啟用 Azure 防火牆公開預覽

[!INCLUDE [firewall-preview-notice](../../includes/firewall-preview-notice.md)]

## <a name="enable-using-azure-powershell"></a>使用 Azure PowerShell 啟用

若要啟用 Azure 防火牆公開預覽，請使用下列 Azure PowerShell 命令：

```PowerShell
Register-AzureRmProviderFeature -FeatureName AllowRegionalGatewayManagerForSecureGateway -ProviderNamespace Microsoft.Network

Register-AzureRmProviderFeature -FeatureName AllowAzureFirewall -ProviderNamespace Microsoft.Network
```

需要 30 分鐘才能完成功能註冊。 您可以執行下列 Azure PowerShell 命令來檢查您的註冊狀態：

```PowerShell

Get-AzureRmProviderFeature -FeatureName AllowRegionalGatewayManagerForSecureGateway -ProviderNamespace Microsoft.Network

Get-AzureRmProviderFeature -FeatureName AllowAzureFirewall -ProviderNamespace Microsoft.Network
```
註冊完成之後，請執行下列命令：

```PowerShell
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```

## <a name="next-steps"></a>後續步驟

- [教學課程：使用 Azure 入口網站來部署和設定 Azure 防火牆](tutorial-firewall-deploy-portal.md)

