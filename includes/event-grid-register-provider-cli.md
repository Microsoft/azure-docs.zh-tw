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
ms.openlocfilehash: a7ed21d8246b618149aa0d116070a14b033d5370
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37869639"
---
## <a name="enable-event-grid-resource-provider"></a>啟用事件格線資源提供者

如果您先前在 Azure 訂用帳戶中從未使用過事件方格，可能必須註冊事件方格資源提供者。 執行以下命令：

```azurecli-interactive
az provider register --namespace Microsoft.EventGrid
```

可能需要一點時間才能完成註冊。 若要檢查狀態，請執行：

```azurecli-interactive
az provider show -n Microsoft.EventGrid --query "registrationState"
```

當 `registrationState` 是 `Registered` 時，代表已準備好繼續進行。