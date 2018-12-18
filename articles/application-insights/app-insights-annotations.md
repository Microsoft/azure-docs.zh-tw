---
title: Application Insights 的發行註解 | Microsoft Docs
description: 在 Application Insights 中對計量瀏覽器新增部署或建置標記。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: 23173e33-d4f2-4528-a730-913a8fd5f02e
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 11/16/2016
ms.author: mbullwin
ms.openlocfilehash: f943f0e371b3092717a62a2e83a98211723e5302
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304402"
---
# <a name="annotations-on-metric-charts-in-application-insights"></a>Application Insights 中度量圖表上的註解
[計量瀏覽器](app-insights-metrics-explorer.md)圖表上的註解會顯示您在哪裡部署了新的組建，或是其他重要事件。 註解可讓您輕鬆查看變更是否對應用程式的效能有任何影響。 [Azure DevOps Services 建置系統](https://docs.microsoft.com/azure/devops/pipelines/tasks/)可以自動建立這些註解。 您也可以[從 PowerShell 建立註解](#create-annotations-from-powershell)來標幟您想要的任何事件。

![註解範例，其會顯示與伺服器回應時間的相互關聯](./media/app-insights-annotations/00.png)



## <a name="release-annotations-with-azure-devops-services-build"></a>與 Azure DevOps Services 組建搭配的發行註解

發行註解是 Azure DevOps Services 的雲端式 Azure Pipelines 服務的功能。 

### <a name="install-the-annotations-extension-one-time"></a>安裝註解擴充功能 (一次)
若要能夠建立發行註解，必須安裝 Visual Studio Marketplace 中的許多可用 Azure DevOps Services 擴充功能之一。

1. 登入您的 [Azure DevOps Services](https://visualstudio.microsoft.com/vso/) 專案。
2. 在 Visual Studio Marketplace 中[取得發行註解擴充功能](https://marketplace.visualstudio.com/items/ms-appinsights.appinsightsreleaseannotations)，並將其新增至您的 Azure DevOps Services 組織。

![在 Azure DevOps Services 網頁右上方開啟 Marketplace。 選取 Azure DevOps Services，然後在 Azure Pipelines 下方選擇 [查看更多]。](./media/app-insights-annotations/10.png)

您只需要為 Azure DevOps Services 組織執行一次此動作。 現在，您即可為組織中的任何專案設定發行註解。 

### <a name="configure-release-annotations"></a>設定發行註解

您必須為每個 Azure DevOps Services 發行範本取得個別的 API 金鑰。

1. 登入 [Microsoft Azure 入口網站](https://portal.azure.com)，然後開啟監視您應用程式的 Application Insights 資源。 (或如果您尚未建立該資源，請[立即建立一個](app-insights-overview.md))。
2. 開啟 [API 存取]、[Application Insights 識別碼]。
   
    ![在 portal.azure.com 中，開啟您的 Application Insights 資源然後選擇 [設定]。 開啟 [API 存取]。 複製應用程式識別碼](./media/app-insights-annotations/20.png)

4. 在另一個瀏覽器視窗中，開啟 (或建立) 可從 Azure DevOps Services 管理部署的發行範本。 
   
    新增工作，然後從功能表中選取 Application Insights 發行註解工作。
   
    將您從 [API 存取] 刀鋒視窗複製的應用程式識別碼  貼上。
   
    ![在 Azure DevOps Services 中開啟 [發行]，選取一個發行管線，然後選擇 [編輯]。 按一下 [新增工作] 然後選取 [Application Insights 發行註解]。 貼上 Application Insights 識別碼。](./media/app-insights-annotations/30.png)
4. 將 [APIKey] 欄位設定為變數 `$(ApiKey)`。

5. 回到 Azure 視窗，建立新的「API 金鑰」並複製它。
   
    ![在 [Azure] 視窗的 [API 存取] 刀鋒視窗中，按一下 [建立 API 金鑰]。 提供註解，檢查 [寫入註解]，然後按一下 [產生金鑰]。 複製新的金鑰。](./media/app-insights-annotations/40.png)

6. 開啟發行範本的 [設定] 索引標籤。
   
    建立 `ApiKey`的變數定義。
   
    將您的 API 金鑰貼上至 ApiKey 變數定義。
   
    ![在 [Azure DevOps Services] 視窗中選取 [組態] 索引標籤，然後按一下 [新增變數]。 設定 ApiKey 的名稱，並在 [值] 中，貼上剛才產生的金鑰，然後按一下鎖定圖示。](./media/app-insights-annotations/50.png)
7. 最後，**儲存**發行管線。


## <a name="view-annotations"></a>檢視註解
現在，每當您使用發行範本來部署新的發行，就會將註解傳送至 Application Insights。 註解將會出現在計量瀏覽器的圖表上。

按一下任一註解標記可開啟關於發行的詳細資料，包括要求者、原始檔控制分支、發行管線、環境等。

![按一下任一版本註解標記。](./media/app-insights-annotations/60.png)

## <a name="create-custom-annotations-from-powershell"></a>從 PowerShell 建立自訂註解
您也可以從任何您慣用的程序建立註解 (不使用 Azure DevOps Services)。 


1. 建立一個[來自 GitHub 的 Powershell 指令碼](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/API/CreateReleaseAnnotation.ps1)的本機複本。

2. 從 [API 存取] 刀鋒視窗中取得「應用程式識別碼」並建立 API 金鑰。

3. 依下列方式呼叫指令碼：

```PS

     .\CreateReleaseAnnotation.ps1 `
      -applicationId "<applicationId>" `
      -apiKey "<apiKey>" `
      -releaseName "<myReleaseName>" `
      -releaseProperties @{
          "ReleaseDescription"="a description";
          "TriggerBy"="My Name" }
```

修改指令碼很簡單，例如，修改成建立過去的註解。

## <a name="next-steps"></a>後續步驟

* [建立工作項目](app-insights-diagnostic-search.md#create-work-item)
* [使用 PowerShell 進行自動化](app-insights-powershell.md)