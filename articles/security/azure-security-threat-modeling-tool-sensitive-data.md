---
title: 敏感性資料 - Microsoft 威脅模型化工具 - Azure | Microsoft Docs
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
ms.openlocfilehash: bcd2b2199628a7f717a7ce1d8eba032f5778379b
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43301456"
---
# <a name="security-frame-sensitive-data--mitigations"></a>安全性架構︰敏感性資料 | 風險降低 
| 產品/服務 | 文章 |
| --------------- | ------- |
| **電腦信任邊界** | <ul><li>[確定包含敏感性資訊的二進位檔已經過模糊處理](#binaries-info)</li><li>[考慮使用加密檔案系統 (EFS) 來保護機密的使用者特定資料](#efs-user)</li><li>[確定應用程式儲存在檔案系統上的敏感性資料已加密](#filesystem)</li></ul> | 
| **Web 應用程式** | <ul><li>[確定瀏覽器上不會快取敏感性內容](#cache-browser)</li><li>[加密包含敏感性資料的 Web 應用程式組態檔區段](#encrypt-data)</li><li>[明確停用敏感性表單和輸入內的自動完成 HTML 屬性](#autocomplete-input)</li><li>[確定使用者畫面上顯示的敏感性資料已遮罩](#data-mask)</li></ul> | 
| **資料庫** | <ul><li>[實作動態資料遮罩來限制敏感性資料暴露非特殊權限使用者](#dynamic-users)</li><li>[確定密碼是以 salted 雜湊格式儲存](#salted-hash)</li><li>[確定資料庫資料行中的敏感性資料已加密](#db-encrypted)</li><li>[確定已啟用資料庫層級加密 (TDE)](#tde-enabled)</li><li>[確定資料庫備份已加密](#backup)</li></ul> | 
| **Web API** | <ul><li>[確定與 Web API 相關的敏感性資料未儲存在瀏覽器的儲存體中](#api-browser)</li></ul> | 
| Azure Document DB | <ul><li>[將儲存在 Azure Cosmos DB 中的敏感性資料加密](#encrypt-docdb)</li></ul> | 
| **Azure IaaS VM 信任邊界** | <ul><li>[使用 Azure 磁碟加密來加密虛擬機器所使用的磁碟](#disk-vm)</li></ul> | 
| **Service Fabric 信任邊界** | <ul><li>[將 Service Fabric 應用程式中的密碼加密](#fabric-apps)</li></ul> | 
| **Dynamics CRM** | <ul><li>[執行安全性模型化並視需要使用業務單位/團隊](#modeling-teams)</li><li>[將共用重要實體功能的存取權最小化](#entities)</li><li>[訓練使用者以讓其了解與 Dynamics CRM 共用功能相關聯的風險以及正確的安全性作法](#good-practices)</li><li>[包含開發標準規則來禁止在例外狀況管理中顯示組態詳細資料](#exception-mgmt)</li></ul> | 
| **Azure 儲存體** | <ul><li>[針對待用資料使用 Azure 儲存體服務加密 (SSE) (預覽)](#sse-preview)</li><li>[使用用戶端加密在 Azure 儲存體中儲存敏感性資料](#client-storage)</li></ul> | 
| **行動用戶端** | <ul><li>[將寫入電話本機儲存體的敏感性資料或 PII 資料加密](#pii-phones)</li><li>[先對進行模糊處理再散發給使用者](#binaries-end)</li></ul> | 
| **WCF** | <ul><li>[將 clientCredentialType 設定為憑證或 Windows](#cert)</li><li>[未啟用 WCF 安全性模式](#security)</li></ul> | 

## <a id="binaries-info"></a>確定包含敏感性資訊的二進位檔已經過模糊處理

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 電腦信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確定包含敏感性資訊的二進位檔已經過模糊處理，例如不應反轉的商業機密、敏感性商業邏輯。 這是為了阻止組件遭到進行反向工程。 `CryptoObfuscator` 之類的工具可用於此用途。 |

## <a id="efs-user"></a>考慮使用加密檔案系統 (EFS) 來保護機密的使用者特定資料

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 電腦信任邊界 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 考慮使用加密檔案系統 (EFS) 來保護機密的使用者特定資料，避免遭到可實際存取電腦的攻擊者竊取。 |

## <a id="filesystem"></a>確定應用程式儲存在檔案系統上的敏感性資料已加密

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 電腦信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確定在無法強制執行 EFS 的狀況下，應用程式儲存在檔案系統上的敏感性資料已加密 (例如使用 DPAPI) |

## <a id="cache-browser"></a>確定瀏覽器上不會快取敏感性內容

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、Web Form、MVC5、MVC6 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 瀏覽器可以儲存資訊以便進行快取和記錄歷程。 這些快取的檔案會儲存在資料夾中，例如若是 Internet Explorer，則會儲存在 Temporary Internet Files 資料夾內。 當您再次參閱這些頁面時，瀏覽器就會顯示快取內的這些頁面。 如果將敏感性資訊顯示給使用者 (例如其地址、信用卡詳細資料、社會安全號碼或使用者名稱)，那麼這項資訊會儲存在瀏覽器的快取內，且因此可透過檢查瀏覽器快取或直接按瀏覽器的 [上一頁] 按鈕來加以擷取。 請將所有頁面的 cache-control 回應標頭值設定為「no-store」。 |

### <a name="example"></a>範例
```XML
<configuration>
  <system.webServer>
   <httpProtocol>
    <customHeaders>
        <add name="Cache-Control" value="no-cache" />
        <add name="Pragma" value="no-cache" />
        <add name="Expires" value="-1" />
    </customHeaders>
  </httpProtocol>
 </system.webServer>
</configuration>
```

### <a name="example"></a>範例
這可透過篩選來實作。 可使用的範例如下︰ 
```csharp
public override void OnActionExecuting(ActionExecutingContext filterContext)
        {
            if (filterContext == null || (filterContext.HttpContext != null && filterContext.HttpContext.Response != null && filterContext.HttpContext.Response.IsRequestBeingRedirected))
            {
                //// Since this is MVC pipeline, this should never be null.
                return;
            }

            var attributes = filterContext.ActionDescriptor.GetCustomAttributes(typeof(System.Web.Mvc.OutputCacheAttribute), false);
            if (attributes == null || **Attributes**.Count() == 0)
            {
                filterContext.HttpContext.Response.Cache.SetNoStore();
                filterContext.HttpContext.Response.Cache.SetCacheability(HttpCacheability.NoCache);
                filterContext.HttpContext.Response.Cache.SetExpires(DateTime.UtcNow.AddHours(-1));
                if (!filterContext.IsChildAction)
                {
                    filterContext.HttpContext.Response.AppendHeader("Pragma", "no-cache");
                }
            }

            base.OnActionExecuting(filterContext);
        }
``` 

## <a id="encrypt-data"></a>加密包含敏感性資料的 Web 應用程式組態檔區段

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [作法：在 ASP.NET 2.0 中使用 DPAPI 加密組態區段](https://msdn.microsoft.com/library/ff647398.aspx)、[指定受保護的組態提供者](https://msdn.microsoft.com/library/68ze1hb2.aspx)、[使用 Azure Key Vault 來保護應用程式的機密資訊](https://azure.microsoft.com/documentation/articles/guidance-multitenant-identity-keyvault/) |
| **步驟** | 組態檔 (例如 Web.config、appsettings.json) 常會用來敏感性資訊，包括使用者名稱、密碼、資料庫連接字串和加密金鑰。 如果您不保護這項資訊，您的應用程式會很容易受到攻擊者或惡意使用者攻擊，而被其取得敏感性資訊，例如帳戶使用者名稱和密碼、資料庫名稱和伺服器名稱。 請根據部署類型 (Azure/內部部署)，使用 DPAPI 或服務 (例如 Azure Key Vault) 加密組態檔的敏感性區段。 |

## <a id="autocomplete-input"></a>明確停用敏感性表單和輸入內的自動完成 HTML 屬性

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [MSDN︰自動完成屬性](http://msdn.microsoft.com/library/ms533486(VS.85).aspx)、[在 HTML 中使用自動完成](http://msdn.microsoft.com/library/ms533032.aspx)[HTML 清理弱點](http://technet.microsoft.com/security/bulletin/MS10-071)、[再次自動完成？！](http://blog.mindedsecurity.com/2011/10/autocompleteagain.html) |
| **步驟** | 自動完成屬性會指定表單應開啟或關閉自動完成。 當自動完成開啟時，瀏覽器會根據使用者之前輸入的值自動填妥值。 例如，在表單中輸入新名稱和密碼並提交表單後，瀏覽器會詢問是否應儲存密碼。之後在顯示表單時，就會自動填入名稱和密碼或在輸入名稱時填妥。 具有本機存取權的攻擊者可從瀏覽器快取取得純文字密碼。 預設會啟用自動完成，但您必須明確地加以停用。 |

### <a name="example"></a>範例
```csharp
<form action="Login.aspx" method="post " autocomplete="off" >
      Social Security Number: <input type="text" name="ssn" />
      <input type="submit" value="Submit" />    
</form>
```

## <a id="data-mask"></a>確定使用者畫面上顯示的敏感性資料已遮罩

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 當敏感性資料 (例如密碼、信用卡號碼 SSN 等) 顯示在螢幕上時，應為其加上遮罩。 這是為了避免未經授權的人員存取資料 (例如背後偷窺密碼、支援人員看到使用者的 SSN 號碼)。 請確定這些資料元素不會以純文字顯示，並會適當地加上遮罩。 這於下列情況必須謹慎處理：接受這些內容為輸入時 (例如， input type="password")，以及在螢幕上顯示時 (例如，只顯示信用卡號碼最後 4 位數)。 |

## <a id="dynamic-users"></a>實作動態資料遮罩來限制敏感性資料暴露非特殊權限使用者

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | Sql Azure、OnPrem |
| **屬性**              | SQL 版本 - V12、SQL 版本 - MsSQL2016 |
| **參考**              | [動態資料遮罩](https://msdn.microsoft.com/library/mt130841) |
| **步驟** | 動態資料遮罩的目的是限制公開敏感性資料，防止不該存取資料的使用者檢視到資料。 動態資料遮罩並非為了避免資料庫使用者直接連線到資料庫並執行可公開敏感性資料片段的詳盡查詢。 動態資料遮罩是為了補充其他 SQL Server 安全性功能 (稽核、加密、資料列層級安全性...)，因此強烈建議您搭配使用這項功能以便加強保護資料庫中的敏感性資料。 請注意，SQL Server 2016 (含) 以上版本和 Azure SQL Database 才支援此功能。 |

## <a id="salted-hash"></a>確定密碼是以 salted 雜湊格式儲存

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [使用 .NET Crypto API 的密碼雜湊](http://docs.asp.net/en/latest/security/data-protection/consumer-apis/password-hashing.html) |
| **步驟** | 密碼不應該儲存在自訂的使用者存放區資料庫中。 應改為使用 salt 值來儲存密碼雜湊。 請確定使用者的 salt 永遠是唯一的，並且先套用 bcrypt、scrypt 或 PBKDF2 再儲存密碼，最少以 150,000 個迴圈的工作因素反覆運算計數來排除暴力密碼破解的可能。| 

## <a id="db-encrypted"></a>確定資料庫資料行中的敏感性資料已加密

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | SQL 版本 - 全部 |
| **參考**              | [加密 SQL Server 中的敏感性資料](https://technet.microsoft.com/library/ff848751(v=sql.105).aspx)、[加密 SQL Server 中的一欄資料](https://msdn.microsoft.com/library/ms179331)、[以憑證加密](https://msdn.microsoft.com/library/ms188061) |
| **步驟** | 資料庫中的敏感性資料 (例如信用卡號碼) 必須加密。 您可以使用資料行層級加密或透過使用加密功能的應用程式函式來加密資料。 |

## <a id="tde-enabled"></a>確定已啟用資料庫層級加密 (TDE)

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [了解 SQL Server 的透明資料加密 (TDE)](https://technet.microsoft.com/library/bb934049(v=sql.105).aspx) |
| **步驟** | SQL Server 的透明資料加密 (TDE) 功能有助於加密資料庫中的敏感性資料，並保護用來以憑證加密資料的金鑰。 這可防止沒有金鑰的人使用資料。 TDE 可以保護「待用」資料，亦即資料和記錄檔。 它提供遵守各種產業中所確立的眾多法律、規定及指導方針的功能。 |

## <a id="backup"></a>確定資料庫備份已加密

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | SQL Azure、OnPrem |
| **屬性**              | SQL 版本 - V12、SQL 版本 - MsSQL2014 |
| **參考**              | [SQL Database 備份加密](https://msdn.microsoft.com/library/dn449489) |
| **步驟** | SQL Server 可在建立備份時加密資料。 藉由在建立備份時指定加密演算法和加密程式 (憑證或非對稱金鑰)，使用者可建立加密的備份檔案。 |

## <a id="api-browser"></a>確定與 Web API 相關的敏感性資料未儲存在瀏覽器的儲存體中

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC 5、MVC 6 |
| **屬性**              | 識別提供者 - ADFS、識別提供者 - Azure AD |
| **參考**              | N/A  |
| **步驟** | <p>在某些實作中，與 Web API 驗證相關的敏感性構件會儲存在瀏覽器的本機儲存體內。 例如，Azure AD 驗證構件，像是 adal.idtoken、adal.nonce.idtoken、adal.access.token.key、adal.token.keys、adal.state.login、adal.session.state、adal.expiration.key 等。</p><p>即使在登出或瀏覽器關閉之後，仍可使用這些構件。 如果攻擊者取得這些構件的存取權，便可重複使用它們來存取受保護的 (API)。 確定與 Web API 相關的所有敏感性構件未儲存在瀏覽器的儲存體中。 萬一只能儲存在用戶端中 (例如，運用隱含 OpenIdConnect/OAuth 流程的單一頁面應用程式 (SPA) 需要在本機儲存存取權杖)，使用的儲存選項沒有持續性。 例如，使用 SessionStorage 勝過 LocalStorage。</p>| 

### <a name="example"></a>範例
以下 JavaScript 程式碼片段來自將驗證構件儲存在本機儲存體的自訂驗證程式庫。 請避免這類實作。 
```javascript
ns.AuthHelper.Authenticate = function () {
window.config = {
instance: 'https://login.microsoftonline.com/',
tenant: ns.Configurations.Tenant,
clientId: ns.Configurations.AADApplicationClientID,
postLogoutRedirectUri: window.location.origin,
cacheLocation: 'localStorage', // enable this for IE, as sessionStorage does not work for localhost.
};
```

## <a id="encrypt-docdb"></a>將儲存在 Cosmos DB 中的敏感性資料加密

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure Document DB | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 請先在應用程式層級加密敏感性資料再儲存於 DocumentDB，或將敏感性資料儲存在其他儲存體解決方案，例如 Azure 儲存體或 Azure SQL| 

## <a id="disk-vm"></a>使用 Azure 磁碟加密來加密虛擬機器所使用的磁碟

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure IaaS VM 信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [使用 Azure 磁碟加密來加密虛擬機器所使用的磁碟](https://azure.microsoft.com/documentation/articles/storage-security-guide/#_using-azure-disk-encryption-to-encrypt-disks-used-by-your-virtual-machines) |
| **步驟** | <p>Azure 磁碟加密是目前預覽版中的新功能。 此功能允許您加密 IaaS 虛擬機器所使用的作業系統磁碟和資料磁碟。 對於 Windows，磁碟機是使用業界標準的 BitLocker 加密技術來加密。 對於 Linux，磁碟是使用 DM-Crypt 技術來加密。 這會與 Azure 金鑰保存庫整合，可讓您控制和管理磁碟加密金鑰。 Azure 磁碟加密解決方案支援下列三個客戶加密案例：</p><ul><li>在透過客戶加密的 VHD 檔案和客戶提供的加密金鑰 (儲存於 Azure 金鑰保存庫中) 建立的新 IaaS VM 上啟用加密。</li><li>在透過 Azure Marketplace 建立的新 IaaS VM 上啟用加密。</li><li>在 Azure 中已執行的現有 IaaS VM 上啟用加密。</li></ul>| 

## <a id="fabric-apps"></a>將 Service Fabric 應用程式中的密碼加密

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Service Fabric 信任邊界 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | 環境 - Azure |
| **參考**              | [管理 Service Fabric 應用程式中的密碼](https://azure.microsoft.com/documentation/articles/service-fabric-application-secret-management/) |
| **步驟** | 密碼可以是任何機密資訊，例如儲存體連接字串、密碼或其他不會以純文字處理的值。 使用 Azure Key Vault 來管理 Service Fabric 應用程式中的金鑰和密碼。 |

## <a id="modeling-teams"></a>執行安全性模型化並視需要使用業務單位/團隊

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Dynamics CRM | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 執行安全性模型化並視需要使用業務單位/團隊 |

## <a id="entities"></a>將共用重要實體功能的存取權最小化

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Dynamics CRM | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 將共用重要實體功能的存取權最小化 |

## <a id="good-practices"></a>訓練使用者以讓其了解與 Dynamics CRM 共用功能相關聯的風險以及正確的安全性作法

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Dynamics CRM | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 訓練使用者以讓其了解與 Dynamics CRM 共用功能相關聯的風險以及正確的安全性作法 |

## <a id="exception-mgmt"></a>包含開發標準規則來禁止在例外狀況管理中顯示組態詳細資料

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Dynamics CRM | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 包含開發標準規則來禁止在開發外的例外狀況管理中顯示組態詳細資料。 請在檢閱程式碼或定期檢查時對此進行測試。|

## <a id="sse-preview"></a>針對待用資料使用 Azure 儲存體服務加密 (SSE) (預覽)

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 儲存體 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | StorageType - Blob |
| **參考**              | [待用資料的 Azure 儲存體服務加密 (預覽)](https://azure.microsoft.com/documentation/articles/storage-service-encryption/) |
| **步驟** | <p>待用資料的 Azure 儲存體服務加密 (SSE) 會協助您保護資料安全，以符合組織安全性和法規遵循承諾。 利用此功能，Azure 儲存體會自動加密資料，再保存到儲存體，以及在擷取之前解密。 以完全無感的方式處理所有加密、解密和金鑰管理。 SSE 只適用於區塊 Blob、分頁 Blob 及附加 Blob。 其他類型的資料 (包括表格、佇列和檔案) 將不會加密。</p><p>加密和解密工作流程：</p><ul><li>客戶會在儲存體帳戶上啟用加密</li><li>當客戶將新資料 (放置 Blob、放置區塊、放置頁面等等) 寫入至 Blob 儲存體，每個寫入是使用 256 位元 AES 加密 (可用的最強區塊加密方式之一) 進行加密</li><li>當客戶需要存取資料 (取得 Blob 等) 時，資料會在傳回給使用者之前自動解密</li><li>如果已停用加密，就不會再加密新的寫入，現有加密資料在使用者重新寫入之前會保持加密。 啟用加密時，寫入至 Blob 儲存體將會加密。 資料的狀態在使用者於啟用/停用儲存體帳戶的加密之間切換時不會變更</li><li>所有加密金鑰會由 Microsoft 儲存、加密及管理</li></ul><p>請注意，在此階段中，用來加密的金鑰是由 Microsoft 所管理。 Microsoft 一開始會產生金鑰，並管理金鑰的安全儲存體，以及如同內部 Microsoft 原則所定義的定期輪換。 客戶未來將會獲得管理自有加密金鑰的功能，並提供從 Microsoft 管理的金鑰到客戶管理的金鑰的移轉路徑。</p>| 

## <a id="client-storage"></a>使用用戶端加密在 Azure 儲存體中儲存敏感性資料

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 儲存體 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [Microsoft Azure 儲存體的用戶端加密和 Azure Key Vault 金鑰保存庫](https://azure.microsoft.com/documentation/articles/storage-client-side-encryption/)、[教學課程：在 Microsoft Azure 儲存體中使用 Azure 金鑰保存庫加密和解密 Blob](https://azure.microsoft.com/documentation/articles/storage-encrypt-decrypt-blobs-key-vault/)、[在 Azure Blob 儲存體中使用 Azure 加密擴充功能安全地存放資料](https://blogs.msdn.microsoft.com/partnercatalystteam/2015/06/17/storing-data-securely-in-azure-blob-storage-with-azure-encryption-extensions/) |
| **步驟** | <p>適用於 .NET NuGet 套件的 Azure 儲存體用戶端程式庫支援在上傳至 Azure 儲存體之前將用戶端應用程式內的資料加密，並在下載至用戶端時解密資料。 程式庫也支援與 Azure 金鑰保存庫整合，以進行儲存體帳戶金鑰管理。 以下是用戶端加密運作方式的簡短描述：</p><ul><li>Azure 儲存體用戶端 SDK 會產生內容加密金鑰 (CEK)，這是使用一次的對稱金鑰</li><li>客戶資料是使用此 CEK 加密</li><li>然後使用金鑰加密金鑰 (KEK) 包裝 (加密) CEK。 KEK 由金鑰識別碼所識別，可以是非對稱金鑰組或對稱金鑰，且可以在本機管理或儲存在 Azure 金鑰保存庫中。 儲存體用戶端本身永遠沒有 KEK 的存取權。 它只是叫用金鑰保存庫所提供的金鑰包裝演算法。 如有需要，客戶可以選擇使用自訂提供者來包裝/取消包裝金鑰</li><li>然後，將加密的資料上傳至 Azure 儲存體服務。 請參閱 [參考] 區段中的連結，以取得低階實作詳細資料。</li></ul>|

## <a id="pii-phones"></a>將寫入電話本機儲存體的敏感性資料或 PII 資料加密

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 行動用戶端 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、Xamarin  |
| **屬性**              | N/A  |
| **參考**              | [使用 Microsoft Intune 原則管理裝置上的設定和功能](https://docs.microsoft.com/intune/deploy-use/manage-settings-and-features-on-your-devices-with-microsoft-intune-policies#create-a-configuration-policy)、[金鑰鏈貼身](https://components.xamarin.com/view/square.valet) |
| **步驟** | <p>如果應用程式在行動裝置的檔案系統上寫入敏感性資訊，例如使用者的 PII (電子郵件、電話號碼、名字、姓氏、喜好設定等)，則應先將它加密再寫入本機檔案系統。 如果應用程式是企業應用程式，則請探究使用 Windows Intune 發佈應用程式的可能性。</p>|

### <a name="example"></a>範例
Intune 可以設定下列安全性原則來保護敏感性資料︰ 
```csharp
Require encryption on mobile device    
Require encryption on storage cards
Allow screen capture
```

### <a name="example"></a>範例
如果應用程式不是企業應用程式，則使用平台提供的金鑰存放區、金鑰鏈來存放加密金鑰，使用可能會在檔案系統上執行的密碼編譯作業。 下列程式碼片段示範如何使用 xamarin 從金鑰鏈存取金鑰︰ 
```csharp
        protected static string EncryptionKey
        {
            get
            {
                if (String.IsNullOrEmpty(_Key))
                {
                    var query = new SecRecord(SecKind.GenericPassword);
                    query.Service = NSBundle.MainBundle.BundleIdentifier;
                    query.Account = "UniqueID";

                    NSData uniqueId = SecKeyChain.QueryAsData(query);
                    if (uniqueId == null)
                    {
                        query.ValueData = NSData.FromString(System.Guid.NewGuid().ToString());
                        var err = SecKeyChain.Add(query);
                        _Key = query.ValueData.ToString();
                    }
                    else
                    {
                        _Key = uniqueId.ToString();
                    }
                }

                return _Key;
            }
        }
```

## <a id="binaries-end"></a>先對進行模糊處理再散發給使用者

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 行動用戶端 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [適用於 .Net 的 CryptoObfuscator](http://www.ssware.com/cryptoobfuscator/obfuscator-net.htm) |
| **步驟** | 產生的二進位檔 (apk 內的組件) 應該進行模糊處理，以阻止組件遭到進行反向工程。`CryptoObfuscator` 之類的工具可用於此用途。 |

## <a id="cert"></a>將 clientCredentialType 設定為憑證或 Windows

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | .NET Framework 3 |
| **屬性**              | N/A  |
| **參考**              | [Fortify](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_weak_token) |
| **步驟** | 在未加密的通道上搭配使用 UsernameToken 與純文字密碼會將密碼洩漏給攻擊者，而讓其竊取到 SOAP 訊息。 使用 UsernameToken 的服務提供者可能會接受以純文字傳送的密碼。 透過未加密通道傳送純文字密碼，會將認證洩漏給攻擊者，而讓其竊取到 SOAP 訊息。 | 

### <a name="example"></a>範例
下列 WCF 服務提供者組態會使用 UsernameToken： 
```
<security mode="Message"> 
<message clientCredentialType="UserName" />
``` 
將 clientCredentialType 設定為憑證或 Windows。 

## <a id="security"></a>未啟用 WCF 安全性模式

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、.NET Framework 3 |
| **屬性**              | 安全性模式 - 傳輸、安全性模式 - 訊息 |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify Kingdom](https://vulncat.hpefod.com/en/detail?id=desc.semantic.dotnet.wcf_misconfiguration_security_not_enabled)、[WCF 安全性基礎概念 CoDe Magazine](http://www.codemag.com/article/0611051) |
| **步驟** | 尚未定義任何傳輸或訊息安全性。 傳輸訊息的應用程式若沒有傳輸或訊息安全性，就無法保證訊息的完整性或機密性。 當 WCF 安全性繫結設定為 [無] 時，會同時停用傳輸和訊息安全性。 |

### <a name="example"></a>範例
下列組態將安全性模式設定為 [無]。 
```
<system.serviceModel> 
  <bindings> 
    <wsHttpBinding> 
      <binding name=""MyBinding""> 
        <security mode=""None""/> 
      </binding> 
  </bindings> 
</system.serviceModel> 
```

### <a name="example"></a>範例
所有服務繫結的安全性模式有五個可能選項︰ 
* 無。 關閉安全性。 
* 傳輸。 使用傳輸安全性來互相保護驗證和訊息。 
* Message. 使用訊息安全性來互相保護驗證和訊息。 
* 兩者。 可讓您提供傳輸和訊息層級安全性的設定 (只有 MSMQ 支援此模式)。 
* TransportWithMessageCredential。 認證會隨訊息傳遞，而且傳輸層會提供訊息保護和伺服器驗證。 
* TransportCredentialOnly。 用戶端認證會使用傳輸層傳遞，但未套用任何訊息保護。 使用傳輸和訊息安全性來保護訊息的完整性和機密性。 下列組態會告知服務搭配使用傳輸安全性和訊息認證。
```
<system.serviceModel>
  <bindings>
    <wsHttpBinding>
    <binding name=""MyBinding""> 
    <security mode=""TransportWithMessageCredential""/> 
    <message clientCredentialType=""Windows""/> 
    </binding> 
  </bindings> 
</system.serviceModel> 
```
