---
title: Azure 健康情況分析藍圖
description: HIPAA/HITRUST 健康情況分析藍圖的部署指導方針
services: security
documentationcenter: na
author: RajeevRangappa
ms.assetid: 26566e0a-0a54-49f4-a91d-48e20b7cef71
ms.service: security
ms.topic: article
ms.date: 07/23/2018
ms.author: rarangap
ms.openlocfilehash: b20da0f31f197ed23aa73b185d127a6d5f2dbd8a
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39214936"
---
# <a name="azure-security-and-compliance-blueprint---hipaahitrust-health-data-and-ai"></a>Azure 安全性與合規性藍圖 - HIPAA/HITRUST 醫療資料和 AI

## <a name="overview"></a>概觀


  **Azure 安全性與合規性藍圖 - HIPAA/HITRUST 醫療資料和 AI 提供 Azure PaaS 解決方案的統包部署，以示範如何安全地內嵌、儲存、分析健康情況資料並與之互動，同時能夠滿足業界合規性需求。該藍圖有助於加速資料受管制之客戶採用及使用雲端。**

Azure 安全性與合規性藍圖 - HIPAA/HITRUST 醫療資料和 AI 藍圖提供工具與指導方針，協助部署安全的健康保險流通與責任法案 (HIPAA)，以及醫療資訊信任聯盟 (HITRUST) 準備就緒的平台即服務 (PaaS) 環境，以便在部署為端對端解決方案的安全、多層式雲端環境中內嵌、儲存、分析個人與非個人醫療記錄並與之互動。 此藍圖將示範常見的參考架構，並融入可輕鬆採用 Microsoft Azure 的設計。 此架構所說明的解決方案符合組織尋求雲端方法來減少部署負擔與成本的需求。

![](images/components.png)

此解決方案的設計旨在使用以快速醫療保健互通性資源 (FHIR，一種以電子方式交換醫療保健資訊的全球標準) 格式化的範例資料集，並以安全方式加以儲存。 然後客戶可以使用 Azure Machine Learning Studio 充分利用功能強大的商業智慧工具和分析，以檢閱針對範例資料所做的預測。 作為 Azure Machine Learning Studio 可協助的實驗種類範例，藍圖包含了範例資料集、指令碼，以及用來預測病患待在醫院設施之時間的工具。 

此藍圖旨在做為模組化基礎，讓客戶調整其特定需求，進而開發新的 Azure Machine Learning 實驗以解決臨床及操作上的使用案例。 其設計是在部署時既安全且符合規範；不過，客戶必須負責正確設定角色並實作任何修改。 請注意：

-   此藍圖提供了一個基準，可協助客戶在 HITRUST 和 HIPAA 環境中使用 Microsoft Azure。

-   雖然藍圖的設計是符合 HIPAA 與 HITRUST (透過一般安全性架構 -- CSF)，但是除非經過外部稽核員依據 HIPAA 和 HITRUST 認證需求進行認證，否則不應視之為符合規範。

-   客戶必須負責針對使用此基礎架構建立的任何解決方案，進行適當的安全性與合規性檢閱。

## <a name="deploying-the-automation"></a>部署自動化

- 若要部署解決方案，請遵循部署指導方針中提供的指示進行。 

[![](./images/deploy.png)](https://aka.ms/healthblueprintdeploy)

如需此解決方案運作方式簡短概觀，請觀賞說明及示範其部署的此[影片](https://aka.ms/healthblueprintvideo) \(英文\)。

- 常見問題位於[常見問題集](https://aka.ms/healthblueprintfaq)指導方針中。

-   **架構圖。** 此圖顯示用於藍圖和範例使用案例的參考架構。

-   **部署範本**。 在此部署中，透過在安裝期間指定設定參數，系統會使用 [Azure Resource Manager 範本](/azure/azure-resource-manager/resource-group-overview#template-deployment)來將架構的元件自動部署至 Microsoft Azure。

-   **[自動化部署指令碼](https://aka.ms/healthblueprintdeploy)**。 這些指令碼可協助您部署解決方案。 指令碼包括：


-   模組安裝和[全域管理員](/azure/active-directory/active-directory-assign-admin-roles-azure-portal)安裝指令碼會用來進行安裝，以及確認必要的 PowerShell 模組和全域管理員角色已正確設定。 
-   安裝 PowerShell 指令碼是用來部署解決方案的，此指令碼是透過包含預先建置之示範函式的 .zip 檔案所提供。

## <a name="solution-components"></a>方案元件


基礎架構是由下列元件組成：

-   **[威脅模型](https://aka.ms/healththreatmodel)** 採用 tm7 格式提供的完整威脅模型，用來搭配 [Microsoft Threat Modeling Tool](https://www.microsoft.com/en-us/download/details.aspx?id=49168) 一起使用，會顯示解決方案的元件、元件之間的資料流程，以及信任界限。 此模型可協助客戶在開發 Machine Learning Studio 元件或其他修改時，了解系統基礎結構中的潛在風險要點。

-   **[客戶實作矩陣](https://aka.ms/healthcrmblueprint)** Microsoft Excel 活頁簿會列出相關的 HITRUST 需求，並說明 Microsoft 和客戶如何負責符合每一個需求。

-   **[健康狀態檢閱。](https://aka.ms/healthreviewpaper)** 解決方案是由 Coalfire systems, Inc. 檢閱。健康情況合規性 (HIPAA，以及 HITRUST) 檢閱和實作指導方針提供稽核員\'對解決方案的檢閱，以及將藍圖轉換為實際可執行部署的考量。

# <a name="architectural-diagram"></a>架構圖


![](images/refarch.png)

## <a name="roles"></a>角色


藍圖為系統管理使用者 (操作員) 定義兩個角色，並為醫院管理與病患照護中的使用者定義三個角色。 第六個角色是針對要評估 HIPAA 與其他法規合規性的稽核員所定義的。 Azure 角色型存取控制 (RBAC) 可透過內建與自訂角色，為解決方案的每個使用者做到精確且專注的存取權管理。 請參閱[在 Azure 入口網站中開始使用角色型存取控制](https://docs.microsoft.com/azure/role-based-access-control/overview)和[Azure 角色型存取控制的內建角色](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)以了解關於 RBAC、角色和權限的詳細資訊。

### <a name="site-administrator"></a>網站系統管理員


網站系統管理員負責客戶的 Azure 訂用帳戶。 他們控制整體部署，但沒有病患記錄的存取權。

-   預設角色指派：[擁有者](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#owner)

-   自訂角色指派：N/A

-   範圍：訂用帳戶

### <a name="database-analyst"></a>資料庫分析師

資料庫分析師會管理 SQL Server 執行個體與資料庫。
他們沒有病患記錄的存取權。

-   內建角色指派：[SQL DB 參與者](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#sql-db-contributor)[SQL Server 參與者](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#sql-server-contributor)

-   自訂角色指派：N/A

-   範圍：ResourceGroup

 ### <a name="data-scientist"></a>資料科學家


資料科學家負責操作 Azure Machine Learning Studio。 他們可以匯入、匯出及管理資料，以及執行報告。 資料科學家具有病患資料的存取權，但沒有系統管理權限。

-   內建角色指派：[儲存體帳戶參與者](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-account-contributor)

-   自訂角色指派：N/A

-   範圍：ResourceGroup

### <a name="chief-medical-information-officer-cmio"></a>醫療資訊長 (CMIO)


CMIO 是橫跨醫療保健組織內的資訊/技術和醫護專業的人。 其職責通常包括利用分析來判斷是否適當分配組織內的資源。

-   內建角色指派：無

### <a name="care-line-manager"></a>醫療服務管理人員


醫療服務管理人員是與病患看護直接相關的人。
此角色需要監視個別病患的狀態，並確保有符合其病患之特定看護需求的人員。 醫療服務管理人員負責新增和更新病患記錄。

-   內建角色指派：無

-   自訂角色指派：擁有執行 HealthcareDemo.ps1 的權限，以便執行病患住院與出院。

-   範圍：ResourceGroup

### <a name="auditor"></a>稽核員


稽核員會評估解決方案的合規性。 他們沒有網路的直接存取權。

-   內建角色指派：[讀者](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#reader)

-   自訂角色指派：N/A

-   範圍：訂用帳戶

## <a name="example-use-case"></a>使用案例範例


此藍圖所含的使用案例範例說明如何使用藍圖在雲端針對健康情況資料啟用機器學習和分析。 Contosoclinic 是位於美國的一家小型醫院。 該醫院的網路系統管理員想要使用 Azure Machine Learning Studio 更準確預測病患住院時的停留時間，以提高操作工作負載的效率，並提升醫院可提供的照護品質。

### <a name="predicting-length-of-stay"></a>預測停留時間


使用案例範例使用 Azure Machine Learning Studio 比較病患入院時的醫療詳細資料與過去病患的彙總歷史資料，來預測新住院病患的停留時間。
藍圖包含大量的匿名醫療記錄，以示範解決方案的訓練和預測能力。 在生產環境部署中，客戶應使用自己的記錄來訓練解決方案，以便獲得可反映其環境、設施和病患的更準確預測。

### <a name="users-and-roles"></a>使用者和角色


**網站系統管理員 -- Alex**

*電子郵件：Alex\_SiteAdmin*

Alex 的工作是評估可減少內部部署網路的管理負擔並降低管理成本的技術。 Alex 已經評估 Azure 一段時間了，但是一直努力設定符合 HiTrust 合規性需求所需的服務，以便將病患資料儲存在雲端。 Alex 選擇 Azure Health AI 部署合規性準備就緒的健康情況解決方案，此解決方案已滿足符合 HiTrust 客戶需求的要求。

**資料科學家 -- Debra**

*電子郵件：Debra\_DataScientist*

Debra 負責使用及建立分析醫療記錄的模型，以便提供病患照護的深入解析。 Debra 使用 SQL 和 R 統計式程式設計語言來建立她的模型。

**資料庫分析師 -- Danny**

*電子郵件：Danny\_DBAnalyst*

Danny 是與儲存 Contosoclinic 所有病患資料之 Microsoft SQL Server 有關事項的主要連絡人。 Danny 是經驗豐富的 SQL Server 系統管理員，最近已經很熟悉 Azure SQL Database。

**醫療資訊長 -- Caroline**

Caroline 與醫療服務管理人員 Chris 及資料科學家 Debra 合作，判斷哪些因素會影響病患的停留時間。
Caroline 使用停留時間 (LOS) 解決方案的預測，來判斷醫院網路中的資源分配是否適當。 例如，使用此解決方案中提供的儀表板。

**醫療服務管理人員 -- Chris**

*電子郵件：Chris\_CareLineManager*

作為在 Contosoclinic 中直接負責管理病患住院與出院的人員，Chris 使用 LOS 解決方案所產生的預測，確保有足夠的適當人員可在病患在設施停留期間提供照護。

**稽核員 -- Han**

*電子郵件：Han\_稽核員*

Han 是獲得認證的稽核員，擁有 ISO、SOC 和 HiTrust 的稽核經驗。 Han 受雇檢閱 Contosoclinc 的網路。 Han 可以檢閱隨解決方案提供的客戶責任對照表，以確保藍圖和 LOS 解決方案可用來儲存、處理及顯示敏感的個人資料。


# <a name="design-configuration"></a>設計組態


本節將詳細說明預設組態，以及概述用於下列事項的藍圖內建安全措施：

- **內嵌**包含 FHIR 資料來源在內的資料原始來源
- **儲存**敏感性資訊
- **分析**及預測結果
- 與結果和預測**互動**
- 解決方案的**身分識別**管理
- 已啟用**安全性**的功能


## <a name="identity"></a>身分識別 

### <a name="azure-active-directory-and-role-based-access-control-rbac"></a>Azure Active Directory 和角色型存取控制 (RBAC)


**驗證：**

-   [Azure Active Directory (Azure AD)](https://azure.microsoft.com/services/active-directory/) 是 Microsoft\' 多租用戶雲端式目錄和身分識別管理服務。 解決方案的所有使用者都是在 Azure Active Directory 中建立，包括存取 SQL Database 的使用者。



-   應用程式的驗證是使用 Azure AD 執行。 如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](/azure/active-directory/develop/active-directory-integrating-applications)。

-   [Azure Active Directory Identity Protection](/azure/active-directory/active-directory-identityprotection) 會偵測影響您組織身分識別的潛在弱點，並為偵測到的組織身分識別相關可疑活動設定自動回應，以及調查可疑事件並採取適當動作來解決這些可疑事件。

-   [Azure 角色型存取控制 (RBAC)](/azure/role-based-access-control/role-assignments-portal) 可以對 Azure 進行精確且專注的存取權管理。 可存取訂用帳戶的身分會限制為訂用帳戶系統管理員，而可存取 Azure Key Vault 的身分則是限制為網站系統管理員。 需要強式密碼 (至少 12 個字元，其中至少一個大寫/小寫字母、數字和特殊字元)。

-   在部署期間啟用 -enableMFA 參數時，支援多重要素驗證。

-   在部署期間啟用 -enableADDomainPasswordPolicy 參數時，密碼會在 60 天後到期。

**角色：**

-   解決方案使用[內建角色](/azure/role-based-access-control/built-in-roles)來管理資源的存取權。

-   所有使用者依預設會被指派特定的內建角色。

### <a name="azure-key-vault"></a>Azure 金鑰保存庫

-   儲存在金鑰保存庫中的資料包括：

    -   Application Insights 金鑰
    -   病患資料儲存體存取金鑰
    -   病患連接字串
    -   病患資料表名稱
    -   Azure ML Web 服務端點
    -   Azure ML 服務 API 金鑰

-   進階存取原則是視需要設定的
-   金鑰保存庫存取原則是使用金鑰和密碼的最低必要權限所定義的
-   金鑰保存庫中的所有金鑰和密碼都有到期日
-   金鑰保存庫中的所有金鑰都受到 HSM 的保護，\[金鑰類型 = 受 HSM 保護的 2048 位元 RSA 金鑰\]
-   所有使用者/身分識別皆使用角色型存取控制 (RBAC) 授與最低要求權限
-   應用程式不會共用金鑰保存庫，除非它們彼此信任且必須在執行階段存取相同金鑰
-   金鑰保存庫的診斷記錄的保留期至少 365 天。
-   允許的金鑰密碼編譯作業僅限於需要的密碼編譯項目

## <a name="ingest"></a>內嵌 

### <a name="azure-functions"></a>Azure Functions
解決方案的設計旨在使用 [Azure Functions](/azure/azure-functions/) 處理分析示範中使用的停留時間資料範例。 函式中有三個功能已建立。

**1.客戶資料 phi 資料的大量匯入**

當使用示範指令碼時。 .\\HealthcareDemo.ps1 與 **BulkPatientAdmission** 參數，如**部署及執行示範**中所概述，它執行下列處理管線：
1. **Azure Blob 儲存體** - 上傳至儲存體的病患資料 .csv 檔案範例
2. **Event Grid** - 事件發佈資料到 Azure Function (大量匯入 - Blob 事件)
3. **Azure Function** - 執行處理並使用安全函式 - event(type; blob url) 將資料儲存到 SQL 儲存體
4. **SQL DB** - 使用標籤進行分類之病患資料的資料庫存放區，並啟動 ML 流程進行訓練實驗。

![](images/dataflow.png)

此外，Azure Function 的設計旨在讀取及保護範例資料集中使用下列標籤的專屬敏感性資料：
- dataProfile => “ePHI”
- 擁有者 => \<網站系統管理員 UPN\>
- 環境 => “Pilot”
- 部門 => “Global Ecosystem" 標籤會套用到病患「名稱」識別為純文字的範例資料集。

**2.新病患住院**

當使用示範指令碼時。 .\\HealthcareDemo.ps1 與 **BulkPatientadmission** 參數，如**部署及執行示範**中所概述，它執行下列處理管線：![](images/securetransact.png)
**1.Azure Function** 會被觸發，而且該函式會從 Azure Active Directory 要求[持有人權杖](/rest/api/)。

**2.金鑰保存庫**會要求與所要求權杖相關聯的密碼。

**3. Azure 角色會驗證要求，並授權金鑰保存庫的存取要求。

**4.金鑰保存庫**會傳回密碼，在此情況下為 SQL DB 連接字串。

**5.Azure Function** 使用連接字串安全地連線到 SQL Database，並繼續進一步處理以儲存 ePHI 資料。

為了實現資料儲存，會遵循快速醫療保健互通性資源 (FHIR，讀音同 Fire) 實作常見的 API 結構描述。 此函式提供下列 FHIR 交換項目：

-   [病患結構描述](https://www.hl7.org/fhir/patient.html)涵蓋有關病患的「身分」資訊。

-   [觀察結構描述](https://www.hl7.org/fhir/observation.html)涵蓋醫療保健中的中心元素，用於支援診斷、監視進度、判斷基準與模式，甚至是擷取人口統計特性。 

-   [看診結構描述](https://www.hl7.org/fhir/encounter.html)涵蓋看診類型，例如不需臥床、緊急、家庭健康、住院病患，以及虛擬看診。

-   [條件結構描述](https://www.hl7.org/fhir/condition.html)涵蓋有關條件、問題、診斷或其他事件、情況、問題或已達令人擔憂程度的臨床概念的詳細資訊。  



### <a name="event-grid"></a>Event Grid


解決方案支援 Azure Event Grid，這是一種可管理從任何來源至任何目的地的所有事件路由的單一服務，只要有下列條件即可：

-   [安全性和驗證](/azure/event-grid/security-authentication)

-   [角色型存取控制](/azure/event-grid/security-authentication#management-access-control)用於各種管理操作，例如列出事件訂用帳戶、建立新訂用帳戶，以及產生金鑰

-   稽核

## <a name="store"></a>儲存 

### <a name="sql-database-and-server"></a>SQL Database 和 Server 


-   [透明資料加密 (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) 使用 Azure Key Vault 中儲存的金鑰，提供 Azure SQL Database 中儲存之資料的即時加密和解密。

-   [SQL 弱點評量](https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment)是容易設定的工具，可探索、追蹤及修復潛在的資料庫弱點。

-   [SQL Database 威脅偵測](/azure/sql-database/sql-database-threat-detection)已啟用。

-   [SQL Database 稽核](/azure/sql-database/sql-database-auditing)已啟用。

-   [SQL Database 計量和診斷記錄](/azure/sql-database/sql-database-metrics-diag-logging)已啟用。

-   [伺服器和資料庫等級防火牆規則](/azure/sql-database/sql-database-firewall-configure)已再加強。

-   [Always Encrypted 資料行](/azure/sql-database/sql-database-always-encrypted-azure-key-vault)是用來保護病患名字、中間名和姓氏。
    此外，資料庫資料行加密也會使用 Azure Active Directory (AD) 向 Azure SQL Database 驗證應用程式。

-   SQL Database 和 SQL Server 的存取權是根據最低權限原則所設定的。

-   只有必要的 IP 位址能夠透過 SQL 防火牆存取。

### <a name="storage-accounts"></a>儲存體帳戶


-   [移動中的資料只會使用 TLS/SSL 來傳輸](/azure/storage/common/storage-require-secure-transfer?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json)。

-   容器不允許匿名存取。

-   警示規則是針對追蹤匿名活動所設定。

-   為了存取儲存體帳戶資源，必須使用 HTTPS。

-   會記錄及監視驗證要求資料。

-   Blob 儲存體中的資料在靜止時會加密。

## <a name="analyze"></a>分析

### <a name="machine-learning"></a>Machine Learning


-   會針對 Machine Learning Studio Web 服務[啟用記錄](/azure/machine-learning/studio/web-services-logging)。
- 使用 [Machine Learning Studio](/azure/machine-learning/desktop-workbench/experimentation-service-configuration) 工作台需要開發實驗，以提供預測解決方案集的能力。 [整合工作台](/azure/machine-learning/desktop-workbench/using-git-ml-project)可協助簡化實驗管理。

## <a name="security"></a>安全性

### <a name="azure-security-center"></a>Azure 資訊安全中心
- [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)可提供所有 Azure 資源安全性狀態的集中檢閱。 只需看一眼，您就可以確認安全性控制項是否已就緒並正確設定，且可以快速找出任何需要注意的資源。 

- [Azure Advisor](/azure/advisor/advisor-overview) 是個人化的雲端顧問，可協助您依最佳做法來最佳化您的 Azure 部署。 它可分析您的資源組態和使用量遙測，然後建議可協助您改善 Azure 資源的成本效益、效能、高可用性和安全性的解決方案。

### <a name="application-insights"></a>Application Insights
- [Application Insights](/azure/application-insights/app-insights-overview) 是多個平台上的 Web 開發人員所適用的可延伸應用程式效能管理 (APM) 服務。 您可以使用它來監視即時 Web 應用程式。 它會偵測效能異常。 其中包括強大的分析工具可協助您診斷問題，並了解使用者實際如何運用您的應用程式。 它是設計來協助您持續改善效能和可用性。

### <a name="azure-alerts"></a>Azure 警示
- [警示會提供監視 Azure 服務的方法，並可讓您針對資料設定條件。 當警示條件符合監視資料時，警示也會提供通知。

### <a name="operations-management-suite-oms"></a>Operations Management Suite (OMS)
[Operations Management Suite (也稱為 OMS)](/azure/operations-management-suite/operations-management-suite-overview) 是管理服務的集合。

-   已針對資訊安全中心啟用工作區

-   已針對工作負載監視啟用工作區

-   已針對下列項目啟用工作負載監視：

    -   身分識別與存取

    -   安全性和稽核

    -   Azure SQL DB 分析

    -   [Azure WebApp 分析](/azure/log-analytics/log-analytics-azure-web-apps-analytics)解決方案

    -   金鑰保存庫分析

    -   變更追蹤

-   [Application Insights Connector (預覽)](/azure/log-analytics/log-analytics-app-insights-connector)已啟用

-   [活動 Log Analytics](/azure/log-analytics/log-analytics-activity) 已啟用
