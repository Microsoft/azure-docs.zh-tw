---
title: 適用於 Machine Learning 的 PowerShell 模組 | Microsoft Docs
description: 適用於 Azure Machine Learning 的 PowerShell 模組提供公開預覽模式。 您可以使用 PowerShell 來建立及管理工作區、實驗、Web 服務等。
keywords: 實驗、線性迴歸、機器學習服務演算法、機器學習服務教學課程、預測性模型技術、資料科學實驗
services: machine-learning
documentationcenter: ''
author: hning86
ms.author: haining
manager: mwinkle
editor: cgronlun
ms.assetid: a9001cc2-3aa0-47e1-b175-1f76408ba1d1
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/15/2017
ms.openlocfilehash: 6ecd2d9a1519cd89058385ad1e40aee9b3fc9082
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835690"
---
# <a name="powershell-module-for-microsoft-azure-machine-learning"></a>適用於 Microsoft Azure Machine Learning 的 PowerShell 模組
適用於 Azure Machine Learning 的 PowerShell 模組是一款功能強大的工具，它能讓您使用 Windows PowerShell 來管理工作區、實驗、資料集、傳統的 Web 服務等。

若要檢視文件及下載模組和完整的原始程式碼，請前往 [https://aka.ms/amlps](https://aka.ms/amlps)。 

> [!NOTE]
> Azure Machine Learning 的 PowerShell 模組目前為預覽模式。 在此預覽期間內，我們將會持續改善及擴充模組。 如需相關新聞和資訊，敬請關注 [Cortana Intelligence 和 Machine Learning 部落格](https://blogs.technet.microsoft.com/machinelearning/)。

## <a name="what-is-the-machine-learning-powershell-module"></a>什麼是 Machine Learning PowerShell 模組？
Machine Learning PowerShell 模組是以 .NET 為基礎的 DLL 模組，它能讓您從 Windows PowerShell 全權管理 Azure Machine Learning 工作區、實驗、資料集、傳統的 Web 服務及傳統的 Web 服務端點。 

您可以隨著此模組一同下載完整的原始程式碼，其中包括完全分隔的 [C# API 層](https://github.com/hning86/azuremlps/blob/master/code/AzureMLSDK.cs)。 您可以從自己的 .NET 專案參考此 DLL，並透過 .NET 程式碼管理 Azure Machine Learning。 此外，對於 DLL 仰賴的基礎 REST API，您可以從常用的用戶端中直接使用。

## <a name="what-can-i-do-with-the-powershell-module"></a>PowerShell 模組有什麼功能？
以下是一些您可以使用此 PowerShell 模組來執行的工作。 如需以下功能和其他更多功能的相關資訊，請參閱 [完整文件](https://aka.ms/amlps) 。

* 使用管理憑證佈建新工作區 ([New-AmlWorkspace](https://github.com/hning86/azuremlps#new-amlworkspace))
* 匯出及匯入代表實驗圖形的 JSON 檔案 ([Export-AmlExperimentGraph](https://github.com/hning86/azuremlps#export-amlexperimentgraph) 和 [Import-AmlExperimentGraph](https://github.com/hning86/azuremlps#import-amlexperimentgraph))
* 執行實驗 ([Start-AmlExperiment](https://github.com/hning86/azuremlps#start-amlexperiment))
* 利用預測性實驗建立 Web 服務 ([New-AmlWebService](https://github.com/hning86/azuremlps#new-amlwebservice))
* 在發佈的 Web 服務上建立端點 ([Add-AmlWebServiceEndpoint](https://github.com/hning86/azuremlps#add-amlwebserviceendpoint))
* 叫用 RRS 和/或 BES Web 服務端點 ([Invoke-AmlWebServiceRRSEndpoint](https://github.com/hning86/azuremlps#invoke-amlwebservicerrsendpoint) 和 [Invoke-AmlWebServicBESEndpoint](https://github.com/hning86/azuremlps#invoke-amlwebservicebesendpoint))

以下是使用 PowerShell 執行現有實驗的簡單範例︰

        #Find the first Experiment named “xyz”
        $exp = (Get-AmlExperiment | where Description -eq ‘xyz’)[0]
        #Run the Experiment
        Start-AmlExperiment -ExperimentId $exp.ExperimentId 

如需更深入的使用案例，請參閱以下有關使用 PowerShell 模組將普遍要求之工作自動化的文章：[使用 PowerShell，從一個實驗中建立許多機器學習服務模型和 Web 服務端點](create-models-and-endpoints-with-powershell.md)。

## <a name="how-do-i-get-started"></a>如何開始使用？
若要開始使用 Machine Learning PowerShell，請從 GitHub 下載[發行套件](https://github.com/hning86/azuremlps/releases)並遵循[安裝指示](https://github.com/hning86/azuremlps/blob/master/README.md)。 指示說明如何解除封鎖已下載/解壓縮的 DLL，然後再匯入 PowerShell 環境。 大部分的 Cmdlet 都會要求您提供工作區識別碼、工作區授權權杖，以及工作區所在的 Azure 區域。 提供值最簡單的方式是透過預設的 config.json 檔案。 指示中也會說明如何設定這個檔案。 

如果您想要，也可以使用 Visual Studio 複製 git 樹狀結構，然後在本機修改它。

## <a name="next-steps"></a>後續步驟
您可以在 [https://aka.ms/amlps](https://aka.ms/amlps) 找到完整的 PowerShell 模組文件。 

如需如何在真實案例中使用此模組的延伸範例，請看深入使用案例[使用 PowerShell，從一個實驗中建立許多 Machine Learning 模型和 Web 服務端點](create-models-and-endpoints-with-powershell.md)。
