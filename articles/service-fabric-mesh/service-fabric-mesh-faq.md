---
title: Azure Service Fabric Mesh 的常見問題 | Microsoft Docs
description: 了解 Azure Service Fabric Mesh 的常見問題和解答。
services: service-fabric-mesh
keywords: ''
author: chackdan
ms.author: chackdan
ms.date: 06/25/2018
ms.topic: troubleshooting
ms.service: service-fabric-mesh
manager: timlt
ms.openlocfilehash: d0ae7fbb22f6d98662f83968158182d447a75394
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39501962"
---
# <a name="commonly-asked-service-fabric-mesh-questions"></a>Service Fabric Mesh 的常見問題
Azure Service Fabric Mesh 是一個受到完整管理的服務，讓開發人員能夠部署微服務應用程式，而不需管理虛擬機器、儲存體或網路功能。 本文提供常見問題的解答。

## <a name="how-do-i-report-an-issue-or-ask-a-question"></a>如何回報問題或詢問問題？

在 [service-fabric-mesh-preview GitHub 存放庫](https://aka.ms/sfmeshissues) \(英文\) 中提出問題、獲得 Microsoft 工程師所提供的解答，以及回報問題。

## <a name="quota-and-cost"></a>配額和成本

**參與預覽版的成本為何？**

將應用程式或容器部署至 Mesh 預覽版不需任何費用。 不過，除非您要積極測試資源，否則建議刪除您所部署的資源且不要讓它們保持執行。

**核心和 RAM 數目有配額限制嗎？**

是的，每個訂用帳戶的配額如下：

- 應用程式數目：5 
- 每個訂用帳戶的核心數目：12 
- 每個應用程式的 RAM 總計：48 GB 
- 網路與輸入端點數目：5  
- 您可以連接的 Azure 磁碟區數目：10 
- 服務複本數目：3 
- 您可以部署的最大容器限制為 4 個核心、16 GB 的 RAM。
- 您可以將部分核心配置給容器，遞增量為 0.5 個核心，最多 6 個核心。

**我是否可以讓應用程式整夜執行？**

是的，可以。不過，除非您要積極測試資源，否則建議刪除您所部署的資源且不要讓它們保持執行。 此原則可能會在未來變更，而且資源如果遭到濫用，便有可能被刪除。

## <a name="supported-container-os-images"></a>支援的容器 OS 映像
部署服務時，可以使用下列容器 OS 映像。

- Windows：windowsservercore 和 nanoserver
    - Windows Server 2016
    - Windows Server 1709 版
- Linux
    - 沒有已知的限制

## <a name="features-gaps-and-known-issues"></a>功能差距和已知問題

**部署應用程式之後，與其相關聯的網路資源似乎沒有 IP 位址**

目前有個已知的問題是 IP 位址會在延遲之後出現。 在幾分鐘內檢查網路資源的狀態，以查看相關聯的 IP 位址。

**我的應用程式無法存取正確的網路/磁碟區資源**

在您的應用程式模型中，您需要使用網路和磁碟區的完整資源識別碼，才能存取相關聯的資源。 以下是此作業在快速入門範例中的模樣：

```json
"networkRefs": [
    {
    "name":  "[resourceId('Microsoft.ServiceFabric/networks', 'SbzVotingNetwork')]" 
    }
]
```

**我沒看到目前支援將祕密加密的應用程式模型**

是的，目前的個人預覽版不支援將祕密加密。 

**DNS 在我的 Service Fabric 開發叢集中以及 Mesh 中無法以相同方式運作**

有一個已知問題是，您可能必須在您的本機開發叢集中以及 Azure Mesh 中以不同方式參考服務。 在您的本機開發叢集中，請使用 {serviceName}.{applicationName}。 在 Azure Service Fabric 中，則使用 {servicename}。 Azure Mesh 目前不支援跨應用程式的 DNS 解析。

對於在 Windows 10 上執行 Service Fabric 開發叢集的其他已知 DNS 問題，請參閱此處：[對 Windows 容器進行偵錯](/azure/service-fabric/service-fabric-how-to-debug-windows-containers)。

**我在使用 CLI 模組時遇到此錯誤，ImportError: 無法匯入名稱 'sdk_no_wait'**

如果您使用的 CLI 版本比 2.0.30 還舊，就可能收到這個錯誤：

```
cannot import name 'sdk_no_wait'
Traceback (most recent call last):
File "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\knack\cli.py", line 193, in invoke cmd_result = self.invocation.execute(args)
File "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\azure\cli\core\commands_init_.py", line 241, in execute self.commands_loader.load_arguments(command)
File "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\azure\cli\core_init_.py", line 201, in load_arguments self.command_table[command].load_arguments() # this loads the arguments via reflection
File "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\azure\cli\core\commands_init_.py", line 142, in load_arguments super(AzCliCommand, self).load_arguments()
File "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\knack\commands.py", line 76, in load_arguments cmd_args = self.arguments_loader()
File "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\azure\cli\core_init_.py", line 332, in default_arguments_loader op = handler or self.get_op_handler(operation)
File "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\azure\cli\core_init_.py", line 375, in get_op_handler op = import_module(mod_to_import)
File "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\importlib_init_.py", line 126, in import_module return _bootstrap._gcd_import(name[level:], package, level)
File "", line 978, in _gcd_import
File "", line 961, in _find_and_load
File "", line 950, in _find_and_load_unlocked
File "", line 655, in _load_unlocked
File "", line 678, in exec_module
File "", line 205, in _call_with_frames_removed
File "C:\Users\annayak.azure\cliextensions\azure-cli-sbz\azext_sbz\custom.py", line 18, in 
from azure.cli.core.util import get_file_json, shell_safe_json_parse, sdk_no_wait
ImportError: cannot import name 'sdk_no_wait'.
```

**安裝 CLI 延伸模組套件時，收到不相符的散發名稱錯誤**

這不表示該延伸模組尚未安裝。 您仍然應該能夠使用 CLI 命令，而不會發生問題。

**當我相應放大時，發現所有容器都受到影響，包括正在執行的容器**

這是一個錯誤 (bug)，應該在下一次執行階段重新整理中加以修正。

## <a name="next-steps"></a>後續步驟

若要深入了解 Service Fabric Mesh，請閱讀[概觀](service-fabric-mesh-overview.md)。
