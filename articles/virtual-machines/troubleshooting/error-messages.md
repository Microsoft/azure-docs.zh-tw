---
title: Azure 中常見的 VM 錯誤碼 | Microsoft Docs
description: 了解當您在 Azure 中佈建及管理虛擬機器時，會遇到的一些常見錯誤碼
services: virtual-machines
documentationcenter: ''
author: xujing-ms
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure
ms.date: 5/22/2017
ms.author: xujing
ms.openlocfilehash: b996d42e5c543235d09b46d29889bc5eaeafd52a
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411526"
---
# <a name="understand-common-error-messages-when-you-manage-virtual-machines-in-azure"></a>了解當您在 Azure 中管理虛擬機器時，常見的錯誤訊息

本文說明當您在 Azure 中建立或管理虛擬機器 (VM) 時，一些最常見的錯誤碼和訊息。

>[!NOTE]
> 您可以在此頁面上，或透過 [Azure 意見反應](https://feedback.azure.com/forums/216843-virtual-machines) 並利用 #azerrormessage 標記，發表對於錯誤訊息的意見反應。

## <a name="error-response-format"></a>錯誤回應的格式 
Azure VM 使用下列 JSON 格式來產生錯誤回應：

```json
{
  "status": "status code",
  "error": {
    "code":"Top level error code",
    "message":"Top level error message",
    "details":[
     {
      "code":"Inner evel error code",
      "message":"Inner level error message"
     }
    ]
   }
}
```

錯誤回應永遠包含狀態碼和錯誤物件。 每個錯誤物件永遠包含錯誤碼和訊息。 如果 VM 是利用範本建立，則錯誤物件也包含詳細區段，其中包含更內層的錯誤碼和訊息。 通常，最內層的錯誤訊息就是失敗根源。


## <a name="common-virtual-machine-management-errors"></a>常見的虛擬機器管理錯誤

本節列出管理 VM 時可能遇到的常見錯誤訊息：

|  錯誤碼  |  錯誤訊息  |  
|  :------| :-------------|  
|  AcquireDiskLeaseFailed  |  使用 URI 為 {1} 的 blob 建立磁碟 '{0}' 時，無法取得租用。 Blob 已在使用中。  |  
|  AllocationFailed  |  配置失敗。 請嘗試降低 VM 大小或 VM 數，然後稍候再試一次，或是部署到其他可用性設定組或其他 Azure 位置。  |  
|  AllocationFailed  |  VM 配置因內部錯誤而失敗。 請稍後重試，或嘗試部署至不同的位置。  |
|  ArtifactNotFound  |  在位置 '{2}' 中找不到發行者為 '{0}' 且類型為 '{1}' 的 VM 擴充功能。  |
|  ArtifactNotFound  |  在延伸模組存放庫中找不到發行者為 '{0}'、類型為 '{1}' 且類型處理常式版本為 '{2}' 的延伸模組。  |
|  ArtifactVersionNotFound  |  在構件存放庫中，找不到任何版本符合所要求的版本 '{0}'。  |
|  ArtifactVersionNotFound  |  在成品存放庫中，找不到任何版本符合所要求版本為 '{0}'、發行者為 '{1}'、類型為 '{2}' 的 VM 擴充功能。  |
|  AttachDiskWhileBeingDetached  |  無法將 '{0}' 的資料磁碟連結至 VM '{1}'，因為目前正在卸離磁碟。 請等到磁碟完全卸離，再試一次。  |
|  BadRequest  |  此區域尚不支援「對齊」可用性設定組。  |
|  BadRequest  |  不支援將具有受控磁碟的 VM 新增至未受管理的可用性設定組，或將具有 blob 磁碟的 VM 新增至受管理的可用性設定組。 請建立設有 'managed' 屬性的可用性設定組，以新增具有受控磁碟的 VM。  |
|  BadRequest  |  此區域不支援受控磁碟。  |
|  BadRequest  |  作業系統類型 '{0}' 不支援每個處理常式有多個 VMExtensions。 已新增或已在輸入中指定處理常式為 '{2}' 的 VMExtension '{1}'。  |
|  BadRequest  |  具受控磁碟的資源 '{1}' 不支援作業 '{0}'。  |
|  CertificateImproperlyFormatted  |  從 {0} 擷取之密碼 (Secret) 的 JSON 表示有一個資料欄位不是正確格式化的 PFX 檔案，或提供的密碼 (Password) 未正確地解碼 PFX 檔案。  |
|  CertificateImproperlyFormatted  |  無法將擷取自 {0} 的資料還原序列化為 JSON。  |
|  衝突  |  只有在建立 VM 時或解除配置 VM 之後，才可調整磁碟大小。  |
|  ConflictingUserInput  |  無法連結磁碟 '{0}'，因為 VM '{1}' 已擁有該磁碟。  |
|  ConflictingUserInput  |  來源與目的地資源群組相同。  |
|  ConflictingUserInput  |  磁碟 {0} 的來源和目的地儲存體帳戶不同。  |
|  ContainerAlreadyOnLease  |  保留 URI 為 {0} 之 blob 的儲存體容器上已有租用。  |
|  CrossSubscriptionMoveWithKeyVaultResources  |  移動資源要求所包含的 KeyVault 資源被要求中的一或多個 {0} 所參考。 跨訂用帳戶移動目前不支援此功能。 請查看 KeyVault 資源識別碼的錯誤詳細資料。  |
|  DiagnosticsOperationInternalError  |  處理 VM {0} 的診斷設定檔時發生內部錯誤。  |
|  DiskBlobAlreadyInUseByAnotherDisk  |  Blob {0} 正由另一個屬於 VM '{1}' 的磁碟使用。 您可以檢查 blob 中繼資料，以取得磁碟參考資訊。  |
|  DiskBlobNotFound  |  在磁碟 '{1}' 中找不到 URI 為 {0} 的 VHD blob。  |
|  DiskBlobNotFound  |  找不到 URI 為 {0} 的 VHD blob。  |
|  DiskEncryptionKeySecretMissingTags  |  {0} 秘密沒有 {1} 標記。 請更新密碼版本，並新增所需的標記，然後再試一次。  |
|  DiskEncryptionKeySecretUnwrapFailed  |  使用金鑰 {1} 將解除秘密 {0} 值的密封失敗。  |
|  DiskImageNotReady  |  磁碟映像 {0} 處於 {1} 狀態。 映像備妥時，請重試。  |
|  DiskPreparationError  |  準備 VM 磁碟時發生一或多個錯誤。 如需詳細資訊，請參閱磁碟執行個體檢視。  |
|  DiskProcessingError  |  因為 VM 有其他磁碟處於失敗狀態，磁碟處理已停止。  |
|  ImageBlobNotFound  |  在磁碟 '{1}' 中找不到 URI 為 {0} 的 VHD blob。  |
|  ImageBlobNotFound  |  找不到 URI 為 {0} 的 VHD blob。  |
|  IncorrectDiskBlobType  |  磁碟 blob 只可是分頁 blob 類型。 磁碟 '{1}' 的 blob {0} 為區塊 blob 類型。  |
|  IncorrectDiskBlobType  |  磁碟 blob 只可是分頁 blob 類型。 Blob {0} 是 '{1}' 類型。  |
|  IncorrectImageBlobType  |  磁碟 blob 只可是分頁 blob 類型。 磁碟 '{1}' 的 blob {0} 為區塊 blob 類型。  |
|  IncorrectImageBlobType  |  磁碟 blob 只可是分頁 blob 類型。 Blob {0} 是 '{1}' 類型。  |
|  InternalOperationError  |  無法解析儲存體帳戶 {0}。 請確定用來建立帳戶的儲存體資源提供者與計算資源位於相同的位置。  |
|  InternalOperationError  |  {0} 目標搜尋工作失敗。  |
|  InternalOperationError  |  驗證 VM '{0}' 的網路設定檔時發生錯誤。  |
|  InvalidAccountType  |  AccountType {0} 無效。  |
|  InvalidParameter  |  參數 {0} 的值無效。  |
|  InvalidParameter  |  不允許指定的系統管理員密碼。  |
|  InvalidParameter  |  提供的密碼長度必須介於 {0}-{1} 個字元之間，而且至少必須符合下列其中 {2} 項對密碼複雜性的要求： <ol><li> 包含大寫字元</li><li>包含小寫字元</li><li>包含數字</li><li>包含特殊字元。</li></ol>  |
|  InvalidParameter  |  不允許指定的系統管理員使用者名稱。  |
|  InvalidParameter  |  如果從平台或使用者映像建立 VM，將無法連結現有的作業系統磁碟。  |
|  InvalidParameter  |  容器名稱 {0} 無效。 容器名稱長度必須是 3-63 個字元，而且只能包含小寫英數字元和連字號。 連字號前後都必須緊接英數字元。  |
|  InvalidParameter  |  URL {1} 中的容器名稱 {0} 無效。 容器名稱長度必須是 3-63 個字元，而且只能包含小寫英數字元和連字號。 連字號前後都必須緊接英數字元。  |
|  InvalidParameter  |  URL {0} 中的 blob 名稱包含斜線。 磁碟目前不支援這種作法。  |
|  InvalidParameter  |  URI {0} 似乎不是正確的 blob URI。  |
|  InvalidParameter  |  名為 '{0}' 的磁碟已使用相同的 LUN：{1}。  |
|  InvalidParameter  |  名為 '{0}' 的磁碟已經存在。  |
|  InvalidParameter  |  無法為指定的映像參考中已定義的磁碟，指定使用者映像覆寫。  |
|  InvalidParameter  |  名為 '{0}' 的磁碟已使用相同的 VHD URL {1}。  |
|  InvalidParameter  |  指定的錯誤網域計數 {0} 必須介於 {1} 到 {2} 之間。  |
|  InvalidParameter  |  授權類型 {0} 無效。 有效授權類型為: Windows_Client 或 Windows_Server (區分大小寫)。  |
|  InvalidParameter  |  Linux 主機名稱長度不可超過 {0} 個字元，也不得包含下列字元：{1}。  |
|  InvalidParameter  |  由於 Linux 佈建代理程式中的已知問題，SSH 公開金鑰的目的地路徑目前僅限於其預設值 {0}。  |
|  InvalidParameter  |  已有磁碟的 LUN 為 {0}。  |
|  InvalidParameter  |  要求的訂用帳戶 {0} 必須與受管理磁碟識別碼內含的訂用帳戶 {1} 相符。  |
|  InvalidParameter  |  OSProfile 中的自訂資料必須以 Base64 編碼，而且長度上限為 {0} 個字元。  |
|  InvalidParameter  |  URL {0} 中的 Blob 名稱必須以 '{1}' 副檔名結尾。  |
|  InvalidParameter  |  '{0}' 不是有效擷取的 VHD blob 名稱前置詞。 有效的前置詞符合 regex '{1}'。  |
|  InvalidParameter  |  如果未佈建 VM 代理程式，則無法將憑證新增至您的 VM。  |
|  InvalidParameter  |  已有磁碟的 LUN 為 {0}。  |
|  InvalidParameter  |  無法建立 VM，因為目前已配置可用性設定組的叢集中沒有所要求的大小 {0} 可用。 可用的大小是：{1}。 請前往 https://aka.ms/azure-resizevm，深入了解 VM 的大小調整策略。  |
|  InvalidParameter  |  目前的區域中沒有所要求的 VM 大小 {0} 可用。 目前區域中可用的大小為：{1}。 請前往 https://aka.ms/azure-regions，深入了解每個區域中可用的 VM 大小。  |
|  InvalidParameter  |  目前的區域中沒有所要求的 VM 大小 {0} 可用。 請前往 https://aka.ms/azure-regions，深入了解每個區域中可用的 VM 大小。  |
|  InvalidParameter  |  Windows 系統管理員使用者名稱長度不可超過 {0} 個字元、以句號 (.) 結尾，或包含下列字元：{1}。  |
|  InvalidParameter  |  Windows 電腦名稱長度不可超過 {0} 個字元、完全是數字，或包含下列字元：{1}。  |
|  MissingMoveDependentResources  |  移動資源要求未包含所有相依的資源。 請查看遺漏資源識別碼的錯誤詳細資料。  |
|  MoveResourcesHaveInvalidState  |  移動資源要求包含無效儲存體帳戶相關聯的 VM。 請查看這些資源識別碼和參考的儲存體帳戶名稱的詳細資料。  |
|  MoveResourcesHavePendingOperations  |  移動資源要求包含已暫止作業的資源。 請查看這些資源識別碼的詳細資料。 暫止的作業完成之後，請重試作業。  |
|  MoveResourcesNotFound  |  移動資源要求包含找不到的資源。 請查看這些資源識別碼的詳細資料。  |
|  NetworkingInternalOperationError  |  不明的網路配置錯誤。  |
|  NetworkingInternalOperationError  |  不明的網路配置錯誤  |
|  NetworkingInternalOperationError  |  處理 VM 的網路設定檔時發生內部錯誤。  |
|  NotFound  |  找不到可用性設定組 {0}。  |
|  NotFound  |  要求中指定的來源虛擬機器 '{0}' 不存在於此 Azure 位置。  |
|  NotFound  |  找不到 id 為 {0} 的租用戶。  |
|  NotFound  |  找不到映像 {0}。  |
|  NotSupported  |  授權類型為 {0}，但映像 Blob {1} 並非來自內部部署。  |
|  OperationNotAllowed  |  無法刪除可用性設定組 {0}。 在刪除可用性設定組之前，請確定它未包含任何 VM。  |
|  OperationNotAllowed  |  不允許將可用性設定組 SKU 從「對齊」變更為「傳統」。  |
|  OperationNotAllowed  |  未執行 VM 時，無法修改 VM 中的擴充。  |
|  OperationNotAllowed  |  只在具有 Blob 磁碟的虛擬機器上才支援擷取動作。 使用「映像」資源的 API，從受控虛擬機器建立映像。  |
|  OperationNotAllowed  |  必須先成功建立映像 {1}，才能從該映像建立資源 {0}。  |
|  OperationNotAllowed  |  若已配置 VM，即無法更新 encryptionSettings。請在 VM 解除配置之後再試一次。  |
|  OperationNotAllowed  |  不支援將受控磁碟新增至具有 blob 磁碟的 VM。  |
|  OperationNotAllowed  |  最多允許 {0} 個資料磁碟連結至此大小的 VM。  |
|  OperationNotAllowed  |  不支援將 blob 磁碟新增至具有受控磁碟的 VM。  |
|  OperationNotAllowed  |  因為映像 '{1}' 已標記為待刪除，不允許對此映像執行作業 '{0}'。 您只能重試刪除作業 (或等候進行中的作業完成)。  |
|  OperationNotAllowed  |  因為已將 VM 一般化，不允許在.VM '{1}' 上執行作業 '{0}'。  |
|  OperationNotAllowed  |  因為還原點集合 '{1}' 已標示待刪除，所以不允許執行作業 '{0}'。  |
|  OperationNotAllowed  |  因為 VM 擴充 '{1}' 已標示為待刪除，所以不允許對其執行作業 '{0}'。 您只能重試刪除作業 (或等候進行中的作業完成)。  |
|  OperationNotAllowed  |  因為正在使用映像 '{2}' 佈建虛擬機器 '{1}'，所以不允許執行作業 '{0}'。  |
|  OperationNotAllowed  |  因為虛擬機器調整集 '{1}' 目前正在使用映像 '{2}'，所以不允許執行作業 '{0}'。  |
|  OperationNotAllowed  |  因為 VM '{1}' 已標示為待刪除，不允許在此 VM 上執行作業 '{0}'。 您只能重試刪除作業 (或等候進行中的作業完成)。  |
|  OperationNotAllowed  |  因為 VM 已解除配置或標示為已解除配置。，不允許在 VM '{1}' 上執行作業 '{0}'。  |
|  OperationNotAllowed  |  因為 VM '{1}' 正在執行，不允許對此 VM 執行作業 '{0}'。 若您是從客體作業系統關閉 VM，請明確關閉 VM。  |
|  OperationNotAllowed  |  因為 VM 未解除配置，所以不允許在 VM '{1}' 上執行作業 '{0}'。  |
|  OperationNotAllowed  |  因為 VM '{1}' 的延伸模組 '{2}' 處於失敗狀態，所以不允許在 VM 上不執行作業 '{0}'。  |
|  OperationNotAllowed  |  不允許在 VM '{1}' 上執行作業 '{0}'，因為有其他作業正在進行。  |
|  OperationNotAllowed  |  作業 '{0}' 會要求將虛擬機器 '{1}' 一般化。  |
|  OperationNotAllowed  |  VM 必須在執行中 (或設定成要執行)，才能執行此作業。  |
|  OperationNotAllowed  |  磁碟的大小為 {0} GB，小於映像中對應磁碟大小的 {1} GB。不得出現此狀況。  |
|  OperationNotAllowed  |  只有在建立 VM 擴展集時，才能新增處理常式 '{0}' 的 VM 擴展集擴充。  |
|  OperationNotAllowed  |  只有在刪除 VM 擴展集時，才能刪除處理常式 '{0}' 的 VM 擴展集擴充。  |
|  OperationNotAllowed  |  VM '{0}' 已在使用受控磁碟。  |
|  OperationNotAllowed  |  VM '{0}' 屬於「傳統」可用性設定組 '{1}'。 請將可用性設定組更新為使用「對齊」SKU，然後再試一次轉換。  |
|  OperationNotAllowed  |  從映像建立的 VM 不能有 blob 磁碟。 所有磁碟都必須是受控磁碟。  |
|  OperationNotAllowed  |  因為 VM 未一般化，無法完成擷取作業。  |
|  OperationNotAllowed  |  因為 VM '{0}' 的磁碟正在轉換成受控磁碟，不允許對此 VM 執行管理作業。  |
|  OperationNotAllowed  |  進行中作業正在將虛擬機器 {0} 的啟動狀態變更為 {1}。 請於稍後再執行作業 {2}。  |
|  OperationNotAllowed  |  無法新增或更新 VM。 現有的配置單位中可能沒有所要求的 VM 大小 {0} 可用。 請前往 https://aka.ms/azure-resizevm，深入了解 VM 的大小調整策略。  |
|  OperationNotAllowed  |  無法調整 VM 大小，因為目前已配置可用性設定組的叢集中沒有所要求的大小 {0} 可用。 可用的大小是：{1}。 請前往 https://aka.ms/azure-resizevm，深入了解 VM 的大小調整策略。  |
|  OperationNotAllowed  |  無法調整 VM 大小，因為目前已配置 VM 的叢集中沒有所要求的大小 {0} 可用。 若要將 VM 大小調整為 {1}，請解除配置 (這是 Azure 入口網站中的 [停止] 作業)，然後再試一次調整大小作業。 請前往 https://aka.ms/azure-resizevm，深入了解 VM 的大小調整策略。  |
|  OSProvisioningClientError  |  因為正在佈建客體 OS，所以 VM '{0}' 的 OS 佈建失敗。  |
|  OSProvisioningClientError  |  VM '{0}' 的 OS 佈建失敗。 錯誤詳細資料：{1} 請確認已正確備妥 (一般化) 映像。 <ul><li>適用於 Windows 的指示： https://azure.microsoft.com/documentation/articles/virtual-machines-windows-upload-image/  </li></ul> |
|  OSProvisioningClientError  |  產生 SSH 主機金鑰失敗。 錯誤詳細資料：{0}。 若要解決此問題，請確認 Linux 代理程式是否已正確設定。 <ul><li>您可以於此查看指示： https://docs.microsoft.com/azure/virtual-machines/extensions/agent-linux/ </li></ul> |
|  OSProvisioningClientError  |  對 VM 指定的使用者名稱不適用於此 Linux 散發套件。 錯誤詳細資料：{0}。  |
|  OSProvisioningInternalError  |  由於發生內部錯誤，VM '{0}' 的 OS 佈建失敗。  |
|  OSProvisioningTimedOut  |  VM '{0}'的 OS 佈建未於配置的時間內完成。 VM 仍可能成功完成佈建。 請稍後再查看佈建狀態。  |
|  OSProvisioningTimedOut  |  VM '{0}'的 OS 佈建未於配置的時間內完成。 VM 仍可能成功完成佈建。 請稍後再查看佈建狀態。 也請確認已正確備妥 (一般化) 映像。   <ul><li>適用於 Windows 的指示： https://azure.microsoft.com/documentation/articles/virtual-machines-windows-upload-image/ </li><li> 適用於 Linux 的指示： https://azure.microsoft.com/documentation/articles/virtual-machines-linux-capture-image/</li></ul>  |
|  OSProvisioningTimedOut  |  VM '{0}'的 OS 佈建未於配置的時間內完成。 不過，已偵測到 VM 客體代理程式正在執行。 這表示客體 OS 尚未備妥做為 VM 映像 (CreateOption=FromImage)。 若要解決此問題，請依現狀使用 VHD 並指定 CreateOption=Attach，或適當備妥它做為映像：   <ul><li>適用於 Windows 的指示： https://azure.microsoft.com/documentation/articles/virtual-machines-windows-upload-image/ </li><li> 適用於 Linux 的指示： https://azure.microsoft.com/documentation/articles/virtual-machines-linux-capture-image/</li></ul>  |
|  OverConstrainedAllocationRequest  |  選取的位置目前沒有所要求的 VM 大小可用。  |
|  ResourceUpdateBlockedOnPlatformUpdate  |  因為目前正在執行平台更新，所以無法更新資源。 請稍後再試一次。  |
|  StorageAccountLimitation  |  儲存體帳戶 '{0}' 不支援建立磁碟所需的分頁 blob。  |
|  StorageAccountLimitation  |  儲存體帳戶 '{0}' 超過其配置的配額。  |
|  StorageAccountLocationMismatch  |  無法解析儲存體帳戶 {0}。 請確定用來建立帳戶的儲存體資源提供者與計算資源位於相同的位置。  |
|  StorageAccountNotFound  |  找不到儲存體帳戶 {0}。 請確定儲存體帳戶未刪除，而且位於 VM 所在的同一 Azure 位置。  |
|  StorageAccountNotRecognized  |  請使用由儲存體資源提供者管理的儲存體帳戶。 不支援使用 {0}。  |
|  StorageAccountOperationInternalError  |  存取儲存體帳戶 {0} 時發生內部錯誤。  |
|  StorageAccountSubscriptionMismatch  |  儲存體帳戶 {0} 不屬於訂用帳戶 {1}。  |
|  StorageAccountTooBusy  |  儲存體帳戶 '{0}' 目前太忙碌。 請考慮使用另一個帳戶。  |
|  StorageAccountTypeNotSupported  |  磁碟 {0} 使用的 {1} 是 Blob 儲存體帳戶。 請建立「一般用途」的儲存體帳戶。  |
|  StorageAccountTypeNotSupported  |  儲存體帳戶 {0} 的類型為 {1}。 開機診斷支援 {2} 儲存體帳戶類型。  <ul><li>如果您使用進階儲存體帳戶來進行開機診斷，該錯誤便會發生。 如需詳細資訊，請參閱[如何使用開機診斷](boot-diagnostics.md)。 </li></ul> |
|  SubscriptionNotAuthorizedForImage  |  此訂用帳戶未獲授權。  |
|  TargetDiskBlobAlreadyExists  |  Blob {0} 已存在。 請提供其他 blob URI，以建立新的空白資料磁碟 '{1}'。  |
|  TargetDiskBlobAlreadyExists  |  由於目標映像 blob {0} 已存在，且未設定覆寫 VHD blob 的旗標，因此擷取作業無法繼續。 請刪除 blob 或設定覆寫 VHD blob 的旗標，然後重試。  |
|  TargetDiskBlobAlreadyExists  |  因為目標映像 blob {0} 有作用中的租用，所以擷取作業無法繼續。   |
|  TargetDiskBlobAlreadyExists  |  Blob {0} 已存在。 請提供其他 blob URI 做為磁碟 '{1}' 的目標。  |
|  TooManyVMRedeploymentRequests  |  VM '{0}' 或此 VM 的相同可用性設定組內的其他 VM 已收到過多部署要求。 請稍後重試。  |
|  VHDSizeInvalid  |  對於具有 blob {2} 的磁碟 '{1}'，指定的磁碟大小值 {0} 無效。 磁碟大小必須介於 {3} 和 {4} 之間。  |
|  VMAgentStatusCommunicationError  |  VM '{0}' 未回報 VM 代理程式或擴充的狀態。 請確認 VM 上有執行中的 VM 代理程式，並可對外建立Azure 儲存體的連線。  |
|  VMArtifactRepositoryInternalError  |  與構件存放庫通訊以擷取 VM 構件詳細資料時發生錯誤。  |
|  VMArtifactRepositoryInternalError  |  從構件存放庫擷取 VM 構件資料時發生內部錯誤。  |
|  VMExtensionHandlerNonTransientError  |  處理常式 '{0}' 回報 VM 擴充功能 '{1}' 失敗。終端機錯誤碼為 '{2}'，錯誤訊息：'{3}'  |
|  VMExtensionManagementInternalError  |  處理 VM 擴充 '{0}' 時發生內部錯誤。  |
|  VMExtensionManagementInternalError  |  準備 VM 擴充時發生多個錯誤。 如需詳細資訊，請參閱 VM 擴充執行個體檢視。  |
|  VMExtensionProvisioningError  |  VM 回報在處理擴充 '{0}' 時失敗。 錯誤訊息："{1}"。  |
|  VMExtensionProvisioningError  |  多個 VM 擴充無法佈建在 VM 上。 如需詳細資訊，請參閱 VM 擴充執行個體檢視。  |
|  VMExtensionProvisioningTimeout  |  VM 擴充 '{0}' 的佈建已逾時。擴充安裝可能花太長的時間，或無法取得擴充狀態。  |
|  VMMarketplaceInvalidInput  |  從非 Marketplace 映像建立虛擬機器不需要方案資訊，請移除要求中的方案資訊。 作業系統磁碟名稱是 {0}。  |
|  VMMarketplaceInvalidInput  |  購買資訊不符。 無法從 Marketplace 映像部署。 作業系統磁碟名稱是 {0}。  |
|  VMMarketplaceInvalidInput  |  要求中必須有方案資訊，才能從 Marketplace 映像建立虛擬機器。 作業系統磁碟名稱是 {0}。  |
|  VMNotFound  |  找不到 VM '{0}'。  |
|  VMRedeploymentFailed  |  因為發生內部錯誤，VM '{0}' 重新部署失敗。 請稍後重試。  |
|  VMRedeploymentTimedOut  |  未在分配的時間內完成 VM '{0}' 的重新部署。 可能在後來某個時候成功完成。 不然，您可以重試要求。  |
|  VMStartTimedOut  |  VM '{0}' 未在分配的時間內啟動。 VM 仍可能成功啟動。 請稍後再檢查啟動狀態。  |


## <a name="next-steps"></a>後續步驟
如果您需要更多協助，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。