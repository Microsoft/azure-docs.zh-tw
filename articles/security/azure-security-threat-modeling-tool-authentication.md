---
title: 驗證 - Microsoft Threat Modeling Tool - Azure | Microsoft Docs
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
ms.openlocfilehash: 23e219fa49146158c97f392427eee7c42c347a7a
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307415"
---
# <a name="security-frame-authentication--mitigations"></a>安全框架︰驗證 | 緩和措施 
| 產品/服務 | 文章 |
| --------------- | ------- |
| **Web 應用程式**    | <ul><li>[考慮使用標準驗證機制來驗證 Web 應用程式](#standard-authn-web-app)</li><li>[應用程式必須安全地處理失敗的驗證案例](#handle-failed-authn)</li><li>[啟用升級或調適性驗證](#step-up-adaptive-authn)</li><li>[確保已適當地鎖定系統管理介面](#admin-interface-lockdown)</li><li>[安全地實作忘記密碼功能](#forgot-pword-fxn)</li><li>[確保已實作密碼和帳戶原則](#pword-account-policy)</li><li>[實作控制項以避免列舉使用者名稱](#controls-username-enum)</li></ul> |
| **資料庫** | <ul><li>[可能的話，使用 Windows 驗證連線到 SQL Server](#win-authn-sql)</li><li>[可能的話，使用 Azure Active Directory 驗證連線到 SQL Database](#aad-authn-sql)</li><li>[使用 SQL 驗證模式時，確保在 SQL Server 上強制執行帳戶和密碼原則](#authn-account-pword)</li><li>[請勿在自主資料庫中使用 SQL 驗證](#autn-contained-db)</li></ul> |
| **Azure 事件中樞** | <ul><li>[使用採用 SaS 權杖的每一裝置驗證認證](#authn-sas-tokens)</li></ul> |
| **Azure 信任邊界** | <ul><li>[啟用 Azure 系統管理員適用的 Azure Multi-Factor Authentication](#multi-factor-azure-admin)</li></ul> |
| **Service Fabric 信任邊界** | <ul><li>[限制 Service Fabric 叢集的匿名存取](#anon-access-cluster)</li><li>[確保 Service Fabric 的用戶端對節點憑證不同於節點對節點憑證](#fabric-cn-nn)</li><li>[使用 AAD 來向 Service Fabric 叢集驗證用戶端對](#aad-client-fabric)</li><li>[確保從經過核准的憑證授權單位 (CA) 取得 Service Fabric 憑證](#fabric-cert-ca)</li></ul> |
| **Identity Server** | <ul><li>[使用 Identity Server 所支援的標準驗證案例](#standard-authn-id)</li><li>[以可調整的替代項目覆寫預設 Identity Server 權杖快取](#override-token)</li></ul> |
| **電腦信任邊界** | <ul><li>[確保已數位簽署所部署應用程式的二進位檔](#binaries-signed)</li></ul> |
| **WCF** | <ul><li>[在連線到 WCF 中的 MSMQ 佇列時啟用驗證](#msmq-queues)</li><li>[WCF - 請勿將訊息 clientCredentialType 設定為 none](#message-none)</li><li>[WCF - 請勿將傳輸 clientCredentialType 設定為 none](#transport-none)</li></ul> |
| **Web API** | <ul><li>[確保使用標準驗證技術來保護 Web API](#authn-secure-api)</li></ul> |
| **Azure AD** | <ul><li>[使用 Azure Active Directory 所支援的標準驗證案例](#authn-aad)</li><li>[以可調整的替代項目覆寫預設 ADAL 權杖快取](#adal-scalable)</li><li>[確保使用 TokenReplayCache 來防止重新執行 ADAL 驗證權杖](#tokenreplaycache-adal)</li><li>[使用 ADAL 程式庫來管理從 OAuth2 用戶端至 AAD (或內部部署 AD) 的權杖要求](#adal-oauth2)</li></ul> |
| **IoT 現場閘道** | <ul><li>[驗證連線到現場閘道的裝置](#authn-devices-field)</li></ul> |
| **IoT 雲端閘道** | <ul><li>[確保會驗證連線到雲端閘道的裝置](#authn-devices-cloud)</li><li>[使用每一裝置驗證認證](#authn-cred)</li></ul> |
| **Azure 儲存體** | <ul><li>[確保只有必要的容器和 blob 會取得匿名讀取權限](#req-containers-anon)</li><li>[使用 SAS 或 SAP 對 Azure 儲存體中的物件授與有限的存取權](#limited-access-sas)</li></ul> |

## <a id="standard-authn-web-app"></a>考慮使用標準驗證機制來驗證 Web 應用程式

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| 詳細資料 | <p>驗證是實體證明其身分識別的程序 (通常透過使用者名稱和密碼等認證)。 有多個驗證通訊協定可列入考量。 下列是其中一些通訊協定︰</p><ul><li>Client certificates</li><li>Windows 架構</li><li>表單架構</li><li>同盟 - ADFS</li><li>同盟 - Azure AD</li><li>同盟 - Identity Server</li></ul><p>考慮使用標準驗證機制來識別來源程序</p>|

## <a id="handle-failed-authn"></a>應用程式必須安全地處理失敗的驗證案例

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| 詳細資料 | <p>明確驗證使用者的應用程式必須安全地處理失敗的驗證案例。驗證機制必須︰</p><ul><li>驗證失敗時拒絕存取特殊權限的資源</li><li>驗證失敗後會顯示一般錯誤訊息，並發生存取遭拒</li></ul><p>測試：</p><ul><li>登入失敗後的特殊權限資源保護</li><li>一般錯誤訊息會顯示於驗證失敗和存取遭拒事件</li><li>帳戶會在過多失敗嘗試後停用</li><ul>|

## <a id="step-up-adaptive-authn"></a>啟用升級或調適性驗證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| 詳細資料 | <p>確認應用程式有額外的授權 (例如透過 Multi-Factor Authentication 的升級或調適性驗證，例如以簡訊、電子郵件等傳送 OTP 或提示重新驗證)，這麼一來，使用者便會在獲得敏感性資訊的存取權之前受到查問。 此規則也適用於對帳戶或動作進行重大變更</p><p>這也表示必須以應用程式正確地強制執行內容相關授權的方式來實作驗證調節，才不會允許藉由竄改參數等方法的未經授權操作</p>|

## <a id="admin-interface-lockdown"></a>確保已適當地鎖定系統管理介面

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| 詳細資料 | 第一個解決方法是授與只從特定來源 IP 範圍到系統管理介面的存取權。 如果相較於一貫的建議，該解決方案並不可行，則強制執行升級或調適性驗證以便登入系統管理介面 |

## <a id="forgot-pword-fxn"></a>安全地實作忘記密碼功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| 詳細資料 | <p>首先，確認忘記密碼和其他復原路徑會傳送包含限時啟用權杖 (而不是密碼本身) 的連結。 在連結傳出之前，也可能需要以軟式權杖 (例如 SMS 權杖、原生行動應用程式等) 為基礎的其他驗證。 第二，在取得新密碼的過程中，您不得鎖定使用者帳戶。</p><p>每當攻擊者決定透過自動化攻擊來刻意鎖定使用者時，這可能會導致拒絕服務攻擊。 第三，在設定新密碼要求的過程中，您顯示的訊息應該一般化，以避免列舉使用者名稱。 第四，一律不允許使用舊密碼並實作強式密碼原則。</p> |

## <a id="pword-account-policy"></a>確保已實作密碼和帳戶原則

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| 詳細資料 | <p>必須實作符合組織原則和最佳作法的密碼和帳戶原則。</p><p>若要防止暴力密碼破解及以字典為基礎的猜測︰必須實作強式密碼原則，以確保使用者建立複雜的密碼 (例如，長度最少 12 個字元、英數字元和特殊字元)。</p><p>帳戶鎖定原則可能會以下列方式實作︰</p><ul><li>**軟式鎖定︰** 這是保護使用者以對抗暴力密碼破解攻擊的理想選項。 例如，當使用者輸入錯誤密碼三次時，應用程式會鎖定帳戶一分鐘，以減緩暴力破解其密碼的進度，讓攻擊者有機可趁的機會變少。 如果您對此範例執行硬式鎖定對策，您可藉由永久鎖定帳戶來達成 "Dos"。 或者，應用程式可產生 OTP (一次性密碼) 並以頻外方式 (透過電子郵件、簡訊等) 將它傳送給使用者。 另一種方法則是在達到嘗試失敗次數閾值之後，實作 CAPTCHA。</li><li>**硬式鎖定︰** 每當您偵測到攻擊應用程式的使用者時，應套用這類型的鎖定，並透過永久鎖定其帳戶來對抗他，直到因應小組有時間進行鑑識為止。 在此程序之後，您可以決定將帳戶還給使用者，或採取進一步的法律動作來對抗他。 這種方法可防止攻擊者進一步入侵您的應用程式和基礎結構。</li></ul><p>若要防禦對於預設和可預測帳戶的攻擊，請確認所有金鑰和密碼均可取代，而且會在安裝階段後產生或取代。</p><p>如果應用程式必須自動產生密碼，請確保所產生的密碼是隨機的且具有高熵。</p>|

## <a id="controls-username-enum"></a>實作控制項以避免列舉使用者名稱

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 應將所有的錯誤訊息一般化，以避免列舉使用者名稱。 而且，您有時無法避免註冊頁面等功能的資訊洩漏。 您需要使用 CAPTCHA 之類的速率限制方法，防止攻擊者的自動化攻擊。 |

## <a id="win-authn-sql"></a>可能的話，使用 Windows 驗證連線到 SQL Server

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | OnPrem |
| **屬性**              | SQL 版本 - 全部 |
| **參考**              | [SQL Server - 選擇驗證模式](https://msdn.microsoft.com/library/ms144284.aspx) |
| **步驟** | Windows 驗證會使用 Kerberos 安全性通訊協定、提供與強式密碼複雜性驗證相關的密碼原則強制執行、提供帳戶鎖定支援，以及支援密碼到期。|

## <a id="aad-authn-sql"></a>可能的話，使用 Azure Active Directory 驗證連線到 SQL Database

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | SQL Azure |
| **屬性**              | SQL 版本 - V12 |
| **參考**              | [使用 Azure Active Directory 驗證連線到 SQL Database](https://azure.microsoft.com/documentation/articles/sql-database-aad-authentication/) |
| **步驟** | **最小版本︰** 允許 Azure SQL Database 對 Microsoft 目錄使用 AAD 驗證所需的 Azure SQL Database V12 |

## <a id="authn-account-pword"></a>使用 SQL 驗證模式時，確保在 SQL Server 上強制執行帳戶和密碼原則

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [SQL Server 密碼原則](https://technet.microsoft.com/library/ms161959(v=sql.110).aspx) |
| **步驟** | 使用 SQL Server 驗證時，系統會在 SQL Server 中建立不是以 Windows 使用者帳戶為基礎的登入。 使用 SQL Server 建立使用者名稱和密碼並儲存在 SQL Server 中。 SQL Server 可以使用 Windows 密碼原則機制。 它可以將 Windows 中使用的相同複雜性和到期原則套用於 SQL Server 內部使用的密碼。 |

## <a id="autn-contained-db"></a>請勿在自主資料庫中使用 SQL 驗證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | Sql Azure、OnPrem |
| **屬性**              | SQL 版本 - MSSQL2012、SQL 版本 - V12 |
| **參考**              | [自主資料庫的安全性最佳作法](http://msdn.microsoft.com/library/ff929055.aspx) |
| **步驟** | 缺少強制執行的密碼原則，可能會增加在自主資料庫中建立弱式認證的可能性。 利用 Windows 驗證。 |

## <a id="authn-sas-tokens"></a>使用採用 SaS 權杖的每一裝置驗證認證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 事件中樞 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [事件中樞驗證和安全性模型概觀](https://azure.microsoft.com/documentation/articles/event-hubs-authentication-and-security-model-overview/) |
| **步驟** | <p>事件中樞安全性模型是以共用存取簽章 (SAS) 權杖和事件發佈者的組合為基礎。 發佈者代表可接收權杖的 DeviceID。 這有助於讓所產生的權杖與個別的裝置建立關聯。</p><p>在允許偵測承載內部原始來源詐騙嘗試的服務端上，所有訊息都會標示建立者。 驗證裝置時，產生範圍限於唯一發佈者的每個裝置 SaS 權杖。</p>|

## <a id="multi-factor-azure-admin"></a>啟用 Azure 系統管理員適用的 Azure Multi-Factor Authentication

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [什麼是 Azure Multi-Factor Authentication？](https://azure.microsoft.com/documentation/articles/multi-factor-authentication/) |
| **步驟** | <p>多因素驗證 (MFA) 是需要多種驗證方法，並在使用者登入和交易中新增重要的第二層安全性的驗證方法。 其運作方式需要下列其中任何二或多個驗證方法：</p><ul><li>您知道的某些資訊 (通常是密碼)</li><li>您擁有的某些東西 (不容易輕易複製的信任裝置，例如電話)</li><li>您身上的某些特徵 (生物識別技術)</li><ul>|

## <a id="anon-access-cluster"></a>限制 Service Fabric 叢集的匿名存取

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Service Fabric 信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | 環境 - Azure  |
| **參考**              | [Service Fabric 叢集安全性案例](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security) |
| **步驟** | <p>叢集應一律受到安全保護，以防止未經授權使用者連線您的叢集，特別是當叢集上有生產工作負載正在執行時。</p><p>建立 Service Fabric 叢集時，確保安全性模式設定為「安全」並設定必要的 X.509 伺服器憑證。 建立「不安全的」叢集將會在叢集向公用網際網路公開管理端點時，允許任何匿名使用者連線到叢集。</p>|

## <a id="fabric-cn-nn"></a>確保 Service Fabric 的用戶端對節點憑證不同於節點對節點憑證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Service Fabric 信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | 環境 - Azure、環境 - 獨立 |
| **參考**              | [Service Fabric 用戶端對節點憑證安全性](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security/#_client-to-node-certificate-security)、[使用用戶端憑證連線到安全的叢集](https://azure.microsoft.com/documentation/articles/service-fabric-connect-to-secure-cluster/) |
| **步驟** | <p>用戶端對節點憑證安全性是在建立叢集 (透過 Azure 入口網站、Resource Manager 範本或 JSON 範本) 時，藉由指定系統管理用戶端憑證和 (或) 使用者用戶端憑證來設定的。</p><p>您指定的系統管理用戶端憑證和使用者用戶端憑證，應該不同於您為節點對節點安全性指定的主要和次要憑證。</p>|

## <a id="aad-client-fabric"></a>使用 AAD 來向 Service Fabric 叢集驗證用戶端對

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Service Fabric 信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | 環境 - Azure |
| **參考**              | [叢集安全性案例 - 安全性建議](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security/#security-recommendations) |
| **步驟** | 除了用戶端憑證，在 Azure 上執行的叢集也可以使用 Azure Active Directory (AAD) 來保護對管理端點的存取。 對於 Azure 叢集，建議您針對節點對節點安全性使用 AAD 安全性來驗證用戶端和憑證。|

## <a id="fabric-cert-ca"></a>確保從經過核准的憑證授權單位 (CA) 取得 Service Fabric 憑證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Service Fabric 信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | 環境 - Azure |
| **參考**              | [X.509 憑證和 Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security/#x509-certificates-and-service-fabric) |
| **步驟** | <p>Service Fabric 會使用 X.509 伺服器憑證驗證節點和用戶端。</p><p>在 Service Fabric 中使用憑證時，所需可量的一些重要事項︰</p><ul><li>執行生產環境工作負載之叢集中所使用的憑證應該是使用已正確設定的 Windows Server 憑證服務來建立，或是從已核准的憑證授權單位 (CA) 取得。 CA 可以是已核准的外部 CA 或妥善管理的內部公開金鑰基礎結構 (PKI)</li><li>絕對不要在生產環境中使用以 MakeCert.exe 這類工具建立的暫時或測試憑證</li><li>您可以使用自我簽署憑證，但應該只用於測試叢集，不應該在生產環境中使用</li></ul>|

## <a id="standard-authn-id"></a>使用 Identity Server 所支援的標準驗證案例

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Identity Server | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [IdentityServer3 - 概觀](https://identityserver.github.io/Documentation/docsv2/overview/bigPicture.html) |
| **步驟** | <p>以下是 Identity Server 支援的典型互動︰</p><ul><li>瀏覽器與 Web 應用程式通訊</li><li>Web 應用程式與 Web API 通訊 (有時靠自己，有時代表使用者)</li><li>瀏覽器架構應用程式與 Web API 通訊</li><li>原生應用程式與 Web API 通訊</li><li>伺服器架構應用程式與 Web API 通訊</li><li>Web API 與 Web API 通訊 (有時靠自己，有時代表使用者)</li></ul>|

## <a id="override-token"></a>以可調整的替代項目覆寫預設 Identity Server 權杖快取

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Identity Server | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [Identity Server 部署 - 快取](https://identityserver.github.io/Documentation/docsv2/advanced/deployment.html) |
| **步驟** | <p>IdentityServer 已內建簡單的記憶體內部快取。 雖然這適用於小規模原生應用程式，但不會針對中間層和後端應用程式進行調整，原因如下︰</p><ul><li>這些應用程式是由許多使用者同時存取。 在相同存放區中儲存所有的存取權杖會在大規模運作時產生隔離問題並帶來挑戰︰許多使用者 (各自有許多權杖) 的數字和代價很高的查閱作業，可能表示許多使用者，每個都有最多的語彙基元為代表他們執行的應用程式存取的資源</li><li>這些應用程式通常會部署在分散式拓撲上，其中多個節點必須存取相同的快取</li><li>快取的權杖必須在程序回收和停用後存留下來</li><li>基於上述原因，實作 Web 應用程式時，建議以可調整的替代項目 (例如 Azure Redis 快取) 覆寫預設 Identity Server 的權杖快取</li></ul>|

## <a id="binaries-signed"></a>確保已數位簽署所部署應用程式的二進位檔

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 電腦信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確保已數位簽署所部署應用程式的二進位檔，以便驗證二進位檔的完整性|

## <a id="msmq-queues"></a>在連線到 WCF 中的 MSMQ 佇列時啟用驗證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、NET Framework 3 |
| **屬性**              | N/A |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx) |
| **步驟** | 程式無法在連線到 MSMQ 佇列時啟用驗證，攻擊者可能會以匿名方式將訊息提交至佇列進行處理。 如果未使用驗證來連線到用來將訊息傳遞至另一個程式的 MSMQ 佇列，攻擊者可能會提交惡意的匿名訊息。|

### <a name="example"></a>範例
以下 WCF 組態檔的 `<netMsmqBinding/>` 元素會指示 WCF 在連線到 MSMQ 佇列進行訊息傳遞時停用驗證。
```
<bindings>
    <netMsmqBinding>
        <binding>
            <security>
                <transport msmqAuthenticationMode=""None"" />
            </security>
        </binding>
    </netMsmqBinding>
</bindings>
```
將 MSMQ 設定為隨時需要 Windows 網域或憑證驗證才能處理傳入或傳出訊息。

### <a name="example"></a>範例
以下 WCF 組態檔的 `<netMsmqBinding/>` 元素會指示 WCF 在連線到 MSMQ 佇列時啟用憑證驗證。 用戶端會使用 X.509 憑證進行驗證。 用戶端憑證必須存在於伺服器的憑證存放區。
```
<bindings>
    <netMsmqBinding>
        <binding>
            <security>
                <transport msmqAuthenticationMode=""Certificate"" />
            </security>
        </binding>
    </netMsmqBinding>
</bindings>
```

## <a id="message-none"></a>WCF - 請勿將訊息 clientCredentialType 設定為 none

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | .NET Framework 3 |
| **屬性**              | 用戶端認證類型 - None |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify](https://vulncat.fortify.com/en/detail?id=desc.semantic.dotnet.wcf_misconfiguration_anonymous_message_client) |
| **步驟** | 缺少驗證表示每個人都能夠存取此服務。 不會驗證其用戶端的服務可允許所有使用者進行存取。 設定應用程式以針對用戶端認證進行驗證。 將訊息 clientCredentialType 設定為 Windows 或 [憑證] 即可完成此作業。 |

### <a name="example"></a>範例
```
<message clientCredentialType=""Certificate""/>
```

## <a id="transport-none"></a>WCF - 請勿將傳輸 clientCredentialType 設定為 none

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、.NET Framework 3 |
| **屬性**              | 用戶端認證類型 - None |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify](https://vulncat.hpefod.com/en/detail?id=desc.semantic.dotnet.wcf_misconfiguration_anonymous_transport_client) |
| **步驟** | 缺少驗證表示每個人都能夠存取此服務。 不會驗證其用戶端的服務可允許所有使用者存取其功能。 設定應用程式以針對用戶端認證進行驗證。 將傳輸 clientCredentialType 設定為 Windows 或 [憑證] 即可完成此作業。 |

### <a name="example"></a>範例
```
<transport clientCredentialType=""Certificate""/>
```

## <a id="authn-secure-api"></a>確保使用標準驗證技術來保護 Web API

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [ASP.NET Web API 中的驗證和授權](http://www.asp.net/web-api/overview/security/authentication-and-authorization-in-aspnet-web-api)、[外部驗證服務與 ASP.NET Web API (C#)](http://www.asp.net/web-api/overview/security/external-authentication-services) |
| **步驟** | <p>驗證是實體證明其身分識別的程序 (通常透過使用者名稱和密碼等認證)。 有多個驗證通訊協定可列入考量。 下列是其中一些通訊協定︰</p><ul><li>Client certificates</li><li>Windows 架構</li><li>表單架構</li><li>同盟 - ADFS</li><li>同盟 - Azure AD</li><li>同盟 - Identity Server</li></ul><p>[參考] 區段中的連結提供如何實作每個驗證方案來保護 Web API 的低階詳細資料。</p>|

## <a id="authn-aad"></a>使用 Azure Active Directory 所支援的標準驗證案例

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure AD | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [Azure AD 的驗證案例](https://azure.microsoft.com/documentation/articles/active-directory-authentication-scenarios/)、[Azure Active Directory 程式碼範例](https://azure.microsoft.com/documentation/articles/active-directory-code-samples/)、[Azure Active Directory 開發人員指南](https://azure.microsoft.com/documentation/articles/active-directory-developers-guide/) |
| **步驟** | <p>Azure Active Directory (Azure AD) 提供身分識別做為服務，支援業界標準通訊協定 (例如 OAuth 2.0 和 OpenID Connect)，以簡化開發人員的驗證工作。 以下是 Azure AD 支援的五個主要應用程式案例：</p><ul><li>Web 瀏覽器到 Web 應用程式：使用者需要登入 Azure AD 所保護的 Web 應用程式</li><li>單一頁面應用程式 (SPA)：使用者需要登入 Azure AD 所保護的單一頁面應用程式</li><li>原生應用程式到 Web API：在電話、平板電腦或電腦執行的原生應用程式需要驗證使用者，以便從 Azure AD 所保護的 Web API 取得資源</li><li>Web 應用程式到 Web API：Web 應用程式需要從 Azure AD 所保護的 Web API 取得資源</li><li>Daemon 或伺服器應用程式到 Web API：無 Web 使用者介面的 Daemon 應用程式或伺服器應用程式，需要從 Azure AD 所保護的 Web API 取得資源</li></ul><p>請參閱 [參考] 區段中的連結，以取得低階實作詳細資料</p>|

## <a id="adal-scalable"></a>以可調整的替代項目覆寫預設 ADAL 權杖快取

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure AD | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [Modern Authentication with Azure Active Directory for Web Applications](https://blogs.msdn.microsoft.com/microsoft_press/2016/01/04/new-book-modern-authentication-with-azure-active-directory-for-web-applications/)、[使用 Redis 作為 ADAL 權杖快取](https://blogs.msdn.microsoft.com/mrochon/2016/09/19/using-redis-as-adal-token-cache/)  |
| **步驟** | <p>ADAL (Active Directory 驗證程式庫) 使用的預設快取是依賴靜態存放區、整個程序可用的記憶體內部快取。 雖然這適用於原生應用程式，但不會針對中間層和後端應用程式進行調整，原因如下︰</p><ul><li>這些應用程式是由許多使用者同時存取。 在相同存放區中儲存所有的存取權杖會在大規模運作時產生隔離問題並帶來挑戰︰許多使用者 (各自有許多權杖) 的數字和代價很高的查閱作業，可能表示許多使用者，每個都有最多的語彙基元為代表他們執行的應用程式存取的資源</li><li>這些應用程式通常會部署在分散式拓撲上，其中多個節點必須存取相同的快取</li><li>快取的權杖必須在程序回收和停用後存留下來</li></ul><p>基於上述原因，實作 Web 應用程式時，建議以可調整的替代項目 (例如 Azure Redis 快取) 覆寫預設 ADAL 權杖快取。</p>|

## <a id="tokenreplaycache-adal"></a>確保使用 TokenReplayCache 來防止重新執行 ADAL 驗證權杖

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure AD | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [Modern Authentication with Azure Active Directory for Web Applications](https://blogs.msdn.microsoft.com/microsoft_press/2016/01/04/new-book-modern-authentication-with-azure-active-directory-for-web-applications/) |
| **步驟** | <p>TokenReplayCache 屬性可讓開發人員為了確認沒有權杖可使用一次以上，而定義權杖重新執行快取以及可用來儲存權杖的存放區。</p><p>這是對抗常見攻擊 (恰如其名的權杖重新執行攻擊) 的措施︰攔截登入時所傳送權杖的攻擊者可能會試著將權杖再次傳送至應用程式 (「重新執行」它)，以便建立新的工作階段。 例如，在 OIDC 程式碼授與流程中，使用者驗證成功後，對信賴憑證者之 "/signin-oidc" 端點的要求是由 "id_token"、"code" 和 "state" 參數所構成。</p><p>信賴憑證者會驗證此要求並建立新的工作階段。 如果敵人擷取此要求並重新執行它，敵人即可建立成功的工作階段並欺騙使用者。 OpenID Connect 中存在的 nonce 可以限制，但未完全消除可成功發動攻擊的情況。 若要保護其應用程式，開發人員可以提供 ITokenReplayCache 的實作，並將執行個體指派給 TokenReplayCache。</p>|

### <a name="example"></a>範例
```csharp
// ITokenReplayCache defined in ADAL
public interface ITokenReplayCache
{
bool TryAdd(string securityToken, DateTime expiresOn);
bool TryFind(string securityToken);
}
```

### <a name="example"></a>範例
以下是 ITokenReplayCache 介面的範例實作。 (請自訂並實作您的專案特定快取架構)
```csharp
public class TokenReplayCache : ITokenReplayCache
{
    private readonly ICacheProvider cache; // Your project-specific cache provider
    public TokenReplayCache(ICacheProvider cache)
    {
        this.cache = cache;
    }
    public bool TryAdd(string securityToken, DateTime expiresOn)
    {
        if (this.cache.Get<string>(securityToken) == null)
        {
            this.cache.Set(securityToken, securityToken);
            return true;
        }
        return false;
    }
    public bool TryFind(string securityToken)
    {
        return this.cache.Get<string>(securityToken) != null;
    }
}
```
實作的快取必須透過 "TokenValidationParameters" 屬性在 OIDC 選項中參照，如下所示。
```csharp
OpenIdConnectOptions openIdConnectOptions = new OpenIdConnectOptions
{
    AutomaticAuthenticate = true,
    ... // other configuration properties follow..
    TokenValidationParameters = new TokenValidationParameters
    {
        TokenReplayCache = new TokenReplayCache(/*Inject your cache provider*/);
    }
}
```

請注意，若要測試此組態的有效性，請登入本機受 OIDC 保護的應用程式並擷取 Fiddler 中對 `"/signin-oidc"` 端點的要求。 若未妥善保護，在 Fiddler 中重新執行此要求會設定新的工作階段 cookie。 在新增 TokenReplayCache 後重新執行要求時，應用程式會擲出例外狀況，如下所示︰`SecurityTokenReplayDetectedException: IDX10228: The securityToken has previously been validated, securityToken: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSIsImtpZCI6Ik1uQ1......`

## <a id="adal-oauth2"></a>使用 ADAL 程式庫來管理從 OAuth2 用戶端至 AAD (或內部部署 AD) 的權杖要求

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure AD | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [ADAL](https://azure.microsoft.com/documentation/articles/active-directory-authentication-libraries/) |
| **步驟** | <p>Azure AD 驗證程式庫 (ADAL) 可讓用戶端應用程式開發人員輕鬆地向雲端或內部部署 Active Directory (AD) 驗證使用者，然後取得存取權杖來保護 API 呼叫。</p><p>ADAL 有許多功能可使開發人員的驗證更容易，例如非同步支援、儲存存取權杖和更新權杖的可設定權杖快取、當存取權杖到期並且更新權杖可供使用時自動更新權杖等等。</p><p>藉由處理大部分的複雜度，ADAL 可以幫助開發人員專注於他們的應用程式中的商務邏輯，並輕鬆地保護資源而不需成為安全性方面的專家。 不同的程式庫適用於 .NET、JavaScript (用戶端和 Node.js)、iOS、Android 和 Java。</p>|

## <a id="authn-devices-field"></a>驗證連線到現場閘道的裝置

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 現場閘道 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確保每個裝置在接受資料之前，以及在利用雲端閘道進行上游通訊之前，已經由現場閘道驗證。 此外，確保裝置與每一裝置認證連線，即可唯一識別個別裝置。|

## <a id="authn-devices-cloud"></a>確保會驗證連線到雲端閘道的裝置

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 雲端閘道 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、C#、Node.JS  |
| **屬性**              | N/A、閘道選擇 - Azure IoT 中樞 |
| **參考**              | N/A、[採用 .NET 的 Azure IoT 中樞](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)、[開始使用 IoT 中樞和 Node JS](https://azure.microsoft.com/documentation/articles/iot-hub-node-node-getstarted)、[使用 SAS 和憑證保護 IoT](https://azure.microsoft.com/documentation/articles/iot-hub-sas-tokens/)、[Git 儲存機制](https://github.com/Azure/azure-iot-sdks/tree/master/node) |
| **步驟** | <ul><li>**泛型︰** 使用傳輸層安全性 (TLS) 或 IPSec 來驗證裝置。 如果裝置無法處理完整的非對稱密碼編譯，則基礎結構應該支援在這些裝置上使用預先共用金鑰 (PSK)。 利用 Azure AD、Oauth。</li><li>**C#：** 建立 DeviceClient 執行個體時，Create 方法預設會建立一個使用 AMQP 通訊協定來與 IoT 中樞通訊的 DeviceClient 執行個體。 若要使用 HTTPS 通訊協定，請使用可讓您指定通訊協定的 Create 方法的覆寫。 若您使用 HTTPS 通訊協定，您也應該將 `Microsoft.AspNet.WebApi.Client` Nuget 套件新增至您的專案，以包含 `System.Net.Http.Formatting` 命名空間。</li></ul>|

### <a name="example"></a>範例
```csharp
static DeviceClient deviceClient;

static string deviceKey = "{device key}";
static string iotHubUri = "{iot hub hostname}";

var messageString = "{message in string format}";
var message = new Message(Encoding.ASCII.GetBytes(messageString));

deviceClient = DeviceClient.Create(iotHubUri, new DeviceAuthenticationWithRegistrySymmetricKey("myFirstDevice", deviceKey));

await deviceClient.SendEventAsync(message);
```

### <a name="example"></a>範例
**Node.JS：驗證**
#### <a name="symmetric-key"></a>對稱金鑰
* 在 Azure 上建立 IoT 中樞
* 在裝置識別登錄中建立一個項目
    ```javascript
    var device = new iothub.Device(null);
    device.deviceId = <DeviceId >
    registry.create(device, function(err, deviceInfo, res) {})
    ```
* 建立模擬裝置
    ```javascript
    var clientFromConnectionString = require('azure-iot-device-amqp').clientFromConnectionString;
    var Message = require('azure-iot-device').Message;
    var connectionString = 'HostName=<HostName>DeviceId=<DeviceId>SharedAccessKey=<SharedAccessKey>';
    var client = clientFromConnectionString(connectionString);
    ```
#### <a name="sas-token"></a>SAS 權杖
* 取得使用對稱金鑰時在內部產生的權杖，但我們也可以明確地產生和使用權杖
* 定義通訊協定︰`var Http = require('azure-iot-device-http').Http;`
* 建立 SAS 權杖：
    ```javascript
    resourceUri = encodeURIComponent(resourceUri.toLowerCase()).toLowerCase();
    var deviceName = "<deviceName >";
    var expires = (Date.now() / 1000) + expiresInMins * 60;
    var toSign = resourceUri + '\n' + expires;
    // using crypto
    var decodedPassword = new Buffer(signingKey, 'base64').toString('binary');
    const hmac = crypto.createHmac('sha256', decodedPassword);
    hmac.update(toSign);
    var base64signature = hmac.digest('base64');
    var base64UriEncoded = encodeURIComponent(base64signature);
    // construct authorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "%2fdevices%2f"+deviceName+"&sig="
    + base64UriEncoded + "&se=" + expires;
    if (policyName) token += "&skn="+policyName;
    return token;
    ```
* 使用 SAS 權杖進行連線︰ 
    ```javascript
    Client.fromSharedAccessSignature(sas, Http); 
    ```
#### <a name="certificates"></a>憑證
* 使用任何工具 (例如 OpenSSL) 產生自我簽署的 X509 憑證，以產生 .cert 和 .key 檔案來分別儲存憑證和金鑰
* 使用憑證佈建可接受安全連線的裝置。
    ```javascript
    var connectionString = '<connectionString>';
    var registry = iothub.Registry.fromConnectionString(connectionString);
    var deviceJSON = {deviceId:"<deviceId>",
    authentication: {
        x509Thumbprint: {
        primaryThumbprint: "<PrimaryThumbprint>",
        secondaryThumbprint: "<SecondaryThumbprint>"
        }
    }}
    var device = deviceJSON;
    registry.create(device, function (err) {});
    ```
* 使用憑證連接裝置
    ```javascript
    var Protocol = require('azure-iot-device-http').Http;
    var Client = require('azure-iot-device').Client;
    var connectionString = 'HostName=<HostName>DeviceId=<DeviceId>x509=true';
    var client = Client.fromConnectionString(connectionString, Protocol);
    var options = {
        key: fs.readFileSync('./key.pem', 'utf8'),
        cert: fs.readFileSync('./server.crt', 'utf8')
    }; 
    // Calling setOptions with the x509 certificate and key (and optionally, passphrase) will configure the client //transport to use x509 when connecting to IoT Hub
    client.setOptions(options);
    //call fn to execute after the connection is set up
    client.open(fn);
    ```

## <a id="authn-cred"></a>使用每一裝置驗證認證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 雲端閘道  | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | 閘道選擇 - Azure IoT 中樞 |
| **參考**              | [Azure IoT 中樞安全性權杖](https://azure.microsoft.com/documentation/articles/iot-hub-sas-tokens/) |
| **步驟** | 使用每一裝置驗證認證搭配以裝置金鑰或用戶端憑證為基礎的 SaS 權杖，而不是 IoT 中樞共用存取原則。 這可防止其他人在一個裝置或現場閘道上重複使用驗證權杖 |

## <a id="req-containers-anon"></a>確保只有必要的容器和 blob 會取得匿名讀取權限

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 儲存體 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | StorageType - Blob |
| **參考**              | [管理對容器與 Blob 的匿名讀取權限](https://azure.microsoft.com/documentation/articles/storage-manage-access-to-resources/)、[共用存取簽章，第 1 部分：了解 SAS 模型](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/) |
| **步驟** | <p>根據預設，可能只有儲存體帳戶的擁有者能存取容器及其內部的任何 Blob。 若要為匿名使用者授與容器及其 Blob 的讀取權限，您可以設定容器權限以允許公用存取。 匿名使用者可以讀取可公開存取之容器內的 Blob，而不需驗證要求。</p><p>容器會提供下列選項來管理容器存取：</p><ul><li>完整的公開讀取權限：可以透過匿名要求讀取容器和 Blob 資料。 用戶端可以透過匿名要求列舉容器內的 Blob，但無法列舉儲存體帳戶內的容器。</li><li>僅對 Blob 有公開讀取權限：您可以透過匿名要求讀取此容器內的 Blob 資料，但您無法使用容器資料。 用戶端無法透過匿名要求列舉容器內的 Blob</li><li>沒有公開讀取權限：只有帳戶擁有者可以讀取容器和 Blob 資料。</li></ul><p>匿名存取適用於某些 Blob 應永遠可供匿名讀取存取的狀況。 如需更精密的控制，您可以建立共用存取簽章，以便在指定的時間間隔內，使用不同的權限來委派受限制的存取權。 確保容器和 blob (可能包含敏感性資料) 不會意外取得匿名存取權</p>|

## <a id="limited-access-sas"></a>使用 SAS 或 SAP 對 Azure 儲存體中的物件授與有限的存取權

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 儲存體 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A |
| **參考**              | [共用存取簽章，第 1 部分：了解 SAS 模型](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)、[共用存取簽章，第 2 部分：透過 Blob 儲存體建立及使用 SAS](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-2/)、[如何使用共用存取簽章和預存存取原則將存取權委派給帳戶中的物件](https://azure.microsoft.com/documentation/articles/storage-security-guide/#_how-to-delegate-access-to-objects-in-your-account-using-shared-access-signatures-and-stored-access-policies) |
| **步驟** | <p>若要在無需公開帳戶存取金鑰的情況下，將儲存體帳戶中物件的限制存取授與其他用戶端，則使用共用存取簽章 (SAS) 會是個佷有效的方式。 SAS 是一種 URI，URI 會在其查詢參數中包含通過驗證存取儲存體資源的所有必要資訊。 若要使用 SAS 存取儲存體資源，用戶端只需在適當的建構函式或方法中傳入 SAS 即可。</p><p>當您想要將儲存體帳戶中的資源存取權提供給無法放心託付帳戶金鑰的用戶端時，您可以使用 SAS。 您的儲存體帳戶金鑰包括主要和次要金鑰，兩者皆可授與帳戶及帳戶內所有資源的系統管理存取權。 提供任一帳戶金鑰都會讓您的帳戶受到惡意或粗心使用的可能性。 共用存取簽章提供一個安全的替代方式，無需帳戶金鑰便可讓其他用戶端根據他們被授與的權限，來讀取、寫入及刪除儲存體帳戶中的資料。</p><p>如果您每次都有一組類似的邏輯參數，使用預存存取原則 (SAP) 會是個不錯的做法。 因為使用衍生自預存存取原則的 SAS 讓您能夠立即撤銷該 SAS，所以建議的最佳做法是一律儲存預存存取原則 (如果可能)。</p>|