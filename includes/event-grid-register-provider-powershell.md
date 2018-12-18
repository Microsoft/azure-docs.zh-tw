---
title: 包含檔案
description: 包含檔案
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: include
ms.date: 07/05/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 0946b703b45f69ede409167213f2629a796f18b5
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37869622"
---
## <a name="enable-event-grid-resource-provider"></a>啟用事件格線資源提供者

如果您先前在 Azure 訂用帳戶中從未使用過事件方格，可能必須註冊事件方格資源提供者。 執行以下命令：

```azurepowershell-interactive
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.EventGrid
```

可能需要一點時間才能完成註冊。 若要檢查狀態，請執行：

```azurepowershell-interactive
Get-AzureRmResourceProvider -ProviderNamespace Microsoft.EventGrid
```

當 `RegistrationStatus` 是 `Registered` 時，代表已準備好繼續進行。