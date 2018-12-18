---
title: Azure Machine Learning REST API 錯誤碼 | Microsoft Docs
description: Azure Machine Learning Web 服務上的作業可以傳回這些錯誤碼。
keywords: ''
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: 0923074b-3728-439d-a1b8-8a7245e39be4
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 11/16/2016
ms.openlocfilehash: fbedf355c6b1c45d581408a962b875544838087f
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836084"
---
# <a name="machine-learning-rest-api-error-codes"></a>Machine Learning REST API 錯誤碼
 
Azure Machine Learning Web 服務上的作業可以傳回下列錯誤碼。
 
## <a name="badargument-http-status-code-400"></a>BadArgument (HTTP 狀態碼 400)
 
提供的引數無效。
 
此類錯誤表示某處提供的引數無效。 對於傳遞至 web 服務的項目而言，這可能是 Azure 儲存體的認證或位置。 請查看 [詳細資料] 區段中的 [錯誤碼] 欄位，以診斷那個特定引數無效。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| BadParameterValue | 所提供的參數值不符合參數上的規則參數 |
| BadSubscriptionId | 用來評分的訂用帳戶識別碼並不是資源中顯示的識別碼 |
| BadVersionCall | 進行 API 呼叫時所傳遞的版本參數無效：{0}。 檢查 API 說明頁面，以便傳遞的正確版本並再試一次。 |
| BatchJobInputsNotSpecified | 未在要求搭配指定下列必要輸入：{0}。 請確定已指定所有的輸入資料，然後再試一次。 |
| BatchJobInputsTooManySpecified | 要求指定的輸入超過服務中所定義的輸入。 接受的輸入清單︰{0}。 請確定已正確指定所有的輸入資料，然後再試一次。 |
| BlobNameTooLong | 針對診斷輸出提供的 Azure Blob 儲存體路徑太長︰{0}。 縮短路徑並再試一次。 |
| BlobNotFound | 無法存取所提供的 Azure Blob - {0}。  Azure 錯誤訊息：{1}。 |
| ContainerIsEmpty | 未提供 Azure 儲存體容器名稱。 提供有效的容器名稱，然後再試一次。 |
| ContainerSegmentInvalid | 容器名稱無效。 提供有效的容器名稱，然後再試一次。 |
| ContainerValidationFailed | Blob 容器驗證失敗，發生下列錯誤︰{0}。 |
| DataTypeNotSupported | 提供不支援的資料類型。 提供有效的資料類型，然後再試一次。 |
| DuplicateInputInBatchCall | 批次要求無效。 無法同時指定單一或多個輸入。 從要求中移除其中一個項目，然後再試一次。 |
| ExpiryTimeInThePast | 所提供的到期時間已經過去︰{0}。 提供未來的到期時間 (UTC)，然後再試一次。 若永遠不要到期，將到期時間設為 NULL。 |
| IncompleteSettings | 診斷設定不完整。 |
| InputBlobRelativeLocationInvalid | 未提供 Azure 儲存體 Blob 名稱。 提供有效的 Blob 名稱，然後再試一次。 |
| InvalidBlob | Blob 的 Blob 規格無效：{0}。 確認連接字串 / 相對路徑或 SAS 權杖規格正確無誤，然後再試一次。 |
| InvalidBlobConnectionString | 針對其中一個輸入/輸出 Blob 指定的連接字串無效︰{0}。 請更正此錯誤，然後再試一次。 |
| InvalidBlobExtension | Blob 參考︰{0} 的副檔名無效或遺漏。 此輸出類型支援的副檔名為："{1}"。 |
| InvalidInputNames | 在要求中指定的服務輸入名稱無效：{0}。 請將輸入資料對應至正確的服務輸入，然後再試一次。 |
| InvalidOutputOverrideName | 無效的輸出覆寫名稱︰{0}。 服務沒有使用此名稱的輸出節點。 請傳入要覆寫的正確輸出節點名稱 (需區分大小寫)。 |
| InvalidQueryParameter | 無效的查詢參數 '{0}'。 {1} |
| MissingInputBlobInformation | Azure 儲存體 Blob 資訊遺失。 提供有效的連接字串和相對路徑或 URI，然後再試一次。 |
| MissingJobId | 未提供作業識別碼。 第一次提交作業時會傳回作業識別碼。 確認作業識別碼正確無誤，然後再試一次。 |
| MissingKeys | 未提供任何金鑰，或未提供其中一個主要或次要金鑰。 |
| MissingModelPackage | 未提供模型套件識別碼或模型套件。 提供有效的模型套件識別碼或模型套件，然後再試一次。 |
| MissingOutputOverrideSpecification | 要求遺漏輸出覆寫 {0} 的 Blob 規格。 請透過要求指定有效的 Blob 位置，或移除輸出規格 (如果不想要位置覆寫)。 |
| MissingRequestInput | Web 服務預期會有輸入，但未提供任何輸入。 確定根據模型中發佈的輸入連接埠提供有效的輸入，然後再試一次。 |
| MissingRequiredGlobalParameters | 未提供所有必要的 Web 服務參數。 確認模組所預期的參數正確無誤，然後再試一次。 |
| MissingRequiredOutputOverrides | 呼叫已加密的服務端點時，強制針對服務的所有輸出傳入輸出覆寫。 目前遺漏下列輸出的覆寫︰{0} |
| MissingWebServiceGroupId | 未提供 Web 服務群組識別碼。 提供有效的 Web 服務群組識別碼，然後再試一次。 |
| MissingWebServiceId | 未提供 Web 服務識別碼。 提供有效的 Web 服務識別碼，然後再試一次。 |
| MissingWebServicePackage | 未提供 Web 服務套件。 提供有效的 Web 服務套件，然後再試一次。 |
| MissingWorkspaceId | 未提供工作區識別碼。 提供有效的工作區識別碼，然後再試一次。 |
| ModelConfigurationInvalid | 模型套件中的模型組態無效。 確定模型組態包含輸出端點定義、標準錯誤端點和標準輸出端點，然後再試一次。 |
| ModelPackageIdInvalid | 模型套件識別碼無效。確認模型套件識別碼正確無誤，然後再試一次。 |
| RequestBodyInvalid | 未提供要求本文，或在還原序列化要求本文時發生錯誤。 |
| RequestIsEmpty | 未提供要求。 提供有效的要求，然後再試一次。 |
| UnexpectedParameter | 提供的參數並非預期。 確認所有參數名稱的拼寫都正確無誤，只傳遞預期的參數，然後再試一次。 |
| UnknownError | 未知的錯誤。 |
| UserParameterInvalid | {0} |
| WebServiceConcurrentRequestRequirementInvalid | 無法變更 {0} Web 服務的並行要求需求。 |
| WebServiceIdInvalid | 提供的 Web 服務識別碼無效。 Web 服務識別碼應該是有效的 guid。 |
| WebServiceTooManyConcurrentRequestRequirement | 無法將並行要求需求設定為 {0} 以上。 |
| WebServiceTypeInvalid | 提供的 Web 服務類型無效。 確認有效的 Web 服務類型正確無誤，然後再試一次。 有效的 Web 服務類型：{0}。 |
 
## <a name="baduserargument-http-status-code-400"></a>BadArgument (HTTP 狀態碼 400)
 
提供的使用者引數無效。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| InputMismatchError | 輸入資料不符合輸入連接埠結構描述。 |
| InputParseError | 剖析輸入向量失敗。  確認輸入向量具有正確的資料行數目和資料類型。  其他詳細資料：{0}。 |
| MissingRequiredGlobalParameters | 遺漏 Web 服務所預期的參數。 確認 Web 服務預期的所有必要參數都正確無誤，然後再試一次。 |
| UnexpectedParameter | 確認只傳遞 Web 服務所預期的必要參數，然後再試一次。 |
| UserParameterInvalid | {0} |
 
## <a name="invalidoperation-http-status-code-400"></a>InvalidOperation (HTTP 狀態碼 400)
 
此要求在目前內容中無效。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| CannotStartJob | 無法啟動作業，因為它處於 {0} 狀態。 |
| IncompatibleModel | 模型與要求版本不相容。 要求版本僅支援單一資料表輸出模型。 |
| MultipleInputsNotAllowed | 模型不允許多個輸入。 |
 
## <a name="libraryexecutionerror-http-status-code-400"></a>LibraryExecutionError (HTTP 狀態碼 400)
 
模組執行發生內部程式庫錯誤。
 
 
## <a name="moduleexecutionerror-http-status-code-400"></a>ModuleExecutionError (HTTP 狀態碼 400)
 
模組執行發生錯誤。
 
 
## <a name="webservicepackageerror-http-status-code-400"></a>WebServicePackageError (HTTP狀態碼 400)
 
Web 服務套件無效。 確認所提供的 Web 服務套件正確無誤，然後再試一次。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| FormatError | Web 服務套件的格式不正確。 詳細資料：{0} |
| RuntimesError | Web 服務套件圖形無效。 詳細資料：{0} |
| ValidationError | Web 服務套件圖形無效。 詳細資料：{0} |
 
## <a name="unauthorized-http-status-code-401"></a>未經授權 (HTTP 狀態碼 401)
 
要求未經授權存取資源。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| AdminRequestUnauthorized | 未經授權 |
| ManagementRequestUnauthorized | 未經授權 |
| ScoreRequestUnauthorized | 提供的認證無效。 |
 
## <a name="notfound-http-status-code-404"></a>NotFound (HTTP 狀態碼 404)
 
找不到資源。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| ModelPackageNotFound | 找不到模型套件。 確認模型套件識別碼正確無誤，然後再試一次。 |
| WebServiceIdNotFoundInWorkspace | 在此工作區下找不到 Web 服務。 webServiceId 與 workspaceId 不相符。 確認所提供的 Web 服務是工作區的一部分，然後再試一次。 |
| WebServiceNotFound | 找不到 Web 服務。 確認 Web 服務識別碼正確無誤，然後再試一次。 |
| WorkspaceNotFound | 找不到工作區。 確認工作區識別碼正確無誤，然後再試一次。 |
 
## <a name="requesttimeout-http-status-code-408"></a>RequestTimeout (HTTP 狀態碼 408)
 
無法在允許的時間內完成作業。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| RequestCanceled | 用戶端已取消要求。 |
| ScoreRequestTimeout | 執行要求已逾時。 |
 
## <a name="conflict-http-status-code-409"></a>衝突 (HTTP 狀態碼 409)
 
資源已經存在。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| ModelOutputMetadataMismatch | 輸出參數名稱無效。 嘗試使用中繼資料編輯器模組來重新命名資料行，然後再試一次。 |
 
## <a name="memoryquotaviolation-http-status-code-413"></a>MemoryQuotaViolation (HTTP 狀態碼 413)
 
模型已超出指派給它的記憶體配額。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| OutOfMemoryLimit | 模型消耗超出其適用的記憶體。 模型允許的最大記憶體為 {0} MB。 請檢查您的模型是否有問題。 |
 
## <a name="internalerror-http-status-code-500"></a>InternalError (HTTP 狀態碼 500)
 
執行發生內部錯誤。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| AdminAuthenticationFailed |  |
| BackendArgumentError |  |
| BackendBadRequest |  |
| ClusterConfigBlobMisconfigured |  |
| ContainerProcessTerminatedWithSystemError | 容器程序因系統錯誤而當機 |
| ContainerProcessTerminatedWithUnknownError | 容器程序因不明錯誤而當機 |
| ContainerValidationFailed | Blob 容器驗證失敗，發生下列錯誤︰{0}。 |
| DeleteWebServiceResourceFailed |  |
| ExceptionDeserializationError |  |
| FailedGettingApiDocument |  |
| FailedStoringWebService |  |
| InvalidMemoryConfiguration | InvalidMemoryConfiguration，ConfigValue：{0} |
| InvalidResourceCacheConfiguration |  |
| InvalidResourceDownloadConfiguration |  |
| InvalidWebServiceResources |  |
| MissingTaskInstance | 未提供引數。 確認已傳遞有效的引數，然後再試一次。 |
| ModelPackageInvalid |  |
| ModuleExecutionFailed |  |
| ModuleLoadFailed |  |
| ModuleObjectCloneFailed |  |
| OutputConversionFailed |  |
| PortDataTypeNotSupported | 連接埠識別碼={0} 具有不支援的資料類型：{1}。 |
| ResourceDownload |  |
| ResourceLoadFailed |  |
| ServiceUrisNotFound |  |
| SwaggerGeneration | Swagger 產生失敗，詳細資料︰{0} |
| UnexpectedScoreStatus |  |
| UnknownBackendErrorResponse |  |
| UnknownError |  |
| UnknownJobStatusCode | 未知的作業狀態碼 {0}。 |
| UnknownModuleError |  |
| UpdateWebServiceResourceFailed |  |
| WebServiceGroupNotFound |  |
| WebServicePackageInvalid | InvalidWebServicePackage，詳細資料︰{0} |
| WorkerAuthorizationFailed |  |
| WorkerUnreachable |  |
 
## <a name="internalerrorsystemlowonmemory-http-status-code-500"></a>InternalErrorSystemLowOnMemory (HTTP 狀態碼 500)
 
執行發生內部錯誤。 系統記憶體不足。 請再試一次。
 
 
## <a name="modelpackageformaterror-http-status-code-500"></a>ModelPackageFormatError (HTTP 狀態碼 500)
 
模型套件無效。 確認所提供的模型套件正確無誤，然後再試一次。
 
 
## <a name="webservicepackageinternalerror-http-status-code-500"></a>WebServicePackageInternalError (HTTP 狀態碼 500)
 
Web 服務套件無效。 確認所提供的 Web 套件正確無誤，然後再試一次。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| ModuleError | Web 服務套件圖形無效。 詳細資料：{0} |
 
## <a name="initializingcontainers-http-status-code-503"></a>InitializingContainers (HTTP 狀態碼 503)
 
因為容器正在初始化，所以無法執行要求。
 
 
## <a name="serviceunavailable-http-status-code-503"></a>ServiceUnavailable (HTTP 狀態碼 503)
 
服務暫時無法使用。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| NoMoreResources | 要求沒有可用的資源。 |
| RequestThrottled | 要求已針對 {0} 端點進行節流處理。 端點的並行處理上限是 {1}。 |
| TooManyConcurrentRequests | 傳送的並行要求太多。 |
| TooManyHostsBeingInitialized | 同時初始化太多的主機。 考慮節流 / 重試。 |
| TooManyHostsBeingInitializedPerModel | 同時初始化太多的主機。 考慮節流 / 重試。 |
 
## <a name="gatewaytimeout-http-status-code-504"></a>GatewayTimeout (HTTP 狀態碼 504)
 
無法在允許的時間內完成作業。
 
| 錯誤碼 | 使用者訊息 |
| ---------- |--------------|
| BackendInitializationTimeout | 無法在允許的時間內完成 Web 服務初始化。 |
| BackendScoreTimeout | 無法在允許的時間內完成 Web 服務要求執行。 |
 
