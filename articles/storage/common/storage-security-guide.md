---
title: Azure 儲存體安全性指南 | Microsoft Docs
description: 詳述許多保護 Azure 儲存體的方法，包括但不限於 RBAC、儲存體服務加密、用戶端加密、SMB 3.0 及 Azure 磁碟加密。
services: storage
author: craigshoemaker
ms.service: storage
ms.topic: article
ms.date: 05/31/2018
ms.author: cshoe
ms.component: common
ms.openlocfilehash: 268459fde7a622079656e637d2c51562cea358a4
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45730944"
---
# <a name="azure-storage-security-guide"></a>Azure 儲存體安全性指南

「Azure 儲存體」提供一組完整的安全性功能，這些功能一起可讓開發人員建置安全的應用程式：

- 所有寫入至「Azure 儲存體」的資料都會使用[儲存體服務加密 (SSE)](storage-service-encryption.md)來自動加密。 如需詳細資訊，請參閱[宣佈 Azure Blob、檔案、資料表及佇列儲存體的預設加密](https://azure.microsoft.com/blog/announcing-default-encryption-for-azure-blobs-files-table-and-queue-storage/) \(英文\)。
- Azure Active Directory (Azure AD) 和角色型存取控制 (RBAC) 可支援 Azure 儲存體，以便進行資源管理作業和資料作業，如下所示：   
    - 您可以將儲存體帳戶範圍內的 RBAC 角色指派給安全性主體，以及使用 Azure AD 來授權資源管理作業，例如金鑰管理。
    - Blob 和佇列服務的資料作業預覽版支援 Azure AD 整合。 您可以將訂用帳戶、資源群組、儲存體帳戶或個別的容器或佇列範圍內的 RBAC 角色，指派給安全性主體或受控服務識別。 如需詳細資訊，請參閱[使用 Azure Active Directory 來驗證 Azure 儲存體的存取權 (預覽)](storage-auth-aad.md)。   
- 您可以使用[用戶端加密](../storage-client-side-encryption.md)、HTTPS 或 SMB 3.0，在應用程式和 Azure 之間進行傳輸時保護資料的安全。  
- 您可以使用 [Azure 磁碟加密](../../security/azure-security-disk-encryption.md)來加密 Azure 虛擬機器所使用的 OS 和資料磁碟。 
- Azure 儲存體中資料物件的委派存取權可以使用 [共用存取簽章](../storage-dotnet-shared-access-signature-part-1.md)來授與。

此文章簡要說明這些安全性功能當中，可與「Azure 儲存體」搭配使用的每個安全性功能。 所提供的文章連結將提供每個功能的詳細資料，讓您能夠輕鬆地進一步調查每個主題。

以下是此文章所涵蓋的主題：

* [管理平面安全性](#management-plane-security) – 保護儲存體帳戶

  管理平面包含用來管理儲存體帳戶的資源。 此節涵蓋 Azure Resource Manager 部署模型，以及如何使用「角色型存取控制」(RBAC) 來控制對儲存體帳戶的存取。 此外，也說明如何管理儲存體帳戶金鑰，以及重新產生這些金鑰。
* [資料平面安全性](#data-plane-security) – 保護資料的存取

  此節將探討如何在儲存體帳戶 (例如 Blob、檔案、查詢及表格) 中，允許使用共用存取簽章和預存存取原則來存取實際的資料物件。 我們將涵蓋服務層級的 SAS 和帳戶層級的 SAS。 我們也會看到如何限制存取特定的 IP 位址 (或 IP 位址範圍)、如何限制用於 HTTPS 的通訊協定，以及如何撤銷共用存取簽章，而不需等到它過期。
* [傳輸中加密](#encryption-in-transit)

  此節討論如何在將資料傳輸至 Azure 儲存體或從中傳出時提供保護。 我們將討論 HTTPS 的建議用法，以及 SMB 3.0 針對 Azure 檔案共用所使用的加密。 同時也會探討用戶端加密，可讓您在將資料傳輸至用戶端應用程式中的儲存體之前加密資料，以及自儲存體傳出後解密資料。
* [待用加密](#encryption-at-rest)

  我們將探討「儲存體服務加密」(SSE)，現在針對新的和現有的儲存體帳戶都會自動啟用此功能。 同時也將探討如何使用 Azure 磁碟加密，並探索磁碟加密與 SSE 與用戶端加密之間的基本差異和案例。 我們將簡短探討美國政府電腦適用的 FIPS 相符性。
* 使用 [儲存體分析](#storage-analytics) 稽核 Azure 儲存體的存取

  此節討論如何在儲存體分析記錄中尋找某個要求的相關資訊。 我們將查看實際的分析記錄資料，並了解如何分辨出要求是否是利用儲存體帳戶金鑰、共用存取簽章或匿名方式所提出，以及該要求是否成功或失敗。
* [使用 CORS 啟用以瀏覽器為基礎的用戶端](#Cross-Origin-Resource-Sharing-CORS)

  此節討論如何允許跨原始來源資源共用 (CORS)。 我們將討論跨網域存取，以及如何使用 Azure 儲存體內建的 CORS 功能來處理它。

## <a name="management-plane-security"></a>管理平面安全性
管理平面包含會影響儲存體帳戶本身的作業。 例如，您可以建立或刪除儲存體帳戶、取得訂用帳戶中的儲存體帳戶清單、擷取儲存體帳戶金鑰，或重新產生儲存體帳戶金鑰。

當您建立新的儲存體帳戶時，可以選取傳統或資源管理員的部署模型。 在 Azure 中建立資源的傳統模型只允許以孤注一擲的方式存取訂用帳戶，接著存取儲存體帳戶。

本指南將焦點放在 Resource Manager 模型，這是建立儲存體帳戶的建議方法。 使用 Resource Manager 儲存體帳戶，而不是提供整個訂用帳戶的存取權，您可以使用角色型存取控制 (RBAC)，來控制更限定層級上對管理平面的存取。

### <a name="how-to-secure-your-storage-account-with-role-based-access-control-rbac"></a>如何使用角色型存取控制 (RBAC) 保護儲存體帳戶
讓我們討論何謂 RBAC 及使用方式。 每一個 Azure 訂用帳戶都具有 Azure Active Directory。 您可以為來自該目錄的使用者、群組和應用程式授與存取權，以便在使用資源管理員部署模型的 Azure 訂用帳戶中管理資源。 這種安全性稱為「角色型存取控制」(RBAC)。 若要管理此存取權，您可以使用 [Azure 入口網站](https://portal.azure.com/)、[Azure CLI 工具](../../cli-install-nodejs.md)、[PowerShell](/powershell/azureps-cmdlets-docs) 或 [Azure 儲存體資源提供者 REST API](https://msdn.microsoft.com/library/azure/mt163683.aspx)。

使用資源管理員模型，您可以將儲存體帳戶放置於資源群組中，並使用 Azure Active Directory 來控制該特定儲存體帳戶之管理平面的存取。 例如，您可以為特定使用者提供存取儲存體帳戶金鑰的能力，而其他使用者可以檢視儲存體帳戶的相關資訊，但是無法存取儲存體帳戶金鑰。

#### <a name="granting-access"></a>授與存取權
在正確範圍將適當的 RBAC 角色指派給使用者、群組和應用程式，即可授與存取權。 若要授與整個訂用帳戶的存取權，請指派訂用帳戶層級的角色。 您可以將權限授與資源群組本身，藉以授與資源群組中所有資源的存取權。 您也可以將特定角色指派給特定資源，例如儲存體帳戶。

以下是您需要知道有關使用 RBAC 存取 Azure 儲存體帳戶之管理作業的重點︰

* 當您指派存取權時，基本上會將角色指派給您希望擁有存取權的帳戶。 您可以控制用來管理該儲存體帳戶之作業的存取權，而不是對於帳戶中資料物件的存取權。 例如，您可以授與權限來擷取儲存體帳戶的屬性 (例如備援)，而不是授與 Blob 儲存體內的容器或容器中的資料。
* 針對有權存取儲存體帳戶中資料物件的使用者，您可以為他們提供權限來讀取儲存體帳戶金鑰，而該使用者接著可以使用這些金鑰來存取 Blob、查詢、格表及檔案。
* 角色可以指派給特定的使用者帳戶、使用者群組或特定的應用程式。
* 每個角色都有「動作」與「沒有動作」清單。 例如，「虛擬機器參與者」角色具有 “listKeys” 的動作，可允許讀取儲存體帳戶金鑰。 「參與者」含有「沒有動作」，例如在 Active Directory 中更新使用者的存取權。
* 儲存體的角色包括 (但不限於) 下列角色：

  * 擁有者 – 他們可以管理一切，包括存取權。
  * 參與者 – 他們可以執行擁有者可執行的所有動作，但指派存取權除外。 擁有此角色的使用者可以檢視和重新產生儲存體帳戶金鑰。 他們可以使用儲存體帳戶金鑰來存取資料物件。
  * 讀者 – 他們可以檢視儲存體帳戶 (密碼除外) 的相關資訊。 例如，如果您將儲存體帳戶中具有讀者權限的角色指派給某人，他們就能檢視儲存體帳戶的屬性，但無法對屬性進行任何變更或檢視儲存體帳戶金鑰。
  * 儲存體帳戶參與者 – 他們可以管理儲存體帳戶 – 他們可以讀取訂用帳戶的資源群組和資源，以及建立和管理訂用帳戶資源群組部署。 他們也可以存取儲存體帳戶金鑰，這表示他們可以存取資料平面。
  * 使用者存取系統管理員 – 他們可以管理儲存體帳戶的使用者存取權。 例如，他們可將「讀者」權限授與特定的使用者。
  * 虛擬機器參與者 – 他們可以管理虛擬機器，但是無法管理他們連接的儲存體帳戶。 這個角色可以列出儲存體帳戶金鑰，這表示您指派此角色的使用者可以更新資料平面。

    為了讓使用者能夠建立虛擬機器，他們必須能夠在儲存體帳戶中建立對應的 VHD 檔案。 若要這麼做，他們需要能夠擷取儲存體帳戶金鑰，並將它傳遞給建立 VM 的 API。 因此，他們必須具備此權限，如此就能列出儲存體帳戶金鑰。
* 定義自訂角色的能力是一個功能，允許您從可在 Azure 資源上執行的可用動作清單中撰寫一系列動作。
* 您必須先在 Azure Active Directory 中設定使用者，才能為他們指派角色。
* 您可以建立一份報告，其中包含哪一位人員已使用 PowerShell 或 Azure CLI，在哪個範圍中，為哪些對象授與/撤銷哪種類型的存取權。

#### <a name="resources"></a>資源
* [Azure Active Directory 角色型存取控制](../../role-based-access-control/role-assignments-portal.md)

  此文章說明 Azure Active Directory 角色型存取控制及其運作方式。
* [RBAC：內建角色](../../role-based-access-control/built-in-roles.md)

  此文章將詳細說明 RBAC 中所有可用的內建角色。
* [了解資源管理員部署和傳統部署](../../azure-resource-manager/resource-manager-deployment-model.md)

  此文章說明資源管理員部署和傳統部署模型，並說明使用資源管理員和資源群組的優點。 內容中會說明 Azure 計算、網路及儲存體提供者在 Resource Manager 模型下的運作方式。
* [使用 REST API 管理角色型存取控制](../../role-based-access-control/role-assignments-rest.md)

  此文章說明如何使用 REST API 來管理 RBAC。
* [Azure 儲存體資源提供者 REST API 參考](https://msdn.microsoft.com/library/azure/mt163683.aspx)

  此 API 參考說明您可用來以程式設計方式管理儲存體帳戶的 API。
* [使用 Resource Manager 驗證 API 來存取訂用帳戶](../../azure-resource-manager/resource-manager-api-authentication.md)

  此文章示範如何使用 Resource Manager API 進行驗證。
* [來自Ignite 且適用於 Microsoft Azure 的角色型存取控制](https://channel9.msdn.com/events/Ignite/2015/BRK2707)

  這個連結會連到 2015 MS Ignite 會議的 Channel 9 上的視訊。 在這個研討會中，他們討論 Azure 中的存取管理和報告功能，並探索使用 Azure Active Directory 安全存取 Azure 訂用帳戶的最佳做法。

### <a name="managing-your-storage-account-keys"></a>管理儲存體帳戶金鑰
儲存體帳戶金鑰是 Azure 所建立的 512 位元字串，搭配儲存體帳戶名稱就能用來存取儲存在儲存體帳戶中的資料物件，例如 Blob、資料表內的實體、佇列訊息，以及 Azure 檔案共用上的檔案。 控制儲存體帳戶金鑰的存取，就能控制該儲存體帳戶之資料平面的存取。

每個儲存體帳戶在 [Azure 入口網站](http://portal.azure.com/) 和 PowerShell Cmdlet 中都有兩個稱為「金鑰 1」和「金鑰 2」的金鑰。 您可以使用下列其中一種方法手動重新產生它們，包括 (但不限於) 使用 [Azure 入口網站](https://portal.azure.com/)、PowerShell、Azure CLI，或以程式設計方式使用 .NET 儲存體用戶端程式庫或 Azure 儲存體服務 REST API。

有許多原因會導致要重新產生儲存體帳戶金鑰。

* 您可能會基於安全性理由定期重新產生它們。
* 如果某人設法駭入應用程式並擷取硬式編碼或儲存於組態檔案中的金鑰，來為他們提供您儲存體帳戶的完整存取權，則您必須重新產生儲存體帳戶金鑰。
* 重新產生金鑰的另一種情況是，如果您的小組使用儲存體總管應用程式，其中會保留儲存體帳戶金鑰，而其中一位小組成員離開時。 在他們離開之後，應用程式還會繼續運作，讓他們能夠存取您的儲存體帳戶。 這實際上是他們建立帳戶層級共用存取簽章的主要原因 – 您可以改用帳戶層級的 SAS，而不是將存取金鑰儲存於組態檔案中。

#### <a name="key-regeneration-plan"></a>金鑰重新產生計畫
您不想在沒有任何規劃的情況下，只是重新產生所使用的金鑰。 如果您這麼做時，可能會切斷對該儲存體帳戶的所有存取權，而這會造成嚴重中斷。 這就是為什麼要有兩個金鑰。 您一次應該重新產生一個金鑰。

重新產生金鑰之前，先確定您擁有相依於儲存體帳戶的所有應用程式清單，以及您在 Azure 中使用的所有其他服務。 例如，如果您使用相依於儲存體帳戶的「Azure 媒體服務」，就必須在重新產生金鑰之後，將存取金鑰與媒體服務重新同步。 如果您使用儲存體總管之類的任何應用程式，也必須為這些應用程式提供新的金鑰。 如果 VM 的 VHD 檔案儲存在儲存體帳戶中，它們將不會受到重新產生儲存體帳戶金鑰影響。

您可以在 Azure 入口網站中重新產生金鑰。 重新產生金鑰之後，最多可能需要 10 分鐘的時間，才能在所有「儲存體服務」完成同步處理。

當您準備好時，以下是一般程序，可詳細說明您應該如何變更金鑰。 在此案例中，假設您目前使用的是金鑰 1，而您想要變更所有項目以改用金鑰 2。

1. 重新產生金鑰 2，以確保該金鑰是安全的。 您可以在 Azure 入口網站中執行此動作。
2. 在儲存體金鑰儲存所在的所有應用程式中，變更儲存體金鑰以使用金鑰 2 的新值。 測試並發佈應用程式。
3. 在所有應用程式和服務已啟動並執行成功之後，重新產生金鑰 1。 這確保您未明確提供新金鑰的任何人都將不再擁有儲存體帳戶的存取權。

如果您目前使用金鑰 2，就可以使用相同的程序，但反轉金鑰名稱。

您可以過幾天後移轉，變更每個應用程式來使用新的金鑰並進行發佈。 全部完成之後，您接著應該返回並重新產生舊的金鑰，使其無法運作。

另一個選項是將儲存體帳戶金鑰放在 [Azure 金鑰保存庫](https://azure.microsoft.com/services/key-vault/) 中做為密碼，並讓您的應用程式可從中擷取該金鑰。 接著，當您重新產生金鑰並更新 Azure 金鑰保存庫時，將不需重新部署應用程式，因為它們會自動從 Azure 金鑰保存庫中挑選新的金鑰。 請注意，您可以讓應用程式每次在您需要金鑰時讀取它，或者，您可以將它快取於記憶體中，如果使用金鑰時失敗，就再次從 Azure 金鑰保存庫中擷取該金鑰。

使用 Azure 金鑰保存庫，也會增加儲存體金鑰的安全性層級。 如果您使用這個方法，永遠都不需要將儲存體金鑰硬式編碼於組態檔中，這樣會移除某人不需特定權限即可存取金鑰的途徑。

使用 Azure 金鑰保存庫的另一個優點是，您也可以使用 Azure Active Directory 來控制金鑰的存取。 這表示您可以將存取權授與少數必須從 Azure 金鑰保存庫擷取金鑰的應用程式，並了解其他應用程式在未特別授與它們權限的情況下無法存取金鑰。

> [!NOTE]
> Microsoft 建議同一時間在您的所有應用程式中僅使用其中一個金鑰。 如果您在某些地方使用金鑰 1 並在其他地方使用金鑰 2，您就無法在沒有部分應用程式遺失存取的情況下輪換您的金鑰。

#### <a name="resources"></a>資源

* [管理 Azure 入口網站中的儲存體帳戶設定](storage-account-manage.md)
* [Azure 儲存體資源提供者 REST API 參考](https://msdn.microsoft.com/library/mt163683.aspx)

## <a name="data-plane-security"></a>資料平面安全性
資料平面安全性是指用來保護儲存在 Azure 儲存體的資料物件 (Blob、佇列、表格和檔案) 的方法。 我們已了解在傳輸資料期間加密資料和安全性的方法，但您該從何處著手來控制對物件的存取？

有三個選項可供您授權 Azure 儲存體資料物件的存取權，包括：

- 使用 Azure AD 來授權容器和佇列的存取權 (預覽)。 Azure AD 在授權方面的優勢勝過其他方法，包括不需要在程式碼中儲存祕密。 如需詳細資訊，請參閱[使用 Azure Active Directory 來驗證 Azure 儲存體的存取權 (預覽)](storage-auth-aad.md)。 
- 使用儲存體帳戶金鑰來透過共用金鑰授權存取權。 透過共用金鑰授權時，需要在應用程式中儲存儲存體帳戶金鑰，因此 Microsoft 建議盡可能改為使用 Azure AD。 對於生產應用程式，或是要授權 Azure 資料表和檔案的存取權，請在 Azure AD 整合處於預覽狀態時，繼續使用共用金鑰。
- 使用共用存取簽章，來授與一段特定時間對特定資料物件的受控權限。

此外，針對 Blob 儲存體，您可以透過設定要據以保存 Blob 之容器的存取層級，來允許對您的 Blob 進行公用存取。 如果您將容器的存取權設定為「Blob」或「容器」，將允許該容器中 Blob 的公用讀取存取權。 這表示 URL 指向該容器中 Blob 的任何人都可以在瀏覽器中開啟它，而不需使用共用存取簽章或擁有儲存體帳戶金鑰。

除了透過授權限制存取，您也可以使用[防火牆和虛擬網路](storage-network-security.md)，根據網路規則來限制對儲存體帳戶的存取。  此方法可讓您拒絕對公用網際網路流量的存取，只授與對特定 Azure 虛擬網路或公用網際網路 IP 位址範圍的存取。

### <a name="storage-account-keys"></a>儲存體帳戶金鑰
儲存體帳戶金鑰是由 Azure 所建立的 512 位元字串，搭配儲存體帳戶名稱就能用來存取儲存於儲存體帳戶中的資料物件。

例如，您可以讀取 Blob、寫入佇列、建立表格，以及修改檔案。 這其中許多動作都可透過 Azure 入口網站，或使用許多儲存體總管應用程式之一來執行。 您也可以撰寫程式碼來使用 REST API 或其中一個儲存體用戶端程式庫來執行這些作業。

如同 [管理平面安全性](#management-plane-security)一節中所討論，對傳統儲存體帳戶的儲存體金鑰的存取權，可以透過為 Azure 訂用帳戶提供完整存取權來授與。 使用 Azure Resource Manager 模型來存取儲存體帳戶的儲存體金鑰，可以透過角色型存取控制 (RBAC) 來控制。

### <a name="how-to-delegate-access-to-objects-in-your-account-using-shared-access-signatures-and-stored-access-policies"></a>如何使用共用存取簽章和預存存取原則委派帳戶中物件的存取權
共用存取簽章是一個字串，包含可附加至 URI 的安全性權杖，可讓您委派儲存體物件的存取權，以及指定存取的權限和日期/時間範圍之類的限制。

您可以授與對 Blob、容器、佇列、檔案和表格的存取權。 使用表格時，您可以實際授與權限來存取表格中某個範圍的實體，方法是指定您想要讓使用者有權存取的分割區和資料列索引鍵範圍。 例如，如果您已使用具備地理狀態的分割區索引鍵儲存資料，您可以為某人提供只能存取加州相關資料的存取權。

在另一個範例中，您可能會為 Web 應用程式提供 SAS 權杖，讓它能夠將項目寫入佇列，並為背景工作角色應用程式提供 SAS 權杖，從佇列中取得訊息並處理。 或者，您可以為某一位客戶提供 SAS 權杖，讓他們可以用來將圖片上傳至 Blob 儲存體中的容器，並為 Web 應用程式提供權限來讀取這些圖片。 在這兩種情況下，產生了關注點分離情況 – 每個應用程式只能取得執行其工作所需的存取權。 這可能是因為使用了共用存取簽章。

#### <a name="why-you-want-to-use-shared-access-signatures"></a>您想要使用共用存取簽章的原因
為什麼您想要使用 SAS，而不只是散發您的儲存體帳戶金鑰，哪一個方法更容易使用？ 散發您的儲存體帳戶金鑰，就像是在您的儲存體王國內共用金鑰。 它會授與完整存取權限。 其他人可以使用您的金鑰，並將其整個音樂媒體櫃上傳至您的儲存體帳戶。 他們可能也會使用受病毒感染的版本來取代您的檔案或竊取您的資料。 無限制散發您儲存體帳戶的存取權不應草率行事。

使用共用存取簽章，您可以只為用戶端提供有限時間內所需的權限。 例如，如果有人將 Blob 上傳至您的帳戶，您可以授與他們剛好足夠時間的寫入權限來上傳 Blob (當然，這取決於 Blob 的大小)。 如果您改變心意，您可以撤銷該存取權。

此外，您可以指定將使用 SAS 所提出的要求限制為特定的 IP 位址或 Azure 外部的 IP 位址範圍。 您也可以要求使用特定通訊協定 (HTTPS 或 HTTP/HTTPS) 來提出要求。 這表示，如果您只想要允許 HTTPS 流量，您可以將必要的通訊協定設定為僅限 HTTPS，而且將會封鎖 HTTP 流量。

#### <a name="definition-of-a-shared-access-signature"></a>定義共用存取簽章
共用存取簽章是一組附加至指向資源之 URL 的查詢參數

其中提供所允許存取權的相關資訊，以及准許該存取權的時間長度。 以下提供一個範例：這個 URI 會提供五分鐘對 Blob 的讀取權限。 請注意，SAS 查詢參數必須以 URL 編碼，例如 %3A 用於冒號 (:) 或 %20 用於空格。

```
http://mystorage.blob.core.windows.net/mycontainer/myblob.txt (URL to the blob)
?sv=2015-04-05 (storage service version)
&st=2015-12-10T22%3A18%3A26Z (start time, in UTC time and URL encoded)
&se=2015-12-10T22%3A23%3A26Z (end time, in UTC time and URL encoded)
&sr=b (resource is a blob)
&sp=r (read access)
&sip=168.1.5.60-168.1.5.70 (requests can only come from this range of IP addresses)
&spr=https (only allow HTTPS requests)
&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D (signature used for the authentication of the SAS)
```

#### <a name="how-the-shared-access-signature-is-authorized-by-the-azure-storage-service"></a>Azure 儲存體服務授權共用存取簽章的方式
當儲存體服務收到要求時，它會取得輸入查詢參數，並使用與呼叫程式相同的方法來建立簽章。 然後比較這兩個簽章。 如果它們同意，儲存體服務接著可以檢查儲存體服務版本，以確定它是有效的、檢查目前的日期和時間是在指定時段內、確定所要求的存取權會對應至所提出的要求，依此類推。

例如，利用上述的 URL，如果 URL 指向檔案而不是 Blob，此要求便會失敗，因為它指定共用存取簽章適用於 Blob。 如果呼叫的 REST 命令會更新 Blob，則該命令會失敗，因為共用存取簽章指定只准許讀取權限。

#### <a name="types-of-shared-access-signatures"></a>共用存取簽章的類型
* 服務層級的 SAS 可用來存取儲存體帳戶中的特定資源。 這其中的一些範例會擷取容器中的 Blob 清單、下載 Blob、更新資料表中的實體、將訊息新增至佇列，或將檔案上傳至檔案共用。
* 帳戶層級的 SAS 可用來存取可使用服務層級 SAS 的任何項目。 此外，它可以為服務層級 SAS 不准許的資源提供選項，例如，能夠建立容器、表格、佇列及檔案共用。 您也可以同時指定多個服務的存取權。 例如，您可能會為其他人提供您儲存體帳戶中 Blob 和檔案的存取權。

#### <a name="creating-a-sas-uri"></a>建立 SAS URI
1. 您可以視需要建立 URI，每次都定義所有查詢參數。

   此方法相當有彈性，但如果您每次都有一組類似的邏輯參數，則使用「預存存取原則」會是個更好的做法。
2. 您可以針對整個容器、檔案共用、表格或佇列建立預存存取原則。 然後使用這個原則做為您所建立的 SAS URI 基礎。 您可以輕鬆地撤銷以預存存取原則為基礎的權限。 您可以在每個容器、佇列、資料表或檔案共用上最多定義 5 個原則。

   例如，如果您要讓許多人讀取特定容器中的 Blob，您可以建立預存存取原則，表示會「提供讀取權」以及每次都一樣的任何其他設定。 接著您可以使用預存存取原則的設定，並指定到期日期/時間，來建立 SAS URI。 這樣做的優點是您不需每次指定所有查詢參數。

#### <a name="revocation"></a>撤銷
假設您的 SAS 已遭洩露，或您想要基於公司安全性或法規遵循需求變更它。 您如何使用該 SAS 撤銷資源的存取權？ 這取決於您建立 SAS URI 的方式。

如果您使用臨機操作的 URI，您就會有三個選項。 您可以發出具有短期到期原則的 SAS 權杖，然後等待 SAS 到期。 您可以重新命名或刪除資源 (假設權杖範圍只限於單一物件)。 您可以變更儲存體帳戶金鑰。 根據使用該儲存體帳戶的服務數目而定，這最後一個選項可能會產生很大的影響，而且可能不是您在沒有任何規劃的情況下會想要執行的動作。

如果您使用衍生自預存存取原則的 SAS，就可以透過撤銷預存存取原則來移除存取權 – 您只能在它已經完全過期時變更 它，或完全移除它。 這會立即生效，並使每個使用該預存存取原則建立的 SAS 失效。 更新或移除預存存取原則可能會影響透過 SAS 存取該特定容器、檔案共用、表格或佇列的使用者，但如果會寫入用戶端，使得他們可在舊的 SAS 變成無效時要求一個新的 SAS，則這將可正常運作。

因為使用衍生自預存存取原則的 SAS 讓您能夠立即撤銷該 SAS，所以建議的最佳做法是一律儲存預存存取原則 (如果可能)。

#### <a name="resources"></a>資源
如需有關使用「共用存取簽章」和「預存存取原則」的更多詳細資訊，請完成範例並參閱下列文章︰

* 以下是參考文章。

  * [服務 SAS](https://msdn.microsoft.com/library/dn140256.aspx)

    此文章提供使用服務層級 SAS 搭配 Blob、佇列、表格範圍及檔案的範例。
  * [建構服務 SAS](https://msdn.microsoft.com/library/dn140255.aspx)
  * [建構帳戶 SAS](https://msdn.microsoft.com/library/mt584140.aspx)
* 這些是使用 .NET 用戶端程式庫，來建立共用存取簽章和預存存取原則的教學課程。

  * [使用共用存取簽章 (SAS)](../storage-dotnet-shared-access-signature-part-1.md)
  * [共用存取簽章，第 2 部分：透過 Blob 服務來建立與使用 SAS](../blobs/storage-dotnet-shared-access-signature-part-2.md)

    此文章包含 SAS 模型的說明、共用存取簽章的範例，以及使用 SAS 最佳做法的建議。 同時也會討論撤銷授與的權限。

* 驗證

  * [Azure 儲存體服務的驗證](https://msdn.microsoft.com/library/azure/dd179428.aspx)
* 共用存取簽章入門教學課程

  * [SAS 入門教學課程](https://github.com/Azure-Samples/storage-dotnet-sas-getting-started)

## <a name="encryption-in-transit"></a>傳輸中加密
### <a name="transport-level-encryption--using-https"></a>Transport-Level Encryption – Using HTTPS
您應該採取以確保 Azure 儲存體資料安全性的另一個步驟是在用戶端和 Azure 儲存體之間加密資料。 第一個建議是一律使用 [HTTPS](https://en.wikipedia.org/wiki/HTTPS) 通訊協定，可確保透過公用網際網路的安全通訊。

如果要有安全的通訊通道，呼叫 REST API 或存取儲存體中的物件時，您應該一律使用 HTTPS。 此外， **共用存取簽章** (可用來委派 Azure 儲存體物件的存取權) 包含一個選項，可指定在使用共用存取簽章時只能使用 HTTPS 通訊協定，以確保任何使用 SAS 權杖送出連結的人都將使用正確的通訊協定。

透過啟用儲存體帳戶[所需的安全傳輸](../storage-require-secure-transfer.md)，您可於呼叫 REST API 來存取儲存體帳戶中的物件時強制使用 HTTPS。 啟用此選項後，使用 HTTP 的連線將被拒絕。

### <a name="using-encryption-during-transit-with-azure-file-shares"></a>傳輸期間透過 Azure 檔案共用使用加密
[Azure 檔案](../files/storage-files-introduction.md)支援透過 SMB 3.0 加密，以及在使用檔案 REST API 時以 HTTPS 加密。 在 Azure 檔案共用所在的 Azure 區域之外 (例如，內部部署或其他 Azure 區域) 掛接時，一律需要 SMB 3.0 (含加密)。 SMB 2.1 不支援加密，因此，預設只允許在 Azure 中的相同區域內連線，但可以透過對儲存體帳戶[要求安全傳輸](../storage-require-secure-transfer.md)來強制採用 SMB 3.0 (含加密)。

SMB 3.0 (含加密) 可於[所有支援的 Windows 和 Windows Server 作業系統](../files/storage-how-to-use-files-windows.md)中使用，但 Windows 7 和 Windows Server 2008 R2 除外，這兩種僅支援 SMB 2.1。 SMB 3.0 也支援 [macOS](../files/storage-how-to-use-files-mac.md) 和 [Linux](../files/storage-how-to-use-files-linux.md) 散發套件 (使用 Linux 核心 4.11 和更新版本)。 SMB 3.0 的加密支援也由數個 Linux 散發套件回溯導入到較舊版本的 Linux 核心，請參閱[了解 SMB 用戶端需求](../files/storage-how-to-use-files-linux.md#smb-client-reqs)。

### <a name="using-client-side-encryption-to-secure-data-that-you-send-to-storage"></a>使用用戶端加密來保護傳送到儲存體的資料
另一個可協助您確保在用戶端應用程式和儲存體之間傳輸時資料安全性的選項是用戶端加密。 資料會先加密，然後才傳輸到 Azure 儲存體。 從 Azure 儲存體擷取資料時，在用戶端上收到資料之後要將之解密。 即使資料在通過連線時已加密，但還是建議您使用 HTTPS，因為其內建資料完整性檢查，有助於降低影響資料完整性的網路錯誤。

用戶端加密也是一個可用來加密待用資料的方法，因為資料是以加密形式所儲存。 我們將在 [待用加密](#encryption-at-rest)一節中詳細討論這一點。

## <a name="encryption-at-rest"></a>待用加密
有三個 Azure 功能可提供待用加密。 Azure 磁碟加密可用來加密 IaaS 虛擬機器中的作業系統和資料磁碟。 用戶端加密和 SSE 都是用來加密「Azure 儲存體」中的資料。 

雖然您可以使用用戶端加密來加密傳輸中的資料 (也會以其加密形式儲存在儲存體中)，但您可能會偏好在傳輸期間使用 HTTPS，然後在儲存資料時使用某種方式來自動加密資料。 有兩種做法可以執行此動作 -- Azure 磁碟加密和 SSE。 其中一種是用來直接加密 VM 所使用的作業系統和資料磁碟上的資料，另一種則是用來加密寫入 Azure Blob 儲存體的資料。

### <a name="storage-service-encryption-sse"></a>儲存體服務加密 (SSE)

系統會針對所有儲存體帳戶啟用 SSE 且無法停用。 在將資料寫入至「Azure 儲存體」時，SSE 會自動加密資料。 當您從「Azure 儲存體」讀取資料時，「Azure 儲存體」會先將資料解密，然後才傳回資料。 SSE 可讓您無須修改程式碼或在任何應用程式中新增程式碼，即可保護資料。

您可以使用 Microsoft 管理的金鑰，或自己的自訂金鑰。 Microsoft 會產生受控金鑰，然後除了依照內部 Microsoft 原則定義的方式處理金鑰的安全儲存體之外，也會處理其定期輪換。 如需有關使用自訂金鑰的詳細資訊，請參閱[使用 Azure Key Vault 中客戶管理的金鑰進行儲存體服務加密](storage-service-encryption-customer-managed-keys.md)。

SSE 會自動將所有效能層 (標準和進階)、所有部署模型 (Azure Resource Manager 和傳統) 以及所有「Azure 儲存體」服務 (Blob、佇列、資料表和檔案) 中的資料加密。 

### <a name="client-side-encryption"></a>用戶端加密
在討論傳輸中的資料加密時，我們曾提及用戶端加密。 此功能可讓您以程式設計方式加密用戶端應用程式中的資料，然後透過連線傳送資料以寫入 Azure 儲存體，並在從 Azure 儲存體擷取資料之後以程式設計方式解密資料。

這會提供在傳輸中加密，但也會提供待用加密的功能。 雖然會在傳輸過程中加密資料，但仍建議使用 HTTPS 來充分利用內建的資料完整性檢查，以協助降低影響資料完整性的網路錯誤風險。

如果 Web 應用程式會儲存 Blob 和擷取 Blob，而您想要讓應用程式和資料儘可能保持安全，則可使用此範例。 在此情況下，您會使用用戶端加密。 用戶端和 Azure Blob 服務之間的流量包含加密的資源，而且沒有人能夠解譯傳輸中的資料並將它重組到您的私人 Blob。

用戶端加密內建在 Java 和 .NET 儲存體用戶端程式庫中，這進而會使用 Azure Key Vault API，使您的實作變簡單。 加密和解密資料的程序會使用信封技術，並在每個儲存體物件中儲存加密所使用的中繼資料。 例如，如果是 Blob，會將它儲存於 Blob 中繼資料中，如果是佇列，則會將它新增至每個佇列訊息。

至於加密本身，您可以產生和管理自己的加密金鑰。 您也可以使用 Azure 儲存體用戶端程式庫所產生的金鑰，或者可以讓 Azure 金鑰保存庫產生金鑰。 您可以將加密金鑰儲存於內部部署金鑰儲存體中，或將它們儲存在 Azure 金鑰保存庫中。 Azure 金鑰保存庫可讓您使用 Azure Active Directory，為特定的使用者授與 Azure 金鑰保存庫中密碼的存取權。 這表示並非人人都能讀取 Azure Key Vault，以及擷取您用來進行用戶端加密的金鑰。

#### <a name="resources"></a>資源
* [在 Microsoft Azure 儲存體中使用 Azure 金鑰保存庫加密和解密 blob](../blobs/storage-encrypt-decrypt-blobs-key-vault.md)

  此文章說明如何搭配 Azure 金鑰保存庫使用用戶端加密，包括如何使用 PowerShell 來建立 KEK 並將它儲存在保存庫中。
* [Microsoft Azure 儲存體的用戶端加密和 Azure Key Vault 金鑰保存庫](../storage-client-side-encryption.md)

  此文章說明用戶端加密，並提供使用儲存體用戶端程式庫，從四個儲存體服務加密和解密資源的範例。 它也會討論 Azure 金鑰保存庫。

### <a name="using-azure-disk-encryption-to-encrypt-disks-used-by-your-virtual-machines"></a>使用 Azure 磁碟加密來加密虛擬機器所使用的磁碟
「Azure 磁碟加密」是一個新功能。 此功能允許您加密 IaaS 虛擬機器所使用的作業系統磁碟和資料磁碟。 對於 Windows，磁碟機是使用業界標準的 BitLocker 加密技術來加密。 對於 Linux，磁碟是使用 DM-Crypt 技術來加密。 這會與 Azure 金鑰保存庫整合，可讓您控制和管理磁碟加密金鑰。

在 Microsoft Azure 中啟用時，解決方案會對 IaaS VM 支援下列案例：

* 與 Azure 金鑰保存庫整合
* 標準層 VM：[A、D、DS、G、GS 等系列 IaaS VM](https://azure.microsoft.com/pricing/details/virtual-machines/)
* 在 Windows 和 Linux IaaS VM 上啟用加密
* 為 Windows IaaS VM 的 OS 和資料磁碟機停用加密
* 為 Linux IaaS VM 的資料磁碟機停用加密
* 在執行 Windows 用戶端 OS 的 IaaS VM 上啟用加密
* 在具有掛接路徑的磁碟區上啟用加密
* 在使用 mdadm 設定了等量磁碟 (RAID) 的 Linux VM 上啟用加密
* 在使用資料磁碟適用之 LVM 的 Linux VM 上啟用加密
* 在使用儲存空間設定的 Windows VM 上啟用加密
* 所有 Azure 公用區域皆受到支援

解決方案不支援此版本中的下列案例、功能和技術：

* 基本層 IaaS VM
* 在 Linux IaaS VM 的 OS 磁碟機上停用加密
* 使用傳統 VM 建立方法所建立的 IaaS VM
* 與您的內部部署金鑰管理服務整合
* Azure 檔案 (共用檔案系統)、網路檔案系統 (NFS)、動態磁碟區和以軟體型 RAID 系統所設定的 Windows VM


> [!NOTE]
> 下列 Linux 散發套件目前支援 Linux OS 磁碟加密：RHEL 7.2、CentOS 7.2n 和 Ubuntu 16.04。
>
>

此功能可確保虛擬機器磁碟上的所有待用資料都會在 Azure 儲存體中加密。

#### <a name="resources"></a>資源
* [Windows 和 Linux IaaS VM 適用的 Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)

### <a name="comparison-of-azure-disk-encryption-sse-and-client-side-encryption"></a>Azure 磁碟加密、SSE 和用戶端加密的比較

#### <a name="iaas-vms-and-their-vhd-files"></a>IaaS VM 及其 VHD 檔案

針對 IaaS VM 所使用的資料磁碟，建議使用「Azure 磁碟加密」。 如果您使用來自 Azure Marketplace 的映像來建立具有非受控磁碟的 VM，Azure 會以[淺層複製 (Shallow Copy)](https://en.wikipedia.org/wiki/Object_copying) 的方式將該映像複製到您在「Azure 儲存體」中的儲存體帳戶，而且即使您已啟用 SSE，也不會將其加密。 當它建立 VM 並開始更新映像之後，SSE 將開始加密資料。 因此，如果您想要將從 Azure Marketplace 中映像建立之具有非受控磁碟的 VM 完整加密，最好是在這些 VM 上使用「Azure 磁碟加密」。 如果您建立的是具有受控磁碟的 VM，則 SSE 預設會使用平台管理的金鑰來加密所有資料。 

如果透過內部部署將預先加密的 VM 帶入 Azure 中，您就能將加密金鑰上傳至 Azure 金鑰保存庫，並繼續針對使用內部部署的 VM 使用加密。 啟用 Azure 磁碟加密即可處理此案例。

如果您有來自內部部署的未加密 VHD，就可以將它上傳到資源庫做為自訂映像並從中佈建 VM。 如果您使用 Resource Manager 範本來執行此動作，就可以要求它在啟動 VM 時開啟 Azure 磁碟加密。

當您新增資料磁碟並掛接於 VM 時，您可以在該資料磁碟上開啟 Azure 磁碟加密。 它將會先在本機加密該資料磁碟，然後傳統部署模型層會對儲存體進行延遲寫入，以便加密儲存體內容。

#### <a name="client-side-encryption"></a>用戶端加密
用戶端加密是加密資料的最安全方法，因為它會在傳輸前加密資料。  不過，它需要您使用儲存體將程式碼加入應用程式，而您可能不想這樣做。 在這些情況下，您可以使用 HTTPS 來保護傳輸中的資料。 當資料到達「Azure 儲存體」時，SSE 會將其加密。

透過用戶端加密，您可以加密表格實體、訊息佇列和 Blob。 

用戶端加密完全是透過應用程式來管理。 這是最安全的方式，但會要求您透過程式設計方式變更應用程式，並將金鑰管理程序放在正確的位置上。 如果您在傳輸期間想要額外的安全性，而且想要將儲存的資料加密，就可以使用這個方式。

用戶端加密會在用戶端上產生較多負載，您必須在延展性計畫中考慮到這一點，特別是當您要加密並傳輸大量資料的時候。

#### <a name="storage-service-encryption-sse"></a>儲存體服務加密 (SSE)

SSE 是由 Azure 儲存體所管理。 SSE 並不針對傳輸中資料提供安全性，但會在資料寫入至「Azure 儲存體」時加密資料。 SSE 不會影響「Azure 儲存體」效能。

您可以使用 SSE 來加密任何種類的儲存體帳戶資料 (區塊 Blob、附加 Blob、分頁 Blob、資料表資料、佇列資料及檔案)。

如果您使用 VHD 檔案的封存或程式庫作為建立新虛擬機器的基礎，則您可以建立新的儲存體帳戶，然後將 VHD 檔案上傳到該帳戶。 這些 VHD 檔案將會由 Azure 儲存體加密。

如果您為 VM 中的磁碟啟用「Azure 磁碟加密」，則任何新寫入的資料都會同時受到 SSE 和「Azure 磁碟加密」加密。

## <a name="storage-analytics"></a>儲存體分析
### <a name="using-storage-analytics-to-monitor-authorization-type"></a>使用儲存體分析來監視授權類型
對於每個儲存體帳戶，您可以啟用 Azure 儲存體分析，來執行記錄和儲存計量資料。 當您想要檢查儲存體帳戶的效能計量，或是因為發生效能問題而需要針對儲存體帳戶進行疑難排解時，這是一個絕佳的工具。

您可以在儲存體分析記錄中看見的另一部分資料是其他人存取儲存體時所使用的驗證方法。 例如，使用 Blob 儲存體，您可以看見他們使用的是否為共用存取簽章或儲存體帳戶金鑰，或者存取的 Blob 是否為公用的。

如果您要嚴密防護對儲存體的存取，這會相當有用。 例如，在 Blob 儲存體中，您可以將所有容器設定為私人，並透過您的應用程式實作 SAS 服務的用法。 然後您可以定期檢查記錄，以查看是否會使用儲存體帳戶金鑰存取您的 Blob (這可能表示安全性缺口)，或者 Blob 是公用的但它們不應該是公用的。

#### <a name="what-do-the-logs-look-like"></a>記錄的外觀
當您透過 Azure 入口網站啟用儲存體帳戶計量和記錄之後，分析資料將開始快速累積。 每個服務的記錄與計量是分開的；只有在該儲存體帳戶中有活動時才會寫入記錄，而根據您設定計量的方式，將會每分鐘、每小時或每天記錄計量。

記錄會儲存於儲存體帳戶中名為 $logs 之容器的區塊 Blob 中。 啟用儲存體分析時，會自動建立此容器。 一旦建立此容器之後，就無法予以刪除，但您可以刪除其內容。

在 $logs 容器下方，每個服務都有一個資料夾，另外還有適用於年/月/日/小時的子資料夾。 在小時下方，記錄會帶有編號。 以下是目錄結構的外觀︰

![記錄檔的檢視](./media/storage-security-guide/image1.png)

對於 Azure 儲存體的每個要求都會記錄。 以下是記錄的快照，會顯示前幾個欄位。

![記錄檔的快照](./media/storage-security-guide/image2.png)

如您所見，您可以使用記錄來追蹤各種對儲存體帳戶的呼叫。

#### <a name="what-are-all-of-those-fields-for"></a>這些所有欄位的用途為何？
有一篇文章會列出下列所有資源，提供記錄中許多欄位的清單及其用途。 以下是依序列出的欄位清單︰

![記錄檔中欄位的快照](./media/storage-security-guide/image3.png)

我們對於 GetBlob 的項目及其授權方法很感興趣，所以必須尋找 operation-type 為 "Get-Blob" 的項目，然後檢查 request-status (第 4</sup> 欄) 和 authorization-type (第 8</sup> 欄)。

例如，在上述清單的前幾個列中，要求狀態為 "Success" 且驗證類型為 "authenticated"。 這表示已使用儲存體帳戶金鑰授權要求。

#### <a name="how-is-access-to-my-blobs-being-authorized"></a>如何授權存取我的 Blob？
以下提供三個我們感興趣的案例。

1. Blob 是公用的且可使用 URL (而不需共用存取簽章) 來存取。 在此案例中，要求狀態為 "AnonymousSuccess" 且驗證類型為 "anonymous"。

   1.0;2015-11-17T02:01:29.0488963Z;GetBlob;**AnonymousSuccess**;200;124;37;**anonymous**;;mystorage…
2. Blob 是私人的且與共用存取簽章搭配使用。 在此案例中，要求狀態為 "SASSuccess" 且驗證類型為 "sas"。

   1.0;2015-11-16T18:30:05.6556115Z;GetBlob;**SASSuccess**;200;416;64;**sas**;;mystorage…
3. Blob 是私人且使用儲存體金鑰來存取它。 在此案例中，request-status 為 "**Success**" 且 authorization-type 為 “**authenticated**”。

   1.0;2015-11-16T18:32:24.3174537Z;GetBlob;**Success**;206;59;22;**authenticated**;mystorage…

您可以使用 Microsoft Message Analyzer 來檢視和分析這些記錄。 它包含搜尋和篩選功能。 例如，您可能會想要搜尋 GetBlob 的執行個體，以查看其使用方式是否如您預期，也就是要確定其他人不會以不適當的方式存取您的儲存體帳戶。

#### <a name="resources"></a>資源
* [Storage Analytics](../storage-analytics.md)

  此文章是儲存體分析及如何啟用它們的概觀。
* [儲存體分析記錄檔格式](https://msdn.microsoft.com/library/azure/hh343259.aspx)

  此文章說明儲存體分析記錄格式，並詳細說明其中的可用欄位，包括驗證類型，其會指出要求所使用的驗證類型。
* [在 Azure 入口網站中監視儲存體帳戶](../storage-monitor-storage-account.md)

  此文章說明如何設定和監視儲存體帳戶的計量與記錄。
* [使用 Azure 儲存體度量和記錄、AzCopy 和 Message Analyzer 進行端對端疑難排解](../storage-e2e-troubleshooting.md)

  此文章討論如何使用儲存體分析進行疑難排解，並示範如何使用 Microsoft Message Analyzer。
* [Microsoft Message Analyzer 操作指南](https://technet.microsoft.com/library/jj649776.aspx)

  此文章是 Microsoft Message Analyzer 的參考，並包含教學課程、快速入門及功能摘要的連結。

## <a name="cross-origin-resource-sharing-cors"></a>跨原始來源資源分享 (CORS)
### <a name="cross-domain-access-of-resources"></a>跨網域存取資源
在某一個網域中執行的 Web 瀏覽器對來自不同網域的資源提出 HTTP 要求時，這稱為跨原始來源的 HTTP 要求。 例如，來自 contoso.com 的 HTML 網頁會對裝載於 fabrikam.blob.core.windows.net 上的 jpeg 提出要求。 基於安全性理由，瀏覽器會限制從指令碼 (例如 JavaScript) 內初始化的跨原始來源 HTTP 要求。 這表示當 contoso.com 的網頁上有一些 JavaScript 程式碼要求 fabrikam.blob.core.windows.net 上的該 jpeg 時，瀏覽器將不允許該要求。

這必須利用 Azure 儲存體來執行哪些動作？ 嗯，如果您正使用名為 Fabrikam 的儲存體帳戶在 Blob 儲存體中儲存 JSON 或 XML 資料檔案之類的靜態資產，則資產的網域將會是 fabrikam.blob.core.windows.net，而 contoso.com web 應用程式將無法使用 JavaScript 來存取它們，因為網域不一樣。 如果您嘗試呼叫其中一個 Azure 儲存體服務 (例如表格儲存體)，以傳回要透過 JavaScript 用戶端處理的 JSON 資料，則這同樣適用。

#### <a name="possible-solutions"></a>可能的解決方案
解決此問題的方法之一是將像是 "storage.contoso.com" 的自訂網域指派給 fabrikam.blob.core.windows.net。 問題在於，您只能將該自訂網域指派給一個儲存體帳戶。 如果資產儲存於多個儲存體帳戶中該怎麼辦？

解決此問題的另一個方法是讓 Web 應用程式做為儲存體呼叫所使用的 Proxy。 這表示，如果您要將檔案上傳至 Blob 儲存體，Web 應用程式可以在本機寫入它，然後將它複製到 Blob 儲存體，或者將它全部讀入記憶體，然後將它寫入 Blob 儲存體。 或者，您可以撰寫專屬的 Web 應用程式 (例如 Web API)，本機上傳檔案並將它們寫入 Blob 儲存體。 無論如何，您都必須在判定延展性的需求時負責該功能。

#### <a name="how-can-cors-help"></a>CORS 如何提供協助？
Azure 儲存體可讓您啟用 CORS – 跨原始來源資源共用。 對於每個儲存體帳戶，您可以指定可存取該儲存體帳戶中之資源的網域。 例如，在上述案例中，我們可以在 fabrikam.blob.core.windows.net 儲存體帳戶上啟用 CORS，然後將它設定為允許存取 contoso.com。 然後 Web 應用程式 contoso.com 就能直接存取 fabrikam.blob.core.windows.net 中的資源。

要注意的一點是，CORS 允許存取，但不提供所有對儲存體資源之非公用存取所需的驗證。 這表示，如果它們是公用的，您就只能存取 Blob，或者您可以包含共用存取簽章來為您提供適當的權限。 表格、佇列和檔案沒有公用存取權且需要 SAS。

根據預設，所有服務上都會停用 CORS。 您可以使用 REST API 或儲存體用戶端程式庫，呼叫其中一個方法來設定服務原則，藉以啟用 CORS。 當您這樣做時，就會在 XML 中包含 CORS 規則。 以下範例會針對儲存體帳戶的 Blob 服務，使用「設定服務屬性」作業來設定 CORS 規則。 您可以使用 Azure 儲存體的儲存體用戶端程式庫或 REST API 來執行該作業。

```xml
<Cors>    
    <CorsRule>
        <AllowedOrigins>http://www.contoso.com, http://www.fabrikam.com</AllowedOrigins>
        <AllowedMethods>PUT,GET</AllowedMethods>
        <AllowedHeaders>x-ms-meta-data*,x-ms-meta-target*,x-ms-meta-abc</AllowedHeaders>
        <ExposedHeaders>x-ms-meta-*</ExposedHeaders>
        <MaxAgeInSeconds>200</MaxAgeInSeconds>
    </CorsRule>
<Cors>
```

以下是每一列的意義︰

* **AllowedOrigins** 這會指出哪些不相符的網域可以向儲存體服務提出要求並從中接收資料。 這表示，contoso.com 和 fabrikam.com 可以針對特定的儲存體帳戶向 Blob 儲存體要求資料。 您也可以將此設定為萬用字元 (\*)，以允許所有網域存取要求。
* **AllowedMethods** 這是提出要求時可以使用的方法 (HTTP 要求動詞命令) 清單。 在此範例中，只允許 PUT 和 GET。 您可以將此設定為萬用字元 (\*)，以允許使用所有方法。
* **AllowedHeaders** 這是在提出要求時原始網域可以指定的要求標頭。 在此範例中，允許以 x-ms-meta-data、x-ms-meta-target 及 x-ms-meta-abc 開頭的所有中繼資料標頭。 萬用字元 (\*) 表示允許任何以指定前置詞開頭的標頭。
* **ExposedHeaders** 這表示瀏覽器應向要求簽發者公開的回應標頭。 在此範例中，將公開任何以 "x-ms-meta-" 開頭的標頭。
* **MaxAgeInSeconds** 這是瀏覽器將快取預檢 OPTIONS 要求的時間量上限。 (如需預檢要求的詳細資訊，請檢查下列第一篇文章)。

#### <a name="resources"></a>資源
如需有關 CORS 及如何啟用它的詳細資訊，請參閱下列資源。

* [在 Azure.com 上 Azure 儲存體服務的跨原始資源共用 (CORS) 支援](../storage-cors-support.md)

  此文章概述 CORS，以及如何設定不同儲存體服務的規則。
* [在 MSDN 上 Azure 儲存體服務的跨原始資源共用 (CORS) 支援](https://msdn.microsoft.com/library/azure/dn535601.aspx)

  這是適用於 Azure 儲存體服務的 CORS 支援的參考文件。 其中提供適用於每個儲存體服務的文章連結，並示範範例且說明 CORS 檔案中的每個元素。
* [Microsoft Azure 儲存體︰CORS 簡介](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/02/03/windows-azure-storage-introducing-cors.aspx)

  這是最初發表 CORS 並示範如何使用之部落格文章的連結。

## <a name="frequently-asked-questions-about-azure-storage-security"></a>關於 Azure 儲存體安全性的常見問題集
1. **如果我無法使用 HTTPS 通訊協定，該如何驗證我傳輸至 Azure 儲存體或從中傳出之 Blob 的完整性？**

   如果基於任何原因，您必須使用 HTTP 而不是 HTTPS，且您要使用區塊 Blob，您可以使用 MD5 檢查，協助確認要傳輸的 Blob 完整性。 這有助於提供保護以防止發生網路/傳輸層錯誤，但不一定是媒介攻擊。

   如果您可以使用 HTTPS 來提供傳輸層安全性，則使用 MD5 檢查是多餘且不必要的。

   如需詳細資訊，請查看 [Azure Blob MD5 概觀](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/02/18/windows-azure-blob-md5-overview.aspx)。
2. **美國政府的 FIPS 相符性為何？**

   美國聯邦資訊處理標準 (FIPS) 會定義由美國聯邦政府電腦系統核准使用的密碼編譯演算法，以便保護機密資料。 在 Windows 伺服器或桌面上啟用 FIPS 模式，會告知作業系統僅應使用 FIPS 驗證的密碼編譯演算法。 如果應用程式使用不相容的演算法，該應用程式將會中斷。 利用 .NET Framework 4.5.2 或更新版本，應用程式會在電腦處於 FIPS 模式時，自動切換密碼編譯演算法來使用 FIPS 相容的演算法。

   Microsoft 可讓每位客戶決定是否要啟用 FIPS 模式。 我們認為沒有充分的理由來對不受政府法規限制的客戶預設啟用 FIPS 模式。

### <a name="resources"></a>資源
* [Why We’re Not Recommending "FIPS Mode" Anymore (為什麼我們不再建議「FIPS 模式」)](https://blogs.technet.microsoft.com/secguide/2014/04/07/why-were-not-recommending-fips-mode-anymore/)

  此部落格文章提供 FIPS 概觀，並說明他們為什麼預設不啟用 FIPS 模式。
* [FIPS 140 Validation (FIPS 140 驗證)](https://technet.microsoft.com/library/cc750357.aspx)

  此文章提供 Microsoft 產品和密碼編譯模組如何符合美國美國聯邦政府的 FIPS 標準的相關資訊。
* [Windows XP 和 Windows 的更新版本中「系統密碼編譯︰使用符合 FIPS 規範的演算法進行加密，雜湊，以及簽章」安全性設定的效果](https://support.microsoft.com/kb/811833)

  此文章討論如何在較舊的 Windows 電腦中使用 FIPS 模式。