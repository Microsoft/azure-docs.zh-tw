---
title: Azure 作業安全性檢查清單 | Microsoft Docs
description: 本文提供一組 Azure 作業安全性的檢查清單。
services: security
documentationcenter: na
author: unifycloud
manager: mbaldwin
editor: tomsh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: tomsh
ms.openlocfilehash: 481fcb869004a6fca83fdf19c32c41010057de32
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39008190"
---
# <a name="azure-operational-security-checklist"></a>Azure 作業安全性檢查清單
在 Azure 上部署應用程式很快速、輕鬆且符合成本效益。 在生產環境中部署雲端應用程式之前，針對您需要考慮的重要和建議作業安全性動作清單，取得檢查清單以協助評估應用程式是很有幫助的。

## <a name="introduction"></a>簡介

Azure 提供的基礎結構服務套件，可讓您用來部署應用程式。 Azure 作業安全性是指使用者可在 Microsoft Azure 中用來保護其資料、應用程式和其他資產的服務、控制及功能。

-   若要發揮雲端平台的最大效益，建議您運用 Azure 服務並遵循下列檢查清單。
-   在啟動之前投入時間和資源評估其應用程式之作業整備狀態的組織，會比未這麼做的組織擁有更高的滿意度。 執行這項工作時，檢查清單可用來作為一項重要機制，確保會以一致且完整的方式來評估應用程式。
-   作業評估的等級會根據組織的雲端成熟度等級以及應用程式的開發階段、可用性需求和資料敏感度需求而有所不同。

## <a name="checklist"></a>檢查清單

這份檢查清單旨在協助企業在 Azure 上部署複雜的企業應用程式之際，思考各種作業的安全性考量。 它也可以用來協助您建置組織的安全雲端移轉和作業策略。

|檢查清單類別| 說明|
| ------------ | -------- |
| [<br>安全性角色和存取控制](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide)|<ul><li>使用 [角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal)可提供用來指派權限給特定範圍的使用者、群組和應用程式的使用者特定。</li></ul> |
| [<br>資料收集和儲存](https://docs.microsoft.com/azure/storage/storage-security-guide)|<ul><li>透過[角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal)，使用管理平面安全性來保護您的儲存體帳戶。</li><li>資料平面安全性可使用[共用存取簽章 (SAS)](https://docs.microsoft.com/azure/storage/storage-dotnet-shared-access-signature-part-1) 和儲存的存取原則來保護您的資料存取。</li><li>使用傳輸等級加密 – 使用 HTTPS 和適用於 [Azure 檔案共用](https://docs.microsoft.com/azure/storage/storage-dotnet-how-to-use-files)的 [SMB (伺服器訊息區塊通訊協定) 3.0](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx) 所使用的加密。</li><li>使用[用戶端加密](https://docs.microsoft.com/azure/storage/storage-client-side-encryption)可保護您需要加密金鑰的唯一控制權時，傳送給儲存體帳戶的資料。 </li><li>使用[儲存體服務加密 (SSE)](https://docs.microsoft.com/azure/storage/storage-service-encryption)可將 Azure 儲存體中的資料自動加密，而使用 [Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)可將 OS 和資料磁碟的虛擬機器磁碟檔案進行加密。</li><li>使用 Azure [儲存體分析](https://docs.microsoft.com/rest/api/storageservices/storage-analytics)可監視授權類型；如同使用 Blob 儲存體，您可以看到使用者是否已使用共用存取簽章還是儲存體帳戶金鑰。</li><li>使用[跨原始資源共用 (CORS)](https://docs.microsoft.com/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services) 可存取來自不同網域的儲存區資源。</li></ul> |
|[<br>安全性原則和建議](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide)|<ul><li>使用 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-install-endpoint-protection)來部署端點解決方案。</li><li>新增 [Web 應用程式防火牆 (WAF)](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-overview) 來保護 Web 應用程式。</li><li>  使用由 Microsoft 合作夥伴提供的[新一代防火牆 (NGFW)](https://docs.microsoft.com/azure/security-center/security-center-add-next-generation-firewall)，以提升您的安全防護。 </li><li>套用 Azure 訂用帳戶的安全性連絡人詳細資料；當 [Microsoft 安全性回應中心 (MSRC)](https://technet.microsoft.com/security/dn528958.aspx) 發現有非法或未經授權的一方存取您的客戶資料時，就會連絡您。</li></ul> |
| [<br>身分識別與存取管理](https://docs.microsoft.com/azure/security/azure-security-identity-management-best-practices)|<ul><li>[使用 Azure AD 同步處理內部部署目錄與雲端目錄](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect)。</li><li>使用[單一登入](https://azure.microsoft.com/resources/videos/overview-of-single-sign-on/)可讓使用者以其在 Azure AD 中的組織帳戶作為基礎，存取其 SaaS 應用程式。</li><li>使用[密碼重設登錄活動](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-get-insights)報告可監視進行註冊的使用者。</li><li>對使用者啟用 [Multi-Factor Authentication (MFA)](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication)。</li><li>開發人員可以針對應用程式使用安全的身分識別功能，例如 [Microsoft 安全性開發週期 (SDL)](https://www.microsoft.com/download/details.aspx?id=12379)。</li><li>主動監視可疑的活動，方法是使用 Azure AD Premium 異常報告和 [Azure AD Identity Protection 功能](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection)。</li></ul> |
|[<br>持續安全性監視](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide)|<ul><li>使用惡意程式碼評定解決方案 [Log Analytics](https://docs.microsoft.com/azure/log-analytics/log-analytics-overview) 來回報基礎結構中的惡意程式碼保護狀態。</li><li>使用[更新評量](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-update-management)判斷潛在安全性問題的整體風險，以及這些更新是否或如何適用於您的環境。</li><li>[身分識別和存取](https://docs.microsoft.com/azure/operations-management-suite/oms-security-monitoring-resources)提供您的使用者概觀 </li><ul><li>使用者身分識別狀態、</li><li>登入之前要嘗試的失敗次數、</li><li>    這些嘗試期間所使用的使用者帳戶、已鎖定的帳戶</li> <li>密碼已變更或重設的帳戶 </li><li>目前已登入的帳戶數目。</li></ul></ul> |
| [<br>Azure 資訊安全中心的偵測功能](https://docs.microsoft.com/azure/security-center/security-center-detection-capabilities)|<ul><li>使用[偵測功能](https://docs.microsoft.com/azure/security-center/security-center-detection-capabilities)，可識別目標為 Microsoft Azure 資源的作用中威脅。</li><li>使用[整合性威脅情報](https://blogs.msdn.microsoft.com/azuresecurity/2016/12/19/get-threat-intelligence-reports-with-azure-security-center/)，可利用 Microsoft 產品和服務、[Microsoft 數位犯罪防治中心 (DCU)](https://www.microsoft.com/en-us/trustcenter/security/cybercrime)、[Microsoft Security Response Center (MSRC)](https://docs.microsoft.com/azure/security/azure-security-response-center) 以及外部摘要的全域威脅情報，尋找已知的不良執行者。</li><li>使用[行為分析](https://blogs.technet.microsoft.com/enterprisemobility/2016/06/30/ata-behavior-analysis-monitoring/)，可套用已知模式來探索惡意行為。 </li><li>使用[異常偵測](https://msdn.microsoft.com/library/azure/dn913096.aspx)，可使用統計分析來建置歷程基準。</li></ul> |
| [<br>開發人員作業 (DevOps)](https://docs.microsoft.com/azure/architecture/checklist/dev-ops)|<ul><li>[基礎結構即程式碼 (IaC)](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/)是一種做法，可用來將建立及卸除網路和虛擬機器加以自動化並驗證，從而協助提供安全且穩定的應用程式裝載平台。</li><li>[持續整合及部署](https://www.visualstudio.com/docs/build/overview)會驅動進行中的合併和程式碼測試，以便及早找出缺失。 </li><li>[Release Management](https://msdn.microsoft.com/library/vs/alm/release/overview) 可透過您管線的每個階段管理自動化的部署。</li><li>執行中應用程式的[應用程式效能監視](https://azure.microsoft.com/documentation/articles/app-insights-start-monitoring-app-health-usage/)包括應用程式健康情況的生產環境以及客戶的使用方式，能協助組織產生假設並快速驗證或反駁策略。</li><li>使用[負載測試和自動縮放](https://www.visualstudio.com/docs/test/performance-testing/getting-started/getting-started-with-performance-testing)，就可以在我們的應用程式中找出效能問題，從而改善部署品質，並確保我們的應用程式一律保持最新狀態，或使用符合商務需求。</li></ul> |


## <a name="conclusion"></a>結論
許多組織已成功在 Azure 上部署和操作其雲端應用程式。 所提供的檢查清單強調顯示數個重要的檢查清單，並協助您對於成功部署更有把握且作業不會失敗。 強烈建議您在 Azure 上進行現有和新的應用程式部署時，使用這些作業和策略考量。

## <a name="next-steps"></a>後續步驟
在本文件中，已向您介紹 OMS 安全性和稽核解決方案。 若要深入了解 OMS 安全性，請參閱下列文章：

- [Operations Management Suite (OMS) 概觀](https://docs.microsoft.com/azure/operations-management-suite/operations-management-suite-overview)。
- [設計與作業安全性](https://www.microsoft.com/trustcenter/security/designopsecurity)。
- [Azure 資訊安全中心規劃和操作](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide)。
