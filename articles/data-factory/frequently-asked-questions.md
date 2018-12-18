---
title: Azure Data Factory：常見問題集 | Microsoft Docs
description: 獲得 Azure Data Factory 常見問題的解答。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 532dec5a-7261-4770-8f54-bfe527918058
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: shlo
ms.openlocfilehash: 73bc8b6954470d11d6369bc733bb7c6f794ce892
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45577127"
---
# <a name="azure-data-factory-faq"></a>Azure Data Factory 常見問題集
本文提供 Azure Data Factory 常見問題集的解答。  

## <a name="what-is-azure-data-factory"></a>Azure 資料處理站是什麼？ 
Data Factory 是完全受控並以雲端為基礎的資料整合服務，用來自動化資料的移動和轉換。 就像會運轉設備以將原物料轉換成成品的工廠一樣，Azure Data Factory 會協調現有的服務來收集未經處理資料，並將其轉換成隨時可用的資訊。 

您可使用 Azure Data Factory 建立資料驅動工作流程，以在內部部署與雲端資料存放區之間移動資料。 可使用 Azure HDInsight、Azure Data Lake Analytics 以及 SQL Server Integration Services (SSIS) 整合執行階段等計算服務來處理和轉換資料。 

透過 Data Factory，能夠在以 Azure 為基礎的雲端服務上執行資料處理，也可以使用自己的自我託管計算環境 (例如 SSIS、SQL Server 或 Oracle)。 建立執行所需動作的管線後，可以將該管線排程為定期 (例如每小時、每天或每週) 執行，可利用時間範圍排程或經由事件發生來觸發管線。 如需詳細資訊，請參閱 [Azure Data Factory 簡介](introduction.md)。

### <a name="control-flows-and-scale"></a>控制流程和規模 
為了支援現代資料倉儲中的各種整合流程和模式，Data Factory 讓內含完整控制流程程式設計的範例的彈性資料管線得以模型化。這些範例包含條件式執行、在資料管線中旳分支，以及明確地將在這些流程內部與之間傳遞參數。 控制流程也包含透過活動分派，將資料轉換至成外部執行引擎和資料流程功能，包括透過複製活動大規模移動資料。

Data Factory 可讓您自由地為資料整合所需的任何流程樣式建立模型。這些流程樣式可依需求分派或是依排程重複分派。 以下為此模型支援的一些常見流程：   

- 控制流程：
    - 將管線內的活動循序鏈結。
    - 將管線內的活動分支。
    - 參數
        - 定義管線層級的參數並傳遞引數，同時依需求或透過觸發程序來叫用管線。
        - 活動可以取用傳遞給管線的引數。
    - 自訂狀態傳遞
        - 活動輸出 (包括狀態) 可供管線中的後續活動取用。
    - 迴圈容器
        - For-each 
- 以觸發程序為基礎的流程：
    - 可依需求或依時鐘時間觸發管線。
- 差異流程：
    - 在從內部部署環境或雲端中的關聯式存放區移動維度或參考資料表，以將資料載入到資料湖中時，使用參數並為差異複本定義上限標準標記。 

如需詳細資訊，請參閱[教學課程：控制流程](tutorial-control-flow.md)。

### <a name="transform-your-data-at-scale-with-code-free-pipelines"></a>透過無程式碼管線大規模轉換資料
以瀏覽器為基礎的工具可讓使用者透過新式的互動式網頁，進行無程式碼管線的撰寫與開發。

對於視覺化資料開發人員及資料工程師而言，ADF Web UI 是可用於建置管線的無程式碼設計環境。 該環境與 Visual Studio Online Git 完全整合，亦可整合 CI/CD，並提供有偵錯選項的反覆式開發法。

### <a name="rich-cross-platform-sdks-for-advanced-users"></a>適用於進階使用者的豐富跨平台 SDK
如果您是進階使用者且正在尋找程式設計介面，ADF V2 提供一組豐富的 SDK，讓您能使用慣用的 IDE 來撰寫、管理及監視管線
1.  Python SDK
2.  Powershell CLI
3.  C# SDK 使用者也可以利用記載的 REST API 作為 ADF V2 的介面

### <a name="iterative-development-and-debugging-using-visual-tools"></a>使用視覺工具進行反覆式開發與偵錯
Azure Data Factory (ADF) 視覺工具可讓您進行反覆式開發與偵錯。 您可以使用管線畫布中的 [偵錯] 功能來建立管線和執行測試回合，無須撰寫任何一行程式碼。 您可以在管線畫布的 [輸出] 視窗中檢視測試回合的結果。 測試回合成功後，您可以將更多活動新增至您的管線，並繼續反覆地偵錯。 您也可以取消進行中的測試回合。 在按一下 [偵錯] 前，您不需先將變更發佈到資料處理站服務。 如果您想要在開發、測試或生產環境中更新資料處理站工作流程前，先確定新增項目或變更如預期般運作，這項功能便很有幫助。 

### <a name="deploy-ssis-packages-to-azure"></a>將 SSIS 套件部署至 Azure 
如果您想要移動 SSIS 工作負載，可以建立 Data Factory，然後佈建 Azure-SSIS 整合執行階段。 Azure-SSIS 整合執行階段是一個完全受控的 Azure VM (節點) 叢集，專門用來執行您的雲端 SSIS 封裝。 如需逐步指示，請參閱教學課程：[將 SSIS 封裝部署至 Azure](tutorial-create-azure-ssis-runtime-portal.md)。 
 
### <a name="sdks"></a>SDK
如果您是進階使用者且正在尋找程式設計介面，ADF 提供一組豐富的 SDK，讓您能使用慣用的 IDE 來撰寫、管理及監視管線。 .NET、PowerShell、Python 和 REST 皆提供語言支援。

### <a name="monitoring"></a>監視
您可以在瀏覽器使用者介面中透過 PowerShell、SDK 或視覺監視工具來監視自己的 Data Factory。 您可以使用高效率且有效的方式，來監視與管理依需求、觸發及時間驅動的自訂流程。 取消現有工作、一覽失敗作業、向下切入以取得詳細錯誤訊息，以及偵錯錯誤，這一切都只需在同一個畫面上就能完成，無須切換環境或在畫面之間來回瀏覽。 

### <a name="new-features-for-ssis-in-adf"></a>ADF 中 SSIS 適用的新功能
自 2017 年首次發行公開預覽版起，Data Factory 已為 SSIS 新增下列功能：

-   支援超過三個 Azure SQL 資料庫 (DB) 設定/變體，以主控專案/封裝 (SSISDB) 的 SSIS 目錄：
-   含 VNet 服務端點的 Azure SQL DB
-   受控執行個體 (MI)
-   彈性集區
-   除了支援 Classic VNet 未來將遭到取代的 Classic VNet 以外，也支援 Azure Resource Manager 虛擬網路 (VNet)。這可讓您將 Azure-SSIS Integration Runtime (IR) 插入/加入 VNet，該 VNet 針對具有 服務端點/MI/內部部署資料存取權的 Azure SQL DB 進行設定，請參閱： https://docs.microsoft.com/azure/data-factory/join-azure-ssis-integration-runtime-virtual-network 
-   除了支援 SQL 驗證，也支援透過 Azure Active Directory (AAD) 驗證連線至 SSISDB。這可讓您使用 AAD 驗證證明您的 ADF 受控服務識別 (MSI)
-   支援攜帶您自己的內部部署 SQL Server 授權，大幅節省採用 Azure Hybrid Benefit (AHB) 的成本
-   支援 Enterprise Edition of Azure-SSIS IR，讓您使用進階/高級功能、可安裝額外元件/擴充功能的自訂設定，以及第三方生態系統，請參閱： https://blogs.msdn.microsoft.com/ssis/2018/04/27/enterprise-edition-custom-setup-and-3rd-party-extensibility-for-ssis-in-adf/ 
-   SSIS 更深入地整合至 ADF ，這可讓您在 ADF 管線中叫用/觸發第一級 Execute SSIS Package 活動，並透過 SSMS 排程這些活動，請參閱： https://blogs.msdn.microsoft.com/ssis/2018/05/23/modernize-and-extend-your-etlelt-workflows-with-ssis-activities-in-adf-pipelines/ 


## <a name="what-is-integration-runtime"></a>什麼是整合執行階段？
整合執行階段是 Azure Data Factory 所使用的計算基礎結構，可跨各種網路環境提供下列資料整合功能：

- **資料移動**：若要移動資料，Integration Runtime 會在來源與目的地資料存放區之間移動資料，並且支援內建連接器、格式轉換、資料行對應，以及高效能且可調整規模的資料轉送作業。
- **分派活動**：若要轉換，Integration Runtime 提供可原生執行 SSIS 套件的功能。
- **執行 SSIS 封裝**：在受控 Azure 計算環境中原生執行 SSIS 封裝。 Integration Runtime 也支援分派和監視在 Azure HDInsight、Azure Machine Learning、Azure SQL Database、SQL Server 等各種計算服務上執行的轉換活動。

您可視需要部署一或多個整合執行階段執行個體，以移動和轉換資料。 整合執行階段可以在 Azure 公用網路中執行，或是在私人網路 (內部部署、Azure 虛擬網路或 Amazon Web Services 虛擬私人雲端 [VPC]) 中執行。 

如需詳細資訊，請參閱 [Azure Data Factory 中的整合執行階段](concepts-integration-runtime.md)。

## <a name="what-is-the-limit-on-the-number-of-integration-runtimes"></a>整合執行階段數目的限制為何？
您在資料處理站中可擁有多少個整合執行階段個體，並無硬性限制。 不過，整合執行階段可根據訂用帳戶用於 SSIS 套件封裝的虛擬機器核心數目，將會受到限制。 如需詳細資訊，請參閱 [Data Factory 限制](../azure-subscription-service-limits.md#data-factory-limits)。

## <a name="what-are-the-top-level-concepts-of-azure-data-factory"></a>Azure Data Factory 的最上層概念為何？
Azure 訂用帳戶可能會有一或多個 Azure Data Factory 執行個體 (或資料處理站)。 Azure Data Factory 是由四個重要元件所組成，這些元件會一起運作，以提供平台讓您撰寫具有資料移動和轉換步驟的資料驅動工作流程。

### <a name="pipelines"></a>管線
資料處理站可以有一或多個管線。 管線是一個執行某個單位工作的活動邏輯群組。 管線中的活動會合作執行一項工作。 例如，管線可能包含一組活動，以從 Azure Blob 內嵌資料，然後對 HDInsight 叢集執行 Hive 查詢來分割資料。 其中的優勢在於，您可使用管線按組別管理活動，而無須個別管理每個活動。 您可將管線中的活動鏈結在一起以循序方式運作，或是以平行方式獨立運作。

### <a name="activity"></a>活動
活動代表管線中的處理步驟。 例如，您可以使用*複製*活動，將資料從某個資料存放區複製到另一個資料存放區。 同樣地，您可以使用在 Azure HDInsight 叢集上執行 Hive 查詢的 Hive 活動，來轉換或分析您的資料。 Data Factory 支援三種類型的活動︰資料移動活動、資料轉換活動，以及控制活動。

### <a name="datasets"></a>資料集
資料集代表資料存放區中的資料結構，其只會指出或參考您要在活動中作為輸入或輸出的資料。 

### <a name="linked-services"></a>連結的服務
已連結的服務非常類似連接字串，可定義 Data Factory 連接到外部資源所需的連線資訊。 這麼說吧：連結的服務會定義與資料來源的連線，而資料集代表資料的結構。 例如，Azure 儲存體連結的服務會指定連接字串以連線到 Azure 儲存體帳戶。 而且，Azure Blob 資料集會指定包含資料的 Blob 容器和資料夾。

Data Factory 中的連結服務，有兩個用途：

- 用來代表*資料存放區*，其包含 (但不限於) 內部部署 SQL Server 執行個體、Oracle 資料庫執行個體、檔案共用或 Azure Blob 儲存體帳戶。 如需支援的資料存放區清單，請參閱 [Azure Data Factory 中的複製活動](copy-activity-overview.md)。
- 用來代表可裝載活動執行的 *計算資源* 。 例如，HDInsight Hive 活動會在 HDInsight Hadoop 叢集上執行。 如需轉換活動和受支援計算環境的清單，請參閱 [Azure Data Factory 中的資料轉換](transform-data.md)。

### <a name="triggers"></a>觸發程序
觸發程序代表用來決定何時需要啟動管線執行的處理單位。 針對不同類型的事件，有不同類型的觸發程序。 

### <a name="pipeline-runs"></a>管線執行
管線執行是管線執行的執行個體。 通常會藉由將引數傳遞給管線中定義的參數，來具現化管線執行。 您可以藉由手動方式，或是在觸發程序定義內傳遞引數。

### <a name="parameters"></a>參數
參數是唯讀設定的機碼值組。 您可在管線中定義參數，並在執行內容的過程中傳遞已定義參數的引數。 執行內容是由觸發程序，或是您手動執行的管線所建立。 管線內的活動會取用參數值。

資料集是一種強型別參數，亦是可重複使用或參考的實體。 活動可以參考資料集，且可取用資料集定義中所定義的屬性。

連結服務亦是一種強型別參數，其包含資料存放區或計算環境的連線資訊。 其亦是可重複使用或參考的實體。

### <a name="control-flows"></a>控制流程
控制流程負責協調管線活動，其中包括將活動循序鏈結、分支，以及可在管線層級定義的參數，還有依需求或從觸發程序叫用管線時所傳遞的引數。 控制流程也包括自訂狀態傳遞和迴圈容器 (亦即 for-each 列舉程式)。


如需 Data Factory 概念的詳細資訊，請參閱下列文章：

- [資料集和連結服務](concepts-datasets-linked-services.md)
- [管線和活動](concepts-pipelines-activities.md)
- [整合執行階段](concepts-integration-runtime.md)

## <a name="what-is-the-pricing-model-for-data-factory"></a>Data Factory 的定價模型為何？
請參閱 [Data Factory 定價詳細資料](https://azure.microsoft.com/pricing/details/data-factory/)頁面，以瞭解 Azure Data Factory 的定價詳細資料。

## <a name="how-can-i-stay-up-to-date-with-information-about-data-factory"></a>如何掌握 Data Factory 的最新資訊？
移至下列網站掌握 Azure Data Factory 的最新資訊：

- [部落格](https://azure.microsoft.com/blog/tag/azure-data-factory/)
- [文件首頁](/azure/data-factory)
- [產品首頁](https://azure.microsoft.com/services/data-factory/)

## <a name="technical-deep-dive"></a>深入技術性討論 

### <a name="how-can-i-schedule-a-pipeline"></a>如何排程管線？ 
您可以利用排程器觸發程序，或時間範圍觸發程序來排程管線。 觸發程序使用時鐘行事曆排程，並可讓您透過定期方式或使用以行事曆為基礎的週期性模式，來排程管線 (例如，每週星期一下午 6 時和星期四下午 9 時)。 如需詳細資訊，請參閱[管道執行和觸發程序](concepts-pipeline-execution-triggers.md)。

### <a name="can-i-pass-parameters-to-a-pipeline-run"></a>可以將參數傳遞給執行的管線嗎？
可以，參數是 ADF 第一級的最上層概念。 您可定義管線層級的參數並傳遞引數，同時依需求或使用觸發程序來執行管線執行。  

### <a name="can-i-define-default-values-for-the-pipeline-parameters"></a>可以定義管線參數的預設值嗎？ 
是。 您可以定義管線中參數的預設值。 

### <a name="can-an-activity-in-a-pipeline-consume-arguments-that-are-passed-to-a-pipeline-run"></a>管線中的活動可否取用傳遞給管線執行的引數？ 
是。 管線內的每個活動皆可使用 `@parameter` 建構，來取用傳遞給管線執行的參數值。 

### <a name="can-an-activity-output-property-be-consumed-in-another-activity"></a>可以在另一個活動中取用活動輸出屬性？ 
是。 可以使用 `@activity` 建構在後續活動中取用活動輸出。
 
### <a name="how-do-i-gracefully-handle-null-values-in-an-activity-output"></a>如何正常處理活動輸出中的 Null 值？ 
您可以在運算式中使用 `@coalesce` 建構來正常處理 Null 值。 

## <a name="next-steps"></a>後續步驟
如需建立資料處理站的逐步指示，請參閱下列教學課程：

- [快速入門：建立 Data Factory](quickstart-create-data-factory-dot-net.md)
- [教學課程：複製雲端中的資料](tutorial-copy-data-dot-net.md)
