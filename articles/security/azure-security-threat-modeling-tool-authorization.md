---
title: 授權 - Microsoft Threat Modeling Tool - Azure | Microsoft Docs
description: 降低威脅模型化工具所暴露的威脅
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.openlocfilehash: eec628a5084dc663978e16e617192802d3ecfcfa
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307651"
---
# <a name="security-frame-authorization--mitigations"></a>安全框架︰授權 | 緩和措施 
| 產品/服務 | 文章 |
| --------------- | ------- |
| **電腦信任邊界** | <ul><li>[確保設定正確的 ACL 以將未經授權的存取限制於裝置上的資料](#acl-restricted-access)</li><li>[確保敏感性使用者特有應用程式內容會儲存在使用者設定檔目錄中](#sensitive-directory)</li><li>[確保已部署的應用程式是以最低權限執行](#deployed-privileges)</li></ul> |
| **Web 應用程式** | <ul><li>[在處理商務邏輯流程時強制執行循序步驟順序](#sequential-logic)</li><li>[實作速率限制機制以防止列舉](#rate-enumeration)</li><li>[確保施行適當的授權並遵循最低權限的原則](#principle-least-privilege)</li><li>[商務邏輯和資源存取授權決策不應以傳入的要求參數為基礎](#logic-request-parameters)</li><li>[確保無法透過強迫瀏覽來列舉或存取內容和資源](#enumerable-browsing)</li></ul> |
| **資料庫** | <ul><li>[確保使用最低權限的帳戶連線到資料庫伺服器](#privileged-server)</li><li>[實作資料列層級安全性 RLS，以防止租用戶存取彼此的資料](#rls-tenants)</li><li>[系統管理員角色只能具備有效的必要使用者](#sysadmin-users)</li></ul> |
| **IoT 雲端閘道** | <ul><li>[使用最低權限的權杖連線到雲端閘道](#cloud-least-privileged)</li></ul> |
| **Azure 事件中樞** | <ul><li>[使用僅限傳送權限 SAS 金鑰來產生裝置權杖](#sendonly-sas)</li><li>[請勿使用可供直接存取事件中樞的存取權杖](#access-tokens-hub)</li><li>[使用具有所需最低權限的 SAS 金鑰來連線到事件中樞](#sas-minimum-permissions)</li></ul> |
| **Azure Document DB** | <ul><li>[盡可能使用資源權杖來連線到 Azure Cosmos DB](#resource-docdb)</li></ul> |
| **Azure 信任邊界** | <ul><li>[使用 RBAC 啟用 Azure 訂用帳戶的精細存取管理](#grained-rbac)</li></ul> |
| **Service Fabric 信任邊界** | <ul><li>[使用 RBAC 限制用戶端對於叢集作業的存取](#cluster-rbac)</li></ul> |
| **Dynamics CRM** | <ul><li>[執行安全性模型化並視需要使用欄位層級安全性](#modeling-field)</li></ul> |
| **Dynamics CRM 入口網站** | <ul><li>[執行入口網站帳戶的安全性模型化，並記住，入口網站的安全性模型與 CRM 的其餘部分不同](#portal-security)</li></ul> |
| **Azure 儲存體** | <ul><li>[在 Azure 表格儲存體中授與某個實體範圍的精細權限](#permission-entities)</li><li>[使用 Azure Resource Manager 啟用 Azure 儲存體帳戶的角色型存取控制 (RBAC)](#rbac-azure-manager)</li></ul> |
| **行動用戶端** | <ul><li>[實作隱含的越獄或 Root 權限入侵偵測](#rooting-detection)</li></ul> |
| **WCF** | <ul><li>[WCF 中的弱式類別參考](#weak-class-wcf)</li><li>[WCF 實作授權控制](#wcf-authz)</li></ul> |
| **Web API** | <ul><li>[在 ASP.NET Web API 中實作適當的授權機制](#authz-aspnet)</li></ul> |
| **IoT 裝置** | <ul><li>[如果支援各種需要不同權限層級的動作，則在裝置中執行授權檢查](#device-permission)</li></ul> |
| **IoT 現場閘道** | <ul><li>[如果支援各種需要不同權限層級的動作，則在現場閘道中執行授權檢查](#field-permission)</li></ul> |

## <a id="acl-restricted-access"></a>確保設定正確的 ACL 以將未經授權的存取限制於裝置上的資料

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 電腦信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確保設定正確的 ACL 以將未經授權的存取限制於裝置上的資料|

## <a id="sensitive-directory"></a>確保敏感性使用者特有應用程式內容會儲存在使用者設定檔目錄中

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 電腦信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確保敏感性使用者特有應用程式內容會儲存在使用者設定檔目錄中。 這是為了防止多位電腦使用者存取彼此的資料。|

## <a id="deployed-privileges"></a>確保已部署的應用程式是以最低權限執行

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 電腦信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確保已部署的應用程式是以最低權限執行。 |

## <a id="sequential-logic"></a>在處理商務邏輯流程時強制執行循序步驟順序

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 若要確認此階段是經由正版使用者執行，您可以強制應用程式只能依照循序步驟順序處理商務邏輯流程 (以現實人類時間處理所有的步驟)，而不處理順序混亂、另一位使用者已略過的步驟、已處理的步驟，或太快提交的交易。|

## <a id="rate-enumeration"></a>實作速率限制機制以防止列舉

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確保敏感性識別碼是隨機的。 在匿名頁面上實作 CAPTCHA 控制。 確保錯誤和例外狀況不得顯示特定資料|

## <a id="principle-least-privilege"></a>確保施行適當的授權並遵循最低權限的原則

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>原則表示只能夠提供該使用者所需的權限給使用者帳戶。 例如，備份使用者不必安裝軟體︰因此，備份使用者只有權限執行備份和備份相關應用程式。 任何其他權限 (例如安裝新的軟體) 都會遭到封鎖。 此原則也適用於通常在一般使用者帳戶中作業的個人電腦使用者，而且只會在情況絕對必要時，才會開啟特殊權限、受密碼保護的帳戶 (也就是超級使用者)。 </p><p>此原則也可適用於您的 Web 應用程式。 與其完全相依於使用工作階段的角色型驗證方法，我們寧願透過以資料庫為基礎的驗證系統將權限指派給使用者。 我們仍使用工作階段，以便識別使用者是否已正確登入，只是現在我們不會將特定角色指派給該使用者，而會將相關權限指派給他，以確認他有權在系統上執行哪些動作。 此外，這個方法的大問題是，每當使用者必須獲派較少的權限時，便會立即套用您的變更，因為指派不會依賴必須先到期的工作階段。</p>|

## <a id="logic-request-parameters"></a>商務邏輯和資源存取授權決策不應以傳入的要求參數為基礎

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 每當您檢查使用者是否受限於檢閱特定資料時，應在伺服器端處理存取限制。 UserID 應儲存在登入時的工作階段變數內，且應用於從資料庫擷取使用者資料 |

### <a name="example"></a>範例
```SQL
SELECT data 
FROM personaldata 
WHERE userID=:id < - session var 
```
可能的攻擊者目前無法竄改及變更應用程式作業，因為會在伺服器端處理用於擷取資料的識別碼。

## <a id="enumerable-browsing"></a>確保無法透過強迫瀏覽來列舉或存取內容和資源

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>敏感性靜態和組態檔不得保留在 web 根目錄中。 對於不需要公開的內容，則應套用適當的存取控制項，或移除內容本身。</p><p>此外，強迫瀏覽通常結合暴力密碼破解技術，會儘可能嘗試存取較多的 URL 以列舉伺服器上的目錄和檔案，藉此搜集資訊。 攻擊者可能會檢查常見現有檔案的所有變化。 例如，密碼檔案搜尋會包含下列檔案：psswd.txt、password.htm、password.dat 和其他變化。</p><p>若要避免這個問題，應包含暴力密碼破解嘗試偵測功能。</p>|

## <a id="privileged-server"></a>確保使用最低權限的帳戶連線到資料庫伺服器

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [SQL Database 權限階層](https://msdn.microsoft.com/library/ms191465)、[SQL Database 安全性實體](https://msdn.microsoft.com/library/ms190401) |
| **步驟** | 應該使用最低權限的帳戶連線到資料庫。 應用程式登入應受限於資料庫，應該只能執行所選的程序。 應用程式的登入應該沒有直接的資料表存取權。 |

## <a id="rls-tenants"></a>實作資料列層級安全性 RLS，以防止租用戶存取彼此的資料

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | Sql Azure、OnPrem |
| **屬性**              | SQL 版本 - V12、SQL 版本 - MsSQL2016 |
| **參考**              | [SQL Server 資料列層級安全性 (RLS)](https://msdn.microsoft.com/library/azure/dn765131.aspx) |
| **步驟** | <p>資料列層級安全性讓客戶能夠根據執行查詢之使用者的特性 (例如，群組成員資格或執行內容) 來控制資料庫資料表中的資料列存取。</p><p>資料列層級安全性 (RLS) 可簡化應用程式安全性的設計和編碼。 RLS 可讓您實作資料的資料列存取限制。 例如，確保背景工作角色只能存取其部門相關資料列，或將客戶的資料存取權限制為其公司的相關資料。</p><p>存取限制邏輯位於資料庫層，而不是遠離另一個應用程式層中的資料。 資料庫系統會在每次嘗試從任何層存取該資料時套用存取限制。 這可藉由縮小安全性系統的接觸區，讓安全性系統更加可靠和健全。</p><p>|

請注意，最新的資料庫功能 RLS 只適用於 2016 以後的 SQL Server 和 Azure SQL Database。 如果未實作最新的 RLS 功能，則應確保資料存取是受限的。使用檢視和程序

## <a id="sysadmin-users"></a>系統管理員角色只能具備有效的必要使用者

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [SQL Database 權限階層](https://msdn.microsoft.com/library/ms191465)、[SQL Database 安全性實體](https://msdn.microsoft.com/library/ms190401) |
| **步驟** | SysAdmin 固定伺服器角色的成員應該非常有限，而且永遠不會包含應用程式所用的帳戶。  請檢閱角色的使用者清單，並移除任何不必要的帳戶|

## <a id="cloud-least-privileged"></a>使用最低權限的權杖連線到雲端閘道

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 雲端閘道 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | 閘道選擇 - Azure IoT 中樞 |
| **參考**              | [IoT 中樞存取控制](https://azure.microsoft.com/documentation/articles/iot-hub-devguide/#Security) |
| **步驟** | 提供連線到雲端閘道 (IoT 中樞) 之各種元件的最低權限。 典型範例 – 裝置管理/佈建元件使用登錄讀取/寫入時、事件處理器 (ASA) 使用服務連線。 使用裝置認證的個別裝置連接|

## <a id="sendonly-sas"></a>使用僅限傳送權限 SAS 金鑰來產生裝置權杖

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 事件中樞 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [事件中樞驗證和安全性模型概觀](https://azure.microsoft.com/documentation/articles/event-hubs-authentication-and-security-model-overview/) |
| **步驟** | SAS 金鑰用來產生個別的裝置權杖。 針對指定的發佈者產生裝置權杖時使用僅限傳送的權限 SAS 金鑰|

## <a id="access-tokens-hub"></a>請勿使用可供直接存取事件中樞的存取權杖

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 事件中樞 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [事件中樞驗證和安全性模型概觀](https://azure.microsoft.com/documentation/articles/event-hubs-authentication-and-security-model-overview/) |
| **步驟** | 應不會將授與事件中樞直接存取權的權杖提供給裝置。 使用裝置的最低權限權杖 (僅可供存取發佈者)，有助於識別惡意或遭到入侵的裝置，並將它列入黑名單。|

## <a id="sas-minimum-permissions"></a>使用具有所需最低權限的 SAS 金鑰來連線到事件中樞

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 事件中樞 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [事件中樞驗證和安全性模型概觀](https://azure.microsoft.com/documentation/articles/event-hubs-authentication-and-security-model-overview/) |
| **步驟** | 將最低權限提供給連線到事件中樞的各種後端應用程式。 為每個後端應用程式產生個別的 SAS 金鑰，並且僅提供必要的權限 (傳送、接收或管理) 給它們。|

## <a id="resource-docdb"></a>盡可能使用資源權杖來連線到 Cosmos DB

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure Document DB | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 資源權杖會與 Azure Cosmos DB 權限資源相關聯，並擷取資料庫使用者與該使用者所具備之特定 Azure Cosmos DB 應用程式資源 (例如集合、文件) 權限之間的關係。 如果無法放心託付用戶端 (例如行動裝置或桌上型電腦用戶端等使用者應用程式) 處理主要或唯讀金鑰，請一律使用資源權杖來存取 Azure Cosmos DB。請從可以安全地儲存主要金鑰或唯讀金鑰的後端應用程式使用這些金鑰。|

## <a id="grained-rbac"></a>使用 RBAC 啟用 Azure 訂用帳戶的精細存取管理

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 信任邊界 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [使用角色指派來管理 Azure 訂用帳戶資源的存取權](https://azure.microsoft.com/documentation/articles/role-based-access-control-configure/)  |
| **步驟** | Azure 角色型存取控制 (RBAC) 可以對 Azure 進行更細緻的存取權管理。 使用 RBAC，您可以僅授與使用者執行其作業所需的存取權。|

## <a id="cluster-rbac"></a>使用 RBAC 限制用戶端對於叢集作業的存取

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Service Fabric 信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | 環境 - Azure |
| **參考**              | [Service Fabric 用戶端的角色型存取控制](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security-roles/) |
| **步驟** | <p>Azure Service Fabric 針對連線到 Service Fabric 叢集的用戶端，支援兩種不同的存取控制類型：系統管理員和使用者。 存取控制可讓叢集系統管理員針對不同的使用者群組限制特定叢集作業的存取權，讓叢集更加安全。</p><p>系統管理員可以完整存取管理功能 (包括讀取/寫入功能)。 使用者預設只具有管理功能的讀取存取權 (例如查詢功能)，以及解析應用程式和服務的能力。</p><p>您可在建立叢集時為每個角色提供個別的憑證，以指定兩個用戶端角色 (系統管理員和用戶端)。</p>|

## <a id="modeling-field"></a>執行安全性模型化並視需要使用欄位層級安全性

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Dynamics CRM | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 執行安全性模型化並視需要使用欄位層級安全性|

## <a id="portal-security"></a>執行入口網站帳戶的安全性模型化，並記住，入口網站的安全性模型與 CRM 的其餘部分不同

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Dynamics CRM 入口網站 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 執行入口網站帳戶的安全性模型化，並記住，入口網站的安全性模型與 CRM 的其餘部分不同|

## <a id="permission-entities"></a>在 Azure 表格儲存體中授與某個實體範圍的精細權限

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 儲存體 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | StorageType - 資料表 |
| **參考**              | [如何使用 SAS 將存取權委派給 Azure 儲存體帳戶中的物件](https://azure.microsoft.com/documentation/articles/storage-security-guide/#_data-plane-security) |
| **步驟** | 在特定商務案例中，Azure 表格儲存體可能必須儲存能滿足不同合作夥伴需求的敏感性資料。 例如，屬於不同的國家/地區的敏感性資料。 在這類情況下，藉由指定資料分割和資料列金鑰範圍來建構 SAS 簽章，使用者便可存取特定國家/地區特有的資料。| 

## <a id="rbac-azure-manager"></a>使用 Azure Resource Manager 啟用 Azure 儲存體帳戶的角色型存取控制 (RBAC)

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 儲存體 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [如何使用角色型存取控制 (RBAC) 保護儲存體帳戶](https://azure.microsoft.com/documentation/articles/storage-security-guide/#management-plane-security) |
| **步驟** | <p>當您建立新的儲存體帳戶時，可以選取傳統或 Azure Resource Manager 的部署模型。 在 Azure 中建立資源的傳統模型只允許以孤注一擲的方式存取訂用帳戶，接著存取儲存體帳戶。</p><p>使用 Azure Resource Manager 模型，您可以將儲存體帳戶放置於資源群組中，並使用 Azure Active Directory 來控制該特定儲存體帳戶之管理平面的存取。 例如，您可以為特定使用者提供存取儲存體帳戶金鑰的能力，而其他使用者可以檢視儲存體帳戶的相關資訊，但是無法存取儲存體帳戶金鑰。</p>|

## <a id="rooting-detection"></a>實作隱含的越獄或 Root 權限入侵偵測

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 行動用戶端 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>如果電話遭到 Root 權限或越獄入侵，應用程式應保護其本身的組態和使用者資料。 Root 權限/越獄入侵意味著未經授權的存取，一般使用者不會在自己的電話上這麼做。 因此在應用程式啟動時，應用程式應具有隱含的偵測邏輯，以偵測電話的 Root 權限是否已遭到入侵。</p><p>偵測邏輯可直接存取通常只有根使用者可以存取的檔案，例如︰</p><ul><li>/system/app/Superuser.apk</li><li>/sbin/su</li><li>/system/bin/su</li><li>/system/xbin/su</li><li>/data/local/xbin/su</li><li>/data/local/bin/su</li><li>/system/sd/xbin/su</li><li>/system/bin/failsafe/su</li><li>/data/local/su</li></ul><p>如果應用程式可以存取上述任何檔案，這表示應用程式是以根使用者身分執行。</p>|

## <a id="weak-class-wcf"></a>WCF 中的弱式類別參考

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、NET Framework 3 |
| **屬性**              | N/A  |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_weak_class_reference) |
| **步驟** | <p>系統會使用弱式類別參考，其可讓攻擊者執行未經授權的程式碼。 程式會參考非唯一識別的使用者定義類別。 當 .NET 載入此弱式識別的類別時，CLR 類型載入器會以指定的順序在下列位置中搜尋此類別︰</p><ol><li>如果已知道類型的組件，則載入器會搜尋組態檔的重新導向位置、GAC、使用組態資訊的目前組件，以及應用程式基底目錄</li><li>如果不知道組件，載入器會搜尋目前組件、mscorlib 以及 TypeResolve 事件處理常式所傳回的位置</li><li>可以使用攔截程序修改此 CLR 搜尋順序，例如類型轉送機制和 AppDomain.TypeResolve 事件</li></ol><p>如果攻擊者建立具有相同名稱的替代類別，並將它放在 CLR 會先載入的替代位置，藉此利用 CLR 搜尋順序，則 CLR 會在無意中執行攻擊者提供的程式碼</p>|

### <a name="example"></a>範例
下列 WCF 組態檔的 `<behaviorExtensions/>` 元素會指示 WCF 將自訂行為類別新增至特定 WCF 擴充功能。
```
<system.serviceModel>
    <extensions>
        <behaviorExtensions>
            <add name=""myBehavior"" type=""MyBehavior"" />
        </behaviorExtensions>
    </extensions>
</system.serviceModel>
```
使用完整 (強式) 名稱來唯一識別類型，可進一步提高您系統的安全性。 在 machine.config 和 app.config 檔案中註冊類型時，使用完整組件名稱。

### <a name="example"></a>範例
下列 WCF 組態檔的 `<behaviorExtensions/>` 元素會指示 WCF 將強式參照的自訂行為類別新增至特定 WCF 擴充功能。
```
<system.serviceModel>
    <extensions>
        <behaviorExtensions>
            <add name=""myBehavior"" type=""Microsoft.ServiceModel.Samples.MyBehaviorSection, MyBehavior,
            Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"" />
        </behaviorExtensions>
    </extensions>
</system.serviceModel>
```

## <a id="wcf-authz"></a>WCF 實作授權控制

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、NET Framework 3 |
| **屬性**              | N/A  |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify Kingdom](https://vulncat.hpefod.com/en/detail?id=desc.semantic.dotnet.wcf_misconfiguration_unauthorized_access) |
| **步驟** | <p>此服務並不會使用授權控制。 當用戶端呼叫特定 WCF 服務時，WCF 會提供各種授權方案，以確認呼叫端有權在伺服器上執行此服務方法。 如果未針對 WCF 服務啟用授權控制，經過驗證的使用者可以獲得權限提升。</p>|

### <a name="example"></a>範例
下列組態會指示 WCF 不要在執行服務時檢查用戶端的授權層級︰
```
<behaviors>
    <serviceBehaviors>
        <behavior>
            ...
            <serviceAuthorization principalPermissionMode=""None"" />
        </behavior>
    </serviceBehaviors>
</behaviors>
```
使用服務授權方案來確認服務方法的呼叫端獲得授權執行這項操作。 WCF 提供兩種模式，並允許定義自訂授權方案。 UseWindowsGroups 模式會使用 Windows 角色和使用者進行驗證，而 UseAspNetRoles 模式會使用 ASP.NET 角色提供者 (例如 SQL Server) 進行驗證。

### <a name="example"></a>範例
下列組態會指示 WCF 在執行 [新增] 服務之前，確定用戶端屬於系統管理員群組︰
```
<behaviors>
    <serviceBehaviors>
        <behavior>
            ...
            <serviceAuthorization principalPermissionMode=""UseWindowsGroups"" />
        </behavior>
    </serviceBehaviors>
</behaviors>
```
然後會宣告此服務，如下所示：
```
[PrincipalPermission(SecurityAction.Demand,
Role = ""Builtin\\Administrators"")]
public double Add(double n1, double n2)
{
double result = n1 + n2;
return result;
}
```

## <a id="authz-aspnet"></a>在 ASP.NET Web API 中實作適當的授權機制

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、MVC5 |
| **屬性**              | N/A、識別提供者 - ADFS、識別提供者 - Azure AD |
| **參考**              | [ASP.NET Web API 中的驗證與授權](http://www.asp.net/web-api/overview/security/authentication-and-authorization-in-aspnet-web-api) |
| **步驟** | <p>如果應用程式依賴 Azure AD 或 ADFS 宣告作為識別提供者，即可從 Azure AD 或 ADFS 宣告擷取應用程式使用者的角色資訊，否則應用程式本身可能會提供此資訊。 在任何這類情況下，自訂授權實作應驗證使用者角色資訊。</p><p>如果應用程式依賴 Azure AD 或 ADFS 宣告作為識別提供者，即可從 Azure AD 或 ADFS 宣告擷取應用程式使用者的角色資訊，否則應用程式本身可能會提供此資訊。 在任何這類情況下，自訂授權實作應驗證使用者角色資訊。</p>

### <a name="example"></a>範例
```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited = true, AllowMultiple = true)]
public class ApiAuthorizeAttribute : System.Web.Http.AuthorizeAttribute
{
        public async override Task OnAuthorizationAsync(HttpActionContext actionContext, CancellationToken cancellationToken)
        {
            if (actionContext == null)
            {
                throw new Exception();
            }

            if (!string.IsNullOrEmpty(base.Roles))
            {
                bool isAuthorized = ValidateRoles(actionContext);
                if (!isAuthorized)
                {
                    HandleUnauthorizedRequest(actionContext);
                }
            }

            base.OnAuthorization(actionContext);
        }

public bool ValidateRoles(actionContext)
{
   //Authorization logic here; returns true or false
}

}
```
應使用上述屬性裝飾所有需要保護的控制器和動作方法。
```csharp
[ApiAuthorize]
public class CustomController : ApiController
{
     //Application code goes here
}
```

## <a id="device-permission"></a>如果支援各種需要不同權限層級的動作，則在裝置中執行授權檢查

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 裝置 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>裝置應授權呼叫端檢查呼叫端是否具有執行所要求動作的必要權限。 例如：假設裝置是可從雲端監視的「智慧型門鎖」，而且會提供遠端鎖定大門等功能。</p><p>只有在有人實際帶著卡片接近大門時，「智慧型門鎖」才會提供解除鎖定功能。 在此案例中，遠端命令和控制項的實作方式應該如下：不提供任何將大門解除鎖定的功能，因為雲端閘道未經授權傳送命令來將大門解除鎖定。</p>|

## <a id="field-permission"></a>如果支援各種需要不同權限層級的動作，則在現場閘道中執行授權檢查

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 現場閘道 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 現場閘道應授權呼叫端檢查呼叫端是否具有執行所要求動作的必要權限。 例如︰系統管理員使用者介面/API 應使用不同的權限來設定與其連線的現場閘道 v/s 裝置。|
