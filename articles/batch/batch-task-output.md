---
title: 從已完成的作業和工作將結果或記錄保存至資料存放區 - Azure Batch | Microsoft Docs
description: 深入了解從 Batch 工作和作業保存輸出資料的不同選項。 您可以將資料保存到 Azure 儲存體，或是另一個資料存放區。
services: batch
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 16e12d0e-958c-46c2-a6b8-7843835d830e
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 06/16/2017
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c0fdcdbf838a0bc283db05f36b900641016211b7
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43121909"
---
# <a name="persist-job-and-task-output"></a>持續作業及工作輸出

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

工作輸出的一些常見範例包括：

- 工作處理輸入資料時，會建立檔案。
- 與工作執行相關聯的記錄檔。 

本文說明保存工作輸出的各種選項，以及每個選項最合適的情節。   

## <a name="about-the-batch-file-conventions-standard"></a>關於 Batch 檔案慣例標準

Batch 會定義一組選擇性的慣例，可在 Azure 儲存體中命名工作輸出檔案。 [Batch 檔案慣例標準](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions) 描述這些慣例。 檔案慣例標準會以作業和工作名稱作為基礎，針對給定之輸出檔案，決定 Azure 儲存體中的目的地容器和 blob 路徑的名稱。

您可決定是否要使用檔案慣例標準來命名輸出資料檔案。 您也可以任何方式命名目的地容器和 blob。 如果您要針對命名輸出檔案使用檔案慣例標準，您的輸出檔案就可在 [Azure 入口網站][portal]中檢視。

您可以使用幾個不同的方式來使用檔案慣例標準：

- 如果您要使用 Batch 服務 API 來保存輸出檔案，可以選擇根據檔案慣例標準來命名目的地容器和 blob。 Batch 服務 API 可讓您保存用戶端程式碼中的輸出檔案，而不需修改工作應用程式。
- 如果您要使用 .NET 進行開發，可以使用[適用於 .NET 的 Batch 檔案慣例程式庫][nuget_package]。 使用此程式庫的優點，是它可支援查詢您的輸出檔案 (根據其識別碼或用途)。 內建查詢功能可讓您從用戶端應用程式或從其他工作輕鬆存取輸出檔案。 不過，必須修改您的工作應用程式，才能呼叫檔案慣例程式庫。 如需詳細資訊，請參閱[適用於 .NET 的檔案慣例程式庫](https://msdn.microsoft.com/library/microsoft.azure.batch.conventions.files.aspx)的參考。
- 如果您要使用 .NET 以外的語言進行開發，可以在您的應用程式中實作檔案慣例標準。

## <a name="design-considerations-for-persisting-output"></a>保存輸出的設計考量 

設計 Batch 方案的時候，請考慮與作業和工作輸出相關的下列因素。

* **計算節點存留期**：計算節點通常是過渡性的，尤其是在啟用自動調整的集區中。 只有當節點存在時，才能取得在該節點上執行的工作輸出，且僅能在您針對該工作所設定的保留期內才能取得。 如果工作所產生的輸出是在工作完成之後可能需要的，工作就必須將其輸出檔案上傳至持久的存放區，例如 Azure 儲存體。

* **輸出儲存體**：建議使用 Azure 儲存體作為工作輸出的資料存放區，但是您可以使用任何持久的儲存體。 Azure 儲存體的寫入工作輸出已整合到 Batch 服務 API。 如果您使用的是另一種持久儲存體，必須將應用程式邏輯寫入，才能自行保存工作輸出。   

* **輸出擷取**：如果您已保存工作輸出，就可以從集區中的計算節點，或從 Azure 儲存體或另一個資料存放區直接擷取工作輸出。 若要直接從計算節點擷取工作的輸出，您需要檔案名稱和該檔案在節點上的輸出位置。 如果您將工作輸出保存到 Azure 儲存體，就需要 Azure 儲存體中的檔案完整路徑，才能下載輸出檔案與 Azure 儲存體 SDK。

* **檢視輸出**：當您在 Azure 入口網站中瀏覽至 Batch 工作並選取 [節點上的檔案] 時，系統將會顯示與該工作相關聯的所有檔案，而不只是您感興趣的輸出檔案。 同樣地，只有當運算節點存在時，且僅在您針對該工作設定的保留期內，才能取得該節點的檔案。 若要檢視您已保存到 Azure 儲存體的工作輸出，可以使用 Azure 入口網站或 Azure 儲存體用戶端應用程式，例如 [Azure 儲存體總管][storage_explorer]。 若要使用入口網站或其他工具檢視 Azure 儲存體中的輸出資料，您必須知道檔案的位置，並直接瀏覽至該位置。

## <a name="options-for-persisting-output"></a>保存輸出的選項

依您的情節而定，可採取幾個不同的方法來保存工作輸出：

- 使用 Batch 服務 API。  
- 使用適用於 .NET 的 Batch 檔案慣例程式庫。  
- 實作應用程式中的 Batch 檔案慣例標準。
- 實作自訂檔案移動解決方案。

下列各節更詳細說明每一種方法。

### <a name="use-the-batch-service-api"></a>使用 Batch 服務 API

在 2017-05-01 版中，當您[將工作新增至作業](https://docs.microsoft.com/rest/api/batchservice/add-a-task-to-a-job)或[將工作集合新增至作業](https://docs.microsoft.com/rest/api/batchservice/add-a-collection-of-tasks-to-a-job)時，Batch 服務會新增支援在 Azure 儲存體中指定工作資料的輸出檔案。

Batch 服務 API 支援從使用虛擬機器設定建立的集區，將工作資料保存到 Azure 儲存體帳戶。 您可以使用 Batch 服務 API 來保存工作資料，而不需要修改您工作所執行的應用程式。 您可以選擇性地遵循 [Batch 檔案慣例標準](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)將您保存到 Azure 儲存體的檔案命名。 

使用 Batch 服務 API 來保存工作輸出的時機：

- 您需要保存的資料，是來自使用虛擬機器設定在集區中建立的 Batch 工作和作業管理員工作。
- 您需要將資料保存到具有任意名稱的 Azure 儲存體容器。
- 您需要將資料保存到根據 [Batch 檔案慣例標準](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)命名的 Azure 儲存體容器。 

> [!NOTE]
> Batch 服務 API 不支援的保存資料，是來自使用雲端服務設定建立之集區中執行的工作。 如需從執行雲端服務設定之集區保存工作輸出的相關資訊，請參閱[使用適用於 .NET 的 Batch 檔案慣例程式庫，將作業和工作資料保存到 Azure 儲存體](batch-task-output-file-conventions.md)
> 
> 

如需有關使用 Batch 服務 API 保存工作輸出的詳細資訊，請參閱[使用 Batch 服務 API 將工作資料保存到 Azure 儲存體](batch-task-output-files.md)。 另請參閱 GitHub 上的 [PersistOutputs][github_persistoutputs] 範例專案，示範如何使用適用於 .NET 的 Batch 用戶端程式庫，將工作輸出保存到永久儲存體。

### <a name="use-the-batch-file-conventions-library-for-net"></a>使用適用於 .NET 的 Batch 檔案慣例程式庫

使用 C# 和 .NET 建置 Batch 解決方案的開發人員可以使用 [適用於 .NET 的檔案慣例][nuget_package] ，根據 [Batch 檔案慣例標準](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)，將工作資料保存至 Azure 儲存體帳戶。 檔案慣例程式庫可處理將輸出檔案移動至 Azure 儲存體，並使用已知方法命名目的地容器和 blob。

檔案慣例程式庫支援依識別碼或用途查詢輸出檔案，讓您不需要完整的檔案 URI，就可以輕鬆地找到它們。 

使用適用於 .NET 的 Batch 檔案慣例程式庫保存工作輸出的時機：

- 您在工作還在執行時，想要將資料串流至 Azure 儲存體。
- 您想要保存的資料是在使用雲端服務設定或虛擬機器設定建立的集區中。
- 用戶端應用程式或作業中的其他工作必須依識別碼或依用途來尋找及下載工作輸出檔案。 
- 您想要執行檢查點或提前上傳初始結果。
- 您想要在 Azure 入口網站中檢視工作輸出。

如需使用適用於 .NET 的檔案慣例程式庫來保存工作輸出的詳細資訊，請參閱[使用適用於 .NET 的 Batch 檔案慣例程式庫，將作業和工作資料保存到 Azure 儲存體](batch-task-output-file-conventions.md)。 另請參閱 GitHub 上的 [PersistOutputs][github_persistoutputs] 範例專案，示範如何使用適用於 .NET 的檔案慣例程式庫，將工作輸出保存到永久儲存體。

GitHub 上的 [PersistOutputs][github_persistoutputs] 範例專案會示範如何使用適用於 .NET 的 Batch 用戶端程式庫，將工作輸出保存到永久儲存體。

### <a name="implement-the-batch-file-conventions-standard"></a>實作 Batch 檔案慣例標準

如果您要使用 .NET 以外的語言，可以在您自己的應用程式中實作 [Batch 檔案慣例標準](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)。 

當您需要經過證實的命名配置，或當想要在 Azure 入口網站中檢視工作輸出時，建議您自行實作檔案慣例命名標準。

### <a name="implement-a-custom-file-movement-solution"></a>實作自訂檔案移動解決方案

您也可以實作自己的完整檔案移動解決方案。 使用這個方法的時機：

- 您想要將工作資料保存到 Azure 儲存體以外的資料存放區。 若要將檔案上傳至諸如 Azure SQL 或 Azure Data Lake 等資料存放區，您可以建立自訂指令碼或可執行檔，以上傳至該位置。 接著，您在執行主要可執行檔之後，可以在命令列上呼叫它。 例如，在 Windows 節點上，您可能會呼叫這兩個命令：`doMyWork.exe && uploadMyFilesToSql.exe`
- 您想要執行檢查點或提前上傳初始結果。
- 您需要保持更精確地控制錯誤處理。 例如，如果您想要使用工作相依性動作，以特定的工作結束代碼作為基礎來採取某些上傳動作，您將需要實作自己的解決方案。 如需有關工作相依性動作的詳細資訊，請參閱[建立工作相依性，以執行相依於其他工作的工作](batch-task-dependencies.md)。 

## <a name="next-steps"></a>後續步驟

- 使用 Batch 服務 API 中的新功能進行瀏覽，可保存[使用 Batch 服務 API 將工作資料保存到 Azure 儲存體](batch-task-output-files.md)中的工作資料。
- 在[使用適用於 .NET 的 Batch 檔案慣例程式庫，將作業和工作資料保存到 Azure 儲存體](batch-task-output-file-conventions.md)中，了解如何使用適用於 .NET 的檔案慣例程式庫。
- 請參閱 GitHub 上的 [PersistOutputs][github_persistoutputs] 範例專案，示範如何使用適用於 .NET 的 Batch 用戶端程式庫和適用於 .NET 的檔案慣例程式庫，將工作輸出保存到永久儲存體。

[nuget_package]: https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files
[portal]: https://portal.azure.com
[storage_explorer]: http://storageexplorer.com/
[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs 
