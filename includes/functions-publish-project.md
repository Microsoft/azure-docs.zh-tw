---
title: 包含檔案
description: 包含檔案
services: functions
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/27/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 1b553cbd720fcb76899844712ce5053af46f7ccb
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452950"
---
## <a name="deploy-the-function-app-project-to-azure"></a>將函式應用程式專案部署至 Azure

在 Azure 中建立函式應用程式之後，您可以使用 [`func azure functionapp publish`](../articles/azure-functions/functions-run-local.md#project-file-deployment) 命令來將專案程式碼部署至 Azure。

```bash
func azure functionapp publish <FunctionAppName>
```

您會看到類似下列的輸出，該輸出已截短，以提高可讀性。

```output
Getting site publishing info...

...

Preparing archive...
Uploading content...
Upload completed successfully.
Deployment completed successfully.
Syncing triggers...
```

您現在可以在 Azure 中測試函式。