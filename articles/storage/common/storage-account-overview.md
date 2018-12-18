---
title: Azure 儲存體帳戶概觀 | Microsoft Docs
description: 了解用於建立和使用 Azure 儲存體帳戶的選項。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 09/13/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: 0b2bf8cdb1af85e5ddbd3b18dd6dfa47bcb835b4
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47432880"
---
# <a name="azure-storage-account-overview"></a>Azure 儲存體帳戶概觀

Azure 儲存體帳戶包含您所有的 Azure 儲存體資料物件：Blob、檔案、佇列、資料表和磁碟。 Azure 儲存體帳戶中的資料具有持久性和高可用性、安全、可大規模調整，而且可以透過 HTTP 或 HTTPS 從世界各地存取任。 

若要了解如何建立 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](storage-quickstart-create-account.md)。

## <a name="types-of-storage-accounts"></a>儲存體帳戶類型

Azure 儲存體提供三種儲存體帳戶。 每個類型都支援不同的功能，而且都有自己的計價模式。 您在建立儲存體帳戶之前請先考量這些差異，以判斷您的應用程式最適用的帳戶類型。 儲存體帳戶的類型如下：

* **一般用途 v2** 帳戶 (建議用於大部分的案例)
* **一般用途 v1** 帳戶
* **Blob 儲存體**帳戶

下表描述儲存體帳戶類型及其功能：

| 儲存體帳戶類型 | 支援的服務                       | 支援的效能層級 | 支援的存取層               | 複寫選項                                                | 部署模型<sup>1</sup>  | 加密<sup>2</sup> |
|----------------------|------------------------------------------|-----------------------------|--------------------------------------|--------------------------------------------------------------------|-------------------|------------|
| 一般用途 V2   | Blob、檔案、佇列、資料表及磁碟       | 標準、進階           | 經常性存取層、非經常性存取層、封存存取層<sup>3</sup> | LRS、ZRS<sup>4</sup>、GRS、RA-GRS | Resource Manager | 已加密  |
| 一般用途 V1   | Blob、檔案、佇列、資料表及磁碟       | 標準、進階           | N/A                                  | LRS、GRS、RA-GRS                                                   | Resource Manager、傳統  | 已加密  |
| Blob 儲存體         | Blob (僅限區塊 Blob 和附加 Blob) | 標準                    | 經常性存取層、非經常性存取層、封存存取層<sup>3</sup>                            | LRS、GRS、RA-GRS                                                   | Resource Manager  | 已加密  |

<sup>1</sup>建議使用 Azure Resource Manager 部署模型。 使用傳統部署模型的儲存體帳戶仍可建立於某些位置，而且會繼續支援現有的傳統帳戶。 如需詳細資訊，請參閱 [Azure Resource Manager vs. 傳統部署：了解資源的部署模型和狀態](../../azure-resource-manager/resource-manager-deployment-model.md)。

<sup>2</sup>所有儲存體帳戶會使用待用資料的 Azure 儲存體服務加密 (SSE) 進行加密。 如需詳細資訊，請參閱[待用資料的 Azure 儲存體服務加密](storage-service-encryption.md)。

<sup>3</sup>封存儲存層只可使用於個別 Blob 層級，而不可使用於儲存體帳戶層級。 只可以封存區塊 Blob 和附加 Blob。 如需詳細資訊，請參閱 [Azure Blob 儲存體︰經常性存取、非經常性存取和封存儲存層](../blobs/storage-blob-storage-tiers.md)。

<sup>4</sup>區域備援儲存體 (ZRS) 僅適用於標準的一般用途 v2 儲存體帳戶。 如需 ZRS 的詳細資訊，請參閱[區域備援儲存體 (ZRS)：高可用性 Azure 儲存體應用程式](storage-redundancy-zrs.md)。 如需其他複寫選項的詳細資訊，請參閱 [Azure 儲存體複寫](storage-redundancy.md)。

### <a name="general-purpose-v2-accounts"></a>一般用途 v2 帳戶

一般用途 v2 儲存體帳戶能支援最新的 Azure 儲存體功能，而且包含一般用途 v1 與 Blob 儲存體帳戶的所有功能。 一般用途 v2 帳戶能針對 Azure 儲存體提供最低的每 GB 容量價格，以及極具業界競爭力的交易價格。 一般用途 v2 儲存體帳戶支援這些 Azure 儲存體服務：

- Blobs (所有類型)
- 檔案
- 磁碟
- 佇列
- 資料表

Microsoft 建議，在大部分情況下使用一般用途 v2 儲存體帳戶。 您不需停機或重寫應用程式，也不必複製資料，即可輕鬆地將一般用途 v1 或 Blob 儲存體帳戶升級至一般用途 v2 帳戶。 如需有關如何升級至一般用途 v2 帳戶的詳細資訊，請參閱[升級至一般用途 v2 儲存體帳戶](storage-account-upgrade.md)。 

一般用途 v2 儲存體帳戶提供多個存取層，以便根據您的使用量模式來儲存資料。 如需詳細資訊，請參閱[區塊 blob 資料的存取層](#access-tiers-for-block-blob-data)。

### <a name="general-purpose-v1-accounts"></a>一般用途 v1 帳戶

一般用途 v1 帳戶可讓使用者存取所有 Azure 儲存體服務，但可能不包含最新功能，每 GB 的定價也並非最低。 一般用途 v1 儲存體帳戶支援這些 Azure 儲存體服務：

- Blobs (所有類型)
- 檔案
- 磁碟
- 佇列
- 資料表

雖然建議在大部分情況下使用一般用途 v2 帳戶，但一般用途 v1 帳戶最適合下列案例： 

* 您的應用程式需要 Azure 傳統部署模型。 一般用途 v2 帳戶和 Blob 儲存體帳戶僅支援 Azure Resource Manager 部署模型。

* 您的應用程式屬於交易密集或使用大量異地複寫頻寬，但不需要大容量。 在此情況下，一般用途 v1 可能是最經濟實惠的選擇。

* 您使用早於 2014-02-14 的 [儲存體服務 REST API](https://msdn.microsoft.com/library/azure/dd894041.aspx) 版本，或版本低於 4.x 的用戶端程式庫，所以無法升級您的應用程式。

### <a name="blob-storage-accounts"></a>Blob 儲存體帳戶

Blob 儲存體帳戶是特製化儲存體帳戶，可將非結構化物件資料儲存為區塊 Blob。 Blob 儲存體帳戶會提供與一般用途 v2 儲存體帳戶相同的持久性、可用性、延展性和效能功能。 Blob 儲存體帳戶支援儲存區塊 Blob 和附加 Blob，但不支援儲存分頁 Blob。

Blob 儲存體帳戶提供多個存取層，以便根據您的使用量模式來儲存資料。 如需詳細資訊，請參閱[區塊 blob 資料的存取層](#access-tiers-for-block-blob-data)。

## <a name="naming-storage-accounts"></a>儲存體帳戶命名

為您的儲存體帳戶命名時，請記住這些規則：

- 儲存體帳戶名稱必須介於 3 到 24 個字元的長度，而且只能包含數字和小寫字母。
- 儲存體帳戶名稱必須在 Azure 中是獨一無二的。 任兩個儲存體帳戶名稱不得相同。

## <a name="performance-tiers"></a>效能層級

可以針對下列任一效能層級設定一般用途儲存體帳戶：

* 標準效能層可供儲存 Blob、檔案、資料表、佇列和 Azure 虛擬機器磁碟。
* 進階效能層僅供儲存 Azure 虛擬機器磁碟。 如需進階儲存體的深入概觀，請參閱 [進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](../../virtual-machines/windows/premium-storage.md) 。

## <a name="access-tiers-for-block-blob-data"></a>區塊 blob 資料的存取層

Azure 儲存體提供不同的選項，以便根據使用量模式來存取區塊 blob 資料。 Azure 儲存體中的每個存取層都已針對特定的資料使用量模式最佳化。 針對您的需求選取適當的存取層，您即可以最具成本效益的方式儲存區塊 blob 資料。

可用的存取層如下：

* **經常性**存取層，最適合用於儲存體帳戶中經常存取的物件。 存取經常性存取層中的資料最具成本效益，然而儲存成本稍高。 預設會在經常性存取層中建立新的儲存體帳戶。
* **非經常性**存取層，最適合用於儲存不常存取且至少儲存 30 天的大量資料。 在非經常性存取層中儲存資料更符合成本效益，但是存取該資料可能比存取經常性存取層中的資料更為昂貴。
* **封存**存取層，只適用於個別的區塊 blob。 封存存取層最適合於可承受擷取延遲數個小時，而且將保留在封存層中至少 180 天的資料。 封存層是最具成本效益的資料儲存選項，但是存取該資料可能比存取經常性或非經常性存取層中的資料更為昂貴。 

> [!NOTE]
> [進階存取層](../blobs/storage-blob-storage-tiers.md#premium-access-tier)會在北歐、美國東部 2、美國中部和美國西部區域，以有限預覽狀態的本地備援儲存體 (LRS) 帳戶形式提供。 若要了解如何註冊預覽版，請參閱 [Azure 進階 Blob 儲存體簡介](http://aka.ms/premiumblob)。

如果您的資料使用模式有變動，您可以隨時在這些存取層之間切換。 

> [!IMPORTANT]
> 變更現有儲存體帳戶或 Blob 的存取層可能會導致額外的費用。

如需存取層的詳細資訊，請參閱 [Azure Blob 儲存體︰進階 (預覽)、經常性存取、非經常性存取和封存儲存層](../blobs/storage-blob-storage-tiers.md)。

## <a name="replication"></a>複寫

[!INCLUDE [storage-common-redundancy-options](../../../includes/storage-common-redundancy-options.md)]

如需儲存體複寫的詳細資訊，請參閱 [Azure 儲存體複寫](storage-redundancy.md)。

## <a name="encryption"></a>加密

您儲存體帳戶中的所有資料都會在服務端加密。 如需加密的詳細資訊，請參閱[待用資料的 Azure 儲存體服務加密](storage-service-encryption.md)。

## <a name="storage-account-endpoints"></a>儲存體帳戶端點

儲存體帳戶會在 Azure 中為您的資料提供唯一命名空間。 每個儲存在 Azure 儲存體中的物件都有一個位址，其中包含您的唯一帳戶名稱。 帳戶名稱與 Azure 儲存體服務端點的組合會形成儲存體帳戶的端點。

例如，如果您的一般用途儲存體帳戶名為 mystorageaccount，則該帳戶的預設端點如下：

* Blob 儲存體： http://*mystorageaccount*.blob.core.windows.net
* 資料表儲存體： http://*mystorageaccount*.table.core.windows.net
* 佇列儲存體： http://*mystorageaccount*.queue.core.windows.net
* Azure 檔案服務： http://*mystorageaccount*.file.core.windows.net

> [!NOTE]
> Blob 儲存體帳戶只會公開 Blob 服務端點。

用以存取儲存體帳戶中某物件的 URL，可藉由在端點後附加該物件在儲存體帳戶中的位置來建構。 例如，blob 位址可能會有如下格式︰ http://*mystorageaccount*.blob.core.windows.net/*mycontainer*/*myblob*。

您也可以將儲存體帳戶設定為使用 Blob 的自訂網域。 如需詳細資訊，請參閱[為 Azure 儲存體帳戶設定自訂網域名稱](../blobs/storage-custom-domain-name.md)。  

## <a name="control-access-to-account-data"></a>控制帳戶資料的存取

根據預設，您帳戶中的資料只有帳戶擁有者 (也就是您) 可以使用。 您可以控制誰可以存取您的資料，以及他們具有哪些權限。

對您的儲存體帳戶提出的每個要求都必須經過授權。 在服務層級，要求必須包含有效的「授權」標頭，其中包含服務執行前驗證要求所需的資訊。

您可以使用下列任何一種方法，授與您儲存體帳戶中資料的存取權：

- **Azure Active Directory：** 使用 Azure Active Directory (Azure AD) 認證來驗證使用者、群組或其他身分識別，以便存取 blob 和佇列資料 (預覽)。 如果身分識別驗證成功，Azure AD 會傳回一個權杖，以使用於對 Azure Blob 儲存體或佇列儲存體的要求授權。 如需詳細資訊，請參閱[使用 Azure Active Directory 來驗證 Azure 儲存體的存取權 (預覽)](storage-auth-aad.md)。
- **共用金鑰授權：** 使用儲存體帳戶存取金鑰來建構一個連接字串，以便您的應用程式在執行階段用來存取 Azure 儲存體。 連接字串中的值用來建構會傳遞至 Azure 儲存體的「授權」標頭。 如需詳細資訊，請參閱[設定 Azure 儲存體連接字串](storage-configure-connection-string.md)。
- **共用存取簽章：** 如果您並未使用 Azure AD 驗證，請使用共用存取簽章來委派儲存體帳戶中資源的存取權。 共用存取簽章是一個權杖，可封裝在 URL 上對 Azure 儲存體的要求授權所需的資訊。 您可以將儲存體資源、授與的權限，以及權限有效的間隔指定為共用存取簽章的一部分。 如需詳細資訊，請參閱[使用共用存取簽章 (SAS)](storage-dotnet-shared-access-signature-part-1.md)。

> [!NOTE]
> 使用 Azure AD 認證來驗證使用者或應用程式，可提供比其他授權方法更高的安全性，也更容易使用。 雖然您可以繼續使用共用金鑰授權於應用程式，但使用 Azure AD 就不需要將帳戶存取金鑰和程式碼一起儲存。 您也可以繼續使用共用存取簽章 (SAS) 將細部存取權授與儲存體帳戶中的資源，但 Azure AD 提供類似功能，卻不必管理 SAS 權杖或擔心需要撤銷遭盜用的 SAS。 
>
> Microsoft 建議 Azure 儲存體 blob 和佇列應用程式儘可能使用 Azure AD 驗證。

## <a name="copying-data-into-a-storage-account"></a>將資料複製到儲存體帳戶中

Microsoft 會提供一些公用程式和程式庫，以便將從內部部署儲存裝置或第三方雲端儲存體提供者匯入您的資料。 您使用的解決方案取決於您要傳送的資料數量。 

當您從一般用途 v1 或 Blob 儲存體帳戶升級至一般用途 v2 儲存體帳戶時，您的資料會自動遷移。 Microsoft 建議使用此路徑來升級您的帳戶。 不過，如果您決定將資料從一般用途 v1 帳戶移至 Blob 儲存體帳戶，您必須使用下列工具和程式庫來手動遷移您的資料。 

### <a name="azcopy"></a>AzCopy

AzCopy 為 Windows 命令列公用程式，可以極高效能將資料複製到 Azure 儲存體，以及從 Azure 儲存體複製資料。 您可以使用 AzCopy 將資料從現有一般用途的儲存體帳戶複製到 Blob 儲存體帳戶中，或從內部部署儲存體裝置上傳資料。 如需詳細資訊，請參閱[使用 AzCopy 命令列公用程式傳輸資料](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。

### <a name="data-movement-library"></a>資料移動程式庫

適用於 .NET 的 Azure 儲存體資料移動程式庫是以支援 AzCopy 的核心資料移動架構為基礎。 此程式庫是針對類似於 AzCopy 的高效能、可靠且簡單的資料傳輸作業而設計的。 這可讓您充分受惠於 AzCopy 在您的應用程式中以原生方式提供的功能，而無需處理 AzCopy 外部執行個體的執行和監視。 如需詳細資訊，請參閱 [適用於 .Net 的 Azure 儲存體資料移動程式庫](https://github.com/Azure/azure-storage-net-data-movement)

### <a name="rest-api-or-client-library"></a>REST API 或用戶端程式庫

您可以建立自訂應用程式，以使用其中一個 Azure 用戶端程式庫或 Azure 儲存體服務 REST API，將您的資料移轉至 Blob 儲存體帳戶。 Azure 儲存體針對以下多種語言和平台提供豐富的用戶端程式庫：例如 .NET、Java、C++、Node.JS、PHP、Ruby 和 Python。 這些用戶端程式庫提供多種進階功能，例如大重試邏輯、記錄與並行上傳等等。 您也可以直接透過 REST API 開發，它可以透過提出 HTTP/HTTPS 要求的任何語言進行呼叫。

如需 Azure 儲存體 REST API 的詳細資訊，請參閱 [Azure 儲存體服務 REST API 參考](https://docs.microsoft.com/rest/api/storageservices/)。 

> [!IMPORTANT]
> 使用用戶端加密來加密的 Blob 會儲存 Blob 加密相關中繼資料。 如果您複製使用用戶端加密來加密的 blob，請確定複製作業會保留 blob 中繼資料，特別是加密相關中繼資料。 如果您複製不含加密中繼資料的 Blob，便無法再次擷取 Blob 內容。 若想進一步了解與加密有關的中繼資料，請參閱 [Azure 儲存體用戶端加密](../common/storage-client-side-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。

### <a name="azure-importexport-service"></a>Azure 匯入/匯出服務

如果您有大量資料要匯入至儲存體帳戶，請考慮 Azure 匯入/匯出服務。 匯入/匯出服務可用來將硬碟運送到 Azure 資料中心，安全地將大量資料匯入 Azure Blob 儲存體和 Azure 檔案服務中。 

匯入/匯出服務也能用來將資料從 Azure Blob 儲存體傳輸到磁碟機，然後運送到您的內部部署網站。 來自一或多個磁碟機的資料可匯入 Azure Blob 儲存體或 Azure 檔案服務中。 如需詳細資訊，請參閱[什麼是 Azure 匯入/匯出服務？](https://docs.microsoft.com/azure/storage/common/storage-import-export-service)

## <a name="storage-account-billing"></a>儲存體帳戶計費

[!INCLUDE [storage-account-billing-include](../../../includes/storage-account-billing-include.md)]

## <a name="next-steps"></a>後續步驟

* 若要了解如何建立 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](storage-quickstart-create-account.md)。
* 若要管理或刪除現有的儲存體帳戶，請參閱[管理 Azure 儲存體帳戶](storage-account-manage.md)。
