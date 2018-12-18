---
title: 管理 Azure 自動化中的 Python 2 封裝
description: 本文說明如何管理 Azure 自動化中的 Python 2 封裝。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 09/11/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 5974a8e622ca0969b2a7b5ee9500766ac95398c9
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45987219"
---
# <a name="manage-python-2-packages-in-azure-automation"></a>管理 Azure 自動化中的 Python 2 封裝

Azure 自動化可讓您在 Azure 和 Linux 混合式 Runbook 背景工作角色上執行 Python 2 Runbook。 若要協助簡化 Runbook，您可以使用 Python 封裝來匯入需要的模組。 本文說明如何在 Azure 自動化中管理和使用 Python 封裝。

## <a name="import-packages"></a>匯入封裝

從您的自動化帳戶中，選取 [共用資源] 下的 [Python 2 封裝]。 按一下 [+ 新增 Python 2 封裝]。

![新增 Python 封裝](media/python-packages/add-python-package.png)

在 [新增 Python 2 封裝] 頁面上，選取要上載的本機封裝。 此封裝可為 `.whl` 檔案或 `.tar.gz` 檔案。 選取後，按一下 [確定] 以上載封裝。

![新增 Python 封裝](media/python-packages/upload-package.png)

匯入封裝後，該封裝會列於自動化帳戶中的 [Python 2 封裝] 頁面。 如果您需要移除封裝，請在封裝頁面上選取封裝，然後選擇 [刪除]。

![封裝清單](media/python-packages/package-list.png)

## <a name="use-a-package-in-a-runbook"></a>在 Runbook 中使用封裝

匯入封裝後，您現在可以在 Runbook 中使用。 下列範例會使用 [Azure 自動化公用程式封裝](https://github.com/azureautomation/azure_automation_utility)。 此封裝可讓您更輕鬆配合使用 Python 與 Azure 自動化。 若要使用此封裝，請遵循 GitHub 存放庫中的指示，並透過使用 `from azure_automation_utility import get_automation_runas_credential` 將其新增至 Runbook (例如，匯入函式以擷取 RunAs 帳戶)。

```python
import azure.mgmt.resource
import automationassets
from azure_automation_utility import get_automation_runas_credential

# Authenticate to Azure using the Azure Automation RunAs service principal
runas_connection = automationassets.get_automation_connection("AzureRunAsConnection")
azure_credential = get_automation_runas_credential()

# Intialize the resource management client with the RunAs credential and subscription
resource_client = azure.mgmt.resource.ResourceManagementClient(
    azure_credential,
    str(runas_connection["SubscriptionId"]))

# Get list of resource groups and print them out
groups = resource_client.resource_groups.list()
for group in groups:
    print group.name
```

## <a name="develop-and-test-runbooks-offline"></a>離線開發和測試 Runbook

若要離線開發和測試 Python 2 Runbook，您可以使用 GitHub 上的 [Azure 自動化 Python 模擬資產模組](https://github.com/azureautomation/python_emulated_assets)。 此模組可讓您參考共用的資源，例如認證、變數、連線和憑證。

## <a name="next-steps"></a>後續步驟

若要開始使用 Python 2 Runbook，請參閱[我的第一個 Python 2 Runbook](automation-first-runbook-textual-python2.md)