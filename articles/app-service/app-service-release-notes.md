---
title: Azure SDK for .NET 2.5.1 版本資訊
description: Azure SDK for .NET 2.5.1 版本資訊
services: app-service
documentationcenter: .net,nodejs,java
author: Juliako
manager: erikre
editor: ''
ms.assetid: 8d3d815f-bb58-447e-8ff0-f9b9603c7b00
ms.service: app-service
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 10/10/2016
ms.author: juliako
ms.openlocfilehash: 0a422b02623a18ac6a1eef460bbada681672e69f
ms.sourcegitcommit: f8437edf5de144b40aed00af5c52a20e35d10ba1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/03/2017
ms.locfileid: "23957410"
---
# <a name="azure-sdk-for-net-251-release-notes"></a>Azure SDK for .NET 2.5.1 版本資訊
本文包含 Azure SDK for .NET 2.5.1 版的版本資訊。 

## <a name="azure-sdk-for-net-251-release-notes"></a>Azure SDK for .NET 2.5.1 版本資訊
以下是 Azure SDK for .NET 2.5.1 的新功能和更新。

* **Web 工具延伸模組**的相關新功能/案例。 
  
  * Azure 網站已重新命名為 Azure 應用程式服務。 
  * 已加入 Azure API 應用程式 (預覽) 支援，讓客戶可以將 ASP.NET 專案發佈為 API 應用程式，然後在 C# 專案中使用 [新增] > [Azure API 應用程式用戶端] 筆勢，根據已部署的 API 應用程式結構產生程式碼。 
  * 在伺服器總管中的 [網站] 節點已淘汰並取代為 Azure 應用程式服務節點，可支援以資源群組為基礎對 Azure API 應用程式、行動應用程式和 Web 應用程式進行分組的功能。
  * 已加入 Azure 行動應用程式 (預覽) 支援，讓客戶可以建立新的行動應用程式專案、新增行動應用程式控制器、發佈專案，以及從遠端偵錯應用程式。
  * [新增] > [Azure API 應用程式用戶端] 筆勢現已支援本機 Swagger JSON 檔案，讓 Web API 開發人員可以使用協力廠商 NuGets (例如 Swashbuckle) 產生 Swagger 或手動加以撰寫。 如此一來，用戶端開發人員將可使用程式碼產生功能，來取用 C# 專案中的任何 Swagger 端點。 
  * 「Web 應用程式」和「API 應用程式」發佈對話方塊已增強，可支援 Azure 入口網站的資源群組概念，而「Azure 資源群組」的選取/建立及「App Service 方案」則呈現在新的「Web 應用程式」和「API 應用程式」佈建對話方塊中。 
  * 「Azure API 應用程式伺服器總管」節點提供對 Azure 入口網站中「API 應用程式」深層連結的連結，以及「記錄檔串流」和「遠端偵錯」等其他功能。
    
    若想了解 Azure SDK .NET 2.5.1 中的已知問題和目前的限制，請參閱下方的 [此](app-service-release-notes.md#known_issues_2_5_1) 節。
* 此版本啟用了 Visual Studio 中與 **HDInsight 工具**有關的新功能/案例。 
  
  * hive 指令碼的本機驗證。 在工具列中按一下 [驗證指令碼] 按鈕，查看您的指令碼中是否有任何錯誤。 
  * 改善了 Hive 工作的偵錯功能。 您現在可以藉由在 Visual Studio 中存取 Yarn 記錄檔，對 Hive 工作進行偵錯。 如果您的應用程式有效能問題，調查 YARN 記錄將會提供有用的資訊。
  * (公用預覽) 為 Hive 提供關鍵字自動完成和 IntelliSense 支援。 為了協助您撰寫 Hive 指令碼、HDInsight Tools for Visual Studio 加入了 Hive 的關鍵字自動完成和 IntelliSense 支援。
  * Storm 支援。 您現在可以使用 HDInsight Tools for Visual Studio，在 C# 中開發 Storm 拓撲/Spout/Bolt。 然後，您可以將已開發的拓撲提交至 Storm 叢集，並查看拓撲/bolt/spout 狀態。 您可以使用系統記錄檔和客戶記錄檔，對您的 Storm 拓撲/Bolt/Spout 進行疑難排解。 您也可以在 Storm on HDInsight 中使用現有的 JAVA 資產。
    
    如需詳細資訊，請參閱 [開始使用 HDInsight Hadoop Tools for Visual Studio](../hdinsight/hadoop/apache-hadoop-visual-studio-tools-get-started.md)。

## <a id="known_issues_2_5_1"></a>Azure SDK for.NET 2.5.1 的已知問題和限制
* Azure API 應用程式會顯示為行動應用程式的部署目標。 在下一個版本之前，Web 應用程式應是行動應用程式的唯一目的地。 
* Azure API 應用程式佈建通常可順利完成，但偶爾會無法更新 Azure 應用程式服務活動視窗中的進度。 解決方法是在 Azure 入口網站中檢查新「Azure API 應用程式」的狀態。 
* 在使用 [檔案] > [新增專案] > [API 應用程式] > [F5] 時，會因為沒有 default/index.html 會產生 HTTP 錯誤。 解決方法是手動瀏覽至 /api/values URL。 
* 伺服器總管圖示偶爾會呈現扁平狀。 重新啟動 VS 可解決這個問題。 
* 如果在 Web 應用程式或 API 應用程式佈建期間擲回例外狀況 (例如，已超出配額錯誤或重複的 Azure API 應用程式閘道名稱)，錯誤會顯示一些原始 JSON 文字。 
* 在建立專案期間查看 Application Insights 時，偶爾會發生專案建立問題。
* 有時候，產生的 Azure API 應用程式用戶端程式碼會遺漏命名空間，而必須以手動方式加入 (或透過 Visual Studio 提示自動匯入)，才能編譯程式碼。 
* 「行動應用程式」專案應發佈至 Web 應用程式，但是您必須在 Azure 入口網站中挑選已建立為「行動應用程式」的網站，因為「行動應用程式」專案需要有資料庫。 
* 行動應用程式的起始頁面使用「行動服務」一詞，而不是 「行動應用程式」 
* 建立行動應用程式專案可能需要一點時間。 
* 在 API 應用程式佈建期間 (在某些情況下)，Azure API 會傳回錯誤，反映權限無法正確設定，但實際上 API 應用程式已正確佈建，並可供使用。 您可以使用 Azure 入口網站來手動設定權限。
* Application Insights 在 API 應用程式範本和行動應用程式範本上不受支援。
* API 應用程式專案無法與雲端服務專案搭配使用。
* API 應用程式專案範本僅適用於 C# 中。
* 只有在 C# 中，才能透過「新增 Azure API 應用程式用戶端」內容功能表來取用 API 應用程式。

