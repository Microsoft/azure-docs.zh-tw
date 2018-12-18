---
title: Azure Machine Learning 部署疑難排解指南 | Microsoft Docs
description: 部署和服務建立的疑難排解指南
services: machine-learning
author: aashishb
ms.author: aashishb
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 11/16/2017
ROBOTS: NOINDEX
ms.openlocfilehash: a2867545e454e2b13360d87f5282e684753d6476
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46994491"
---
# <a name="troubleshooting-service-deployment-and-environment-setup"></a>針對服務部署和環境設定進行疑難排解

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

下列資訊可協助判斷在設定模型管理環境時的錯誤原因。

## <a name="model-management-environment"></a>模型管理環境
### <a name="contributor-permission-required"></a>需要參與者權限
您需要訂用帳戶或資源群組的參與者存取權，才能為 Web 服務的部署設定叢集。

### <a name="resource-availability"></a>資源可用性
您的訂用帳戶中需要有足夠的可用資源，才能佈建環境資源。

### <a name="subscription-caps"></a>訂用帳戶上限
您的訂用帳戶可能有帳單上限，這會讓您無法佈建環境資源。 移除該上限才能啟用佈建。

### <a name="enable-debug-and-verbose-options"></a>啟用偵錯和詳細資訊選項
在設定命令中使用 `--debug` 和 `--verbose` 旗標，以在佈建環境時顯示偵錯和追蹤資訊。

```
az ml env setup -l <location> -n <name> -c --debug --verbose 
```

## <a name="service-deployment"></a>服務部署
下列資訊可協助判斷在部署期間或呼叫 web 服務時的錯誤原因。

### <a name="service-logs"></a>服務記錄
服務 CLI 的 `logs` 選項會提供來自 Docker 和 Kubernetes 的記錄資料。

```
az ml service logs realtime -i <web service id>
```

如需其他記錄設定，請使用 `--help` (或 `-h`) 選項。

```
az ml service logs realtime -h
```

### <a name="debug-and-verbose-options"></a>偵錯和 Verbose 選項
在部署服務時，使用 `--debug` 旗標來顯示偵錯記錄。

```
az ml service create realtime -m <modelfile>.pkl -f score.py -n <service name> -r python --debug
```

在部署服務時，使用 `--verbose` 旗標來查看其他詳細資料。

```
az ml service create realtime -m <modelfile>.pkl -f score.py -n <service name> -r python --verbose
```

### <a name="enable-request-logging-in-app-insights"></a>在 App Insights 中啟用要求記錄
建立 web 服務來啟用要求層級記錄時，將 `-l` 旗標設定為 true。 要求記錄會寫入您在 Azure 中的 App Insights 執行個體環境。 使用您在使用 `az ml env setup` 命令時所用的環境名稱來搜尋此執行個體。

- 建立服務時，將 `-l` 設定為 true。
- 在 Azure 入口網站中開啟 App Insights。 使用您的環境名稱來尋找 App Insights 執行個體。
- 在 App Insights 後，於上方功能表中按一下搜尋來檢視結果。
- 或移至 `Analytics` > `Exceptions` > `exceptions take | 10`。


### <a name="add-error-handling-in-scoring-script"></a>在評分指令碼中新增錯誤處理
使用 `scoring.py` 指令碼的 **run** 函式中的例外狀況處理來傳回錯誤訊息，作為您 web 服務輸出的一部分。

Python 範例：
```
    try:
        <code to load model and score>
   except Exception as e:
        return(str(e))
```

## <a name="other-common-problems"></a>其他常見問題
- 如果在 Windows 電腦上 azure-cli-ml 的 pip 安裝失敗且有錯誤 `cannot find the path specified`，則您必須啟用長路徑支援。 請參閱 https://blogs.msdn.microsoft.com/jeremykuhne/2016/07/30/net-4-6-2-and-long-paths-on-windows-10/。 
- 如果 `env setup` 命令因 `LocationNotAvailableForResourceType` 而失敗，您可能為機器學習資源使用錯誤的位置 (區域)。 請確定您使用 `-l` 參數指定的位置是 `eastus2`、`westcentralus` 或 `australiaeast`。
- 如果 `env setup` 命令因 `Resource quota limit exceeded` 而失敗，請確定您的訂用帳戶中有足夠的核心，而且其他處理序沒有耗盡您的資源。
- 如果 `env setup` 命令因 `Invalid environment name. Name must only contain lowercase alphanumeric characters` 而失敗，請確定服務名稱不包含大寫字母、符號或底線 (_) (如 *my_environment*)。
- 如果 `service create` 命令因 `Service Name: [service_name] is invalid. The name of a service must consist of lower case alphanumeric characters (etc.)` 而失敗，請確定服務名稱的長度是 3 到 32 個字元、開頭和結尾都是小寫的英數字元，而且不包含大寫字母、除了連字號 ( - ) 和句號( . ) 或底線 (_) 以外的符號 (如*my_webservice*)。
- 如果您在呼叫 Web 服務時收到 `502 Bad Gateway` 錯誤，請重試。 這通常表示容器尚未部署到叢集中。
- 如果您在建立服務時收到 `CrashLoopBackOff` 錯誤，請檢查您的記錄。 這通常是因為在 **init** 函式中缺少相依性。
