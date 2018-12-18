---
title: 稽核和記錄 - Microsoft Threat Modeling Tool - Azure | Microsoft Docs
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
ms.openlocfilehash: 990f300055f7c0c7132dd44271dea73044649fc5
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306993"
---
# <a name="security-frame-auditing-and-logging--mitigations"></a>安全框架︰稽核和記錄 | 緩和措施 
| 產品/服務 | 文章 |
| --------------- | ------- |
| **Dynamics CRM**    | <ul><li>[在您的解決方案中識別敏感性實體及實作變更稽核](#sensitive-entities)</li></ul> |
| **Web 應用程式** | <ul><li>[確保在應用程式上強制執行稽核和記錄功能](#auditing)</li><li>[確保施行記錄輪替和區隔功能](#log-rotation)</li><li>[確保應用程式不會記錄敏感性使用者資料](#log-sensitive-data)</li><li>[確保稽核和記錄檔有限制存取](#log-restricted-access)</li><li>[確保已記錄使用者管理事件](#user-management)</li><li>[確保系統已內建誤用防禦措施](#inbuilt-defenses)</li><li>[在 Azure App Service 中針對 Web 應用程式啟用診斷記錄功能](#diagnostics-logging)</li></ul> |
| **資料庫** | <ul><li>[確保已在 SQL Server 上啟用登入稽核功能](#identify-sensitive-entities)</li><li>[在 Azure SQL 上啟用威脅偵測功能](#threat-detection)</li></ul> |
| **Azure 儲存體** | <ul><li>[使用 Azure 儲存體分析來稽核 Azure 儲存體的存取](#analytics)</li></ul> |
| **WCF** | <ul><li>[實作足夠的記錄](#sufficient-logging)</li><li>[實作足夠的稽核失敗處理](#audit-failure-handling)</li></ul> |
| **Web API** | <ul><li>[確保在 Web API 上強制執行稽核和記錄功能](#logging-web-api)</li></ul> |
| **IoT 現場閘道** | <ul><li>[確保在現場閘道上強制執行適當的稽核和記錄功能](#logging-field-gateway)</li></ul> |
| **IoT 雲端閘道** | <ul><li>[確保在雲端閘道上強制執行適當的稽核和記錄功能](#logging-cloud-gateway)</li></ul> |

## <a id="sensitive-entities"></a>在您的解決方案中識別敏感性實體及實作變更稽核

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Dynamics CRM | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟**                   | 識別您的解決方案中包含敏感性資料的實體，以及在這些實體和現場實作變更稽核 |

## <a id="auditing"></a>確保在應用程式上強制執行稽核和記錄

| Title                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟**                   | 在所有元件上啟用稽核和記錄。 稽核記錄檔應擷取使用者內容。 識別所有重要的事件及記錄這些事件。 實作集中式記錄 |

## <a id="log-rotation"></a>確保施行記錄輪替和區隔功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟**                   | <p>記錄輪替是一個自動化程序，使用於封存過時記錄檔的系統管理作業中。 執行大型應用程式的伺服器通常會記錄每一個要求︰面對大量記錄檔，記錄檔輪替可以限制記錄檔大小總計，同時還可進行最新事件的分析。 </p><p>記錄檔區隔基本上表示您必須將記錄檔儲存在不同於 OS/應用程式執行所在的磁碟分割上，以避開拒絕服務攻擊或降低應用程式的效能</p>|

## <a id="log-sensitive-data"></a>確保應用程式不會記錄敏感性使用者資料

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟**                   | <p>檢查您不會記錄使用者提交至您網站的任何敏感性資料。 檢查刻意的記錄，以及設計問題所造成的副作用。 敏感性資料的範例包括︰</p><ul><li>使用者認證</li><li>社會安全號碼或其他識別資訊</li><li>信用卡號碼或其他財務資訊</li><li>健康狀態資訊</li><li>可用來將已加密資訊解密的私密金鑰或其他資料</li><li>可用來更有效地攻擊應用程式的系統或應用程式資訊</li></ul>|

## <a id="log-restricted-access"></a>確保稽核和記錄檔有限制存取

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟**                   | <p>檢查以確保適當地設定記錄檔的存取權限。 應用程式帳戶應具有唯寫權限，而操作人員和支援人員應具有所需的唯讀權限。</p><p>系統管理員帳戶是唯一應具有完整存取權限的帳戶。 檢查記錄檔上的 Windows ACL，以確保它們受到適當限制︰</p><ul><li>應用程式帳戶應具有唯寫權限</li><li>操作人員和支援人員應具有所需的唯讀存取權限</li><li>系統管理員是唯一應具有完整存取權限的帳戶</li></ul>|

## <a id="user-management"></a>確保已記錄使用者管理事件

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟**                   | <p>確保應用程式會監視使用者管理事件，例如：成功和失敗的使用者登入、密碼重設、密碼變更、帳戶鎖定、使用者註冊。 此舉有助於偵測及回應可疑的行為。 也能夠蒐集作業資料；例如，追蹤正在存取應用程式的人員</p>|

## <a id="inbuilt-defenses"></a>確保系統已內建誤用防禦措施

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟**                   | <p>應備妥控制項，以便在發生應用程式誤用時擲回安全性例外狀況。 例如，如果已施行輸入驗證，而且攻擊者嘗試插入不符合 regex 的惡意程式碼，則擲回的安全性例外狀況可能暗示系統誤用。</p><p>例如，建議您針對下列問題記錄安全性例外狀況並採取相關行動︰</p><ul><li>輸入驗證</li><li>CSRF 違規</li><li>暴力密碼破解 (每項資源之每個使用者的要求數目上限)</li><li>檔案上傳違規</li><ul>|

## <a id="diagnostics-logging"></a>在 Azure App Service 中針對 Web 應用程式啟用診斷記錄功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | EnvironmentType - Azure |
| **參考**              | N/A  |
| **步驟** | <p>Azure 提供內建診斷功能，可協助對 App Service Web 應用程式進行偵錯。 它也適用於 API 應用程式與行動應用程式。 App Service Web 應用程式會針對來自 Web 伺服器和 Web 應用程式的記錄資訊提供診斷功能。</p><p>這些資訊邏輯上可區分為 Web 伺服器診斷和應用程式診斷</p>|

## <a id="identify-sensitive-entities"></a>確保已在 SQL Server 上啟用登入稽核功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [設定登入稽核](https://msdn.microsoft.com/library/ms175850.aspx) |
| **步驟** | <p>必須啟用資料庫伺服器登入稽核功能，才能偵測/確認密碼猜測攻擊。 請務必擷取失敗的登入嘗試。 擷取成功和失敗的登入嘗試，可在鑑識調查期間提供額外的好處</p>|

## <a id="threat-detection"></a>在 Azure SQL 上啟用威脅偵測功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | SQL Azure |
| **屬性**              | SQL 版本 - V12 |
| **參考**              | [開始使用 SQL Database 威脅偵測](https://azure.microsoft.com/documentation/articles/sql-database-threat-detection-get-started/)|
| **步驟** |<p>威脅偵測會偵測異常資料庫活動，指出資料庫有潛在的安全性威脅。 其提供新的一層安全性，在發生異常活動時會提供安全性警示，讓客戶偵測並回應潛在威脅。</p><p>使用者可以使用 Azure SQL Database 稽核來探查可疑的事件，判斷是否有人嘗試存取、破壞或利用資料庫中的資料。</p><p>您不必是安全性專家，也不需要管理進階的安全性監視系統，威脅偵測讓您輕鬆解決資料庫的潛在威脅</p>|

## <a id="analytics"></a>使用 Azure 儲存體分析來稽核 Azure 儲存體的存取

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 儲存體 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A |
| **參考**              | [使用儲存體分析來監視授權類型](https://azure.microsoft.com/documentation/articles/storage-security-guide/#storage-analytics) |
| **步驟** | <p>對於每個儲存體帳戶，您可以啟用 Azure 儲存體分析，來執行記錄和儲存計量資料。 儲存體分析記錄檔可提供重要資訊，例如其他人存取儲存體時所使用的驗證方法。</p><p>如果您要嚴密監視儲存體的存取，這真的很實用。 例如，在 Blob 儲存體中，您可以將所有容器設定為私人，並透過您的應用程式實作 SAS 服務的用法。 然後您可以定期檢查記錄，以查看是否會使用儲存體帳戶金鑰存取您的 Blob (這可能表示安全性缺口)，或者 Blob 是公用的但它們不應該是公用的。</p>|

## <a id="sufficient-logging"></a>實作足夠的記錄

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | .NET Framework |
| **屬性**              | N/A  |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_insufficient_logging) |
| **步驟** | <p>在安全性事件之後，若缺少適當的稽核線索，可能會拖累鑑識努力。 Windows Communication Foundation (WCF) 可讓您記錄成功和/或失敗的驗證嘗試。</p><p>記錄失敗的驗證嘗試可以警告系統管理員可能的暴力破解攻擊。 同樣地，記錄成功的驗證事件可以在合法帳戶遭到入侵時提供有用的稽核線索。 啟用 WCF 的服務安全性稽核功能 |

### <a name="example"></a>範例
下列是已啟用稽核功能的範例組態︰
```
<system.serviceModel>
    <behaviors>
        <serviceBehaviors>
            <behavior name=""NewBehavior"">
                <serviceSecurityAudit auditLogLocation=""Default""
                suppressAuditFailure=""false"" 
                serviceAuthorizationAuditLevel=""SuccessAndFailure""
                messageAuthenticationAuditLevel=""SuccessAndFailure"" />
                ...
            </behavior>
        </servicebehaviors>
    </behaviors>
</system.serviceModel>
```

## <a id="audit-failure-handling"></a>實作足夠的稽核失敗處理

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | .NET Framework |
| **屬性**              | N/A  |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_insufficient_audit_failure_handling) |
| **步驟** | <p>所開發的解決方案已設定成：不會在無法寫入稽核記錄檔時產生例外狀況。 如果 WCF 已設定成：不會在無法寫入稽核記錄檔時擲回例外狀況，程式就不會接獲失敗通知，而可能不會發生重大安全性事件的稽核。</p>|

### <a name="example"></a>範例
以下 WCF 組態檔的 `<behavior/>` 元素會指示 WCF 不要在 WCF 無法寫入稽核記錄檔時通知應用程式。
````
<behaviors>
    <serviceBehaviors>
        <behavior name="NewBehavior">
            <serviceSecurityAudit auditLogLocation="Application"
            suppressAuditFailure="true"
            serviceAuthorizationAuditLevel="Success"
            messageAuthenticationAuditLevel="Success" />
        </behavior>
    </serviceBehaviors>
</behaviors>
````
將 WCF 設定成在無法寫入稽核記錄檔時通知程式。 程式應備妥替代的通知方案，以警示組織並未維護稽線索。 

## <a id="logging-web-api"></a>確保在 Web API 上強制執行稽核和記錄功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 在 Web API 上啟用稽核和記錄功能。 稽核記錄檔應擷取使用者內容。 識別所有重要的事件及記錄這些事件。 實作集中式記錄 |

## <a id="logging-field-gateway"></a>確保在現場閘道上強制執行適當的稽核和記錄功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 現場閘道 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>當多個裝置連線至現場閘道時，確保在現場閘道上記錄及維護個別裝置的連線嘗試和驗證狀態 (成功或失敗)。</p><p>此外，在現場閘道維護個別裝置的 IoT 中樞認證的情況下，請確保在擷取這些認證時執行稽核。開發相關程序，以便將記錄檔定期上傳至 Azure IoT 中樞/儲存體進行長期保留。</p> |

## <a id="logging-cloud-gateway"></a>確保在雲端閘道上強制執行適當的稽核和記錄功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 雲端閘道 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [IoT 中樞作業監視簡介](https://azure.microsoft.com/documentation/articles/iot-hub-operations-monitoring/) |
| **步驟** | <p>設計用來收集及儲存透過 IoT 中樞作業監視所蒐集的稽核資料。 啟用下列監視類別：</p><ul><li>裝置身分識別作業</li><li>裝置到雲端通訊</li><li>雲端到裝置通訊</li><li>連線</li><li>檔案上傳</li></ul>|