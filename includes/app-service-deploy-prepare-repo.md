---
title: 包含檔案
description: 包含檔案
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 06/05/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: c212bda3b59037f99139e02ee6adc63b0084cbe5
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44305609"
---
## <a name="prepare-your-repository"></a>準備您的存放庫

若要從 App Service Kudu 組建伺服器取得自動組建，請確定您的儲存機制根路徑在專案中具有正確檔案。

| 執行階段 | 根目錄檔案 |
|-|-|
| ASP.NET (僅限 Windows) | _*.sln_、_*.csproj_ 或 _default.aspx_ |
| ASP.NET Core | _*.sln_ 或 _*.csproj_ |
| PHP | _index.php_ |
| Ruby (僅限 Linux) | _Gemfile_ |
| Node.js | 含啟動指令碼的 _server.js_、_app.js_, 或 _package.json_ |
| Python (僅限 Windows) | _\*.py_、_requirements.txt_ 或 _runtime.txt_ |
| HTML | _default.htm_、_default.html_、_default.asp_、_index.htm_、_index.html_ 或 _iisstart.htm_ |
| WebJobs | _App\_Data/jobs/continuous_ (適用於持續的 WebJobs) 或 _App\_Data/jobs/triggered_ (適用於觸發的 WebJobs) 底下的 _\<job_name>/run.\<extension>_。 如需詳細資訊，請參閱 [Kudu WebJobs 文件](https://github.com/projectkudu/kudu/wiki/WebJobs) \(英文\)。 |
| Functions | 請參閱 [Azure Functions 的持續部署](../articles/azure-functions/functions-continuous-deployment.md#continuous-deployment-requirements)。 |

若要自訂部署，您可以在儲存機制根路徑中包含 _.deployment_ 檔案。 如需詳細資訊，請參閱[自訂部署](https://github.com/projectkudu/kudu/wiki/Customizing-deployments) \(英文\) 和[自訂部署指令碼](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script) \(英文\)。

> [!NOTE]
> 如果您是在 Visual Studio 中開發，可讓 [Visual Studio 為您建立存放庫](/azure/devops/repos/git/creatingrepo?view=vsts&tabs=visual-studio)。 專案立即可透過使用 Git 進行部署。
>
>

