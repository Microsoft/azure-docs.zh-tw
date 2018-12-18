---
title: Azure 搜尋服務中的安全性和資料隱私權 | Microsoft Docs
description: Azure 搜尋服務符合 SOC 2、HIPAA 及其他認證規範。 連線與加密、驗證，以及透過「Azure 搜尋服務」篩選條件中的使用者和群組安全性識別碼來進行的身分識別存取。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 09/06/2018
ms.author: heidist
ms.openlocfilehash: 4b1307aa00fae26d7425c9a95ed673b11ba2e9b4
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44092626"
---
# <a name="security-and-data-privacy-in-azure-search"></a>Azure 搜尋服務中的安全性和資料隱私權

Azure 搜尋服務中內建完整的安全性功能及存取控制，藉此確保私用內容會可保持原樣。 本文將列舉 Azure 搜尋服務內建的安全性功能和標準合規性。

Azure 搜尋服務具備跨實體安全性、加密傳輸、加密儲存體及全平台標準合規性的安全性架構。 在操作方面，「Azure 搜尋服務」只接受經過驗證的要求。 您可以視需要透過安全性篩選，在內容上新增個別使用者存取控制。 本文涵蓋每一層的安全性，但主要著重於資料和作業如何在「Azure 搜尋服務」中受到保護。

## <a name="standards-compliance-iso-27001-soc-2-hipaa"></a>標準合規性：ISO 27001、SOC 2、HIPAA

如 [2018 年 6 月的公告](https://azure.microsoft.com/blog/azure-search-is-now-certified-for-several-levels-of-compliance/)，Azure 搜尋服務已通過下列標準的認證：

+ [ISO 27001:2013](https://www.iso.org/isoiec-27001-information-security.html) 
+ [SOC 2 Type 2 合規性](https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/aicpasoc2report.html) 如需完整的報告，請移至 [Azure - 和 Azure Government SOC 2 Type II 報告](https://servicetrust.microsoft.com/ViewPage/MSComplianceGuide?command=Download&downloadType=Document&downloadId=93292f19-f43e-4c4e-8615-c38ab953cf95&docTab=4ce99610-c9c0-11e7-8c2c-f908a777fa4d_SOC%20%2F%20SSAE%2016%20Reports) \(英文\)。 
+ [健康保險流通與責任法案 (HIPAA)](https://en.wikipedia.org/wiki/Health_Insurance_Portability_and_Accountability_Act)
+ [GxP (法規 21 章第 11 條)](https://en.wikipedia.org/wiki/Title_21_CFR_Part_11)
+ [HITRUST](https://en.wikipedia.org/wiki/HITRUST)
+ [PCI DSS 層級 1](https://en.wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)
+ [澳洲 IRAP 的未分類 DLM](https://asd.gov.au/infosec/irap/certified_clouds.htm)

標準合規性適用於正式推出的功能。 預覽功能在轉換為正式推出的功能時會通過認證，且不得用於具有嚴格標準需求的解決方案。 合規性憑證會記載於 [Microsoft Azure 合規性概觀](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942)和[信任中心](https://www.microsoft.com/en-us/trustcenter)。 

## <a name="encrypted-transmission-and-storage"></a>加密的傳輸和儲存體

加密會延伸至整個索引編製管線：從連線到傳輸，再到儲存在「Azure 搜尋服務」中已編製索引的資料。

| 安全性階層 | 說明 |
|----------------|-------------|
| 傳輸中加密 <br>(HTTPS/SSL/TLS) | 「Azure 搜尋服務」會在 HTTPS 連接埠 443 上進行接聽。 在整個平台上，連至 Azure 服務的連線都會受到加密。 <br/><br/>用戶端對服務的所有 Azure 搜尋服務互動都支援 SSL/TLS 1.2。  請務必對服務的 SSL 連線使用 TLSv1.2。|
| 待用加密 | 加密會完全在索引編製程序內進行，對索引編製完成時間或索引大小沒有任何重大的影響。 它會自動針對所有索引編製程序進行，包括針對未完全加密 (建立時間在 2018 年 1 月以前) 之索引的增量更新進行。<br><br>就內部而言，加密會根據 [Azure 儲存體服務加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption) \(機器翻譯\)，使用的是 256 位元的 [AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) \(英文\)。|

加密是在「Azure 搜尋服務」內部進行的，由 Microsoft 在內部管理憑證和加密金鑰，並且全面套用。 您無法在入口網站中或透過程式設計方式，開啟或關閉加密、管理或替代自己的金鑰，或是檢視加密設定。 

在 2018 年 1 月 24 日已宣布推出待用加密功能，並且適用於所有區域中的所有服務層，包括共用 (免費) 服務。 若要達到完整加密的目的，針對在該日期之前建立的索引，您必須先捨棄再重新建置，才能進行加密。 否則，系統只會加密 1 月 24 日之後新增的新資料。

## <a name="azure-wide-user-access-controls"></a>整個 Azure 的使用者存取控制

整個 Azure 有數個可用的安全性機制，因此會自動提供給您建立的「Azure 搜尋服務」資源使用。

+ [可防止刪除的訂用帳戶或資源層級鎖定](../azure-resource-manager/resource-group-lock-resources.md)
+ [可控制對資訊和系統管理作業之存取的角色型存取控制 (RBAC)](../role-based-access-control/overview.md)

所有 Azure 服務都支援角色型存取控制 (RBAC)，可在所有服務以一致的方式設定存取層級。 例如，限制只有「擁有者」和「參與者」角色才能檢視敏感性資料 (例如管理金鑰)，而讓所有角色的成員都可以檢視服務狀態。 RBAC 提供的角色包括「擁有者」、「參與者」及「讀者」。 根據預設，所有服務管理員都是擁有者角色的成員。

## <a name="service-access-and-authentication"></a>服務存取與驗證

雖然「Azure 搜尋服務」繼承 Azure 平台的安全性防護措施，但它也提供自己的金鑰型驗證。 API 金鑰是由隨機產生的數字和字母所組成的字串。 金鑰的類型 (管理或查詢) 會決定存取層級。 提交有效的金鑰可證明要求源自受信任的實體。 存取搜尋服務的金鑰有兩種類型：

* 管理員 (適用於服務的任何讀寫作業)
* 查詢 (適用於唯讀作業，例如針對索引的查詢)

佈建服務時會建立管理金鑰。 有兩個管理員金鑰，指定為主要和次要以將它們保持在各自的位置，但事實上，它們是可互換。 每個服務有兩個管理員金鑰，以便您在轉換其中一個時不會無法存取您的服務。 您可以重新產生任何一種管理員金鑰，但您無法增加管理員金鑰的總數量。 每個搜尋服務最多有 2 個管理員金鑰。

查詢金鑰是視需要建立的，專為直接呼叫搜尋的用戶端應用程式所設計。 您最多可以建立 50 個查詢金鑰。 在應用程式程式碼中，您可以指定搜尋 URL 和查詢 API 金鑰以允許唯讀存取服務。 應用程式程式碼也會指定應用程式所使用的索引。 端點、可供唯讀存取的 API 金鑰以及目標索引共同定義來自用戶端應用程式連接的範圍和存取層級。

針對每個要求都必須進行驗證，其中每個要求會由強制性金鑰、作業及物件所組成。 當兩個權限層級 (完整和唯讀) 加上內容 (例如，索引上的查詢作業) 鏈結在一起時，即足以在服務作業上提供全面的安全性。 如需關於金鑰的詳細資訊，請參閱[建立及管理 API 金鑰](search-security-api-keys.md)。

## <a name="index-access"></a>索引存取權

在「Azure 搜尋服務」中，個別索引是無法保護的物件。 取而代之的是，會在服務層搭配作業的內容來決定對索引的存取權 (讀取或寫入存取權)。

就使用者存取來說，您可以將查詢要求建構成使用查詢金鑰來進行連線 (這可讓所有查詢都變成唯讀)，並且包含應用程式所使用的特定索引。 在查詢要求中，並沒有聯結索引或同時存取多個索引的概念，所以所有要求都會依據定義以單一索引為目標。 因此，查詢要求本身的結構 (一個金鑰加上單一目標索引) 即可定義安全性界限。

系統管理員和開發人員對索引的存取權並無差別：兩者都需要有寫入存取權，才能建立、刪除及更新服務所管理的物件。 任何人只要擁有您服務的管理金鑰，便可讀取、修改或刪除該相同服務中的任何索引。 在防止發生意外或惡意刪除索引的情況方面，您內部對程式碼資產實施的來源控制，將會是反轉不需要之索引刪除或修改的補救措施。 「Azure 搜尋服務」在叢集內具有容錯移轉功能來確保可用性，但它不會儲存或執行用來建立或載入索引的專屬程式碼。

以需要索引層級之安全性界限的多組織用戶管理解決方案來說，這類解決方案通常會包含可供客戶用來處理索引隔離的中間層。 如需有關多租用戶使用案例的詳細資訊，請參閱[多租用戶 SaaS 應用程式與 Azure 搜尋服務的設計模式](search-modeling-multitenant-saas-applications.md)。

## <a name="admin-access-from-client-apps"></a>來自用戶端應用程式的管理存取

Azure 搜尋服務管理 REST API 為 Azure 資源管理員的擴充功能並且共享相依性。 因此，Active Directory 是 Azure 搜尋服務管理的先決條件。 所有來自用戶端程式碼的系統管理要求都必須先經過 Azure Active Directory 驗證，然後才會傳送給 Resource Manager。

對「Azure 搜尋服務」端點發出的資料要求 (例如建立索引 (Azure 搜尋服務 REST API) 或搜尋文件 (Azure 搜尋服務 REST API)) 會在要求標頭中使用 API 金鑰。

如果您的應用程式程式碼除了處理搜尋索引上的資料作業之外，也會處理服務系統管理作業，請在您的程式碼中實作兩個驗證方法：「Azure 搜尋服務」原生的存取金鑰，以及 Resource Manager 所需的 Active Directory 驗證方法。 

如需有關在「Azure 搜尋服務」中建構要求的資訊，請參閱 [Azure 搜尋服務 REST API](https://docs.microsoft.com/rest/api/searchservice/) \(英文\)。 如需有關 Resource Manager 之驗證需求的詳細資訊，請參閱[使用 Resource Manager 驗證 API 來存取訂用帳戶](../azure-resource-manager/resource-manager-api-authentication.md)。

## <a name="user-access-to-index-content"></a>使用者對索引內容的存取權

實作個別使用者對索引內容的存取權時，會透過您查詢上的安全性篩選來實作，這會傳回與指定之安全性身分識別關聯的文件。 身分識別型存取控制並不使用預先定義的角色和角色指派，而是以篩選的形式來實作，可根據身分識別縮減文件和內容的搜尋結果。 下表說明兩個針對未經授權的內容縮減搜尋結果的方法。

| 方法 | 說明 |
|----------|-------------|
|[根據身分識別篩選進行安全性範圍縮減](search-security-trimming-for-azure-search.md)  | 記載實作使用者身分識別存取控制的基本工作流程。 其中涵蓋在索引中新增安全性識別碼，然後說明如何針對該欄位進行篩選來縮減所禁止內容的結果。 |
|[根據 Azure Active Directory 身分識別進行安全性範圍縮減](search-security-trimming-for-azure-search-with-aad.md)  | 本文是上一篇文章的延伸，提供從 Azure Active Directory (AAD) (Azure 雲端平台上的其中一項[免費服務](https://azure.microsoft.com/free/)) 擷取身分識別的步驟。 |

## <a name="table-permissioned-operations"></a>表格：許可的作業

下表摘要說明「Azure 搜尋服務」中允許的作業，以及哪個金鑰可開啟對特定作業的存取權。

| 作業 | 權限 |
|-----------|-------------------------|
| 建立服務 | Azure 訂用帳戶持有者|
| 調整服務規模 | 管理金鑰、資源的 RBAC 擁有者或參與者  |
| 刪除服務 | 管理金鑰、資源的 RBAC 擁有者或參與者 |
| 在服務上建立、修改、刪除物件： <br>索引和元件組件 (包括分析器定義、評分設定檔、CORS 選項)、索引子、資料來源、同義字、建議工具。 | 管理金鑰、資源的 RBAC 擁有者或參與者  |
| 查詢索引 | 管理或查詢金鑰 (RBAC 不適用) |
| 查詢系統資訊，例如傳回統計資料、計數及物件清單。 | 管理金鑰、資源的 RBAC (擁有者、參與者、讀者) |
| 管理管理金鑰 | 管理金鑰、資源的 RBAC 擁有者或參與者。 |
| 管理查詢金鑰 |  管理金鑰、資源的 RBAC 擁有者或參與者。  |

## <a name="physical-security"></a>實體安全性

Microsoft 資料中心提供業界頂尖的實體安全性，符合一組廣泛的標準與法規規範。 若要深入了解，請前往[全球資料中心](https://www.microsoft.com/cloud-platform/global-datacenters)頁面，或觀看有關資料中心安全性的短片。

> [!VIDEO https://www.youtube.com/embed/r1cyTL8JqRg]


## <a name="see-also"></a>另請參閱

+ [開始使用 .NET (使用管理金鑰來建立索引以進行示範)](search-create-index-dotnet.md)
+ [開始使用 REST (使用管理金鑰來建立索引以進行示範)](search-create-index-rest-api.md)
+ [使用 Azure 搜尋服務篩選來進行身分識別型存取控制](search-security-trimming-for-azure-search.md)
+ [使用 Azure 搜尋服務篩選來進行 Active Directory 身分識別型存取控制](search-security-trimming-for-azure-search-with-aad.md)
+ [Azure 搜尋服務中的篩選條件](search-filters.md)