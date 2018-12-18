---
title: Azure Resource Manager 資源提供者作業 | Microsoft Docs
description: 列出 Microsoft Azure Resource Manager 資源提供者可用的作業
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/27/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: d9cf46a1fd28985d9ca98d173bbef17380b7de21
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49354388"
---
# <a name="azure-resource-manager-resource-provider-operations"></a>Azure Resource Manager 資源提供者作業

本文會列出每個 Azure Resource Manager 資源提供者可用的作業。 這些作業可用在[自訂角色](custom-roles.md)中，以對 Azure 中的資源提供細微的[角色型存取控制 (RBAC)](overview.md)。 作業字串的格式如下：`{Company}.{ProviderName}/{resourceType}/{action}`

資源提供者作業會不斷發展。 若要取得最新的作業，請使用 [Get-AzureRmProviderOperation](/powershell/module/azurerm.resources/get-azurermprovideroperation) 或 [az provider operation list](/cli/azure/provider/operation#az-provider-operation-list)。

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.AAD/domainServices/delete | 刪除網域服務 |
> | 動作 | Microsoft.AAD/domainServices/oucontainer/delete | 刪除 Ou 容器 |
> | 動作 | Microsoft.AAD/domainServices/oucontainer/read | 讀取 Ou 容器 |
> | 動作 | Microsoft.AAD/domainServices/oucontainer/write | 寫入 Ou 容器 |
> | 動作 | Microsoft.AAD/domainServices/read | 讀取網域服務 |
> | 動作 | Microsoft.AAD/domainServices/write | 寫入網域服務 |
> | 動作 | Microsoft.AAD/locations/operationresults/read |  |
> | 動作 | Microsoft.AAD/Operations/read |  |
> | 動作 | Microsoft.AAD/register/action | 註冊網域服務 |

## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | microsoft.aadiam/diagnosticsettings/delete | 正在刪除診斷設定 |
> | 動作 | microsoft.aadiam/diagnosticsettings/read | 正在讀取診斷設定 |
> | 動作 | microsoft.aadiam/diagnosticsettings/write | 正在寫入診斷設定 |
> | 動作 | microsoft.aadiam/diagnosticsettingscategories/read | 正在讀取診斷設定類別 |
> | 動作 | microsoft.aadiam/tenants/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | microsoft.aadiam/tenants/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | microsoft.aadiam/tenants/providers/Microsoft.Insights/logDefinitions/read | 取得租用戶的可用記錄 |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Addons/operations/read | 取得支援的 RP 作業。 |
> | 動作 | Microsoft.Addons/register/action | 向 Microsoft.Addons 註冊指定的訂用帳戶 |
> | 動作 | Microsoft.Addons/supportProviders/listsupportplaninfo/action | 列出指定訂用帳戶目前的支援方案資訊。 |
> | 動作 | Microsoft.Addons/supportProviders/supportPlanTypes/delete | 移除指定的 Canonical 支援方案 |
> | 動作 | Microsoft.Addons/supportProviders/supportPlanTypes/read | 取得指定的 Canonical 支援方案狀態。 |
> | 動作 | Microsoft.Addons/supportProviders/supportPlanTypes/write | 新增指定的 Canonical 支援方案類型。 |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/action | 為租用戶建立新的樹系。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/addomainservicemembers/read | 取得指定服務名稱的所有伺服器。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/alerts/read | 取得樹系的警示詳細資料，例如 alertid、警示引發日期、上次偵測到的警示、警示說明、上次更新日期、警示層級、警示狀態、警示疑難排解連結等等。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/configuration/read | 取得樹系的服務組態。 範例 - 樹系名稱、功能等級、網域命名主機 FSMO 角色、架構主機 FSMO 角色等等。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/delete | 刪除服務及其伺服器以及健康情況資料。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/dimensions/read | 取得樹系的網域和網站詳細資料。 範例 - 健康情況狀態、作用中警示、已解決的警示、屬性 (例如，網域功能等級、樹系、基礎結構主機、PDC、RID 主機等等)。  |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/features/userpreference/read | 取得樹系的使用者喜好設定。<br>範例 - MetricCounterName，例如 ldapsuccessfulbinds、ntlmauthentications、kerberosauthentications、addsinsightsagentprivatebytes、ldapsearches。<br>UI 圖表等項目的設定。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/forestsummary/read | 取得指定樹系的樹系摘要，例如樹系名稱、此樹系下的網域數目、網站數目和網站詳細資料等。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/metricmetadata/read | 取得指定服務的支援計量清單。<br>例如，ADFS 服務的外部網路帳戶鎖定、失敗要求總數、未處理的權杖要求數 (Proxy)、每秒權杖要求數等等。<br>ADDomainService 的每秒 NTLM 驗證數、每秒 LDAP 成功繫結數、LDAP 繫結時間、LDAP 作用中執行緒、每秒 Kerberos 驗證數、ATQ 執行緒總數等。<br>ADSync 服務的執行設定檔延遲、所建立的 TCP 連線數、Insights 代理程式私用位元組數、將統計資料匯出至 Azure AD。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/metrics/groups/read | 在指定了服務時，此 API 會取得計量資訊。<br>例如，此 API 可用來取得與下列項目相關的資訊：ADFederation 服務的外部網路帳戶鎖定、失敗要求總數、未處理的權杖要求數 (Proxy)、每秒權杖要求數等等。<br>ADDomain 服務的每秒 NTLM 驗證數、每秒 LDAP 成功繫結數、LDAP 繫結時間、LDAP 作用中執行緒、每秒 Kerberos 驗證數、ATQ 執行緒總數等。<br>Sync 服務的執行設定檔延遲、所建立的 TCP 連線數、Insights 代理程式私用位元組數、將統計資料匯出至 Azure AD。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/premiumcheck/read | 此 API 會取得進階租用戶所有已上架 ADDomainServices 的清單。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/read | 取得指定服務名稱的服務詳細資料。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/replicationdetails/read | 取得指定服務名稱的所有伺服器複寫詳細資料。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/replicationstatus/read | 取得網域控制站數目和其存在的任何複寫錯誤。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/replicationsummary/read | 取得完整的網域控制站清單，以及指定樹系的複寫詳細資料。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/servicemembers/action | 對服務新增伺服器執行個體。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/servicemembers/credentials/read | 在 ADDomainService 的伺服器註冊期間，會呼叫此 API 來取得用於將新伺服器上架的認證。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/servicemembers/delete | 刪除指定服務和租用戶的伺服器。 |
> | 動作 | Microsoft.ADHybridHealthService/addsservices/write | 建立或更新租用戶的 ADDomainService 執行個體。 |
> | 動作 | Microsoft.ADHybridHealthService/configuration/action | 更新租用戶組態。 |
> | 動作 | Microsoft.ADHybridHealthService/configuration/read | 讀取租用戶組態。 |
> | 動作 | Microsoft.ADHybridHealthService/configuration/write | 建立租用戶組態。 |
> | 動作 | Microsoft.ADHybridHealthService/logs/contents/read | 取得 Blob 中所儲存的代理程式安裝和註冊記錄內容。 |
> | 動作 | Microsoft.ADHybridHealthService/logs/read | 取得租用戶的代理程式安裝和註冊記錄。 |
> | 動作 | Microsoft.ADHybridHealthService/operations/read | 取得系統所支援作業的清單。 |
> | 動作 | Microsoft.ADHybridHealthService/register/action | 註冊 ADHybrid 健康情況服務資源提供者，並允許建立 ADHybrid 健康情況服務資源。 |
> | 動作 | Microsoft.ADHybridHealthService/reports/availabledeployments/read | 取得可用區域清單，以供 DevOps 用來支援客戶事件。 |
> | 動作 | Microsoft.ADHybridHealthService/reports/badpassword/read | 取得 Active Directory Federation Service 中所有使用者的錯誤密碼嘗試清單。 |
> | 動作 | Microsoft.ADHybridHealthService/reports/badpassworduseridipfrequency/read | 取得 Blob SAS URI，其包含新近排入佇列的報告作業狀態和最終結果，執行該作業是為了了解指定租用戶每個 UserId、每個 IPAddress、每天發生的錯誤使用者名稱/密碼嘗試頻率。 |
> | 動作 | Microsoft.ADHybridHealthService/reports/consentedtodevopstenants/read | 取得 DevOps 同意的租用戶清單。 通常用於客戶支援。 |
> | 動作 | Microsoft.ADHybridHealthService/reports/isdevops/read | 取得可指出租用戶是否已獲得 DevOps 同意的值。 |
> | 動作 | Microsoft.ADHybridHealthService/reports/selectdevopstenant/read | 更新所選 Dev Ops 租用戶的 userid (objectid)。 |
> | 動作 | Microsoft.ADHybridHealthService/reports/selecteddeployment/read | 取得指定租用戶所選取的部署。 |
> | 動作 | Microsoft.ADHybridHealthService/reports/tenantassigneddeployment/read | 在指定了租用戶識別碼時，會取得租用戶儲存位置。 |
> | 動作 | Microsoft.ADHybridHealthService/reports/updateselecteddeployment/read | 取得會存取資料的地理位置。 |
> | 動作 | Microsoft.ADHybridHealthService/services/action | 更新租用戶中的服務執行個體。 |
> | 動作 | Microsoft.ADHybridHealthService/services/alerts/read | 讀取服務的警示。 |
> | 動作 | Microsoft.ADHybridHealthService/services/alerts/read | 讀取服務的警示。 |
> | 動作 | Microsoft.ADHybridHealthService/services/checkservicefeatureavailibility/read | 在指定了功能名稱時，會確認服務是否已備妥一切所需而可使用該功能。 |
> | 動作 | Microsoft.ADHybridHealthService/services/delete | 刪除租用戶中的服務執行個體。 |
> | 動作 | Microsoft.ADHybridHealthService/services/exporterrors/read | 取得指定 Sync 服務的匯出錯誤。 |
> | 動作 | Microsoft.ADHybridHealthService/services/exportstatus/read | 取得指定服務的匯出狀態。 |
> | 動作 | Microsoft.ADHybridHealthService/services/feedbacktype/feedback/read | 取得指定服務和伺服器的警示意見反應。 |
> | 動作 | Microsoft.ADHybridHealthService/services/metricmetadata/read | 取得指定服務的支援計量清單。<br>例如，ADFS 服務的外部網路帳戶鎖定、失敗要求總數、未處理的權杖要求數 (Proxy)、每秒權杖要求數等等。<br>ADDomainService 的每秒 NTLM 驗證數、每秒 LDAP 成功繫結數、LDAP 繫結時間、LDAP 作用中執行緒、每秒 Kerberos 驗證數、ATQ 執行緒總數等。<br>ADSync 服務的執行設定檔延遲、所建立的 TCP 連線數、Insights 代理程式私用位元組數、將統計資料匯出至 Azure AD。 |
> | 動作 | Microsoft.ADHybridHealthService/services/metrics/groups/average/read | 在指定了服務時，此 API 會取得指定服務的計量平均值。<br>例如，此 API 可用來取得與下列項目相關的資訊：ADFederation 服務的外部網路帳戶鎖定、失敗要求總數、未處理的權杖要求數 (Proxy)、每秒權杖要求數等等。<br>ADDomain 服務的每秒 NTLM 驗證數、每秒 LDAP 成功繫結數、LDAP 繫結時間、LDAP 作用中執行緒、每秒 Kerberos 驗證數、ATQ 執行緒總數等。<br>Sync 服務的執行設定檔延遲、所建立的 TCP 連線數、Insights 代理程式私用位元組數、將統計資料匯出至 Azure AD。 |
> | 動作 | Microsoft.ADHybridHealthService/services/metrics/groups/read | 在指定了服務時，此 API 會取得計量資訊。<br>例如，此 API 可用來取得與下列項目相關的資訊：ADFederation 服務的外部網路帳戶鎖定、失敗要求總數、未處理的權杖要求數 (Proxy)、每秒權杖要求數等等。<br>ADDomain 服務的每秒 NTLM 驗證數、每秒 LDAP 成功繫結數、LDAP 繫結時間、LDAP 作用中執行緒、每秒 Kerberos 驗證數、ATQ 執行緒總數等。<br>Sync 服務的執行設定檔延遲、所建立的 TCP 連線數、Insights 代理程式私用位元組數、將統計資料匯出至 Azure AD。 |
> | 動作 | Microsoft.ADHybridHealthService/services/metrics/groups/sum/read | 在指定了服務時，此 API 會取得指定服務計量的彙總檢視。<br>例如，此 API 可用來取得與下列項目相關的資訊：ADFederation 服務的外部網路帳戶鎖定、失敗要求總數、未處理的權杖要求數 (Proxy)、每秒權杖要求數等等。<br>ADDomain 服務的每秒 NTLM 驗證數、每秒 LDAP 成功繫結數、LDAP 繫結時間、LDAP 作用中執行緒、每秒 Kerberos 驗證數、ATQ 執行緒總數等。<br>Sync 服務的執行設定檔延遲、所建立的 TCP 連線數、Insights 代理程式私用位元組數、將統計資料匯出至 Azure AD。 |
> | 動作 | Microsoft.ADHybridHealthService/services/monitoringconfiguration/write | 新增或更新服務的監視組態。 |
> | 動作 | Microsoft.ADHybridHealthService/services/monitoringconfigurations/read | 取得指定服務的監視組態。 |
> | 動作 | Microsoft.ADHybridHealthService/services/monitoringconfigurations/write | 新增或更新服務的監視組態。 |
> | 動作 | Microsoft.ADHybridHealthService/services/premiumcheck/read | 此 API 會取得進階租用戶所有已上架服務的清單。 |
> | 動作 | Microsoft.ADHybridHealthService/services/read | 讀取租用戶中的服務執行個體。 |
> | 動作 | Microsoft.ADHybridHealthService/services/reports/details/read | 取得過去 7 天排行前 50 名有不正確密碼錯誤的使用者報告 |
> | 動作 | Microsoft.ADHybridHealthService/services/servicemembers/action | 在服務中建立伺服器執行個體。 |
> | 動作 | Microsoft.ADHybridHealthService/services/servicemembers/alerts/read | 讀取伺服器的警示。 |
> | 動作 | Microsoft.ADHybridHealthService/services/servicemembers/credentials/read | 在伺服器註冊期間，會呼叫此 API 來取得用於將新伺服器上架的認證。 |
> | 動作 | Microsoft.ADHybridHealthService/services/servicemembers/datafreshness/read | 針對指定的伺服器，此 API 會取得伺服器所要上傳的資料類型清單，以及每個上傳的最新時間。 |
> | 動作 | Microsoft.ADHybridHealthService/services/servicemembers/delete | 刪除服務中的伺服器執行個體。 |
> | 動作 | Microsoft.ADHybridHealthService/services/servicemembers/exportstatus/read | 取得指定 Sync 服務的 Sync 匯出錯誤。 |
> | 動作 | Microsoft.ADHybridHealthService/services/servicemembers/metrics/groups/read | 在指定了服務時，此 API 會取得計量資訊。<br>例如，此 API 可用來取得與下列項目相關的資訊：ADFederation 服務的外部網路帳戶鎖定、失敗要求總數、未處理的權杖要求數 (Proxy)、每秒權杖要求數等等。<br>ADDomain 服務的每秒 NTLM 驗證數、每秒 LDAP 成功繫結數、LDAP 繫結時間、LDAP 作用中執行緒、每秒 Kerberos 驗證數、ATQ 執行緒總數等。<br>Sync 服務的執行設定檔延遲、所建立的 TCP 連線數、Insights 代理程式私用位元組數、將統計資料匯出至 Azure AD。 |
> | 動作 | Microsoft.ADHybridHealthService/services/servicemembers/read | 讀取服務中的伺服器執行個體。 |
> | 動作 | Microsoft.ADHybridHealthService/services/servicemembers/serviceconfiguration/read | 取得指定租用戶的服務組態。 |
> | 動作 | Microsoft.ADHybridHealthService/services/tenantwhitelisting/read | 取得指定租用戶的功能允許清單狀態。 |
> | 動作 | Microsoft.ADHybridHealthService/services/write | 建立租用戶中的服務執行個體。 |
> | 動作 | Microsoft.ADHybridHealthService/unregister/action | 將 ADHybrid 健康情況服務資源提供者的訂用帳戶取消註冊。 |

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Advisor/configurations/read | 取得組態 |
> | 動作 | Microsoft.Advisor/configurations/write | 建立/更新設定 |
> | 動作 | Microsoft.Advisor/generateRecommendations/action | 產生建議 |
> | 動作 | Microsoft.Advisor/generateRecommendations/read | 取得產生建議狀態 |
> | 動作 | Microsoft.Advisor/operations/read | 取得 Microsoft Advisor 的作業 |
> | 動作 | Microsoft.Advisor/recommendations/available/action | 新的建議可從 Microsoft Advisor 中取得 |
> | 動作 | Microsoft.Advisor/recommendations/read | 讀取建議 |
> | 動作 | Microsoft.Advisor/recommendations/suppressions/delete | 刪除隱藏項目 |
> | 動作 | Microsoft.Advisor/recommendations/suppressions/read | 取得隱藏項目 |
> | 動作 | Microsoft.Advisor/recommendations/suppressions/write | 建立/更新隱藏項目 |
> | 動作 | Microsoft.Advisor/register/action | 針對 Microsoft Advisor 註冊訂用帳戶 |
> | 動作 | Microsoft.Advisor/suppressions/delete | 刪除隱藏項目 |
> | 動作 | Microsoft.Advisor/suppressions/read | 取得隱藏項目 |
> | 動作 | Microsoft.Advisor/suppressions/write | 建立/更新隱藏項目 |
> | 動作 | Microsoft.Advisor/unregister/action | 為 Microsoft Advisor 取消註冊訂用帳戶 |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.AlertsManagement/alerts/changestate/action | 變更警示的狀態。 |
> | 動作 | Microsoft.AlertsManagement/alerts/read | 取得輸入篩選的所有警示。 |
> | 動作 | Microsoft.AlertsManagement/alertsSummary/read | 取得警示的摘要 |
> | 動作 | Microsoft.AlertsManagement/Operations/read | 讀取已提供的作業 |
> | 動作 | Microsoft.AlertsManagement/smartGroups/changestate/action | 變更智慧群組的狀態 |
> | 動作 | Microsoft.AlertsManagement/smartGroups/read | 取得輸入篩選的所有智慧群組 |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.AnalysisServices/locations/checkNameAvailability/action | 確認給定的 Analysis Server 名稱有效，且並非使用中。 |
> | 動作 | Microsoft.AnalysisServices/locations/operationresults/read | 擷取指定作業結果的資訊。 |
> | 動作 | Microsoft.AnalysisServices/locations/operationstatuses/read | 擷取指定作業狀態的資訊。 |
> | 動作 | Microsoft.AnalysisServices/operations/read | 擷取作業的資訊 |
> | 動作 | Microsoft.AnalysisServices/register/action | 註冊 Analysis Services 資源提供者。 |
> | 動作 | Microsoft.AnalysisServices/servers/delete | 刪除 Analysis Server。 |
> | 動作 | Microsoft.AnalysisServices/servers/listGatewayStatus/action | 列出與伺服器相關聯的閘道狀態。 |
> | 動作 | Microsoft.AnalysisServices/servers/providers/Microsoft.Insights/diagnosticSettings/read | 取得 Analysis Server 的診斷設定 |
> | 動作 | Microsoft.AnalysisServices/servers/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新 Analysis Server 的診斷設定 |
> | 動作 | Microsoft.AnalysisServices/servers/providers/Microsoft.Insights/logDefinitions/read | 取得伺服器的可用記錄 |
> | 動作 | Microsoft.AnalysisServices/servers/providers/Microsoft.Insights/metricDefinitions/read | 取得 Analysis Server 的可用計量 |
> | 動作 | Microsoft.AnalysisServices/servers/read | 擷取所指定 Analysis Server 的資訊。 |
> | 動作 | Microsoft.AnalysisServices/servers/resume/action | 繼續 Analysis Server。 |
> | 動作 | Microsoft.AnalysisServices/servers/skus/read | 擷取伺服器的可用 SKU 資訊 |
> | 動作 | Microsoft.AnalysisServices/servers/suspend/action | 暫止 Analysis Server。 |
> | 動作 | Microsoft.AnalysisServices/servers/write | 建立或更新所指定的 Analysis Server。 |
> | 動作 | Microsoft.AnalysisServices/skus/read | 擷取 SKU 的資訊 |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ApiManagement/checkNameAvailability/read | 檢查所提供的服務名稱是否可用 |
> | 動作 | Microsoft.ApiManagement/operations/read | 讀取可供 Microsoft.ApiManagement 資源使用的所有 API 作業 |
> | 動作 | Microsoft.ApiManagement/register/action | 針對 Microsoft.ApiManagement 資源提供者註冊訂用帳戶 |
> | 動作 | Microsoft.ApiManagement/reports/read | 取得依時間週期、地理區域、開發人員、產品、API、作業、訂用帳戶和依需求彙總的報告。 |
> | 動作 | Microsoft.ApiManagement/service/apis/delete | 移除現有 API |
> | 動作 | Microsoft.ApiManagement/service/apis/diagnostics/delete | 移除現有診斷 |
> | 動作 | Microsoft.ApiManagement/service/apis/diagnostics/read | 取得診斷清單，或取得診斷的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/diagnostics/write | 新增診斷，或更新現有的診斷詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/issues/attachments/delete | 移除現有附件 |
> | 動作 | Microsoft.ApiManagement/service/apis/issues/attachments/read | 取得問題附件或取得 API 管理問題附件的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/issues/attachments/write | 新增 API 問題附件 |
> | 動作 | Microsoft.ApiManagement/service/apis/issues/comments/delete | 移除現有註解 |
> | 動作 | Microsoft.ApiManagement/service/apis/issues/comments/read | 取得問題註解或取得 API 管理問題註解的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/issues/comments/write | 新增 API 問題註解 |
> | 動作 | Microsoft.ApiManagement/service/apis/issues/delete | 移除現有問題 |
> | 動作 | Microsoft.ApiManagement/service/apis/issues/read | 取得 API 相關問題或取得 API 管理問題的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/issues/write | 新增 API 問題或更新 API 問題 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/delete | 移除現有的 API 作業 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/policies/delete | 從 API 作業原則中移除原則設定 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/policies/read | 取得 API 作業的原則，或取得 API 作業的原則設定詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/policies/write | 設定 API 作業的原則設定詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/policy/delete | 從作業中移除原則組態 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/policy/read | 取得作業的原則組態詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/policy/write | 設定作業的原則組態詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/read | 取得現有 API 作業的清單，或取得 API 作業的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/tags/delete | 刪除現有標記與現有作業的關聯 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/tags/read | 取得與作業相關聯的標記，或取得標記詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/tags/write | 將現有標記關聯至現有作業 |
> | 動作 | Microsoft.ApiManagement/service/apis/operations/write | 建立新的 API 作業，或更新現有的 API 作業 |
> | 動作 | Microsoft.ApiManagement/service/apis/operationsByTags/read | 取得作業/標記關聯的清單 |
> | 動作 | Microsoft.ApiManagement/service/apis/policies/delete | 從 API 原則中移除原則設定 |
> | 動作 | Microsoft.ApiManagement/service/apis/policies/read | 取得 API 的原則，或取得 API 的原則設定詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/policies/write | 設定 API 的原則組態詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/policy/delete | 從 API 中移除原則組態 |
> | 動作 | Microsoft.ApiManagement/service/apis/policy/read | 取得 API 的原則組態詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/policy/write | 設定 API 的原則組態詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/products/read | 取得 API 所屬的所有產品 |
> | 動作 | Microsoft.ApiManagement/service/apis/read | 取得所有已註冊之 API 的清單，或取得 API 的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/releases/delete | 移除 API 的所有版本，或移除 API 版本 |
> | 動作 | Microsoft.ApiManagement/service/apis/releases/read | 取得 API 的版本，或取得 API 版本的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/releases/write | 建立新的 API，或更新現有 API 版本 |
> | 動作 | Microsoft.ApiManagement/service/apis/revisions/delete | 移除 API 的所有修訂 |
> | 動作 | Microsoft.ApiManagement/service/apis/revisions/read | 取得屬於 API 的修訂 |
> | 動作 | Microsoft.ApiManagement/service/apis/schemas/delete | 移除現有結構描述 |
> | 動作 | Microsoft.ApiManagement/service/apis/schemas/document/read | 取得說明結構描述的文件 |
> | 動作 | Microsoft.ApiManagement/service/apis/schemas/document/write | 更新說明結構描述的文件 |
> | 動作 | Microsoft.ApiManagement/service/apis/schemas/read | 取得指定 API 的所有結構描述，或取得 API 所使用的結構描述 |
> | 動作 | Microsoft.ApiManagement/service/apis/schemas/write | 設定 API 所使用的結構描述 |
> | 動作 | Microsoft.ApiManagement/service/apis/tagDescriptions/delete | 從 API 中移除標記描述 |
> | 動作 | Microsoft.ApiManagement/service/apis/tagDescriptions/read | 取得 API 範圍內的多個標記描述，或取得 API 範圍內的單一標記描述 |
> | 動作 | Microsoft.ApiManagement/service/apis/tagDescriptions/write | 建立/變更 API 範圍內的標記描述 |
> | 動作 | Microsoft.ApiManagement/service/apis/tags/delete | 移除現有的 API/標記關聯 |
> | 動作 | Microsoft.ApiManagement/service/apis/tags/read | 取得 API 的所有 API/標記關聯，或取得 API/標記關聯的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apis/tags/write | 新增 API/標記關聯 |
> | 動作 | Microsoft.ApiManagement/service/apis/write | 建立新的 API，或更新現有 API 的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/apisByTags/read | 取得 API/標記關聯的清單 |
> | 動作 | Microsoft.ApiManagement/service/api-version-sets/delete | 移除現有的 VersionSet |
> | 動作 | Microsoft.ApiManagement/service/api-version-sets/read | 取得版本群組實體的清單，或取得 VersionSet 的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/api-version-sets/versions/read | 取得版本實體的清單 |
> | 動作 | Microsoft.ApiManagement/service/api-version-sets/write | 建立新的 VersionSet，或更新現有 VersionSet 詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/applynetworkconfigurationupdates/action | 更新虛擬網路中執行的 Microsoft.ApiManagement 資源，以挑選更新後的網路設定。 |
> | 動作 | Microsoft.ApiManagement/service/authorizationServers/delete | 移除現有的授權伺服器 |
> | 動作 | Microsoft.ApiManagement/service/authorizationServers/read | 取得授權伺服器的清單，或取得授權伺服器的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/authorizationServers/write | 建立新的授權伺服器，或更新現有授權伺服器的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/backends/delete | 移除現有後端 |
> | 動作 | Microsoft.ApiManagement/service/backends/read | 取得後端清單，或取得後端的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/backends/reconnect/action | 建立重新連線要求 |
> | 動作 | Microsoft.ApiManagement/service/backends/write | 新增後端，或更新現有的後端詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/backup/action | 將 API 管理服務備份到使用者所提供之儲存體帳戶中的指定容器 |
> | 動作 | Microsoft.ApiManagement/service/certificates/delete | 移除現有憑證 |
> | 動作 | Microsoft.ApiManagement/service/certificates/read | 取得憑證清單，或取得憑證的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/certificates/write | 新增憑證 |
> | 動作 | Microsoft.ApiManagement/service/delete | 刪除 API 管理服務執行個體 |
> | 動作 | Microsoft.ApiManagement/service/diagnostics/delete | 移除現有診斷 |
> | 動作 | Microsoft.ApiManagement/service/diagnostics/read | 取得診斷清單，或取得診斷的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/diagnostics/write | 新增診斷，或更新現有的診斷詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/getssotoken/action | 取得 SSO 權杖，以供用來登入 API 管理服務舊版入口網站 (登入身分為系統管理員) |
> | 動作 | Microsoft.ApiManagement/service/groups/delete | 移除現有群組 |
> | 動作 | Microsoft.ApiManagement/service/groups/read | 取得群組清單，或取得群組的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/groups/users/delete | 從現有群組移除現有使用者 |
> | 動作 | Microsoft.ApiManagement/service/groups/users/read | 取得群組使用者的清單 |
> | 動作 | Microsoft.ApiManagement/service/groups/users/write | 將現有使用者新增至現有群組 |
> | 動作 | Microsoft.ApiManagement/service/groups/write | 建立新的群組，或更新現有群組的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/identityProviders/delete | 移除現有的識別提供者 |
> | 動作 | Microsoft.ApiManagement/service/identityProviders/read | 取得識別提供者清單，或取得識別提供者的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/identityProviders/write | 建立新的識別提供者，或更新現有識別提供者的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/locations/networkstatus/read | 取得位置中服務相依資源的網路存取狀態。 |
> | 動作 | Microsoft.ApiManagement/service/loggers/delete | 移除現有的記錄器 |
> | 動作 | Microsoft.ApiManagement/service/loggers/read | 取得記錄器的清單，或取得記錄器的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/loggers/write | 新增記錄器，或更新現有的記錄器詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/managedeployments/action | 變更 SKU/單位、新增/移除 API 管理服務的區域部署 |
> | 動作 | Microsoft.ApiManagement/service/networkstatus/read | 取得服務相依資源的網路存取狀態。 |
> | 動作 | Microsoft.ApiManagement/service/notifications/action | 將通知傳送給指定的使用者 |
> | 動作 | Microsoft.ApiManagement/service/notifications/read | 取得所有的 API 管理發行者通知，或取得 API 管理發行者通知詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/notifications/recipientEmails/delete | 移除目前與通知相關聯的電子郵件 |
> | 動作 | Microsoft.ApiManagement/service/notifications/recipientEmails/read | 取得與 API 管理發行者通知相關聯的電子郵件收件者 |
> | 動作 | Microsoft.ApiManagement/service/notifications/recipientEmails/write | 建立通知的新電子郵件收件者 |
> | 動作 | Microsoft.ApiManagement/service/notifications/recipientUsers/delete | 移除與通知收件者相關聯的使用者 |
> | 動作 | Microsoft.ApiManagement/service/notifications/recipientUsers/read | 取得與通知相關聯的收件者使用者 |
> | 動作 | Microsoft.ApiManagement/service/notifications/recipientUsers/write | 將使用者新增至通知收件者 |
> | 動作 | Microsoft.ApiManagement/service/notifications/write | 建立或更新 API 管理發行者通知 |
> | 動作 | Microsoft.ApiManagement/service/openidConnectProviders/delete | 移除現有的 OpenID Connect 提供者 |
> | 動作 | Microsoft.ApiManagement/service/openidConnectProviders/read | 取得 OpenID Connect 提供者的清單，或取得 OpenID Connect 提供者的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/openidConnectProviders/write | 建立新的 OpenID Connect 提供者，或更新現有 OpenID Connect 提供者的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/operationresults/read | 取得長時間執行之作業的目前狀態 |
> | 動作 | Microsoft.ApiManagement/service/policies/delete | 從租用戶原則中移除原則設定 |
> | 動作 | Microsoft.ApiManagement/service/policies/read | 取得租用戶的原則，或取得租用戶的原則設定詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/policies/write | 設定租用戶的原則設定詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/policySnippets/read | 取得所有原則程式碼片段 |
> | 動作 | Microsoft.ApiManagement/service/portalsettings/read | 取得入口網站的註冊設定、取得入口網站的登入設定，或取得入口網站的委派設定 |
> | 動作 | Microsoft.ApiManagement/service/portalsettings/write | 更新註冊設定、更新登入設定，或更新委派設定 |
> | 動作 | Microsoft.ApiManagement/service/products/apis/delete | 從現有產品中移除現有 API |
> | 動作 | Microsoft.ApiManagement/service/products/apis/read | 取得新增至現有產品之 API 的清單 |
> | 動作 | Microsoft.ApiManagement/service/products/apis/write | 將現有 API 新增至現有產品 |
> | 動作 | Microsoft.ApiManagement/service/products/delete | 移除現有產品 |
> | 動作 | Microsoft.ApiManagement/service/products/groups/delete | 刪除現有開發人員群組與現有產品的關聯 |
> | 動作 | Microsoft.ApiManagement/service/products/groups/read | 取得與產品相關聯的開發人員群組清單 |
> | 動作 | Microsoft.ApiManagement/service/products/groups/write | 將現有開發人員群組與現有產品相關聯 |
> | 動作 | Microsoft.ApiManagement/service/products/policies/delete | 從產品原則中移除原則設定 |
> | 動作 | Microsoft.ApiManagement/service/products/policies/read | 取得產品的原則，或取得產品的原則設定詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/products/policies/write | 設定產品的原則設定詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/products/policy/delete | 從現有產品中移除原則組態 |
> | 動作 | Microsoft.ApiManagement/service/products/policy/read | 取得現有產品的原則組態 |
> | 動作 | Microsoft.ApiManagement/service/products/policy/write | 設定現有產品的原則組態 |
> | 動作 | Microsoft.ApiManagement/service/products/read | 取得產品清單，或取得產品的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/products/subscriptions/read | 取得產品訂用帳戶的清單 |
> | 動作 | Microsoft.ApiManagement/service/products/tags/delete | 刪除現有標記與現有產品的關聯 |
> | 動作 | Microsoft.ApiManagement/service/products/tags/read | 取得與產品相關聯的標記，或取得標記詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/products/tags/write | 將現有標記關聯至現有產品 |
> | 動作 | Microsoft.ApiManagement/service/products/write | 建立新的產品，或更新現有產品的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/productsByTags/read | 取得產品/標記關聯的清單 |
> | 動作 | Microsoft.ApiManagement/service/properties/delete | 移除現有屬性 |
> | 動作 | Microsoft.ApiManagement/service/properties/read | 取得所有屬性的清單，或取得指定之屬性的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/properties/write | 建立新的屬性，或更新指定之屬性的值 |
> | 動作 | Microsoft.ApiManagement/service/providers/Microsoft.Insights/diagnosticSettings/read | 取得 API 管理服務的診斷設定 |
> | 動作 | Microsoft.ApiManagement/service/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新 API 管理服務的診斷設定 |
> | 動作 | Microsoft.ApiManagement/service/providers/Microsoft.Insights/logDefinitions/read | 取得 API 管理服務的可用記錄 |
> | 動作 | Microsoft.ApiManagement/service/providers/Microsoft.Insights/metricDefinitions/read | 取得 API 管理服務的可用計量 |
> | 動作 | Microsoft.ApiManagement/service/quotas/periods/read | 取得期間的配額計數器值 |
> | 動作 | Microsoft.ApiManagement/service/quotas/periods/write | 設定配額計數器目前的值 |
> | 動作 | Microsoft.ApiManagement/service/quotas/read | 取得配額的值 |
> | 動作 | Microsoft.ApiManagement/service/quotas/write | 設定配額計數器目前的值 |
> | 動作 | Microsoft.ApiManagement/service/read | 讀取 API 管理服務執行個體的中繼資料 |
> | 動作 | Microsoft.ApiManagement/service/reports/read | 取得依時間週期彙總的報告、取得依地理區域彙總的報告，或取得依開發人員彙總的報告。<br>或者，取得依產品彙總的報告。<br>或者，取得依 API 彙總的報告、取得依作業彙總的報告，或取得依訂用帳戶彙總的報告。<br>或者，取得報告資料的要求 |
> | 動作 | Microsoft.ApiManagement/service/restore/action | 從使用者所提供之儲存體帳戶中的指定容器來還原 API 管理服務 |
> | 動作 | Microsoft.ApiManagement/service/subscriptions/delete | 刪除訂用帳戶。 這項作業可用來刪除訂用帳戶 |
> | 動作 | Microsoft.ApiManagement/service/subscriptions/read | 取得產品訂用帳戶清單，或取得產品訂用帳戶的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/subscriptions/regeneratePrimaryKey/action | 重新產生訂用帳戶的主要金鑰 |
> | 動作 | Microsoft.ApiManagement/service/subscriptions/regenerateSecondaryKey/action | 重新產生訂用帳戶的次要金鑰 |
> | 動作 | Microsoft.ApiManagement/service/subscriptions/write | 讓現有使用者訂閱現有產品，或更新現有的訂用帳戶詳細資料。 這項作業可用來續訂訂用帳戶 |
> | 動作 | Microsoft.ApiManagement/service/tagResources/read | 取得具有相關聯資源的標記清單 |
> | 動作 | Microsoft.ApiManagement/service/tags/delete | 移除現有標記 |
> | 動作 | Microsoft.ApiManagement/service/tags/read | 取得標記的清單，或取得標記的詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/tags/write | 新增標記，或更新現有的標記詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/templates/delete | 重設預設的 API 管理電子郵件範本 |
> | 動作 | Microsoft.ApiManagement/service/templates/read | 取得所有電子郵件範本，或取得 API 管理電子郵件範本詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/templates/write | 建立或更新 API 管理電子郵件範本，或更新 API 管理電子郵件範本 |
> | 動作 | Microsoft.ApiManagement/service/tenant/delete | 移除租用戶的原則組態 |
> | 動作 | Microsoft.ApiManagement/service/tenant/deploy/action | 執行部署工作，以將所指定 git 分支的變更套用至資料庫中的組態。 |
> | 動作 | Microsoft.ApiManagement/service/tenant/operationResults/read | 取得作業結果的清單，或取得特定作業的結果 |
> | 動作 | Microsoft.ApiManagement/service/tenant/read | 取得租用戶的原則設定，或取得租用戶存取資訊詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/tenant/regeneratePrimaryKey/action | 重新產生主要存取金鑰 |
> | 動作 | Microsoft.ApiManagement/service/tenant/regenerateSecondaryKey/action | 重新產生次要存取金鑰 |
> | 動作 | Microsoft.ApiManagement/service/tenant/save/action | 在存放庫的指定分支中建立具有組態快照集的認可 |
> | 動作 | Microsoft.ApiManagement/service/tenant/syncState/read | 取得上次 git 同步處理的狀態 |
> | 動作 | Microsoft.ApiManagement/service/tenant/validate/action | 驗證所指定 git 分支的變更 |
> | 動作 | Microsoft.ApiManagement/service/tenant/write | 設定租用戶的原則設定，或更新租用戶存取資訊詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/updatecertificate/action | 上傳 API 管理服務的 SSL 憑證 |
> | 動作 | Microsoft.ApiManagement/service/updatehostname/action | 設定、更新或移除 API 管理服務的自訂網域名稱 |
> | 動作 | Microsoft.ApiManagement/service/users/action | 註冊新的使用者 |
> | 動作 | Microsoft.ApiManagement/service/users/applications/attachments/delete | 移除附件 |
> | 動作 | Microsoft.ApiManagement/service/users/applications/attachments/read | 取得應用程式附件，或取得附件 |
> | 動作 | Microsoft.ApiManagement/service/users/applications/attachments/write | 將附件新增至應用程式 |
> | 動作 | Microsoft.ApiManagement/service/users/applications/delete | 移除現有應用程式 |
> | 動作 | Microsoft.ApiManagement/service/users/applications/read | 取得所有使用者應用程式的清單，或取得 API 管理應用程式詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/users/applications/write | 向 API 管理註冊應用程式，或更新應用程式詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/users/delete | 移除使用者帳戶 |
> | 動作 | Microsoft.ApiManagement/service/users/generateSsoUrl/action | 產生 SSO URL。 URL 可用來存取管理入口網站 |
> | 動作 | Microsoft.ApiManagement/service/users/groups/read | 取得使用者群組的清單 |
> | 動作 | Microsoft.ApiManagement/service/users/keys/read | 取得使用者金鑰的清單 |
> | 動作 | Microsoft.ApiManagement/service/users/read | 取得已註冊的使用者清單，或取得使用者的帳戶詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/users/subscriptions/read | 取得使用者訂用帳戶的清單 |
> | 動作 | Microsoft.ApiManagement/service/users/token/action | 為使用者取得權杖的存取權杖 |
> | 動作 | Microsoft.ApiManagement/service/users/write | 註冊新的使用者，或更新現有使用者的帳戶詳細資料 |
> | 動作 | Microsoft.ApiManagement/service/write | 建立 API 管理服務的新執行個體 |
> | 動作 | Microsoft.ApiManagement/unregister/action | 針對 Microsoft.ApiManagement 資源提供者取消註冊訂用帳戶 |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Authorization/checkAccess/action | 檢查呼叫者是否有權執行特定動作 |
> | 動作 | Microsoft.Authorization/classicAdministrators/delete | 從訂用帳戶中移除系統管理員。 |
> | 動作 | Microsoft.Authorization/classicAdministrators/operationstatuses/read | 取得訂用帳戶的管理員作業狀態。 |
> | 動作 | Microsoft.Authorization/classicAdministrators/read | 讀取訂用帳戶的系統管理員。 |
> | 動作 | Microsoft.Authorization/classicAdministrators/write | 對訂用帳戶新增或修改系統管理員。 |
> | 動作 | Microsoft.Authorization/denyAssignments/delete | 在指定範圍刪除拒絕指派。 |
> | 動作 | Microsoft.Authorization/denyAssignments/read | 取得拒絕指派的資訊。 |
> | 動作 | Microsoft.Authorization/denyAssignments/write | 在指定範圍建立拒絕指派。 |
> | 動作 | Microsoft.Authorization/elevateAccess/action | 對呼叫者授與租用戶範圍的使用者存取系統管理員存取權 |
> | 動作 | Microsoft.Authorization/locks/delete | 刪除指定範圍的鎖定。 |
> | 動作 | Microsoft.Authorization/locks/read | 取得指定範圍的鎖定。 |
> | 動作 | Microsoft.Authorization/locks/write | 新增指定範圍的鎖定。 |
> | 動作 | Microsoft.Authorization/operations/read | 取得作業的清單 |
> | 動作 | Microsoft.Authorization/permissions/read | 列出呼叫者在給定範圍內所具有的所有權限。 |
> | 動作 | Microsoft.Authorization/policyAssignments/delete | 刪除指定範圍的原則指派。 |
> | 動作 | Microsoft.Authorization/policyAssignments/read | 取得關於原則指派的資訊。 |
> | 動作 | Microsoft.Authorization/policyAssignments/write | 建立指定範圍的原則指派。 |
> | 動作 | Microsoft.Authorization/policyDefinitions/delete | 刪除原則定義。 |
> | 動作 | Microsoft.Authorization/policyDefinitions/read | 取得關於原則定義的資訊。 |
> | 動作 | Microsoft.Authorization/policyDefinitions/write | 建立自訂的原則定義。 |
> | 動作 | Microsoft.Authorization/policySetDefinitions/delete | 刪除原則集合定義。 |
> | 動作 | Microsoft.Authorization/policySetDefinitions/read | 取得原則集合定義的相關資訊。 |
> | 動作 | Microsoft.Authorization/policySetDefinitions/write | 建立自訂原則集合定義。 |
> | 動作 | Microsoft.Authorization/providerOperations/read | 針對所有資源提供者取得可在角色定義中使用的作業。 |
> | 動作 | Microsoft.Authorization/roleAssignments/delete | 刪除指定範圍內的角色指派。 |
> | 動作 | Microsoft.Authorization/roleAssignments/read | 取得關於角色指派的資訊。 |
> | 動作 | Microsoft.Authorization/roleAssignments/write | 建立指定範圍的角色指派。 |
> | 動作 | Microsoft.Authorization/roleDefinitions/delete | 刪除指定的自訂角色定義。 |
> | 動作 | Microsoft.Authorization/roleDefinitions/read | 取得關於角色定義的資訊。 |
> | 動作 | Microsoft.Authorization/roleDefinitions/write | 以指定權限和可指派的範圍來建立或更新自訂角色定義。 |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Automation/automationAccounts/agentRegistrationInformation/read | 讀取 Azure Automation DSC 的註冊資訊 |
> | 動作 | Microsoft.Automation/automationAccounts/agentRegistrationInformation/regenerateKey/action | 寫入要求以重新產生 Azure Automation DSC 金鑰 |
> | 動作 | Microsoft.Automation/automationAccounts/certificates/delete | 刪除 Azure 自動化憑證資產 |
> | 動作 | Microsoft.Automation/automationAccounts/certificates/getCount/action | 讀取憑證的計數 |
> | 動作 | Microsoft.Automation/automationAccounts/certificates/read | 取得 Azure 自動化憑證資產 |
> | 動作 | Microsoft.Automation/automationAccounts/certificates/write | 建立或更新 Azure 自動化憑證資產 |
> | 動作 | Microsoft.Automation/automationAccounts/compilationjobs/read | 讀取 Azure Automation DSC 的編譯 |
> | 動作 | Microsoft.Automation/automationAccounts/compilationjobs/read | 讀取 Azure Automation DSC 的編譯 |
> | 動作 | Microsoft.Automation/automationAccounts/compilationjobs/write | 寫入 Azure Automation DSC 的編譯 |
> | 動作 | Microsoft.Automation/automationAccounts/compilationjobs/write | 寫入 Azure Automation DSC 的編譯 |
> | 動作 | Microsoft.Automation/automationAccounts/configurations/content/read | 讀取組態媒體內容 |
> | 動作 | Microsoft.Automation/automationAccounts/configurations/delete | 刪除 Azure Automation DSC 的內容 |
> | 動作 | Microsoft.Automation/automationAccounts/configurations/getCount/action | 讀取 Azure Automation DSC 內容的計數 |
> | 動作 | Microsoft.Automation/automationAccounts/configurations/read | 取得 Azure Automation DSC 的內容 |
> | 動作 | Microsoft.Automation/automationAccounts/configurations/write | 寫入 Azure Automation DSC 的內容 |
> | 動作 | Microsoft.Automation/automationAccounts/connections/delete | 刪除 Azure 自動化連線資產 |
> | 動作 | Microsoft.Automation/automationAccounts/connections/getCount/action | 讀取連線的計數 |
> | 動作 | Microsoft.Automation/automationAccounts/connections/read | 取得 Azure 自動化連線資產 |
> | 動作 | Microsoft.Automation/automationAccounts/connections/write | 建立或更新 Azure 自動化連線資產 |
> | 動作 | Microsoft.Automation/automationAccounts/connectionTypes/delete | 刪除 Azure 自動化連線類型資產 |
> | 動作 | Microsoft.Automation/automationAccounts/connectionTypes/read | 取得 Azure 自動化連線類型資產 |
> | 動作 | Microsoft.Automation/automationAccounts/connectionTypes/write | 建立 Azure 自動化連線類型資產 |
> | 動作 | Microsoft.Automation/automationAccounts/credentials/delete | 刪除 Azure 自動化認證資產 |
> | 動作 | Microsoft.Automation/automationAccounts/credentials/getCount/action | 讀取認證的計數 |
> | 動作 | Microsoft.Automation/automationAccounts/credentials/read | 取得 Azure 自動化認證資產 |
> | 動作 | Microsoft.Automation/automationAccounts/credentials/write | 建立或更新 Azure 自動化認證資產 |
> | 動作 | Microsoft.Automation/automationAccounts/delete | 刪除 Azure 自動化帳戶 |
> | 動作 | Microsoft.Automation/automationAccounts/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Automation/automationAccounts/diagnosticSettings/write | 設定資源的診斷設定 |
> | 動作 | Microsoft.Automation/automationAccounts/hybridRunbookWorkerGroups/delete | 刪除混合式 Runbook 背景工作角色資源 |
> | 動作 | Microsoft.Automation/automationAccounts/hybridRunbookWorkerGroups/read | 讀取混合式 Runbook 背景工作角色資源 |
> | 動作 | Microsoft.Automation/automationAccounts/jobs/output/read | 取得作業的輸出 |
> | 動作 | Microsoft.Automation/automationAccounts/jobs/read | 取得 Azure 自動化作業 |
> | 動作 | Microsoft.Automation/automationAccounts/jobs/resume/action | 繼續 Azure 自動化作業 |
> | 動作 | Microsoft.Automation/automationAccounts/jobs/runbookContent/action | 在作業執行時取得 Azure 自動化 Runbook 的內容 |
> | 動作 | Microsoft.Automation/automationAccounts/jobs/stop/action | 停止 Azure 自動化作業 |
> | 動作 | Microsoft.Automation/automationAccounts/jobs/streams/read | 取得 Azure 自動化作業串流 |
> | 動作 | Microsoft.Automation/automationAccounts/jobs/streams/read | 取得 Azure 自動化作業串流 |
> | 動作 | Microsoft.Automation/automationAccounts/jobs/suspend/action | 暫止 Azure 自動化作業 |
> | 動作 | Microsoft.Automation/automationAccounts/jobs/write | 建立 Azure 自動化作業 |
> | 動作 | Microsoft.Automation/automationAccounts/jobSchedules/delete | 刪除 Azure 自動化作業排程 |
> | 動作 | Microsoft.Automation/automationAccounts/jobSchedules/read | 取得 Azure 自動化作業排程 |
> | 動作 | Microsoft.Automation/automationAccounts/jobSchedules/write | 建立 Azure 自動化作業排程 |
> | 動作 | Microsoft.Automation/automationAccounts/linkedWorkspace/read | 取得連結至自動化帳戶的工作區 |
> | 動作 | Microsoft.Automation/automationAccounts/listKeys/action | 讀取自動化帳戶的金鑰 |
> | 動作 | Microsoft.Automation/automationAccounts/logDefinitions/read | 取得自動化帳戶的可用記錄 |
> | 動作 | Microsoft.Automation/automationAccounts/modules/activities/read | 取得 Azure 自動化活動 |
> | 動作 | Microsoft.Automation/automationAccounts/modules/delete | 刪除 Azure 自動化 Powershell 模組 |
> | 動作 | Microsoft.Automation/automationAccounts/modules/getCount/action | 取得自動化帳戶內的 Powershell 模組計數 |
> | 動作 | Microsoft.Automation/automationAccounts/modules/read | 取得 Azure 自動化 Powershell 模組 |
> | 動作 | Microsoft.Automation/automationAccounts/modules/write | 建立或更新 Azure 自動化 Powershell 模組 |
> | 動作 | Microsoft.Automation/automationAccounts/nodeConfigurations/delete | 刪除 Azure Automation DSC 的節點設定 |
> | 動作 | Microsoft.Automation/automationAccounts/nodeConfigurations/rawContent/action | 讀取 Azure Automation DSC 的節點設定內容 |
> | 動作 | Microsoft.Automation/automationAccounts/nodeConfigurations/read | 讀取 Azure Automation DSC 的節點設定 |
> | 動作 | Microsoft.Automation/automationAccounts/nodeConfigurations/write | 寫入 Azure Automation DSC 的節點設定 |
> | 動作 | Microsoft.Automation/automationAccounts/nodecounts/read | 讀取指定類型的節點計數摘要 |
> | 動作 | Microsoft.Automation/automationAccounts/nodes/delete | 刪除 Azure Automation DSC 節點 |
> | 動作 | Microsoft.Automation/automationAccounts/nodes/read | 讀取 Azure Automation DSC 節點 |
> | 動作 | Microsoft.Automation/automationAccounts/nodes/reports/content/read | 讀取 Azure Automation DSC 報告內容 |
> | 動作 | Microsoft.Automation/automationAccounts/nodes/reports/read | 讀取 Azure Automation DSC 報告 |
> | 動作 | Microsoft.Automation/automationAccounts/nodes/write | 建立或更新 Azure Automation DSC 節點 |
> | 動作 | Microsoft.Automation/automationAccounts/objectDataTypes/fields/read | 取得 Azure 自動化 TypeField |
> | 動作 | Microsoft.Automation/automationAccounts/providers/Microsoft.Insights/metricDefinitions/read | 取得自動化計量定義 |
> | 動作 | Microsoft.Automation/automationAccounts/python2Packages/delete | 刪除 Azure 自動化 Python 2 套件 |
> | 動作 | Microsoft.Automation/automationAccounts/python2Packages/read | 取得 Azure 自動化 Python 2 套件 |
> | 動作 | Microsoft.Automation/automationAccounts/python2Packages/write | 建立或更新 Azure 自動化 Python 2 套件 |
> | 動作 | Microsoft.Automation/automationAccounts/python3Packages/delete | 刪除 Azure 自動化 Python 3 套件 |
> | 動作 | Microsoft.Automation/automationAccounts/python3Packages/read | 取得 Azure 自動化 Python 3 套件 |
> | 動作 | Microsoft.Automation/automationAccounts/python3Packages/write | 建立或更新 Azure 自動化 Python 3 套件 |
> | 動作 | Microsoft.Automation/automationAccounts/read | 取得 Azure 自動化帳戶 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/content/read | 取得 Azure 自動化 Runbook 的內容 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/delete | 刪除 Azure 自動化 Runbook |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/draft/content/write | 建立 Azure 自動化 Runbook 草稿的內容 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/draft/operationResults/read | 取得 Azure 自動化 Runbook 草稿作業結果 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/draft/read | 取得 Azure 自動化 Runbook 草稿 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/draft/testJob/read | 取得 Azure 自動化 Runbook 草稿測試作業 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/draft/testJob/resume/action | 繼續 Azure 自動化 Runbook 草稿測試作業 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/draft/testJob/stop/action | 停止 Azure 自動化 Runbook 草稿測試作業 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/draft/testJob/suspend/action | 暫止 Azure 自動化 Runbook 草稿測試作業 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/draft/testJob/write | 建立 Azure 自動化 Runbook 草稿測試作業 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/draft/undoEdit/action | 復原對 Azure 自動化 Runbook 草稿所做的編輯 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/getCount/action | 取得 Azure 自動化 Runbook 的計數 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/publish/action | 發佈 Azure 自動化 Runbook 草稿 |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/read | 取得 Azure 自動化 Runbook |
> | 動作 | Microsoft.Automation/automationAccounts/runbooks/write | 建立或更新 Azure 自動化 Runbook |
> | 動作 | Microsoft.Automation/automationAccounts/schedules/delete | 刪除 Azure 自動化排程資產 |
> | 動作 | Microsoft.Automation/automationAccounts/schedules/getCount/action | 取得 Azure 自動化排程的計數 |
> | 動作 | Microsoft.Automation/automationAccounts/schedules/read | 取得 Azure 自動化排程資產 |
> | 動作 | Microsoft.Automation/automationAccounts/schedules/write | 建立或更新 Azure 自動化排程資產 |
> | 動作 | Microsoft.Automation/automationAccounts/softwareUpdateConfigurations/delete | 刪除 Azure 自動化軟體更新設定 |
> | 動作 | Microsoft.Automation/automationAccounts/softwareUpdateConfigurations/read | 取得 Azure 自動化軟體更新設定 |
> | 動作 | Microsoft.Automation/automationAccounts/softwareUpdateConfigurations/write | 建立或更新 Azure 自動化軟體更新設定 |
> | 動作 | Microsoft.Automation/automationAccounts/statistics/read | 取得 Azure 自動化統計資料 |
> | 動作 | Microsoft.Automation/automationAccounts/updateDeploymentMachineRuns/read | 取得 Azure 自動化更新部署機器 |
> | 動作 | Microsoft.Automation/automationAccounts/updateManagementPatchJob/read | 取得 Azure 自動化更新管理修補作業 |
> | 動作 | Microsoft.Automation/automationAccounts/usages/read | 取得 Azure 自動化使用方式 |
> | 動作 | Microsoft.Automation/automationAccounts/variables/delete | 刪除 Azure 自動化變數資產 |
> | 動作 | Microsoft.Automation/automationAccounts/variables/read | 讀取 Azure 自動化變數資產 |
> | 動作 | Microsoft.Automation/automationAccounts/variables/write | 建立或更新 Azure 自動化變數資產 |
> | 動作 | Microsoft.Automation/automationAccounts/watchers/delete | 刪除 Azure 自動化監看員作業 |
> | 動作 | Microsoft.Automation/automationAccounts/watchers/read | 取得 Azure 自動化監看員作業 |
> | 動作 | Microsoft.Automation/automationAccounts/watchers/start/action | 啟動 Azure 自動化監看員作業 |
> | 動作 | Microsoft.Automation/automationAccounts/watchers/stop/action | 停止 Azure 自動化監看員作業 |
> | 動作 | Microsoft.Automation/automationAccounts/watchers/streams/read | 取得 Azure 自動化監看員作業串流 |
> | 動作 | Microsoft.Automation/automationAccounts/watchers/watcherActions/delete | 刪除 Azure 自動化監看員作業動作 |
> | 動作 | Microsoft.Automation/automationAccounts/watchers/watcherActions/read | 取得 Azure 自動化監看員作業動作 |
> | 動作 | Microsoft.Automation/automationAccounts/watchers/watcherActions/write | 建立 Azure 自動化監看員作業動作 |
> | 動作 | Microsoft.Automation/automationAccounts/watchers/write | 建立 Azure 自動化監看員作業 |
> | 動作 | Microsoft.Automation/automationAccounts/webhooks/action | 產生 Azure 自動化 Webhook 的 URI |
> | 動作 | Microsoft.Automation/automationAccounts/webhooks/delete | 刪除 Azure 自動化 Webhook  |
> | 動作 | Microsoft.Automation/automationAccounts/webhooks/read | 讀取 Azure 自動化 Webhook |
> | 動作 | Microsoft.Automation/automationAccounts/webhooks/write | 建立或更新 Azure 自動化 Webhook |
> | 動作 | Microsoft.Automation/automationAccounts/write | 建立或更新 Azure 自動化帳戶 |
> | 動作 | Microsoft.Automation/operations/read | 取得 Azure 自動化資源的可用作業 |
> | 動作 | Microsoft.Automation/register/action | 對 Azure 自動化註冊訂用帳戶 |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.AzureActiveDirectory/b2cDirectories/delete | 刪除 B2C 目錄資源 |
> | 動作 | Microsoft.AzureActiveDirectory/b2cDirectories/read | 檢視 B2C 目錄資源 |
> | 動作 | Microsoft.AzureActiveDirectory/b2cDirectories/write | 建立或更新 B2C 目錄資源 |
> | 動作 | Microsoft.AzureActiveDirectory/operations/read | 讀取可供 Microsoft.AzureActiveDirectory 資源提供者使用的所有 API 作業 |
> | 動作 | Microsoft.AzureActiveDirectory/register/action | 為 Microsoft.AzureActiveDirectory 資源提供者註冊訂用帳戶 |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.AzureStack/Operations/read | 取得資源提供者作業的屬性 |
> | 動作 | Microsoft.AzureStack/register/action | 向 Microsoft.AzureStack 資源提供者註冊訂用帳戶 |
> | 動作 | Microsoft.AzureStack/registrations/customerSubscriptions/delete | 刪除 Azure Stack 客戶訂用帳戶 |
> | 動作 | Microsoft.AzureStack/registrations/customerSubscriptions/read | 取得 Azure Stack 客戶訂用帳戶的屬性 |
> | 動作 | Microsoft.AzureStack/registrations/customerSubscriptions/write | 建立或更新 Azure Stack 客戶訂用帳戶 |
> | 動作 | Microsoft.AzureStack/registrations/delete | 刪除 Azure Stack 註冊 |
> | 動作 | Microsoft.AzureStack/registrations/getActivationKey/action | 取得最新的 Azure Stack 啟用金鑰 |
> | 動作 | Microsoft.AzureStack/registrations/products/listDetails/action | 擷取 Azure Stack Marketplace 產品的延伸詳細資料 |
> | 動作 | Microsoft.AzureStack/registrations/products/read | 取得 Azure Stack Marketplace 產品的屬性 |
> | 動作 | Microsoft.AzureStack/registrations/read | 取得 Azure Stack 註冊的屬性 |
> | 動作 | Microsoft.AzureStack/registrations/write | 建立或更新 Azure Stack 註冊 |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Batch/batchAccounts/applications/delete | 刪除應用程式 |
> | 動作 | Microsoft.Batch/batchAccounts/applications/read | 列出應用程式，或取得應用程式的屬性 |
> | 動作 | Microsoft.Batch/batchAccounts/applications/versions/activate/action | 啟動應用程式套件 |
> | 動作 | Microsoft.Batch/batchAccounts/applications/versions/delete | 刪除應用程式套件 |
> | 動作 | Microsoft.Batch/batchAccounts/applications/versions/read | 取得應用程式套件的屬性 |
> | 動作 | Microsoft.Batch/batchAccounts/applications/versions/write | 建立新的應用程式套件，或更新現有的應用程式套件 |
> | 動作 | Microsoft.Batch/batchAccounts/applications/write | 建立新的應用程式，或更新現有應用程式 |
> | 動作 | Microsoft.Batch/batchAccounts/certificateOperationResults/read | 取得 Batch 帳戶上長時間執行憑證作業的結果 |
> | 動作 | Microsoft.Batch/batchAccounts/certificates/cancelDelete/action | 取消 Batch 帳戶上無法刪除憑證的作業 |
> | 動作 | Microsoft.Batch/batchAccounts/certificates/delete | 從 Batch 帳戶刪除憑證 |
> | 動作 | Microsoft.Batch/batchAccounts/certificates/read | 列出批 Batch 帳戶上的憑證，或取得憑證的屬性 |
> | 動作 | Microsoft.Batch/batchAccounts/certificates/write | 在 Batch 帳戶上建立新的憑證，或更新現有憑證 |
> | 動作 | Microsoft.Batch/batchAccounts/delete | 刪除 Batch 帳戶 |
> | 動作 | Microsoft.Batch/batchAccounts/listkeys/action | 列出 Batch 帳戶的存取金鑰 |
> | 動作 | Microsoft.Batch/batchAccounts/operationResults/read | 取得長時間執行 Batch 帳戶作業的結果 |
> | 動作 | Microsoft.Batch/batchAccounts/poolOperationResults/read | 取得 Batch 帳戶上長時間執行集區作業的結果 |
> | 動作 | Microsoft.Batch/batchAccounts/pools/delete | 從 Batch 帳戶刪除集區 |
> | 動作 | Microsoft.Batch/batchAccounts/pools/disableAutoscale/action | 停用 Batch 帳戶集區的自動縮放比例 |
> | 動作 | Microsoft.Batch/batchAccounts/pools/read | 列出 Batch 帳戶上的集區，或取得集區的屬性 |
> | 動作 | Microsoft.Batch/batchAccounts/pools/stopResize/action | 停止 Batch 帳戶集區上正在進行的調整大小作業 |
> | 動作 | Microsoft.Batch/batchAccounts/pools/upgradeOs/action | 將 Batch 帳戶集區的作業系統升級 |
> | 動作 | Microsoft.Batch/batchAccounts/pools/write | 在 Batch 帳戶上建立新的集區，或更新現有的集區 |
> | 動作 | Microsoft.Batch/batchAccounts/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Batch/batchAccounts/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Batch/batchAccounts/providers/Microsoft.Insights/logDefinitions/read | 取得 Batch 服務的可用記錄 |
> | 動作 | Microsoft.Batch/batchAccounts/providers/Microsoft.Insights/metricDefinitions/read | 取得 Batch 服務的可用計量 |
> | 動作 | Microsoft.Batch/batchAccounts/read | 列出 Batch 帳戶，或取得 Batch 帳戶的屬性 |
> | 動作 | Microsoft.Batch/batchAccounts/regeneratekeys/action | 重新產生 Batch 帳戶的存取金鑰 |
> | 動作 | Microsoft.Batch/batchAccounts/syncAutoStorageKeys/action | 針對為 Batch 帳戶所設定的自動儲存體帳戶同步處理存取金鑰 |
> | 動作 | Microsoft.Batch/batchAccounts/write | 建立新的 Batch 帳戶，或更新現有的 Batch 帳戶 |
> | 動作 | Microsoft.Batch/locations/checkNameAvailability/action | 檢查帳戶名稱有效，且並非使用中。 |
> | 動作 | Microsoft.Batch/locations/quotas/read | 取得指定訂用帳戶在指定 Azure 區域內的 Batch 配額 |
> | 動作 | Microsoft.Batch/operations/read | 列出可對 Microsoft.Batch 資源提供者進行的作業 |
> | 動作 | Microsoft.Batch/register/action | 針對 Batch 資源提供者註冊訂用帳戶，並讓您能夠建立 Batch 帳戶 |
> | 動作 | Microsoft.Batch/unregister/action | 為 Batch 資源提供者取消註冊訂用帳戶，以防止建立 Batch 帳戶 |

## <a name="microsoftbatchai"></a>Microsoft.BatchAI

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.BatchAI/clusters/read | 列出 Batch AI 叢集，或取得 Batch AI 叢集的屬性 |
> | 動作 | Microsoft.BatchAI/fileservers/read | 列出 Batch AI 檔案伺服器，或取得 Batch AI 檔案伺服器的屬性 |
> | 動作 | Microsoft.BatchAI/locations/operationresults/read | 取得所指定 Azure 區域的 Batch AI 非同步作業結果 |
> | 動作 | Microsoft.BatchAI/locations/operationstatuses/read | 取得所指定 Azure 區域的 Batch AI 非同步作業狀態 |
> | 動作 | Microsoft.BatchAI/locations/usages/read | 取得指定 Azure 區域中指定訂用帳戶的 Batch AI 使用情況 |
> | 動作 | Microsoft.BatchAI/register/action | 為 Batch AI 資源提供者註冊訂用帳戶，並讓您能夠建立 Batch AI 資源 |
> | 動作 | Microsoft.BatchAI/unregister/action | 將防止建立 Batch AI 資源之 Batch AI 資源提供者的訂用帳戶取消註冊 |
> | 動作 | Microsoft.BatchAI/workspaces/clusters/delete | 刪除 Batch AI 叢集 |
> | 動作 | Microsoft.BatchAI/workspaces/clusters/read | 列出 Batch AI 叢集，或取得 Batch AI 叢集的屬性 |
> | 動作 | Microsoft.BatchAI/workspaces/clusters/remoteLoginInformation/action | 列出 Batch AI 叢集的遠端登入資訊 |
> | 動作 | Microsoft.BatchAI/workspaces/clusters/write | 建立新的 Batch AI 叢集，或更新現有的 Batch AI 叢集 |
> | 動作 | Microsoft.BatchAI/workspaces/delete | 刪除 Batch AI 工作區 |
> | 動作 | Microsoft.BatchAI/workspaces/experiments/delete | 刪除 Batch AI 實驗 |
> | 動作 | Microsoft.BatchAI/workspaces/experiments/jobs/delete | 刪除 Batch AI 作業 |
> | 動作 | Microsoft.BatchAI/workspaces/experiments/jobs/listoutputfiles/action | 列出 Batch AI 作業的輸出檔案 |
> | 動作 | Microsoft.BatchAI/workspaces/experiments/jobs/read | 列出 Batch AI 作業，或取得 Batch AI 作業的屬性 |
> | 動作 | Microsoft.BatchAI/workspaces/experiments/jobs/remoteLoginInformation/action | 列出 Batch AI 作業的遠端登入資訊 |
> | 動作 | Microsoft.BatchAI/workspaces/experiments/jobs/terminate/action | 終止 Batch AI 作業 |
> | 動作 | Microsoft.BatchAI/workspaces/experiments/jobs/write | 建立新的 Batch AI 作業，或更新現有的 Batch AI 作業 |
> | 動作 | Microsoft.BatchAI/workspaces/experiments/read | 列出 Batch AI 實驗，或取得 Batch AI 實驗的屬性 |
> | 動作 | Microsoft.BatchAI/workspaces/experiments/write | 建立新的 Batch AI 實驗，或更新現有的 Batch AI 實驗 |
> | 動作 | Microsoft.BatchAI/workspaces/fileservers/delete | 刪除 Batch AI 檔案伺服器 |
> | 動作 | Microsoft.BatchAI/workspaces/fileservers/read | 列出 Batch AI 檔案伺服器，或取得 Batch AI 檔案伺服器的屬性 |
> | 動作 | Microsoft.BatchAI/workspaces/fileservers/write | 建立新的 Batch AI 檔案伺服器，或更新現有的 Batch AI 檔案伺服器 |
> | 動作 | Microsoft.BatchAI/workspaces/read | 列出 Batch AI 工作區，或取得 Batch AI 工作區的屬性 |
> | 動作 | Microsoft.BatchAI/workspaces/write | 建立新的 Batch AI 工作區，或更新現有的 Batch AI 工作區 |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Billing/billingPeriods/read | 列出可用的計費週期 |
> | 動作 | Microsoft.Billing/invoices/read | 列出可用的發票 |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.BingMaps/mapApis/Delete | 刪除作業 |
> | 動作 | Microsoft.BingMaps/mapApis/listSecrets/action | 列出祕密 |
> | 動作 | Microsoft.BingMaps/mapApis/listSingleSignOnToken/action | 讀取資源的單一登入授權權杖 |
> | 動作 | Microsoft.BingMaps/mapApis/Read | 讀取作業 |
> | 動作 | Microsoft.BingMaps/mapApis/regenerateKey/action | 重新產生金鑰 |
> | 動作 | Microsoft.BingMaps/mapApis/Write | 寫入作業 |
> | 動作 | Microsoft.BingMaps/Operations/read | 作業的描述。 |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.BotService/botServices/channels/delete | 刪除 Bot 服務 |
> | 動作 | Microsoft.BotService/botServices/channels/read | 讀取 Bot 服務 |
> | 動作 | Microsoft.BotService/botServices/channels/write | 寫入 Bot 服務 |
> | 動作 | Microsoft.BotService/botServices/connections/delete | 刪除 Bot 服務 |
> | 動作 | Microsoft.BotService/botServices/connections/read | 讀取 Bot 服務 |
> | 動作 | Microsoft.BotService/botServices/connections/write | 寫入 Bot 服務 |
> | 動作 | Microsoft.BotService/botServices/delete | 刪除 Bot 服務 |
> | 動作 | Microsoft.BotService/botServices/read | 讀取 Bot 服務 |
> | 動作 | Microsoft.BotService/botServices/write | 寫入 Bot 服務 |
> | 動作 | Microsoft.BotService/locations/operationresults/read | 讀取非同步作業的狀態 |
> | 動作 | Microsoft.BotService/Operations/read | 讀取所有資源類型的作業 |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Cache/checknameavailability/action | 檢查名稱是否可用於新的 Redis 快取 |
> | 動作 | Microsoft.Cache/locations/operationresults/read | 取得先前已將 'Location' 標頭傳回用戶端之長時間執行作業的結果 |
> | 動作 | Microsoft.Cache/operations/read | 列出 'Microsoft.Cache' 提供者支援的作業。 |
> | 動作 | Microsoft.Cache/redis/delete | 刪除整個 Redis 快取 |
> | 動作 | Microsoft.Cache/redis/export/action | 以指定格式將 Redis 資料匯出至有前置詞的儲存體 Blob |
> | 動作 | Microsoft.Cache/redis/firewallRules/delete | 刪除 Redis 快取的 IP 防火牆規則 |
> | 動作 | Microsoft.Cache/redis/firewallRules/read | 取得 Redis 快取的 IP 防火牆規則 |
> | 動作 | Microsoft.Cache/redis/firewallRules/write | 編輯 Redis 快取的 IP 防火牆規則 |
> | 動作 | Microsoft.Cache/redis/forceReboot/action | 強制重新啟動快取執行個體，但可能會造成資料遺失。 |
> | 動作 | Microsoft.Cache/redis/import/action | 從多個 Blob 將指定格式的資料匯入 Redis |
> | 動作 | Microsoft.Cache/redis/linkedservers/delete | 從 Redis 快取中刪除連結伺服器 |
> | 動作 | Microsoft.Cache/redis/linkedservers/read | 取得與 Redis 快取相關聯的連結伺服器。 |
> | 動作 | Microsoft.Cache/redis/linkedservers/write | 對 Redis 快取新增連結伺服器 |
> | 動作 | Microsoft.Cache/redis/listKeys/action | 在管理入口網站中檢視 Redis 快取存取金鑰的值 |
> | 動作 | Microsoft.Cache/redis/listUpgradeNotifications/read | 列出快取租用戶的最新升級通知。 |
> | 動作 | Microsoft.Cache/redis/metricDefinitions/read | 取得 Redis 快取可用的計量 |
> | 動作 | Microsoft.Cache/redis/patchSchedules/delete | 刪除 Redis 快取的修補排程 |
> | 動作 | Microsoft.Cache/redis/patchSchedules/read | 取得 Redis 快取的修補排程 |
> | 動作 | Microsoft.Cache/redis/patchSchedules/write | 修改 Redis 快取的修補排程 |
> | 動作 | Microsoft.Cache/redis/read | 在管理入口網站中檢視 Redis 快取的設定和組態 |
> | 動作 | Microsoft.Cache/redis/recommendations/read | 讀取 Azure Redis 快取建議 |
> | 動作 | Microsoft.Cache/redis/regenerateKey/action | 在管理入口網站中變更 Redis 快取存取金鑰的值 |
> | 動作 | Microsoft.Cache/redis/start/action | 啟動快取執行個體。 |
> | 動作 | Microsoft.Cache/redis/start/action | 啟動快取執行個體。 |
> | 動作 | Microsoft.Cache/redis/stop/action | 停止快取執行個體。 |
> | 動作 | Microsoft.Cache/redis/write | 在管理入口網站中修改 Redis 快取的設定和組態 |
> | 動作 | Microsoft.Cache/register/action | 向訂用帳戶註冊 'Microsoft.Cache' 資源提供者 |
> | 動作 | Microsoft.Cache/unregister/action | 向訂用帳戶取消註冊 'Microsoft.Cache' 資源提供者 |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Capacity/register/action | 註冊容量資源提供者，並讓您能夠建立容量資源。 |
> | 動作 | Microsoft.Capacity/reservationorders/action | 更新任何保留項目 |
> | 動作 | Microsoft.Capacity/reservationorders/delete | 刪除任何保留項目 |
> | 動作 | Microsoft.Capacity/reservationorders/read | 讀取所有保留項目 |
> | 動作 | Microsoft.Capacity/reservationorders/reservations/action | 更新任何保留項目 |
> | 動作 | Microsoft.Capacity/reservationorders/reservations/delete | 刪除任何保留項目 |
> | 動作 | Microsoft.Capacity/reservationorders/reservations/read | 讀取所有保留項目 |
> | 動作 | Microsoft.Capacity/reservationorders/reservations/revisions/read | 讀取所有保留項目 |
> | 動作 | Microsoft.Capacity/reservationorders/reservations/write | 建立任何保留項目 |
> | 動作 | Microsoft.Capacity/reservationorders/write | 建立任何保留項目 |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Cdn/CheckNameAvailability/action |  |
> | 動作 | Microsoft.Cdn/CheckResourceUsage/action |  |
> | 動作 | Microsoft.Cdn/edgenodes/delete |  |
> | 動作 | Microsoft.Cdn/edgenodes/read |  |
> | 動作 | Microsoft.Cdn/edgenodes/write |  |
> | 動作 | Microsoft.Cdn/operationresults/delete |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/CheckResourceUsage/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/delete |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/CheckResourceUsage/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/customdomainresults/delete |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/customdomainresults/DisableCustomHttps/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/customdomainresults/EnableCustomHttps/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/customdomainresults/read |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/customdomainresults/write |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/delete |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/Load/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/originresults/delete |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/originresults/read |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/originresults/write |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/Purge/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/read |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/Start/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/Stop/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/ValidateCustomDomain/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/endpointresults/write |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/GenerateSsoUri/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/GetSupportedOptimizationTypes/action |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/read |  |
> | 動作 | Microsoft.Cdn/operationresults/profileresults/write |  |
> | 動作 | Microsoft.Cdn/operationresults/read |  |
> | 動作 | Microsoft.Cdn/operationresults/write |  |
> | 動作 | Microsoft.Cdn/operations/read |  |
> | 動作 | Microsoft.Cdn/profiles/CheckResourceUsage/action |  |
> | 動作 | Microsoft.Cdn/profiles/delete |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/CheckResourceUsage/action |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/customdomains/delete |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/customdomains/DisableCustomHttps/action |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/customdomains/EnableCustomHttps/action |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/customdomains/read |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/customdomains/write |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/delete |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/Load/action |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/origins/delete |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/origins/read |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/origins/write |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Cdn/profiles/endpoints/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Cdn/profiles/endpoints/providers/Microsoft.Insights/logDefinitions/read | 取得 Microsoft.Cdn 的可用記錄 |
> | 動作 | Microsoft.Cdn/profiles/endpoints/Purge/action |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/read |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/Start/action |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/Stop/action |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/ValidateCustomDomain/action |  |
> | 動作 | Microsoft.Cdn/profiles/endpoints/write |  |
> | 動作 | Microsoft.Cdn/profiles/GenerateSsoUri/action |  |
> | 動作 | Microsoft.Cdn/profiles/GetSupportedOptimizationTypes/action |  |
> | 動作 | Microsoft.Cdn/profiles/read |  |
> | 動作 | Microsoft.Cdn/profiles/write |  |
> | 動作 | Microsoft.Cdn/register/action | 為 CDN 資源提供者註冊訂用帳戶，並讓您能夠建立 CDN 設定檔。 |
> | 動作 | Microsoft.Cdn/ValidateProbe/action |  |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/certificates/Delete | 刪除現有憑證 |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/certificates/Read | 取得憑證清單 |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/certificates/Write | 新增憑證或更新現有憑證 |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/Delete | 刪除現有的 AppServiceCertificate |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/Read | 取得 CertificateOrders 的清單 |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/reissue/Action | 重新發行現有的 certificateorder |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/renew/Action | 更新現有的 certificateorder |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/resendEmail/Action | 重新傳送憑證電子郵件 |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/resendRequestEmails/Action | 將要求電子郵件重新傳送至其他電子郵件地址 |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/resendRequestEmails/Action | 擷取已發行之 App Service 憑證的網站密封 |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/retrieveCertificateActions/Action | 擷取憑證動作的清單 |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/retrieveEmailHistory/Action | 擷取憑證電子郵件的歷程記錄 |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/verifyDomainOwnership/Action | 確認網域擁有權 |
> | 動作 | Microsoft.CertificateRegistration/certificateOrders/Write | 新增 certificateOrder 或更新現有 certificateOrder |
> | 動作 | Microsoft.CertificateRegistration/operations/Read | 列出 App Service 憑證註冊的所有作業 |
> | 動作 | Microsoft.CertificateRegistration/provisionGlobalAppServicePrincipalInUserTenant/Action | 為服務應用程式主體佈建服務主體 |
> | 動作 | Microsoft.CertificateRegistration/register/action | 針對訂用帳戶註冊 Microsoft 憑證資源提供者 |
> | 動作 | Microsoft.CertificateRegistration/validateCertificateRegistrationInformation/Action | 驗證憑證購買物件，但不將其提交出去 |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ClassicCompute/capabilities/read | 顯示功能 |
> | 動作 | Microsoft.ClassicCompute/checkDomainNameAvailability/action | 檢查給定網域名稱的可用性。 |
> | 動作 | Microsoft.ClassicCompute/checkDomainNameAvailability/read | 取得指定網域名稱的可用性。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/active/write | 設定作用中的網域名稱。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/availabilitySets/read | 顯示資源的可用性設定組。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/capabilities/read | 顯示網域名稱功能 |
> | 動作 | Microsoft.ClassicCompute/domainNames/delete | 移除資源的網域名稱。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/deploymentslots/read | 顯示部署位置。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/deploymentslots/roles/read | 取得網域名稱之部署位置上的角色 |
> | 動作 | Microsoft.ClassicCompute/domainNames/deploymentslots/roles/roleinstances/read | 取得網域名稱之部署位置上的角色執行個體 |
> | 動作 | Microsoft.ClassicCompute/domainNames/deploymentslots/state/read | 取得部署位置狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/deploymentslots/state/write | 新增部署位置狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/deploymentslots/upgradedomain/read | 取得網域名稱上之部署位置的升級網域 |
> | 動作 | Microsoft.ClassicCompute/domainNames/deploymentslots/upgradedomain/write | 更新網域名稱上之部署位置的升級網域 |
> | 動作 | Microsoft.ClassicCompute/domainNames/deploymentslots/write | 建立或更新部署。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/extensions/delete | 移除網域名稱副檔名。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/extensions/operationStatuses/read | 讀取網域名稱副檔名的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/extensions/read | 傳回網域名稱副檔名。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/extensions/write | 新增網域名稱副檔名。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/internalLoadBalancers/delete | 移除新的內部負載平衡。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/internalLoadBalancers/operationStatuses/read | 讀取網域名稱內部負載平衡器的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/internalLoadBalancers/read | 取得內部負載平衡器。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/internalLoadBalancers/write | 建立新的內部負載平衡。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/loadBalancedEndpointSets/operationStatuses/read | 讀取網域名稱之已負載平衡端點集的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/loadBalancedEndpointSets/read | 取得已負載平衡的端點集。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/loadBalancedEndpointSets/write | 新增已負載平衡的端點集。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/operationstatuses/read | 取得網域名稱的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/operationStatuses/read | 讀取網域名稱副檔名的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/read | 傳回資源的網域名稱。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/serviceCertificates/delete | 刪除所使用的服務憑證。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/serviceCertificates/operationStatuses/read | 讀取網域名稱服務憑證的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/serviceCertificates/read | 傳回所使用的服務憑證。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/serviceCertificates/write | 新增或修改所使用的服務憑證。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/abortMigration/action | 中止部署位置的移轉。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/commitMigration/action | 認可部署位置的移轉。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/delete | 刪除給定的部署位置。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/operationStatuses/read | 讀取網域名稱位置的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/prepareMigration/action | 準備部署位置的移轉。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/read | 顯示部署位置。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/extensionReferences/delete | 移除部署位置角色的擴充功能參考。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/extensionReferences/operationStatuses/read | 讀取網域名稱位置角色擴充功能參考的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/extensionReferences/read | 傳回部署位置角色的擴充功能參考。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/extensionReferences/write | 新增或修改部署位置角色的擴充功能參考。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/metricdefinitions/read | 取得網域名稱的角色計量定義。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/metrics/read | 取得網域名稱的角色計量。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/operationstatuses/read | 取得網域名稱位置角色的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/providers/Microsoft.Insights/diagnosticSettings/read | 取得診斷設定。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/providers/Microsoft.Insights/diagnosticSettings/write | 新增或修改診斷設定。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/providers/Microsoft.Insights/metricDefinitions/read | 取得計量定義。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/read | 取得部署位置的角色。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/roleInstances/downloadremotedesktopconnectionfile/action | 下載網域名稱位置角色上角色執行個體的遠端桌面連線檔案。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/roleInstances/operationStatuses/read | 取得網域名稱位置角色上角色執行個體的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/roleInstances/read | 取得角色執行個體。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/roleInstances/rebuild/action | 重建角色執行個體。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/roleInstances/reimage/action | 針對角色執行個體重新安裝映像。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/roleInstances/restart/action | 重新啟動角色執行個體。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/skus/read | 取得部署位置的角色 SKU。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/roles/write | 新增部署位置的角色。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/start/action | 啟動部署位置。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/state/start/write | 將部署位置狀態變更為停止。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/state/stop/write | 將部署位置狀態變更為啟動。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/stop/action | 暫止部署位置。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/upgradeDomain/write | 逐步升級網域。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/validateMigration/action | 驗證部署位置的移轉。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/slots/write | 建立或更新部署。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/swap/action | 將預備位置切換到生產位置。 |
> | 動作 | Microsoft.ClassicCompute/domainNames/write | 新增或修改資源的網域名稱。 |
> | 動作 | Microsoft.ClassicCompute/moveSubscriptionResources/action | 將所有傳統資源移動到不同訂用帳戶。 |
> | 動作 | Microsoft.ClassicCompute/operatingSystemFamilies/read | 列出可在 Microsoft Azure 使用的客體作業系統系列，同時也會列出各系列適用的作業系統版本。 |
> | 動作 | Microsoft.ClassicCompute/operatingSystems/read | 列出 Microsoft Azure 中目前可用的客體作業系統版本。 |
> | 動作 | Microsoft.ClassicCompute/operations/read | 取得作業的清單。 |
> | 動作 | Microsoft.ClassicCompute/operationStatuses/read | 讀取資源的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/quotas/read | 取得訂用帳戶配額。 |
> | 動作 | Microsoft.ClassicCompute/register/action | 傳統計算的暫存器 |
> | 動作 | Microsoft.ClassicCompute/resourceTypes/skus/read | 取得受支援之資源類型的 SKU 清單。 |
> | 動作 | Microsoft.ClassicCompute/validateSubscriptionMoveAvailability/action | 驗證訂用帳戶是否可進行傳統移動作業。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/associatedNetworkSecurityGroups/delete | 刪除與虛擬機器相關聯的網路安全性群組。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/associatedNetworkSecurityGroups/operationStatuses/read | 讀取與網路安全性群組相關聯之虛擬機器的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/associatedNetworkSecurityGroups/read | 取得與虛擬機器相關聯的網路安全性群組。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/associatedNetworkSecurityGroups/write | 新增與虛擬機器相關聯的網路安全性群組。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/asyncOperations/read | 取得可能的非同步作業 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/attachDisk/action | 將資料磁碟連結至虛擬機器。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/capture/action | 擷取虛擬機器。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/delete | 移除虛擬機器。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/detachDisk/action | 中斷資料磁碟對虛擬機器的連結。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/diagnosticsettings/read | 取得虛擬機器診斷設定。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/disks/read | 擷取資料磁碟清單 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/downloadRemoteDesktopConnectionFile/action | 下載虛擬機器的 RDP 檔案。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/extensions/operationStatuses/read | 讀取虛擬機器擴充功能的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/extensions/read | 取得虛擬機器擴充功能。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/extensions/write | 放置虛擬機器擴充功能。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/metricdefinitions/read | 取得虛擬機器計量定義。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/metrics/read | 取得計量。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/networkInterfaces/associatedNetworkSecurityGroups/delete | 刪除與網路介面相關聯的網路安全性群組。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/networkInterfaces/associatedNetworkSecurityGroups/operationStatuses/read | 讀取與網路安全性群組相關聯之虛擬機器的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/networkInterfaces/associatedNetworkSecurityGroups/read | 取得與網路介面相關聯的網路安全性群組。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/networkInterfaces/associatedNetworkSecurityGroups/write | 新增與網路介面相關聯的網路安全性群組。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/operationStatuses/read | 讀取虛擬機器的作業狀態。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/performMaintenance/action | 執行虛擬機器的維護。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/providers/Microsoft.Insights/diagnosticSettings/read | 取得診斷設定。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/providers/Microsoft.Insights/diagnosticSettings/write | 新增或修改診斷設定。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/providers/Microsoft.Insights/metricDefinitions/read | 取得計量定義。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/read | 擷取虛擬機器清單。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/redeploy/action | 重新部署虛擬機器。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/restart/action | 重新啟動虛擬機器。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/shutdown/action | 關閉虛擬機器。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/start/action | 啟動虛擬機器。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/stop/action | 停止虛擬機器。 |
> | 動作 | Microsoft.ClassicCompute/virtualMachines/write | 新增或修改虛擬機器。 |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ClassicNetwork/expressroutecrossconnections/operationstatuses/read | 取得 Express Route 交叉連線作業狀態。 |
> | 動作 | Microsoft.ClassicNetwork/expressroutecrossconnections/peerings/delete | 刪除 Express Route 交叉連線對等互連。 |
> | 動作 | Microsoft.ClassicNetwork/expressroutecrossconnections/peerings/operationstatuses/read | 取得 Express Route 交叉連線對等互連作業狀態。 |
> | 動作 | Microsoft.ClassicNetwork/expressroutecrossconnections/peerings/read | 取得 Express Route 交叉連線對等互連。 |
> | 動作 | Microsoft.ClassicNetwork/expressroutecrossconnections/peerings/write | 新增 Express Route 交叉連線對等互連。 |
> | 動作 | Microsoft.ClassicNetwork/expressroutecrossconnections/read | 取得 Express Route 交叉連線。 |
> | 動作 | Microsoft.ClassicNetwork/expressroutecrossconnections/write | 新增 Express Route 交叉連線。 |
> | 動作 | Microsoft.ClassicNetwork/gatewaySupportedDevices/read | 擷取支援裝置清單。 |
> | 動作 | Microsoft.ClassicNetwork/networkSecurityGroups/delete | 刪除網路安全性群組。 |
> | 動作 | Microsoft.ClassicNetwork/networkSecurityGroups/operationStatuses/read | 讀取網路安全性群組的作業狀態。 |
> | 動作 | Microsoft.ClassicNetwork/networksecuritygroups/providers/Microsoft.Insights/diagnosticSettings/read | 取得網路安全性群組診斷設定 |
> | 動作 | Microsoft.ClassicNetwork/networksecuritygroups/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新網路安全性群組診斷設定，此作業會由 Insights 資源提供者來補充。 |
> | 動作 | Microsoft.ClassicNetwork/networksecuritygroups/providers/Microsoft.Insights/logDefinitions/read | 取得網路安全性群組的事件 |
> | 動作 | Microsoft.ClassicNetwork/networkSecurityGroups/read | 取得網路安全性群組。 |
> | 動作 | Microsoft.ClassicNetwork/networkSecurityGroups/securityRules/delete | 刪除安全性規則。 |
> | 動作 | Microsoft.ClassicNetwork/networkSecurityGroups/securityRules/operationStatuses/read | 讀取網路安全性群組安全性規則的作業狀態。 |
> | 動作 | Microsoft.ClassicNetwork/networkSecurityGroups/securityRules/read | 取得安全性規則。 |
> | 動作 | Microsoft.ClassicNetwork/networkSecurityGroups/securityRules/write | 新增或更新安全性規則。 |
> | 動作 | Microsoft.ClassicNetwork/networkSecurityGroups/write | 新增網路安全性群組。 |
> | 動作 | Microsoft.ClassicNetwork/operations/read | 取得傳統網路作業。 |
> | 動作 | Microsoft.ClassicNetwork/quotas/read | 取得訂用帳戶配額。 |
> | 動作 | Microsoft.ClassicNetwork/register/action | 傳統網路的暫存器 |
> | 動作 | Microsoft.ClassicNetwork/reservedIps/delete | 刪除保留的 IP。 |
> | 動作 | Microsoft.ClassicNetwork/reservedIps/join/action | 加入保留的 IP |
> | 動作 | Microsoft.ClassicNetwork/reservedIps/link/action | 連結保留的 IP |
> | 動作 | Microsoft.ClassicNetwork/reservedIps/operationStatuses/read | 讀取保留之 IP 的作業狀態。 |
> | 動作 | Microsoft.ClassicNetwork/reservedIps/read | 取得保留的 IP |
> | 動作 | Microsoft.ClassicNetwork/reservedIps/write | 新增保留的 IP |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/abortMigration/action | 中止虛擬網路移轉 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/capabilities/read | 顯示功能 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/checkIPAddressAvailability/action | 檢查虛擬網路中給定 IP 位址的可用性。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/commitMigration/action | 認可虛擬網路移轉 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/delete | 刪除虛擬網路。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/clientRevokedCertificates/delete | 取消撤銷用戶端憑證。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/clientRevokedCertificates/read | 讀取已撤銷的用戶端憑證。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/clientRevokedCertificates/write | 撤銷用戶端憑證。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/clientRootCertificates/delete | 刪除虛擬網路閘道用戶端憑證。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/clientRootCertificates/download/action | 依指紋下載憑證。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/clientRootCertificates/listPackage/action | 列出虛擬網路閘道憑證套件。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/clientRootCertificates/read | 尋找用戶端根憑證。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/clientRootCertificates/write | 上傳新的用戶端根憑證。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/connections/connect/action | 建立網站對網站閘道連線。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/connections/disconnect/action | 中斷網站對網站閘道連線。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/connections/read | 擷取連線清單。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/connections/test/action | 測試網站對網站閘道連線。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/delete | 刪除虛擬網路閘道。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/downloadDeviceConfigurationScript/action | 下載裝置組態指令碼。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/downloadDiagnostics/action | 下載閘道診斷。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/listCircuitServiceKey/action | 擷取迴路服務金鑰。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/listPackage/action | 列出虛擬網路閘道套件。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/operationStatuses/read | 讀取虛擬網路閘道的作業狀態。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/packages/read | 取得虛擬網路閘道套件。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/read | 取得虛擬網路閘道。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/startDiagnostics/action | 啟動虛擬網路閘道的診斷。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/stopDiagnostics/action | 停止虛擬網路閘道的診斷。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/gateways/write | 新增虛擬網路閘道。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/join/action | 加入虛擬網路。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/operationStatuses/read | 讀取虛擬網路的作業狀態。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/peer/action | 讓某個虛擬網路與另一個虛擬網路對等互連。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/prepareMigration/action | 準備虛擬網路移轉 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/read | 取得虛擬網路。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/remoteVirtualNetworkPeeringProxies/delete | 刪除遠端虛擬網路對等互連 Proxy。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/remoteVirtualNetworkPeeringProxies/read | 取得遠端虛擬網路對等互連 Proxy。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/remoteVirtualNetworkPeeringProxies/write | 新增或更新遠端虛擬網路對等互連 Proxy。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/subnets/associatedNetworkSecurityGroups/delete | 刪除與子網路相關聯的網路安全性群組。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/subnets/associatedNetworkSecurityGroups/operationStatuses/read | 讀取與網路安全性群組相關聯之虛擬網路子網路的作業狀態。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/subnets/associatedNetworkSecurityGroups/read | 取得與子網路相關聯的網路安全性群組。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/subnets/associatedNetworkSecurityGroups/write | 新增與子網路相關聯的網路安全性群組。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/validateMigration/action | 驗證虛擬網路移轉 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/virtualNetworkPeerings/read | 取得虛擬網路對等互連。 |
> | 動作 | Microsoft.ClassicNetwork/virtualNetworks/write | 新增虛擬網路。 |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ClassicStorage/capabilities/read | 顯示功能 |
> | 動作 | Microsoft.ClassicStorage/checkStorageAccountAvailability/action | 檢查儲存體帳戶的可用性。 |
> | 動作 | Microsoft.ClassicStorage/checkStorageAccountAvailability/read | 取得儲存體帳戶的可用性。 |
> | 動作 | Microsoft.ClassicStorage/disks/read | 傳回儲存體帳戶磁碟。 |
> | 動作 | Microsoft.ClassicStorage/images/operationstatuses/read | 取得映像作業狀態。 |
> | 動作 | Microsoft.ClassicStorage/images/read | 傳回映像。 |
> | 動作 | Microsoft.ClassicStorage/operations/read | 取得傳統儲存體作業 |
> | 動作 | Microsoft.ClassicStorage/osImages/read | 傳回作業系統映像。 |
> | 動作 | Microsoft.ClassicStorage/osPlatformImages/read | 取得作業系統平台映像。 |
> | 動作 | Microsoft.ClassicStorage/publicImages/read | 取得公用虛擬機器映像。 |
> | 動作 | Microsoft.ClassicStorage/quotas/read | 取得訂用帳戶配額。 |
> | 動作 | Microsoft.ClassicStorage/register/action | 傳統儲存體的暫存器 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/abortMigration/action | 中止儲存體帳戶的移轉。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/commitMigration/action | 認可儲存體帳戶的移轉。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/delete | 刪除儲存體帳戶。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/disks/delete | 刪除給定的儲存體帳戶磁碟。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/disks/operationStatuses/read | 讀取資源的作業狀態。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/disks/read | 傳回儲存體帳戶磁碟。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/disks/write | 新增儲存體帳戶磁碟。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/images/delete | 刪除給定的儲存體帳戶映像。 (已淘汰。 使用 'Microsoft.ClassicStorage/storageAccounts/vmImages') |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/images/operationstatuses/read | 傳回儲存體帳戶映像作業狀態。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/images/read | 傳回儲存體帳戶映像。 (已淘汰。 使用 'Microsoft.ClassicStorage/storageAccounts/vmImages') |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/listKeys/action | 列出儲存體帳戶的存取金鑰。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/operationStatuses/read | 讀取資源的作業狀態。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/osImages/delete | 刪除給定之儲存體帳戶的作業系統映像。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/osImages/read | 傳回儲存體帳戶的作業系統映像。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/osImages/write | 新增給定儲存體帳戶作業系統映像。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/prepareMigration/action | 準備儲存體帳戶的移轉。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/read | 傳回具有給定帳戶的儲存體帳戶。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/regenerateKey/action | 重新產生儲存體帳戶的現有存取金鑰。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/services/diagnosticSettings/read | 取得診斷設定。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/services/diagnosticSettings/write | 新增或修改診斷設定。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/services/metricDefinitions/read | 取得計量定義。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/services/metrics/read | 取得計量。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/services/read | 取得可用服務。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/validateMigration/action | 驗證儲存體帳戶的移轉。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/vmImages/delete | 刪除指定的虛擬機器映像。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/vmImages/operationstatuses/read | 取得指定的虛擬機器映像作業狀態。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/vmImages/read | 傳回虛擬機器映像。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/vmImages/write | 新增指定的虛擬機器映像。 |
> | 動作 | Microsoft.ClassicStorage/storageAccounts/write | 新增儲存體帳戶。 |
> | 動作 | Microsoft.ClassicStorage/vmImages/read | 列出虛擬機器映像。 |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.CognitiveServices/accounts/delete | 刪除 API 帳戶 |
> | 動作 | Microsoft.CognitiveServices/accounts/listKeys/action | 列出金鑰 |
> | 動作 | Microsoft.CognitiveServices/accounts/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定。 |
> | 動作 | Microsoft.CognitiveServices/accounts/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定。 |
> | 動作 | Microsoft.CognitiveServices/accounts/providers/Microsoft.Insights/logDefinitions/read | 取得認知服務帳戶的可用記錄檔 |
> | 動作 | Microsoft.CognitiveServices/accounts/providers/Microsoft.Insights/metricDefinitions/read | 取得認知服務的可用計量。 |
> | 動作 | Microsoft.CognitiveServices/accounts/read | 讀取 API 帳戶。 |
> | 動作 | Microsoft.CognitiveServices/accounts/regenerateKey/action | 重新產生金鑰 |
> | 動作 | Microsoft.CognitiveServices/accounts/skus/read | 讀取現有資源的可用 SKU。 |
> | 動作 | Microsoft.CognitiveServices/accounts/usages/read | 取得現有資源的配額使用情況。 |
> | 動作 | Microsoft.CognitiveServices/accounts/write | 寫入 API 帳戶。 |
> | 動作 | Microsoft.CognitiveServices/locations/checkSkuAvailability/action | 讀取訂用帳戶的可用 SKU。 |
> | 動作 | Microsoft.CognitiveServices/Operations/read | 列出所有可用的作業 |
> | 動作 | Microsoft.CognitiveServices/register/action | 為認知服務註冊訂用帳戶 |
> | 動作 | Microsoft.CognitiveServices/skus/read | 讀取認知服務的可用 SKU。 |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Commerce/RateCard/read | 傳回給定訂用帳戶的供應項目資料、資源/計量中繼資料和速率。 |
> | 動作 | Microsoft.Commerce/UsageAggregates/read | 擷取訂用帳戶的 Microsoft Azure 耗用量。 結果會包含特定時間範圍內的彙總使用量資料、訂用帳戶和資源相關資訊。 |

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Compute/availabilitySets/delete | 刪除可用性設定組 |
> | 動作 | Microsoft.Compute/availabilitySets/read | 取得可用性設定組的屬性 |
> | 動作 | Microsoft.Compute/availabilitySets/vmSizes/read | 列出在可用性設定組中建立或更新虛擬機器時可以使用的大小 |
> | 動作 | Microsoft.Compute/availabilitySets/write | 建立新的可用性設定組，或更新現有的可用性設定組 |
> | 動作 | Microsoft.Compute/disks/beginGetAccess/action | 取得磁碟用於 Blob 存取的 SAS URI |
> | 動作 | Microsoft.Compute/disks/delete | 刪除磁碟 |
> | 動作 | Microsoft.Compute/disks/endGetAccess/action | 撤銷磁碟的 SAS URI |
> | 動作 | Microsoft.Compute/disks/read | 取得磁碟的屬性 |
> | 動作 | Microsoft.Compute/disks/write | 建立新的磁碟，或更新現有磁碟 |
> | 動作 | Microsoft.Compute/galleries/delete | 刪除資源庫 |
> | 動作 | Microsoft.Compute/galleries/images/delete | 刪除資源庫映像 |
> | 動作 | Microsoft.Compute/galleries/images/read | 取得資源庫映像的屬性 |
> | 動作 | Microsoft.Compute/galleries/images/versions/delete | 刪除資源庫映像版本 |
> | 動作 | Microsoft.Compute/galleries/images/versions/read | 取得資源庫映像版本的屬性 |
> | 動作 | Microsoft.Compute/galleries/images/versions/write | 建立新的資源庫映像版本或更新現有的資源庫映像版本 |
> | 動作 | Microsoft.Compute/galleries/images/write | 建立新的資源庫映像或更新現有的資源庫映像 |
> | 動作 | Microsoft.Compute/galleries/read | 取得資源庫的屬性 |
> | 動作 | Microsoft.Compute/galleries/write | 建立新的資源庫或更新現有的資源庫 |
> | 動作 | Microsoft.Compute/images/delete | 刪除映像 |
> | 動作 | Microsoft.Compute/images/read | 取得映像的屬性 |
> | 動作 | Microsoft.Compute/images/write | 建立新的映像，或更新現有映像 |
> | 動作 | Microsoft.Compute/locations/capsOperations/read | 取得非同步 Caps 作業的狀態 |
> | 動作 | Microsoft.Compute/locations/diskOperations/read | 取得非同步磁碟作業的狀態 |
> | 動作 | Microsoft.Compute/locations/logAnalytics/getRequestRateByInterval/action | 建立記錄檔以依時間間隔顯示要求總數以協助進行節流診斷。 |
> | 動作 | Microsoft.Compute/locations/logAnalytics/getThrottledRequests/action | 建立記錄檔以依 ResourceName、OperationName 或已套用的節流原則顯示已節流要求彙總。 |
> | 動作 | Microsoft.Compute/locations/operations/read | 取得非同步作業的狀態 |
> | 動作 | Microsoft.Compute/locations/publishers/artifacttypes/offers/read | 取得平台映像提供的屬性 |
> | 動作 | Microsoft.Compute/locations/publishers/artifacttypes/offers/skus/read | 取得平台映像 SKU 的屬性 |
> | 動作 | Microsoft.Compute/locations/publishers/artifacttypes/offers/skus/versions/read | 取得平台映像版本的屬性 |
> | 動作 | Microsoft.Compute/locations/publishers/artifacttypes/types/read | 取得 VMExtension 類型的屬性 |
> | 動作 | Microsoft.Compute/locations/publishers/artifacttypes/types/versions/read | 取得 VMExtension 版本的屬性 |
> | 動作 | Microsoft.Compute/locations/publishers/read | 取得發行者的屬性 |
> | 動作 | Microsoft.Compute/locations/runCommands/read | 列出位置中可用的執行命令 |
> | 動作 | Microsoft.Compute/locations/usages/read | 取得訂用帳戶在某個位置之計算資源的服務限制和目前的使用量 |
> | 動作 | Microsoft.Compute/locations/vmSizes/read | 列出某個位置可用的虛擬機器大小 |
> | 動作 | Microsoft.Compute/operations/read | 列出 Microsoft.Compute 資源提供者的可用作業 |
> | 動作 | Microsoft.Compute/register/action | 向 Microsoft.Compute 資源提供者註冊訂用帳戶 |
> | 動作 | Microsoft.Compute/restorePointCollections/delete | 刪除還原點集合和內含的還原點 |
> | 動作 | Microsoft.Compute/restorePointCollections/read | 取得還原點集合的屬性 |
> | 動作 | Microsoft.Compute/restorePointCollections/restorePoints/delete | 刪除還原點 |
> | 動作 | Microsoft.Compute/restorePointCollections/restorePoints/read | 取得還原點的屬性 |
> | 動作 | Microsoft.Compute/restorePointCollections/restorePoints/retrieveSasUris/action | 取得還原點屬性以及 Blob SAS URI |
> | 動作 | Microsoft.Compute/restorePointCollections/restorePoints/write | 建立新的還原點 |
> | 動作 | Microsoft.Compute/restorePointCollections/write | 建立新的還原點集合，或更新現有還原點集合 |
> | 動作 | Microsoft.Compute/sharedVMImages/delete | 刪除 SharedVMImage |
> | 動作 | Microsoft.Compute/sharedVMImages/read | 取得 SharedVMImage 的屬性 |
> | 動作 | Microsoft.Compute/sharedVMImages/versions/delete | 刪除 SharedVMImageVersion |
> | 動作 | Microsoft.Compute/sharedVMImages/versions/read | 取得 SharedVMImageVersion 的屬性 |
> | 動作 | Microsoft.Compute/sharedVMImages/versions/replicate/action | 將 SharedVMImageVersion 複寫至目標區域 |
> | 動作 | Microsoft.Compute/sharedVMImages/versions/write | 建立新的 SharedVMImageVersion，或更新現有的 SharedVMImageVersion |
> | 動作 | Microsoft.Compute/sharedVMImages/write | 建立新的 SharedVMImage，或更新現有的 SharedVMImage |
> | 動作 | Microsoft.Compute/skus/read | 取得可供您的訂用帳戶使用的 Microsoft.Compute SKU 清單 |
> | 動作 | Microsoft.Compute/snapshots/beginGetAccess/action | 取得快照集的 SAS URI 以用於 Blob 存取 |
> | 動作 | Microsoft.Compute/snapshots/delete | 刪除快照集 |
> | 動作 | Microsoft.Compute/snapshots/endGetAccess/action | 撤銷快照集的 SAS URI |
> | 動作 | Microsoft.Compute/snapshots/read | 取得快照集的屬性 |
> | 動作 | Microsoft.Compute/snapshots/write | 建立新的快照集，或更新現有快照集 |
> | 動作 | Microsoft.Compute/virtualMachines/capture/action | 藉由複製虛擬硬碟來擷取虛擬機器，並產生可用來建立類似虛擬機器的範本 |
> | 動作 | Microsoft.Compute/virtualMachines/convertToManagedDisks/action | 將虛擬機器的 Blob 型磁碟轉換為受控磁碟 |
> | 動作 | Microsoft.Compute/virtualMachines/deallocate/action | 關閉虛擬機器的電源，並將計算資源釋出 |
> | 動作 | Microsoft.Compute/virtualMachines/delete | 刪除虛擬機器 |
> | 動作 | Microsoft.Compute/virtualMachines/extensions/delete | 刪除虛擬機器擴充功能 |
> | 動作 | Microsoft.Compute/virtualMachines/extensions/read | 取得虛擬機器擴充功能的屬性 |
> | 動作 | Microsoft.Compute/virtualMachines/extensions/write | 建立新的虛擬機器擴充功能，或更新現有的虛擬機器擴充功能 |
> | 動作 | Microsoft.Compute/virtualMachines/generalize/action | 將虛擬機器的狀態設定為「已一般化」，並準備虛擬機器以供擷取 |
> | 動作 | Microsoft.Compute/virtualMachines/instanceView/read | 取得虛擬機器和其資源的詳細執行階段狀態 |
> | DataAction | Microsoft.Compute/virtualMachines/login/action | 以一般使用者身分登入虛擬機器 |
> | DataAction | Microsoft.Compute/virtualMachines/loginAsAdmin/action | 以 Windows 系統管理員或 Linux 根使用者權限登入虛擬機器 |
> | 動作 | Microsoft.Compute/virtualMachines/performMaintenance/action | 在 VM 上執行維護作業。 |
> | 動作 | Microsoft.Compute/virtualMachines/powerOff/action | 關閉虛擬機器的電源。 請注意，您會繼續收到虛擬機器的帳單。 |
> | 動作 | Microsoft.Compute/virtualMachines/providers/Microsoft.Insights/metricDefinitions/read | 讀取虛擬機器計量定義 |
> | 動作 | Microsoft.Compute/virtualMachines/read | 取得虛擬機器的屬性 |
> | 動作 | Microsoft.Compute/virtualMachines/redeploy/action | 重新部署虛擬機器 |
> | 動作 | Microsoft.Compute/virtualMachines/reimage/action | 為正在使用差異磁碟的虛擬機器重新安裝映像。 |
> | 動作 | Microsoft.Compute/virtualMachines/restart/action | 重新啟動虛擬機器 |
> | 動作 | Microsoft.Compute/virtualMachines/runCommand/action | 在虛擬機器上執行預先定義的指令碼 |
> | 動作 | Microsoft.Compute/virtualMachines/start/action | 啟動虛擬機器 |
> | 動作 | Microsoft.Compute/virtualMachines/vmSizes/read | 列出虛擬機器所能更新成的大小 |
> | 動作 | Microsoft.Compute/virtualMachines/write | 建立新的虛擬機器，或更新現有虛擬機器 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/deallocate/action | 關閉虛擬機器擴展集之執行個體的電源，並將其計算資源釋出  |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/delete | 刪除虛擬機器擴展集 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/delete/action | 刪除虛擬機器擴展集的執行個體 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/extensions/delete | 刪除虛擬機器擴展集延伸模組 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/extensions/read | 取得虛擬機器擴展集延伸模組的屬性 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/extensions/write | 建立新的虛擬機器擴展集延伸模組或更新現有的 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/forceRecoveryServiceFabricPlatformUpdateDomainWalk/action | 手動徹查 Service Fabric 虛擬機器擴展集的平台更新網域，以完成停滯的暫止更新 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/instanceView/read | 取得虛擬機器擴展集的執行個體檢視 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/manualUpgrade/action | 將虛擬機器擴展集的執行個體手動更新至最新模型 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/networkInterfaces/read | 取得虛擬機器擴展集之所有網路介面的屬性 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/osRollingUpgrade/action | 開始輪流升級，以將所有虛擬機器擴展集執行個體移至最新可用的平台映像作業系統版本。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/osUpgradeHistory/read | 取得虛擬機器擴展集的 OS 升級歷程記錄 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/performMaintenance/action | 在虛擬機器擴展集的執行個體上執行規劃的維護 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/powerOff/action | 關閉虛擬機器擴展集執行個體的電源 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/providers/Microsoft.Insights/diagnosticSettings/read | 取得虛擬機器擴展集的診斷設定。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新虛擬機器擴展集的診斷設定。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/providers/Microsoft.Insights/logDefinitions/read | 取得虛擬機器擴展集的可用記錄檔。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/providers/Microsoft.Insights/metricDefinitions/read | 讀取虛擬機器擴展集計量定義 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/publicIPAddresses/read | 取得虛擬機器擴展集之所有公用 IP 位址的屬性 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/read | 取得虛擬機器擴展集的屬性 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/redeploy/action | 重新部署虛擬機器擴展集的執行個體 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/reimage/action | 重新安裝虛擬機器擴展集執行個體的映像 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/reimageAll/action | 為虛擬機器擴展集之執行個體的所有磁碟重新安裝映像 (OS 磁碟和資料磁碟)  |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/restart/action | 重新啟動虛擬機器擴展集的執行個體 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/rollingUpgrades/cancel/action | 取消虛擬機器擴展集的輪流升級 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/rollingUpgrades/read | 取得虛擬機器擴展集最新的輪流升級狀態 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/scale/action | 驗證現有的虛擬機器擴展集是否可以相應縮小/相應放大到指定的執行個體計數 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/skus/read | 列出現有虛擬機器擴展集的有效 SKU |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/start/action | 啟動虛擬機器擴展集的執行個體 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/deallocate/action | 關閉 VM 擴展集之虛擬機器的電源，並將其計算資源釋出。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/delete | 刪除 VM 擴展集內的特定虛擬機器。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/instanceView/read | 擷取 VM 擴展集內的虛擬機器執行個體檢視。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/networkInterfaces/ipConfigurations/publicIPAddresses/read | 取得使用虛擬機器擴展集建立的公用 IP 位址屬性。 虛擬機器擴展集最多可以在每個 ipconfiguration (私人 IP) 建立一個公用 IP |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/networkInterfaces/ipConfigurations/read | 取得使用虛擬機器擴展集建立的網路介面其中一個或所有 IP 設定屬性。 IP 設定代表私人 IP |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/networkInterfaces/read | 取得使用虛擬機器擴展集建立的虛擬機器其中一個或所有網路介面屬性 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/performMaintenance/action | 在虛擬機器擴展集的虛擬機器執行個體上執行規劃的維護 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/powerOff/action | 將 VM 擴展集內的虛擬機器執行個體關閉電源。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/providers/Microsoft.Insights/metricDefinitions/read | 讀取擴展集中的虛擬機器計量定義 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/read | 擷取 VM 擴展集內的虛擬機器屬性 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/redeploy/action | 重新部署虛擬機器擴展集中的虛擬機器執行個體 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/reimage/action | 重新安裝虛擬機器擴展集中的虛擬機器執行個體映像。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/reimageAll/action | 為虛擬機器擴展集中虛擬機器執行個體的所有磁碟 (OS 磁碟和資料磁碟) 重新安裝映像。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/restart/action | 重新啟動 VM 擴展集內的虛擬機器執行個體。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/runCommand/action | 在虛擬機器擴展集中的虛擬機器執行個體上執行預先定義的指令碼。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/start/action | 啟動 VM 擴展集內的虛擬機器執行個體。 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/virtualMachines/write | 更新 VM 擴展集中虛擬機器的屬性 |
> | 動作 | Microsoft.Compute/virtualMachineScaleSets/write | 建立新的虛擬機器擴展集或更新現有的 |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Consumption/balances/read | 針對管理群組列出計費期的使用率摘要。 |
> | 動作 | Microsoft.Consumption/budgets/read | 依訂用帳戶或管理群組列出預算。 |
> | 動作 | Microsoft.Consumption/budgets/write | 依訂用帳戶或管理群組建立、更新和刪除預算。 |
> | 動作 | Microsoft.Consumption/marketplaces/read | 針對適用於 EA 與 WebDirect 訂用帳戶的範圍列出 Marketplace 資源使用狀況詳細資料。 |
> | 動作 | Microsoft.Consumption/operations/read | 列出 Microsoft.Consumption 資源提供者支援的所有作業。 |
> | 動作 | Microsoft.Consumption/pricesheets/read | 列出適用於訂用帳戶或管理群組的價位表資料。 |
> | 動作 | Microsoft.Consumption/reservationDetails/read | 依保留順序或管理群組列出保留執行個體的使用率詳細資料。 詳細資料屬於每日層級的每個執行個體。 |
> | 動作 | Microsoft.Consumption/reservationRecommendations/read | 列出訂用帳戶保留執行個體的單一或共用建議。 |
> | 動作 | Microsoft.Consumption/reservationSummaries/read | 依保留順序或管理群組列出保留執行個體的使用率摘要。 摘要資料位於每月或每日層級。 |
> | 動作 | Microsoft.Consumption/reservationTransactions/read | 依管理群組列出保留執行個體的交易記錄。 |
> | 動作 | Microsoft.Consumption/tenants/register/action | 依某個租用戶註冊 Microsoft.Consumption 範圍的動作。 |
> | 動作 | Microsoft.Consumption/terms/read | 列出適用於訂用帳戶或管理群組的條款。 |
> | 動作 | Microsoft.Consumption/usageDetails/read | 針對適用於 EA 與 WebDirect 訂用帳戶的範圍列出使用狀況詳細資料。 |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ContainerInstance/containerGroups/containers/logs/read | 取得特定容器的記錄。 |
> | 動作 | Microsoft.ContainerInstance/containerGroups/delete | 刪除特定容器群組。 |
> | 動作 | Microsoft.ContainerInstance/containerGroups/providers/Microsoft.Insights/diagnosticSettings/read | 取得容器群組的診斷設定。 |
> | 動作 | Microsoft.ContainerInstance/containerGroups/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新容器群組的診斷設定。 |
> | 動作 | Microsoft.ContainerInstance/containerGroups/providers/Microsoft.Insights/metricDefinitions/read | 取得容器群組的可用計量。 |
> | 動作 | Microsoft.ContainerInstance/containerGroups/read | 取得所有容器群組。 |
> | 動作 | Microsoft.ContainerInstance/containerGroups/restart/action | 重新啟動特定容器群組。 |
> | 動作 | Microsoft.ContainerInstance/containerGroups/stop/action | 停止特定容器群組。 計算資源將會取消配置，計費也會停止。 |
> | 動作 | Microsoft.ContainerInstance/containerGroups/write | 建立或更新特定容器群組。 |
> | 動作 | Microsoft.ContainerInstance/register/action | 為容器執行個體資源提供者註冊訂用帳戶，並允許建立容器群組。 |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ContainerRegistry/checkNameAvailability/read | 檢查容器登錄名稱是否可供使用。 |
> | 動作 | Microsoft.ContainerRegistry/locations/deleteVirtualNetworkOrSubnets/action | 讓 Microsoft.ContainerRegistry 知道虛擬網路或子網路即將刪除 |
> | 動作 | Microsoft.ContainerRegistry/locations/operationResults/read | 取得非同步作業結果 |
> | 動作 | Microsoft.ContainerRegistry/operations/read | 列出所有可用的 Azure Container Registry REST API 作業 |
> | 動作 | Microsoft.ContainerRegistry/register/action | 針對容器登錄資源提供者註冊訂用帳戶，並讓您能夠建立容器登錄。 |
> | 動作 | Microsoft.ContainerRegistry/registries/builds/cancel/action | 取消現有組建。 |
> | 動作 | Microsoft.ContainerRegistry/registries/builds/getLogLink/action | 取得下載組建記錄檔的連結。 |
> | 動作 | Microsoft.ContainerRegistry/registries/builds/read | 取得指定組建的屬性，或列出指定容器登錄的所有組建。 |
> | 動作 | Microsoft.ContainerRegistry/registries/builds/write | 使用指定參數更新容器登錄的組建。 |
> | 動作 | Microsoft.ContainerRegistry/registries/buildTasks/delete | 刪除容器登錄中的組建工作。 |
> | 動作 | Microsoft.ContainerRegistry/registries/buildTasks/listSourceRepositoryProperties/action | 列出組建工作的原始檔控制屬性。 |
> | 動作 | Microsoft.ContainerRegistry/registries/buildTasks/read | 取得指定組建工作的屬性，或列出指定容器登錄的所有組建工作。 |
> | 動作 | Microsoft.ContainerRegistry/registries/buildTasks/steps/delete | 從組建工作中刪除組建步驟。 |
> | 動作 | Microsoft.ContainerRegistry/registries/buildTasks/steps/listBuildArguments/action | 列出組建步驟的組建引數，包括祕密引數。 |
> | 動作 | Microsoft.ContainerRegistry/registries/buildTasks/steps/read | 取得指定組建步驟的屬性，或列出指定組建工作的所有組建步驟。 |
> | 動作 | Microsoft.ContainerRegistry/registries/buildTasks/steps/write | 使用指定參數建立或更新組建工作的組建步驟。 |
> | 動作 | Microsoft.ContainerRegistry/registries/buildTasks/write | 使用指定參數建立或更新容器登錄的組建工作。 |
> | 動作 | Microsoft.ContainerRegistry/registries/delete | 刪除容器登錄。 |
> | 動作 | Microsoft.ContainerRegistry/registries/eventGridFilters/delete | 從容器登錄中刪除事件格線篩選。 |
> | 動作 | Microsoft.ContainerRegistry/registries/eventGridFilters/read | 取得指定事件格線篩選的屬性，或列出指定容器登錄的所有事件格線篩選。 |
> | 動作 | Microsoft.ContainerRegistry/registries/eventGridFilters/write | 使用指定參數，建立或更新容器登錄的事件格線篩選。 |
> | 動作 | Microsoft.ContainerRegistry/registries/getBuildSourceUploadUrl/action | 取得上傳位置，讓使用者可上傳來源。 |
> | 動作 | Microsoft.ContainerRegistry/registries/importImage/action | 使用指定參數將映像匯入到容器登錄。 |
> | 動作 | Microsoft.ContainerRegistry/registries/listCredentials/action | 列出指定容器登錄的登入認證。 |
> | 動作 | Microsoft.ContainerRegistry/registries/listPolicies/read | 列出指定容器登錄的原則 |
> | 動作 | Microsoft.ContainerRegistry/registries/listUsages/read | 列出指定容器登錄的配額使用量。 |
> | 動作 | Microsoft.ContainerRegistry/registries/operationStatuses/read | 取得登錄非同步作業狀態 |
> | 動作 | Microsoft.ContainerRegistry/registries/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.ContainerRegistry/registries/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.ContainerRegistry/registries/providers/Microsoft.Insights/metricDefinitions/read | 取得 Microsoft ContainerRegistry 的可用計量 |
> | 動作 | Microsoft.ContainerRegistry/registries/pull/read | 從容器登錄中提取或取得映像。 |
> | 動作 | Microsoft.ContainerRegistry/registries/push/write | 將映像推送或寫入至容器登錄。 |
> | 動作 | Microsoft.ContainerRegistry/registries/quarantineRead/read | 從容器登錄中提取或取得隔離的映像 |
> | 動作 | Microsoft.ContainerRegistry/registries/quarantineWrite/write | 寫入/修改已隔離映像的隔離狀態 |
> | 動作 | Microsoft.ContainerRegistry/registries/queueBuild/action | 根據要求參數建立新的組建，並將它新增至組建佇列。 |
> | 動作 | Microsoft.ContainerRegistry/registries/read | 取得指定容器登錄的屬性，或列出指定資源群組或訂用帳戶下的所有容器登錄。 |
> | 動作 | Microsoft.ContainerRegistry/registries/regenerateCredential/action | 針對指定容器登錄重新產生一個登入認證。 |
> | 動作 | Microsoft.ContainerRegistry/registries/replications/delete | 刪除容器登錄中的複寫。 |
> | 動作 | Microsoft.ContainerRegistry/registries/replications/operationStatuses/read | 取得複寫非同步作業狀態 |
> | 動作 | Microsoft.ContainerRegistry/registries/replications/read | 取得指定複寫的屬性，或列出指定容器登錄的所有複寫。 |
> | 動作 | Microsoft.ContainerRegistry/registries/replications/write | 使用指定參數，建立或更新容器登錄的複寫。 |
> | 動作 | Microsoft.ContainerRegistry/registries/sign/write | 推送/提取容器登錄的內容信任中繼資料。 |
> | 動作 | Microsoft.ContainerRegistry/registries/updatePolicies/write | 更新指定容器登錄的原則 |
> | 動作 | Microsoft.ContainerRegistry/registries/webhooks/delete | 刪除容器登錄中的 Webhook。 |
> | 動作 | Microsoft.ContainerRegistry/registries/webhooks/getCallbackConfig/action | 針對 Webhook 取得服務 URI 的設定和自訂標頭。 |
> | 動作 | Microsoft.ContainerRegistry/registries/webhooks/listEvents/action | 針對指定的 Webhook 列出最近的事件。 |
> | 動作 | Microsoft.ContainerRegistry/registries/webhooks/operationStatuses/read | 取得 Webhook 非同步作業狀態 |
> | 動作 | Microsoft.ContainerRegistry/registries/webhooks/ping/action | 觸發要傳送到 Webhook 的 Ping 事件。 |
> | 動作 | Microsoft.ContainerRegistry/registries/webhooks/read | 取得指定 Webhook 的屬性，或列出指定容器登錄的所有 Webhook。 |
> | 動作 | Microsoft.ContainerRegistry/registries/webhooks/write | 使用指定參數，建立或更新容器登錄的 Webhook。 |
> | 動作 | Microsoft.ContainerRegistry/registries/write | 使用指定參數，建立或更新容器登錄。 |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ContainerService/containerServices/delete | 刪除容器服務 |
> | 動作 | Microsoft.ContainerService/containerServices/read | 取得容器服務 |
> | 動作 | Microsoft.ContainerService/containerServices/write | 建立新的容器服務，或更新現有的容器服務 |
> | 動作 | Microsoft.ContainerService/locations/operationresults/read | 取得非同步作業結果的狀態 |
> | 動作 | Microsoft.ContainerService/locations/operations/read | 取得非同步作業的狀態 |
> | 動作 | Microsoft.ContainerService/locations/orchestrators/read | 列出支援的協調器 |
> | 動作 | Microsoft.ContainerService/managedClusters/accessProfiles/listCredential/action | 使用清單認證依角色名稱取得受控叢集存取設定檔 |
> | 動作 | Microsoft.ContainerService/managedClusters/accessProfiles/read | 依角色名稱取得受控叢集存取設定檔 |
> | 動作 | Microsoft.ContainerService/managedClusters/delete | 刪除受控叢集 |
> | 動作 | Microsoft.ContainerService/managedClusters/listClusterAdminCredential/action | 列出受控叢集的 clusterAdmin 認證 |
> | 動作 | Microsoft.ContainerService/managedClusters/listClusterUserCredential/action | 列出受控叢集的 clusterUser 認證 |
> | 動作 | Microsoft.ContainerService/managedClusters/providers/Microsoft.Insights/diagnosticSettings/read | 取得受控叢集資源的診斷設定 |
> | 動作 | Microsoft.ContainerService/managedClusters/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新受控叢集資源的診斷設定 |
> | 動作 | Microsoft.ContainerService/managedClusters/providers/Microsoft.Insights/logDefinitions/read | 取得受控叢集的可用記錄 |
> | 動作 | Microsoft.ContainerService/managedClusters/providers/Microsoft.Insights/metricDefinitions/read | 取得受控叢集的可用計量 |
> | 動作 | Microsoft.ContainerService/managedClusters/read | 取得受控叢集 |
> | 動作 | Microsoft.ContainerService/managedClusters/upgradeprofiles/read | 取得叢集的升級設定檔 |
> | 動作 | Microsoft.ContainerService/managedClusters/write | 建立新的受控叢集，或更新現有的受控叢集 |
> | 動作 | Microsoft.ContainerService/operations/read | 列出 Microsoft.ContainerService 資源提供者的可用作業 |
> | 動作 | Microsoft.ContainerService/register/action | 向 Microsoft.ContainerService 資源提供者註冊訂用帳戶 |
> | 動作 | Microsoft.ContainerService/unregister/action | 向 Microsoft.ContainerService 資源提供者取消註冊訂用帳戶 |

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ContentModerator/applications/delete | 刪除作業 |
> | 動作 | Microsoft.ContentModerator/applications/listSecrets/action | 列出祕密 |
> | 動作 | Microsoft.ContentModerator/applications/listSingleSignOnToken/action | 讀取單一登入權杖 |
> | 動作 | Microsoft.ContentModerator/applications/read | 讀取作業 |
> | 動作 | Microsoft.ContentModerator/applications/write | 寫入作業 |
> | 動作 | Microsoft.ContentModerator/applications/write | 寫入作業 |
> | 動作 | Microsoft.ContentModerator/listCommunicationPreference/action | 列出通訊喜好設定 |
> | 動作 | Microsoft.ContentModerator/operations/read | 讀取作業 |
> | 動作 | Microsoft.ContentModerator/updateCommunicationPreference/action | 更新通訊喜好設定 |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.CostManagement/dimensions/read | 依範圍列出所有支援的維度。 |
> | 動作 | Microsoft.CostManagement/query/action | 依範圍查使用狀況資料。 |
> | 動作 | Microsoft.CostManagement/query/read | 依範圍查使用狀況資料。 |
> | 動作 | Microsoft.CostManagement/reports/action | 依範圍排定使用狀況資料報告。 |
> | 動作 | Microsoft.CostManagement/reports/read | 依範圍排定使用狀況資料報告。 |

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.CustomerInsights/hubs/adobemetadata/action | 建立或更新任何 Azure Customer Insights Adobe 中繼資料 |
> | 動作 | Microsoft.CustomerInsights/hubs/adobemetadata/read | 讀取任何 Azure Customer Insights Adobe 中繼資料 |
> | 動作 | Microsoft.CustomerInsights/hubs/authorizationPolicies/delete | 刪除任何 Azure Customer Insights 共用存取簽章原則 |
> | 動作 | Microsoft.CustomerInsights/hubs/authorizationPolicies/read | 讀取任何 Azure Customer Insights 共用存取簽章原則 |
> | 動作 | Microsoft.CustomerInsights/hubs/authorizationPolicies/regeneratePrimaryKey/action | 重新產生 Azure Customer Insights 共用存取簽章原則的主要金鑰 |
> | 動作 | Microsoft.CustomerInsights/hubs/authorizationPolicies/regenerateSecondaryKey/action | 重新產生 Azure Customer Insights 共用存取簽章原則的次要金鑰 |
> | 動作 | Microsoft.CustomerInsights/hubs/authorizationPolicies/write | 建立或更新任何 Azure Customer Insights 共用存取簽章原則 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/activate/action | 啟動任何 Azure Customer Insights 連接器 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/activate/action | 啟動任何 Azure Customer Insights 連接器 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/delete | 刪除任何 Azure Customer Insights 連接器 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/getruntimestatus/action | 取得任何 Azure Customer Insights 連接器執行階段狀態 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/mappings/activate/action | 啟動任何 Azure Customer Insights 連接器對應 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/mappings/delete | 刪除任何 Azure Customer Insights 連接器對應 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/mappings/operations/read | 讀取任何 Azure Customer Insights 連接器對應作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/mappings/read | 讀取任何 Azure Customer Insights 連接器對應 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/mappings/write | 建立或更新任何 Azure Customer Insights 連接器對應 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/operations/read | 讀取任何 Azure Customer Insights 連接器作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/read | 讀取任何 Azure Customer Insights 連接器 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/saveauthinfo/action | 建立或更新任何 Azure Customer Insights 連接器驗證資訊 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/update/action | 更新任何 Azure Customer Insights 連接器 |
> | 動作 | Microsoft.CustomerInsights/hubs/connectors/write | 建立或更新任何 Azure Customer Insights 連接器 |
> | 動作 | Microsoft.CustomerInsights/hubs/crmmetadata/action | 建立或更新任何 Azure Customer Insights Crm 中繼資料 |
> | 動作 | Microsoft.CustomerInsights/hubs/crmmetadata/read | 讀取任何 Azure Customer Insights Crm 中繼資料 |
> | 動作 | Microsoft.CustomerInsights/hubs/delete | 刪除任何 Azure Customer Insights 中樞 |
> | 動作 | Microsoft.CustomerInsights/hubs/gdpr/delete | 刪除任何 Azure Customer Insights Gdpr |
> | 動作 | Microsoft.CustomerInsights/hubs/gdpr/read | 讀取任何 Azure Customer Insights Gdpr |
> | 動作 | Microsoft.CustomerInsights/hubs/gdpr/write | 建立或更新任何 Azure Customer Insights Gdpr |
> | 動作 | Microsoft.CustomerInsights/hubs/getbillingcredits/read | 取得 Azure Customer Insights 中樞帳單信用額度 |
> | 動作 | Microsoft.CustomerInsights/hubs/getbillinghistory/read | 取得 Azure Customer Insights 中樞帳單記錄 |
> | 動作 | Microsoft.CustomerInsights/hubs/images/delete | 刪除任何 Azure Customer Insights 映像 |
> | 動作 | Microsoft.CustomerInsights/hubs/images/read | 讀取任何 Azure Customer Insights 映像 |
> | 動作 | Microsoft.CustomerInsights/hubs/images/write | 建立或更新任何 Azure Customer Insights 映像 |
> | 動作 | Microsoft.CustomerInsights/hubs/interactions/delete | 刪除任何 Azure Customer Insights 互動 |
> | 動作 | Microsoft.CustomerInsights/hubs/interactions/operations/read | 讀取任何 Azure Customer Insights 互動作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/interactions/read | 讀取任何 Azure Customer Insights 互動 |
> | 動作 | Microsoft.CustomerInsights/hubs/interactions/suggestrelationshiplinks/action | 提供適用於任何 Azure Customer Insights 互動的關聯連結建議 |
> | 動作 | Microsoft.CustomerInsights/hubs/interactions/write | 建立或更新任何 Azure Customer Insights 互動 |
> | 動作 | Microsoft.CustomerInsights/hubs/kpi/delete | 刪除任何 Azure Customer Insights 關鍵效能指標 |
> | 動作 | Microsoft.CustomerInsights/hubs/kpi/operations/read | 讀取任何 Azure Customer Insights 關鍵效能指標作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/kpi/read | 讀取任何 Azure Customer Insights 關鍵效能指標 |
> | 動作 | Microsoft.CustomerInsights/hubs/kpi/reprocess/action | 重新處理任何 Azure Customer Insights 關鍵效能指標 |
> | 動作 | Microsoft.CustomerInsights/hubs/kpi/write | 建立或更新任何 Azure Customer Insights 關鍵效能指標 |
> | 動作 | Microsoft.CustomerInsights/hubs/links/delete | 刪除任何 Azure Customer Insights 連結 |
> | 動作 | Microsoft.CustomerInsights/hubs/links/operations/read | 讀取任何 Azure Customer Insights 連結作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/links/read | 讀取任何 Azure Customer Insights 連結 |
> | 動作 | Microsoft.CustomerInsights/hubs/links/write | 建立或更新任何 Azure Customer 連結 |
> | 動作 | Microsoft.CustomerInsights/hubs/msemetadata/action | 建立或更新任何 Azure Customer Insights Mse 中繼資料 |
> | 動作 | Microsoft.CustomerInsights/hubs/msemetadata/read | 讀取任何 Azure Customer Insights Mse 中繼資料 |
> | 動作 | Microsoft.CustomerInsights/hubs/operationresults/read | 取得 Azure Customer Insights 中樞作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/predictions/delete | 刪除任何 Azure Customer Insights 預測 |
> | 動作 | Microsoft.CustomerInsights/hubs/predictions/operations/read | 讀取任何 Azure Customer Insights 預測作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/predictions/read | 讀取任何 Azure Customer Insights 預測 |
> | 動作 | Microsoft.CustomerInsights/hubs/predictions/write | 建立或更新任何 Azure Customer 預測 |
> | 動作 | Microsoft.CustomerInsights/hubs/predictivematchpolicies/delete | 刪除任何 Azure Customer Insights 預測比對原則 |
> | 動作 | Microsoft.CustomerInsights/hubs/predictivematchpolicies/operations/read | 讀取任何 Azure Customer Insights 預測比對原則作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/predictivematchpolicies/read | 讀取任何 Azure Customer Insights 預測比對原則 |
> | 動作 | Microsoft.CustomerInsights/hubs/predictivematchpolicies/write | 建立或更新任何 Azure Customer Insights 預測比對原則 |
> | 動作 | Microsoft.CustomerInsights/hubs/profiles/delete | 刪除任何 Azure Customer Insights 設定檔 |
> | 動作 | Microsoft.CustomerInsights/hubs/profiles/operations/read | 讀取任何 Azure Customer Insights 設定檔作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/profiles/read | 讀取任何 Azure Customer Insights 設定檔 |
> | 動作 | Microsoft.CustomerInsights/hubs/profiles/write | 寫入任何 Azure Customer Insights 設定檔 |
> | 動作 | Microsoft.CustomerInsights/hubs/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.CustomerInsights/hubs/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.CustomerInsights/hubs/providers/Microsoft.Insights/logDefinitions/read | 取得資源的可用記錄 |
> | 動作 | Microsoft.CustomerInsights/hubs/providers/Microsoft.Insights/metricDefinitions/read | 取得資源的可用計量 |
> | 動作 | Microsoft.CustomerInsights/hubs/read | 讀取任何 Azure Customer Insights 中樞 |
> | 動作 | Microsoft.CustomerInsights/hubs/relationshiplinks/delete | 刪除任何 Azure Customer Insights 關聯連結 |
> | 動作 | Microsoft.CustomerInsights/hubs/relationshiplinks/operations/read | 讀取任何 Azure Customer Insights 關聯連結作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/relationshiplinks/read | 讀取任何 Azure Customer Insights 關聯連結 |
> | 動作 | Microsoft.CustomerInsights/hubs/relationshiplinks/write | 建立或更新任何 Azure Customer Insights 關聯連結 |
> | 動作 | Microsoft.CustomerInsights/hubs/relationships/delete | 刪除任何 Azure Customer Insights 關聯 |
> | 動作 | Microsoft.CustomerInsights/hubs/relationships/operations/read | 讀取任何 Azure Customer Insights 關聯作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/relationships/read | 讀取任何 Azure Customer Insights 關聯 |
> | 動作 | Microsoft.CustomerInsights/hubs/relationships/write | 建立或更新任何 Azure Customer Insights 關聯 |
> | 動作 | Microsoft.CustomerInsights/hubs/roleAssignments/delete | 刪除任何 Azure Customer Insights RBAC 指派 |
> | 動作 | Microsoft.CustomerInsights/hubs/roleAssignments/operations/read | 讀取任何 Azure Customer Insights RBAC 指派作業結果 |
> | 動作 | Microsoft.CustomerInsights/hubs/roleAssignments/read | 讀取任何 Azure Customer Insights RBAC 指派 |
> | 動作 | Microsoft.CustomerInsights/hubs/roleAssignments/write | 建立或更新任何 Azure Customer Insights RBAC 指派 |
> | 動作 | Microsoft.CustomerInsights/hubs/roles/read | 讀取任何 Azure Customer Insights RBAC 角色 |
> | 動作 | Microsoft.CustomerInsights/hubs/salesforcemetadata/action | 建立或更新任何 Azure Customer Insights SalesForce 中繼資料 |
> | 動作 | Microsoft.CustomerInsights/hubs/salesforcemetadata/read | 讀取任何 Azure Customer Insights SalesForce 中繼資料 |
> | 動作 | Microsoft.CustomerInsights/hubs/segments/delete | 刪除任何 Azure Customer Insights 區段 |
> | 動作 | Microsoft.CustomerInsights/hubs/segments/dynamic/action | 管理任何 Azure Customer Insight 動態區段 |
> | 動作 | Microsoft.CustomerInsights/hubs/segments/read | 讀取任何 Azure Customer Insights 區段 |
> | 動作 | Microsoft.CustomerInsights/hubs/segments/static/action | 管理任何 Azure Customer Insight 靜態區段 |
> | 動作 | Microsoft.CustomerInsights/hubs/segments/write | 建立或更新任何 Azure Customer Insights 區段 |
> | 動作 | Microsoft.CustomerInsights/hubs/sqlconnectionstrings/delete | 刪除任何 Azure Customer Insights SqlConnectionStrings |
> | 動作 | Microsoft.CustomerInsights/hubs/sqlconnectionstrings/read | 讀取任何 Azure Customer Insights SqlConnectionStrings |
> | 動作 | Microsoft.CustomerInsights/hubs/sqlconnectionstrings/write | 建立或更新任何 Azure Customer Insights SqlConnectionStrings |
> | 動作 | Microsoft.CustomerInsights/hubs/suggesttypeschema/action | 從範例資料產生建議類型結構描述 |
> | 動作 | Microsoft.CustomerInsights/hubs/tenantmanagement/read | 管理任何 Azure Customer Insights 中樞設定 |
> | 動作 | Microsoft.CustomerInsights/hubs/views/delete | 刪除任何 Azure Customer Insights 應用程式檢視 |
> | 動作 | Microsoft.CustomerInsights/hubs/views/read | 讀取任何 Azure Customer Insights 應用程式檢視 |
> | 動作 | Microsoft.CustomerInsights/hubs/views/write | 建立或更新任何 Azure Customer Insights 應用程式檢視 |
> | 動作 | Microsoft.CustomerInsights/hubs/widgettypes/read | 讀取任何 Azure Customer Insights 應用程式小工具類型 |
> | 動作 | Microsoft.CustomerInsights/hubs/write | 建立或更新任何 Azure Customer Insights 中樞 |
> | 動作 | Microsoft.CustomerInsights/operations/read | 讀取 Azure Customer Insights API 中繼資料 |
> | 動作 | Microsoft.CustomerInsights/register/action | 為 Customer Insights 資源提供者註冊訂用帳戶，並讓您能夠建立 Customer Insights 資源 |
> | 動作 | Microsoft.CustomerInsights/unregister/action | 為 Customer Insights 資源提供者取消註冊訂用帳戶 |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DataBox/jobs/bookShipmentPickUp/action | 允許預約退貨商品的取貨。 |
> | 動作 | Microsoft.DataBox/jobs/cancel/action | 取消進行中的訂單。 |
> | 動作 | Microsoft.DataBox/jobs/delete | 刪除訂單 |
> | 動作 | Microsoft.DataBox/jobs/listCredentials/action | 列出與訂單相關的未加密認證。 |
> | 動作 | Microsoft.DataBox/jobs/read | 列出或取得訂單 |
> | 動作 | Microsoft.DataBox/jobs/write | 建立或更新訂單 |
> | 動作 | Microsoft.DataBox/locations/availableSkus/action | 此方法會傳回可用的 SKU 清單。 |
> | 動作 | Microsoft.DataBox/locations/validateAddress/action | 驗證出貨地址，並提供備用的地址 (若有的話)。 |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/alerts/read | 列出或取得警示 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/alerts/read | 列出或取得警示 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/bandwidthSchedules/delete | 刪除頻寬排程 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/bandwidthSchedules/read | 列出或取得頻寬排程 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/bandwidthSchedules/read | 列出或取得頻寬排程 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/bandwidthSchedules/write | 建立或更新頻寬排程 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/delete | 刪除 Data Box Edge 裝置 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/downloadUpdates/action | 在裝置中下載更新 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/extendedInformation/action | 擷取資源延伸資訊 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/extendedInformation/write | 建立或更新資源延伸資訊 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/installUpdates/action | 在裝置上安裝更新 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/jobs/read | 列出或取得作業 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/networkSettings/read | 列出或取得裝置網路設定 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/providers/Microsoft.Insights/metricDefinitions/read | 取得可用的 Data Box Edge 裝置層級計量 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/read | 列出或取得 Data Box Edge 裝置 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/read | 列出或取得 Data Box Edge 裝置 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/read | 列出或取得 Data Box Edge 裝置 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/roles/delete | 刪除 ArmApiRes_roles |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/roles/read | 列出或取得 ArmApiRes_roles |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/roles/read | 列出或取得 ArmApiRes_roles |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/roles/write | 建立或更新 ArmApiRes_roles |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/scanForUpdates/action | 掃描更新 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/securitySettings/update/action | 更新安全性設定 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/shares/delete | 刪除共用 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/shares/read | 列出或取得共用 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/shares/read | 列出或取得共用 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/shares/refresh/action | ArmApiDesc_action_refresh_shares |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/shares/write | 建立或更新共用 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/storageAccountCredentials/delete | 刪除儲存體帳戶認證 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/storageAccountCredentials/read | 列出或取得儲存體帳戶認證 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/storageAccountCredentials/read | 列出或取得儲存體帳戶認證 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/storageAccountCredentials/write | 建立或更新儲存體帳戶認證 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/updateSummary/read | 列出或取得更新摘要 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/uploadCertificate/action | 上傳裝置註冊的憑證 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/users/delete | 刪除共用使用者 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/users/read | 列出或取得共用使用者 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/users/read | 列出或取得共用使用者 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/users/write | 建立或更新共用使用者 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/write | 建立或更新 Data Box Edge 裝置 |
> | 動作 | Microsoft.DataBoxEdge/dataBoxEdgeDevices/write | 建立或更新 Data Box Edge 裝置 |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Databricks/workspaces/delete | 移除工作區。 |
> | 動作 | Microsoft.Databricks/workspaces/read | 擷取工作區的清單。 |
> | 動作 | Microsoft.Databricks/workspaces/write | 建立工作區。 |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DataCatalog/catalogs/delete | 刪除目錄。 |
> | 動作 | Microsoft.DataCatalog/catalogs/read | 取得訂用帳戶或資源群組下一或多個目錄的屬性。 |
> | 動作 | Microsoft.DataCatalog/catalogs/write | 建立目錄，或更新目錄的標記和屬性。 |
> | 動作 | Microsoft.DataCatalog/checkNameAvailability/action | 檢查租用戶的目錄名稱可用性。 |
> | 動作 | Microsoft.DataCatalog/operations/read | 列出可以對 Microsoft.DataCatalog 資源提供者執行的作業。 |
> | 動作 | Microsoft.DataCatalog/register/action | 向 Microsoft.DataCatalog 資源提供者註冊訂用帳戶。 |
> | 動作 | Microsoft.DataCatalog/unregister/action | 從 Microsoft.DataCatalog 資源提供者取消註冊訂用帳戶。 |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DataFactory/datafactories/activitywindows/read | 使用指定參數讀取 Data Factory 中的活動視窗。 |
> | 動作 | Microsoft.DataFactory/datafactories/datapipelines/activities/activitywindows/read | 使用指定參數讀取管線活動的活動視窗。 |
> | 動作 | Microsoft.DataFactory/datafactories/datapipelines/activitywindows/read | 使用指定參數讀取管線的活動視窗。 |
> | 動作 | Microsoft.DataFactory/datafactories/datapipelines/delete | 刪除任何管線。 |
> | 動作 | Microsoft.DataFactory/datafactories/datapipelines/pause/action | 暫停任何管線。 |
> | 動作 | Microsoft.DataFactory/datafactories/datapipelines/read | 讀取任何管線。 |
> | 動作 | Microsoft.DataFactory/datafactories/datapipelines/resume/action | 繼續任何管線。 |
> | 動作 | Microsoft.DataFactory/datafactories/datapipelines/update/action | 更新任何管線。 |
> | 動作 | Microsoft.DataFactory/datafactories/datapipelines/write | 建立或更新任何管線。 |
> | 動作 | Microsoft.DataFactory/datafactories/datasets/activitywindows/read | 使用指定參數讀取資料集的活動視窗。 |
> | 動作 | Microsoft.DataFactory/datafactories/datasets/delete | 刪除任何資料集。 |
> | 動作 | Microsoft.DataFactory/datafactories/datasets/read | 讀取任何資料集。 |
> | 動作 | Microsoft.DataFactory/datafactories/datasets/sliceruns/read | 針對具有指定開始時間的指定資料集讀取資料配量執行。 |
> | 動作 | Microsoft.DataFactory/datafactories/datasets/slices/read | 取得指定期間內的資料配量。 |
> | 動作 | Microsoft.DataFactory/datafactories/datasets/slices/write | 更新資料配量的狀態。 |
> | 動作 | Microsoft.DataFactory/datafactories/datasets/write | 建立或更新任何資料集。 |
> | 動作 | Microsoft.DataFactory/datafactories/delete | 刪除 Data Factory。 |
> | 動作 | Microsoft.DataFactory/datafactories/gateways/connectioninfo/action | 讀取任何閘道的連線資訊。 |
> | 動作 | Microsoft.DataFactory/datafactories/gateways/delete | 刪除任何閘道。 |
> | 動作 | Microsoft.DataFactory/datafactories/gateways/listauthkeys/action | 列出任何閘道的驗證金鑰。 |
> | 動作 | Microsoft.DataFactory/datafactories/gateways/read | 讀取任何閘道。 |
> | 動作 | Microsoft.DataFactory/datafactories/gateways/regenerateauthkey/action | 重新產生任何閘道的驗證金鑰。 |
> | 動作 | Microsoft.DataFactory/datafactories/gateways/write | 建立或更新任何閘道。 |
> | 動作 | Microsoft.DataFactory/datafactories/linkedServices/delete | 刪除任何連結服務。 |
> | 動作 | Microsoft.DataFactory/datafactories/linkedServices/read | 讀取任何連結服務。 |
> | 動作 | Microsoft.DataFactory/datafactories/linkedServices/write | 建立或更新任何連結服務。 |
> | 動作 | Microsoft.DataFactory/datafactories/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.DataFactory/datafactories/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.DataFactory/datafactories/providers/Microsoft.Insights/metricDefinitions/read | 取得 Datafactory 的可用計量 |
> | 動作 | Microsoft.DataFactory/datafactories/read | 讀取 Data Factory。 |
> | 動作 | Microsoft.DataFactory/datafactories/runs/loginfo/read | 讀取包含記錄的 Blob 容器 SAS URI。 |
> | 動作 | Microsoft.DataFactory/datafactories/tables/delete | 刪除任何資料集。 |
> | 動作 | Microsoft.DataFactory/datafactories/tables/read | 讀取任何資料集。 |
> | 動作 | Microsoft.DataFactory/datafactories/tables/write | 建立或更新任何資料集。 |
> | 動作 | Microsoft.DataFactory/datafactories/write | 建立或更新 Data Factory。 |
> | 動作 | Microsoft.DataFactory/factories/cancelpipelinerun/action | 取消執行識別碼所指定的管線執行。 |
> | 動作 | Microsoft.DataFactory/factories/datasets/delete | 刪除任何資料集。 |
> | 動作 | Microsoft.DataFactory/factories/datasets/read | 讀取任何資料集。 |
> | 動作 | Microsoft.DataFactory/factories/datasets/write | 建立或更新任何資料集。 |
> | 動作 | Microsoft.DataFactory/factories/delete | 刪除 Data Factory。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/delete | 刪除任何 Integration Runtime。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/getconnectioninfo/read | 讀取 Integration Runtime 連線資訊。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/getstatus/read | 讀取 Integration Runtime 狀態。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/listauthkeys/read | 列出任何 Integration Runtime 的驗證金鑰。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/monitoringdata/read | 取得任何 Integration Runtime 的監視資料。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/nodes/delete | 刪除所指定 Integration Runtime 的節點。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/nodes/ipAddress/action | 傳回 Integration Runtime 所指定節點的 IP 位址。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/nodes/write | 更新自我裝載的 Integration Runtime 節點。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/read | 讀取任何 Integration Runtime。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/regenerateauthkey/action | 重新產生所指定 Integration Runtime 的驗證金鑰。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/start/action | 取動任何 Integration Runtime。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/stop/action | 停止任何 Integration Runtime。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/synccredentials/action | 同步處理所指定 Integration Runtime 的認證。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/upgrade/action | 將指定的 Integration Runtime 升級。 |
> | 動作 | Microsoft.DataFactory/factories/integrationruntimes/write | 建立或更新任何 Integration Runtime。 |
> | 動作 | Microsoft.DataFactory/factories/linkedServices/delete | 刪除連結服務。 |
> | 動作 | Microsoft.DataFactory/factories/linkedServices/read | 讀取連結服務。 |
> | 動作 | Microsoft.DataFactory/factories/linkedServices/write | 建立或更新連結服務 |
> | 動作 | Microsoft.DataFactory/factories/pipelineruns/activityruns/read | 讀取所指定管線執行識別碼的活動執行。 |
> | 動作 | Microsoft.DataFactory/factories/pipelineruns/cancel/action | 取消執行識別碼所指定的管線執行。 |
> | 動作 | Microsoft.DataFactory/factories/pipelineruns/queryactivityruns/action | 查詢所指定管線執行識別碼的活動執行。 |
> | 動作 | Microsoft.DataFactory/factories/pipelineruns/queryactivityruns/read | 讀取所指定管線執行識別碼的查詢活動執行結果。 |
> | 動作 | Microsoft.DataFactory/factories/pipelineruns/read | 讀取管線執行。 |
> | 動作 | Microsoft.DataFactory/factories/pipelines/createrun/action | 建立管線的執行。 |
> | 動作 | Microsoft.DataFactory/factories/pipelines/delete | 刪除管線。 |
> | 動作 | Microsoft.DataFactory/factories/pipelines/pipelineruns/activityruns/progress/read | 取得活動執行的進度。 |
> | 動作 | Microsoft.DataFactory/factories/pipelines/pipelineruns/read | 讀取管線執行。 |
> | 動作 | Microsoft.DataFactory/factories/pipelines/read | 讀取管線。 |
> | 動作 | Microsoft.DataFactory/factories/pipelines/write | 建立或更新管線 |
> | 動作 | Microsoft.DataFactory/factories/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.DataFactory/factories/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.DataFactory/factories/providers/Microsoft.Insights/logDefinitions/read | 取得處理站的可用記錄 |
> | 動作 | Microsoft.DataFactory/factories/providers/Microsoft.Insights/metricDefinitions/read | 取得處理站的可用計量 |
> | 動作 | Microsoft.DataFactory/factories/querypipelineruns/action | 查詢管線執行。 |
> | 動作 | Microsoft.DataFactory/factories/querypipelineruns/read | 讀取查詢管線執行的結果。 |
> | 動作 | Microsoft.DataFactory/factories/querytriggerruns/action | 查詢觸發程序執行。 |
> | 動作 | Microsoft.DataFactory/factories/querytriggerruns/read | 讀取觸發程序執行的結果。 |
> | 動作 | Microsoft.DataFactory/factories/read | 讀取 Data Factory。 |
> | 動作 | Microsoft.DataFactory/factories/triggerruns/read | 讀取觸發程序執行。 |
> | 動作 | Microsoft.DataFactory/factories/triggers/delete | 刪除任何觸發程序。 |
> | 動作 | Microsoft.DataFactory/factories/triggers/read | 讀取任何觸發程序。 |
> | 動作 | Microsoft.DataFactory/factories/triggers/start/action | 啟動任何觸發程序。 |
> | 動作 | Microsoft.DataFactory/factories/triggers/stop/action | 停止任何觸發程序。 |
> | 動作 | Microsoft.DataFactory/factories/triggers/triggerruns/read | 讀取觸發程序執行。 |
> | 動作 | Microsoft.DataFactory/factories/triggers/write | 建立或更新任何觸發程序。 |
> | 動作 | Microsoft.DataFactory/factories/write | 建立或更新 Data Factory |
> | 動作 | Microsoft.DataFactory/locations/configureFactoryRepo/action | 設定中心的存放庫。 |
> | 動作 | Microsoft.DataFactory/operations/read | 讀取 Microsoft Data Factory 提供者中的所有作業。 |
> | 動作 | Microsoft.DataFactory/register/action | 為 Data Factory 資源提供者註冊訂用帳戶。 |
> | 動作 | Microsoft.DataFactory/unregister/action | 為 Data Factory 資源提供者取消註冊訂用帳戶。 |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/computePolicies/delete | 刪除計算原則。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/computePolicies/read | 取得計算原則的相關資訊。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/computePolicies/write | 建立或更新計算原則。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/dataLakeStoreAccounts/delete | 取消 DataLakeStore 帳戶與 DataLakeAnalytics 帳戶的連結。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/dataLakeStoreAccounts/read | 取得 DataLakeAnalytics 帳戶所連結之 DataLakeStore 帳戶的相關資訊。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/dataLakeStoreAccounts/write | 建立或更新 DataLakeAnalytics 帳戶所連結的 DataLakeStore 帳戶。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/delete | 刪除 DataLakeAnalytics 帳戶。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/firewallRules/delete | 刪除防火牆規則。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/firewallRules/read | 取得防火牆規則的相關資訊。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/firewallRules/write | 建立或更新防火牆規則。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/operationResults/read | 取得 DataLakeAnalytics 帳戶作業的結果。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/providers/Microsoft.Insights/diagnosticSettings/read | 取得 DataLakeAnalytics 帳戶的診斷設定。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新 DataLakeAnalytics 帳戶的診斷設定。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/providers/Microsoft.Insights/logDefinitions/read | 取得 DataLakeAnalytics 帳戶的可用記錄。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/providers/Microsoft.Insights/metricDefinitions/read | 取得 DataLakeAnalytics 帳戶的可用計量。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/read | 取得現有 DataLakeAnalytics 帳戶的相關資訊。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/storageAccounts/Containers/listSasTokens/action | 針對 DataLakeAnalytics 帳戶所連結之儲存體帳戶的儲存體容器列出 SAS 權杖。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/storageAccounts/Containers/read | 取得 DataLakeAnalytics 帳戶所連結之儲存體帳戶的容器。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/storageAccounts/delete | 取消儲存體帳戶與 DataLakeAnalytics 帳戶的連結。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/storageAccounts/read | 取得 DataLakeAnalytics 帳戶所連結之儲存體帳戶的相關資訊。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/storageAccounts/write | 建立或更新 DataLakeAnalytics 帳戶所連結的儲存體帳戶。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/TakeOwnership/action | 授與權限以取消其他使用者所提交的作業。 |
> | 動作 | Microsoft.DataLakeAnalytics/accounts/write | 建立或更新 DataLakeAnalytics 帳戶。 |
> | 動作 | Microsoft.DataLakeAnalytics/locations/capability/read | 取得訂用帳戶中關於使用 DataLakeAnalytics 的功能資訊。 |
> | 動作 | Microsoft.DataLakeAnalytics/locations/checkNameAvailability/action | 檢查 DataLakeAnalytics 帳戶名稱的可用性。 |
> | 動作 | Microsoft.DataLakeAnalytics/locations/operationResults/read | 取得 DataLakeAnalytics 帳戶作業的結果。 |
> | 動作 | Microsoft.DataLakeAnalytics/operations/read | 取得 DataLakeAnalytics 的可用作業。 |
> | 動作 | Microsoft.DataLakeAnalytics/register/action | 向 DataLakeAnalytics 註冊訂用帳戶。 |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DataLakeStore/accounts/delete | 刪除 DataLakeStore 帳戶。 |
> | 動作 | Microsoft.DataLakeStore/accounts/enableKeyVault/action | 針對 DataLakeStore 帳戶啟用 KeyVault。 |
> | 動作 | Microsoft.DataLakeStore/accounts/eventGridFilters/delete | 刪除 EventGrid 篩選。 |
> | 動作 | Microsoft.DataLakeStore/accounts/eventGridFilters/read | 取得 EventGrid 篩選。 |
> | 動作 | Microsoft.DataLakeStore/accounts/eventGridFilters/write | 建立或更新 EventGrid 篩選。 |
> | 動作 | Microsoft.DataLakeStore/accounts/firewallRules/delete | 刪除防火牆規則。 |
> | 動作 | Microsoft.DataLakeStore/accounts/firewallRules/read | 取得防火牆規則的相關資訊。 |
> | 動作 | Microsoft.DataLakeStore/accounts/firewallRules/write | 建立或更新防火牆規則。 |
> | 動作 | Microsoft.DataLakeStore/accounts/operationResults/read | 取得 DataLakeStore 帳戶作業的結果。 |
> | 動作 | Microsoft.DataLakeStore/accounts/providers/Microsoft.Insights/diagnosticSettings/read | 取得 DataLakeStore 帳戶的診斷設定。 |
> | 動作 | Microsoft.DataLakeStore/accounts/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新 DataLakeStore 帳戶的診斷設定。 |
> | 動作 | Microsoft.DataLakeStore/accounts/providers/Microsoft.Insights/logDefinitions/read | 取得 DataLakeStore 帳戶的可用記錄。 |
> | 動作 | Microsoft.DataLakeStore/accounts/providers/Microsoft.Insights/metricDefinitions/read | 取得 DataLakeStore 帳戶的可用計量。 |
> | 動作 | Microsoft.DataLakeStore/accounts/read | 取得現有 DataLakeStore 帳戶的相關資訊。 |
> | 動作 | Microsoft.DataLakeStore/accounts/Superuser/action | 使用 Microsoft.Authorization/roleAssignments/write 授與時，授與 Data Lake Store 的超級使用者。 |
> | 動作 | Microsoft.DataLakeStore/accounts/trustedIdProviders/delete | 刪除所信任的識別提供者。 |
> | 動作 | Microsoft.DataLakeStore/accounts/trustedIdProviders/read | 取得所信任識別提供者的相關資訊。 |
> | 動作 | Microsoft.DataLakeStore/accounts/trustedIdProviders/write | 建立或更新所信任的識別提供者。 |
> | 動作 | Microsoft.DataLakeStore/accounts/write | 建立或更新 DataLakeStore 帳戶。 |
> | 動作 | Microsoft.DataLakeStore/locations/capability/read | 取得訂用帳戶中關於使用 DataLakeStore 的功能資訊。 |
> | 動作 | Microsoft.DataLakeStore/locations/checkNameAvailability/action | 檢查 DataLakeStore 帳戶名稱的可用性。 |
> | 動作 | Microsoft.DataLakeStore/locations/operationResults/read | 取得 DataLakeStore 帳戶作業的結果。 |
> | 動作 | Microsoft.DataLakeStore/operations/read | 取得 DataLakeStore 的可用作業。 |
> | 動作 | Microsoft.DataLakeStore/register/action | 向 DataLakeStore 註冊訂用帳戶。 |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DataMigration/locations/operationResults/read | 取得與「202 已接受」回應相關且長時間執行之作業的狀態 |
> | 動作 | Microsoft.DataMigration/locations/operationStatuses/read | 取得與「202 已接受」回應相關且長時間執行之作業的狀態 |
> | 動作 | Microsoft.DataMigration/register/action | 向「Azure 資料庫移轉服務」提供者註冊訂用帳戶 |
> | 動作 | Microsoft.DataMigration/services/checkStatus/action | 檢查服務是否已部署且在執行中 |
> | 動作 | Microsoft.DataMigration/services/delete | 刪除資源及其所有子系 |
> | 動作 | Microsoft.DataMigration/services/projects/accessArtifacts/action | 產生可用來對專案成品進行 GET 或 PUT 的 URL |
> | 動作 | Microsoft.DataMigration/services/projects/delete | 刪除資源及其所有子系 |
> | 動作 | Microsoft.DataMigration/services/projects/files/delete | 刪除資源及其所有子系 |
> | 動作 | Microsoft.DataMigration/services/projects/files/read | 閱讀資源的相關資訊 |
> | 動作 | Microsoft.DataMigration/services/projects/files/read/action | 取得可用來讀取檔案內容的 URL |
> | 動作 | Microsoft.DataMigration/services/projects/files/readWrite/action | 取得可用來讀取或寫入檔案內容的 URL |
> | 動作 | Microsoft.DataMigration/services/projects/files/write | 建立或更新資源及其屬性 |
> | 動作 | Microsoft.DataMigration/services/projects/read | 閱讀資源的相關資訊 |
> | 動作 | Microsoft.DataMigration/services/projects/tasks/cancel/action | 取消目前正在執行的工作 |
> | 動作 | Microsoft.DataMigration/services/projects/tasks/delete | 刪除資源及其所有子系 |
> | 動作 | Microsoft.DataMigration/services/projects/tasks/read | 閱讀資源的相關資訊 |
> | 動作 | Microsoft.DataMigration/services/projects/tasks/write | 執行工作「Azure 資料庫移轉服務」工作 |
> | 動作 | Microsoft.DataMigration/services/projects/write | 執行工作「Azure 資料庫移轉服務」工作 |
> | 動作 | Microsoft.DataMigration/services/read | 閱讀資源的相關資訊 |
> | 動作 | Microsoft.DataMigration/services/slots/delete | 刪除資源及其所有子系 |
> | 動作 | Microsoft.DataMigration/services/slots/read | 閱讀資源的相關資訊 |
> | 動作 | Microsoft.DataMigration/services/slots/write | 建立或更新資源及其屬性 |
> | 動作 | Microsoft.DataMigration/services/start/action | 啟動 DMS 服務以讓它再次處理移轉 |
> | 動作 | Microsoft.DataMigration/services/stop/action | 停止 DMS 服務以將其成本降到最低 |
> | 動作 | Microsoft.DataMigration/services/write | 建立或更新資源及其屬性 |
> | 動作 | Microsoft.DataMigration/skus/read | 取得 DMS 資源所支援的 SKU 清單。 |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DBforMariaDB/locations/performanceTiers/read | 傳回可用的效能層級清單。 |
> | 動作 | Microsoft.DBforMariaDB/performanceTiers/read | 傳回可用的效能層級清單。 |
> | 動作 | Microsoft.DBforMariaDB/servers/configurations/read | 傳回伺服器的組態清單，或取得指定組態的屬性。 |
> | 動作 | Microsoft.DBforMariaDB/servers/configurations/write | 更新指定組態的值 |
> | 動作 | Microsoft.DBforMariaDB/servers/delete | 刪除現有伺服器。 |
> | 動作 | Microsoft.DBforMariaDB/servers/firewallRules/delete | 刪除現有防火牆規則。 |
> | 動作 | Microsoft.DBforMariaDB/servers/firewallRules/read | 傳回伺服器的防火牆規則清單，或取得指定防火牆規則的屬性。 |
> | 動作 | Microsoft.DBforMariaDB/servers/firewallRules/write | 使用指定參數建立防火牆規則，或更新現有的規則。 |
> | 動作 | Microsoft.DBforMariaDB/servers/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.DBforMariaDB/servers/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.DBforMariaDB/servers/providers/Microsoft.Insights/metricDefinitions/read | 傳回可供資料庫使用之計量的類型 |
> | 動作 | Microsoft.DBforMariaDB/servers/read | 傳回伺服器清單，或取得指定伺服器的屬性。 |
> | 動作 | Microsoft.DBforMariaDB/servers/recoverableServers/read | 傳回可復原的 MariaDB 伺服器資訊 |
> | 動作 | Microsoft.DBforMariaDB/servers/updateConfigurations/action | 更新指定伺服器的組態 |
> | 動作 | Microsoft.DBforMariaDB/servers/virtualNetworkRules/delete | 刪除現有虛擬網路規則 |
> | 動作 | Microsoft.DBforMariaDB/servers/virtualNetworkRules/read | 傳回虛擬網路規則的清單，或取得指定虛擬網路規則的屬性。 |
> | 動作 | Microsoft.DBforMariaDB/servers/virtualNetworkRules/write | 使用指定參數建立虛擬網路規則，或更新指定虛擬網路規則的屬性或標記。 |
> | 動作 | Microsoft.DBforMariaDB/servers/write | 使用指定參數建立伺服器，或更新指定伺服器的屬性或標記。 |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DBforMySQL/locations/performanceTiers/read | 傳回可用的效能層級清單。 |
> | 動作 | Microsoft.DBforMySQL/performanceTiers/read | 傳回可用的效能層級清單。 |
> | 動作 | Microsoft.DBforMySQL/servers/configurations/read | 傳回伺服器的組態清單，或取得指定組態的屬性。 |
> | 動作 | Microsoft.DBforMySQL/servers/configurations/write | 更新指定組態的值 |
> | 動作 | Microsoft.DBforMySQL/servers/delete | 刪除現有伺服器。 |
> | 動作 | Microsoft.DBforMySQL/servers/firewallRules/delete | 刪除現有防火牆規則。 |
> | 動作 | Microsoft.DBforMySQL/servers/firewallRules/read | 傳回伺服器的防火牆規則清單，或取得指定防火牆規則的屬性。 |
> | 動作 | Microsoft.DBforMySQL/servers/firewallRules/write | 使用指定參數建立防火牆規則，或更新現有的規則。 |
> | 動作 | Microsoft.DBforMySQL/servers/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.DBforMySQL/servers/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.DBforMySQL/servers/providers/Microsoft.Insights/logDefinitions/read | 取得 MySQL 伺服器的可用記錄 |
> | 動作 | Microsoft.DBforMySQL/servers/providers/Microsoft.Insights/metricDefinitions/read | 傳回可供資料庫使用之計量的類型 |
> | 動作 | Microsoft.DBforMySQL/servers/read | 傳回伺服器清單，或取得指定伺服器的屬性。 |
> | 動作 | Microsoft.DBforMySQL/servers/recoverableServers/read | 傳回可復原的 MySQL 伺服器資訊 |
> | 動作 | Microsoft.DBforMySQL/servers/securityAlertPolicies/read | 擷取給定伺服器上所設定的伺服器威脅偵測原則詳細資料 |
> | 動作 | Microsoft.DBforMySQL/servers/securityAlertPolicies/write | 變更指定伺服器的伺服器威脅偵測原則 |
> | 動作 | Microsoft.DBforMySQL/servers/updateConfigurations/action | 更新指定伺服器的組態 |
> | 動作 | Microsoft.DBforMySQL/servers/virtualNetworkRules/delete | 刪除現有虛擬網路規則 |
> | 動作 | Microsoft.DBforMySQL/servers/virtualNetworkRules/read | 傳回虛擬網路規則的清單，或取得指定虛擬網路規則的屬性。 |
> | 動作 | Microsoft.DBforMySQL/servers/virtualNetworkRules/write | 使用指定參數建立虛擬網路規則，或更新指定虛擬網路規則的屬性或標記。 |
> | 動作 | Microsoft.DBforMySQL/servers/write | 使用指定參數建立伺服器，或更新指定伺服器的屬性或標記。 |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DBforPostgreSQL/locations/performanceTiers/read | 傳回可用的效能層級清單。 |
> | 動作 | Microsoft.DBforPostgreSQL/performanceTiers/read | 傳回可用的效能層級清單。 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/advisors/read | 傳回建議程式清單 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/advisors/recommendedActions/read | 傳回建議的動作清單 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/advisors/recommendedActionSessions/action | 提出建議 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/configurations/read | 傳回伺服器的組態清單，或取得指定組態的屬性。 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/configurations/write | 更新指定組態的值 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/delete | 刪除現有伺服器。 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/firewallRules/delete | 刪除現有防火牆規則。 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/firewallRules/read | 傳回伺服器的防火牆規則清單，或取得指定防火牆規則的屬性。 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/firewallRules/write | 使用指定參數建立防火牆規則，或更新現有的規則。 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/providers/Microsoft.Insights/logDefinitions/read | 取得 Postgres 伺服器的可用記錄 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/providers/Microsoft.Insights/metricDefinitions/read | 傳回可供資料庫使用之計量的類型 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/queryTexts/action | 傳回查詢的文字 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/read | 傳回伺服器清單，或取得指定伺服器的屬性。 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/recoverableServers/read | 傳回可復原的 PostgreSQL 伺服器資訊 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/read | 擷取給定伺服器上所設定的伺服器威脅偵測原則詳細資料 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/write | 變更指定伺服器的伺服器威脅偵測原則 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/topQueryStatistics/read | 傳回最高排名查詢的查詢統計資料清單。 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/updateConfigurations/action | 更新指定伺服器的組態 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/virtualNetworkRules/delete | 刪除現有虛擬網路規則 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/virtualNetworkRules/read | 傳回虛擬網路規則的清單，或取得指定虛擬網路規則的屬性。 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/virtualNetworkRules/write | 使用指定參數建立虛擬網路規則，或更新指定虛擬網路規則的屬性或標記。 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/waitStatistics/read | 傳回執行個體的等候統計資料 |
> | 動作 | Microsoft.DBforPostgreSQL/servers/write | 使用指定參數建立伺服器，或更新指定伺服器的屬性或標記。 |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Devices/checkNameAvailability/Action | 檢查 IotHub 名稱是否可供使用 |
> | 動作 | Microsoft.Devices/checkProvisioningServiceNameAvailability/Action | 確認佈建服務名稱是否可用 |
> | 動作 | Microsoft.Devices/ElasticPools/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Devices/ElasticPools/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/certificates/Delete | 刪除憑證 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/certificates/generateVerificationCode/Action | 產生驗證碼 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/certificates/Read | 取得憑證 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/certificates/verify/Action | 驗證憑證資源 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/certificates/Write | 建立或更新憑證 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/Delete | 刪除 IotHub 租用戶資源 |
> | 動作 | Microsoft.Devices/ElasticPools/IotHubTenants/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Devices/ElasticPools/IotHubTenants/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/eventHubEndpoints/consumerGroups/Delete | 刪除 EventHub 取用者群組 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/eventHubEndpoints/consumerGroups/Read | 取得 EventHub 取用者群組 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/eventHubEndpoints/consumerGroups/Write | 建立 EventHub 取用者群組 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/exportDevices/Action | 匯出裝置 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/getStats/Read | 取得 IotHub 租用戶統計資料資源 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/importDevices/Action | 匯入裝置 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/iotHubKeys/listkeys/Action | 取得 IotHub 租用戶金鑰 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/jobs/Read | 取得在給定的 IotHub 上所提交的作業詳細資料 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/listKeys/Action | 取得 IotHub 租用戶金鑰 |
> | 動作 | Microsoft.Devices/ElasticPools/IotHubTenants/logDefinitions/read | 取得 IotHub 服務的可用記錄定義 |
> | 動作 | Microsoft.Devices/ElasticPools/IotHubTenants/metricDefinitions/read | 取得 IotHub 服務的可用計量 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/quotaMetrics/Read | 取得配額計量 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/Read | 取得 IotHub 租用戶資源 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/routing/routes/$testall/Action | 針對所有現有路由來測試訊息 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/routing/routes/$testnew/Action | 針對所提供的測試路由來測試訊息 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/routingEndpointsHealth/Read | 取得 IotHub 之所有路由端點的健康狀態 |
> | 動作 | Microsoft.Devices/elasticPools/iotHubTenants/Write | 建立或更新 IotHub 租用戶資源 |
> | 動作 | Microsoft.Devices/ElasticPools/metricDefinitions/read | 取得 IotHub 服務的可用計量 |
> | 動作 | Microsoft.Devices/iotHubs/certificates/Delete | 刪除憑證 |
> | 動作 | Microsoft.Devices/iotHubs/certificates/generateVerificationCode/Action | 產生驗證碼 |
> | 動作 | Microsoft.Devices/iotHubs/certificates/Read | 取得憑證 |
> | 動作 | Microsoft.Devices/iotHubs/certificates/verify/Action | 驗證憑證資源 |
> | 動作 | Microsoft.Devices/iotHubs/certificates/Write | 建立或更新憑證 |
> | 動作 | Microsoft.Devices/iotHubs/Delete | 刪除 IotHub 資源 |
> | 動作 | Microsoft.Devices/IotHubs/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Devices/IotHubs/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Devices/iotHubs/eventGridFilters/Delete | 刪除事件格線篩選 |
> | 動作 | Microsoft.Devices/iotHubs/eventGridFilters/Read | 取得事件格線篩選 |
> | 動作 | Microsoft.Devices/iotHubs/eventGridFilters/Write | 建立新的或更新現有的事件格線篩選 |
> | 動作 | Microsoft.Devices/iotHubs/eventHubEndpoints/consumerGroups/Delete | 刪除 EventHub 取用者群組 |
> | 動作 | Microsoft.Devices/iotHubs/eventHubEndpoints/consumerGroups/Read | 取得 EventHub 取用者群組 |
> | 動作 | Microsoft.Devices/iotHubs/eventHubEndpoints/consumerGroups/Write | 建立 EventHub 取用者群組 |
> | 動作 | Microsoft.Devices/iotHubs/exportDevices/Action | 匯出裝置 |
> | 動作 | Microsoft.Devices/iotHubs/importDevices/Action | 匯入裝置 |
> | 動作 | Microsoft.Devices/iotHubs/iotHubKeys/listkeys/Action | 取得給定名稱的 IotHub 金鑰 |
> | 動作 | Microsoft.Devices/iotHubs/iotHubStats/Read | 取得 IotHub 統計資料 |
> | 動作 | Microsoft.Devices/iotHubs/jobs/Read | 取得在給定的 IotHub 上所提交的作業詳細資料 |
> | 動作 | Microsoft.Devices/iotHubs/listkeys/Action | 取得所有 IotHub 金鑰 |
> | 動作 | Microsoft.Devices/IotHubs/logDefinitions/read | 取得 IotHub 服務的可用記錄定義 |
> | 動作 | Microsoft.Devices/IotHubs/metricDefinitions/read | 取得 IotHub 服務的可用計量 |
> | 動作 | Microsoft.Devices/iotHubs/operationresults/Read | 取得作業結果 (過時的 API) |
> | 動作 | Microsoft.Devices/iotHubs/quotaMetrics/Read | 取得配額計量 |
> | 動作 | Microsoft.Devices/iotHubs/Read | 取得 IotHub 資源 |
> | 動作 | Microsoft.Devices/iotHubs/routing/$testall/Action | 針對所有現有路由來測試訊息 |
> | 動作 | Microsoft.Devices/iotHubs/routing/$testnew/Action | 針對所提供的測試路由來測試訊息 |
> | 動作 | Microsoft.Devices/iotHubs/routingEndpointsHealth/Read | 取得 IotHub 之所有路由端點的健康狀態 |
> | 動作 | Microsoft.Devices/iotHubs/skus/Read | 取得有效的 IotHub SKU |
> | 動作 | Microsoft.Devices/iotHubs/Write | 建立或更新 IotHub 資源 |
> | 動作 | Microsoft.Devices/locations/operationresults/Read | 取得以位置為基礎的作業結果 |
> | 動作 | Microsoft.Devices/operationresults/Read | 取得作業結果 |
> | 動作 | Microsoft.Devices/operations/Read | 取得所有 ResourceProvider 作業 |
> | 動作 | Microsoft.Devices/provisioningServices/certificates/Delete | 刪除憑證 |
> | 動作 | Microsoft.Devices/provisioningServices/certificates/generateVerificationCode/Action | 產生驗證碼 |
> | 動作 | Microsoft.Devices/provisioningServices/certificates/Read | 取得憑證 |
> | 動作 | Microsoft.Devices/provisioningServices/certificates/verify/Action | 驗證憑證資源 |
> | 動作 | Microsoft.Devices/provisioningServices/certificates/Write | 建立或更新憑證 |
> | 動作 | Microsoft.Devices/provisioningServices/Delete | 刪除 IotDps 資源 |
> | 動作 | Microsoft.Devices/provisioningServices/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Devices/provisioningServices/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Devices/provisioningServices/keys/listkeys/Action | 取得金鑰名稱的 IotDps 金鑰 |
> | 動作 | Microsoft.Devices/provisioningServices/listkeys/Action | 取得所有 IotDps 金鑰 |
> | 動作 | Microsoft.Devices/provisioningServices/logDefinitions/read | 取得佈建服務的可用記錄定義 |
> | 動作 | Microsoft.Devices/provisioningServices/metricDefinitions/read | 取得佈建服務的可用計量 |
> | 動作 | Microsoft.Devices/provisioningServices/operationresults/Read | 取得 DPS 作業結果 |
> | 動作 | Microsoft.Devices/provisioningServices/Read | 取得 IotDps 資源 |
> | 動作 | Microsoft.Devices/provisioningServices/skus/Read | 取得有效的 IotDps SKU |
> | 動作 | Microsoft.Devices/provisioningServices/Write | 建立 IotDps 資源 |
> | 動作 | Microsoft.Devices/register/action | 針對 IotHub 資源提供者註冊訂用帳戶，並讓您能夠建立 IotHub 資源 |
> | 動作 | Microsoft.Devices/register/action | 針對 IotHub 資源提供者註冊訂用帳戶，並讓您能夠建立 IotHub 資源 |
> | 動作 | Microsoft.Devices/usages/Read | 取得此提供者的訂用帳戶使用量詳細資料。 |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DevTestLab/labCenters/delete | 刪除實驗室中心。 |
> | 動作 | Microsoft.DevTestLab/labCenters/read | 讀取實驗室中心。 |
> | 動作 | Microsoft.DevTestLab/labCenters/write | 新增或修改實驗室中心。 |
> | 動作 | Microsoft.DevTestLab/labs/artifactSources/armTemplates/read | 讀取 Azure Resource Manager 範本。 |
> | 動作 | Microsoft.DevTestLab/labs/artifactSources/artifacts/GenerateArmTemplate/action | 產生給定構件的 ARM 範本，將所需的檔案上傳至儲存體帳戶，然後驗證所產生的構件。 |
> | 動作 | Microsoft.DevTestLab/labs/artifactSources/artifacts/read | 讀取構件。 |
> | 動作 | Microsoft.DevTestLab/labs/artifactSources/delete | 刪除構件來源。 |
> | 動作 | Microsoft.DevTestLab/labs/artifactSources/read | 讀取構件來源。 |
> | 動作 | Microsoft.DevTestLab/labs/artifactSources/write | 新增或修改構件來源。 |
> | 動作 | Microsoft.DevTestLab/labs/ClaimAnyVm/action | 在實驗室中宣告隨機的可宣告虛擬機器。 |
> | 動作 | Microsoft.DevTestLab/labs/costs/read | 讀取成本。 |
> | 動作 | Microsoft.DevTestLab/labs/costs/write | 新增或修改成本。 |
> | 動作 | Microsoft.DevTestLab/labs/CreateEnvironment/action | 在實驗室中建立虛擬機器。 |
> | 動作 | Microsoft.DevTestLab/labs/customImages/delete | 刪除自訂映像。 |
> | 動作 | Microsoft.DevTestLab/labs/customImages/read | 讀取自訂映像。 |
> | 動作 | Microsoft.DevTestLab/labs/customImages/write | 新增或修改自訂映像。 |
> | 動作 | Microsoft.DevTestLab/labs/delete | 刪除實驗室。 |
> | 動作 | Microsoft.DevTestLab/labs/ExportResourceUsage/action | 將實驗室資源使用狀況匯出到儲存體帳戶 |
> | 動作 | Microsoft.DevTestLab/labs/formulas/delete | 刪除公式。 |
> | 動作 | Microsoft.DevTestLab/labs/formulas/read | 讀取公式。 |
> | 動作 | Microsoft.DevTestLab/labs/formulas/write | 新增或修改公式。 |
> | 動作 | Microsoft.DevTestLab/labs/galleryImages/read | 讀取資源庫映像。 |
> | 動作 | Microsoft.DevTestLab/labs/GenerateUploadUri/action | 產生 URI 以用來將自訂磁碟映像上傳到實驗室。 |
> | 動作 | Microsoft.DevTestLab/labs/ImportVirtualMachine/action | 將虛擬機器匯入不同的實驗室。 |
> | 動作 | Microsoft.DevTestLab/labs/ListVhds/action | 列出可供建立自訂映像的磁碟映像。 |
> | 動作 | Microsoft.DevTestLab/labs/notificationChannels/delete | 刪除 notificationchannels。 |
> | 動作 | Microsoft.DevTestLab/labs/notificationChannels/Notify/action | 對所提供的通道傳送通知。 |
> | 動作 | Microsoft.DevTestLab/labs/notificationChannels/read | 讀取 notificationchannels。 |
> | 動作 | Microsoft.DevTestLab/labs/notificationChannels/write | 新增或修改 notificationchannels。 |
> | 動作 | Microsoft.DevTestLab/labs/policySets/EvaluatePolicies/action | 評估實驗室原則。 |
> | 動作 | Microsoft.DevTestLab/labs/policySets/policies/delete | 刪除原則。 |
> | 動作 | Microsoft.DevTestLab/labs/policySets/policies/read | 讀取原則。 |
> | 動作 | Microsoft.DevTestLab/labs/policySets/policies/write | 新增或修改原則。 |
> | 動作 | Microsoft.DevTestLab/labs/read | 讀取實驗室。 |
> | 動作 | Microsoft.DevTestLab/labs/schedules/delete | 刪除排程。 |
> | 動作 | Microsoft.DevTestLab/labs/schedules/Execute/action | 執行排程。 |
> | 動作 | Microsoft.DevTestLab/labs/schedules/ListApplicable/action | 列出所有適用排程 |
> | 動作 | Microsoft.DevTestLab/labs/schedules/read | 讀取排程。 |
> | 動作 | Microsoft.DevTestLab/labs/schedules/write | 新增或修改排程。 |
> | 動作 | Microsoft.DevTestLab/labs/serviceRunners/delete | 刪除服務執行器。 |
> | 動作 | Microsoft.DevTestLab/labs/serviceRunners/read | 讀取服務執行器。 |
> | 動作 | Microsoft.DevTestLab/labs/serviceRunners/write | 新增或修改服務執行器。 |
> | 動作 | Microsoft.DevTestLab/labs/users/delete | 刪除使用者設定檔。 |
> | 動作 | Microsoft.DevTestLab/labs/users/disks/Attach/action | 將磁碟連結至虛擬機器並建立磁碟租用。 |
> | 動作 | Microsoft.DevTestLab/labs/users/disks/delete | 刪除磁碟。 |
> | 動作 | Microsoft.DevTestLab/labs/users/disks/Detach/action | 將連結至虛擬機器的磁碟中斷連結並中斷磁碟租用。 |
> | 動作 | Microsoft.DevTestLab/labs/users/disks/read | 讀取磁碟。 |
> | 動作 | Microsoft.DevTestLab/labs/users/disks/write | 新增或修改磁碟。 |
> | 動作 | Microsoft.DevTestLab/labs/users/environments/delete | 刪除環境。 |
> | 動作 | Microsoft.DevTestLab/labs/users/environments/read | 讀取環境。 |
> | 動作 | Microsoft.DevTestLab/labs/users/environments/write | 新增或修改環境。 |
> | 動作 | Microsoft.DevTestLab/labs/users/read | 讀取使用者設定檔。 |
> | 動作 | Microsoft.DevTestLab/labs/users/secrets/delete | 刪除祕密。 |
> | 動作 | Microsoft.DevTestLab/labs/users/secrets/read | 讀取祕密。 |
> | 動作 | Microsoft.DevTestLab/labs/users/secrets/write | 新增或修改祕密。 |
> | 動作 | Microsoft.DevTestLab/labs/users/serviceFabrics/delete | 刪除 Service Fabric。 |
> | 動作 | Microsoft.DevTestLab/labs/users/serviceFabrics/ListApplicableSchedules/action | 列出所有適用排程 |
> | 動作 | Microsoft.DevTestLab/labs/users/serviceFabrics/read | 讀取 Service Fabric。 |
> | 動作 | Microsoft.DevTestLab/labs/users/serviceFabrics/schedules/delete | 刪除排程。 |
> | 動作 | Microsoft.DevTestLab/labs/users/serviceFabrics/schedules/Execute/action | 執行排程。 |
> | 動作 | Microsoft.DevTestLab/labs/users/serviceFabrics/schedules/read | 讀取排程。 |
> | 動作 | Microsoft.DevTestLab/labs/users/serviceFabrics/schedules/write | 新增或修改排程。 |
> | 動作 | Microsoft.DevTestLab/labs/users/serviceFabrics/Start/action | 啟動 Service Fabric。 |
> | 動作 | Microsoft.DevTestLab/labs/users/serviceFabrics/Stop/action | 停止 Service Fabric |
> | 動作 | Microsoft.DevTestLab/labs/users/serviceFabrics/write | 新增或修改 Service Fabric。 |
> | 動作 | Microsoft.DevTestLab/labs/users/write | 新增或修改使用者設定檔。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/AddDataDisk/action | 將新的或現有的資料磁碟連結至虛擬機器。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/ApplyArtifacts/action | 將構件套用至虛擬機器。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/Claim/action | 取得現有虛擬機器的擁有權 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/delete | 刪除虛擬機器。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/DetachDataDisk/action | 中斷指定磁碟與虛擬機器的連結。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/GetRdpFileContents/action | 取得代表虛擬機器 RDP 檔案內容的字串 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/ListApplicableSchedules/action | 列出所有適用排程 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/read | 讀取虛擬機器。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/Redeploy/action | 重新部署虛擬機器 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/Resize/action | 調整虛擬機器的大小。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/Restart/action | 重新啟動虛擬機器。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/schedules/delete | 刪除排程。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/schedules/Execute/action | 執行排程。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/schedules/read | 讀取排程。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/schedules/write | 新增或修改排程。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/Start/action | 啟動虛擬機器。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/Stop/action | 停止虛擬機器 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/TransferDisks/action | 將虛擬機器資料磁碟的擁有權轉移給自己 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/UnClaim/action | 釋放現有虛擬機器的擁有權 |
> | 動作 | Microsoft.DevTestLab/labs/virtualMachines/write | 新增或修改虛擬機器。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualNetworks/delete | 刪除虛擬網路。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualNetworks/read | 讀取虛擬網路。 |
> | 動作 | Microsoft.DevTestLab/labs/virtualNetworks/write | 新增或修改虛擬網路。 |
> | 動作 | Microsoft.DevTestLab/labs/vmPools/delete | 刪除虛擬機器集區。 |
> | 動作 | Microsoft.DevTestLab/labs/vmPools/read | 讀取虛擬機器集區。 |
> | 動作 | Microsoft.DevTestLab/labs/vmPools/write | 新增或修改虛擬機器集區。 |
> | 動作 | Microsoft.DevTestLab/labs/write | 新增或修改實驗室。 |
> | 動作 | Microsoft.DevTestLab/locations/operations/read | 讀取作業。 |
> | 動作 | Microsoft.DevTestLab/register/action | 註冊訂用帳戶 |
> | 動作 | Microsoft.DevTestLab/schedules/delete | 刪除排程。 |
> | 動作 | Microsoft.DevTestLab/schedules/Execute/action | 執行排程。 |
> | 動作 | Microsoft.DevTestLab/schedules/read | 讀取排程。 |
> | 動作 | Microsoft.DevTestLab/schedules/Retarget/action | 更新排程的目標資源識別碼。 |
> | 動作 | Microsoft.DevTestLab/schedules/write | 新增或修改排程。 |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DocumentDB/databaseAccountNames/read | 檢查名稱可用性。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/changeResourceGroup/action | 變更資料庫帳戶的資源群組 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/databases/collections/metricDefinitions/read | 讀取集合計量定義。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/databases/collections/metrics/read | 讀取集合計量。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/databases/collections/partitionKeyRangeId/metrics/read | 讀取資料庫帳戶分割區索引鍵層級計量 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/databases/collections/partitions/metrics/read | 讀取資料庫帳戶分割區層級計量 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/databases/collections/partitions/read | 讀取集合中的資料庫帳戶分割區 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/databases/collections/partitions/usages/read | 讀取資料庫帳戶分割區層級使用量 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/databases/collections/usages/read | 讀取集合使用量。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/databases/metricDefinitions/read | 讀取資料庫計量定義 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/databases/metrics/read | 讀取資料庫計量。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/databases/usages/read | 讀取資料庫使用量。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/delete | 刪除資料庫帳戶。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/failoverPriorityChange/action | 變更資料庫帳戶區域的容錯移轉優先順序。 這會用來執行手動容錯移轉作業 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/listConnectionStrings/action | 取得資料庫帳戶的連接字串 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/listKeys/action | 列出資料庫帳戶的金鑰 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/metricDefinitions/read | 讀取資料庫帳戶的計量定義。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/metrics/read | 讀取資料庫帳戶的計量。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/offlineRegion/action | 讓資料庫帳戶的某個區域離線。  |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/onlineRegion/action | 讓資料庫帳戶的某個區域上線。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/operationResults/read | 讀取非同步作業的狀態 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/percentile/metrics/read | 讀取延遲計量 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/percentile/read | 讀取複寫延遲的百分位數 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/percentile/sourceRegion/targetRegion/metrics/read | 讀取指定來源和目標區域的延遲計量 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/percentile/targetRegion/metrics/read | 讀取指定目標區域的延遲計量 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/providers/Microsoft.Insights/logDefinitions/read | 取得資料庫帳戶的可用記錄類別 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/providers/Microsoft.Insights/metricDefinitions/read | 取得資料庫帳戶的可用計量 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/read | 讀取資料庫帳戶。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/readonlykeys/action | 讀取資料庫帳戶的唯讀金鑰。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/readonlykeys/read | 讀取資料庫帳戶的唯讀金鑰。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/regenerateKey/action | 輪替資料庫帳戶的金鑰 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/region/databases/collections/metrics/read | 讀取區域集合計量。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/region/databases/collections/partitionKeyRangeId/metrics/read | 讀取區域資料庫帳戶分割區索引鍵層級計量 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/region/databases/collections/partitions/metrics/read | 讀取區域資料庫帳戶分割區層級計量 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/region/databases/collections/partitions/read | 讀取集合中的區域資料庫帳戶分割區 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/region/metrics/read | 讀取區域和資料庫帳戶計量。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/usages/read | 讀取資料庫帳戶的使用量。 |
> | 動作 | Microsoft.DocumentDB/databaseAccounts/write | 更新資料庫帳戶。 |
> | 動作 | Microsoft.DocumentDB/locations/deleteVirtualNetworkOrSubnets/action | 通知 Microsoft.DocumentDB 正在刪除虛擬網路或子網路 |
> | 動作 | Microsoft.DocumentDB/locations/deleteVirtualNetworkOrSubnets/operationResults/read | 讀取 deleteVirtualNetworkOrSubnets 非同步作業的狀態 |
> | 動作 | Microsoft.DocumentDB/operationResults/read | 讀取非同步作業的狀態 |
> | 動作 | Microsoft.DocumentDB/operations/read | 讀取可供 Microsoft DocumentDB 使用的作業  |
> | 動作 | Microsoft.DocumentDB/register/action |  為訂用帳戶註冊 Microsoft DocumentDB 資源提供者 |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DomainRegistration/checkDomainAvailability/Action | 檢查網域是否可供購買 |
> | 動作 | Microsoft.DomainRegistration/domains/Delete | 刪除現有網域。 |
> | 動作 | Microsoft.DomainRegistration/domains/domainownershipidentifiers/Delete | 刪除擁有權識別碼 |
> | 動作 | Microsoft.DomainRegistration/domains/domainownershipidentifiers/Read | 列出擁有權識別碼 |
> | 動作 | Microsoft.DomainRegistration/domains/domainownershipidentifiers/Read | 取得擁有權識別碼 |
> | 動作 | Microsoft.DomainRegistration/domains/domainownershipidentifiers/Write | 建立或更新識別碼 |
> | 動作 | Microsoft.DomainRegistration/domains/operationresults/Read | 取得網域作業 |
> | 動作 | Microsoft.DomainRegistration/domains/Read | 取得網域清單 |
> | 動作 | Microsoft.DomainRegistration/domains/Read | 取得網域 |
> | 動作 | Microsoft.DomainRegistration/domains/renew/Action | 更新現有網域。 |
> | 動作 | Microsoft.DomainRegistration/domains/Write | 新增網域或更新現有網域 |
> | 動作 | Microsoft.DomainRegistration/generateSsoRequest/Action | 產生要求以便登入網域控制中心。 |
> | 動作 | Microsoft.DomainRegistration/listDomainRecommendations/Action | 根據關鍵字來擷取清單網域建議 |
> | 動作 | Microsoft.DomainRegistration/operations/Read | 列出 App Service 網域註冊的所有作業 |
> | 動作 | Microsoft.DomainRegistration/register/action | 針對訂用帳戶註冊 Microsoft 網域資源提供者 |
> | 動作 | Microsoft.DomainRegistration/topLevelDomains/listAgreements/Action | 列出合約動作 |
> | 動作 | Microsoft.DomainRegistration/topLevelDomains/Read | 取得最上層網域 |
> | 動作 | Microsoft.DomainRegistration/topLevelDomains/Read | 取得最上層網域 |
> | 動作 | Microsoft.DomainRegistration/validateDomainRegistrationInformation/Action | 驗證網域購買物件，但不將其提交出去 |

## <a name="microsoftdynamicslcs"></a>Microsoft.DynamicsLcs

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.DynamicsLcs/lcsprojects/clouddeployments/read | 顯示屬於某位使用者之 Microsoft Dynamics 週期服務專案內的 Microsoft Dynamics AX 2012 R3 評估部署 |
> | 動作 | Microsoft.DynamicsLcs/lcsprojects/clouddeployments/write | 在屬於某位使用者的 Microsoft Dynamics 週期服務專案內，建立 Microsoft Dynamics AX 2012 R3 評估部署。 您可以從 Azure 管理入口網站欄管理部署 |
> | 動作 | Microsoft.DynamicsLcs/lcsprojects/connectors/read | 讀取屬於 Microsoft Dynamics 週期服務專案的連接器 |
> | 動作 | Microsoft.DynamicsLcs/lcsprojects/connectors/write | 建立和更新屬於 Microsoft Dynamics 週期服務專案的連接器 |
> | 動作 | Microsoft.DynamicsLcs/lcsprojects/delete | 刪除屬於某位使用者的 Microsoft Dynamics 週期服務專案 |
> | 動作 | Microsoft.DynamicsLcs/lcsprojects/read | 顯示屬於某位使用者的 Microsoft Dynamics 週期服務專案 |
> | 動作 | Microsoft.DynamicsLcs/lcsprojects/write | 建立並更新屬於某位使用者的 Microsoft Dynamics 週期服務專案。 您只能更新名稱和描述屬性。 專案建立後，您就無法更新與其相關聯的訂用帳戶和位置 |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.EventGrid/eventSubscriptions/delete | 刪除 eventSubscription |
> | 動作 | Microsoft.EventGrid/eventSubscriptions/getFullUrl/action | 取得事件訂閱的完整 URL |
> | 動作 | Microsoft.EventGrid/eventSubscriptions/providers/Microsoft.Insights/diagnosticSettings/read | 取得事件訂閱的診斷設定 |
> | 動作 | Microsoft.EventGrid/eventSubscriptions/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新事件訂閱的診斷設定 |
> | 動作 | Microsoft.EventGrid/eventSubscriptions/providers/Microsoft.Insights/metricDefinitions/read | 取得 eventSubscriptions 的可用計量 |
> | 動作 | Microsoft.EventGrid/eventSubscriptions/read | 讀取 eventSubscription |
> | 動作 | Microsoft.EventGrid/eventSubscriptions/write | 建立或更新 eventSubscription |
> | 動作 | Microsoft.EventGrid/extensionTopics/providers/Microsoft.Insights/diagnosticSettings/read | 取得主題的診斷設定 |
> | 動作 | Microsoft.EventGrid/extensionTopics/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新主題的診斷設定 |
> | 動作 | Microsoft.EventGrid/extensionTopics/providers/Microsoft.Insights/metricDefinitions/read | 取得主題的可用計量 |
> | 動作 | Microsoft.EventGrid/register/action | 為 EventGrid 資源提供者註冊 eventSubscription，並讓您能夠建立事件格線訂用帳戶。 |
> | 動作 | Microsoft.EventGrid/topics/delete | 刪除主題 |
> | 動作 | Microsoft.EventGrid/topics/listKeys/action | 列出主題的金鑰 |
> | 動作 | Microsoft.EventGrid/topics/providers/Microsoft.Insights/diagnosticSettings/read | 取得主題的診斷設定 |
> | 動作 | Microsoft.EventGrid/topics/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新主題的診斷設定 |
> | 動作 | Microsoft.EventGrid/topics/providers/Microsoft.Insights/metricDefinitions/read | 取得主題的可用計量 |
> | 動作 | Microsoft.EventGrid/topics/read | 讀取主題 |
> | 動作 | Microsoft.EventGrid/topics/regenerateKey/action | 重新產生主題的金鑰 |
> | 動作 | Microsoft.EventGrid/topics/write | 建立或更新主題 |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.EventHub/checkNameAvailability/action | 檢查給定訂用帳戶下的命名空間是否可用。 |
> | 動作 | Microsoft.EventHub/checkNamespaceAvailability/action | 檢查給定訂用帳戶下的命名空間是否可用。 此 API 即將淘汰。請改用 CheckNameAvailabiltiy。 |
> | 動作 | Microsoft.EventHub/clusters/providers/Microsoft.Insights/metricDefinitions/read | 取得叢集計量資源描述的清單 |
> | 動作 | Microsoft.EventHub/clusters/read | 取得叢集資源描述 |
> | 動作 | Microsoft.EventHub/clusters/write | 取得叢集資源描述 |
> | 動作 | Microsoft.EventHub/namespaces/authorizationRules/action | 更新命名空間授權規則。 此 API 即將淘汰。 請改用 PUT 呼叫來更新命名空間授權規則。 API 版本 2017-04-01 不支援此作業。 |
> | 動作 | Microsoft.EventHub/namespaces/authorizationRules/delete | 刪除命名空間授權規則。 您無法刪除預設的命名空間授權規則。  |
> | 動作 | Microsoft.EventHub/namespaces/authorizationRules/listkeys/action | 取得命名空間的連接字串 |
> | 動作 | Microsoft.EventHub/namespaces/authorizationRules/read | 取得命名空間授權規則描述的清單。 |
> | 動作 | Microsoft.EventHub/namespaces/authorizationRules/regenerateKeys/action | 對資源重新產生主要或次要金鑰 |
> | 動作 | Microsoft.EventHub/namespaces/authorizationRules/write | 建立命名空間層級的授權規則，並更新其屬性。 您可以更新授權規則的存取權限、主要金鑰和次要金鑰。 |
> | 動作 | Microsoft.EventHub/namespaces/Delete | 刪除命名空間資源 |
> | 動作 | Microsoft.EventHub/namespaces/disasterRecoveryConfigs/authorizationRules/listkeys/action | 取得災害復原主要命名空間的授權規則金鑰 |
> | 動作 | Microsoft.EventHub/namespaces/disasterRecoveryConfigs/authorizationRules/read | 取得災害復原主要命名空間的授權規則 |
> | 動作 | Microsoft.EventHub/namespaces/disasterRecoveryConfigs/breakPairing/action | 停用災害復原，並停止將變更從主要命名空間複寫至次要命名空間。 |
> | 動作 | Microsoft.EventHub/namespaces/disasterrecoveryconfigs/checkNameAvailability/action | 檢查指定訂用帳戶下命名空間別名的可用性。 |
> | 動作 | Microsoft.EventHub/namespaces/disasterRecoveryConfigs/delete | 刪除與命名空間相關聯的災害復原設定。 此作業只能透過主要命名空間叫用。 |
> | 動作 | Microsoft.EventHub/namespaces/disasterRecoveryConfigs/failover/action | 叫用異地災害復原容錯移轉，並將命名空間別名重新設定為指向次要命名空間。 |
> | 動作 | Microsoft.EventHub/namespaces/disasterRecoveryConfigs/read | 取得與命名空間相關聯的災害復原設定。 |
> | 動作 | Microsoft.EventHub/namespaces/disasterRecoveryConfigs/write | 建立或更新與命名空間相關聯的災害復原設定。 |
> | 動作 | Microsoft.EventHub/namespaces/eventhubs/authorizationRules/action | 更新 EventHub 的作業。 API 版本 2017-04-01 不支援此作業。 授權規則。 請使用 PUT 呼叫來更新授權規則。 |
> | 動作 | Microsoft.EventHub/namespaces/eventhubs/authorizationRules/delete | 用來刪除 EventHub 授權規則的作業 |
> | 動作 | Microsoft.EventHub/namespaces/eventhubs/authorizationRules/listkeys/action | 取得 EventHub 的連接字串 |
> | 動作 | Microsoft.EventHub/namespaces/eventhubs/authorizationRules/read |  取得 EventHub 授權規則的清單 |
> | 動作 | Microsoft.EventHub/namespaces/eventhubs/authorizationRules/regenerateKeys/action | 對資源重新產生主要或次要金鑰 |
> | 動作 | Microsoft.EventHub/namespaces/eventhubs/authorizationRules/write | 建立 EventHub 授權規則，並更新其屬性。 您可以更新授權規則存取權限。 |
> | 動作 | Microsoft.EventHub/namespaces/eventHubs/consumergroups/Delete | 用來刪除 ConsumerGroup 資源的作業 |
> | 動作 | Microsoft.EventHub/namespaces/eventHubs/consumergroups/read | 取得 ConsumerGroup 資源描述的清單 |
> | 動作 | Microsoft.EventHub/namespaces/eventHubs/consumergroups/write | 建立或更新 ConsumerGroup 屬性。 |
> | 動作 | Microsoft.EventHub/namespaces/eventhubs/Delete | 用來刪除 EventHub 資源的作業 |
> | 動作 | Microsoft.EventHub/namespaces/eventhubs/read | 取得 EventHub 資源描述的清單 |
> | 動作 | Microsoft.EventHub/namespaces/eventhubs/write | 建立或更新 EventHub 屬性。 |
> | 動作 | Microsoft.EventHub/namespaces/messagingPlan/read | 取得命名空間的傳訊方案。<br>此 API 即將淘汰。<br>透過 MessagingPlan 資源公開的屬性，已移至新版 API 的 (父系) 命名空間資源。<br>API 版本 2017-04-01 不支援此作業。 |
> | 動作 | Microsoft.EventHub/namespaces/messagingPlan/write | 更新命名空間的傳訊方案。<br>此 API 即將淘汰。<br>透過 MessagingPlan 資源公開的屬性，已移至新版 API 的 (父系) 命名空間資源。<br>API 版本 2017-04-01 不支援此作業。 |
> | 動作 | Microsoft.EventHub/namespaces/operationresults/read | 取得命名空間作業的狀態 |
> | 動作 | Microsoft.EventHub/namespaces/providers/Microsoft.Insights/diagnosticSettings/read | 取得命名空間診斷設定資源描述的清單 |
> | 動作 | Microsoft.EventHub/namespaces/providers/Microsoft.Insights/diagnosticSettings/write | 取得命名空間診斷設定資源描述的清單 |
> | 動作 | Microsoft.EventHub/namespaces/providers/Microsoft.Insights/logDefinitions/read | 取得命名空間記錄資源描述的清單 |
> | 動作 | Microsoft.EventHub/namespaces/providers/Microsoft.Insights/metricDefinitions/read | 取得命名空間計量資源描述的清單 |
> | 動作 | Microsoft.EventHub/namespaces/read | 取得命名空間資源描述的清單 |
> | 動作 | Microsoft.EventHub/namespaces/removeAcsNamepsace/action | 移除 ACS 命名空間 |
> | 動作 | Microsoft.EventHub/namespaces/write | 建立命名空間資源，並更新其屬性。 命名空間的標記和容量是可以更新的屬性。 |
> | 動作 | Microsoft.EventHub/operations/read | 取得作業 |
> | 動作 | Microsoft.EventHub/register/action | 針對 EventHub 資源提供者註冊訂用帳戶，並讓您能夠建立 EventHub 資源 |
> | 動作 | Microsoft.EventHub/sku/read | 取得 SKU 資源描述的清單 |
> | 動作 | Microsoft.EventHub/sku/regions/read | 取得 SkuRegions 資源描述的清單 |
> | 動作 | Microsoft.EventHub/unregister/action | 註冊 EventHub 資源提供者 |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Features/features/read | 取得訂用帳戶的功能。 |
> | 動作 | Microsoft.Features/operations/read | 取得作業的清單。 |
> | 動作 | Microsoft.Features/providers/features/read | 取得給定資源提供者中某個訂用帳戶的功能。 |
> | 動作 | Microsoft.Features/providers/features/register/action | 註冊給定資源提供者中某個訂用帳戶的功能。 |
> | 動作 | Microsoft.Features/providers/features/unregister/action | 在指定的資源提供者內取消註冊訂用帳戶的功能。 |
> | 動作 | Microsoft.Features/register/action | 註冊訂用帳戶的功能。 |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.GuestConfiguration/guestConfigurationAssignments/read | 取得來賓組態指派。 |
> | 動作 | Microsoft.GuestConfiguration/guestConfigurationAssignments/write | 取得新的來賓組態指派。 |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.HDInsight/clusters/applications/delete | 刪除 HDInsight 叢集的應用程式 |
> | 動作 | Microsoft.HDInsight/clusters/applications/read | 取得 HDInsight 叢集的應用程式 |
> | 動作 | Microsoft.HDInsight/clusters/applications/write | 建立或更新 HDInsight 叢集的應用程式 |
> | 動作 | Microsoft.HDInsight/clusters/changerdpsetting/action | 變更 HDInsight 叢集的 RDP 設定 |
> | 動作 | Microsoft.HDInsight/clusters/configurations/action | 更新 HDInsight 叢集組態 |
> | 動作 | Microsoft.HDInsight/clusters/configurations/read | 取得 HDInsight 叢集組態 |
> | 動作 | Microsoft.HDInsight/clusters/delete | 刪除 HDInsight 叢集 |
> | 動作 | Microsoft.HDInsight/clusters/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源 HDInsight 叢集的診斷設定 |
> | 動作 | Microsoft.HDInsight/clusters/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源 HDInsight 叢集的診斷設定 |
> | 動作 | Microsoft.HDInsight/clusters/providers/Microsoft.Insights/metricDefinitions/read | 取得 HDInsight 叢集的可用計量 |
> | 動作 | Microsoft.HDInsight/clusters/read | 取得 HDInsight 叢集的相關詳細資料 |
> | 動作 | Microsoft.HDInsight/clusters/roles/resize/action | 調整 HDInsight 叢集的大小 |
> | 動作 | Microsoft.HDInsight/clusters/write | 建立或更新 HDInsight 叢集 |
> | 動作 | Microsoft.HDInsight/locations/capabilities/read | 取得訂用帳戶功能 |
> | 動作 | Microsoft.HDInsight/locations/checkNameAvailability/read | 檢查名稱可用性 |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ImportExport/jobs/delete | 刪除現有作業。 |
> | 動作 | Microsoft.ImportExport/jobs/listBitLockerKeys/action | 取得指定作業的 BitLocker 金鑰。 |
> | 動作 | Microsoft.ImportExport/jobs/read | 取得指定作業的屬性，或傳回作業清單。 |
> | 動作 | Microsoft.ImportExport/jobs/write | 使用指定參數建立作業，或更新指定作業的屬性或標記。 |
> | 動作 | Microsoft.ImportExport/locations/read | 取得指定位置的屬性，或傳回位置清單。 |
> | 動作 | Microsoft.ImportExport/register/action | 針對匯入/匯出資源提供者註冊訂用帳戶，並讓您能夠建立匯入/匯出作業。 |

## <a name="microsoftinsights"></a>Microsoft.Insights

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Insights/ActionGroups/Delete | 刪除動作群組 |
> | 動作 | Microsoft.Insights/ActionGroups/Read | 讀取動作群組 |
> | 動作 | Microsoft.Insights/ActionGroups/Write | 建立或更新動作群組 |
> | 動作 | Microsoft.Insights/ActivityLogAlerts/Activated/Action | 活動記錄警示已啟用 |
> | 動作 | Microsoft.Insights/ActivityLogAlerts/Delete | 刪除活動記錄警示 |
> | 動作 | Microsoft.Insights/ActivityLogAlerts/Read | 讀取活動記錄警示 |
> | 動作 | Microsoft.Insights/ActivityLogAlerts/Write | 建立或更新活動記錄警示 |
> | 動作 | Microsoft.Insights/AlertRules/Activated/Action | 傳統計量警示已啟用 |
> | 動作 | Microsoft.Insights/AlertRules/Delete | 刪除傳統計量警示 |
> | 動作 | Microsoft.Insights/AlertRules/Incidents/Read | 讀取傳統計量警示事件 |
> | 動作 | Microsoft.Insights/AlertRules/Read | 讀取傳統計量警示 |
> | 動作 | Microsoft.Insights/AlertRules/Resolved/Action | 傳統計量警示已解決 |
> | 動作 | Microsoft.Insights/AlertRules/Throttled/Action | 傳統計量警示規則已節流 |
> | 動作 | Microsoft.Insights/AlertRules/Write | 建立或更新傳統計量警示 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/Delete | 刪除自動調整設定 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/providers/Microsoft.Insights/diagnosticSettings/Read | 讀取資源診斷設定 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/providers/Microsoft.Insights/diagnosticSettings/Write | 建立或更新資源診斷設定 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/providers/Microsoft.Insights/logDefinitions/Read | 讀取記錄定義 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/providers/Microsoft.Insights/MetricDefinitions/Read | 讀取計量定義 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/Read | 讀取自動調整設定 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/Scaledown/Action | 自動調整相應減少已起始 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/ScaledownResult/Action | 自動調整相應減少已完成 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/Scaleup/Action | 自動調整相應增加已起始 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/ScaleupResult/Action | 自動調整相應增加已完成 |
> | 動作 | Microsoft.Insights/AutoscaleSettings/Write | 建立或更新自動調整設定 |
> | 動作 | Microsoft.Insights/Components/AnalyticsItems/Delete | 刪除 Application Insights 分析項目 |
> | 動作 | Microsoft.Insights/Components/AnalyticsItems/Read | 讀取 Application Insights 分析項目 |
> | 動作 | Microsoft.Insights/Components/AnalyticsItems/Write | 寫入 Application Insights 分析項目 |
> | 動作 | Microsoft.Insights/Components/AnalyticsTables/Action | Application Insights 分析資料表動作 |
> | 動作 | Microsoft.Insights/Components/AnalyticsTables/Delete | 刪除 Application Insights 分析資料表結構描述 |
> | 動作 | Microsoft.Insights/Components/AnalyticsTables/Read | 讀取 Application Insights 分析資料表結構描述 |
> | 動作 | Microsoft.Insights/Components/AnalyticsTables/Write | 寫入 Application Insights 分析資料表結構描述 |
> | 動作 | Microsoft.Insights/Components/Annotations/Delete | 刪除 Application Insights 註釋 |
> | 動作 | Microsoft.Insights/Components/Annotations/Read | 讀取 Application Insights 註釋 |
> | 動作 | Microsoft.Insights/Components/Annotations/Write | 寫入 Application Insights 註釋 |
> | 動作 | Microsoft.Insights/Components/Api/Read | 讀取 Application Insights 元件資料 API |
> | 動作 | Microsoft.Insights/Components/ApiKeys/Action | 產生 Application Insights API 金鑰 |
> | 動作 | Microsoft.Insights/Components/ApiKeys/Delete | 刪除 Application Insights API 金鑰 |
> | 動作 | Microsoft.Insights/Components/ApiKeys/Read | 讀取 Application Insights API 金鑰 |
> | 動作 | Microsoft.Insights/Components/BillingPlanForComponent/Read | 讀取 Application Insights 元件的計費方案 |
> | 動作 | Microsoft.Insights/Components/CurrentBillingFeatures/Read | 在讀取 Application Insights 元件目前的計費功能 |
> | 動作 | Microsoft.Insights/Components/CurrentBillingFeatures/Write | 寫入 Application Insights 元件目前的計費功能 |
> | 動作 | Microsoft.Insights/Components/DefaultWorkItemConfig/Read | 讀取 Application Insights 預設的 ALM 整合設定 |
> | 動作 | Microsoft.Insights/Components/Delete | 刪除 Application Insights 元件設定 |
> | 動作 | Microsoft.Insights/Components/Events/Read | 使用 OData 查詢格式從 Application Insights 取得記錄 |
> | 動作 | Microsoft.Insights/Components/ExportConfiguration/Action | Application Insights 匯出設定動作 |
> | 動作 | Microsoft.Insights/Components/ExportConfiguration/Delete | 刪除 Application Insights 匯出設定 |
> | 動作 | Microsoft.Insights/Components/ExportConfiguration/Read | 讀取 Application Insights 匯出設定 |
> | 動作 | Microsoft.Insights/Components/ExportConfiguration/Write | 寫入 Application Insights 匯出設定 |
> | 動作 | Microsoft.Insights/Components/ExtendQueries/Read | 讀取 Application Insights 元件擴充的查詢結果 |
> | 動作 | Microsoft.Insights/Components/Favorites/Delete | 刪除 Application Insights 我的最愛 |
> | 動作 | Microsoft.Insights/Components/Favorites/Read | 讀取 Application Insights 我的最愛 |
> | 動作 | Microsoft.Insights/Components/Favorites/Write | 寫入 Application Insights 我的最愛 |
> | 動作 | Microsoft.Insights/Components/FeatureCapabilities/Read | 讀取 Application Insights 元件功能的能力 |
> | 動作 | Microsoft.Insights/Components/GetAvailableBillingFeatures/Read | 讀取 Application Insights 元件可用的計費功能 |
> | 動作 | Microsoft.Insights/Components/GetToken/Read | 讀取 Application Insights 元件權杖 |
> | 動作 | Microsoft.Insights/Components/MetricDefinitions/Read | 讀取 Application Insights 元件計量定義 |
> | 動作 | Microsoft.Insights/Components/Metrics/Read | 讀取 Application Insights 元件計量 |
> | 動作 | Microsoft.Insights/Components/Move/Action | 將 Application Insights 元件移到另一個資源群組或訂用帳戶 |
> | 動作 | Microsoft.Insights/Components/MyAnalyticsItems/Delete | 刪除 Application Insights 個人分析項目 |
> | 動作 | Microsoft.Insights/Components/MyAnalyticsItems/Read | 讀取 Application Insights 個人分析項目 |
> | 動作 | Microsoft.Insights/Components/MyAnalyticsItems/Write | 寫入 Application Insights 個人分析項目 |
> | 動作 | Microsoft.Insights/Components/MyFavorites/Read | 讀取 Application Insights 個人我的最愛 |
> | 動作 | Microsoft.Insights/Components/Operations/Read | 取得 Application Insights 中長時間執行之作業的狀態 |
> | 動作 | Microsoft.Insights/Components/PricingPlans/Read | 讀取 Application Insights 元件定價方案 |
> | 動作 | Microsoft.Insights/Components/PricingPlans/Write | 寫入 Application Insights 元件定價方案 |
> | 動作 | Microsoft.Insights/Components/ProactiveDetectionConfigs/Read | 讀取 Application Insights 主動式偵測設定 |
> | 動作 | Microsoft.Insights/Components/ProactiveDetectionConfigs/Write | 寫入 Application Insights 主動式偵測設定 |
> | 動作 | Microsoft.Insights/Components/providers/Microsoft.Insights/MetricDefinitions/Read | 讀取計量定義 |
> | 動作 | Microsoft.Insights/Components/Purge/Action | 從 Application Insights 清除資料 |
> | 動作 | Microsoft.Insights/Components/Query/Read | 針對 Application Insights 記錄執行查詢 |
> | 動作 | Microsoft.Insights/Components/QuotaStatus/Read | 讀取 Application Insights 元件配額狀態 |
> | 動作 | Microsoft.Insights/Components/Read | 讀取 Application Insights 元件設定 |
> | 動作 | Microsoft.Insights/Components/SyntheticMonitorLocations/Read | 讀取 Application Insights webtest 位置 |
> | 動作 | Microsoft.Insights/Components/Webtests/Read | 讀取 Webtest 設定 |
> | 動作 | Microsoft.Insights/Components/WorkItemConfigs/Delete | 刪除 Application Insights ALM 整合設定 |
> | 動作 | Microsoft.Insights/Components/WorkItemConfigs/Read | 讀取 Application Insights ALM 整合設定 |
> | 動作 | Microsoft.Insights/Components/WorkItemConfigs/Write | 寫入 Application Insights ALM 整合設定 |
> | 動作 | Microsoft.Insights/Components/Write | 寫入 Application Insights 元件設定 |
> | 動作 | Microsoft.Insights/DiagnosticSettings/Delete | 刪除資源診斷設定 |
> | 動作 | Microsoft.Insights/DiagnosticSettings/Read | 讀取資源診斷設定 |
> | 動作 | Microsoft.Insights/DiagnosticSettings/Write | 建立或更新資源診斷設定 |
> | 動作 | Microsoft.Insights/EventCategories/Read | 讀取可用的活動記錄事件類別 |
> | 動作 | Microsoft.Insights/eventtypes/digestevents/Read | 讀取管理事件類型摘要 |
> | 動作 | Microsoft.Insights/eventtypes/values/Read | 讀取活動記錄事件 |
> | 動作 | Microsoft.Insights/ExtendedDiagnosticSettings/Delete | 刪除網路流量記錄診斷設定 |
> | 動作 | Microsoft.Insights/ExtendedDiagnosticSettings/Read | 讀取網路流量記錄診斷設定 |
> | 動作 | Microsoft.Insights/ExtendedDiagnosticSettings/Write | 建立或更新網路流量記錄診斷設定 |
> | 動作 | Microsoft.Insights/ListMigrationDate/Action | 取回訂用帳戶移轉日期 |
> | 動作 | Microsoft.Insights/ListMigrationDate/Read | 取回訂用帳戶移轉日期 |
> | 動作 | Microsoft.Insights/LogDefinitions/Read | 讀取記錄定義 |
> | 動作 | Microsoft.Insights/LogProfiles/Delete | 刪除活動記錄的記錄設定檔 |
> | 動作 | Microsoft.Insights/LogProfiles/Read | 讀取活動記錄的記錄設定檔 |
> | 動作 | Microsoft.Insights/LogProfiles/Write | 建立或更新活動記錄的記錄設定檔 |
> | 動作 | Microsoft.Insights/Logs/ADAssessmentRecommendation/Read | 從 ADAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ADReplicationResult/Read | 從 ADReplicationResult 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ADSecurityAssessmentRecommendation/Read | 從 ADSecurityAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/Alert/Read | 從 Alert 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/AlertHistory/Read | 從 AlertHistory 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ApplicationInsights/Read | 從 ApplicationInsights 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/AzureActivity/Read | 從 AzureActivity 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/AzureMetrics/Read | 從 AzureMetrics 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/BoundPort/Read | 從 BoundPort 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/CommonSecurityLog/Read | 從 CommonSecurityLog 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ComputerGroup/Read | 從 ComputerGroup 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ConfigurationChange/Read | 從 ConfigurationChange 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ConfigurationData/Read | 從 ConfigurationData 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ContainerImageInventory/Read | 從 ContainerImageInventory 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ContainerInventory/Read | 從 ContainerInventory 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ContainerLog/Read | 從 ContainerLog 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ContainerServiceLog/Read | 從 ContainerServiceLog 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceAppCrash/Read | 從 DeviceAppCrash 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceAppLaunch/Read | 從 DeviceAppLaunch 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceCalendar/Read | 從 DeviceCalendar 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceCleanup/Read | 從 DeviceCleanup 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceConnectSession/Read | 從 DeviceConnectSession 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceEtw/Read | 從 DeviceEtw 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceHardwareHealth/Read | 從 DeviceHardwareHealth 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceHealth/Read | 從 DeviceHealth 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceHeartbeat/Read | 從 DeviceHeartbeat 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceSkypeHeartbeat/Read | 從 DeviceSkypeHeartbeat 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceSkypeSignIn/Read | 從 DeviceSkypeSignIn 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DeviceSleepState/Read | 從 DeviceSleepState 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DHAppFailure/Read | 從 DHAppFailure 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DHAppReliability/Read | 從 DHAppReliability 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DHDriverReliability/Read | 從 DHDriverReliability 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DHLogonFailures/Read | 從 DHLogonFailures 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DHLogonMetrics/Read | 從 DHLogonMetrics 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DHOSCrashData/Read | 從 DHOSCrashData 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DHOSReliability/Read | 從 DHOSReliability 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DHWipAppLearning/Read | 從 DHWipAppLearning 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DnsEvents/Read | 從 DnsEvents 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/DnsInventory/Read | 從 DnsInventory 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ETWEvent/Read | 從 ETWEvent 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/Event/Read | 從 Event 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ExchangeAssessmentRecommendation/Read | 從 ExchangeAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ExchangeOnlineAssessmentRecommendation/Read | 從 ExchangeOnlineAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/Heartbeat/Read | 從 Heartbeat 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/IISAssessmentRecommendation/Read | 從 IISAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/InboundConnection/Read | 從 InboundConnection 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/KubeNodeInventory/Read | 從 KubeNodeInventory 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/KubePodInventory/Read | 從 KubePodInventory 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/LinuxAuditLog/Read | 從 LinuxAuditLog 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAApplication/Read | 從 MAApplication 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAApplicationHealth/Read | 從 MAApplicationHealth 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAApplicationHealthAlternativeVersions/Read | 從 MAApplicationHealthAlternativeVersions 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAApplicationHealthIssues/Read | 從 MAApplicationHealthIssues 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAApplicationInstance/Read | 從 MAApplicationInstance 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAApplicationInstanceReadiness/Read | 從 MAApplicationInstanceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAApplicationReadiness/Read | 從 MAApplicationReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MADeploymentPlan/Read | 從 MADeploymentPlan 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MADevice/Read | 從 MADevice 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MADevicePnPHealth/Read | 從 MADevicePnPHealth 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MADevicePnPHealthAlternativeVersions/Read | 從 MADevicePnPHealthAlternativeVersions 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MADevicePnPHealthIssues/Read | 從 MADevicePnPHealthIssues 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MADeviceReadiness/Read | 從 MADeviceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MADriverInstanceReadiness/Read | 從 MADriverInstanceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MADriverReadiness/Read | 從 MADriverReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeAddin/Read | 從 MAOfficeAddin 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeAddinHealth/Read | 從 MAOfficeAddinHealth 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeAddinHealthIssues/Read | 從 MAOfficeAddinHealthIssues 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeAddinInstance/Read | 從 MAOfficeAddinInstance 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeAddinInstanceReadiness/Read | 從 MAOfficeAddinInstanceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeAddinReadiness/Read | 從 MAOfficeAddinReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeApp/Read | 從 MAOfficeApp 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeAppHealth/Read | 從 MAOfficeAppHealth 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeAppInstance/Read | 從 MAOfficeAppInstance 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeAppReadiness/Read | 從 MAOfficeAppReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeBuildInfo/Read | 從 MAOfficeBuildInfo 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeCurrencyAssessment/Read | 從 MAOfficeCurrencyAssessment 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeCurrencyAssessmentDailyCounts/Read | 從 MAOfficeCurrencyAssessmentDailyCounts 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeDeploymentStatus/Read | 從 MAOfficeDeploymentStatus 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeMacroHealth/Read | 從 MAOfficeMacroHealth 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeMacroHealthIssues/Read | 從 MAOfficeMacroHealthIssues 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeMacroIssueInstanceReadiness/Read | 從 MAOfficeMacroIssueInstanceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeMacroIssueReadiness/Read | 從 MAOfficeMacroIssueReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeMacroSummary/Read | 從 MAOfficeMacroSummary 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeSuite/Read | 從 MAOfficeSuite 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAOfficeSuiteInstance/Read | 從 MAOfficeSuiteInstance 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAProposedPilotDevices/Read | 從 MAProposedPilotDevices 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAWindowsBuildInfo/Read | 從 MAWindowsBuildInfo 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAWindowsCurrencyAssessment/Read | 從 MAWindowsCurrencyAssessment 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAWindowsCurrencyAssessmentDailyCounts/Read | 從 MAWindowsCurrencyAssessmentDailyCounts 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAWindowsDeploymentStatus/Read | 從 MAWindowsDeploymentStatus 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/MAWindowsSysReqInstanceReadiness/Read | 從 MAWindowsSysReqInstanceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/NetworkMonitoring/Read | 從 NetworkMonitoring 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/OfficeActivity/Read | 從 OfficeActivity 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/Operation/Read | 從 Operation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/OutboundConnection/Read | 從 OutboundConnection 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/Perf/Read | 從 Perf 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ProtectionStatus/Read | 從 ProtectionStatus 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/Read | 從您的所有記錄檔讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ReservedAzureCommonFields/Read | 從 ReservedAzureCommonFields 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ReservedCommonFields/Read | 從 ReservedCommonFields 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SCCMAssessmentRecommendation/Read | 從 SCCMAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SCOMAssessmentRecommendation/Read | 從 SCOMAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SecurityAlert/Read | 從 SecurityAlert 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SecurityBaseline/Read | 從 SecurityBaseline 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SecurityBaselineSummary/Read | 從 SecurityBaselineSummary 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SecurityDetection/Read | 從 SecurityDetection 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SecurityEvent/Read | 從 SecurityEvent 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ServiceFabricOperationalEvent/Read | 從 ServiceFabricOperationalEvent 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ServiceFabricReliableActorEvent/Read | 從 ServiceFabricReliableActorEvent 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/ServiceFabricReliableServiceEvent/Read | 從 ServiceFabricReliableServiceEvent 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SfBAssessmentRecommendation/Read | 從 SfBAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SfBOnlineAssessmentRecommendation/Read | 從 SfBOnlineAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SharePointOnlineAssessmentRecommendation/Read | 從 SharePointOnlineAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SPAssessmentRecommendation/Read | 從 SPAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SQLAssessmentRecommendation/Read | 從 SQLAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SQLQueryPerformance/Read | 從 SQLQueryPerformance 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/Syslog/Read | 從 Syslog 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/SysmonEvent/Read | 從 SysmonEvent 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UAApp/Read | 從 UAApp 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UAComputer/Read | 從 UAComputer 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UAComputerRank/Read | 從 UAComputerRank 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UADriver/Read | 從 UADriver 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UADriverProblemCodes/Read | 從 UADriverProblemCodes 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UAFeedback/Read | 從 UAFeedback 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UAHardwareSecurity/Read | 從 UAHardwareSecurity 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UAIESiteDiscovery/Read | 從 UAIESiteDiscovery 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UAOfficeAddIn/Read | 從 UAOfficeAddIn 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UAProposedActionPlan/Read | 從 UAProposedActionPlan 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UASysReqIssue/Read | 從 UASysReqIssue 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UAUpgradedComputer/Read | 從 UAUpgradedComputer 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/Update/Read | 從 Update 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UpdateRunProgress/Read | 從 UpdateRunProgress 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/UpdateSummary/Read | 從 UpdateSummary 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/Usage/Read | 從 Usage 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/W3CIISLog/Read | 從 W3CIISLog 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WaaSDeploymentStatus/Read | 從 WaaSDeploymentStatus 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WaaSInsiderStatus/Read | 從 WaaSInsiderStatus 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WaaSUpdateStatus/Read | 從 WaaSUpdateStatus 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WDAVStatus/Read | 從 WDAVStatus 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WDAVThreat/Read | 從 WDAVThreat 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WindowsClientAssessmentRecommendation/Read | 從 WindowsClientAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WindowsFirewall/Read | 從 WindowsFirewall 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WindowsServerAssessmentRecommendation/Read | 從 WindowsServerAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WireData/Read | 從 WireData 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WUDOAggregatedStatus/Read | 從 WUDOAggregatedStatus 資料表讀取資料 |
> | 動作 | Microsoft.Insights/Logs/WUDOStatus/Read | 從 WUDOStatus 資料表讀取資料 |
> | 動作 | Microsoft.Insights/MetricAlerts/Delete | 刪除計量警示 |
> | 動作 | Microsoft.Insights/MetricAlerts/Read | 讀取計量警示 |
> | 動作 | Microsoft.Insights/MetricAlerts/Status/Read | 讀取計量警示狀態 |
> | 動作 | Microsoft.Insights/MetricAlerts/Write | 建立或更新計量警示 |
> | 動作 | Microsoft.Insights/MetricDefinitions/Microsoft.Insights/Read | 讀取計量定義 |
> | 動作 | Microsoft.Insights/MetricDefinitions/providers/Microsoft.Insights/Read | 讀取計量定義 |
> | 動作 | Microsoft.Insights/MetricDefinitions/Read | 讀取計量定義 |
> | 動作 | Microsoft.Insights/Metrics/providers/Metrics/Read | 讀取計量 |
> | 動作 | Microsoft.Insights/Metrics/Read | 讀取計量 |
> | DataAction | Microsoft.Insights/Metrics/Write | 寫入計量 |
> | 動作 | Microsoft.Insights/MigrateToNewpricingModel/Action | 將訂用帳戶移轉至新的計價模式 |
> | 動作 | Microsoft.Insights/Operations/Read | 讀取作業 |
> | 動作 | Microsoft.Insights/Register/Action | 註冊 Microsoft Insights 提供者 |
> | 動作 | Microsoft.Insights/RollbackToLegacyPricingModel/Action | 將訂用帳戶復原至舊版計價模式 |
> | 動作 | Microsoft.Insights/ScheduledQueryRules/Delete | 刪除排程的查詢規則 |
> | 動作 | Microsoft.Insights/ScheduledQueryRules/Read | 讀取排程的查詢規則 |
> | 動作 | Microsoft.Insights/ScheduledQueryRules/Write | 寫入排程的查詢規則 |
> | 動作 | Microsoft.Insights/Tenants/Register/Action | 將 Microsoft Insights 提供者初始化 |
> | 動作 | Microsoft.Insights/Unregister/Action | 註冊 Microsoft Insights 提供者 |
> | 動作 | Microsoft.Insights/Webtests/Delete | 刪除 Webtest 設定 |
> | 動作 | Microsoft.Insights/Webtests/GetToken/Read | 讀取 Webtest 權杖 |
> | 動作 | Microsoft.Insights/Webtests/MetricDefinitions/Read | 讀取 Webtest 計量定義 |
> | 動作 | Microsoft.Insights/Webtests/Metrics/Read | 讀取 Webtest 計量 |
> | 動作 | Microsoft.Insights/Webtests/Read | 讀取 Webtest 設定 |
> | 動作 | Microsoft.Insights/Webtests/Write | 寫入至 Webtest 設定 |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.IoTSpaces/Graph/delete | 刪除 Microsoft.IoTSpaces Graph 資源 |
> | 動作 | Microsoft.IoTSpaces/Graph/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.IoTSpaces/Graph/providers/Microsoft.Insights/diagnosticSettings/write | 設定資源的診斷設定 |
> | 動作 | Microsoft.IoTSpaces/Graph/providers/Microsoft.Insights/logDefinitions/read | 取得 Microsoft.IoTSpaces 服務的可用記錄檔定義 |
> | 動作 | Microsoft.IoTSpaces/Graph/providers/Microsoft.Insights/metricDefinitions/read | 取得 Microsoft.IoTSpaces 服務的可用計量定義 |
> | 動作 | Microsoft.IoTSpaces/Graph/read | 取得 Microsoft.IoTSpaces Graph 資源 |
> | 動作 | Microsoft.IoTSpaces/Graph/write | 建立 Microsoft.IoTSpaces Graph 資源 |
> | 動作 | Microsoft.IoTSpaces/register/action | 為 Microsoft.IoTSpaces Graph 資源提供者註冊訂用帳戶以啟用資源的建立作業 |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.KeyVault/checkNameAvailability/read | 確認 Key Vault 名稱有效，且並非使用中 |
> | 動作 | Microsoft.KeyVault/deletedVaults/read | 檢視虛刪除之 Key Vault 的屬性 |
> | 動作 | Microsoft.KeyVault/hsmPools/delete | 刪除 HSM 集區 |
> | 動作 | Microsoft.KeyVault/hsmPools/joinVault/action | 將金鑰保存庫加入至 HSM 集區 |
> | 動作 | Microsoft.KeyVault/hsmPools/read | 檢視 HSM 集區的屬性 |
> | 動作 | Microsoft.KeyVault/hsmPools/write | 建立新的 HSM 集區，或更新現有 HSM 集區的屬性 |
> | 動作 | Microsoft.KeyVault/locations/deletedVaults/purge/action | 清除虛刪除的 Key Vault |
> | 動作 | Microsoft.KeyVault/locations/deletedVaults/read | 檢視虛刪除之 Key Vault 的屬性 |
> | 動作 | Microsoft.KeyVault/locations/deleteVirtualNetworkOrSubnets/action | 通知 Microsoft.KeyVault 正在刪除虛擬網路或子網路 |
> | 動作 | Microsoft.KeyVault/locations/operationResults/read | 檢查長時間執行之作業的結果 |
> | 動作 | Microsoft.KeyVault/operations/read | 列出可以對 Microsoft.KeyVault 資源提供者執行的作業 |
> | 動作 | Microsoft.KeyVault/register/action | 註冊訂用帳戶 |
> | 動作 | Microsoft.KeyVault/unregister/action | 取消註冊訂用帳戶 |
> | 動作 | Microsoft.KeyVault/vaults/accessPolicies/write | 藉由合併或取代來更新現有存取原則，或在保存庫中新增存取原則。 |
> | 動作 | Microsoft.KeyVault/vaults/delete | 刪除 Key Vault |
> | 動作 | Microsoft.KeyVault/vaults/deploy/action | 在部署 Azure 資源時啟用 Key Vault 中祕密的存取權 |
> | 動作 | Microsoft.KeyVault/vaults/providers/Microsoft.Insights/diagnosticSettings/Read | 取得資源的診斷設定 |
> | 動作 | Microsoft.KeyVault/vaults/providers/Microsoft.Insights/diagnosticSettings/Write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.KeyVault/vaults/providers/Microsoft.Insights/logDefinitions/read | 取得金鑰保存庫的可用記錄 |
> | 動作 | Microsoft.KeyVault/vaults/providers/Microsoft.Insights/metricDefinitions/read | 取得金鑰保存庫的可用計量 |
> | 動作 | Microsoft.KeyVault/vaults/read | 檢視 Key Vault 的屬性 |
> | 動作 | Microsoft.KeyVault/vaults/secrets/read | 檢視祕密的屬性而非其值 |
> | 動作 | Microsoft.KeyVault/vaults/secrets/write | 建立新的祕密，或更新現有祕密的值 |
> | 動作 | Microsoft.KeyVault/vaults/write | 建立新的 Key Vault，或更新現有 Key Vault 的屬性 |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Kusto/Clusters/Databases/delete | 刪除資料庫資源。 |
> | 動作 | Microsoft.Kusto/Clusters/Databases/EventHubConnections/delete | 刪除事件中樞連線資源。 |
> | 動作 | Microsoft.Kusto/Clusters/Databases/EventHubConnections/read | 讀取事件中樞連線資源。 |
> | 動作 | Microsoft.Kusto/Clusters/Databases/EventHubConnections/write | 寫入事件中樞連線資源。 |
> | 動作 | Microsoft.Kusto/Clusters/Databases/read | 讀取資料庫資源。 |
> | 動作 | Microsoft.Kusto/Clusters/Databases/write | 寫入資料庫資源。 |
> | 動作 | Microsoft.Kusto/Clusters/delete | 刪除叢集資源。 |
> | 動作 | Microsoft.Kusto/Clusters/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Kusto/Clusters/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Kusto/Clusters/providers/Microsoft.Insights/metricDefinitions/read | 取得資源的計量定義 |
> | 動作 | Microsoft.Kusto/Clusters/read | 讀取叢集資源。 |
> | 動作 | Microsoft.Kusto/Clusters/write | 寫入叢集資源。 |
> | 動作 | Microsoft.Kusto/Locations/CheckNameAvailability/write | 讀取檢查名稱可用性資源 |
> | 動作 | Microsoft.Kusto/locations/operationresults/read | 讀取作業資源 |
> | 動作 | Microsoft.Kusto/Operations/read | 讀取作業資源 |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.LabServices/labAccounts/CreateLab/action | 在實驗室帳戶中建立實驗室。 |
> | 動作 | Microsoft.LabServices/labAccounts/delete | 刪除實驗室帳戶。 |
> | 動作 | Microsoft.LabServices/labAccounts/galleryImages/delete | 刪除資源庫映像。 |
> | 動作 | Microsoft.LabServices/labAccounts/galleryImages/read | 讀取資源庫映像。 |
> | 動作 | Microsoft.LabServices/labAccounts/galleryImages/write | 新增或修改資源庫映像。 |
> | 動作 | Microsoft.LabServices/labAccounts/GetRegionalAvailability/action | 取得在實驗室帳戶下設定的每個大小類別的區域可用性資訊 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/AddUsers/action | 將使用者新增至實驗室 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/delete | 刪除實驗室。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/ClaimAny/action | 在環境設定中為使用者宣告隨機環境 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/delete | 刪除環境設定。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/environments/Claim/action | 宣告環境並將它指派給使用者 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/environments/delete | 刪除環境。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/environments/read | 讀取環境。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/environments/Start/action | 啟動環境內的所有資源以啟動環境。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/environments/Stop/action | 停止環境內的所有資源以停止環境 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/environments/write | 新增或修改環境。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/Publish/action | 根據實驗室/環境設定的目前狀態，佈建/取消佈建環境設定所需的資源。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/read | 讀取環境設定。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/Start/action | 啟動範本內的所有資源以啟動範本。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/Stop/action | 啟動範本內的所有資源以啟動範本。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/environmentSettings/write | 新增或修改環境設定。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/read | 讀取實驗室。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/Register/action | 向受控實驗室註冊。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/users/delete | 刪除使用者。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/users/read | 讀取使用者。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/users/write | 新增或修改使用者。 |
> | 動作 | Microsoft.LabServices/labAccounts/labs/write | 新增或修改實驗室。 |
> | 動作 | Microsoft.LabServices/labAccounts/read | 讀取實驗室帳戶。 |
> | 動作 | Microsoft.LabServices/labAccounts/write | 新增或修改實驗室帳戶。 |
> | 動作 | Microsoft.LabServices/locations/operations/read | 讀取作業。 |
> | 動作 | Microsoft.LabServices/register/action | 註冊訂用帳戶 |
> | 動作 | Microsoft.LabServices/users/GetEnvironment/action | 取得虛擬機器詳細資料 |
> | 動作 | Microsoft.LabServices/users/GetOperationBatchStatus/action | 取得批次作業狀態 |
> | 動作 | Microsoft.LabServices/users/GetOperationStatus/action | 取得長時間執行之作業的狀態 |
> | 動作 | Microsoft.LabServices/users/ListEnvironments/action | 列出使用者的環境 |
> | 動作 | Microsoft.LabServices/users/ListLabs/action | 列出使用者的實驗室。 |
> | 動作 | Microsoft.LabServices/users/Register/action | 向受控實驗室註冊使用者 |
> | 動作 | Microsoft.LabServices/users/StartEnvironment/action | 啟動環境內的所有資源以啟動環境。 |
> | 動作 | Microsoft.LabServices/users/StopEnvironment/action | 停止環境內的所有資源以停止環境 |

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.LocationBasedServices/accounts/delete | (已淘汰：請使用 /providers/Microsoft.Maps) 刪除位置服務帳戶。 |
> | 動作 | Microsoft.LocationBasedServices/accounts/listKeys/action | (已淘汰：請使用 /providers/Microsoft.Maps) 列出位置服務帳戶金鑰 |
> | 動作 | Microsoft.LocationBasedServices/accounts/providers/Microsoft.Insights/diagnosticSettings/read | (已淘汰：請使用 /providers/Microsoft.Maps) 取得資源的診斷設定 |
> | 動作 | Microsoft.LocationBasedServices/accounts/providers/Microsoft.Insights/diagnosticSettings/write | (已淘汰：請使用 /providers/Microsoft.Maps) 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.LocationBasedServices/accounts/providers/Microsoft.Insights/metricDefinitions/read | (已淘汰：請使用 /providers/Microsoft.Maps) 取得位置服務帳戶的可用計量 |
> | 動作 | Microsoft.LocationBasedServices/accounts/read | (已淘汰：請使用 /providers/Microsoft.Maps) 取得位置服務帳戶。 |
> | 動作 | Microsoft.LocationBasedServices/accounts/regenerateKey/action | (已淘汰：請使用 /providers/Microsoft.Maps) 產生新位置服務帳戶的主要或次要金鑰 |
> | 動作 | Microsoft.LocationBasedServices/accounts/write | (已淘汰：請使用 /providers/Microsoft.Maps) 建立或更新位置服務帳戶。 |
> | 動作 | Microsoft.LocationBasedServices/register/action | (已淘汰：請使用 /providers/Microsoft.Maps) 註冊提供者 |

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.LocationServices/accounts/delete | (已淘汰：請使用 /providers/Microsoft.Maps) 刪除位置服務帳戶。 |
> | 動作 | Microsoft.LocationServices/accounts/listKeys/action | (已淘汰：請使用 /providers/Microsoft.Maps) 列出位置服務帳戶金鑰 |
> | 動作 | Microsoft.LocationServices/accounts/providers/Microsoft.Insights/diagnosticSettings/read | (已淘汰：請使用 /providers/Microsoft.Maps) 取得資源的診斷設定 |
> | 動作 | Microsoft.LocationServices/accounts/providers/Microsoft.Insights/diagnosticSettings/write | (已淘汰：請使用 /providers/Microsoft.Maps) 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.LocationServices/accounts/providers/Microsoft.Insights/metricDefinitions/read | (已淘汰：請使用 /providers/Microsoft.Maps) 取得位置服務帳戶的可用計量 |
> | 動作 | Microsoft.LocationServices/accounts/read | (已淘汰：請使用 /providers/Microsoft.Maps) 取得位置服務帳戶。 |
> | 動作 | Microsoft.LocationServices/accounts/regenerateKey/action | (已淘汰：請使用 /providers/Microsoft.Maps) 產生新位置服務帳戶的主要或次要金鑰 |
> | 動作 | Microsoft.LocationServices/accounts/write | (已淘汰：請使用 /providers/Microsoft.Maps) 建立或更新位置服務帳戶。 |
> | 動作 | Microsoft.LocationServices/register/action | (已淘汰：請使用 /providers/Microsoft.Maps) 註冊提供者 |

## <a name="microsoftloganalytics"></a>Microsoft.LogAnalytics

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | DataAction | Microsoft.LogAnalytics/logs/ADAssessmentRecommendation/read | 從 ADAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ADReplicationResult/read | 從 ADReplicationResult 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ADSecurityAssessmentRecommendation/read | 從 ADSecurityAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/Alert/read | 從 Alert 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/AlertHistory/read | 從 AlertHistory 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ApplicationInsights/read | 從 ApplicationInsights 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/AuditLogs/read | 從 AuditLogs 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/AzureActivity/read | 從 AzureActivity 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/AzureMetrics/read | 從 AzureMetrics 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/BoundPort/read | 從 BoundPort 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/CommonSecurityLog/read | 從 CommonSecurityLog 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ComputerGroup/read | 從 ComputerGroup 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ConfigurationChange/read | 從 ConfigurationChange 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ConfigurationData/read | 從 ConfigurationData 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ContainerImageInventory/read | 從 ContainerImageInventory 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ContainerInventory/read | 從 ContainerInventory 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ContainerLog/read | 從 ContainerLog 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ContainerServiceLog/read | 從 ContainerServiceLog 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceAppCrash/read | 從 DeviceAppCrash 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceAppLaunch/read | 從 DeviceAppLaunch 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceCalendar/read | 從 DeviceCalendar 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceCleanup/read | 從 DeviceCleanup 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceConnectSession/read | 從 DeviceConnectSession 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceEtw/read | 從 DeviceEtw 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceHardwareHealth/read | 從 DeviceHardwareHealth 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceHealth/read | 從 DeviceHealth 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceHeartbeat/read | 從 DeviceHeartbeat 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceSkypeHeartbeat/read | 從 DeviceSkypeHeartbeat 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceSkypeSignIn/read | 從 DeviceSkypeSignIn 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DeviceSleepState/read | 從 DeviceSleepState 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DHAppFailure/read | 從 DHAppFailure 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DHAppReliability/read | 從 DHAppReliability 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DHDriverReliability/read | 從 DHDriverReliability 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DHLogonFailures/read | 從 DHLogonFailures 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DHLogonMetrics/read | 從 DHLogonMetrics 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DHOSCrashData/read | 從 DHOSCrashData 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DHOSReliability/read | 從 DHOSReliability 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DHWipAppLearning/read | 從 DHWipAppLearning 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DnsEvents/read | 從 DnsEvents 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/DnsInventory/read | 從 DnsInventory 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ETWEvent/read | 從 ETWEvent 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/Event/read | 從 Event 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ExchangeAssessmentRecommendation/read | 從 ExchangeAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ExchangeOnlineAssessmentRecommendation/read | 從 ExchangeOnlineAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/Heartbeat/read | 從 Heartbeat 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/IISAssessmentRecommendation/read | 從 IISAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/InboundConnection/read | 從 InboundConnection 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/KubeNodeInventory/read | 從 KubeNodeInventory 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/KubePodInventory/read | 從 KubePodInventory 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/LinuxAuditLog/read | 從 LinuxAuditLog 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAApplication/read | 從 MAApplication 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAApplicationHealth/read | 從 MAApplicationHealth 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAApplicationHealthAlternativeVersions/read | 從 MAApplicationHealthAlternativeVersions 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAApplicationHealthIssues/read | 從 MAApplicationHealthIssues 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAApplicationInstance/read | 從 MAApplicationInstance 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAApplicationInstanceReadiness/read | 從 MAApplicationInstanceReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAApplicationReadiness/read | 從 MAApplicationReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MADeploymentPlan/read | 從 MADeploymentPlan 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MADevice/read | 從 MADevice 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MADevicePnPHealth/read | 從 MADevicePnPHealth 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MADevicePnPHealthAlternativeVersions/read | 從 MADevicePnPHealthAlternativeVersions 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MADevicePnPHealthIssues/read | 從 MADevicePnPHealthIssues 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MADeviceReadiness/read | 從 MADeviceReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MADriverInstanceReadiness/read | 從 MADriverInstanceReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MADriverReadiness/read | 從 MADriverReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeAddin/read | 從 MAOfficeAddin 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeAddinHealth/read | 從 MAOfficeAddinHealth 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeAddinHealthIssues/read | 從 MAOfficeAddinHealthIssues 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeAddinInstance/read | 從 MAOfficeAddinInstance 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeAddinInstanceReadiness/read | 從 MAOfficeAddinInstanceReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeAddinReadiness/read | 從 MAOfficeAddinReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeApp/read | 從 MAOfficeApp 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeAppHealth/read | 從 MAOfficeAppHealth 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeAppInstance/read | 從 MAOfficeAppInstance 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeAppReadiness/read | 從 MAOfficeAppReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeBuildInfo/read | 從 MAOfficeBuildInfo 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeCurrencyAssessment/read | 從 MAOfficeCurrencyAssessment 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeCurrencyAssessmentDailyCounts/read | 從 MAOfficeCurrencyAssessmentDailyCounts 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeDeploymentStatus/read | 從 MAOfficeDeploymentStatus 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeMacroHealth/read | 從 MAOfficeMacroHealth 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeMacroHealthIssues/read | 從 MAOfficeMacroHealthIssues 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeMacroIssueInstanceReadiness/read | 從 MAOfficeMacroIssueInstanceReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeMacroIssueReadiness/read | 從 MAOfficeMacroIssueReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeMacroSummary/read | 從 MAOfficeMacroSummary 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeSuite/read | 從 MAOfficeSuite 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAOfficeSuiteInstance/read | 從 MAOfficeSuiteInstance 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAProposedPilotDevices/read | 從 MAProposedPilotDevices 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAWindowsBuildInfo/read | 從 MAWindowsBuildInfo 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAWindowsCurrencyAssessment/read | 從 MAWindowsCurrencyAssessment 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAWindowsCurrencyAssessmentDailyCounts/read | 從 MAWindowsCurrencyAssessmentDailyCounts 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAWindowsDeploymentStatus/read | 從 MAWindowsDeploymentStatus 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/MAWindowsSysReqInstanceReadiness/read | 從 MAWindowsSysReqInstanceReadiness 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/NetworkMonitoring/read | 從 NetworkMonitoring 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/OfficeActivity/read | 從 OfficeActivity 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/Operation/read | 從 Operation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/OutboundConnection/read | 從 OutboundConnection 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/Perf/read | 從 Perf 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ProtectionStatus/read | 從 ProtectionStatus 資料表讀取資料 |
> | 動作 | Microsoft.LogAnalytics/logs/read | 從您的所有記錄檔讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ReservedAzureCommonFields/read | 從 ReservedAzureCommonFields 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ReservedCommonFields/read | 從 ReservedCommonFields 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SCCMAssessmentRecommendation/read | 從 SCCMAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SCOMAssessmentRecommendation/read | 從 SCOMAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SecurityAlert/read | 從 SecurityAlert 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SecurityBaseline/read | 從 SecurityBaseline 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SecurityBaselineSummary/read | 從 SecurityBaselineSummary 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SecurityDetection/read | 從 SecurityDetection 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SecurityEvent/read | 從 SecurityEvent 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ServiceFabricOperationalEvent/read | 從 ServiceFabricOperationalEvent 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ServiceFabricReliableActorEvent/read | 從 ServiceFabricReliableActorEvent 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/ServiceFabricReliableServiceEvent/read | 從 ServiceFabricReliableServiceEvent 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SfBAssessmentRecommendation/read | 從 SfBAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SfBOnlineAssessmentRecommendation/read | 從 SfBOnlineAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SharePointOnlineAssessmentRecommendation/read | 從 SharePointOnlineAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SigninLogs/read | 從 SigninLogs 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SPAssessmentRecommendation/read | 從 SPAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SQLAssessmentRecommendation/read | 從 SQLAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SQLQueryPerformance/read | 從 SQLQueryPerformance 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/Syslog/read | 從 Syslog 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/SysmonEvent/read | 從 SysmonEvent 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/Tables.Custom/read | 從任何自訂記錄檔讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UAApp/read | 從 UAApp 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UAComputer/read | 從 UAComputer 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UAComputerRank/read | 從 UAComputerRank 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UADriver/read | 從 UADriver 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UADriverProblemCodes/read | 從 UADriverProblemCodes 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UAFeedback/read | 從 UAFeedback 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UAHardwareSecurity/read | 從 UAHardwareSecurity 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UAIESiteDiscovery/read | 從 UAIESiteDiscovery 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UAOfficeAddIn/read | 從 UAOfficeAddIn 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UAProposedActionPlan/read | 從 UAProposedActionPlan 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UASysReqIssue/read | 從 UASysReqIssue 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UAUpgradedComputer/read | 從 UAUpgradedComputer 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/Update/read | 從 Update 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UpdateRunProgress/read | 從 UpdateRunProgress 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/UpdateSummary/read | 從 UpdateSummary 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/Usage/read | 從 Usage 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/VMBoundPort/read | 從 VMBoundPort 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/VMConnection/read | 從 VMConnection 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/W3CIISLog/read | 從 W3CIISLog 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WaaSDeploymentStatus/read | 從 WaaSDeploymentStatus 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WaaSInsiderStatus/read | 從 WaaSInsiderStatus 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WaaSUpdateStatus/read | 從 WaaSUpdateStatus 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WDAVStatus/read | 從 WDAVStatus 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WDAVThreat/read | 從 WDAVThreat 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WindowsClientAssessmentRecommendation/read | 從 WindowsClientAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WindowsEvent/read | 從 WindowsEvent 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WindowsFirewall/read | 從 WindowsFirewall 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WindowsServerAssessmentRecommendation/read | 從 WindowsServerAssessmentRecommendation 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WireData/read | 從 WireData 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WorkloadMonitoringPerf/read | 從 WorkloadMonitoringPerf 資料表中讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WUDOAggregatedStatus/read | 從 WUDOAggregatedStatus 資料表讀取資料 |
> | DataAction | Microsoft.LogAnalytics/logs/WUDOStatus/read | 從 WUDOStatus 資料表讀取資料 |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Logic/integrationAccounts/agreements/delete | 刪除整合帳戶中的合約。 |
> | 動作 | Microsoft.Logic/integrationAccounts/agreements/listContentCallbackUrl/action | 取得整合帳戶中合約內容的回呼 URL。 |
> | 動作 | Microsoft.Logic/integrationAccounts/agreements/read | 讀取整合帳戶中的合約。 |
> | 動作 | Microsoft.Logic/integrationAccounts/agreements/write | 建立或更新整合帳戶中的合約。 |
> | 動作 | Microsoft.Logic/integrationAccounts/assemblies/delete | 刪除整合帳戶中的組件。 |
> | 動作 | Microsoft.Logic/integrationAccounts/assemblies/listContentCallbackUrl/action | 取得整合帳戶中組件內容的回呼 URL。 |
> | 動作 | Microsoft.Logic/integrationAccounts/assemblies/read | 讀取整合帳戶中的組件。 |
> | 動作 | Microsoft.Logic/integrationAccounts/assemblies/write | 建立或更新整合帳戶中的組件。 |
> | 動作 | Microsoft.Logic/integrationAccounts/batchConfigurations/delete | 刪除整合帳戶中的批次設定。 |
> | 動作 | Microsoft.Logic/integrationAccounts/batchConfigurations/read | 讀取整合帳戶中的批次設定。 |
> | 動作 | Microsoft.Logic/integrationAccounts/batchConfigurations/write | 建立或更新整合帳戶中的批次設定。 |
> | 動作 | Microsoft.Logic/integrationAccounts/certificates/delete | 刪除整合帳戶中的憑證。 |
> | 動作 | Microsoft.Logic/integrationAccounts/certificates/read | 讀取整合帳戶中的憑證。 |
> | 動作 | Microsoft.Logic/integrationAccounts/certificates/write | 建立或更新整合帳戶中的憑證。 |
> | 動作 | Microsoft.Logic/integrationAccounts/delete | 刪除整合帳戶。 |
> | 動作 | Microsoft.Logic/integrationAccounts/join/action | 加入整合帳戶。 |
> | 動作 | Microsoft.Logic/integrationAccounts/listCallbackUrl/action | 取得整合帳戶的回呼 URL。 |
> | 動作 | Microsoft.Logic/integrationAccounts/listKeyVaultKeys/action | 取得金鑰保存庫中的金鑰。 |
> | 動作 | Microsoft.Logic/integrationAccounts/logTrackingEvents/action | 記錄整合帳戶中的追蹤事件。 |
> | 動作 | Microsoft.Logic/integrationAccounts/maps/delete | 刪除整合帳戶中的對應。 |
> | 動作 | Microsoft.Logic/integrationAccounts/maps/listContentCallbackUrl/action | 取得整合帳戶中對應內容的回呼 URL。 |
> | 動作 | Microsoft.Logic/integrationAccounts/maps/read | 讀取整合帳戶中的對應。 |
> | 動作 | Microsoft.Logic/integrationAccounts/maps/write | 建立或更新整合帳戶中的對應。 |
> | 動作 | Microsoft.Logic/integrationAccounts/partners/delete | 刪除整合帳戶中的合作夥伴。 |
> | 動作 | Microsoft.Logic/integrationAccounts/partners/listContentCallbackUrl/action | 取得整合帳戶中合作夥伴內容的回呼 URL。 |
> | 動作 | Microsoft.Logic/integrationAccounts/partners/read | 讀取整合帳戶中的合作夥伴。 |
> | 動作 | Microsoft.Logic/integrationAccounts/partners/write | 建立或更新整合帳戶中的合作夥伴。 |
> | 動作 | Microsoft.Logic/integrationAccounts/providers/Microsoft.Insights/logDefinitions/read | 讀取整合帳戶記錄定義。 |
> | 動作 | Microsoft.Logic/integrationAccounts/read | 讀取整合帳戶。 |
> | 動作 | Microsoft.Logic/integrationAccounts/regenerateAccessKey/action | 重新產生存取金鑰祕密。 |
> | 動作 | Microsoft.Logic/integrationAccounts/schemas/delete | 刪除整合帳戶中的結構描述。 |
> | 動作 | Microsoft.Logic/integrationAccounts/schemas/listContentCallbackUrl/action | 取得整合帳戶中結構描述內容的回呼 URL。 |
> | 動作 | Microsoft.Logic/integrationAccounts/schemas/read | 讀取整合帳戶中的結構描述。 |
> | 動作 | Microsoft.Logic/integrationAccounts/schemas/write | 建立或更新整合帳戶中的結構描述。 |
> | 動作 | Microsoft.Logic/integrationAccounts/sessions/delete | 刪除整合帳戶中的工作階段。 |
> | 動作 | Microsoft.Logic/integrationAccounts/sessions/read | 讀取整合帳戶中的批次設定。 |
> | 動作 | Microsoft.Logic/integrationAccounts/sessions/write | 建立或更新整合帳戶中的工作階段。 |
> | 動作 | Microsoft.Logic/integrationAccounts/write | 建立或更新整合帳戶。 |
> | 動作 | Microsoft.Logic/integrationServiceEnvironments/delete | 刪除整合服務環境。 |
> | 動作 | Microsoft.Logic/integrationServiceEnvironments/join/action | 聯結整合服務環境。 |
> | 動作 | Microsoft.Logic/integrationServiceEnvironments/managedApis/apiOperations/read | 讀取整合服務環境受控 API 作業。 |
> | 動作 | Microsoft.Logic/integrationServiceEnvironments/managedApis/read | 讀取整合服務環境受控 API。 |
> | 動作 | Microsoft.Logic/integrationServiceEnvironments/providers/Microsoft.Insights/metricDefinitions/read | 讀取整合服務環境計量定義。 |
> | 動作 | Microsoft.Logic/integrationServiceEnvironments/read | 讀取整合服務環境。 |
> | 動作 | Microsoft.Logic/integrationServiceEnvironments/write | 建立或更新整合服務環境。 |
> | 動作 | Microsoft.Logic/locations/workflows/recommendOperationGroups/action | 取得工作流程建議的作業群組。 |
> | 動作 | Microsoft.Logic/locations/workflows/validate/action | 驗證工作流程。 |
> | 動作 | Microsoft.Logic/operations/read | 取得作業。 |
> | 動作 | Microsoft.Logic/register/action | 為指定的訂用帳戶註冊 Microsoft.Logic 資源提供者。 |
> | 動作 | Microsoft.Logic/workflows/accessKeys/delete | 刪除存取金鑰。 |
> | 動作 | Microsoft.Logic/workflows/accessKeys/list/action | 列出存取金鑰祕密。 |
> | 動作 | Microsoft.Logic/workflows/accessKeys/read | 讀取存取金鑰。 |
> | 動作 | Microsoft.Logic/workflows/accessKeys/regenerate/action | 重新產生存取金鑰祕密。 |
> | 動作 | Microsoft.Logic/workflows/accessKeys/write | 建立或更新存取金鑰。 |
> | 動作 | Microsoft.Logic/workflows/delete | 刪除工作流程。 |
> | 動作 | Microsoft.Logic/workflows/disable/action | 停用工作流程。 |
> | 動作 | Microsoft.Logic/workflows/enable/action | 啟用工作流程。 |
> | 動作 | Microsoft.Logic/workflows/listCallbackUrl/action | 取得工作流程的回呼 URL。 |
> | 動作 | Microsoft.Logic/workflows/listSwagger/action | 取得工作流程 Swagger 定義。 |
> | 動作 | Microsoft.Logic/workflows/move/action | 將工作流程從其現有的訂用帳戶識別碼、資源群組和/或名稱改為不同的訂用帳戶識別碼、資源群組和/或名稱。 |
> | 動作 | Microsoft.Logic/workflows/providers/Microsoft.Insights/diagnosticSettings/read | 讀取工作流程診斷設定。 |
> | 動作 | Microsoft.Logic/workflows/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新工作流程診斷設定。 |
> | 動作 | Microsoft.Logic/workflows/providers/Microsoft.Insights/logDefinitions/read | 讀取工作流程記錄定義。 |
> | 動作 | Microsoft.Logic/workflows/providers/Microsoft.Insights/metricDefinitions/read | 讀取工作流程計量定義。 |
> | 動作 | Microsoft.Logic/workflows/read | 讀取工作流程。 |
> | 動作 | Microsoft.Logic/workflows/regenerateAccessKey/action | 重新產生存取金鑰祕密。 |
> | 動作 | Microsoft.Logic/workflows/run/action | 啟動工作流程的執行。 |
> | 動作 | Microsoft.Logic/workflows/runs/actions/listExpressionTraces/action | 取得工作流程執行動作運算式追蹤。 |
> | 動作 | Microsoft.Logic/workflows/runs/actions/read | 讀取工作流程的執行動作。 |
> | 動作 | Microsoft.Logic/workflows/runs/actions/repetitions/listExpressionTraces/action | 取得工作流程執行動作重複作業運算式追蹤。 |
> | 動作 | Microsoft.Logic/workflows/runs/actions/repetitions/read | 讀取工作流程執行動作重複作業。 |
> | 動作 | Microsoft.Logic/workflows/runs/actions/repetitions/requestHistories/read | 讀取工作流程執行重複動作要求歷程記錄。 |
> | 動作 | Microsoft.Logic/workflows/runs/actions/requestHistories/read | 讀取工作流程執行動作要求歷程記錄。 |
> | 動作 | Microsoft.Logic/workflows/runs/actions/scoperepetitions/read | 讀取工作流程執行動作範圍重複作業。 |
> | 動作 | Microsoft.Logic/workflows/runs/cancel/action | 取消工作流程的執行。 |
> | 動作 | Microsoft.Logic/workflows/runs/operations/read | 讀取工作流程的執行作業狀態。 |
> | 動作 | Microsoft.Logic/workflows/runs/read | 讀取工作流程的執行。 |
> | 動作 | Microsoft.Logic/workflows/suspend/action | 暫止工作流程。 |
> | 動作 | Microsoft.Logic/workflows/triggers/histories/read | 讀取觸發程序歷程記錄。 |
> | 動作 | Microsoft.Logic/workflows/triggers/histories/resubmit/action | 重新提交工作流程的觸發程序。 |
> | 動作 | Microsoft.Logic/workflows/triggers/listCallbackUrl/action | 取得觸發程序的回呼 URL。 |
> | 動作 | Microsoft.Logic/workflows/triggers/read | 讀取觸發程序。 |
> | 動作 | Microsoft.Logic/workflows/triggers/reset/action | 重設觸發程序。 |
> | 動作 | Microsoft.Logic/workflows/triggers/run/action | 執行觸發程序。 |
> | 動作 | Microsoft.Logic/workflows/triggers/setState/action | 設定觸發程序狀態。 |
> | 動作 | Microsoft.Logic/workflows/validate/action | 驗證工作流程。 |
> | 動作 | Microsoft.Logic/workflows/versions/read | 讀取工作流程版本。 |
> | 動作 | Microsoft.Logic/workflows/versions/triggers/listCallbackUrl/action | 取得觸發程序的回呼 URL。 |
> | 動作 | Microsoft.Logic/workflows/write | 建立或更新工作流程。 |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.MachineLearning/commitmentPlans/commitmentAssociations/move/action | 移動任何 Machine Learning 承諾用量方案關聯 |
> | 動作 | Microsoft.MachineLearning/commitmentPlans/commitmentAssociations/read | 讀取任何 Machine Learning 承諾用量方案關聯 |
> | 動作 | Microsoft.MachineLearning/commitmentPlans/delete | 刪除任何 Machine Learning 承諾用量方案 |
> | 動作 | Microsoft.MachineLearning/commitmentPlans/join/action | 加入任何 Machine Learning 承諾用量方案 |
> | 動作 | Microsoft.MachineLearning/commitmentPlans/read | 讀取任何 Machine Learning 承諾用量方案 |
> | 動作 | Microsoft.MachineLearning/commitmentPlans/write | 建立或更新任何 Machine Learning 承諾用量方案 |
> | 動作 | Microsoft.MachineLearning/locations/operationresults/read | 取得 Machine Learning 作業的結果 |
> | 動作 | Microsoft.MachineLearning/locations/operationsstatus/read | 取得執行中 Machine Learning 作業的狀態 |
> | 動作 | Microsoft.MachineLearning/operations/read | 取得 Machine Learning 作業 |
> | 動作 | Microsoft.MachineLearning/register/action | 針對 Machine Learning Web 服務資源提供者註冊訂用帳戶，並讓您能夠建立 Web 服務。 |
> | 動作 | Microsoft.MachineLearning/skus/read | 取得 Machine Learning 承諾用量方案 SKU |
> | 動作 | Microsoft.MachineLearning/webServices/action | 建立所支援區域的區域性 Web 服務屬性 |
> | 動作 | Microsoft.MachineLearning/webServices/delete | 刪除任何 Machine Learning Web 服務 |
> | 動作 | Microsoft.MachineLearning/webServices/listkeys/read | 取得 Machine Learning Web 服務的金鑰 |
> | 動作 | Microsoft.MachineLearning/webServices/read | 讀取任何 Machine Learning Web 服務 |
> | 動作 | Microsoft.MachineLearning/webServices/write | 建立或更新任何 Machine Learning Web 服務 |
> | 動作 | Microsoft.MachineLearning/Workspaces/delete | 刪除任何 Machine Learning 工作區 |
> | 動作 | Microsoft.MachineLearning/Workspaces/listworkspacekeys/action | 列出 Machine Learning 工作區的金鑰 |
> | 動作 | Microsoft.MachineLearning/Workspaces/read | 讀取任何 Machine Learning 工作區 |
> | 動作 | Microsoft.MachineLearning/Workspaces/resyncstoragekeys/action | 重新同步處理針對 Machine Learning 工作區所設定之儲存體帳戶的金鑰 |
> | 動作 | Microsoft.MachineLearning/Workspaces/write | 建立或更新任何 Machine Learning 工作區 |

## <a name="microsoftmachinelearningcompute"></a>Microsoft.MachineLearningCompute

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.MachineLearningCompute/operationalizationClusters/checkUpdate/action | 檢查是否有可供作業叢集之系統服務使用的更新 |
> | 動作 | Microsoft.MachineLearningCompute/operationalizationClusters/delete | 刪除任何裝載帳戶 |
> | 動作 | Microsoft.MachineLearningCompute/operationalizationClusters/listKeys/action | 列出與作業叢集相關聯的金鑰 |
> | 動作 | Microsoft.MachineLearningCompute/operationalizationClusters/read | 讀取任何裝載帳戶 |
> | 動作 | Microsoft.MachineLearningCompute/operationalizationClusters/updateSystem/action | 更新作業叢集中的系統服務 |
> | 動作 | Microsoft.MachineLearningCompute/operationalizationClusters/write | 建立或更新任何裝載帳戶 |
> | 動作 | Microsoft.MachineLearningCompute/register/action | 向資源提供者註冊訂用帳戶識別碼，並讓您能夠建立機器學習服務計算資源 |

## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.MachineLearningModelManagement/accounts/delete | 刪除任何裝載帳戶 |
> | 動作 | Microsoft.MachineLearningModelManagement/accounts/read | 讀取任何裝載帳戶 |
> | 動作 | Microsoft.MachineLearningModelManagement/accounts/write | 建立或更新任何裝載帳戶 |
> | 動作 | Microsoft.MachineLearningModelManagement/register/action | 向資源提供者註冊訂用帳戶識別碼，並讓您能夠建立裝載帳戶 |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.MachineLearningServices/register/action | 為機器學習服務資源提供者註冊訂用帳戶 |
> | 動作 | Microsoft.MachineLearningServices/workspaces/computes/delete | 刪除機器學習服務工作區中的計算資源 |
> | 動作 | Microsoft.MachineLearningServices/workspaces/computes/listKeys/action | 列出機器學習服務工作區中的計算資源祕密 |
> | 動作 | Microsoft.MachineLearningServices/workspaces/computes/read | 取得機器學習服務工作區中的計算資源 |
> | 動作 | Microsoft.MachineLearningServices/workspaces/computes/write | 在機器學習服務工作區中建立或更新計算資源 |
> | 動作 | Microsoft.MachineLearningServices/workspaces/delete | 刪除機器學習服務工作區 |
> | 動作 | Microsoft.MachineLearningServices/workspaces/listKeys/action | 列出機器學習服務工作區的祕密 |
> | 動作 | Microsoft.MachineLearningServices/workspaces/read | 取得機器學習服務工作區 |
> | 動作 | Microsoft.MachineLearningServices/workspaces/write | 建立或更新機器學習服務工作區 |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ManagedIdentity/userAssignedIdentities/assign/action | 用來將現有已指派使用者的身分識別指派給資源的 RBAC 動作 |
> | 動作 | Microsoft.ManagedIdentity/userAssignedIdentities/delete | 刪除現有已指派使用者的身分識別 |
> | 動作 | Microsoft.ManagedIdentity/userAssignedIdentities/read | 取得現有已指派使用者的身分識別 |
> | 動作 | Microsoft.ManagedIdentity/userAssignedIdentities/write | 建立新的已指派使用者的身分識別，或更新與現有已指派使用者之身分識別相關聯的標記 |

## <a name="microsoftmanagedlab"></a>Microsoft.ManagedLab

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ManagedLab/labAccounts/CreateLab/action | 在實驗室帳戶中建立實驗室。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/delete | 刪除實驗室帳戶。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/delete | 刪除實驗室。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/environmentSettings/delete | 刪除環境設定。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/environmentSettings/environments/delete | 刪除環境。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/environmentSettings/environments/read | 讀取環境。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/environmentSettings/environments/write | 新增或修改環境。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/environmentSettings/read | 讀取環境設定。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/environmentSettings/write | 新增或修改環境設定。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/labVms/delete | 刪除實驗室虛擬機器。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/labVms/read | 讀取實驗室虛擬機器。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/labVms/write | 新增或修改實驗室虛擬機器。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/read | 讀取實驗室。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/labs/write | 新增或修改實驗室。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/read | 讀取實驗室帳戶。 |
> | 動作 | Microsoft.ManagedLab/labAccounts/write | 新增或修改實驗室帳戶。 |
> | 動作 | Microsoft.ManagedLab/locations/operations/read | 讀取作業。 |
> | 動作 | Microsoft.ManagedLab/register/action | 註冊訂用帳戶 |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Management/checkNameAvailability/action | 檢查指定的管理群組名稱是否有效且唯一。 |
> | 動作 | Microsoft.Management/getEntities/action | 列出已驗證之使用者的所有實體 (管理群組、訂用帳戶等)。 |
> | 動作 | Microsoft.Management/managementGroups/delete | 刪除管理群組。 |
> | 動作 | Microsoft.Management/managementGroups/read | 列出已驗證之使用者的管理群組。 |
> | 動作 | Microsoft.Management/managementGroups/subscriptions/delete | 從管理群組中取消訂用帳戶的關聯。 |
> | 動作 | Microsoft.Management/managementGroups/subscriptions/write | 將現有的訂用帳戶關聯至管理群組。 |
> | 動作 | Microsoft.Management/managementGroups/write | 建立或更新管理群組。 |
> | 動作 | Microsoft.Management/register/action | 向 Microsoft.Management 註冊指定的訂用帳戶 |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | DataAction | Microsoft.Maps/accounts/data/read | 將資料讀取權限授與地圖服務帳戶。 |
> | 動作 | Microsoft.Maps/accounts/delete | 刪除地圖服務帳戶。 |
> | 動作 | Microsoft.Maps/accounts/listKeys/action | 列出地圖服務帳戶金鑰 |
> | 動作 | Microsoft.Maps/accounts/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Maps/accounts/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Maps/accounts/providers/Microsoft.Insights/metricDefinitions/read | 取得地圖服務帳戶的可用計量 |
> | 動作 | Microsoft.Maps/accounts/read | 取得地圖服務帳戶。 |
> | 動作 | Microsoft.Maps/accounts/regenerateKey/action | 產生新地圖服務帳戶的主要或次要金鑰 |
> | 動作 | Microsoft.Maps/accounts/write | 建立或更新地圖服務帳戶。 |
> | 動作 | Microsoft.Maps/register/action | 註冊提供者 |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Marketplace/offerTypes/publishers/offers/plans/agreements/read | 傳回協議。 |
> | 動作 | Microsoft.Marketplace/offerTypes/publishers/offers/plans/agreements/write | 接受已簽署的協議。 |
> | 動作 | Microsoft.Marketplace/offerTypes/publishers/offers/plans/configs/importImage/action | 將映像匯入至終端使用者的 ACR。 |
> | 動作 | Microsoft.Marketplace/offerTypes/publishers/offers/plans/configs/read | 傳回組態。 |
> | 動作 | Microsoft.Marketplace/offerTypes/publishers/offers/plans/configs/write | 儲存組態。 |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.MarketplaceApps/ClassicDevServices/delete | 在傳統開發服務資源上進行 DELETE 作業。 |
> | 動作 | Microsoft.MarketplaceApps/ClassicDevServices/listSecrets/action | 取得傳統開發服務資源管理金鑰。 |
> | 動作 | Microsoft.MarketplaceApps/ClassicDevServices/listSingleSignOnToken/action | 取得傳統開發服務的單一登入 URL。 |
> | 動作 | Microsoft.MarketplaceApps/ClassicDevServices/read | 在傳統開發服務上進行 GET 作業。 |
> | 動作 | Microsoft.MarketplaceApps/ClassicDevServices/regenerateKey/action | 產生傳統開發服務資源管理金鑰。 |
> | 動作 | Microsoft.MarketplaceApps/Operations/read | 讀取所有資源類型的作業。 |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.MarketplaceOrdering/agreements/offers/plans/cancel/action | 取消給定 Marketplace 項目的合約 |
> | 動作 | Microsoft.MarketplaceOrdering/agreements/offers/plans/read | 傳回給定 Marketplace 項目的合約 |
> | 動作 | Microsoft.MarketplaceOrdering/agreements/offers/plans/sign/action | 簽署給定 Marketplace 項目的合約 |
> | 動作 | Microsoft.MarketplaceOrdering/agreements/read | 傳回指定訂用帳戶下的所有合約 |
> | 動作 | Microsoft.MarketplaceOrdering/offertypes/publishers/offers/plans/agreements/read | 取得指定 Marketplace 虛擬機器項目的合約 |
> | 動作 | Microsoft.MarketplaceOrdering/offertypes/publishers/offers/plans/agreements/write | 簽署或取消指定 Marketplace 虛擬機器項目的合約 |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Media/checknameavailability/action | 檢查媒體服務帳戶名稱是否可供使用 |
> | 動作 | Microsoft.Media/locations/checkNameAvailability/action | 檢查媒體服務帳戶名稱是否可供使用 |
> | 動作 | Microsoft.Media/mediaservices/assets/delete | 刪除任何資產 |
> | 動作 | Microsoft.Media/mediaservices/assets/getEncryptionKey/action | 取得資產加密金鑰 |
> | 動作 | Microsoft.Media/mediaservices/assets/listContainerSas/action | 列出資產容器 SAS URL |
> | 動作 | Microsoft.Media/mediaservices/assets/read | 讀取任何資產 |
> | 動作 | Microsoft.Media/mediaservices/assets/write | 建立或更新任何資產 |
> | 動作 | Microsoft.Media/mediaservices/contentKeyPolicies/delete | 刪除任何內容金鑰原則 |
> | 動作 | Microsoft.Media/mediaservices/contentKeyPolicies/getPolicyPropertiesWithSecrets/action | 取得含有祕密的原則屬性 |
> | 動作 | Microsoft.Media/mediaservices/contentKeyPolicies/read | 讀取任何內容金鑰原則 |
> | 動作 | Microsoft.Media/mediaservices/contentKeyPolicies/write | 建立或更新任何內容金鑰原則 |
> | 動作 | Microsoft.Media/mediaservices/delete | 刪除任何媒體服務帳戶 |
> | 動作 | Microsoft.Media/mediaservices/eventGridFilters/delete | 刪除任何事件方格篩選 |
> | 動作 | Microsoft.Media/mediaservices/eventGridFilters/read | 讀取任何事件方格篩選 |
> | 動作 | Microsoft.Media/mediaservices/eventGridFilters/write | 建立或更新任何事件方格篩選 |
> | 動作 | Microsoft.Media/mediaservices/liveEventOperations/read | 讀取任何即時事件作業 |
> | 動作 | Microsoft.Media/mediaservices/liveEvents/delete | 刪除任何即時事件 |
> | 動作 | Microsoft.Media/mediaservices/liveEvents/liveOutputs/delete | 刪除任何即時輸出 |
> | 動作 | Microsoft.Media/mediaservices/liveEvents/liveOutputs/read | 讀取任何即時輸出 |
> | 動作 | Microsoft.Media/mediaservices/liveEvents/liveOutputs/write | 建立或更新任何即時輸出 |
> | 動作 | Microsoft.Media/mediaservices/liveEvents/read | 讀取任何即時事件 |
> | 動作 | Microsoft.Media/mediaservices/liveEvents/reset/action | 重設任何即時事件作業 |
> | 動作 | Microsoft.Media/mediaservices/liveEvents/start/action | 啟動任何即時事件作業 |
> | 動作 | Microsoft.Media/mediaservices/liveEvents/stop/action | 停止任何即時事件作業 |
> | 動作 | Microsoft.Media/mediaservices/liveEvents/write | 建立或更新任何即時事件 |
> | 動作 | Microsoft.Media/mediaservices/liveOutputOperations/read | 讀取任何即時輸出作業 |
> | 動作 | Microsoft.Media/mediaservices/read | 讀取任何媒體服務帳戶 |
> | 動作 | Microsoft.Media/mediaservices/streamingEndpointOperations/read | 讀取任何串流端點作業 |
> | 動作 | Microsoft.Media/mediaservices/streamingEndpoints/delete | 刪除任何串流端點 |
> | 動作 | Microsoft.Media/mediaservices/streamingEndpoints/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定。 |
> | 動作 | Microsoft.Media/mediaservices/streamingEndpoints/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定。 |
> | 動作 | Microsoft.Media/mediaservices/streamingEndpoints/providers/Microsoft.Insights/metricDefinitions/read | 取得媒體服務串流端點計量定義的清單。 |
> | 動作 | Microsoft.Media/mediaservices/streamingEndpoints/read | 讀取任何串流端點 |
> | 動作 | Microsoft.Media/mediaservices/streamingEndpoints/scale/action | 調整任何串流端點作業 |
> | 動作 | Microsoft.Media/mediaservices/streamingEndpoints/start/action | 啟動任何串流端點作業 |
> | 動作 | Microsoft.Media/mediaservices/streamingEndpoints/stop/action | 停止任何串流端點作業 |
> | 動作 | Microsoft.Media/mediaservices/streamingEndpoints/write | 建立或更新任何串流端點 |
> | 動作 | Microsoft.Media/mediaservices/streamingLocators/delete | 刪除任何串流定位器 |
> | 動作 | Microsoft.Media/mediaservices/streamingLocators/listContentKeys/action | 列出內容金鑰 |
> | 動作 | Microsoft.Media/mediaservices/streamingLocators/listPaths/action | 列出路徑 |
> | 動作 | Microsoft.Media/mediaservices/streamingLocators/read | 讀取任何串流定位器 |
> | 動作 | Microsoft.Media/mediaservices/streamingLocators/write | 建立或更新任何串流定位器 |
> | 動作 | Microsoft.Media/mediaservices/streamingPolicies/delete | 刪除任何串流原則 |
> | 動作 | Microsoft.Media/mediaservices/streamingPolicies/read | 讀取任何串流原則 |
> | 動作 | Microsoft.Media/mediaservices/streamingPolicies/write | 建立或更新任何串流原則 |
> | 動作 | Microsoft.Media/mediaservices/syncStorageKeys/action | 同步處理已連結之 Azure 儲存體帳戶的儲存體金鑰 |
> | 動作 | Microsoft.Media/mediaservices/transforms/delete | 刪除任何轉換 |
> | 動作 | Microsoft.Media/mediaservices/transforms/jobs/cancelJob/action | 取消作業 |
> | 動作 | Microsoft.Media/mediaservices/transforms/jobs/delete | 刪除任何作業 |
> | 動作 | Microsoft.Media/mediaservices/transforms/jobs/read | 讀取任何作業 |
> | 動作 | Microsoft.Media/mediaservices/transforms/jobs/write | 建立或更新任何作業 |
> | 動作 | Microsoft.Media/mediaservices/transforms/read | 讀取任何轉換 |
> | 動作 | Microsoft.Media/mediaservices/transforms/write | 建立或更新任何轉換 |
> | 動作 | Microsoft.Media/mediaservices/write | 建立或更新任何媒體服務帳戶 |
> | 動作 | Microsoft.Media/operations/read | 取得可用的作業 |
> | 動作 | Microsoft.Media/register/action | 為媒體服務資源提供者註冊訂用帳戶，並讓您能夠建立媒體服務帳戶 |
> | 動作 | Microsoft.Media/unregister/action | 為媒體服務資源提供者取消註冊訂用帳戶 |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Migrate/locations/assessmentOptions/read | 取得可在指定位置使用的評定選項 |
> | 動作 | Microsoft.Migrate/locations/checknameavailability/action | 檢查指定位置中，指定訂用帳戶的資源名稱可用性 |
> | 動作 | Microsoft.Migrate/Operations/read | 列出可對 Microsoft.Migrate 資源提供者進行的作業 |
> | 動作 | Microsoft.Migrate/projects/assessments/read | 列出專案內的評定 |
> | 動作 | Microsoft.Migrate/projects/delete | 刪除專案 |
> | 動作 | Microsoft.Migrate/projects/groups/assessments/assessedmachines/read | 取得已評定電腦的屬性 |
> | 動作 | Microsoft.Migrate/projects/groups/assessments/delete | 刪除評定 |
> | 動作 | Microsoft.Migrate/projects/groups/assessments/downloadurl/action | 下載評定報告的 URL |
> | 動作 | Microsoft.Migrate/projects/groups/assessments/read | 取得評定的屬性 |
> | 動作 | Microsoft.Migrate/projects/groups/assessments/write | 建立新的評定或更新現有的評定 |
> | 動作 | Microsoft.Migrate/projects/groups/delete | 刪除群組 |
> | 動作 | Microsoft.Migrate/projects/groups/read | 取得群組的屬性 |
> | 動作 | Microsoft.Migrate/projects/groups/write | 建立新的群組或更新現有的群組 |
> | 動作 | Microsoft.Migrate/projects/keys/action | 取得專案的共用金鑰 |
> | 動作 | Microsoft.Migrate/projects/machines/read | 取得電腦的屬性 |
> | 動作 | Microsoft.Migrate/projects/read | 取得專案的屬性 |
> | 動作 | Microsoft.Migrate/projects/write | 建立新的專案或更新現有的專案 |
> | 動作 | Microsoft.Migrate/register/action | 向 Microsoft.Migrate 資源提供者註冊訂用帳戶 |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.NetApp/locations/operationresults/read | 讀取作業結果資源。 |
> | 動作 | Microsoft.NetApp/locations/read | 讀取可用性檢查資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/delete | 刪除集區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/providers/Microsoft.Insights/diagnosticSettings/read | 刪除集區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/providers/Microsoft.Insights/diagnosticSettings/write | 刪除集區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/providers/Microsoft.Insights/metricDefinitions/read | 刪除集區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/read | 讀取集區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/delete | 刪除磁碟區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/MountTargets/delete | 刪除掛接目標資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/MountTargets/read | 讀取掛接目標資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/MountTargets/write | 寫入掛接目標資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/providers/Microsoft.Insights/diagnosticSettings/read | 刪除磁碟區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/providers/Microsoft.Insights/diagnosticSettings/write | 刪除磁碟區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/providers/Microsoft.Insights/metricDefinitions/read | 刪除磁碟區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/read | 讀取磁碟區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/Snapshots/delete | 刪除快照集資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/Snapshots/read | 讀取快照集資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/Snapshots/write | 寫入快照集資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/write | 寫入磁碟區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/capacityPools/write | 寫入集區資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/delete | 刪除帳戶資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/read | 讀取帳戶資源。 |
> | 動作 | Microsoft.NetApp/netAppAccounts/write | 寫入帳戶資源。 |
> | 動作 | Microsoft.NetApp/Operations/read | 讀取作業資源。 |

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Network/applicationGatewayAvailableSslOptions/predefinedPolicies/read | 應用程式閘道 SSL 預先定義的原則 |
> | 動作 | Microsoft.Network/applicationGatewayAvailableSslOptions/read | 應用程式閘道可用的 SSL 選項 |
> | 動作 | Microsoft.Network/applicationGatewayAvailableWafRuleSets/read | 取得應用程式閘道可用的 WAF 規則集 |
> | 動作 | Microsoft.Network/applicationGateways/backendAddressPools/join/action | 加入應用程式閘道後端位址集區 |
> | 動作 | Microsoft.Network/applicationGateways/backendhealth/action | 取得應用程式閘道後端的健康狀態 |
> | 動作 | Microsoft.Network/applicationGateways/delete | 刪除應用程式閘道 |
> | 動作 | Microsoft.Network/applicationGateways/effectiveNetworkSecurityGroups/action | 取得應用程式閘道上所設定的路由表 |
> | 動作 | Microsoft.Network/applicationGateways/effectiveRouteTable/action | 取得應用程式閘道上所設定的路由表 |
> | 動作 | Microsoft.Network/applicationGateways/providers/Microsoft.Insights/logDefinitions/read | 取得應用程式閘道的事件 |
> | 動作 | Microsoft.Network/applicationGateways/providers/Microsoft.Insights/metricDefinitions/read | 取得應用程式閘道的可用計量 |
> | 動作 | Microsoft.Network/applicationGateways/read | 取得應用程式閘道 |
> | 動作 | Microsoft.Network/applicationGateways/setSecurityCenterConfiguration/action | 設定應用程式閘道資訊安全中心設定 |
> | 動作 | Microsoft.Network/applicationGateways/start/action | 啟動應用程式閘道 |
> | 動作 | Microsoft.Network/applicationGateways/stop/action | 停止應用程式閘道 |
> | 動作 | Microsoft.Network/applicationGateways/write | 建立或更新應用程式閘道 |
> | 動作 | Microsoft.Network/applicationSecurityGroups/delete | 刪除應用程式安全性群組 |
> | 動作 | Microsoft.Network/applicationSecurityGroups/joinIpConfiguration/action | 將 IP 設定加入至應用程式安全性群組。 |
> | 動作 | Microsoft.Network/applicationSecurityGroups/joinNetworkSecurityRule/action | 將安全性規則加入至應用程式安全性群組。 |
> | 動作 | Microsoft.Network/applicationSecurityGroups/read | 取得應用程式安全性群組識別碼。 |
> | 動作 | Microsoft.Network/applicationSecurityGroups/write | 建立應用程式安全性群組，或更新現有的應用程式安全性群組。 |
> | 動作 | Microsoft.Network/azureFirewallFqdnTags/read | 取得 Azure 防火牆 FQDN 標記 |
> | 動作 | Microsoft.Network/azurefirewalls/delete | 刪除 Azure 防火牆 |
> | 動作 | Microsoft.Network/azurefirewalls/providers/Microsoft.Insights/logDefinitions/read | 取得「Azure 防火牆」的事件 |
> | 動作 | Microsoft.Network/azurefirewalls/providers/Microsoft.Insights/metricDefinitions/read | 取得 Azure 防火牆的可用計量 |
> | 動作 | Microsoft.Network/azurefirewalls/read | 取得 Azure 防火牆 |
> | 動作 | Microsoft.Network/azurefirewalls/write | 建立或更新 Azure 防火牆 |
> | 動作 | Microsoft.Network/bgpServiceCommunities/read | 取得 BGP 服務社群 |
> | 動作 | Microsoft.Network/checkTrafficManagerNameAvailability/action | 檢查流量管理員相對 DNS 名稱的可用性。 |
> | 動作 | Microsoft.Network/connections/delete | 刪除 VirtualNetworkGatewayConnection |
> | 動作 | Microsoft.Network/connections/providers/Microsoft.Insights/diagnosticSettings/read | 取得連線的診斷設定 |
> | 動作 | Microsoft.Network/connections/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新連線的診斷設定 |
> | 動作 | Microsoft.Network/connections/providers/Microsoft.Insights/metricDefinitions/read | 取得連線的計量定義 |
> | 動作 | Microsoft.Network/connections/read | 取得 VirtualNetworkGatewayConnection |
> | 動作 | Microsoft.Network/connections/sharedkey/action | 取得 VirtualNetworkGatewayConnection SharedKey |
> | 動作 | Microsoft.Network/connections/sharedKey/read | 取得 VirtualNetworkGatewayConnection SharedKey |
> | 動作 | Microsoft.Network/connections/sharedKey/write | 建立或更新現有的 VirtualNetworkGatewayConnection SharedKey |
> | 動作 | Microsoft.Network/connections/vpndeviceconfigurationscript/action | 取得 VirtualNetworkGatewayConnection 的 VPN 裝置設定 |
> | 動作 | Microsoft.Network/connections/write | 建立或更新現有的 VirtualNetworkGatewayConnection |
> | 動作 | Microsoft.Network/ddosProtectionPlans/ddosProtectionPlanProxies/delete | 刪除 DDoS 保護計劃 Proxy |
> | 動作 | Microsoft.Network/ddosProtectionPlans/ddosProtectionPlanProxies/read | 取得 DDoS 保護計劃 Proxy 定義 |
> | 動作 | Microsoft.Network/ddosProtectionPlans/ddosProtectionPlanProxies/write | 建立 DDoS 保護計劃 Proxy，或更新現有的 DDoS 保護計劃 Proxy |
> | 動作 | Microsoft.Network/ddosProtectionPlans/delete | 刪除 DDoS 保護計劃 |
> | 動作 | Microsoft.Network/ddosProtectionPlans/join/action | 加入 DDoS 保護計劃 |
> | 動作 | Microsoft.Network/ddosProtectionPlans/read | 取得 DDoS 保護計劃 |
> | 動作 | Microsoft.Network/ddosProtectionPlans/write | 建立 DDoS 保護計劃，或更新 DDoS 保護計劃  |
> | 動作 | Microsoft.Network/dnsoperationresults/read | 取得 DNS 作業的結果 |
> | 動作 | Microsoft.Network/dnsoperationstatuses/read | 取得 DNS 作業的狀態  |
> | 動作 | Microsoft.Network/dnszones/A/delete | 從 DNS 區域中移除名稱已給定且類型為 'A' 的記錄集。 |
> | 動作 | Microsoft.Network/dnszones/A/read | 取得 JSON 格式的 'A' 類型記錄集。 記錄集包含記錄清單以及 TTL、標記和 etag。 |
> | 動作 | Microsoft.Network/dnszones/A/write | 在 DNS 區域內建立或更新 'A' 類型的記錄集。 指定的記錄將會取代記錄集內的目前記錄。 |
> | 動作 | Microsoft.Network/dnszones/AAAA/delete | 從 DNS 區域中移除名稱已給定且類型為 'AAAA' 的記錄集。 |
> | 動作 | Microsoft.Network/dnszones/AAAA/read | 取得 JSON 格式的 'AAAA' 類型記錄集。 記錄集包含記錄清單以及 TTL、標記和 etag。 |
> | 動作 | Microsoft.Network/dnszones/AAAA/write | 在 DNS 區域內建立或更新 'AAAA' 類型的記錄集。 指定的記錄將會取代記錄集內的目前記錄。 |
> | 動作 | Microsoft.Network/dnszones/all/read | 取得跨類型的 DNS 記錄集 |
> | 動作 | Microsoft.Network/dnszones/CAA/delete | 從 DNS 區域中移除指定名稱且類型為 'CAA' 的記錄集。 |
> | 動作 | Microsoft.Network/dnszones/CAA/read | 取得 JSON 格式的 'CAA' 類型記錄集。 記錄集包含 TTL、標記和 etag。 |
> | 動作 | Microsoft.Network/dnszones/CAA/write | 在 DNS 區域內建立或更新 'CAA' 類型的記錄集。 指定的記錄將會取代記錄集內的目前記錄。 |
> | 動作 | Microsoft.Network/dnszones/CNAME/delete | 從 DNS 區域中移除名稱已給定且類型為 'CNAME' 的記錄集。 |
> | 動作 | Microsoft.Network/dnszones/CNAME/read | 取得 JSON 格式的 'CNAME' 類型記錄集。 記錄集包含 TTL、標記和 etag。 |
> | 動作 | Microsoft.Network/dnszones/CNAME/write | 在 DNS 區域內建立或更新 'CNAME' 類型的記錄集。 指定的記錄將會取代記錄集內的目前記錄。 |
> | 動作 | Microsoft.Network/dnszones/delete | 刪除 JSON 格式的 DNS 區域。 區域屬性包含標記、etag、numberOfRecordSets 和 maxNumberOfRecordSets。 |
> | 動作 | Microsoft.Network/dnszones/MX/delete | 從 DNS 區域中移除名稱已給定且類型為 'MX' 的記錄集。 |
> | 動作 | Microsoft.Network/dnszones/MX/read | 取得 JSON 格式的 'MX' 類型記錄集。 記錄集包含記錄清單以及 TTL、標記和 etag。 |
> | 動作 | Microsoft.Network/dnszones/MX/write | 在 DNS 區域內建立或更新 'MX' 類型的記錄集。 指定的記錄將會取代記錄集內的目前記錄。 |
> | 動作 | Microsoft.Network/dnszones/NS/delete | 刪除 NS 類型的 DNS 記錄集 |
> | 動作 | Microsoft.Network/dnszones/NS/read | 取得 NS 類型的 DNS 記錄集 |
> | 動作 | Microsoft.Network/dnszones/NS/write | 建立或更新 NS 類型的 DNS 記錄集 |
> | 動作 | Microsoft.Network/dnszones/providers/Microsoft.Insights/diagnosticSettings/read | 取得 DNS 區域診斷設定 |
> | 動作 | Microsoft.Network/dnszones/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新 DNS 區域診斷設定 |
> | 動作 | Microsoft.Network/dnszones/providers/Microsoft.Insights/metricDefinitions/read | 取得 DNS 區域計量定義 |
> | 動作 | Microsoft.Network/dnszones/PTR/delete | 從 DNS 區域中移除名稱已給定且類型為 'PTR' 的記錄集。 |
> | 動作 | Microsoft.Network/dnszones/PTR/read | 取得 JSON 格式的 'PTR' 類型記錄集。 記錄集包含記錄清單以及 TTL、標記和 etag。 |
> | 動作 | Microsoft.Network/dnszones/PTR/write | 在 DNS 區域內建立或更新 'PTR' 類型的記錄集。 指定的記錄將會取代記錄集內的目前記錄。 |
> | 動作 | Microsoft.Network/dnszones/read | 取得 JSON 格式的 DNS 區域。 區域屬性包含標記、etag、numberOfRecordSets 和 maxNumberOfRecordSets。 請注意，此命令不會擷取區域內所包含的記錄集。 |
> | 動作 | Microsoft.Network/dnszones/recordsets/read | 取得跨類型的 DNS 記錄集 |
> | 動作 | Microsoft.Network/dnszones/SOA/read | 取得 SOA 類型的 DNS 記錄集 |
> | 動作 | Microsoft.Network/dnszones/SOA/write | 建立或更新 SOA 類型的 DNS 記錄集 |
> | 動作 | Microsoft.Network/dnszones/SRV/delete | 從 DNS 區域中移除名稱已給定且類型為 'SRV' 的記錄集。 |
> | 動作 | Microsoft.Network/dnszones/SRV/read | 取得 JSON 格式的 'SRV' 類型記錄集。 記錄集包含記錄清單以及 TTL、標記和 etag。 |
> | 動作 | Microsoft.Network/dnszones/SRV/write | 建立或更新 SRV 類型的記錄集 |
> | 動作 | Microsoft.Network/dnszones/TXT/delete | 從 DNS 區域中移除名稱已給定且類型為 'TXT' 的記錄集。 |
> | 動作 | Microsoft.Network/dnszones/TXT/read | 取得 JSON 格式的 'TXT' 類型記錄集。 記錄集包含記錄清單以及 TTL、標記和 etag。 |
> | 動作 | Microsoft.Network/dnszones/TXT/write | 在 DNS 區域內建立或更新 'TXT' 類型的記錄集。 指定的記錄將會取代記錄集內的目前記錄。 |
> | 動作 | Microsoft.Network/dnszones/write | 在資源群組內建立或更新 DNS 區域。  用來更新 DNS 區域資源上的標記。 請注意，此命令無法用來在區域內建立或更新記錄集。 |
> | 動作 | Microsoft.Network/expressRouteCircuits/authorizations/delete | 刪除 ExpressRouteCircuit 授權 |
> | 動作 | Microsoft.Network/expressRouteCircuits/authorizations/read | 取得 ExpressRouteCircuit 授權 |
> | 動作 | Microsoft.Network/expressRouteCircuits/authorizations/write | 建立或更新現有的 ExpressRouteCircuit 授權 |
> | 動作 | Microsoft.Network/expressRouteCircuits/delete | 刪除 ExpressRouteCircuit |
> | 動作 | Microsoft.Network/expressRouteCircuits/join/action | 加入 ExpressRoute 線路 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/arpTables/action | 取得 ExpressRouteCircuit 對等互連 ArpTable |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/connections/delete | 刪除 ExpressRouteCircuit 連線 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/connections/read | 取得 ExpressRouteCircuit 連線 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/connections/write | 建立或更新現有的 ExpressRouteCircuit 連線資源 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/delete | 刪除 ExpressRouteCircuit 對等互連 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/providers/Microsoft.Insights/diagnosticSettings/read | 取得 ExpressRoute 線路對等的診斷設定 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新 ExpressRoute 線路對等的診斷設定 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/providers/Microsoft.Insights/metricDefinitions/read | 取得 ExpressRoute 線路對等的計量定義 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/read | 取得 ExpressRouteCircuit 對等互連 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/routeTables/action | 取得 ExpressRouteCircuit 對等互連 RouteTable |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/routeTablesSummary/action | 取得 ExpressRouteCircuit 對等互連 RouteTable 摘要 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/stats/read | 取得 ExpressRouteCircuit 對等互連統計資料 |
> | 動作 | Microsoft.Network/expressRouteCircuits/peerings/write | 建立或更新現有的 ExpressRouteCircuit 對等互連 |
> | 動作 | Microsoft.Network/expressRouteCircuits/providers/Microsoft.Insights/diagnosticSettings/read | 取得 ExpressRoute 線路的診斷設定 |
> | 動作 | Microsoft.Network/expressRouteCircuits/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新 ExpressRoute 線路的診斷設定 |
> | 動作 | Microsoft.Network/expressRouteCircuits/providers/Microsoft.Insights/logDefinitions/read | 取得 ExpressRoute 線路的事件 |
> | 動作 | Microsoft.Network/expressRouteCircuits/providers/Microsoft.Insights/metricDefinitions/read | 取得 ExpressRoute 線路的計量定義 |
> | 動作 | Microsoft.Network/expressRouteCircuits/read | 取得 ExpressRouteCircuit |
> | 動作 | Microsoft.Network/expressRouteCircuits/stats/read | 取得 ExpressRouteCircuit 統計資料 |
> | 動作 | Microsoft.Network/expressRouteCircuits/write | 建立或更新現有的 ExpressRouteCircuit |
> | 動作 | Microsoft.Network/expressRouteCrossConnections/delete | 刪除快速路由交叉連線 |
> | 動作 | Microsoft.Network/expressRouteCrossConnections/join/action | 加入快速路由交叉連線 |
> | 動作 | Microsoft.Network/expressRouteCrossConnections/peerings/arpTables/action | 取得快速路由交叉連線對等互連 ARP 資料表 |
> | 動作 | Microsoft.Network/expressRouteCrossConnections/peerings/delete | 刪除快速路由交叉連線對等互連 |
> | 動作 | Microsoft.Network/expressRouteCrossConnections/peerings/read | 取得快速路由交叉連線對等互連 |
> | 動作 | Microsoft.Network/expressRouteCrossConnections/peerings/routeTables/action | 取得快速路由交叉連線對等互連路由表 |
> | 動作 | Microsoft.Network/expressRouteCrossConnections/peerings/routeTableSummary/action | 取得快速路由交叉連線對等互連路由表摘要 |
> | 動作 | Microsoft.Network/expressRouteCrossConnections/peerings/write | 建立快速路由交叉連線對等互連，或更新現有的快速路由交叉連線對等互連 |
> | 動作 | Microsoft.Network/expressRouteCrossConnections/read | 取得快速路由交叉連線 |
> | 動作 | Microsoft.Network/expressRouteCrossConnections/write | 建立或更新快速路由交叉連線 |
> | 動作 | Microsoft.Network/expressRouteGateways/delete | 刪除 ExpressRoute 閘道 |
> | 動作 | Microsoft.Network/expressRouteGateways/expressRouteConnections/delete | 刪除 ExpressRoute 連線 |
> | 動作 | Microsoft.Network/expressRouteGateways/expressRouteConnections/read | 取得 ExpressRoute 連線 |
> | 動作 | Microsoft.Network/expressRouteGateways/expressRouteConnections/write | 建立 ExpressRoute 連線，或更新現有的 ExpressRoute 連線 |
> | 動作 | Microsoft.Network/expressRouteGateways/join/action | 加入 ExpressRoute 閘道 |
> | 動作 | Microsoft.Network/expressRouteGateways/read | 取得 ExpressRoute 閘道 |
> | 動作 | Microsoft.Network/expressRouteGateways/write | 建立或更新 ExpressRoute 閘道 |
> | 動作 | Microsoft.Network/expressRoutePorts/delete | 刪除 ExpressRoutePorts |
> | 動作 | Microsoft.Network/expressRoutePorts/join/action | 加入 ExpressRoutePorts |
> | 動作 | Microsoft.Network/expressRoutePorts/links/read | 取得 ExpressRouteLink |
> | 動作 | Microsoft.Network/expressRoutePorts/providers/Microsoft.Insights/metricDefinitions/read | 取得 ExpressRoute 連接埠的計量定義 |
> | 動作 | Microsoft.Network/expressRoutePorts/read | 取得 ExpressRoutePorts |
> | 動作 | Microsoft.Network/expressRoutePorts/write | 建立或更新 ExpressRoutePorts |
> | 動作 | Microsoft.Network/expressRoutePortsLocations/read | 取得 Express Route 連接埠位置 |
> | 動作 | Microsoft.Network/expressRouteServiceProviders/read | 取得 ExpressRoute 服務提供者 |
> | 動作 | Microsoft.Network/frontdoors/providers/Microsoft.Insights/diagnosticSettings/read | 取得 Frontdoor 資源的診斷設定 |
> | 動作 | Microsoft.Network/frontdoors/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新 Frontdoor 資源的診斷設定 |
> | 動作 | Microsoft.Network/frontdoors/providers/Microsoft.Insights/logDefinitions/read | 取得 Frontdoor 資源的可用記錄檔 |
> | 動作 | Microsoft.Network/frontdoors/providers/Microsoft.Insights/metricDefinitions/read | 取得 Frontdoor 資源的可用計量 |
> | 動作 | Microsoft.Network/frontdoors/read | 取得 Frontdoor |
> | 動作 | Microsoft.Network/getDnsResourceReference/action | DNS 別名資源相依性要求 |
> | 動作 | Microsoft.Network/interfaceEndpoints/delete | 刪除介面端點資源。 |
> | 動作 | Microsoft.Network/interfaceEndpoints/read | 取得介面端點資源。 |
> | 動作 | Microsoft.Network/interfaceEndpoints/write | 建立新的介面端點或更新現有的介面端點。 |
> | 動作 | Microsoft.Network/internalNotify/action | DNS 別名資源通知 |
> | 動作 | Microsoft.Network/loadBalancers/backendAddressPools/join/action | 加入負載平衡器的後端位址集區 |
> | 動作 | Microsoft.Network/loadBalancers/backendAddressPools/read | 取得負載平衡器的後端位址集區定義 |
> | 動作 | Microsoft.Network/loadBalancers/delete | 刪除負載平衡器 |
> | 動作 | Microsoft.Network/loadBalancers/inboundNatPools/join/action | 加入負載平衡器的輸入 NAT 集區 |
> | 動作 | Microsoft.Network/loadBalancers/inboundNatPools/read | 取得負載平衡器的輸入 NAT 集區定義 |
> | 動作 | Microsoft.Network/loadBalancers/inboundNatRules/delete | 刪除負載平衡器的輸入 NAT 規則 |
> | 動作 | Microsoft.Network/loadBalancers/inboundNatRules/join/action | 加入負載平衡器的輸入 NAT 規則 |
> | 動作 | Microsoft.Network/loadBalancers/inboundNatRules/read | 取得負載平衡器的輸入 NAT 規則定義 |
> | 動作 | Microsoft.Network/loadBalancers/inboundNatRules/write | 建立負載平衡器的輸入 NAT 規則，或更新現有的負載平衡器輸入 NAT 規則 |
> | 動作 | Microsoft.Network/loadBalancers/loadBalancingRules/read | 取得負載平衡器的負載平衡規則定義 |
> | 動作 | Microsoft.Network/loadBalancers/networkInterfaces/read | 取得負載平衡器下所有網路介面的參考 |
> | 動作 | Microsoft.Network/loadBalancers/outboundRules/read | 取得負載平衡器的輸出規則定義 |
> | 動作 | Microsoft.Network/loadBalancers/probes/join/action | 允許使用負載平衡器的探查。 例如，使用此權限，VM 擴展集的 healthProbe 屬性就可以參考探查。 |
> | 動作 | Microsoft.Network/loadBalancers/probes/read | 取得負載平衡器探查 |
> | 動作 | Microsoft.Network/loadBalancers/providers/Microsoft.Insights/diagnosticSettings/read | 取得負載平衡器診斷設定 |
> | 動作 | Microsoft.Network/loadBalancers/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新負載平衡器診斷設定 |
> | 動作 | Microsoft.Network/loadBalancers/providers/Microsoft.Insights/logDefinitions/read | 取得負載平衡器的事件 |
> | 動作 | Microsoft.Network/loadBalancers/providers/Microsoft.Insights/metricDefinitions/read | 取得負載平衡器的可用計量 |
> | 動作 | Microsoft.Network/loadBalancers/read | 取得負載平衡器定義 |
> | 動作 | Microsoft.Network/loadBalancers/virtualMachines/read | 取得負載平衡器下所有虛擬機器的參考 |
> | 動作 | Microsoft.Network/loadBalancers/write | 建立負載平衡器，或更新現有的負載平衡器 |
> | 動作 | Microsoft.Network/localnetworkgateways/delete | 刪除 LocalNetworkGateway |
> | 動作 | Microsoft.Network/localnetworkgateways/read | 取得 LocalNetworkGateway |
> | 動作 | Microsoft.Network/localnetworkgateways/write | 建立或更新現有的 LocalNetworkGateway |
> | 動作 | Microsoft.Network/locations/availableDelegations/read | 取得可用的委派 |
> | 動作 | Microsoft.Network/locations/bareMetalTenants/action | 配置或驗證裸機租用戶 |
> | 動作 | Microsoft.Network/locations/checkAcceleratedNetworkingSupport/action | 檢查加速網路支援 |
> | 動作 | Microsoft.Network/locations/checkDnsNameAvailability/read | 檢查指定的位置是否有 DNS 標籤 |
> | 動作 | Microsoft.Network/locations/effectiveResourceOwnership/action | 取得有效的資源擁有權 |
> | 動作 | Microsoft.Network/locations/operationResults/read | 取得非同步 POST 或 DELETE 作業的作業結果 |
> | 動作 | Microsoft.Network/locations/operations/read | 取得代表非同步作業狀態的作業資源 |
> | 動作 | Microsoft.Network/locations/setResourceOwnership/action | 設定資源擁有權 |
> | 動作 | Microsoft.Network/locations/supportedVirtualMachineSizes/read | 取得支援的虛擬機器大小 |
> | 動作 | Microsoft.Network/locations/usages/read | 取得資源使用量計量 |
> | 動作 | Microsoft.Network/locations/validateResourceOwnership/action | 驗證資源擁有權 |
> | 動作 | Microsoft.Network/locations/virtualNetworkAvailableEndpointServices/read | 取得可用虛擬網路端點服務的清單 |
> | 動作 | Microsoft.Network/networkIntentPolicies/delete | 刪除網路意圖原則 |
> | 動作 | Microsoft.Network/networkIntentPolicies/read | 取得網路意圖原則描述 |
> | 動作 | Microsoft.Network/networkIntentPolicies/write | 建立網路意圖原則或更新現有的網路意圖原則 |
> | 動作 | Microsoft.Network/networkInterfaces/delete | 刪除網路介面 |
> | 動作 | Microsoft.Network/networkInterfaces/diagnosticIdentity/read | 取得資源的診斷身分識別 |
> | 動作 | Microsoft.Network/networkInterfaces/effectiveNetworkSecurityGroups/action | 取得 VM 網路介面上所設定的網路安全性群組 |
> | 動作 | Microsoft.Network/networkInterfaces/effectiveRouteTable/action | 取得 VM 網路介面上所設定的路由表 |
> | 動作 | Microsoft.Network/networkInterfaces/ipconfigurations/join/action | 加入網路介面 IP 設定。 |
> | 動作 | Microsoft.Network/networkInterfaces/ipconfigurations/read | 取得網路介面 IP 組態定義。  |
> | 動作 | Microsoft.Network/networkInterfaces/join/action | 將虛擬機器加入網路介面 |
> | 動作 | Microsoft.Network/networkInterfaces/joinViaPrivateIp/action | 透過服務關聯將資源加入網路介面 |
> | 動作 | Microsoft.Network/networkInterfaces/loadBalancers/read | 取得網路介面所屬的所有負載平衡器 |
> | 動作 | Microsoft.Network/networkInterfaces/providers/Microsoft.Insights/metricDefinitions/read | 取得網路介面的可用計量 |
> | 動作 | Microsoft.Network/networkInterfaces/read | 取得網路介面定義。  |
> | 動作 | Microsoft.Network/networkInterfaces/serviceAssociations/delete | 刪除服務關聯 |
> | 動作 | Microsoft.Network/networkInterfaces/serviceAssociations/read | 取得服務關聯定義 |
> | 動作 | Microsoft.Network/networkInterfaces/serviceAssociations/validate/action | 驗證服務關聯 |
> | 動作 | Microsoft.Network/networkInterfaces/serviceAssociations/write | 建立新的服務關聯或修改現有的服務關聯 |
> | 動作 | Microsoft.Network/networkInterfaces/tapConfigurations/delete | 刪除網路介面 Tap 設定。 |
> | 動作 | Microsoft.Network/networkInterfaces/tapConfigurations/read | 取得網路介面 Tap 設定。 |
> | 動作 | Microsoft.Network/networkInterfaces/tapConfigurations/write | 建立網路介面 Tap 設定，或更新現有的網路介面 Tap 設定。 |
> | 動作 | Microsoft.Network/networkInterfaces/write | 建立網路介面，或更新現有的網路介面。  |
> | 動作 | Microsoft.Network/networkProfiles/delete | 刪除網路設定檔 |
> | 動作 | Microsoft.Network/networkProfiles/read | 取得網路設定檔 |
> | 動作 | Microsoft.Network/networkProfiles/removeContainers/action | 移除容器 |
> | 動作 | Microsoft.Network/networkProfiles/setContainers/action | 設定容器 |
> | 動作 | Microsoft.Network/networkProfiles/setNetworkInterfaces/action | 設定容器網路介面 |
> | 動作 | Microsoft.Network/networkProfiles/write | 建立或更新網路設定檔 |
> | 動作 | Microsoft.Network/networkSecurityGroups/defaultSecurityRules/read | 取得預設的安全性規則定義 |
> | 動作 | Microsoft.Network/networkSecurityGroups/delete | 刪除網路安全性群組 |
> | 動作 | Microsoft.Network/networkSecurityGroups/join/action | 加入網路安全性群組 |
> | 動作 | Microsoft.Network/networksecuritygroups/providers/Microsoft.Insights/diagnosticSettings/read | 取得網路安全性群組診斷設定 |
> | 動作 | Microsoft.Network/networksecuritygroups/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新網路安全性群組診斷設定，此作業會由 Insights 資源提供者來補充。 |
> | 動作 | Microsoft.Network/networksecuritygroups/providers/Microsoft.Insights/logDefinitions/read | 取得網路安全性群組的事件 |
> | 動作 | Microsoft.Network/networkSecurityGroups/read | 取得網路安全性群組定義 |
> | 動作 | Microsoft.Network/networkSecurityGroups/securityRules/delete | 刪除安全性規則 |
> | 動作 | Microsoft.Network/networkSecurityGroups/securityRules/read | 取得安全性規則定義 |
> | 動作 | Microsoft.Network/networkSecurityGroups/securityRules/write | 建立安全性規則，或更新現有的安全性規則 |
> | 動作 | Microsoft.Network/networkSecurityGroups/write | 建立網路安全性群組，或更新現有的網路安全性群組 |
> | 動作 | Microsoft.Network/networkWatchers/availableProvidersList/action | 針對指定的 Azure 區域傳回所有可用的網際網路服務提供者。 |
> | 動作 | Microsoft.Network/networkWatchers/azureReachabilityReport/action | 傳回從指定位置到 Azure 區域之網際網路服務提供者的相對延遲分數。 |
> | 動作 | Microsoft.Network/networkWatchers/configureFlowLog/action | 設定目標資源的流程記錄。 |
> | 動作 | Microsoft.Network/networkWatchers/connectionMonitors/delete | 刪除連線監視 |
> | 動作 | Microsoft.Network/networkWatchers/connectionMonitors/providers/Microsoft.Insights/diagnosticSettings/read | 取得連線監視的診斷設定 |
> | 動作 | Microsoft.Network/networkWatchers/connectionMonitors/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新連線監視診斷設定 |
> | 動作 | Microsoft.Network/networkWatchers/connectionMonitors/providers/Microsoft.Insights/metricDefinitions/read | 取得連線監視的可用計量 |
> | 動作 | Microsoft.Network/networkWatchers/connectionMonitors/query/action | 查詢監視指定端點之間的連線能力 |
> | 動作 | Microsoft.Network/networkWatchers/connectionMonitors/read | 取得連線監視詳細資料 |
> | 動作 | Microsoft.Network/networkWatchers/connectionMonitors/start/action | 開始監視指定端點之間的連線能力 |
> | 動作 | Microsoft.Network/networkWatchers/connectionMonitors/stop/action | 停止/暫停監視指定端點之間的連線能力 |
> | 動作 | Microsoft.Network/networkWatchers/connectionMonitors/write | 建立連線監視 |
> | 動作 | Microsoft.Network/networkWatchers/connectivityCheck/action | 確認是否可以建立從虛擬機器到包含另一台 VM 或任意遠端伺服器之指定端點的直接 TCP 連線。 |
> | 動作 | Microsoft.Network/networkWatchers/delete | 刪除網路監看員 |
> | 動作 | Microsoft.Network/networkWatchers/ipFlowVerify/action | 傳回是要允許還是拒絕特定目的地的往返封包。 |
> | 動作 | Microsoft.Network/networkWatchers/lenses/delete | 刪除功能濾鏡 |
> | 動作 | Microsoft.Network/networkWatchers/lenses/query/action | 查詢監視指定端點上的網路流量 |
> | 動作 | Microsoft.Network/networkWatchers/lenses/read | 取得功能濾鏡詳細資料 |
> | 動作 | Microsoft.Network/networkWatchers/lenses/start/action | 開始監視指定端點上的網路流量 |
> | 動作 | Microsoft.Network/networkWatchers/lenses/stop/action | 停止/暫停監視指定端點上的網路流量 |
> | 動作 | Microsoft.Network/networkWatchers/lenses/write | 建立功能濾鏡 |
> | 動作 | Microsoft.Network/networkWatchers/networkConfigurationDiagnostic/action | 網路組態的診斷。 |
> | 動作 | Microsoft.Network/networkWatchers/nextHop/action | 針對指定的目標和目的地 IP 位址，傳回下一個躍點的類型和 IP 位址。 |
> | 動作 | Microsoft.Network/networkWatchers/packetCaptures/delete | 刪除封包擷取 |
> | 動作 | Microsoft.Network/networkWatchers/packetCaptures/queryStatus/action | 取得屬性的相關資訊和封包擷取資源的狀態。 |
> | 動作 | Microsoft.Network/networkWatchers/packetCaptures/read | 取得封包擷取定義 |
> | 動作 | Microsoft.Network/networkWatchers/packetCaptures/stop/action | 停止正在執行的封包擷取工作階段。 |
> | 動作 | Microsoft.Network/networkWatchers/packetCaptures/write | 建立封包擷取 |
> | 動作 | Microsoft.Network/networkWatchers/pingMeshes/delete | 刪除 PingMesh |
> | 動作 | Microsoft.Network/networkWatchers/pingMeshes/read | 取得 PingMesh 詳細資料 |
> | 動作 | Microsoft.Network/networkWatchers/pingMeshes/start/action | 在指定的 VM 之間啟動 PingMesh |
> | 動作 | Microsoft.Network/networkWatchers/pingMeshes/stop/action | 在指定的 VM 之間停止 PingMesh |
> | 動作 | Microsoft.Network/networkWatchers/pingMeshes/write | 建立 PingMesh |
> | 動作 | Microsoft.Network/networkWatchers/queryConnectionMonitors/action | 批次查詢監視指定端點之間的連線能力 |
> | 動作 | Microsoft.Network/networkWatchers/queryFlowLogStatus/action | 取得資源的流程記錄狀態。 |
> | 動作 | Microsoft.Network/networkWatchers/queryTroubleshootResult/action | 取得上一次執行的疑難排解結果或目前執行的疑難排解作業。 |
> | 動作 | Microsoft.Network/networkWatchers/read | 取得網路監看員定義 |
> | 動作 | Microsoft.Network/networkWatchers/securityGroupView/action | 檢視對 VM 套用之已設定且生效的網路安全性群組規則。 |
> | 動作 | Microsoft.Network/networkWatchers/topology/action | 取得資源的網路層級檢視和資源在資源群組中的關聯性。 |
> | 動作 | Microsoft.Network/networkWatchers/troubleshoot/action | 開始針對 Azure 中的網路資源進行疑難排解。 |
> | 動作 | Microsoft.Network/networkWatchers/write | 建立網路監看員，或更新現有的網路監看員 |
> | 動作 | Microsoft.Network/operations/read | 取得可用的作業 |
> | 動作 | Microsoft.Network/p2sVpnGateways/delete | 刪除 P2SVpnGateway。 |
> | 動作 | Microsoft.Network/p2sVpnGateways/generatevpnprofile/action | 產生 P2SVpnGateway 的 Vpn 設定檔 |
> | 動作 | Microsoft.Network/p2sVpnGateways/read | 取得 P2SVpnGateway。 |
> | 動作 | Microsoft.Network/p2sVpnGateways/write | 放置 P2SVpnGateway。 |
> | 動作 | Microsoft.Network/privateLinkServices/delete | 刪除私人連結服務資源。 |
> | 動作 | Microsoft.Network/privateLinkServices/interfaceEndpointConnections/delete | 刪除介面端點連線。 |
> | 動作 | Microsoft.Network/privateLinkServices/interfaceEndpointConnections/read | 取得介面端點連線定義。 |
> | 動作 | Microsoft.Network/privateLinkServices/interfaceEndpointConnections/write | 建立新的介面端點連線或更新現有的介面端點連線。 |
> | 動作 | Microsoft.Network/privateLinkServices/read | 取得私人連結服務資源。 |
> | 動作 | Microsoft.Network/privateLinkServices/write | 建立新的私人連結服務或更新現有的私人連結服務。 |
> | 動作 | Microsoft.Network/publicIPAddresses/delete | 刪除公用 IP 位址。 |
> | 動作 | Microsoft.Network/publicIPAddresses/dnsAliases/delete | 刪除公用 IP 位址 DNS 別名資源 |
> | 動作 | Microsoft.Network/publicIPAddresses/dnsAliases/read | 取得公用 IP 位址 DNS 別名資源 |
> | 動作 | Microsoft.Network/publicIPAddresses/dnsAliases/write | 建立公用 IP 位址 DNS 別名資源 |
> | 動作 | Microsoft.Network/publicIPAddresses/join/action | 加入公用 IP 位址 |
> | 動作 | Microsoft.Network/publicIPAddresses/providers/Microsoft.Insights/diagnosticSettings/read | 取得公用 IP 位址的診斷設定 |
> | 動作 | Microsoft.Network/publicIPAddresses/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新公用 IP 位址的診斷設定 |
> | 動作 | Microsoft.Network/publicIPAddresses/providers/Microsoft.Insights/logDefinitions/read | 取得公用 IP 位址的記錄定義 |
> | 動作 | Microsoft.Network/publicIPAddresses/providers/Microsoft.Insights/metricDefinitions/read | 取得公用 IP 位址的計量定義 |
> | 動作 | Microsoft.Network/publicIPAddresses/read | 取得公用 IP 位址定義。 |
> | 動作 | Microsoft.Network/publicIPAddresses/write | 建立公用 IP 位址，或更新現有的公用 IP 位址。  |
> | 動作 | Microsoft.Network/publicIPPrefixes/delete | 刪除公用 IP 首碼 |
> | 動作 | Microsoft.Network/publicIPPrefixes/join/action | 聯結 PublicIPPrefix |
> | 動作 | Microsoft.Network/publicIPPrefixes/read | 取得公用 IP 首碼定義 |
> | 動作 | Microsoft.Network/publicIPPrefixes/write | 建立公用 IP 首碼，或更新現有的公用 IP 首碼 |
> | 動作 | Microsoft.Network/register/action | 註冊訂用帳戶 |
> | 動作 | Microsoft.Network/routeFilters/delete | 刪除路由篩選的定義 |
> | 動作 | Microsoft.Network/routeFilters/join/action | 加入路由篩選 |
> | 動作 | Microsoft.Network/routeFilters/read | 取得路由篩選的定義 |
> | 動作 | Microsoft.Network/routeFilters/routeFilterRules/delete | 刪除路由篩選規則的定義 |
> | 動作 | Microsoft.Network/routeFilters/routeFilterRules/read | 取得路由篩選規則的定義 |
> | 動作 | Microsoft.Network/routeFilters/routeFilterRules/write | 建立路由篩選規則，或更新現有的路由篩選規則 |
> | 動作 | Microsoft.Network/routeFilters/write | 建立路由篩選，或更新現有的路由篩選 |
> | 動作 | Microsoft.Network/routeTables/delete | 刪除路由表定義 |
> | 動作 | Microsoft.Network/routeTables/join/action | 加入路由表 |
> | 動作 | Microsoft.Network/routeTables/read | 取得路由表定義 |
> | 動作 | Microsoft.Network/routeTables/routes/delete | 刪除路由定義 |
> | 動作 | Microsoft.Network/routeTables/routes/read | 取得路由定義 |
> | 動作 | Microsoft.Network/routeTables/routes/write | 建立路由，或更新現有路由 |
> | 動作 | Microsoft.Network/routeTables/write | 建立路由表，或更新現有的路由表 |
> | 動作 | Microsoft.Network/securegateways/applicationRuleCollections/delete | 刪除安全閘道的應用程式規則集合 |
> | 動作 | Microsoft.Network/securegateways/applicationRuleCollections/read | 擷取指定安全閘道的應用程式規則集合 |
> | 動作 | Microsoft.Network/securegateways/applicationRuleCollections/write | 建立或更新安全閘道的應用程式規則集合 |
> | 動作 | Microsoft.Network/securegateways/delete | 刪除安全閘道 |
> | 動作 | Microsoft.Network/securegateways/networkRuleCollections/delete | 刪除安全閘道的網路規則集合 |
> | 動作 | Microsoft.Network/securegateways/networkRuleCollections/read | 擷取指定安全閘道的網路規則集合 |
> | 動作 | Microsoft.Network/securegateways/networkRuleCollections/write | 建立或更新安全閘道的網路規則集合 |
> | 動作 | Microsoft.Network/securegateways/providers/Microsoft.Insights/logDefinitions/read | 取得「Azure 防火牆」的事件 |
> | 動作 | Microsoft.Network/securegateways/read | 取得安全閘道 |
> | 動作 | Microsoft.Network/securegateways/write | 建立或更新安全閘道 |
> | 動作 | Microsoft.Network/serviceEndpointPolicies/delete | 刪除服務端點原則 |
> | 動作 | Microsoft.Network/serviceEndpointPolicies/join/action | 加入服務端點原則 |
> | 動作 | Microsoft.Network/serviceEndpointPolicies/joinSubnet/action | 將子網路加入至服務端點原則 |
> | 動作 | Microsoft.Network/serviceEndpointPolicies/read | 取得服務端點原則描述 |
> | 動作 | Microsoft.Network/serviceEndpointPolicies/serviceEndpointPolicyDefinitions/delete | 刪除服務端點原則定義 |
> | 動作 | Microsoft.Network/serviceEndpointPolicies/serviceEndpointPolicyDefinitions/read | 取得服務端點原則定義描述 |
> | 動作 | Microsoft.Network/serviceEndpointPolicies/serviceEndpointPolicyDefinitions/write | 建立服務端點原則定義，或更新現有的服務端點原則定義 |
> | 動作 | Microsoft.Network/serviceEndpointPolicies/write | 建立服務端點原則，或更新現有的服務端點原則 |
> | 動作 | Microsoft.Network/trafficManagerGeographicHierarchies/read | 取得流量管理員的地理階層，其所含的區域可用在地理流量路由方法中 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/azureEndpoints/delete | 從現有的流量管理員設定檔刪除 Azure 端點。 流量管理員將會停止將流量路由傳送到已刪除的 Azure 端點。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/azureEndpoints/read | 取得屬於流量管理員設定檔的 Azure 端點，其中包含該 Azure 端點的所有屬性。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/azureEndpoints/write | 在現有流量管理員設定檔中新增 Azure 端點，或在該流量管理員設定檔中更新現有 Azure 端點的屬性。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/delete | 刪除流量管理員設定檔。 與流量管理員設定檔相關聯的所有設定都會遺失，而且您無法再使用設定檔來路由傳送流量。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/externalEndpoints/delete | 從現有的流量管理員設定檔刪除外部端點。 流量管理員將會停止將流量路由傳送到已刪除的外部端點。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/externalEndpoints/read | 取得屬於流量管理員設定檔的外部端點，其中包含該外部端點的所有屬性。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/externalEndpoints/write | 在現有流量管理員設定檔中新增外部端點，或在該流量管理員設定檔中更新現有外部端點的屬性。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/heatMaps/read | 取得指定流量管理員設定檔的流量管理員熱度圖，其中包含依位置和來源 IP 的查詢計數和延遲資料。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/nestedEndpoints/delete | 從現有的流量管理員設定檔刪除巢狀端點。 流量管理員將會停止將流量路由傳送到已刪除的巢狀端點。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/nestedEndpoints/read | 取得屬於流量管理員設定檔的巢狀端點，其中包含該巢狀端點的所有屬性。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/nestedEndpoints/write | 在現有流量管理員設定檔中新增巢狀端點，或在該流量管理員設定檔中更新現有巢狀端點的屬性。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/providers/Microsoft.Insights/diagnosticSettings/read | 取得流量管理員診斷設定 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新流量管理員診斷設定，此作業會由 Insights 資源提供者來補充。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/providers/Microsoft.Insights/logDefinitions/read | 取得流量管理員的事件 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/providers/Microsoft.Insights/metricDefinitions/read | 取得流量管理員的可用計量。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/read | 取得流量管理員設定檔組態。 這包括 DNS 設定、流量路由設定、端點監視設定，以及此流量管理員設定檔所路由之端點的清單。 |
> | 動作 | Microsoft.Network/trafficManagerProfiles/write | 建立流量管理員設定檔，或修改現有流量管理員設定檔的組態。<br>這包括啟用或停用設定檔以及修改 DNS 設定、流量路由設定或端點監視設定。<br>您可以新增、移除、啟用或停用由流量管理員設定檔所路由傳送的端點。 |
> | 動作 | Microsoft.Network/trafficManagerUserMetricsKeys/delete | 刪除針對即時使用者計量集合使用的訂用帳戶層級金鑰。 |
> | 動作 | Microsoft.Network/trafficManagerUserMetricsKeys/read | 取得針對即時使用者計量集合使用的訂用帳戶層級金鑰。 |
> | 動作 | Microsoft.Network/trafficManagerUserMetricsKeys/write | 建立要針對即時使用者計量集合使用的新訂用帳戶層級金鑰。 |
> | 動作 | Microsoft.Network/unregister/action | 取消註冊訂用帳戶 |
> | 動作 | Microsoft.Network/virtualHubs/delete | 刪除虛擬中樞 |
> | 動作 | Microsoft.Network/virtualHubs/hubVirtualNetworkConnections/delete | 刪除 HubVirtualNetworkConnection |
> | 動作 | Microsoft.Network/virtualHubs/hubVirtualNetworkConnections/read | 取得 HubVirtualNetworkConnection |
> | 動作 | Microsoft.Network/virtualHubs/hubVirtualNetworkConnections/write | 建立或更新 HubVirtualNetworkConnection |
> | 動作 | Microsoft.Network/virtualHubs/read | 取得虛擬中樞 |
> | 動作 | Microsoft.Network/virtualHubs/write | 建立或更新虛擬中樞 |
> | 動作 | microsoft.network/virtualnetworkgateways/connections/read | 取得 VirtualNetworkGatewayConnection |
> | 動作 | Microsoft.Network/virtualNetworkGateways/delete | 刪除 virtualNetworkGateway |
> | 動作 | microsoft.network/virtualnetworkgateways/generatevpnclientpackage/action | 針對 virtualNetworkGateway 產生 VpnClient 套件 |
> | 動作 | microsoft.network/virtualnetworkgateways/generatevpnprofile/action | 針對 VirtualNetworkGateway 產生 VpnProfile 套件 |
> | 動作 | microsoft.network/virtualnetworkgateways/getadvertisedroutes/action | 取得 virtualNetworkGateway 通告路由 |
> | 動作 | microsoft.network/virtualnetworkgateways/getbgppeerstatus/action | 取得 virtualNetworkGateway bgp 對等狀態 |
> | 動作 | microsoft.network/virtualnetworkgateways/getlearnedroutes/action | 取得 virtualnetworkgateway 學習到的路由 |
> | 動作 | microsoft.network/virtualnetworkgateways/getvpnclientconnectionhealth/action | 針對 VirtualNetworkGateway 取得每個 VPN 用戶端連線健康情況 |
> | 動作 | microsoft.network/virtualnetworkgateways/getvpnclientipsecparameters/action | 取得適用於 VirtualNetworkGateway P2S 用戶端的 Vpnclient Ipsec 參數。 |
> | 動作 | microsoft.network/virtualnetworkgateways/getvpnprofilepackageurl/action | 取得預先產生之 VPN 用戶端設定檔套件的 URL |
> | 動作 | Microsoft.Network/virtualNetworkGateways/providers/Microsoft.Insights/diagnosticSettings/read | 取得虛擬網路閘道診斷設定 |
> | 動作 | Microsoft.Network/virtualNetworkGateways/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新虛擬網路閘道診斷設定，此作業會由 Insights 資源提供者來補充。 |
> | 動作 | Microsoft.Network/virtualNetworkGateways/providers/Microsoft.Insights/logDefinitions/read | 取得虛擬網路閘道的事件 |
> | 動作 | Microsoft.Network/virtualNetworkGateways/providers/Microsoft.Insights/metricDefinitions/read | 取得虛擬網路閘道的可用計量 |
> | 動作 | Microsoft.Network/virtualNetworkGateways/read | 取得 VirtualNetworkGateway |
> | 動作 | microsoft.network/virtualnetworkgateways/reset/action | 重設 virtualNetworkGateway |
> | 動作 | microsoft.network/virtualnetworkgateways/resetvpnclientsharedkey/action | 重設適用於 VirtualNetworkGateway P2S 用戶端的 Vpnclient 共用金鑰。 |
> | 動作 | microsoft.network/virtualnetworkgateways/setvpnclientipsecparameters/action | 設定適用於 VirtualNetworkGateway P2S 用戶端的 Vpnclient Ipsec 參數。 |
> | 動作 | Microsoft.Network/virtualnetworkgateways/supportedvpndevices/action | 列出支援的 VPN 裝置 |
> | 動作 | Microsoft.Network/virtualNetworkGateways/write | 建立或更新 VirtualNetworkGateway |
> | 動作 | Microsoft.Network/virtualNetworks/checkIpAddressAvailability/read | 檢查指定的虛擬網路中是否有 IP 位址 |
> | 動作 | Microsoft.Network/virtualNetworks/customViews/get/action | 取得虛擬網路自訂檢視內容 |
> | 動作 | Microsoft.Network/virtualNetworks/customViews/read | 取得虛擬網路自訂檢視的定義 |
> | 動作 | Microsoft.Network/virtualNetworks/delete | 刪除虛擬網路 |
> | 動作 | Microsoft.Network/virtualNetworks/peer/action | 讓某個虛擬網路與另一個虛擬網路對等互連 |
> | 動作 | Microsoft.Network/virtualNetworks/providers/Microsoft.Insights/diagnosticSettings/read | 取得虛擬網路的診斷設定 |
> | 動作 | Microsoft.Network/virtualNetworks/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新虛擬網路的診斷設定 |
> | 動作 | Microsoft.Network/virtualNetworks/providers/Microsoft.Insights/logDefinitions/read | 取得虛擬網路的記錄定義 |
> | 動作 | Microsoft.Network/virtualNetworks/providers/Microsoft.Insights/metricDefinitions/read | 取得 PingMesh 的可用計量 |
> | 動作 | Microsoft.Network/virtualNetworks/read | 取得虛擬網路定義 |
> | 動作 | Microsoft.Network/virtualNetworks/remoteVirtualNetworkPeeringProxies/delete | 刪除虛擬網路對等互連 Proxy |
> | 動作 | Microsoft.Network/virtualNetworks/remoteVirtualNetworkPeeringProxies/read | 取得虛擬網路對等互連 Proxy 定義 |
> | 動作 | Microsoft.Network/virtualNetworks/remoteVirtualNetworkPeeringProxies/write | 建立虛擬網路對等互連 Proxy，或更新現有的虛擬網路對等互連 Proxy |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/delete | 刪除虛擬網路子網路 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/join/action | 加入虛擬網路 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action | 將資源 (例如，儲存體帳戶或 SQL Database) 加入至子網路。 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/read | 取得虛擬網路子網路定義 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/resourceNavigationLinks/delete | 刪除資源瀏覽連結 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/resourceNavigationLinks/read | 取得資源瀏覽連結定義 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/resourceNavigationLinks/write | 建立資源瀏覽連結，或更新現有的資源瀏覽連結 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/serviceAssociationLinks/delete | 刪除服務關聯連結 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/serviceAssociationLinks/details/read | 取得服務關聯連結詳細資料定義 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/serviceAssociationLinks/read | 取得服務關聯連結定義 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/serviceAssociationLinks/validate/action | 驗證服務關聯連結 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/serviceAssociationLinks/write | 建立服務關聯連結或更新現有的服務關聯連結 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/virtualMachines/read | 取得虛擬網路子網路中所有虛擬機器的參考 |
> | 動作 | Microsoft.Network/virtualNetworks/subnets/write | 建立虛擬網路子網路，或更新現有的虛擬網路子網路 |
> | 動作 | Microsoft.Network/virtualNetworks/taggedTrafficConsumers/delete | 刪除標記的流量取用者 |
> | 動作 | Microsoft.Network/virtualNetworks/taggedTrafficConsumers/read | 取得標記的流量取用者定義 |
> | 動作 | Microsoft.Network/virtualNetworks/taggedTrafficConsumers/validate/action | 驗證標記的流量取用者 |
> | 動作 | Microsoft.Network/virtualNetworks/taggedTrafficConsumers/write | 建立標記的流量取用者，或更新現有的標記流量取用者 |
> | 動作 | Microsoft.Network/virtualNetworks/usages/read | 取得虛擬網路中每個子網路的 IP 使用方式 |
> | 動作 | Microsoft.Network/virtualNetworks/virtualMachines/read | 取得虛擬網路中所有虛擬機器的參考 |
> | 動作 | Microsoft.Network/virtualNetworks/virtualNetworkPeerings/delete | 刪除虛擬網路對等互連 |
> | 動作 | Microsoft.Network/virtualNetworks/virtualNetworkPeerings/read | 取得虛擬網路對等互連定義 |
> | 動作 | Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write | 建立虛擬網路對等互連，或更新現有的虛擬網路對等互連 |
> | 動作 | Microsoft.Network/virtualNetworks/write | 建立虛擬網路，或更新現有的虛擬網路 |
> | 動作 | Microsoft.Network/virtualNetworkTaps/delete | 刪除Virtual Network Tap |
> | 動作 | Microsoft.Network/virtualNetworkTaps/join/action | 加入 Virtual Network Tap |
> | 動作 | Microsoft.Network/virtualNetworkTaps/networkInterfaceTapConfigurationProxies/delete | 刪除網路介面 Tap 設定 Proxy。 |
> | 動作 | Microsoft.Network/virtualNetworkTaps/networkInterfaceTapConfigurationProxies/read | 取得網路介面 Tap 設定 Proxy。 |
> | 動作 | Microsoft.Network/virtualNetworkTaps/networkInterfaceTapConfigurationProxies/write | 建立網路介面 Tap 設定 Proxy，或更新現有的網路介面 Tap 設定 Proxy。 |
> | 動作 | Microsoft.Network/virtualNetworkTaps/read | 取得 Virtual Network Tap |
> | 動作 | Microsoft.Network/virtualNetworkTaps/write | 建立或更新 Virtual Network Tap |
> | 動作 | Microsoft.Network/virtualWans/delete | 刪除虛擬 WAN |
> | 動作 | Microsoft.Network/virtualWans/p2sVpnGatewayProxies/delete | 刪除 P2SVpnGateway Proxy |
> | 動作 | Microsoft.Network/virtualWans/p2sVpnGatewayProxies/read | 取得 P2SVpnGateway Proxy 定義 |
> | 動作 | Microsoft.Network/virtualWans/p2sVpnGatewayProxies/write | 建立 P2SVpnGateway Proxy 或更新 P2SVpnGateway Proxy |
> | 動作 | Microsoft.Network/virtualWans/p2sVpnServerConfigurations/delete | 刪除虛擬 Wan P2SVpnServerConfiguration |
> | 動作 | Microsoft.Network/virtualWans/p2sVpnServerConfigurations/read | 取得虛擬 Wan P2SVpnServerConfiguration 定義 |
> | 動作 | Microsoft.Network/virtualWans/p2sVpnServerConfigurations/write | 建立虛擬 Wan P2SVpnServerConfiguration 或更新現有的虛擬 Wan P2SVpnServerConfiguration |
> | 動作 | Microsoft.Network/virtualWans/read | 取得虛擬 WAN |
> | 動作 | Microsoft.Network/virtualwans/supportedSecurityProviders/read | 取得支援的 VirtualWan 安全性提供者。 |
> | 動作 | Microsoft.Network/virtualWans/virtualHubProxies/delete | 刪除虛擬中樞 Proxy |
> | 動作 | Microsoft.Network/virtualWans/virtualHubProxies/read | 取得虛擬中樞 Proxy 定義 |
> | 動作 | Microsoft.Network/virtualWans/virtualHubProxies/write | 建立虛擬中樞 Proxy，或更新虛擬中樞 Proxy |
> | 動作 | Microsoft.Network/virtualWans/virtualHubs/read | 取得所有參考虛擬 WAN 的虛擬中樞。 |
> | 動作 | Microsoft.Network/virtualwans/vpnconfiguration/action | 取得 VPN 設定 |
> | 動作 | Microsoft.Network/virtualWans/vpnSiteProxies/delete | 刪除 VPN 網站 Proxy |
> | 動作 | Microsoft.Network/virtualWans/vpnSiteProxies/read | 取得 VPN 網站 Proxy 定義 |
> | 動作 | Microsoft.Network/virtualWans/vpnSiteProxies/write | 建立 VPN 網站 Proxy，或更新 VPN 網站 Proxy |
> | 動作 | Microsoft.Network/virtualWans/vpnSites/read | 取得所有參考虛擬 WAN 的 VPN 網站。 |
> | 動作 | Microsoft.Network/virtualWans/write | 建立或更新虛擬 WAN |
> | 動作 | Microsoft.Network/vpnGateways/delete | 刪除 VpnGateway。 |
> | 動作 | microsoft.network/vpngateways/listvpnconnectionshealth/action | 取得 VpnGateway 上所有或部分連線的連線健康情況 |
> | 動作 | Microsoft.Network/vpnGateways/read | 取得 VpnGateway。 |
> | 動作 | microsoft.network/vpngateways/reset/action | 重設 VpnGateway |
> | 動作 | microsoft.network/vpnGateways/vpnConnections/delete | 刪除 VpnConnection。 |
> | 動作 | microsoft.network/vpnGateways/vpnConnections/read | 取得 VpnConnection。 |
> | 動作 | microsoft.network/vpnGateways/vpnConnections/write | 放置 VpnConnection。 |
> | 動作 | Microsoft.Network/vpnGateways/write | 放置 VpnGateway。 |
> | 動作 | Microsoft.Network/vpnsites/delete | 刪除 VPN 網站資源。 |
> | 動作 | Microsoft.Network/vpnsites/read | 取得 VPN 網站資源。 |
> | 動作 | Microsoft.Network/vpnsites/write | 建立或更新 VPN 網站資源。 |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.NotificationHubs/CheckNamespaceAvailability/action | 檢查在 NotificationHub 服務內，給定的命名空間資源名稱是否可供使用。 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/authorizationRules/action | 取得命名空間授權規則描述的清單。 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/authorizationRules/delete | 刪除命名空間授權規則。 您無法刪除預設的命名空間授權規則。  |
> | 動作 | Microsoft.NotificationHubs/Namespaces/authorizationRules/listkeys/action | 取得命名空間的連接字串 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/authorizationRules/read | 取得命名空間授權規則描述的清單。 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/authorizationRules/regenerateKeys/action | 命名空間授權規則會重新產生主要/次要金鑰，指定需要重新產生的金鑰 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/authorizationRules/write | 建立命名空間層級的授權規則，並更新其屬性。 您可以更新授權規則的存取權限、主要金鑰和次要金鑰。 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/CheckNotificationHubAvailability/action | 檢查在命名空間內，給定的 NotificationHub 名稱是否可供使用。 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/Delete | 刪除命名空間資源 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/action | 取得通知中樞授權規則的清單 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/delete | 刪除通知中樞的授權規則 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/listkeys/action | 取得通知中樞的連接字串 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/read | 取得通知中樞授權規則的清單 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/regenerateKeys/action | 通知中樞授權規則會重新產生主要/次要金鑰，指定需要重新產生的金鑰 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/write | 建立通知中樞授權規則，並更新其屬性。 您可以更新授權規則的存取權限、主要金鑰和次要金鑰。 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/debugSend/action | 傳送測試推播通知。 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/Delete | 刪除通知中樞資源 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/metricDefinitions/read | 取得命名空間計量資源描述的清單 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/pnsCredentials/action | 取得所有通知中樞的 PNS 認證。 這包括 WNS、MPNS、APNS、GCM 和百度認證 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/read | 取得通知中樞資源描述的清單 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/NotificationHubs/write | 建立通知中樞，並更新其屬性。 其屬性主要包括 PNS 認證。 授權規則與 TTL |
> | 動作 | Microsoft.NotificationHubs/Namespaces/read | 取得命名空間資源描述的清單 |
> | 動作 | Microsoft.NotificationHubs/Namespaces/write | 建立命名空間資源，並更新其屬性。 命名空間的標記和容量是可以更新的屬性。 |
> | 動作 | Microsoft.NotificationHubs/operationResults/read | 傳回通知中樞提供者的作業結果 |
> | 動作 | Microsoft.NotificationHubs/operations/read | 傳回通知中樞提供者支援的作業清單 |
> | 動作 | Microsoft.NotificationHubs/register/action | 針對 NotifciationHubs 資源提供者註冊訂用帳戶，並讓您能夠建立命名空間和 NotificationHubs |
> | 動作 | Microsoft.NotificationHubs/unregister/action | 取消 NotificationHubs 資源提供者訂用帳戶的註冊，以便建立 Namespaces 與 NotificationHubs |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.OffAzure/HyperVSites/clusters/read | 取得 Hyper-V 叢集的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/clusters/read | 取得 Hyper-V 叢集的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/clusters/write | 建立或更新 Hyper-V 叢集 |
> | 動作 | Microsoft.OffAzure/HyperVSites/clusters/write | 建立或更新 Hyper-V 叢集 |
> | 動作 | Microsoft.OffAzure/HyperVSites/delete | 刪除 Hyper-V 站台 |
> | 動作 | Microsoft.OffAzure/HyperVSites/delete | 刪除 Hyper-V 站台 |
> | 動作 | Microsoft.OffAzure/HyperVSites/hosts/read | 取得 Hyper-V 主機的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/hosts/read | 取得 Hyper-V 主機的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/hosts/write | 建立或更新 Hyper-V 主機 |
> | 動作 | Microsoft.OffAzure/HyperVSites/hosts/write | 建立或更新 Hyper-V 主機 |
> | 動作 | Microsoft.OffAzure/HyperVSites/jobs/read | 取得 Hyper-V 作業的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/jobs/read | 取得 Hyper-V 作業的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/machines/read | 取得 Hyper-V 機器的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/machines/read | 取得 Hyper-V 機器的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/machines/stop/action | 停止 Hyper-V 機器 |
> | 動作 | Microsoft.OffAzure/HyperVSites/machines/stop/action | 停止 Hyper-V 機器 |
> | 動作 | Microsoft.OffAzure/HyperVSites/operationsstatus/read | 取得 Hyper-V 作業狀態的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/operationsstatus/read | 取得 Hyper-V 作業狀態的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/read | 取得 Hyper-V 站台的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/read | 取得 Hyper-V 站台的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/refresh/action | 重新整理 Hyper-V 站台內的物件 |
> | 動作 | Microsoft.OffAzure/HyperVSites/refresh/action | 重新整理 Hyper-V 站台內的物件 |
> | 動作 | Microsoft.OffAzure/HyperVSites/runasaccounts/read | 取得 Hyper-V 執行身分帳戶的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/runasaccounts/read | 取得 Hyper-V 執行身分帳戶的屬性 |
> | 動作 | Microsoft.OffAzure/HyperVSites/usage/read | 取得 Hyper-V 站台的使用情形 |
> | 動作 | Microsoft.OffAzure/HyperVSites/usage/read | 取得 Hyper-V 站台的使用情形 |
> | 動作 | Microsoft.OffAzure/HyperVSites/write | 建立或更新 Hyper-V 站台 |
> | 動作 | Microsoft.OffAzure/HyperVSites/write | 建立或更新 Hyper-V 站台 |
> | 動作 | Microsoft.OffAzure/Operations/read | 讀取公開的作業 |
> | 動作 | Microsoft.OffAzure/VMwareSites/delete | 刪除 VMware 站台 |
> | 動作 | Microsoft.OffAzure/VMwareSites/delete | 刪除 VMware 站台 |
> | 動作 | Microsoft.OffAzure/VMwareSites/jobs/read | 取得 VMware 作業的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/jobs/read | 取得 VMware 作業的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/machines/read | 取得 VMware 機器的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/machines/read | 取得 VMware 機器的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/machines/stop/action | 停止 VMware 機器 |
> | 動作 | Microsoft.OffAzure/VMwareSites/machines/stop/action | 停止 VMware 機器 |
> | 動作 | Microsoft.OffAzure/VMwareSites/operationsstatus/read | 取得 VMware 作業狀態的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/operationsstatus/read | 取得 VMware 作業狀態的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/read | 取得 VMware 站台的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/read | 取得 VMware 站台的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/refresh/action | 重新整理 VMware 站台內的物件 |
> | 動作 | Microsoft.OffAzure/VMwareSites/refresh/action | 重新整理 VMware 站台內的物件 |
> | 動作 | Microsoft.OffAzure/VMwareSites/runasaccounts/read | 取得 VMware 執行身分帳戶的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/runasaccounts/read | 取得 VMware 執行身分帳戶的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/usage/read | 取得 VMware 站台的使用情形 |
> | 動作 | Microsoft.OffAzure/VMwareSites/usage/read | 取得 VMware 站台的使用情形 |
> | 動作 | Microsoft.OffAzure/VMwareSites/vcenters/read | 取得 VMware vCenter 的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/vcenters/read | 取得 VMware vCenter 的屬性 |
> | 動作 | Microsoft.OffAzure/VMwareSites/vcenters/write | 建立或更新 VMware vCenter |
> | 動作 | Microsoft.OffAzure/VMwareSites/vcenters/write | 建立或更新 VMware vCenter |
> | 動作 | Microsoft.OffAzure/VMwareSites/write | 建立或更新 VMware 站台 |
> | 動作 | Microsoft.OffAzure/VMwareSites/write | 建立或更新 VMware 站台 |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.OperationalInsights/linkTargets/read | 列出未與 Azure 訂用帳戶相關聯的現有帳戶。 若要將此 Azure 訂用帳戶連結到工作區，請在「建立工作區」作業的客戶識別碼屬性中，使用這項作業所傳回的客戶識別碼。 |
> | 動作 | Microsoft.OperationalInsights/register/action | 向資源提供者註冊訂用帳戶。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/analytics/query/action | 使用新的引擎進行搜尋。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/analytics/query/schema/read | 取得搜尋結構描述 V2。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/api/query/action | 使用新的引擎進行搜尋。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/api/query/schema/read | 取得搜尋結構描述 V2。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/configurationScopes/delete | 刪除組態範圍 |
> | 動作 | Microsoft.OperationalInsights/workspaces/configurationScopes/read | 取得組態範圍 |
> | 動作 | Microsoft.OperationalInsights/workspaces/configurationScopes/write | 設定組態範圍 |
> | 動作 | Microsoft.OperationalInsights/workspaces/datasources/delete | 刪除工作區下的資料來源。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/datasources/read | 取得工作區下的資料來源。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/datasources/write | 在工作區下建立/更新資料來源。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/delete | 刪除工作區。 如果工作區在建立時連結到現有工作區，則系統不會刪除其所連結的工作區。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/generateregistrationcertificate/action | 為工作區產生註冊憑證。 此憑證可用來將 Microsoft System Center Operation Manager 連線到工作區。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/intelligencepacks/disable/action | 為給定工作區停用智慧套件。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/intelligencepacks/enable/action | 為給定工作區啟用智慧套件。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/intelligencepacks/read | 列出給定工作區可見的所有智慧套件，並且也會列出該工作區會啟用還是停用套件。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/linkedServices/delete | 刪除指定工作區下已連結的服務。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/linkedServices/read | 取得指定工作區下已連結的服務。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/linkedServices/write | 建立/更新指定工作區下已連結的服務。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/listKeys/action | 擷取工作區的清單金鑰。 這些金鑰可用來將 Microsoft Operational Insights 代理程式連線到工作區。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/listKeys/read | 擷取工作區的清單金鑰。 這些金鑰可用來將 Microsoft Operational Insights 代理程式連線到工作區。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/managementGroups/read | 取得與此工作區連線之 System Center Operations Manager 管理群組的名稱和中繼資料。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/metricDefinitions/read | 取得工作區下的計量定義 |
> | 動作 | Microsoft.OperationalInsights/workspaces/notificationSettings/delete | 針對工作區刪除使用者的通知設定。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/notificationSettings/read | 針對工作區取得使用者的通知設定。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/notificationSettings/write | 針對工作區設定使用者的通知設定。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/purge/action | 從工作區刪除指定的資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ADAssessmentRecommendation/read | 從 ADAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ADReplicationResult/read | 從 ADReplicationResult 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ADSecurityAssessmentRecommendation/read | 從 ADSecurityAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/Alert/read | 從 Alert 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/AlertHistory/read | 從 AlertHistory 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ApplicationInsights/read | 從 ApplicationInsights 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/AuditLogs/read | 從 AuditLogs 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/AzureActivity/read | 從 AzureActivity 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/AzureMetrics/read | 從 AzureMetrics 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/BoundPort/read | 從 BoundPort 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/CommonSecurityLog/read | 從 CommonSecurityLog 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ComputerGroup/read | 從 ComputerGroup 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ConfigurationChange/read | 從 ConfigurationChange 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ConfigurationData/read | 從 ConfigurationData 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ContainerImageInventory/read | 從 ContainerImageInventory 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ContainerInventory/read | 從 ContainerInventory 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ContainerLog/read | 從 ContainerLog 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ContainerServiceLog/read | 從 ContainerServiceLog 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceAppCrash/read | 從 DeviceAppCrash 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceAppLaunch/read | 從 DeviceAppLaunch 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceCalendar/read | 從 DeviceCalendar 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceCleanup/read | 從 DeviceCleanup 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceConnectSession/read | 從 DeviceConnectSession 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceEtw/read | 從 DeviceEtw 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceHardwareHealth/read | 從 DeviceHardwareHealth 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceHealth/read | 從 DeviceHealth 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceHeartbeat/read | 從 DeviceHeartbeat 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceSkypeHeartbeat/read | 從 DeviceSkypeHeartbeat 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceSkypeSignIn/read | 從 DeviceSkypeSignIn 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DeviceSleepState/read | 從 DeviceSleepState 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DHAppFailure/read | 從 DHAppFailure 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DHAppReliability/read | 從 DHAppReliability 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DHDriverReliability/read | 從 DHDriverReliability 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DHLogonFailures/read | 從 DHLogonFailures 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DHLogonMetrics/read | 從 DHLogonMetrics 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DHOSCrashData/read | 從 DHOSCrashData 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DHOSReliability/read | 從 DHOSReliability 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DHWipAppLearning/read | 從 DHWipAppLearning 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DnsEvents/read | 從 DnsEvents 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/DnsInventory/read | 從 DnsInventory 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ETWEvent/read | 從 ETWEvent 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/Event/read | 從 Event 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ExchangeAssessmentRecommendation/read | 從 ExchangeAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ExchangeOnlineAssessmentRecommendation/read | 從 ExchangeOnlineAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/Heartbeat/read | 從 Heartbeat 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/IISAssessmentRecommendation/read | 從 IISAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/InboundConnection/read | 從 InboundConnection 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/KubeNodeInventory/read | 從 KubeNodeInventory 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/KubePodInventory/read | 從 KubePodInventory 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/LinuxAuditLog/read | 從 LinuxAuditLog 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAApplication/read | 從 MAApplication 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAApplicationHealth/read | 從 MAApplicationHealth 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAApplicationHealthAlternativeVersions/read | 從 MAApplicationHealthAlternativeVersions 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAApplicationHealthIssues/read | 從 MAApplicationHealthIssues 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAApplicationInstance/read | 從 MAApplicationInstance 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAApplicationInstanceReadiness/read | 從 MAApplicationInstanceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAApplicationReadiness/read | 從 MAApplicationReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MADeploymentPlan/read | 從 MADeploymentPlan 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MADevice/read | 從 MADevice 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MADevicePnPHealth/read | 從 MADevicePnPHealth 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MADevicePnPHealthAlternativeVersions/read | 從 MADevicePnPHealthAlternativeVersions 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MADevicePnPHealthIssues/read | 從 MADevicePnPHealthIssues 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MADeviceReadiness/read | 從 MADeviceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MADriverInstanceReadiness/read | 從 MADriverInstanceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MADriverReadiness/read | 從 MADriverReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeAddin/read | 從 MAOfficeAddin 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeAddinHealth/read | 從 MAOfficeAddinHealth 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeAddinHealthIssues/read | 從 MAOfficeAddinHealthIssues 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeAddinInstance/read | 從 MAOfficeAddinInstance 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeAddinInstanceReadiness/read | 從 MAOfficeAddinInstanceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeAddinReadiness/read | 從 MAOfficeAddinReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeApp/read | 從 MAOfficeApp 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeAppHealth/read | 從 MAOfficeAppHealth 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeAppInstance/read | 從 MAOfficeAppInstance 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeAppReadiness/read | 從 MAOfficeAppReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeBuildInfo/read | 從 MAOfficeBuildInfo 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeCurrencyAssessment/read | 從 MAOfficeCurrencyAssessment 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeCurrencyAssessmentDailyCounts/read | 從 MAOfficeCurrencyAssessmentDailyCounts 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeDeploymentStatus/read | 從 MAOfficeDeploymentStatus 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeMacroHealth/read | 從 MAOfficeMacroHealth 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeMacroHealthIssues/read | 從 MAOfficeMacroHealthIssues 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeMacroIssueInstanceReadiness/read | 從 MAOfficeMacroIssueInstanceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeMacroIssueReadiness/read | 從 MAOfficeMacroIssueReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeMacroSummary/read | 從 MAOfficeMacroSummary 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeSuite/read | 從 MAOfficeSuite 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAOfficeSuiteInstance/read | 從 MAOfficeSuiteInstance 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAProposedPilotDevices/read | 從 MAProposedPilotDevices 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAWindowsBuildInfo/read | 從 MAWindowsBuildInfo 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAWindowsCurrencyAssessment/read | 從 MAWindowsCurrencyAssessment 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAWindowsCurrencyAssessmentDailyCounts/read | 從 MAWindowsCurrencyAssessmentDailyCounts 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAWindowsDeploymentStatus/read | 從 MAWindowsDeploymentStatus 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/MAWindowsSysReqInstanceReadiness/read | 從 MAWindowsSysReqInstanceReadiness 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/NetworkMonitoring/read | 從 NetworkMonitoring 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/OfficeActivity/read | 從 OfficeActivity 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/Operation/read | 從 Operation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/OutboundConnection/read | 從 OutboundConnection 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/Perf/read | 從 Perf 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ProtectionStatus/read | 從 ProtectionStatus 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/read | 針對工作區中的資料執行查詢 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ReservedAzureCommonFields/read | 從 ReservedAzureCommonFields 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ReservedCommonFields/read | 從 ReservedCommonFields 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SCCMAssessmentRecommendation/read | 從 SCCMAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SCOMAssessmentRecommendation/read | 從 SCOMAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SecurityAlert/read | 從 SecurityAlert 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SecurityBaseline/read | 從 SecurityBaseline 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SecurityBaselineSummary/read | 從 SecurityBaselineSummary 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SecurityDetection/read | 從 SecurityDetection 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SecurityEvent/read | 從 SecurityEvent 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ServiceFabricOperationalEvent/read | 從 ServiceFabricOperationalEvent 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ServiceFabricReliableActorEvent/read | 從 ServiceFabricReliableActorEvent 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/ServiceFabricReliableServiceEvent/read | 從 ServiceFabricReliableServiceEvent 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SfBAssessmentRecommendation/read | 從 SfBAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SfBOnlineAssessmentRecommendation/read | 從 SfBOnlineAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SharePointOnlineAssessmentRecommendation/read | 從 SharePointOnlineAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SigninLogs/read | 從 SigninLogs 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SPAssessmentRecommendation/read | 從 SPAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SQLAssessmentRecommendation/read | 從 SQLAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SQLQueryPerformance/read | 從 SQLQueryPerformance 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/Syslog/read | 從 Syslog 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/SysmonEvent/read | 從 SysmonEvent 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/Tables.Custom/read | 從任何自訂記錄檔讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UAApp/read | 從 UAApp 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UAComputer/read | 從 UAComputer 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UAComputer/read | 從 UAComputerRank 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UADriver/read | 從 UADriver 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UADriverProblemCodes/read | 從 UADriverProblemCodes 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UAFeedback/read | 從 UAFeedback 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UAHardwareSecurity/read | 從 UAHardwareSecurity 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UAIESiteDiscovery/read | 從 UAIESiteDiscovery 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UAOfficeAddIn/read | 從 UAOfficeAddIn 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UAProposedActionPlan/read | 從 UAProposedActionPlan 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UASysReqIssue/read | 從 UASysReqIssue 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UAUpgradedComputer/read | 從 UAUpgradedComputer 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/Update/read | 從 Update 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UpdateRunProgress/read | 從 UpdateRunProgress 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/UpdateSummary/read | 從 UpdateSummary 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/Usage/read | 從 Usage 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/VMBoundPort/read | 從 VMBoundPort 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/VMConnection/read | 從 VMConnection 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/W3CIISLog/read | 從 W3CIISLog 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WaaSDeploymentStatus/read | 從 WaaSDeploymentStatus 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WaaSInsiderStatus/read | 從 WaaSInsiderStatus 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WaaSUpdateStatus/read | 從 WaaSUpdateStatus 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WDAVStatus/read | 從 WDAVStatus 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WDAVThreat/read | 從 WDAVThreat 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WindowsClientAssessmentRecommendation/read | 從 WindowsClientAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WindowsEvent/read | 從 WindowsEvent 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WindowsFirewall/read | 從 WindowsFirewall 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WindowsServerAssessmentRecommendation/read | 從 WindowsServerAssessmentRecommendation 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WireData/read | 從 WireData 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WorkloadMonitoringPerf/read | 從 WorkloadMonitoringPerf 資料表中讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WUDOAggregatedStatus/read | 從 WUDOAggregatedStatus 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/query/WUDOStatus/read | 從 WUDOStatus 資料表讀取資料 |
> | 動作 | Microsoft.OperationalInsights/workspaces/read | 取得現有工作區 |
> | 動作 | Microsoft.OperationalInsights/workspaces/savedSearches/delete | 刪除已儲存的搜尋查詢 |
> | 動作 | Microsoft.OperationalInsights/workspaces/savedSearches/read | 取得已儲存的搜尋查詢 |
> | 動作 | Microsoft.OperationalInsights/workspaces/savedSearches/write | 建立已儲存的搜尋查詢 |
> | 動作 | Microsoft.OperationalInsights/workspaces/schema/read | 取得工作區的搜尋結構描述。  搜尋結構描述包括公開的欄位和其類型。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/search/action | 執行搜尋查詢 |
> | 動作 | Microsoft.OperationalInsights/workspaces/sharedKeys/action | 擷取工作區的共用金鑰。 這些金鑰可用來將 Microsoft Operational Insights 代理程式連線到工作區。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/sharedKeys/read | 擷取工作區的共用金鑰。 這些金鑰可用來將 Microsoft Operational Insights 代理程式連線到工作區。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/storageinsightconfigs/delete | 刪除儲存體組態。 這會讓 Microsoft Operational Insights 停止從儲存體帳戶讀取資料。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/storageinsightconfigs/read | 取得儲存體組態。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/storageinsightconfigs/write | 建立新的儲存體組態。 這些組態可用來提取現有儲存體帳戶中來自某個位置的資料。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/usages/read | 取得工作區的使用量資料，包括工作區所讀取的資料量。 |
> | 動作 | Microsoft.OperationalInsights/workspaces/write | 建立新的工作區，或藉由提供來自現有工作區的客戶識別碼來連結至現有工作區。 |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.OperationsManagement/managementAssociations/delete | 刪除現有的管理關聯 |
> | 動作 | Microsoft.OperationsManagement/managementAssociations/read | 取得現有的管理關聯 |
> | 動作 | Microsoft.OperationsManagement/managementAssociations/write | 建立新的管理關聯 |
> | 動作 | Microsoft.OperationsManagement/managementConfigurations/delete | 刪除現有的管理設定 |
> | 動作 | Microsoft.OperationsManagement/managementConfigurations/read | 取得現有的管理設定 |
> | 動作 | Microsoft.OperationsManagement/managementConfigurations/write | 建立新的管理設定 |
> | 動作 | Microsoft.OperationsManagement/register/action | 向資源提供者註冊訂用帳戶。 |
> | 動作 | Microsoft.OperationsManagement/solutions/delete | 刪除現有的 OMS 解決方案 |
> | 動作 | Microsoft.OperationsManagement/solutions/read | 取得現有的 OMS 解決方案 |
> | 動作 | Microsoft.OperationsManagement/solutions/write | 建立新的 OMS 解決方案 |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.PolicyInsights/asyncOperationResults/read | 取得非同步作業結果。 |
> | 動作 | Microsoft.PolicyInsights/policyEvents/queryResults/action | 查詢原則事件的相關資訊。 |
> | 動作 | Microsoft.PolicyInsights/policyEvents/queryResults/read | 查詢原則事件的相關資訊。 |
> | 動作 | Microsoft.PolicyInsights/policyStates/queryResults/action | 查詢原則狀態的相關資訊。 |
> | 動作 | Microsoft.PolicyInsights/policyStates/queryResults/read | 查詢原則狀態的相關資訊。 |
> | 動作 | Microsoft.PolicyInsights/policyStates/summarize/action | 查詢原則最新狀態的摘要資訊。 |
> | 動作 | Microsoft.PolicyInsights/policyStates/summarize/read | 查詢原則最新狀態的摘要資訊。 |
> | 動作 | Microsoft.PolicyInsights/policyStates/triggerEvaluation/action | 對選取的範圍觸發新的合規性評估。 |
> | 動作 | Microsoft.PolicyInsights/policyTrackedResources/queryResults/read | 查詢 DeployIfNotExists 原則所需之資源的相關資訊。 |
> | 動作 | Microsoft.PolicyInsights/register/action | 註冊原則見解資源提供者，並允許對其執行動作。 |
> | 動作 | Microsoft.PolicyInsights/remediations/cancel/action | 取消正在進行的原則補救。 |
> | 動作 | Microsoft.PolicyInsights/remediations/delete | 刪除原則補救。 |
> | 動作 | Microsoft.PolicyInsights/remediations/listDeployments/read | 列出原則補救所需的部署。 |
> | 動作 | Microsoft.PolicyInsights/remediations/read | 取得原則補救。 |
> | 動作 | Microsoft.PolicyInsights/remediations/write | 建立或更新原則補救。 |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Portal/dashboards/delete | 從訂用帳戶移除儀表板。 |
> | 動作 | Microsoft.Portal/dashboards/read | 讀取訂用帳戶的儀表板。 |
> | 動作 | Microsoft.Portal/dashboards/write | 新增或修改訂用帳戶的儀表板。 |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.PowerBIDedicated/capacities/checkNameAvailability/action | 檢查指定的 Power BI 專用容量名稱有效且未在使用中。 |
> | 動作 | Microsoft.PowerBIDedicated/capacities/delete | 刪除 Power BI 專用容量。 |
> | 動作 | Microsoft.PowerBIDedicated/capacities/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.PowerBIDedicated/capacities/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.PowerBIDedicated/capacities/providers/Microsoft.Insights/logDefinitions/read | 取得 Power BI 專用容量的可用記錄 |
> | 動作 | Microsoft.PowerBIDedicated/capacities/providers/Microsoft.Insights/metricDefinitions/read | 取得 Power BI 專用容量的可用計量。 |
> | 動作 | Microsoft.PowerBIDedicated/capacities/read | 擷取指定 Power BI 專用容量的資訊。 |
> | 動作 | Microsoft.PowerBIDedicated/capacities/resume/action | 繼續容量。 |
> | 動作 | Microsoft.PowerBIDedicated/capacities/skus/read | 擷取容量的可用 SKU 資訊 |
> | 動作 | Microsoft.PowerBIDedicated/capacities/suspend/action | 暫止容量。 |
> | 動作 | Microsoft.PowerBIDedicated/capacities/write | 建立或更新指定的 Power BI 專用容量。 |
> | 動作 | Microsoft.PowerBIDedicated/locations/operationresults/read | 擷取指定作業結果的資訊。 |
> | 動作 | Microsoft.PowerBIDedicated/locations/operationstatuses/read | 擷取指定作業狀態的資訊。 |
> | 動作 | Microsoft.PowerBIDedicated/operations/read | 擷取作業的資訊 |
> | 動作 | Microsoft.PowerBIDedicated/register/action | 註冊 Power BI 專用資源提供者。 |
> | 動作 | Microsoft.PowerBIDedicated/skus/read | 擷取 SKU 的資訊 |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.RecoveryServices/locations/allocatedStamp/read | GetAllocatedStamp 是服務所使用的內部作業 |
> | 動作 | Microsoft.RecoveryServices/locations/allocateStamp/action | AllocateStamp 是服務所使用的內部作業 |
> | 動作 | Microsoft.RecoveryServices/locations/backupPreValidateProtection/action |  |
> | 動作 | Microsoft.RecoveryServices/locations/backupStatus/action | 檢查復原服務保存庫的備份狀態 |
> | 動作 | Microsoft.RecoveryServices/locations/backupValidateFeatures/action | 驗證功能 |
> | 動作 | Microsoft.RecoveryServices/locations/operationStatus/read | 取得給定作業的作業狀態 |
> | 動作 | Microsoft.RecoveryServices/operations/read | 作業會傳回資源提供者的作業清單 |
> | 動作 | Microsoft.RecoveryServices/register/action | 為指定的資源提供者註冊訂用帳戶 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupconfig/read | 傳回復原服務保存庫的組態。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupconfig/write | 更新復原服務保存庫的組態。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupEngines/read | 傳回已向保存庫註冊的所有備份管理伺服器。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/backupProtectionIntent/delete | 刪除備份保護用途 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/backupProtectionIntent/read | 取得備份保護用途 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/backupProtectionIntent/write | 建立備份保護用途 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/operationResults/read | 傳回作業的狀態 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectableContainers/read | 取得所有可保護的容器 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/delete | 刪除已註冊的容器 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/inquire/action | 執行容器內工作負載的查詢 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/items/read | 取得容器中的所有項目 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/operationResults/read | 取得對保護容器執行之作業的結果。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/backup/action | 對受保護的項目執行備份。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/delete | 刪除受保護的項目 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/operationResults/read | 取得對受保護項目執行之作業的結果。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/operationsStatus/read | 傳回對受保護項目執行之作業的狀態。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/read | 傳回受保護項目的物件詳細資料 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/provisionInstantItemRecovery/action | 為受保護的項目佈建即時項目復原 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/read | 取得受保護項目的復原點。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/restore/action | 還原受保護項目的復原點。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/revokeInstantItemRecovery/action | 為受保護的項目撤銷即時項目復原 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/write | 建立備用的受保護項目 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/read | 傳回所有已註冊的容器 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/write | 建立已註冊的容器 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupFabrics/refreshContainers/action | 重新整理容器清單 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupJobs/cancel/action | 取消作業 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupJobs/operationResults/read | 傳回作業的作業結果。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupJobs/read | 傳回所有作業物件 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupJobsExport/action | 匯出作業 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupJobsExport/operationResults/read | 傳回匯出作業的作業結果。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupManagementMetaData/read | 傳回復原服務保存庫的備份管理中繼資料。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupOperationResults/read | 傳回復原服務保存庫的備份作業結果。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupOperations/read | 傳回復原服務保存庫的備份作業狀態。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupPolicies/delete | 刪除保護原則 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupPolicies/operationResults/read | 取得原則作業的結果。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupPolicies/operations/read | 取得原則作業的狀態。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupPolicies/read | 傳回所有保護原則 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupPolicies/write | 建立保護原則 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupProtectableItems/read | 傳回所有可保護項目的清單。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupProtectedItems/read | 傳回所有受保護項目的清單。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupProtectionContainers/read | 傳回屬於訂用帳戶的所有容器 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupProtectionIntents/read | 列出所有的備份保護用途 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupSecurityPIN/action | 傳回復原服務保存庫的安全性 PIN 碼資訊。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupstorageconfig/read | 傳回復原服務保存庫的儲存體組態。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupstorageconfig/write | 更新復原服務保存庫的儲存體組態。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupUsageSummaries/read | 傳回復原服務之受保護項目和受保護伺服器的摘要。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/backupValidateOperation/action | 驗證受保護項目上的作業 |
> | 動作 | Microsoft.RecoveryServices/Vaults/certificates/write | 「更新資源憑證」作業會更新資源/保存庫的認證憑證。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/delete | 「刪除保存庫」作業會刪除 'vault' 類型的指定 Azure 資源 |
> | 動作 | Microsoft.RecoveryServices/Vaults/extendedInformation/delete | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | 動作 | Microsoft.RecoveryServices/Vaults/extendedInformation/read | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | 動作 | Microsoft.RecoveryServices/Vaults/extendedInformation/write | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | 動作 | Microsoft.RecoveryServices/Vaults/monitoringAlerts/read | 取得復原服務保存庫的警示。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/monitoringAlerts/write | 解決警示。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/monitoringConfigurations/read | 取得復原服務保存庫通知組態。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/monitoringConfigurations/write | 設定復原服務保存庫的電子郵件通知。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/providers/Microsoft.Insights/diagnosticSettings/read | Azure 備份診斷 |
> | 動作 | Microsoft.RecoveryServices/Vaults/providers/Microsoft.Insights/diagnosticSettings/write | Azure 備份診斷 |
> | 動作 | Microsoft.RecoveryServices/Vaults/providers/Microsoft.Insights/logDefinitions/read | Azure 備份記錄 |
> | 動作 | Microsoft.RecoveryServices/Vaults/providers/Microsoft.Insights/metricDefinitions/read | Azure 備份計量 |
> | 動作 | Microsoft.RecoveryServices/Vaults/read | 「取得保存庫」作業會取得物件，此物件代表 'vault' 類型的 Azure 資源 |
> | 動作 | Microsoft.RecoveryServices/Vaults/registeredIdentities/delete | 「取消註冊容器」作業可用來取消註冊容器。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/registeredIdentities/operationResults/read | 「取得作業結果」作業可用來取得以非同步方式提交之作業的作業狀態和結果 |
> | 動作 | Microsoft.RecoveryServices/Vaults/registeredIdentities/read | 「取得容器」作業可用來取得為資源註冊的容器。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/registeredIdentities/write | 「註冊服務容器」作業可用來向復原服務註冊容器。 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationAlertSettings/read | 讀取任何警示設定 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationAlertSettings/write | 建立或更新任何警示設定 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationEvents/read | 讀取任何事件 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/checkConsistency/action | 檢查網狀架構的一致性 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/delete | 刪除任何網狀架構 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/deployProcessServerImage/action | 部署處理序伺服器的映像 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/migratetoaad/action | 將網狀架構遷移至 AAD |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/read | 讀取任何網狀架構 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/reassociateGateway/action | 重新關聯閘道 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/remove/action | 移除網狀架構 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/renewcertificate/action | 更新網狀架構的憑證 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationLogicalNetworks/read | 讀取任何邏輯網路 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationNetworks/read | 讀取任何網路 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationNetworks/replicationNetworkMappings/delete | 刪除任何網路對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationNetworks/replicationNetworkMappings/read | 讀取任何網路對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationNetworks/replicationNetworkMappings/write | 建立或更新任何網路對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/discoverProtectableItem/action | 探索可保護的項目 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/read | 讀取任何保護容器 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/remove/action | 移除保護容器 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectableItems/read | 讀取任何可保護的項目 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/applyRecoveryPoint/action | 套用復原點 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/delete | 刪除任何受保護的項目 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/failoverCommit/action | 容錯移轉認可 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/plannedFailover/action | 計劃性容錯移轉 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/read | 讀取任何受保護的項目 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/recoveryPoints/read | 讀取任何複寫復原點 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/remove/action | 移除受保護的項目 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/repairReplication/action | 修復複寫 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/reProtect/action | 重新保護受保護的項目 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/submitFeedback/action | 提交意見 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/targetComputeSizes/read | 讀取任何目標計算大小 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/testFailover/action | Test Failover |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/testFailoverCleanup/action | 測試容錯移轉清理 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/unplannedFailover/action | 容錯移轉 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/updateMobilityService/action | 更新行動服務 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/write | 建立或更新任何受保護的項目 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectionContainerMappings/delete | 刪除任何保護容器對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectionContainerMappings/read | 讀取任何保護容器對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectionContainerMappings/remove/action | 移除保護容器對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectionContainerMappings/write | 建立或更新任何保護容器對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/switchprotection/action | 切換保護容器 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/write | 建立或更新任何保護容器 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationRecoveryServicesProviders/delete | 刪除任何復原服務提供者 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationRecoveryServicesProviders/read | 讀取任何復原服務提供者 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationRecoveryServicesProviders/refreshProvider/action | 重新整理提供者 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationRecoveryServicesProviders/remove/action | 移除復原服務提供者 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationRecoveryServicesProviders/write | 建立或更新任何復原服務提供者 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationStorageClassifications/read | 讀取任何存放裝置分類 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationStorageClassifications/replicationStorageClassificationMappings/delete | 刪除任何存放裝置分類對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationStorageClassifications/replicationStorageClassificationMappings/read | 讀取任何存放裝置分類對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationStorageClassifications/replicationStorageClassificationMappings/write | 建立或更新任何存放裝置分類對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationvCenters/delete | 刪除任何 vCenter |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationvCenters/read | 讀取任何 vCenter |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationvCenters/write | 建立或更新任何 vCenter |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationFabrics/write | 建立或更新任何網狀架構 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationJobs/cancel/action | 取消作業 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationJobs/read | 讀取任何作業 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationJobs/restart/action | 重新啟動作業 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationJobs/resume/action | 繼續作業 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationNetworkMappings/read | 讀取任何網路對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationNetworks/read | 讀取任何網路 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationPolicies/delete | 刪除任何原則 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationPolicies/read | 讀取任何原則 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationPolicies/write | 建立或更新任何原則 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationProtectedItems/read | 讀取任何受保護的項目 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationProtectionContainerMappings/read | 讀取任何保護容器對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationProtectionContainers/read | 讀取任何保護容器 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/delete | 刪除任何復原方案 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/failoverCommit/action | 容錯移轉認可復原方案 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/plannedFailover/action | 計劃性容錯移轉復原方案 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/read | 讀取任何復原方案 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/reProtect/action | 重新保護復原方案 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/testFailover/action | 測試容錯移轉復原方案 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/testFailoverCleanup/action | 測試容錯移轉清理復原方案 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/unplannedFailover/action | 容錯移轉復原方案 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/write | 建立或更新任何復原方案 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationRecoveryServicesProviders/read | 讀取任何復原服務提供者 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationStorageClassificationMappings/read | 讀取任何存放裝置分類對應 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationStorageClassifications/read | 讀取任何存放裝置分類 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationUsages/read | 讀取任何保存庫複寫使用量 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationVaultHealth/read | 讀取任何保存庫複寫健康情況 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationVaultHealth/refresh/action | 重新整理保存庫健康情況 |
> | 動作 | Microsoft.RecoveryServices/vaults/replicationvCenters/read | 讀取任何 vCenter |
> | 動作 | Microsoft.RecoveryServices/Vaults/tokenInfo/read | 傳回復原服務保存庫的權杖資訊。 |
> | 動作 | Microsoft.RecoveryServices/vaults/usages/read | 讀取任何保存庫使用量 |
> | 動作 | Microsoft.RecoveryServices/Vaults/usages/read | 傳回復原服務保存庫的使用量詳細資料。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/vaultTokens/read | 「保存庫權杖」作業可用來取得保存庫層級後端作業的保存庫權杖。 |
> | 動作 | Microsoft.RecoveryServices/Vaults/write | 「建立保存庫」作業會建立 'vault' 類型的 Azure 資源 |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Relay/checkNameAvailability/action | 檢查給定訂用帳戶下的命名空間是否可用。 |
> | 動作 | Microsoft.Relay/checkNamespaceAvailability/action | 檢查給定訂用帳戶下的命名空間是否可用。 此 API 即將淘汰。請改用 CheckNameAvailabiltiy。 |
> | 動作 | Microsoft.Relay/namespaces/authorizationRules/action | 更新命名空間授權規則。 此 API 即將淘汰。 請改用 PUT 呼叫來更新命名空間授權規則。 API 版本 2017-04-01 不支援此作業。 |
> | 動作 | Microsoft.Relay/namespaces/authorizationRules/delete | 刪除命名空間授權規則。 您無法刪除預設的命名空間授權規則。  |
> | 動作 | Microsoft.Relay/namespaces/authorizationRules/listkeys/action | 取得命名空間的連接字串 |
> | 動作 | Microsoft.Relay/namespaces/authorizationRules/read | 取得命名空間授權規則描述的清單。 |
> | 動作 | Microsoft.Relay/namespaces/authorizationRules/regenerateKeys/action | 對資源重新產生主要或次要金鑰 |
> | 動作 | Microsoft.Relay/namespaces/authorizationRules/write | 建立命名空間層級的授權規則，並更新其屬性。 您可以更新授權規則的存取權限、主要金鑰和次要金鑰。 |
> | 動作 | Microsoft.Relay/namespaces/Delete | 刪除命名空間資源 |
> | 動作 | Microsoft.Relay/namespaces/disasterRecoveryConfigs/authorizationRules/listkeys/action | 取得災害復原主要命名空間的授權規則金鑰 |
> | 動作 | Microsoft.Relay/namespaces/disasterRecoveryConfigs/authorizationRules/read | 取得災害復原主要命名空間的授權規則 |
> | 動作 | Microsoft.Relay/namespaces/disasterRecoveryConfigs/breakPairing/action | 停用災害復原，並停止將變更從主要命名空間複寫至次要命名空間。 |
> | 動作 | Microsoft.Relay/namespaces/disasterrecoveryconfigs/checkNameAvailability/action | 檢查指定訂用帳戶下命名空間別名的可用性。 |
> | 動作 | Microsoft.Relay/namespaces/disasterRecoveryConfigs/delete | 刪除與命名空間相關聯的災害復原設定。 此作業只能透過主要命名空間叫用。 |
> | 動作 | Microsoft.Relay/namespaces/disasterRecoveryConfigs/failover/action | 叫用異地災害復原容錯移轉，並將命名空間別名重新設定為指向次要命名空間。 |
> | 動作 | Microsoft.Relay/namespaces/disasterRecoveryConfigs/read | 取得與命名空間相關聯的災害復原設定。 |
> | 動作 | Microsoft.Relay/namespaces/disasterRecoveryConfigs/write | 建立或更新與命名空間相關聯的災害復原設定。 |
> | 動作 | Microsoft.Relay/namespaces/HybridConnections/authorizationRules/action | 更新 HybridConnection 的作業。 API 版本 2017-04-01 不支援此作業。 授權規則。 請使用 PUT 呼叫來更新授權規則。 |
> | 動作 | Microsoft.Relay/namespaces/HybridConnections/authorizationRules/delete | 用來刪除 HybridConnection 授權規則的作業 |
> | 動作 | Microsoft.Relay/namespaces/HybridConnections/authorizationRules/listkeys/action | 取得 HybridConnection 的連接字串 |
> | 動作 | Microsoft.Relay/namespaces/HybridConnections/authorizationRules/read |  取得 HybridConnection 授權規則的清單 |
> | 動作 | Microsoft.Relay/namespaces/HybridConnections/authorizationRules/regeneratekeys/action | 對資源重新產生主要或次要金鑰 |
> | 動作 | Microsoft.Relay/namespaces/HybridConnections/authorizationRules/write | 建立 HybridConnection 授權規則，並更新其屬性。 您可以更新授權規則存取權限。 |
> | 動作 | Microsoft.Relay/namespaces/HybridConnections/Delete | 用來刪除 HybridConnection 資源的作業 |
> | 動作 | Microsoft.Relay/namespaces/HybridConnections/read | 取得 HybridConnection 資源描述的清單 |
> | 動作 | Microsoft.Relay/namespaces/HybridConnections/write | 建立或更新 HybridConnection 屬性。 |
> | 動作 | Microsoft.Relay/namespaces/messagingPlan/read | 取得命名空間的傳訊方案。<br>此 API 即將淘汰。<br>透過 MessagingPlan 資源公開的屬性，已移至新版 API 的 (父系) 命名空間資源。<br>API 版本 2017-04-01 不支援此作業。 |
> | 動作 | Microsoft.Relay/namespaces/messagingPlan/write | 更新命名空間的傳訊方案。<br>此 API 即將淘汰。<br>透過 MessagingPlan 資源公開的屬性，已移至新版 API 的 (父系) 命名空間資源。<br>API 版本 2017-04-01 不支援此作業。 |
> | 動作 | Microsoft.Relay/namespaces/operationresults/read | 取得命名空間作業的狀態 |
> | 動作 | Microsoft.Relay/namespaces/providers/Microsoft.Insights/metricDefinitions/read | 取得命名空間計量資源描述的清單 |
> | 動作 | Microsoft.Relay/namespaces/read | 取得命名空間資源描述的清單 |
> | 動作 | Microsoft.Relay/namespaces/removeAcsNamepsace/action | 移除 ACS 命名空間 |
> | 動作 | Microsoft.Relay/namespaces/WcfRelays/authorizationRules/action | 更新 WcfRelay 的作業。 API 版本 2017-04-01 不支援此作業。 授權規則。 請使用 PUT 呼叫來更新授權規則。 |
> | 動作 | Microsoft.Relay/namespaces/WcfRelays/authorizationRules/delete | 用來刪除 WcfRelay 授權規則的作業 |
> | 動作 | Microsoft.Relay/namespaces/WcfRelays/authorizationRules/listkeys/action | 取得 WcfRelay 的連接字串 |
> | 動作 | Microsoft.Relay/namespaces/WcfRelays/authorizationRules/read |  取得 WcfRelay 授權規則的清單 |
> | 動作 | Microsoft.Relay/namespaces/WcfRelays/authorizationRules/regeneratekeys/action | 對資源重新產生主要或次要金鑰 |
> | 動作 | Microsoft.Relay/namespaces/WcfRelays/authorizationRules/write | 建立 WcfRelay 授權規則，並更新其屬性。 您可以更新授權規則存取權限。 |
> | 動作 | Microsoft.Relay/namespaces/WcfRelays/Delete | 用來刪除 WcfRelay 資源的作業 |
> | 動作 | Microsoft.Relay/namespaces/WcfRelays/read | 取得 WcfRelay 資源描述的清單 |
> | 動作 | Microsoft.Relay/namespaces/WcfRelays/write | 建立或更新 WcfRelay 屬性。 |
> | 動作 | Microsoft.Relay/namespaces/write | 建立命名空間資源，並更新其屬性。 命名空間的標記和容量是可以更新的屬性。 |
> | 動作 | Microsoft.Relay/operations/read | 取得作業 |
> | 動作 | Microsoft.Relay/register/action | 針對轉送資源提供者註冊訂用帳戶，並讓您能夠建立轉送資源 |
> | 動作 | Microsoft.Relay/unregister/action | 針對轉送資源提供者註冊訂用帳戶，並讓您能夠建立轉送資源 |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ResourceHealth/AvailabilityStatuses/current/read | 取得指定資源的可用性狀態 |
> | 動作 | Microsoft.ResourceHealth/AvailabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | 動作 | Microsoft.ResourceHealth/events/read | 取得指定訂用帳戶的服務健康狀態事件 |
> | 動作 | Microsoft.Resourcehealth/healthevent/action | 代表指定資源健康狀態的變更 |
> | 動作 | Microsoft.Resourcehealth/healthevent/Activated/action | 代表指定資源健康狀態的變更 |
> | 動作 | Microsoft.Resourcehealth/healthevent/InProgress/action | 代表指定資源健康狀態的變更 |
> | 動作 | Microsoft.Resourcehealth/healthevent/Pending/action | 代表指定資源健康狀態的變更 |
> | 動作 | Microsoft.Resourcehealth/healthevent/Resolved/action | 代表指定資源健康狀態的變更 |
> | 動作 | Microsoft.Resourcehealth/healthevent/Updated/action | 代表指定資源健康狀態的變更 |
> | 動作 | Microsoft.ResourceHealth/impactedResources/read | 取得指定訂用帳戶的受影響資源 |
> | 動作 | Microsoft.ResourceHealth/register/action | 為 Microsoft ResourceHealth 註冊訂用帳戶 |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Resources/checkPolicyCompliance/action | 檢查指定的資源是否符合資源原則，以了解合規性狀態。 |
> | 動作 | Microsoft.Resources/checkResourceName/action | 檢查資源名稱是否有效。 |
> | 動作 | Microsoft.Resources/deployments/cancel/action | 取消部署。 |
> | 動作 | Microsoft.Resources/deployments/delete | 刪除部署。 |
> | 動作 | Microsoft.Resources/deployments/operations/read | 取得或列出部署作業。 |
> | 動作 | Microsoft.Resources/deployments/read | 取得或列出部署。 |
> | 動作 | Microsoft.Resources/deployments/validate/action | 驗證部署。 |
> | 動作 | Microsoft.Resources/deployments/write | 建立或更新部署。 |
> | 動作 | Microsoft.Resources/links/delete | 刪除資源連結。 |
> | 動作 | Microsoft.Resources/links/read | 取得或列出資源連結。 |
> | 動作 | Microsoft.Resources/links/write | 建立或更新資源連結。 |
> | 動作 | Microsoft.Resources/marketplace/purchase/action | 從 Marketplace 購買資源。 |
> | 動作 | Microsoft.Resources/providers/read | 取得提供者清單。 |
> | 動作 | Microsoft.Resources/resources/read | 根據篩選來取得資源清單。 |
> | 動作 | Microsoft.Resources/subscriptions/locations/read | 取得所支援位置的清單。 |
> | 動作 | Microsoft.Resources/subscriptions/operationresults/read | 取得訂用帳戶作業結果。 |
> | 動作 | Microsoft.Resources/subscriptions/providers/read | 取得或列出資源提供者。 |
> | 動作 | Microsoft.Resources/subscriptions/read | 取得訂用帳戶清單。 |
> | 動作 | Microsoft.Resources/subscriptions/resourceGroups/delete | 刪除資源群組和其所有資源。 |
> | 動作 | Microsoft.Resources/subscriptions/resourcegroups/deployments/operations/read | 取得或列出部署作業。 |
> | 動作 | Microsoft.Resources/subscriptions/resourcegroups/deployments/operationstatuses/read | 取得或列出部署作業狀態。 |
> | 動作 | Microsoft.Resources/subscriptions/resourcegroups/deployments/read | 取得或列出部署。 |
> | 動作 | Microsoft.Resources/subscriptions/resourcegroups/deployments/write | 建立或更新部署。 |
> | 動作 | Microsoft.Resources/subscriptions/resourceGroups/moveResources/action | 將資源從某個資源群組移到另一個資源群組。 |
> | 動作 | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | 動作 | Microsoft.Resources/subscriptions/resourcegroups/resources/read | 取得資源群組的資源。 |
> | 動作 | Microsoft.Resources/subscriptions/resourceGroups/validateMoveResources/action | 驗證資源從某個資源群組移到另一個資源群組的動作。 |
> | 動作 | Microsoft.Resources/subscriptions/resourceGroups/write | 建立或更新資源群組。 |
> | 動作 | Microsoft.Resources/subscriptions/resources/read | 取得訂用帳戶的資源。 |
> | 動作 | Microsoft.Resources/subscriptions/tagNames/delete | 刪除訂用帳戶標記。 |
> | 動作 | Microsoft.Resources/subscriptions/tagNames/read | 取得或列出訂用帳戶標記。 |
> | 動作 | Microsoft.Resources/subscriptions/tagNames/tagValues/delete | 刪除訂用帳戶標記值。 |
> | 動作 | Microsoft.Resources/subscriptions/tagNames/tagValues/read | 取得或列出訂用帳戶標記值。 |
> | 動作 | Microsoft.Resources/subscriptions/tagNames/tagValues/write | 新增訂用帳戶標記值。 |
> | 動作 | Microsoft.Resources/subscriptions/tagNames/write | 新增訂用帳戶標記。 |
> | 動作 | Microsoft.Resources/tenants/read | 取得租用戶清單。 |

## <a name="microsoftscheduler"></a>Microsoft.Scheduler

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Scheduler/jobcollections/delete | 刪除作業集合。 |
> | 動作 | Microsoft.Scheduler/jobcollections/disable/action | 停用作業集合。 |
> | 動作 | Microsoft.Scheduler/jobcollections/enable/action | 啟用作業集合。 |
> | 動作 | Microsoft.Scheduler/jobcollections/jobs/delete | 刪除作業。 |
> | 動作 | Microsoft.Scheduler/jobcollections/jobs/generateLogicAppDefinition/action | 根據排程器作業來產生邏輯應用程式定義。 |
> | 動作 | Microsoft.Scheduler/jobcollections/jobs/jobhistories/read | 取得工作歷程記錄。 |
> | 動作 | Microsoft.Scheduler/jobcollections/jobs/read | 取得作業。 |
> | 動作 | Microsoft.Scheduler/jobcollections/jobs/run/action | 執行作業。 |
> | 動作 | Microsoft.Scheduler/jobcollections/jobs/write | 建立或更新作業。 |
> | 動作 | Microsoft.Scheduler/jobcollections/read | 取得作業集合 |
> | 動作 | Microsoft.Scheduler/jobcollections/write | 建立或更新作業集合。 |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Search/checkNameAvailability/action | 檢查服務名稱的可用性。 |
> | 動作 | Microsoft.Search/operations/read | 列出 Microsoft.Search 提供者的所有可用作業。 |
> | 動作 | Microsoft.Search/register/action | 針對搜尋資源提供者註冊訂用帳戶，並讓您能夠建立搜尋服務。 |
> | 動作 | Microsoft.Search/searchServices/createQueryKey/action | 建立查詢金鑰。 |
> | 動作 | Microsoft.Search/searchServices/delete | 刪除搜尋服務。 |
> | 動作 | Microsoft.Search/searchServices/deleteQueryKey/delete | 刪除查詢金鑰。 |
> | 動作 | Microsoft.Search/searchServices/diagnosticSettings/read | 取得針對資源讀取的診斷設定 |
> | 動作 | Microsoft.Search/searchServices/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Search/searchServices/listAdminKeys/action | 讀取系統管理金鑰。 |
> | 動作 | Microsoft.Search/searchServices/listQueryKeys/read | 傳回指定 Azure 搜尋服務的查詢 API 金鑰清單。 |
> | 動作 | Microsoft.Search/searchServices/logDefinitions/read | 取得搜尋服務的可用記錄 |
> | 動作 | Microsoft.Search/searchServices/metricDefinitions/read | 取得搜尋服務的可用計量 |
> | 動作 | Microsoft.Search/searchServices/read | 讀取搜尋服務。 |
> | 動作 | Microsoft.Search/searchServices/regenerateAdminKey/action | 重新產生系統管理金鑰。 |
> | 動作 | Microsoft.Search/searchServices/start/action | 啟動搜尋服務。 |
> | 動作 | Microsoft.Search/searchServices/stop/action | 停止搜尋服務。 |
> | 動作 | Microsoft.Search/searchServices/write | 建立或更新搜尋服務。 |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Security/advancedThreatProtectionSettings/read | 取得資源的進階威脅防護設定 |
> | 動作 | Microsoft.Security/advancedThreatProtectionSettings/write | 更新資源的進階威脅防護設定 |
> | 動作 | Microsoft.Security/alerts/read | 取得所有可用的安全性警訊 |
> | 動作 | Microsoft.Security/applicationWhitelistings/read | 取得應用程式允許清單 |
> | 動作 | Microsoft.Security/applicationWhitelistings/write | 建立新的應用程式允許清單，或更新現有的應用程式允許清單 |
> | 動作 | Microsoft.Security/complianceResults/read | 取得資源的合規性結果 |
> | 動作 | Microsoft.Security/locations/alerts/activate/action | 啟動安全性警訊 |
> | 動作 | Microsoft.Security/locations/alerts/dismiss/action | 關閉安全性警示 |
> | 動作 | Microsoft.Security/locations/alerts/read | 取得所有可用的安全性警訊 |
> | 動作 | Microsoft.Security/locations/jitNetworkAccessPolicies/delete | 刪除 Just-in-Time 網路存取原則 |
> | 動作 | Microsoft.Security/locations/jitNetworkAccessPolicies/initiate/action | 起始 Just-in-Time 網路存取原則要求 |
> | 動作 | Microsoft.Security/locations/jitNetworkAccessPolicies/read | 取得 Just-in-Time 網路存取原則 |
> | 動作 | Microsoft.Security/locations/jitNetworkAccessPolicies/write | 建立新的 Just-in-Time 網路存取原則，或更新現有的 Just-in-Time 網路存取原則 |
> | 動作 | Microsoft.Security/locations/read | 取得安全性資料位置 |
> | 動作 | Microsoft.Security/locations/tasks/activate/action | 啟用安全性建議 |
> | 動作 | Microsoft.Security/locations/tasks/dismiss/action | 關閉安全性建議 |
> | 動作 | Microsoft.Security/locations/tasks/read | 取得所有可用的安全性建議 |
> | 動作 | Microsoft.Security/locations/tasks/resolve/action | 解析安全性建議 |
> | 動作 | Microsoft.Security/locations/tasks/start/action | 啟動安全性建議 |
> | 動作 | Microsoft.Security/policies/read | 取得安全性原則 |
> | 動作 | Microsoft.Security/policies/write | 更新安全性原則 |
> | 動作 | Microsoft.Security/pricings/delete | 刪除該範圍的定價設定 |
> | 動作 | Microsoft.Security/pricings/read | 取得該範圍的定價設定 |
> | 動作 | Microsoft.Security/pricings/write | 更新該範圍的定價設定 |
> | 動作 | Microsoft.Security/register/action | 為 Azure 資訊安全中心註冊訂用帳戶 |
> | 動作 | Microsoft.Security/securityContacts/delete | 刪除安全性連絡人 |
> | 動作 | Microsoft.Security/securityContacts/read | 取得安全性連絡人 |
> | 動作 | Microsoft.Security/securityContacts/write | 更新安全性連絡人 |
> | 動作 | Microsoft.Security/securitySolutions/delete | 刪除安全性解決方案 |
> | 動作 | Microsoft.Security/securitySolutions/read | 取得安全性解決方案 |
> | 動作 | Microsoft.Security/securitySolutions/write | 建立新的安全性解決方案，或更新現有的安全性解決方案 |
> | 動作 | Microsoft.Security/securitySolutionsReferenceData/read | 取得安全性解決方案參考資料 |
> | 動作 | Microsoft.Security/securityStatuses/read | 取得 Azure 資源的安全性健康狀態 |
> | 動作 | Microsoft.Security/securityStatusesSummaries/read | 取得該範圍的安全性狀態摘要 |
> | 動作 | Microsoft.Security/tasks/read | 取得所有可用的安全性建議 |
> | 動作 | Microsoft.Security/webApplicationFirewalls/delete | 刪除 Web 應用程式防火牆 |
> | 動作 | Microsoft.Security/webApplicationFirewalls/read | 取得 Web 應用程式防火牆 |
> | 動作 | Microsoft.Security/webApplicationFirewalls/write | 建立新的 Web 應用程式防火牆，或更新現有的 Web 應用程式防火牆 |
> | 動作 | Microsoft.Security/workspaceSettings/connect/action | 變更工作區設定的重新連線設定 |
> | 動作 | Microsoft.Security/workspaceSettings/delete | 刪除工作區設定 |
> | 動作 | Microsoft.Security/workspaceSettings/read | 取得工作區設定 |
> | 動作 | Microsoft.Security/workspaceSettings/write | 更新工作區設定 |

## <a name="microsoftsecuritygraph"></a>Microsoft.SecurityGraph

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.SecurityGraph/diagnosticsettings/delete | 正在刪除診斷設定 |
> | 動作 | Microsoft.SecurityGraph/diagnosticsettings/read | 正在讀取診斷設定 |
> | 動作 | Microsoft.SecurityGraph/diagnosticsettings/write | 正在寫入診斷設定 |
> | 動作 | Microsoft.SecurityGraph/diagnosticsettingscategories/read | 正在讀取診斷設定類別 |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ServiceBus/checkNameAvailability/action | 檢查給定訂用帳戶下的命名空間是否可用。 |
> | 動作 | Microsoft.ServiceBus/checkNamespaceAvailability/action | 檢查給定訂用帳戶下的命名空間是否可用。 此 API 即將淘汰。請改用 CheckNameAvailabiltiy。 |
> | 動作 | Microsoft.ServiceBus/namespaces/authorizationRules/action | 更新命名空間授權規則。 此 API 即將淘汰。 請改用 PUT 呼叫來更新命名空間授權規則。 API 版本 2017-04-01 不支援此作業。 |
> | 動作 | Microsoft.ServiceBus/namespaces/authorizationRules/delete | 刪除命名空間授權規則。 您無法刪除預設的命名空間授權規則。  |
> | 動作 | Microsoft.ServiceBus/namespaces/authorizationRules/listkeys/action | 取得命名空間的連接字串 |
> | 動作 | Microsoft.ServiceBus/namespaces/authorizationRules/read | 取得命名空間授權規則描述的清單。 |
> | 動作 | Microsoft.ServiceBus/namespaces/authorizationRules/regenerateKeys/action | 對資源重新產生主要或次要金鑰 |
> | 動作 | Microsoft.ServiceBus/namespaces/authorizationRules/write | 建立命名空間層級的授權規則，並更新其屬性。 您可以更新授權規則的存取權限、主要金鑰和次要金鑰。 |
> | 動作 | Microsoft.ServiceBus/namespaces/Delete | 刪除命名空間資源 |
> | 動作 | Microsoft.ServiceBus/namespaces/disasterRecoveryConfigs/authorizationRules/listkeys/action | 取得災害復原主要命名空間的授權規則金鑰 |
> | 動作 | Microsoft.ServiceBus/namespaces/disasterRecoveryConfigs/authorizationRules/read | 取得災害復原主要命名空間的授權規則 |
> | 動作 | Microsoft.ServiceBus/namespaces/disasterRecoveryConfigs/breakPairing/action | 停用災害復原，並停止將變更從主要命名空間複寫至次要命名空間。 |
> | 動作 | Microsoft.ServiceBus/namespaces/disasterrecoveryconfigs/checkNameAvailability/action | 檢查指定訂用帳戶下命名空間別名的可用性。 |
> | 動作 | Microsoft.ServiceBus/namespaces/disasterRecoveryConfigs/delete | 刪除與命名空間相關聯的災害復原設定。 此作業只能透過主要命名空間叫用。 |
> | 動作 | Microsoft.ServiceBus/namespaces/disasterRecoveryConfigs/failover/action | 叫用異地災害復原容錯移轉，並將命名空間別名重新設定為指向次要命名空間。 |
> | 動作 | Microsoft.ServiceBus/namespaces/disasterRecoveryConfigs/read | 取得與命名空間相關聯的災害復原設定。 |
> | 動作 | Microsoft.ServiceBus/namespaces/disasterRecoveryConfigs/write | 建立或更新與命名空間相關聯的災害復原設定。 |
> | 動作 | Microsoft.ServiceBus/namespaces/eventGridFilters/delete | 刪除與命名空間相關聯的事件格線篩選。 |
> | 動作 | Microsoft.ServiceBus/namespaces/eventGridFilters/read | 取得與命名空間相關聯的事件格線篩選。 |
> | 動作 | Microsoft.ServiceBus/namespaces/eventGridFilters/write | 建立或更新與命名空間相關聯的事件格線篩選。 |
> | 動作 | Microsoft.ServiceBus/namespaces/eventhubs/read | 取得 EventHub 資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/messagingPlan/read | 取得命名空間的傳訊方案。<br>此 API 即將淘汰。<br>透過 MessagingPlan 資源公開的屬性，已移至新版 API 的 (父系) 命名空間資源。<br>API 版本 2017-04-01 不支援此作業。 |
> | 動作 | Microsoft.ServiceBus/namespaces/messagingPlan/write | 更新命名空間的傳訊方案。<br>此 API 即將淘汰。<br>透過 MessagingPlan 資源公開的屬性，已移至新版 API 的 (父系) 命名空間資源。<br>API 版本 2017-04-01 不支援此作業。 |
> | 動作 | Microsoft.ServiceBus/namespaces/migrate/action | 移轉命名空間作業 |
> | 動作 | Microsoft.ServiceBus/namespaces/operationresults/read | 取得命名空間作業的狀態 |
> | 動作 | Microsoft.ServiceBus/namespaces/providers/Microsoft.Insights/diagnosticSettings/read | 取得命名空間診斷設定資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/providers/Microsoft.Insights/diagnosticSettings/write | 取得命名空間診斷設定資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/providers/Microsoft.Insights/logDefinitions/read | 取得命名空間記錄資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/providers/Microsoft.Insights/metricDefinitions/read | 取得命名空間計量資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/queues/authorizationRules/action | 更新佇列的作業。 API 版本 2017-04-01 不支援此作業。 授權規則。 請使用 PUT 呼叫來更新授權規則。 |
> | 動作 | Microsoft.ServiceBus/namespaces/queues/authorizationRules/delete | 用來刪除佇列授權規則的作業 |
> | 動作 | Microsoft.ServiceBus/namespaces/queues/authorizationRules/listkeys/action | 取得佇列的連接字串 |
> | 動作 | Microsoft.ServiceBus/namespaces/queues/authorizationRules/read |  取得佇列授權規則的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/queues/authorizationRules/regenerateKeys/action | 對資源重新產生主要或次要金鑰 |
> | 動作 | Microsoft.ServiceBus/namespaces/queues/authorizationRules/write | 建立佇列授權規則，並更新其屬性。 您可以更新授權規則存取權限。 |
> | 動作 | Microsoft.ServiceBus/namespaces/queues/Delete | 用來刪除佇列資源的作業 |
> | 動作 | Microsoft.ServiceBus/namespaces/queues/read | 取得佇列資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/queues/write | 建立或更新佇列屬性。 |
> | 動作 | Microsoft.ServiceBus/namespaces/read | 取得命名空間資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/removeAcsNamepsace/action | 移除 ACS 命名空間 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/authorizationRules/action | 更新主題的作業。 API 版本 2017-04-01 不支援此作業。 授權規則。 請使用 PUT 呼叫來更新授權規則。 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/authorizationRules/delete | 用來刪除主題授權規則的作業 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/authorizationRules/listkeys/action | 取得主題的連接字串 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/authorizationRules/read |  取得主題授權規則的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/authorizationRules/regenerateKeys/action | 對資源重新產生主要或次要金鑰 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/authorizationRules/write | 建立主題授權規則，並更新其屬性。 您可以更新授權規則存取權限。 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/Delete | 用來刪除主題資源的作業 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/read | 取得主題資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/subscriptions/Delete | 用來刪除 TopicSubscription 資源的作業 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/subscriptions/read | 取得 TopicSubscription 資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/subscriptions/rules/Delete | 用來刪除規則資源的作業 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/subscriptions/rules/read | 取得規則資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/subscriptions/rules/write | 建立或更新規則屬性。 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/subscriptions/write | 建立或更新 TopicSubscription 屬性。 |
> | 動作 | Microsoft.ServiceBus/namespaces/topics/write | 建立或更新主題屬性。 |
> | 動作 | Microsoft.ServiceBus/namespaces/write | 建立命名空間資源，並更新其屬性。 命名空間的標記和容量是可以更新的屬性。 |
> | 動作 | Microsoft.ServiceBus/operations/read | 取得作業 |
> | 動作 | Microsoft.ServiceBus/register/action | 針對 ServiceBus 資源提供者註冊訂用帳戶，並讓您能夠建立 ServiceBus 資源 |
> | 動作 | Microsoft.ServiceBus/sku/read | 取得 SKU 資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/sku/regions/read | 取得 SkuRegions 資源描述的清單 |
> | 動作 | Microsoft.ServiceBus/unregister/action | 針對 ServiceBus 資源提供者註冊訂用帳戶，並讓您能夠建立 ServiceBus 資源 |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.ServiceFabric/clusters/applications/delete | 刪除任何應用程式 |
> | 動作 | Microsoft.ServiceFabric/clusters/applications/read | 讀取任何應用程式 |
> | 動作 | Microsoft.ServiceFabric/clusters/applications/services/delete | 刪除任何服務 |
> | 動作 | Microsoft.ServiceFabric/clusters/applications/services/partitions/read | 讀取任何分割區 |
> | 動作 | Microsoft.ServiceFabric/clusters/applications/services/partitions/replicas/read | 讀取任何複本 |
> | 動作 | Microsoft.ServiceFabric/clusters/applications/services/read | 讀取任何服務 |
> | 動作 | Microsoft.ServiceFabric/clusters/applications/services/statuses/read | 讀取任何服務狀態 |
> | 動作 | Microsoft.ServiceFabric/clusters/applications/services/write | 建立或更新任何服務 |
> | 動作 | Microsoft.ServiceFabric/clusters/applications/write | 建立或更新任何應用程式 |
> | 動作 | Microsoft.ServiceFabric/clusters/applicationTypes/delete | 刪除任何應用程式類型 |
> | 動作 | Microsoft.ServiceFabric/clusters/applicationTypes/read | 讀取任何應用程式類型 |
> | 動作 | Microsoft.ServiceFabric/clusters/applicationTypes/versions/delete | 刪除任何應用程式類型版本 |
> | 動作 | Microsoft.ServiceFabric/clusters/applicationTypes/versions/read | 讀取任何應用程式類型版本 |
> | 動作 | Microsoft.ServiceFabric/clusters/applicationTypes/versions/write | 建立或更新任何應用程式類型版本 |
> | 動作 | Microsoft.ServiceFabric/clusters/applicationTypes/write | 建立或更新任何應用程式類型 |
> | 動作 | Microsoft.ServiceFabric/clusters/delete | 刪除任何叢集 |
> | 動作 | Microsoft.ServiceFabric/clusters/nodes/read | 讀取任何節點 |
> | 動作 | Microsoft.ServiceFabric/clusters/read | 讀取任何叢集 |
> | 動作 | Microsoft.ServiceFabric/clusters/statuses/read | 讀取任何叢集狀態 |
> | 動作 | Microsoft.ServiceFabric/clusters/write | 建立或更新任何叢集 |
> | 動作 | Microsoft.ServiceFabric/locations/clusterVersions/read | 讀取任何叢集版本 |
> | 動作 | Microsoft.ServiceFabric/locations/environments/clusterVersions/read | 讀取特定環境的任何叢集版本 |
> | 動作 | Microsoft.ServiceFabric/locations/operationresults/read | 讀取任何作業結果 |
> | 動作 | Microsoft.ServiceFabric/locations/operations/read | 依位置讀取任何作業 |
> | 動作 | Microsoft.ServiceFabric/operations/read | 讀取任何可用的作業 |
> | 動作 | Microsoft.ServiceFabric/register/action | 註冊任何動作 |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.SignalRService/checknameavailability/action | 檢查名稱是否可用於新的 SignalR 服務 |
> | 動作 | Microsoft.SignalRService/register/action | 向訂用帳戶註冊 'Microsoft.SignalRService' 資源提供者 |
> | 動作 | Microsoft.SignalRService/SignalR/delete | 刪除整個 SignalR |
> | 動作 | Microsoft.SignalRService/SignalR/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.SignalRService/SignalR/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.SignalRService/SignalR/providers/Microsoft.Insights/metricDefinitions/read | 取得 SignalR 的可用計量 |
> | 動作 | Microsoft.SignalRService/SignalR/read | 在管理入口網站中或透過 API 檢視 SignalR 的設定和組態 |
> | 動作 | Microsoft.SignalRService/SignalR/write | 在管理入口網站中或透過 API 修改 SignalR 的設定和組態 |
> | 動作 | Microsoft.SignalRService/unregister/action | 向訂用帳戶取消註冊 'Microsoft.SignalRService' 資源提供者 |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Solutions/applicationDefinitions/delete | 移除應用程式定義。 |
> | 動作 | Microsoft.Solutions/applicationDefinitions/read | 擷取應用程式定義清單。 |
> | 動作 | Microsoft.Solutions/applicationDefinitions/write | 新增或修改應用程式定義。 |
> | 動作 | Microsoft.Solutions/applications/delete | 移除應用程式。 |
> | 動作 | Microsoft.Solutions/applications/read | 擷取應用程式清單。 |
> | 動作 | Microsoft.Solutions/applications/write | 建立應用程式。 |
> | 動作 | Microsoft.Solutions/jitRequests/delete | 移除 JitRequest |
> | 動作 | Microsoft.Solutions/jitRequests/read | 擷取 JitRequests 清單 |
> | 動作 | Microsoft.Solutions/jitRequests/write | 建立 JitRequest |
> | 動作 | Microsoft.Solutions/locations/operationStatuses/read | 讀取資源的作業狀態。 |
> | 動作 | Microsoft.Solutions/register/action | 向 Solutions 註冊。 |

## <a name="microsoftsql"></a>Microsoft.Sql

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Sql/checkNameAvailability/action | 確認指定的伺服器名稱是否可用來針對指定的訂用帳戶進行全球佈建。 |
> | 動作 | Microsoft.Sql/locations/auditingSettingsAzureAsyncOperation/read | 擷取擴充伺服器的 Blob 稽核原則設定作業結果 |
> | 動作 | Microsoft.Sql/locations/auditingSettingsOperationResults/read | 擷取伺服器的「Blob 稽核原則設定」作業結果 |
> | 動作 | Microsoft.Sql/locations/capabilities/read | 在指定位置取得此訂用帳戶的功能 |
> | 動作 | Microsoft.Sql/locations/databaseAzureAsyncOperation/read | 取得資料庫作業的狀態。 |
> | 動作 | Microsoft.Sql/locations/databaseOperationResults/read | 取得資料庫作業的狀態。 |
> | 動作 | Microsoft.Sql/locations/deletedServerAsyncOperation/read | 取得已刪除伺服器上進行中的作業 |
> | 動作 | Microsoft.Sql/locations/deletedServerOperationResults/read | 取得已刪除伺服器上進行中的作業 |
> | 動作 | Microsoft.Sql/locations/deletedServers/read | 傳回已刪除伺服器的清單，或取得指定已刪除伺服器的屬性。 |
> | 動作 | Microsoft.Sql/locations/deletedServers/recover/action | 復原已刪除的伺服器 |
> | 動作 | Microsoft.Sql/locations/deleteVirtualNetworkOrSubnets/action | 刪除與虛擬網路或子網路相關聯的虛擬網路規則 |
> | 動作 | Microsoft.Sql/locations/elasticPoolAzureAsyncOperation/read | 取得彈性集區非同步作業的 Aure 非同步作業 |
> | 動作 | Microsoft.Sql/locations/elasticPoolOperationResults/read | 取得彈性集區作業的結果。 |
> | 動作 | Microsoft.Sql/locations/extendedAuditingSettingsAzureAsyncOperation/read | 擷取擴充伺服器的 Blob 稽核原則設定作業結果 |
> | 動作 | Microsoft.Sql/locations/extendedAuditingSettingsOperationResults/read | 擷取擴充伺服器的 Blob 稽核原則設定作業結果 |
> | 動作 | Microsoft.Sql/locations/instanceFailoverGroups/delete | 刪除現有的執行個體容錯移轉群組。 |
> | 動作 | Microsoft.Sql/locations/instanceFailoverGroups/failover/action | 在現有的執行個體容錯移轉群組中執行規劃的容錯移轉。 |
> | 動作 | Microsoft.Sql/locations/instanceFailoverGroups/forceFailoverAllowDataLoss/action | 在現有的執行個體容錯移轉群組中執行強制容錯移轉。 |
> | 動作 | Microsoft.Sql/locations/instanceFailoverGroups/read | 傳回執行個體容錯移轉群組的清單，或取得指定執行個體容錯移轉群組的屬性。 |
> | 動作 | Microsoft.Sql/locations/instanceFailoverGroups/write | 使用指定參數建立執行個體容錯移轉群組，或更新指定執行個體容錯移轉的屬性或標記。 |
> | 動作 | Microsoft.Sql/locations/interfaceEndpointProfileAzureAsyncOperation/read | 傳回特定介面端點 Azure 非同步作業的詳細資料 |
> | 動作 | Microsoft.Sql/locations/interfaceEndpointProfileOperationResults/read | 傳回特定介面端點 設定檔作業的詳細資料 |
> | 動作 | Microsoft.Sql/locations/longTermRetentionBackups/read | 列出位置中每個伺服器上每個資料庫的長期保留備份 |
> | 動作 | Microsoft.Sql/locations/longTermRetentionServers/longTermRetentionBackups/read | 列出伺服器上每個資料庫的長期保留備份 |
> | 動作 | Microsoft.Sql/locations/longTermRetentionServers/longTermRetentionDatabases/longTermRetentionBackups/delete | 刪除長期保留備份 |
> | 動作 | Microsoft.Sql/locations/longTermRetentionServers/longTermRetentionDatabases/longTermRetentionBackups/read | 列出資料庫的長期保留備份 |
> | 動作 | Microsoft.Sql/locations/managedDatabaseRestoreAzureAsyncOperation/completeRestore/action | 完成受控資料庫還原作業 |
> | 動作 | Microsoft.Sql/locations/managedTransparentDataEncryptionAzureAsyncOperation/read | 取得有關受控資料庫之透明資料加密的進行中作業 |
> | 動作 | Microsoft.Sql/locations/managedTransparentDataEncryptionOperationResults/read | 取得有關受控資料庫之透明資料加密的進行中作業 |
> | 動作 | Microsoft.Sql/locations/read | 取得指定訂用帳戶的可用位置 |
> | 動作 | Microsoft.Sql/locations/syncAgentOperationResults/read | 擷取同步代理程式資源作業的結果 |
> | 動作 | Microsoft.Sql/locations/syncDatabaseIds/read | 擷取特定區域和訂用帳戶的同步資料庫識別碼 |
> | 動作 | Microsoft.Sql/locations/syncGroupOperationResults/read | 擷取同步群組資源作業的結果 |
> | 動作 | Microsoft.Sql/locations/syncMemberOperationResults/read | 擷取同步成員資源作業的結果 |
> | 動作 | Microsoft.Sql/locations/usages/read | 在位置中取得此訂用帳戶的使用計量集合 |
> | 動作 | Microsoft.Sql/locations/virtualNetworkRulesAzureAsyncOperation/read | 傳回指定虛擬網路規則 Aure 非同步作業的詳細資料  |
> | 動作 | Microsoft.Sql/locations/virtualNetworkRulesOperationResults/read | 傳回指定虛擬網路規則作業的詳細資料  |
> | 動作 | Microsoft.Sql/managedInstances/administrators/delete | 刪除受控執行個體的現有系統管理員。 |
> | 動作 | Microsoft.Sql/managedInstances/administrators/read | 取得受控執行個體系統管理員的清單。 |
> | 動作 | Microsoft.Sql/managedInstances/administrators/write | 使用指定參數，建立或更新受控執行個體的系統管理員。 |
> | 動作 | Microsoft.Sql/managedInstances/databases/delete | 刪除現有的受控資料庫 |
> | 動作 | Microsoft.Sql/managedInstances/databases/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Sql/managedInstances/databases/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Sql/managedInstances/databases/providers/Microsoft.Insights/logDefinitions/read | 取得受控執行個體資料庫的可用記錄檔 |
> | 動作 | Microsoft.Sql/managedInstances/databases/read | 取得現有的受控資料庫 |
> | 動作 | Microsoft.Sql/managedInstances/databases/securityAlertPolicies/read | 擷取指定受控資料庫上所設定的資料庫威脅偵測原則詳細資料 |
> | 動作 | Microsoft.Sql/managedInstances/databases/securityAlertPolicies/write | 變更指定受控資料庫的資料庫威脅偵測原則 |
> | 動作 | Microsoft.Sql/managedInstances/databases/securityEvents/read | 擷取受控資料庫的安全性事件 |
> | 動作 | Microsoft.Sql/managedInstances/databases/transparentDataEncryption/read | 擷取指定受控資料庫上資料庫透明資料加密的詳細資料 |
> | 動作 | Microsoft.Sql/managedInstances/databases/transparentDataEncryption/write | 變更指定受控資料庫的資料庫透明資料加密 |
> | 動作 | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/delete | 移除指定資料庫的弱點評量 |
> | 動作 | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/read | 擷取指定資料庫上所設定之弱點評量的詳細資料 |
> | 動作 | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/rules/baselines/delete | 移除指定資料庫的弱點評量規則基準 |
> | 動作 | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/rules/baselines/read | 取得指定資料庫的弱點評量規則基準 |
> | 動作 | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/rules/baselines/write | 變更指定資料庫的弱點評量規則基準 |
> | 動作 | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/scans/export/action | 將現有的掃描結果轉換成人類可讀取的格式。 如果已存在，則不會執行任何動作 |
> | 動作 | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/scans/initiateScan/action | 執行弱點評估資料庫掃描。 |
> | 動作 | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/scans/read | 傳回資料庫弱點評量掃描記錄的清單，或取得指定掃描識別碼的掃描記錄。 |
> | 動作 | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/write | 變更指定資料庫的弱點評量 |
> | 動作 | Microsoft.Sql/managedInstances/databases/write | 建立新的資料庫或更新現有資料庫。 |
> | 動作 | Microsoft.Sql/managedInstances/delete | 刪除現有的受控執行個體。 |
> | 動作 | Microsoft.Sql/managedInstances/encryptionProtector/read | 傳回伺服器加密保護裝置的清單，或取得指定伺服器加密保護裝置的屬性。 |
> | 動作 | Microsoft.Sql/managedInstances/encryptionProtector/write | 更新指定伺服器加密保護裝置的屬性。 |
> | 動作 | Microsoft.Sql/managedInstances/keys/delete | 刪除現有的 Azure SQL 受控執行個體金鑰。 |
> | 動作 | Microsoft.Sql/managedInstances/keys/read | 傳回受控執行個體金鑰的清單，或取得指定受控執行個體金鑰的屬性。 |
> | 動作 | Microsoft.Sql/managedInstances/keys/write | 使用指定參數建立金鑰，或更新指定受控執行個體金鑰的屬性或標記。 |
> | 動作 | Microsoft.Sql/managedInstances/metricDefinitions/read | 取得受控執行個體的計量定義 |
> | 動作 | Microsoft.Sql/managedInstances/metrics/read | 取得受控執行個體計量 |
> | 動作 | Microsoft.Sql/managedInstances/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Sql/managedInstances/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Sql/managedInstances/providers/Microsoft.Insights/logDefinitions/read | 取得受控執行個體的可用記錄檔 |
> | 動作 | Microsoft.Sql/managedInstances/providers/Microsoft.Insights/metricDefinitions/read | 傳回可供受控執行個體使用的計量類型 |
> | 動作 | Microsoft.Sql/managedInstances/read | 傳回受控執行個體的清單，或取得指定受控執行個體的屬性。 |
> | 動作 | Microsoft.Sql/managedInstances/restorableDroppedDatabases/read | 傳回可還原的已卸除受控資料庫清單。 |
> | 動作 | Microsoft.Sql/managedInstances/securityAlertPolicies/read | 擷取指定受控伺服器上所設定的受控伺服器威脅偵測原則詳細資料 |
> | 動作 | Microsoft.Sql/managedInstances/securityAlertPolicies/write | 變更指定受控伺服器的受控伺服器威脅偵測原則 |
> | 動作 | Microsoft.Sql/managedInstances/tdeCertificates/action | 建立/更新 TDE 憑證 |
> | 動作 | Microsoft.Sql/managedInstances/vulnerabilityAssessments/delete | 移除指定受控執行個體的弱點評量 |
> | 動作 | Microsoft.Sql/managedInstances/vulnerabilityAssessments/read | 擷取指定受控執行個體上所設定之弱點評量的詳細資料 |
> | 動作 | Microsoft.Sql/managedInstances/vulnerabilityAssessments/write | 變更指定受控執行個體的弱點評量 |
> | 動作 | Microsoft.Sql/managedInstances/write | 使用指定參數建立受控執行個體，或更新指定受控執行個體的屬性或標記。 |
> | 動作 | Microsoft.Sql/operations/read | 取得可用的 REST 作業 |
> | 動作 | Microsoft.Sql/register/action | 為 Microsoft SQL Database 資源提供者註冊訂用帳戶，並讓您能夠建立 Microsoft SQL Database。 |
> | 動作 | Microsoft.Sql/servers/administratorOperationResults/read | 取得關於伺服器系統管理員的進行中作業 |
> | 動作 | Microsoft.Sql/servers/administrators/delete | 刪除伺服器系統管理員 |
> | 動作 | Microsoft.Sql/servers/administrators/read | 擷取伺服器管理員的詳細資料 |
> | 動作 | Microsoft.Sql/servers/administrators/write | 建立或更新伺服器管理員 |
> | 動作 | Microsoft.Sql/servers/advisors/read | 傳回伺服器可用 Advisor 的清單 |
> | 動作 | Microsoft.Sql/servers/advisors/recommendedActions/read | 傳回伺服器之指定 Advisor 的建議動作清單 |
> | 動作 | Microsoft.Sql/servers/advisors/recommendedActions/write | 對伺服器套用建議動作 |
> | 動作 | Microsoft.Sql/servers/advisors/write | 更新伺服器層級上 Advisor 的自動執行狀態。 |
> | 動作 | Microsoft.Sql/servers/auditingPolicies/read | 擷取給定伺服器上所設定之預設伺服器資料表稽核原則的詳細資料 |
> | 動作 | Microsoft.Sql/servers/auditingPolicies/write | 變更給定伺服器的預設伺服器資料表稽核 |
> | 動作 | Microsoft.Sql/servers/auditingSettings/operationResults/read | 擷取伺服器的「Blob 稽核原則設定」作業結果 |
> | 動作 | Microsoft.Sql/servers/auditingSettings/read | 擷取給定伺服器上所設定之伺服器 Blob 稽核原則的詳細資料 |
> | 動作 | Microsoft.Sql/servers/auditingSettings/write | 變更給定伺服器的伺服器 Blob 稽核 |
> | 動作 | Microsoft.Sql/servers/automaticTuning/read | 傳回伺服器的自動調整設定 |
> | 動作 | Microsoft.Sql/servers/automaticTuning/write | 更新伺服器的自動調整設定，並傳回更新的設定 |
> | 動作 | Microsoft.Sql/servers/backupLongTermRetentionVaults/delete | 刪除現有的備份封存保存庫。 |
> | 動作 | Microsoft.Sql/servers/backupLongTermRetentionVaults/read | 這項作業可用來取得備用的長期保留保存庫。 它會傳回向此伺服器註冊之保存庫的相關資訊 |
> | 動作 | Microsoft.Sql/servers/backupLongTermRetentionVaults/write | 此作業可用來向伺服器註冊備份的長期保留保存庫 |
> | 動作 | Microsoft.Sql/servers/communicationLinks/delete | 刪除現有的伺服器通訊連結。 |
> | 動作 | Microsoft.Sql/servers/communicationLinks/read | 傳回指定伺服器的通訊連結清單。 |
> | 動作 | Microsoft.Sql/servers/communicationLinks/write | 建立或更新伺服器通訊連結。 |
> | 動作 | Microsoft.Sql/servers/connectionPolicies/read | 傳回指定伺服器的伺服器連線原則清單。 |
> | 動作 | Microsoft.Sql/servers/connectionPolicies/write | 建立或更新伺服器連線原則。 |
> | 動作 | Microsoft.Sql/servers/databases/advisors/read | 傳回資料庫可用 Advisor 的清單 |
> | 動作 | Microsoft.Sql/servers/databases/advisors/recommendedActions/read | 傳回資料庫之指定 Advisor 的建議動作清單 |
> | 動作 | Microsoft.Sql/servers/databases/advisors/recommendedActions/write | 對資料庫套用建議動作 |
> | 動作 | Microsoft.Sql/servers/databases/advisors/write | 更新資料庫層級上 Advisor 的自動執行狀態。 |
> | 動作 | Microsoft.Sql/servers/databases/auditingPolicies/read | 擷取給定資料庫上所設定之資料表稽核原則的詳細資料 |
> | 動作 | Microsoft.Sql/servers/databases/auditingPolicies/write | 變更給定資料庫的資料表稽核原則 |
> | 動作 | Microsoft.Sql/servers/databases/auditingSettings/read | 擷取給定資料庫上所設定之 Blob 稽核原則的詳細資料 |
> | 動作 | Microsoft.Sql/servers/databases/auditingSettings/write | 變更給定資料庫的 Blob 稽核原則 |
> | 動作 | Microsoft.Sql/servers/databases/auditRecords/read | 擷取資料庫 Blob 稽核記錄 |
> | 動作 | Microsoft.Sql/servers/databases/automaticTuning/read | 傳回資料庫的自動調整設定 |
> | 動作 | Microsoft.Sql/servers/databases/automaticTuning/write | 更新資料庫的自動調整設定，並傳回更新的設定 |
> | 動作 | Microsoft.Sql/servers/databases/azureAsyncOperation/read | 取得資料庫作業的狀態。 |
> | 動作 | Microsoft.Sql/servers/databases/backupLongTermRetentionPolicies/read | 傳回指定資料庫的備份封存原則清單。 |
> | 動作 | Microsoft.Sql/servers/databases/backupLongTermRetentionPolicies/write | 建立或更新資料庫備份封存原則。 |
> | 動作 | Microsoft.Sql/servers/databases/connectionPolicies/read | 擷取給定資料庫上所設定的連線原則詳細資料 |
> | 動作 | Microsoft.Sql/servers/databases/connectionPolicies/write | 變更給定資料庫的連線原則 |
> | 動作 | Microsoft.Sql/servers/databases/dataMaskingPolicies/read | 傳回資料庫資料遮罩原則的清單。 |
> | 動作 | Microsoft.Sql/servers/databases/dataMaskingPolicies/rules/delete | 刪除指定資料庫的資料遮罩原則規則 |
> | 動作 | Microsoft.Sql/servers/databases/dataMaskingPolicies/rules/read | 擷取給定資料庫上所設定之資料遮罩原則規則的詳細資料 |
> | 動作 | Microsoft.Sql/servers/databases/dataMaskingPolicies/rules/write | 變更給定伺服器的資料遮罩原則規則 |
> | 動作 | Microsoft.Sql/servers/databases/dataMaskingPolicies/write | 變更給定伺服器的資料遮罩原則 |
> | 動作 | Microsoft.Sql/servers/databases/dataWarehouseQueries/dataWarehouseQuerySteps/read | 傳回所選步驟識別碼之資料倉儲查詢的分散式查詢步驟資訊 |
> | 動作 | Microsoft.Sql/servers/databases/dataWarehouseQueries/read | 傳回所選查詢識別碼的資料倉儲散發查詢資訊 |
> | 動作 | Microsoft.Sql/servers/databases/dataWarehouseUserActivities/read | 擷取 SQL 資料倉儲執行個體的使用者活動，其中包含執行中和暫止的查詢 |
> | 動作 | Microsoft.Sql/servers/databases/delete | 刪除現有的資料庫。 |
> | 動作 | Microsoft.Sql/servers/databases/export/action | 匯出 Azure SQL Database |
> | 動作 | Microsoft.Sql/servers/databases/extendedAuditingSettings/read | 擷取指定資料庫上所設定之擴充 Blob 稽核原則的詳細資料 |
> | 動作 | Microsoft.Sql/servers/databases/extendedAuditingSettings/write | 變更指定資料庫的擴充 Blob 稽核原則 |
> | 動作 | Microsoft.Sql/servers/databases/extensions/read | 取得資料庫的擴充功能集合。 |
> | 動作 | Microsoft.Sql/servers/databases/extensions/write | 變更指定資料庫的擴充功能 |
> | 動作 | Microsoft.Sql/servers/databases/geoBackupPolicies/read | 擷取指定資料庫的異地備份原則 |
> | 動作 | Microsoft.Sql/servers/databases/geoBackupPolicies/write | 建立或更新資料庫異地備份原則 |
> | 動作 | Microsoft.Sql/servers/databases/importExportOperationResults/read | 取得進行中的匯入/匯出作業 |
> | 動作 | Microsoft.Sql/servers/databases/maintenanceWindowOptions/read | 取得所選資料庫的可用維護時間範圍清單。 |
> | 動作 | Microsoft.Sql/servers/databases/maintenanceWindows/read | 取得所選資料庫的維護時間範圍設定。 |
> | 動作 | Microsoft.Sql/servers/databases/maintenanceWindows/write | 設定所選資料庫的維護時間範圍設定。 |
> | 動作 | Microsoft.Sql/servers/databases/metricDefinitions/read | 傳回可供資料庫使用之計量的類型 |
> | 動作 | Microsoft.Sql/servers/databases/metrics/read | 傳回資料庫的計量 |
> | 動作 | Microsoft.Sql/servers/databases/move/action | 為 Azure SQL Database 重新命名 |
> | 動作 | Microsoft.Sql/servers/databases/operationResults/read | 取得資料庫作業的狀態。 |
> | 動作 | Microsoft.Sql/servers/databases/operations/cancel/action | 取消尚未完成的 Azure SQL Database 暫止非同步作業。 |
> | 動作 | Microsoft.Sql/servers/databases/operations/read | 傳回在資料庫上執行的作業清單 |
> | 動作 | Microsoft.Sql/servers/databases/pause/action | 暫停 Azure SQL 資料倉儲資料庫 |
> | 動作 | Microsoft.Sql/servers/databases/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Sql/servers/databases/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Sql/servers/databases/providers/Microsoft.Insights/logDefinitions/read | 取得資料庫的可用記錄 |
> | 動作 | Microsoft.Sql/servers/databases/providers/Microsoft.Insights/metricDefinitions/read | 傳回可供資料庫使用之計量的類型 |
> | 動作 | Microsoft.Sql/servers/databases/queryStore/queryTexts/read | 傳回對應至指定參數的查詢文字集合。 |
> | 動作 | Microsoft.Sql/servers/databases/queryStore/read | 傳回資料庫之查詢存放區設定目前的值。 |
> | 動作 | Microsoft.Sql/servers/databases/queryStore/write | 更新資料庫的查詢存放區設定 |
> | 動作 | Microsoft.Sql/servers/databases/read | 傳回資料庫清單，或取得指定資料庫的屬性。 |
> | 動作 | Microsoft.Sql/servers/databases/replicationLinks/delete | 強制終止複寫關係，但可能遺失資料 |
> | 動作 | Microsoft.Sql/servers/databases/replicationLinks/failover/action | 從主要資料庫同步處理所有變更後進行容錯移轉，讓此資料庫變成複寫關係的主要資料庫，並讓遠端的主要資料庫變成次要資料庫 |
> | 動作 | Microsoft.Sql/servers/databases/replicationLinks/forceFailoverAllowDataLoss/action | 立即進行容錯移轉 (但可能遺失資料)，讓此資料庫變成複寫關係的主要資料庫，並讓遠端的主要資料庫變成次要資料庫 |
> | 動作 | Microsoft.Sql/servers/databases/replicationLinks/read | 傳回複寫連結的清單，或取得指定複寫連結的屬性。 |
> | 動作 | Microsoft.Sql/servers/databases/replicationLinks/unlink/action | 強制終止複寫關係，或在與合作夥伴進行同步處理之後終止 |
> | 動作 | Microsoft.Sql/servers/databases/replicationLinks/updateReplicationMode/action | 將連結的複寫模式更新為同步或非同步模式 |
> | 動作 | Microsoft.Sql/servers/databases/restorePoints/action | 建立新的還原點 |
> | 動作 | Microsoft.Sql/servers/databases/restorePoints/delete | 刪除資料庫的還原點。 |
> | 動作 | Microsoft.Sql/servers/databases/restorePoints/read | 傳回資料庫的還原點。 |
> | 動作 | Microsoft.Sql/servers/databases/resume/action | 繼續 Azure SQL 資料倉儲資料庫 |
> | 動作 | Microsoft.Sql/servers/databases/schemas/read | 擷取資料庫的結構描述清單 |
> | 動作 | Microsoft.Sql/servers/databases/schemas/tables/columns/read | 擷取資料表的資料行清單 |
> | 動作 | Microsoft.Sql/servers/databases/schemas/tables/columns/sensitivityLabels/delete | 刪除指定資料行的敏感度標籤 |
> | 動作 | Microsoft.Sql/servers/databases/schemas/tables/columns/sensitivityLabels/read | 取得指定資料行的敏感度標籤 |
> | 動作 | Microsoft.Sql/servers/databases/schemas/tables/columns/sensitivityLabels/write | 建立或更新指定資料行的敏感度標籤 |
> | 動作 | Microsoft.Sql/servers/databases/schemas/tables/read | 擷取資料庫的資料表清單 |
> | 動作 | Microsoft.Sql/servers/databases/schemas/tables/recommendedIndexes/read | 擷取資料庫上的索引建議清單 |
> | 動作 | Microsoft.Sql/servers/databases/schemas/tables/recommendedIndexes/write | 套用索引建議 |
> | 動作 | Microsoft.Sql/servers/databases/securityAlertPolicies/read | 擷取給定資料庫上所設定的威脅偵測原則詳細資料 |
> | 動作 | Microsoft.Sql/servers/databases/securityAlertPolicies/write | 變更給定資料庫的威脅偵測原則 |
> | 動作 | Microsoft.Sql/servers/databases/securityMetrics/read | 取得資料庫安全性計量的集合 |
> | 動作 | Microsoft.Sql/servers/databases/sensitivityLabels/read | 列出指定資料庫的敏感度標籤 |
> | 動作 | Microsoft.Sql/servers/databases/serviceTierAdvisors/read | 傳回相關建議，以讓您根據查詢執行統計資料來相應增加或減少資料庫，以提升效能或降低成本 |
> | 動作 | Microsoft.Sql/servers/databases/skus/read | 取得可供資料庫使用的 SKU 集合 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/cancelSync/action | 取消同步群組同步處理 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/delete | 刪除現有的同步群組。 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/hubSchemas/read | 傳回同步中樞資料庫結構描述的清單 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/logs/read | 傳回同步群組記錄的清單 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/read | 傳回同步群組的清單，或取得指定同步群組的屬性。 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/refreshHubSchema/action | 重新整理同步中樞資料庫結構描述 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/refreshHubSchemaOperationResults/read | 擷取同步中樞結構描述重新整理作業的結果 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/syncMembers/delete | 刪除現有的同步成員。 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/syncMembers/read | 傳回同步成員的清單，或取得指定同步成員的屬性。 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/syncMembers/refreshSchema/action | 重新整理同步成員結構描述 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/syncMembers/refreshSchemaOperationResults/read | 擷取同步成員結構描述重新整理作業的結果 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/syncMembers/schemas/read | 傳回同步成員資料庫結構描述的清單 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/syncMembers/write | 使用指定參數建立同步成員，或更新指定同步成員的屬性。 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/triggerSync/action | 觸發同步群組同步處理 |
> | 動作 | Microsoft.Sql/servers/databases/syncGroups/write | 使用指定參數建立同步群組，或更新指定同步群組的屬性。 |
> | 動作 | Microsoft.Sql/servers/databases/topQueries/queryText/action | 傳回所選查詢識別碼的 Transact-SQL 文字 |
> | 動作 | Microsoft.Sql/servers/databases/topQueries/read | 傳回所選查詢在所選時段內彙總的執行階段統計資料 |
> | 動作 | Microsoft.Sql/servers/databases/topQueries/statistics/read | 傳回所選查詢在所選時段內彙總的執行階段統計資料 |
> | 動作 | Microsoft.Sql/servers/databases/transparentDataEncryption/operationResults/read | 取得有關透明資料加密的進行中作業 |
> | 動作 | Microsoft.Sql/servers/databases/transparentDataEncryption/read | 擷取給定資料庫之透明資料加密安全性功能的狀態和詳細資料 |
> | 動作 | Microsoft.Sql/servers/databases/transparentDataEncryption/write | 變更透明資料加密狀態 |
> | 動作 | Microsoft.Sql/servers/databases/upgradeDataWarehouse/action | 將 Azure SQL 資料倉儲資料庫升級 |
> | 動作 | Microsoft.Sql/servers/databases/usages/read | 取得 Azure SQL Database 的使用方式資訊 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessments/delete | 移除指定資料庫的弱點評量 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessments/read | 擷取指定資料庫上所設定之弱點評量的詳細資料 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessments/rules/baselines/delete | 移除指定資料庫的弱點評量規則基準 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessments/rules/baselines/read | 取得指定資料庫的弱點評量規則基準 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessments/rules/baselines/write | 變更指定資料庫的弱點評量規則基準 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessments/scans/export/action | 將現有的掃描結果轉換成人類可讀取的格式。 如果已存在，則不會執行任何動作 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessments/scans/initiateScan/action | 執行弱點評估資料庫掃描。 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessments/scans/read | 傳回資料庫弱點評量掃描記錄的清單，或取得指定掃描識別碼的掃描記錄。 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessments/write | 變更指定資料庫的弱點評量 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessmentScans/action | 執行弱點評估資料庫掃描。 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessmentScans/operationResults/read | 擷取資料庫弱點評量掃描執行作業的結果 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessmentSettings/read | 擷取指定資料庫上所設定之弱點評量的詳細資料 |
> | 動作 | Microsoft.Sql/servers/databases/vulnerabilityAssessmentSettings/write | 變更指定資料庫的弱點評量 |
> | 動作 | Microsoft.Sql/servers/databases/write | 使用指定參數建立資料庫，或更新指定資料庫的屬性或標記。 |
> | 動作 | Microsoft.Sql/servers/delete | 刪除現有伺服器。 |
> | 動作 | Microsoft.Sql/servers/disasterRecoveryConfiguration/delete | 刪除指定伺服器的現有災害復原設定 |
> | 動作 | Microsoft.Sql/servers/disasterRecoveryConfiguration/failover/action | 進行 DisasterRecoveryConfiguration 的容錯移轉 |
> | 動作 | Microsoft.Sql/servers/disasterRecoveryConfiguration/forceFailoverAllowDataLoss/action | 強制進行 DisasterRecoveryConfiguration 的容錯移轉 |
> | 動作 | Microsoft.Sql/servers/disasterRecoveryConfiguration/read | 取得包括此伺服器在內的災害復原設定集合 |
> | 動作 | Microsoft.Sql/servers/disasterRecoveryConfiguration/write | 變更伺服器的災害復原設定 |
> | 動作 | Microsoft.Sql/servers/elasticPoolEstimates/read | 傳回已針對此伺服器建立之彈性集區估計值的清單 |
> | 動作 | Microsoft.Sql/servers/elasticPoolEstimates/write | 針對所提供的資料庫清單建立新的彈性集區估計值 |
> | 動作 | Microsoft.Sql/servers/elasticPools/advisors/read | 傳回彈性集區可用 Advisor 的清單 |
> | 動作 | Microsoft.Sql/servers/elasticPools/advisors/recommendedActions/read | 傳回彈性集區之指定 Advisor 的建議動作清單 |
> | 動作 | Microsoft.Sql/servers/elasticPools/advisors/recommendedActions/write | 對彈性集區套用建議動作 |
> | 動作 | Microsoft.Sql/servers/elasticPools/advisors/write | 更新彈性集區層級上 Advisor 的自動執行狀態。 |
> | 動作 | Microsoft.Sql/servers/elasticPools/databases/read | 取得彈性集區的資料庫清單 |
> | 動作 | Microsoft.Sql/servers/elasticPools/delete | 刪除現有彈性集區 |
> | 動作 | Microsoft.Sql/servers/elasticPools/elasticPoolActivity/read | 擷取給定彈性資料庫集區的活動和詳細資料 |
> | 動作 | Microsoft.Sql/servers/elasticPools/elasticPoolDatabaseActivity/read | 擷取屬於彈性資料庫集區之給定資料庫的活動和詳細資料 |
> | 動作 | Microsoft.Sql/servers/elasticPools/metricDefinitions/read | 傳回可供彈性資料庫集區使用之計量的類型 |
> | 動作 | Microsoft.Sql/servers/elasticPools/metrics/read | 傳回彈性資料庫集區的計量 |
> | 動作 | Microsoft.Sql/servers/elasticPools/operations/cancel/action | 取消尚未完成的 Azure SQL 彈性集區暫止非同步作業。 |
> | 動作 | Microsoft.Sql/servers/elasticPools/operations/read | 傳回在彈性集區上執行的作業清單 |
> | 動作 | Microsoft.Sql/servers/elasticPools/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.Sql/servers/elasticPools/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Sql/servers/elasticPools/providers/Microsoft.Insights/metricDefinitions/read | 傳回可供彈性資料庫集區使用之計量的類型 |
> | 動作 | Microsoft.Sql/servers/elasticPools/read | 擷取指定伺服器上彈性集區的詳細資料 |
> | 動作 | Microsoft.Sql/servers/elasticPools/skus/read | 取得可供彈性集區使用的 SKU 集合 |
> | 動作 | Microsoft.Sql/servers/elasticPools/write | 建立新的彈性集區，或變更現有彈性集區的屬性 |
> | 動作 | Microsoft.Sql/servers/encryptionProtector/read | 傳回伺服器加密保護裝置的清單，或取得指定伺服器加密保護裝置的屬性。 |
> | 動作 | Microsoft.Sql/servers/encryptionProtector/write | 更新指定伺服器加密保護裝置的屬性。 |
> | 動作 | Microsoft.Sql/servers/extendedAuditingSettings/read | 擷取指定伺服器上所設定之擴充伺服器 Blob 稽核原則的詳細資料 |
> | 動作 | Microsoft.Sql/servers/extendedAuditingSettings/write | 變更指定伺服器的擴充伺服器 Blob 稽核 |
> | 動作 | Microsoft.Sql/servers/failoverGroups/delete | 刪除現有的容錯移轉群組。 |
> | 動作 | Microsoft.Sql/servers/failoverGroups/failover/action | 在現有的容錯移轉群組中執行規劃的容錯移轉。 |
> | 動作 | Microsoft.Sql/servers/failoverGroups/forceFailoverAllowDataLoss/action | 在現有的容錯移轉群組中執行強制容錯移轉。 |
> | 動作 | Microsoft.Sql/servers/failoverGroups/read | 傳回容錯移轉群組的清單，或取得指定容錯移轉群組的屬性。 |
> | 動作 | Microsoft.Sql/servers/failoverGroups/write | 使用指定參數建立容錯移轉群組，或更新指定容錯移轉的屬性或標記。 |
> | 動作 | Microsoft.Sql/servers/firewallRules/delete | 刪除現有的伺服器防火牆規則。 |
> | 動作 | Microsoft.Sql/servers/firewallRules/read | 傳回伺服器防火牆規則的清單，或取得指定伺服器防火牆規則的屬性。 |
> | 動作 | Microsoft.Sql/servers/firewallRules/write | 使用指定參數建立伺服器防火牆規則、更新指定規則的屬性，或使用新的伺服器防火牆規則覆寫所有現有的規則。 |
> | 動作 | Microsoft.Sql/servers/import/action | 在伺服器上建立新的資料庫，並部署來自 DacPac 套件的結構描述和資料 |
> | 動作 | Microsoft.Sql/servers/importExportOperationResults/read | 取得進行中的匯入/匯出作業 |
> | 動作 | Microsoft.Sql/servers/interfaceEndpointProfiles/delete | 刪除指定的介面端點設定檔 |
> | 動作 | Microsoft.Sql/servers/interfaceEndpointProfiles/read | 傳回指定介面端點設定檔的屬性 |
> | 動作 | Microsoft.Sql/servers/interfaceEndpointProfiles/write | 使用指定參數建立介面端點設定檔，或更新指定介面端點的屬性或標記 |
> | 動作 | Microsoft.Sql/servers/keys/delete | 刪除現有的伺服器金鑰。 |
> | 動作 | Microsoft.Sql/servers/keys/read | 傳回伺服器金鑰的清單，或取得指定伺服器金鑰的屬性。 |
> | 動作 | Microsoft.Sql/servers/keys/write | 使用指定參數建立金鑰，或更新指定伺服器金鑰的屬性或標記。 |
> | 動作 | Microsoft.Sql/servers/operationResults/read | 取得進行中的伺服器作業 |
> | 動作 | Microsoft.Sql/servers/providers/Microsoft.Insights/metricDefinitions/read | 傳回可供伺服器使用的計量類型 |
> | 動作 | Microsoft.Sql/servers/read | 傳回伺服器清單，或取得指定伺服器的屬性。 |
> | 動作 | Microsoft.Sql/servers/recommendedElasticPools/databases/read | 擷取給定伺服器之建議彈性資料庫集區的計量 |
> | 動作 | Microsoft.Sql/servers/recommendedElasticPools/read | 擷取彈性資料庫集區的建議，以根據資源的歷史使用量來降低成本或改善效能 |
> | 動作 | Microsoft.Sql/servers/recoverableDatabases/read | 這項作業可用於即時資料庫的災害復原，以將資料庫還原至最後一個已知良好的備份點。 它會傳回最後一個良好備份的相關資訊，但它實際上並不會還原資料庫。 |
> | 動作 | Microsoft.Sql/servers/replicationLinks/read | 傳回複寫連結的清單，或取得指定複寫連結的屬性。 |
> | 動作 | Microsoft.Sql/servers/restorableDroppedDatabases/read | 取得指定伺服器上雖已捨棄但仍留在保留原則內的資料庫清單。 |
> | 動作 | Microsoft.Sql/servers/securityAlertPolicies/operationResults/read | 擷取伺服器威脅偵測原則寫入作業的結果 |
> | 動作 | Microsoft.Sql/servers/securityAlertPolicies/read | 擷取給定伺服器上所設定的伺服器威脅偵測原則詳細資料 |
> | 動作 | Microsoft.Sql/servers/securityAlertPolicies/write | 變更指定伺服器的伺服器威脅偵測原則 |
> | 動作 | Microsoft.Sql/servers/serviceObjectives/read | 擷取給定伺服器上可用的服務等級目標 (也稱為效能層級) 清單 |
> | 動作 | Microsoft.Sql/servers/syncAgents/delete | 刪除現有的同步代理程式。 |
> | 動作 | Microsoft.Sql/servers/syncAgents/generateKey/action | 產生同步代理程式登錄金鑰 |
> | 動作 | Microsoft.Sql/servers/syncAgents/linkedDatabases/read | 傳回已連結同步代理程式的資料庫清單 |
> | 動作 | Microsoft.Sql/servers/syncAgents/read | 傳回同步代理程式的清單，或取得指定同步代理程式的屬性。 |
> | 動作 | Microsoft.Sql/servers/syncAgents/write | 使用指定參數建立同步代理程式，或更新指定同步代理程式的屬性。 |
> | 動作 | Microsoft.Sql/servers/tdeCertificates/action | 建立/更新 TDE 憑證 |
> | 動作 | Microsoft.Sql/servers/usages/read | 傳回伺服器內所有資料庫的伺服器 DTU 配額和目前的 DTU 耗用量 |
> | 動作 | Microsoft.Sql/servers/virtualNetworkRules/delete | 刪除現有虛擬網路規則 |
> | 動作 | Microsoft.Sql/servers/virtualNetworkRules/read | 傳回虛擬網路規則的清單，或取得指定虛擬網路規則的屬性。 |
> | 動作 | Microsoft.Sql/servers/virtualNetworkRules/write | 使用指定參數建立虛擬網路規則，或更新指定虛擬網路規則的屬性或標記。 |
> | 動作 | Microsoft.Sql/servers/vulnerabilityAssessments/delete | 移除指定伺服器的弱點評量 |
> | 動作 | Microsoft.Sql/servers/vulnerabilityAssessments/read | 擷取指定伺服器上所設定之弱點評量的詳細資料 |
> | 動作 | Microsoft.Sql/servers/vulnerabilityAssessments/write | 變更指定伺服器的弱點評量 |
> | 動作 | Microsoft.Sql/servers/write | 使用指定參數建立伺服器，或更新指定伺服器的屬性或標記。 |
> | 動作 | Microsoft.Sql/unregister/action | 為 Microsoft SQL Database 資源提供者取消註冊訂用帳戶，並讓您能夠建立 Microsoft SQL Database。 |
> | 動作 | Microsoft.Sql/virtualClusters/read | 傳回虛擬叢集清單，或取得指定虛擬叢集的屬性。 |
> | 動作 | Microsoft.Sql/virtualClusters/write | 更新虛擬叢集標記。 |

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Storage/checknameavailability/read | 確認帳戶名稱有效，且並非使用中。 |
> | 動作 | Microsoft.Storage/locations/deleteVirtualNetworkOrSubnets/action | 讓 Microsoft.Storage 知道虛擬網路或子網路即將刪除 |
> | 動作 | Microsoft.Storage/locations/usages/read | 傳回指定訂用帳戶資源的限制和目前的使用量計數 |
> | 動作 | Microsoft.Storage/operations/read | 輪詢非同步作業的狀態。 |
> | 動作 | Microsoft.Storage/register/action | 針對儲存體資源提供者註冊訂用帳戶，並讓您能夠建立儲存體帳戶。 |
> | 動作 | Microsoft.Storage/skus/read | 列出 Microsoft.Storage 所支援的 SKU。 |
> | DataAction | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action | 傳回新增 Blob 內容的結果 |
> | DataAction | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete | 傳回刪除 Blob 的結果 |
> | DataAction | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/deleteAutomaticSnapshot/action | 傳回刪除自動快照集的結果 |
> | DataAction | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read | 傳回 Blob 或 Blob 清單 |
> | DataAction | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write | 傳回寫入 Blob 的結果 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/containers/clearLegalHold/action | 清除 Blob 容器法務保存措施 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/containers/delete | 傳回刪除容器的結果 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/containers/immutabilityPolicies/delete | 刪除 Blob 容器不變性原則 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/containers/immutabilityPolicies/extend/action | 擴充 Blob 容器不變性原則 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/containers/immutabilityPolicies/lock/action | 鎖定 Blob 容器不變性原則 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/containers/immutabilityPolicies/read | 取得 Blob 容器不變性原則 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/containers/immutabilityPolicies/write | 放置 Blob 容器不變性原則 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/containers/read | 傳回容器或容器清單 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/containers/setLegalHold/action | 設定 Blob 容器法務保存措施 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/containers/write | 傳回放置或租用 Blob 容器的結果 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey/action | 傳回 Blob 服務的使用者委派金鑰 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/providers/Microsoft.Insights/logDefinitions/read | 取得 Blob 的記錄定義 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/providers/Microsoft.Insights/metricDefinitions/read | 取得 Microsoft 儲存體計量定義的清單。 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/read | 傳回 Blob 服務屬性或統計資料 |
> | 動作 | Microsoft.Storage/storageAccounts/blobServices/write | 傳回放置 Blob 服務屬性的結果 |
> | 動作 | Microsoft.Storage/storageAccounts/delete | 刪除現有的儲存體帳戶。 |
> | 動作 | Microsoft.Storage/storageAccounts/fileServices/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/fileServices/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/fileServices/providers/Microsoft.Insights/logDefinitions/read | 取得檔案的記錄定義 |
> | 動作 | Microsoft.Storage/storageAccounts/fileServices/providers/Microsoft.Insights/metricDefinitions/read | 取得 Microsoft 儲存體計量定義的清單。 |
> | 動作 | Microsoft.Storage/storageAccounts/lastsynctime/read | 傳回儲存體帳戶的上次同步時間 |
> | 動作 | Microsoft.Storage/storageAccounts/listAccountSas/action | 傳回指定儲存體帳戶的帳戶 SAS 權杖。 |
> | 動作 | Microsoft.Storage/storageAccounts/listkeys/action | 傳回指定儲存體帳戶的存取金鑰。 |
> | 動作 | Microsoft.Storage/storageAccounts/listServiceSas/action | 傳回指定儲存體帳戶的服務 SAS 權杖。 |
> | 動作 | Microsoft.Storage/storageAccounts/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/providers/Microsoft.Insights/metricDefinitions/read | 取得 Microsoft 儲存體計量定義的清單。 |
> | 動作 | Microsoft.Storage/storageAccounts/queueServices/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/queueServices/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/queueServices/providers/Microsoft.Insights/logDefinitions/read | 取得佇列的記錄定義 |
> | 動作 | Microsoft.Storage/storageAccounts/queueServices/providers/Microsoft.Insights/metricDefinitions/read | 取得 Microsoft 儲存體計量定義的清單。 |
> | 動作 | Microsoft.Storage/storageAccounts/queueServices/queues/delete | 傳回刪除佇列的結果 |
> | DataAction | Microsoft.Storage/storageAccounts/queueServices/queues/messages/add/action | 傳回新增訊息的結果 |
> | DataAction | Microsoft.Storage/storageAccounts/queueServices/queues/messages/delete | 傳回刪除訊息的結果 |
> | DataAction | Microsoft.Storage/storageAccounts/queueServices/queues/messages/process/action | 傳回處理訊息的結果 |
> | DataAction | Microsoft.Storage/storageAccounts/queueServices/queues/messages/read | 傳回訊息 |
> | DataAction | Microsoft.Storage/storageAccounts/queueServices/queues/messages/write | 傳回寫入訊息的結果 |
> | 動作 | Microsoft.Storage/storageAccounts/queueServices/queues/read | 傳回佇列或佇列清單。 |
> | 動作 | Microsoft.Storage/storageAccounts/queueServices/queues/write | 傳回寫入佇列的結果 |
> | 動作 | Microsoft.Storage/storageAccounts/queueServices/read | 傳回佇列服務屬性或統計資料。 |
> | 動作 | Microsoft.Storage/storageAccounts/queueServices/write | 傳回設定佇列服務屬性的結果 |
> | 動作 | Microsoft.Storage/storageAccounts/read | 傳回儲存體帳戶清單，或取得指定儲存體帳戶的屬性。 |
> | 動作 | Microsoft.Storage/storageAccounts/regeneratekey/action | 重新產生指定儲存體帳戶的存取金鑰。 |
> | 動作 | Microsoft.Storage/storageAccounts/revokeUserDelegationKeys/action | 撤銷所指定儲存體帳戶的所有使用者委派金鑰。 |
> | 動作 | Microsoft.Storage/storageAccounts/services/diagnosticSettings/write | 建立/更新儲存體帳戶的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/tableServices/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/tableServices/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定。 |
> | 動作 | Microsoft.Storage/storageAccounts/tableServices/providers/Microsoft.Insights/logDefinitions/read | 取得資料表的記錄定義 |
> | 動作 | Microsoft.Storage/storageAccounts/tableServices/providers/Microsoft.Insights/metricDefinitions/read | 取得 Microsoft 儲存體計量定義的清單。 |
> | 動作 | Microsoft.Storage/storageAccounts/write | 使用指定參數來建立儲存體帳戶、更新指定儲存體帳戶的屬性或標記，或新增指定儲存體帳戶的自訂網域。 |
> | 動作 | Microsoft.Storage/usages/read | 傳回指定訂用帳戶資源的限制和目前的使用量計數 |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | microsoft.storagesync/locations/checkNameAvailability/action | 確認儲存體同步服務名稱有效，且並非使用中。 |
> | 動作 | microsoft.storagesync/locations/workflows/operations/read | 取得非同步作業的狀態 |
> | 動作 | microsoft.storagesync/storageSyncServices/delete | 刪除任何儲存體同步服務 |
> | 動作 | microsoft.storagesync/storageSyncServices/providers/Microsoft.Insights/metricDefinitions/read | 取得儲存體同步服務的可用計量 |
> | 動作 | microsoft.storagesync/storageSyncServices/read | 讀取任何儲存體同步服務 |
> | 動作 | microsoft.storagesync/storageSyncServices/registeredServers/delete | 刪除任何已註冊的伺服器 |
> | 動作 | microsoft.storagesync/storageSyncServices/registeredServers/providers/Microsoft.Insights/metricDefinitions/read | 取得已註冊的伺服器可用的計量 |
> | 動作 | microsoft.storagesync/storageSyncServices/registeredServers/read | 讀取任何已註冊的伺服器 |
> | 動作 | microsoft.storagesync/storageSyncServices/registeredServers/write | 建立或更新任何已註冊的伺服器 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/cloudEndpoints/delete | 刪除任何雲端端點 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/cloudEndpoints/operationresults/read | 取得非同步備份/還原作業的狀態 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/cloudEndpoints/postbackup/action | 在備份之後呼叫此動作 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/cloudEndpoints/postrestore/action | 在還原之後呼叫此動作 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/cloudEndpoints/prebackup/action | 在備份之前呼叫此動作 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/cloudEndpoints/prerestore/action | 在還原之前呼叫此動作 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/cloudEndpoints/read | 讀取任何雲端端點 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/cloudEndpoints/restoreheartbeat/action | 還原活動訊號 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/cloudEndpoints/write | 建立或更新任何雲端端點 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/delete | 刪除任何同步群組 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/providers/Microsoft.Insights/metricDefinitions/read | 取得同步群組的可用計量 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/read | 讀取任何同步群組 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/serverEndpoints/delete | 刪除任何伺服器端點 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/serverEndpoints/providers/Microsoft.Insights/metricDefinitions/read | 取得伺服器端點的可用計量 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/serverEndpoints/read | 讀取任何伺服器端點 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/serverEndpoints/recallAction/action | 呼叫此動作以將檔案回復到伺服器 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/serverEndpoints/write | 建立或更新任何伺服器端點 |
> | 動作 | microsoft.storagesync/storageSyncServices/syncGroups/write | 建立或更新任何同步群組 |
> | 動作 | microsoft.storagesync/storageSyncServices/workflows/operationresults/read | 取得非同步作業的狀態 |
> | 動作 | microsoft.storagesync/storageSyncServices/workflows/operations/read | 取得非同步作業的狀態 |
> | 動作 | microsoft.storagesync/storageSyncServices/workflows/read | 讀取工作流程 |
> | 動作 | microsoft.storagesync/storageSyncServices/write | 建立或更新任何儲存體同步服務 |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.StorSimple/managers/accessControlRecords/delete | 刪除存取控制記錄 |
> | 動作 | Microsoft.StorSimple/managers/accessControlRecords/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/accessControlRecords/read | 列出或取得存取控制記錄 |
> | 動作 | Microsoft.StorSimple/managers/accessControlRecords/write | 建立或更新存取控制記錄 |
> | 動作 | Microsoft.StorSimple/managers/alerts/read | 列出或取得警示 |
> | 動作 | Microsoft.StorSimple/managers/backups/read | 列出或取得備份組 |
> | 動作 | Microsoft.StorSimple/managers/bandwidthSettings/delete | 刪除現有頻寬設定 (僅限 8000 系列) |
> | 動作 | Microsoft.StorSimple/managers/bandwidthSettings/operationResults/read | 列出作業結果 |
> | 動作 | Microsoft.StorSimple/managers/bandwidthSettings/read | 列出頻寬設定 (僅限 8000 系列) |
> | 動作 | Microsoft.StorSimple/managers/bandwidthSettings/write | 新建或更新頻寬設定 (僅限 8000 系列) |
> | 動作 | Microsoft.StorSimple/managers/certificates/write | 建立或更新憑證 |
> | 動作 | Microsoft.StorSimple/Managers/certificates/write | 「更新資源憑證」作業會更新資源/保存庫的認證憑證。 |
> | 動作 | Microsoft.StorSimple/managers/clearAlerts/action | 清除與裝置管理員相關聯的所有警示。 |
> | 動作 | Microsoft.StorSimple/managers/cloudApplianceConfigurations/read | 列出雲端應用裝置所支援的組態 |
> | 動作 | Microsoft.StorSimple/managers/configureDevice/action | 設定裝置 |
> | 動作 | Microsoft.StorSimple/managers/delete | 刪除裝置管理員 |
> | 動作 | Microsoft.StorSimple/Managers/delete | 「刪除保存庫」作業會刪除 'vault' 類型的指定 Azure 資源 |
> | 動作 | Microsoft.StorSimple/managers/devices/alertSettings/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/alertSettings/read | 列出或取得警示設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/alertSettings/write | 建立或更新警示設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/authorizeForServiceEncryptionKeyRollover/action | 授權裝置的服務加密金鑰變換 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupPolicies/backup/action | 製作手動備份，以針對原則所保護的所有磁碟區建立隨選備份。 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupPolicies/delete | 刪除現有的備份原則 (僅限 8000 系列) |
> | 動作 | Microsoft.StorSimple/managers/devices/backupPolicies/operationResults/read | 列出作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupPolicies/read | 列出備份原則 (僅限 8000 系列) |
> | 動作 | Microsoft.StorSimple/managers/devices/backupPolicies/schedules/delete | 刪除現有排程 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupPolicies/schedules/operationResults/read | 列出作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupPolicies/schedules/read | 列出排程 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupPolicies/schedules/write | 新建或更新排程 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupPolicies/write | 新建或更新備份原則 (僅限 8000 系列) |
> | 動作 | Microsoft.StorSimple/managers/devices/backups/delete | 刪除備份組 |
> | 動作 | Microsoft.StorSimple/managers/devices/backups/elements/clone/action | 使用備份元素來複製共用或磁碟區。 |
> | 動作 | Microsoft.StorSimple/managers/devices/backups/elements/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/backups/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/backups/read | 列出或取得備份組 |
> | 動作 | Microsoft.StorSimple/managers/devices/backups/restore/action | 從備份組還原所有磁碟區。 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupScheduleGroups/delete | 刪除備份排程群組 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupScheduleGroups/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupScheduleGroups/read | 列出或取得備份排程群組 |
> | 動作 | Microsoft.StorSimple/managers/devices/backupScheduleGroups/write | 建立或更新備份排程群組 |
> | 動作 | Microsoft.StorSimple/managers/devices/chapSettings/delete | 刪除 Chap 設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/chapSettings/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/chapSettings/read | 列出或取得 Chap 設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/chapSettings/write | 建立或更新 Chap 設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/deactivate/action | 停用裝置。 |
> | 動作 | Microsoft.StorSimple/managers/devices/delete | 刪除裝置 |
> | 動作 | Microsoft.StorSimple/managers/devices/disks/read | 列出或取得磁碟 |
> | 動作 | Microsoft.StorSimple/managers/devices/download/action | 下載裝置的更新。 |
> | 動作 | Microsoft.StorSimple/managers/devices/failover/action | 裝置的容錯移轉。 |
> | 動作 | Microsoft.StorSimple/managers/devices/failover/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/failoverTargets/read | 列出或取得裝置的容錯移轉目標 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/backup/action | 製作檔案伺服器的備份。 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/delete | 刪除檔案伺服器 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/metrics/read | 列出或取得計量 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/metricsDefinitions/read | 列出或取得計量定義 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/read | 列出或取得檔案伺服器 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/shares/delete | 刪除共用 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/shares/metrics/read | 列出或取得計量 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/shares/metricsDefinitions/read | 列出或取得計量定義 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/shares/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/shares/read | 列出或取得共用 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/shares/write | 建立或更新共用 |
> | 動作 | Microsoft.StorSimple/managers/devices/fileservers/write | 建立或更新檔案伺服器 |
> | 動作 | Microsoft.StorSimple/managers/devices/hardwareComponentGroups/changeControllerPowerState/action | 變更硬體元件群組的控制器電源狀態 |
> | 動作 | Microsoft.StorSimple/managers/devices/hardwareComponentGroups/operationResults/read | 列出作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/hardwareComponentGroups/read | 列出硬體元件群組 |
> | 動作 | Microsoft.StorSimple/managers/devices/install/action | 在裝置上安裝更新。 |
> | 動作 | Microsoft.StorSimple/managers/devices/installUpdates/action | 在裝置 (僅限 8000 系列) 上安裝更新。 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/backup/action | 製作 iSCSI 伺服器的備份。 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/delete | 刪除 iSCSI 伺服器 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/disks/delete | 刪除磁碟 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/disks/metrics/read | 列出或取得計量 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/disks/metricsDefinitions/read | 列出或取得計量定義 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/disks/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/disks/read | 列出或取得磁碟 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/disks/write | 建立或更新磁碟 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/metrics/read | 列出或取得計量 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/metricsDefinitions/read | 列出或取得計量定義 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/read | 列出或取得 iSCSI 伺服器 |
> | 動作 | Microsoft.StorSimple/managers/devices/iscsiservers/write | 建立或更新 iSCSI 伺服器 |
> | 動作 | Microsoft.StorSimple/managers/devices/jobs/cancel/action | 取消正在執行的作業 |
> | 動作 | Microsoft.StorSimple/managers/devices/jobs/operationResults/read | 列出作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/jobs/read | 列出或取得作業 |
> | 動作 | Microsoft.StorSimple/managers/devices/listFailoverSets/action | 列出現有裝置 (僅限 8000 系列) 的容錯移轉集。 |
> | 動作 | Microsoft.StorSimple/managers/devices/listFailoverTargets/action | 列出裝置 (僅限 8000 系列) 的容錯移轉目標。 |
> | 動作 | Microsoft.StorSimple/managers/devices/metrics/read | 列出或取得計量 |
> | 動作 | Microsoft.StorSimple/managers/devices/metricsDefinitions/read | 列出或取得計量定義 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/confirmMigration/action | 確認移轉成功，並加以認可。 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/confirmMigrationStatus/read | 列出確認移轉狀態 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/fetchConfirmMigrationStatus/action | 提取移轉的確認狀態。 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/fetchMigrationEstimate/action | 提取移轉估計作業的狀態。 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/fetchMigrationStatus/action | 提取移轉狀態。 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/import/action | 匯入用於移轉的來源組態 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/migrationEstimate/read | 列出移轉估計 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/migrationStatus/read | 列出移轉狀態 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/operationResults/read | 列出作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/startMigration/action | 使用來源組態來開始移轉 |
> | 動作 | Microsoft.StorSimple/managers/devices/migrationSourceConfigurations/startMigrationEstimate/action | 啟動作業來預估移轉程序的持續時間。 |
> | 動作 | Microsoft.StorSimple/managers/devices/networkSettings/operationResults/read | 列出作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/networkSettings/read | 列出或取得網路設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/networkSettings/write | 新建或更新網路設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/publicEncryptionKey/action | 列出裝置管理員的公用加密金鑰 |
> | 動作 | Microsoft.StorSimple/managers/devices/publishSupportPackage/action | 發佈裝置的支援套件以供進行 Microsoft 支援服務疑難排解。 |
> | 動作 | Microsoft.StorSimple/managers/devices/read | 列出或取得裝置 |
> | 動作 | Microsoft.StorSimple/managers/devices/scanForUpdates/action | 掃描裝置的更新。 |
> | 動作 | Microsoft.StorSimple/managers/devices/securitySettings/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/securitySettings/read | 列出安全性設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/securitySettings/syncRemoteManagementCertificate/action | 同步處理裝置的遠端管理憑證。 |
> | 動作 | Microsoft.StorSimple/managers/devices/securitySettings/update/action | 更新安全性設定。 |
> | 動作 | Microsoft.StorSimple/managers/devices/securitySettings/write | 新建或更新安全性設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/sendTestAlertEmail/action | 傳送測試警示電子郵件給所設定的電子郵件收件者。 |
> | 動作 | Microsoft.StorSimple/managers/devices/shares/read | 列出或取得共用 |
> | 動作 | Microsoft.StorSimple/managers/devices/timeSettings/operationResults/read | 列出作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/timeSettings/read | 列出或取得時間設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/timeSettings/write | 新建或更新時間設定 |
> | 動作 | Microsoft.StorSimple/managers/devices/updates/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/updateSummary/read | 列出或取得更新摘要 |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/delete | 刪除現有的磁碟區容器 (僅限 8000 系列) |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/metrics/read | 列出計量 |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/metricsDefinitions/read | 列出計量定義 |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/operationResults/read | 列出作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/read | 列出磁碟區容器 (僅限 8000 系列) |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/volumes/delete | 刪除現有磁碟區 |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/volumes/metrics/read | 列出計量 |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/volumes/metricsDefinitions/read | 列出計量定義 |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/volumes/operationResults/read | 列出作業結果 |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/volumes/read | 列出磁碟區 |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/volumes/write | 新建或更新磁碟區 |
> | 動作 | Microsoft.StorSimple/managers/devices/volumeContainers/write | 新建或更新磁碟區容器 (僅限 8000 系列) |
> | 動作 | Microsoft.StorSimple/managers/devices/volumes/read | 列出磁碟區 |
> | 動作 | Microsoft.StorSimple/managers/devices/write | 建立或更新裝置 |
> | 動作 | Microsoft.StorSimple/managers/encryptionSettings/read | 列出或取得加密設定 |
> | 動作 | Microsoft.StorSimple/managers/extendedInformation/delete | 刪除擴充的保存庫資訊 |
> | 動作 | Microsoft.StorSimple/Managers/extendedInformation/delete | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | 動作 | Microsoft.StorSimple/managers/extendedInformation/read | 列出或取得延伸的保存庫資訊 |
> | 動作 | Microsoft.StorSimple/Managers/extendedInformation/read | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | 動作 | Microsoft.StorSimple/managers/extendedInformation/write | 建立或更新延伸的保存庫資訊 |
> | 動作 | Microsoft.StorSimple/Managers/extendedInformation/write | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | 動作 | Microsoft.StorSimple/managers/features/read | 列出功能 |
> | 動作 | Microsoft.StorSimple/managers/fileservers/read | 列出或取得檔案伺服器 |
> | 動作 | Microsoft.StorSimple/managers/getActivationKey/action | 取得裝置管理員的啟用金鑰。 |
> | 動作 | Microsoft.StorSimple/managers/getEncryptionKey/action | 取得裝置管理員的加密金鑰。 |
> | 動作 | Microsoft.StorSimple/managers/iscsiservers/read | 列出或取得 iSCSI 伺服器 |
> | 動作 | Microsoft.StorSimple/managers/jobs/read | 列出或取得作業 |
> | 動作 | Microsoft.StorSimple/managers/listActivationKey/action | 取得 StorSimple 裝置管理員的啟用金鑰。 |
> | 動作 | Microsoft.StorSimple/managers/listPublicEncryptionKey/action | 列出 StorSimple 裝置管理員的公用加密金鑰。 |
> | 動作 | Microsoft.StorSimple/managers/metrics/read | 列出或取得計量 |
> | 動作 | Microsoft.StorSimple/managers/metricsDefinitions/read | 列出或取得計量定義 |
> | 動作 | Microsoft.StorSimple/managers/migrateClassicToResourceManager/action | 從傳統的管理員移轉至 Resource Manager 管理員 |
> | 動作 | Microsoft.StorSimple/managers/migrationSourceConfigurations/read | 列出移轉來源設定 (僅限 8000 系列) |
> | 動作 | Microsoft.StorSimple/managers/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/provisionCloudAppliance/action | 建立新的雲端應用裝置。 |
> | 動作 | Microsoft.StorSimple/managers/read | 列出或取得裝置管理員 |
> | 動作 | Microsoft.StorSimple/Managers/read | 「取得保存庫」作業會取得物件，此物件代表 'vault' 類型的 Azure 資源 |
> | 動作 | Microsoft.StorSimple/managers/regenarateRegistationCertificate/action | 重新產生裝置管理員的註冊憑證。 |
> | 動作 | Microsoft.StorSimple/managers/regenerateActivationKey/action | 重新產生裝置管理員的啟用金鑰。 |
> | 動作 | Microsoft.StorSimple/managers/storageAccountCredentials/delete | 刪除儲存體帳戶認證 |
> | 動作 | Microsoft.StorSimple/managers/storageAccountCredentials/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/storageAccountCredentials/read | 列出或取得儲存體帳戶認證 |
> | 動作 | Microsoft.StorSimple/managers/storageAccountCredentials/write | 建立或更新儲存體帳戶認證 |
> | 動作 | Microsoft.StorSimple/managers/storageDomains/delete | 刪除儲存體網域 |
> | 動作 | Microsoft.StorSimple/managers/storageDomains/operationResults/read | 列出或取得作業結果 |
> | 動作 | Microsoft.StorSimple/managers/storageDomains/read | 列出或取得儲存體網域 |
> | 動作 | Microsoft.StorSimple/managers/storageDomains/write | 建立或更新儲存體網域 |
> | 動作 | Microsoft.StorSimple/managers/write | 建立或更新裝置管理員 |
> | 動作 | Microsoft.StorSimple/Managers/write | 「建立保存庫」作業會建立 'vault' 類型的 Azure 資源 |
> | 動作 | Microsoft.StorSimple/operations/read | 列出或取得作業 |
> | 動作 | Microsoft.StorSimple/register/action | 註冊提供者 Microsoft.StorSimple |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.StreamAnalytics/locations/quotas/Read | 讀取串流分析訂用帳戶配額 |
> | 動作 | Microsoft.StreamAnalytics/operations/Read | 讀取串流分析作業 |
> | 動作 | Microsoft.StreamAnalytics/Register/action | 向串流分析資源提供者註冊訂用帳戶 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/Delete | 刪除串流分析作業 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/functions/Delete | 刪除串流分析作業函式 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/functions/operationresults/Read | 讀取串流分析作業函式的作業結果 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/functions/Read | 讀取串流分析作業函式 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/functions/RetrieveDefaultDefinition/action | 擷取串流分析作業函式的預設定義 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/functions/Test/action | 測試串流分析作業函式 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/functions/Write | 寫入串流分析作業函式 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/inputs/Delete | 刪除串流分析作業輸入 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/inputs/operationresults/Read | 讀取串流分析作業輸入的作業結果 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/inputs/Read | 讀取串流分析作業輸入 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/inputs/Sample/action | 進行串流分析作業輸入的取樣 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/inputs/Test/action | 測試串流分析作業輸入 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/inputs/Write | 寫入串流分析作業輸入 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/metricdefinitions/Read | 讀取計量定義 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/operationresults/Read | 讀取串流分析作業的作業結果 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/outputs/Delete | 刪除串流分析作業輸出 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/outputs/operationresults/Read | 讀取串流分析作業輸出的作業結果 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/outputs/Read | 讀取串流分析作業輸出 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/outputs/Test/action | 測試串流分析作業輸出 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/outputs/Write | 寫入串流分析作業輸出 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/providers/Microsoft.Insights/diagnosticSettings/read | 讀取診斷設定。 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/providers/Microsoft.Insights/diagnosticSettings/write | 寫入診斷設定。 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/providers/Microsoft.Insights/logDefinitions/read | 取得 streamingjobs 的可用記錄 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/providers/Microsoft.Insights/metricDefinitions/read | 取得 streamingjobs 的可用計量 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/Read | 讀取串流分析作業 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/Start/action | 啟動串流分析作業 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/Stop/action | 停止串流分析作業 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/transformations/Delete | 刪除串流分析作業轉換 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/transformations/Read | 讀取串流分析作業轉換 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/transformations/Write | 寫入串流分析作業轉換 |
> | 動作 | Microsoft.StreamAnalytics/streamingjobs/Write | 寫入串流分析作業 |

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Subscription/CreateSubscription/action | 建立 Azure 訂用帳戶 |
> | 動作 | Microsoft.Subscription/SubscriptionDefinitions/read | 取得管理群組中的 Azure 訂用帳戶定義。 |
> | 動作 | Microsoft.Subscription/SubscriptionDefinitions/write | 建立 Azure 訂用帳戶定義 |

## <a name="microsoftsupport"></a>Microsoft.Support

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.Support/register/action | 支援資源提供者的暫存器 |
> | 動作 | Microsoft.Support/supportTickets/read | 取得支援票證的詳細資料 (包括狀態、嚴重性、連絡人詳細資料和通訊)，或取得跨訂用帳戶之支援票證的清單。 |
> | 動作 | Microsoft.Support/supportTickets/write | 建立或更新支援票證。 您可以建立技術、帳單、配額或訂用帳戶管理相關問題的支援票證。 您可以更新現有支援票證的嚴重性、連絡人詳細資料和通訊。 |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.TimeSeriesInsights/environments/accesspolicies/delete | 刪除存取原則。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/accesspolicies/read | 取得存取原則的屬性。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/accesspolicies/write | 針對環境建立新的存取原則，或更新現有的存取原則。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/delete | 刪除環境。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/eventsources/delete | 刪除事件來源。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/eventsources/eventsources/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/eventsources/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/eventsources/providers/Microsoft.Insights/metricDefinitions/read | 取得事件來源的可用計量 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/eventsources/read | 取得事件來源的屬性。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/eventsources/write | 針對環境建立新的事件來源，或更新現有的事件來源。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/providers/Microsoft.Insights/metricDefinitions/read | 取得環境的可用計量 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/read | 取得環境的屬性。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/referencedatasets/delete | 刪除參考資料集。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/referencedatasets/read | 取得參考資料集的屬性。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/referencedatasets/write | 針對環境建立新的參考資料集，或更新現有的參考資料集。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/status/read | 取得環境的狀態，其相關聯作業 (例如輸入) 的狀態。 |
> | 動作 | Microsoft.TimeSeriesInsights/environments/write | 建立新的環境，或更新現有的環境。 |
> | 動作 | Microsoft.TimeSeriesInsights/register/action | 為時間序列深入解析資源提供者註冊訂用帳戶，並讓您能夠建立時間序列深入解析環境。 |

## <a name="microsoftvisualstudio"></a>Microsoft.VisualStudio

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.VisualStudio/Account/Delete | 刪除帳戶 |
> | 動作 | Microsoft.VisualStudio/Account/Extension/Read | 讀取帳戶/擴充功能 |
> | 動作 | Microsoft.VisualStudio/Account/Project/Read | 讀取帳戶/專案 |
> | 動作 | Microsoft.VisualStudio/Account/Project/Write | 讀取帳戶/專案 |
> | 動作 | Microsoft.VisualStudio/Account/Read | 讀取帳戶 |
> | 動作 | Microsoft.VisualStudio/Account/Write | 設定帳戶 |
> | 動作 | Microsoft.VisualStudio/Extension/Delete | 刪除擴充功能 |
> | 動作 | Microsoft.VisualStudio/Extension/Read | 讀取擴充功能 |
> | 動作 | Microsoft.VisualStudio/Extension/Write | 設定擴充功能 |
> | 動作 | Microsoft.VisualStudio/Project/Delete | 刪除專案 |
> | 動作 | Microsoft.VisualStudio/Project/Read | 讀取專案 |
> | 動作 | Microsoft.VisualStudio/Project/Write | 設定專案 |
> | 動作 | Microsoft.VisualStudio/Register/Action | 向 Microsoft.VisualStudio 提供者註冊 Azure 訂用帳戶 |

## <a name="microsoftweb"></a>microsoft.web

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | microsoft.web/apimanagementaccounts/apiacls/read | 取得 API 管理帳戶 Apiacl。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/apiacls/delete | 刪除 API 管理帳戶 API Apiacl。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/apiacls/read | 讀取 API 管理帳戶 API Apiacl。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/apiacls/write | 更新 API 管理帳戶 API Apiacl。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connectionacls/read | 取得 API 管理帳戶 API Connectionacl。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connections/confirmconsentcode/action | 確認同意碼 API 管理帳戶 API 連線。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connections/connectionacls/delete | 刪除 API 管理帳戶 API 連線 Connectionacl。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connections/connectionacls/read | 取得 API 管理帳戶 API 連線 Connectionacl。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connections/connectionacls/write | 更新 API 管理帳戶 API 連線 Connectionacl。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connections/delete | 刪除 API 管理帳戶 API 連線。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connections/getconsentlinks/action | 取得 API 管理帳戶 API 連線的同意連結。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connections/listconnectionkeys/action | 列出 API 管理帳戶 API 連線的連線金鑰。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connections/listsecrets/action | 列出 API 管理帳戶 API 連線的祕密。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connections/read | 取得 API 管理帳戶 API 連線。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/connections/write | 更新 API 管理帳戶 API 連線。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/delete | 刪除 API 管理帳戶 API。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/localizeddefinitions/delete | 刪除 API 管理帳戶 API 已當地語系化的定義。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/localizeddefinitions/read | 取得 API 管理帳戶 API 已當地語系化的定義。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/localizeddefinitions/write | 更新 API 管理帳戶 API 已當地語系化的定義。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/read | 取得 API 管理帳戶 API。 |
> | 動作 | microsoft.web/apimanagementaccounts/apis/write | 更新 API 管理帳戶 API。 |
> | 動作 | microsoft.web/apimanagementaccounts/connectionacls/read | 取得 API 管理帳戶 Connectionacl。 |
> | 動作 | microsoft.web/availablestacks/read | 取得可用堆疊。 |
> | 動作 | microsoft.web/billingmeters/read | 取得計費計量的清單。 |
> | 動作 | Microsoft.Web/certificates/Delete | 刪除現有憑證。 |
> | 動作 | Microsoft.Web/certificates/Read | 取得憑證清單。 |
> | 動作 | Microsoft.Web/certificates/Write | 新增憑證或更新現有憑證。 |
> | 動作 | microsoft.web/checknameavailability/read | 檢查資源名稱是否可供使用。 |
> | 動作 | microsoft.web/classicmobileservices/read | 取得傳統的行動服務。 |
> | 動作 | Microsoft.Web/connectionGateways/Delete | 刪除連接閘道器。 |
> | 動作 | Microsoft.Web/connectionGateways/Join/Action | 加入連線閘道器。 |
> | 動作 | Microsoft.Web/connectionGateways/ListStatus/Action | 列出連線閘道的狀態。 |
> | 動作 | Microsoft.Web/connectionGateways/Move/Action | 移動連線閘道。 |
> | 動作 | Microsoft.Web/connectionGateways/Read | 取得連線閘道器的清單。 |
> | 動作 | Microsoft.Web/connectionGateways/Write | 建立或更新連線閘道器。 |
> | 動作 | microsoft.web/connections/confirmconsentcode/action | 確認連線同意程式碼。 |
> | 動作 | Microsoft.Web/connections/Delete | 刪除連線。 |
> | 動作 | Microsoft.Web/connections/Join/Action | 加入連線。 |
> | 動作 | microsoft.web/connections/listconsentlinks/action | 列出連線的同意連結。 |
> | 動作 | Microsoft.Web/connections/Move/Action | 移動連線。 |
> | 動作 | Microsoft.Web/connections/Read | 取得連線清單。 |
> | 動作 | Microsoft.Web/connections/Write | 建立或更新連線。 |
> | 動作 | Microsoft.Web/customApis/Delete | 刪除自訂 API。 |
> | 動作 | Microsoft.Web/customApis/extractApiDefinitionFromWsdl/Action | 從 WSDL 中擷取 API 定義。 |
> | 動作 | Microsoft.Web/customApis/Join/Action | 加入自訂 API。 |
> | 動作 | Microsoft.Web/customApis/listWsdlInterfaces/Action | 列出自訂 API 的 WSDL 介面。 |
> | 動作 | Microsoft.Web/customApis/Move/Action | 移動自訂 API。 |
> | 動作 | Microsoft.Web/customApis/Read | 取得自訂 API 的清單。 |
> | 動作 | Microsoft.Web/customApis/Write | 建立或更新自訂 API。 |
> | 動作 | Microsoft.Web/deletedSites/Read | 取得所刪除 Web 應用程式的屬性 |
> | 動作 | microsoft.web/deploymentlocations/read | 取得部署位置。 |
> | 動作 | Microsoft.Web/geoRegions/Read | 取得地理區域清單。 |
> | 動作 | microsoft.web/hostingenvironments/capacities/read | 取得裝載環境容量。 |
> | 動作 | Microsoft.Web/hostingEnvironments/Delete | 刪除 App Service 環境 |
> | 動作 | microsoft.web/hostingenvironments/detectors/read | 取得裝載環境偵測器。 |
> | 動作 | microsoft.web/hostingenvironments/diagnostics/read | 取得裝載環境診斷。 |
> | 動作 | microsoft.web/hostingenvironments/inboundnetworkdependenciesendpoints/read | 取得所有輸入相依性的網路端點。 |
> | 動作 | microsoft.web/hostingenvironments/metricdefinitions/read | 取得裝載環境的計量定義。 |
> | 動作 | microsoft.web/hostingenvironments/multirolepools/metricdefinitions/read | 取得裝載環境 MultiRole 集區計量定義。 |
> | 動作 | microsoft.web/hostingenvironments/multirolepools/metrics/read | 取得裝載環境 MultiRole 集區計量。 |
> | 動作 | Microsoft.Web/hostingEnvironments/multiRolePools/providers/Microsoft.Insights/metricDefinitions/Read | 取得 App Service 環境 MultiRole 的可用計量 |
> | 動作 | Microsoft.Web/hostingEnvironments/multiRolePools/Read | 取得 App Service 環境中前端集區的屬性 |
> | 動作 | microsoft.web/hostingenvironments/multirolepools/skus/read | 取得裝載環境 MultiRole 集區 SKU。 |
> | 動作 | microsoft.web/hostingenvironments/multirolepools/usages/read | 取得裝載環境 MultiRole 集區使用量。 |
> | 動作 | Microsoft.Web/hostingEnvironments/multiRolePools/Write | 在 App Service 環境中建立新的前端集區，或更新 App Service 環境中現有的前端集區 |
> | 動作 | microsoft.web/hostingenvironments/operations/read | 取得裝載環境作業。 |
> | 動作 | microsoft.web/hostingenvironments/outboundnetworkdependenciesendpoints/read | 取得所有輸出相依性的網路端點。 |
> | 動作 | microsoft.web/hostingenvironments/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | microsoft.web/hostingenvironments/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Web/hostingEnvironments/Read | 取得 App Service 環境的屬性 |
> | 動作 | Microsoft.Web/hostingEnvironments/reboot/Action | 將 App Service 環境中的所有機器重新開機 |
> | 動作 | microsoft.web/hostingenvironments/resume/action | 繼續裝載環境。 |
> | 動作 | microsoft.web/hostingenvironments/serverfarms/read | 取得裝載環境 App Service 方案。 |
> | 動作 | microsoft.web/hostingenvironments/sites/read | 取得裝載環境 Web Apps。 |
> | 動作 | microsoft.web/hostingenvironments/suspend/action | 暫止裝載環境。 |
> | 動作 | microsoft.web/hostingenvironments/usages/read | 取得裝載環境使用量。 |
> | 動作 | microsoft.web/hostingenvironments/workerpools/metricdefinitions/read | 取得裝載環境 Workerpools 計量定義。 |
> | 動作 | microsoft.web/hostingenvironments/workerpools/metrics/read | 取得裝載環境 Workerpools 計量。 |
> | 動作 | Microsoft.Web/hostingEnvironments/workerPools/providers/Microsoft.Insights/metricDefinitions/Read | 取得 App Service 環境 WorkerPool 的可用計量 |
> | 動作 | Microsoft.Web/hostingEnvironments/workerPools/Read | 取得 App Service 環境中背景工作集區的屬性 |
> | 動作 | microsoft.web/hostingenvironments/workerpools/skus/read | 取得裝載環境 Workerpools SKU。 |
> | 動作 | microsoft.web/hostingenvironments/workerpools/usages/read | 取得裝載環境 Workerpools 使用量。 |
> | 動作 | Microsoft.Web/hostingEnvironments/workerPools/Write | 在 App Service 環境中建立新的背景工作集區，或更新 App Service 環境中現有的背景工作集區 |
> | 動作 | Microsoft.Web/hostingEnvironments/Write | 建立新的 App Service 環境，或更新現有的 App Service 環境 |
> | 動作 | microsoft.web/ishostingenvironmentnameavailable/read | 取得裝載環境名稱是否可供使用。 |
> | 動作 | microsoft.web/ishostnameavailable/read | 檢查主機名稱是否可供使用。 |
> | 動作 | microsoft.web/isusernameavailable/read | 檢查使用者名稱是否可供使用。 |
> | 動作 | Microsoft.Web/listSitesAssignedToHostName/Read | 取得指派給主機名稱之網站的名稱。 |
> | 動作 | microsoft.web/locations/apioperations/read | 取得位置 API 作業。 |
> | 動作 | microsoft.web/locations/connectiongatewayinstallations/read | 取得位置連線閘道器安裝。 |
> | 動作 | microsoft.web/locations/extractapidefinitionfromwsdl/action | 針對位置從 WSDL 中擷取 API 定義。 |
> | 動作 | microsoft.web/locations/listwsdlinterfaces/action | 列出適用於位置的 WSDL 介面。 |
> | 動作 | microsoft.web/locations/managedapis/apioperations/read | 取得位置受控 API 作業。 |
> | 動作 | Microsoft.Web/locations/managedapis/Join/Action | 加入受控 API。 |
> | 動作 | microsoft.web/locations/managedapis/read | 取得位置管理的 API。 |
> | 動作 | microsoft.web/operations/read | 取得作業。 |
> | 動作 | microsoft.web/publishingusers/read | 取得發佈使用者。 |
> | 動作 | microsoft.web/publishingusers/write | 更新發佈使用者。 |
> | 動作 | Microsoft.Web/recommendations/Read | 取得訂用帳戶的建議清單。 |
> | 動作 | microsoft.web/register/action | 針對訂用帳戶註冊 Microsoft.Web 資源提供者。 |
> | 動作 | microsoft.web/resourcehealthmetadata/read | 取得資源健康狀態中繼資料。 |
> | 動作 | microsoft.web/serverfarms/capabilities/read | 取得 App Service 方案的容量。 |
> | 動作 | Microsoft.Web/serverfarms/Delete | 刪除現有的 App Service 方案 |
> | 動作 | microsoft.web/serverfarms/firstpartyapps/settings/delete | 刪除 App Service 方案的第一方應用程式設定。 |
> | 動作 | microsoft.web/serverfarms/firstpartyapps/settings/read | 取得 App Service 方案的第一方應用程式設定。 |
> | 動作 | microsoft.web/serverfarms/firstpartyapps/settings/write | 更新 App Service 方案的第一方應用程式設定。 |
> | 動作 | microsoft.web/serverfarms/hybridconnectionnamespaces/relays/delete | 刪除 App Service 方案的混合式連線命名空間轉送。 |
> | 動作 | microsoft.web/serverfarms/hybridconnectionnamespaces/relays/read | 取得 App Service 方案的混合式連線命名空間轉送。 |
> | 動作 | microsoft.web/serverfarms/hybridconnectionnamespaces/relays/sites/read | 取得 App Service 方案的混合式連線命名空間轉送 Web Apps。 |
> | 動作 | microsoft.web/serverfarms/hybridconnectionplanlimits/read | 取得 App Service 方案的混合式連線方案限制。 |
> | 動作 | microsoft.web/serverfarms/hybridconnectionrelays/read | 取得 App Service 方案的混合式連線轉送。 |
> | 動作 | microsoft.web/serverfarms/metricdefinitions/read | 取得 App Service 方案的計量定義。 |
> | 動作 | microsoft.web/serverfarms/metrics/read | 取得 App Service 方案的計量。 |
> | 動作 | microsoft.web/serverfarms/operationresults/read | 取得 App Service 方案的作業結果。 |
> | 動作 | microsoft.web/serverfarms/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | microsoft.web/serverfarms/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | Microsoft.Web/serverfarms/providers/Microsoft.Insights/metricDefinitions/Read | 取得 App Service 方案的可用計量 |
> | 動作 | Microsoft.Web/serverfarms/Read | 取得 App Service 方案的屬性 |
> | 動作 | Microsoft.Web/serverfarms/restartSites/Action | 重新啟動 App Service 方案中的所有 Web Apps |
> | 動作 | microsoft.web/serverfarms/sites/read | 取得 App Service 方案的 Web Apps。 |
> | 動作 | microsoft.web/serverfarms/skus/read | 取得 App Service 方案 SKU。 |
> | 動作 | microsoft.web/serverfarms/usages/read | 取得 App Service 方案的使用量。 |
> | 動作 | microsoft.web/serverfarms/virtualnetworkconnections/gateways/write | 更新 App Service 方案的虛擬網路連線閘道器。 |
> | 動作 | microsoft.web/serverfarms/virtualnetworkconnections/read | 取得 App Service 方案的虛擬網路連線。 |
> | 動作 | microsoft.web/serverfarms/virtualnetworkconnections/routes/delete | 刪除 App Service 方案的虛擬網路連線路由。 |
> | 動作 | microsoft.web/serverfarms/virtualnetworkconnections/routes/read | 取得 App Service 方案的虛擬網路連線路由。 |
> | 動作 | microsoft.web/serverfarms/virtualnetworkconnections/routes/write | 更新 App Service 方案的虛擬網路連線路由。 |
> | 動作 | microsoft.web/serverfarms/workers/reboot/action | 重新啟動 App Service 方案的背景工作角色。 |
> | 動作 | Microsoft.Web/serverfarms/Write | 建立新的 App Service 方案，或更新現有的 App Service 方案 |
> | 動作 | microsoft.web/sites/analyzecustomhostname/read | 分析自訂主機名稱。 |
> | 動作 | Microsoft.Web/sites/applySlotConfig/Action | 將目標位置的 Web 應用程式位置組態套用到目前的 Web 應用程式 |
> | 動作 | Microsoft.Web/sites/backup/Action | 建立新的 Web 應用程式備份 |
> | 動作 | microsoft.web/sites/backup/read | 取得 Web Apps 備份。 |
> | 動作 | microsoft.web/sites/backup/write | 更新 Web Apps 備份。 |
> | 動作 | microsoft.web/sites/backups/action | 探索可從 Azure 儲存體中 Blob 還原的現有應用程式備份。 |
> | 動作 | microsoft.web/sites/backups/delete | 刪除 Web Apps 備份。 |
> | 動作 | microsoft.web/sites/backups/list/action | 列出 Web Apps 備份。 |
> | 動作 | Microsoft.Web/sites/backups/Read | 取得 Web 應用程式備份的屬性 |
> | 動作 | microsoft.web/sites/backups/restore/action | 還原 Web Apps 備份。 |
> | 動作 | microsoft.web/sites/backups/write | 更新 Web Apps 備份。 |
> | 動作 | microsoft.web/sites/config/delete | 刪除 Web Apps 設定。 |
> | 動作 | Microsoft.Web/sites/config/list/Action | 列出 Web 應用程式的安全性機密設定，例如發佈認證、應用程式設定和連接字串 |
> | 動作 | Microsoft.Web/sites/config/Read | 取得 Web 應用程式的組態設定 |
> | 動作 | microsoft.web/sites/config/snapshots/read | 取得 Web Apps 組態快照集。 |
> | 動作 | Microsoft.Web/sites/config/Write | 更新 Web 應用程式的組態設定 |
> | 動作 | microsoft.web/sites/containerlogs/action | 取得 Web 應用程式的壓縮容器記錄檔。 |
> | 動作 | microsoft.web/sites/continuouswebjobs/delete | 刪除 Web Apps 的連續 Web 作業。 |
> | 動作 | microsoft.web/sites/continuouswebjobs/read | 取得 Web Apps 的連續 Web 作業。 |
> | 動作 | microsoft.web/sites/continuouswebjobs/start/action | 啟動 Web Apps 的連續 Web 作業。 |
> | 動作 | microsoft.web/sites/continuouswebjobs/stop/action | 停止 Web Apps 的連續 Web 作業。 |
> | 動作 | Microsoft.Web/sites/Delete | 刪除現有的 Web 應用程式 |
> | 動作 | microsoft.web/sites/deployments/delete | 刪除 Web Apps 部署。 |
> | 動作 | microsoft.web/sites/deployments/log/read | 取得 Web Apps 部署記錄。 |
> | 動作 | microsoft.web/sites/deployments/read | 取得 Web Apps 部署。 |
> | 動作 | microsoft.web/sites/deployments/write | 更新 Web Apps 部署。 |
> | 動作 | microsoft.web/sites/detectors/read | 取得 Web Apps 偵測器。 |
> | 動作 | microsoft.web/sites/diagnostics/analyses/execute/Action | 執行 Web Apps 診斷分析。 |
> | 動作 | microsoft.web/sites/diagnostics/analyses/read | 取得 Web Apps 診斷分析。 |
> | 動作 | microsoft.web/sites/diagnostics/aspnetcore/read | 取得 ASP.NET Core 應用程式的 Web Apps 診斷。 |
> | 動作 | microsoft.web/sites/diagnostics/autoheal/read | 取得 Web Apps 診斷 Autoheal。 |
> | 動作 | microsoft.web/sites/diagnostics/deployment/read | 取得 Web Apps 診斷部署。 |
> | 動作 | microsoft.web/sites/diagnostics/deployments/read | 取得 Web Apps 診斷部署。 |
> | 動作 | microsoft.web/sites/diagnostics/detectors/execute/Action | 執行 Web Apps 診斷偵測器。 |
> | 動作 | microsoft.web/sites/diagnostics/detectors/read | 取得 Web Apps 診斷偵測器。 |
> | 動作 | microsoft.web/sites/diagnostics/failedrequestsperuri/read | 取得 Web Apps 診斷對於每個 URI 的失敗要求。 |
> | 動作 | microsoft.web/sites/diagnostics/frebanalysis/read | 取得 Web Apps 診斷的 FREB 分析。 |
> | 動作 | microsoft.web/sites/diagnostics/loganalyzer/read | 取得 Web Apps 診斷的記錄分析器。 |
> | 動作 | microsoft.web/sites/diagnostics/read | 取得 Web Apps 診斷類別。 |
> | 動作 | microsoft.web/sites/diagnostics/runtimeavailability/read | 取得 Web Apps 診斷的執行階段可用性。 |
> | 動作 | microsoft.web/sites/diagnostics/servicehealth/read | 取得 Web Apps 診斷的服務健康狀態。 |
> | 動作 | microsoft.web/sites/diagnostics/sitecpuanalysis/read | 取得 Web Apps 診斷的網站 CPU 分析。 |
> | 動作 | microsoft.web/sites/diagnostics/sitecrashes/read | 取得 Web Apps 診斷的網站損毀。 |
> | 動作 | microsoft.web/sites/diagnostics/sitelatency/read | 取得 Web Apps 診斷的網站延遲。 |
> | 動作 | microsoft.web/sites/diagnostics/sitememoryanalysis/read | 取得 Web Apps 診斷的網站記憶體分析。 |
> | 動作 | microsoft.web/sites/diagnostics/siterestartsettingupdate/read | 取得 Web Apps 診斷的網站重新啟動設定更新。 |
> | 動作 | microsoft.web/sites/diagnostics/siterestartuserinitiated/read | 取得 Web Apps 診斷中使用者初始的網站重新啟動。 |
> | 動作 | microsoft.web/sites/diagnostics/siteswap/read | 取得 Web Apps 診斷的網站交換。 |
> | 動作 | microsoft.web/sites/diagnostics/threadcount/read | 取得 Web Apps 診斷的執行緒計數。 |
> | 動作 | microsoft.web/sites/diagnostics/workeravailability/read | 取得 Web Apps 診斷的 Workeravailability。 |
> | 動作 | microsoft.web/sites/diagnostics/workerprocessrecycle/read | 取得 Web Apps 診斷的背景工作處理序回收。 |
> | 動作 | microsoft.web/sites/domainownershipidentifiers/read | 取得 Web Apps 網域擁有權識別碼。 |
> | 動作 | microsoft.web/sites/domainownershipidentifiers/write | 更新 Web Apps 網域擁有權識別碼。 |
> | 動作 | microsoft.web/sites/functions/action | 函式 Web Apps。 |
> | 動作 | microsoft.web/sites/functions/delete | 刪除 Web Apps 函式。 |
> | 動作 | microsoft.web/sites/functions/listsecrets/action | 列出祕密 Web Apps 函式。 |
> | 動作 | microsoft.web/sites/functions/masterkey/read | 取得 Web Apps 函式的主要金鑰。 |
> | 動作 | microsoft.web/sites/functions/read | 取得 Web Apps 函式。 |
> | 動作 | microsoft.web/sites/functions/token/read | 取得 Web Apps 函式的權杖。 |
> | 動作 | microsoft.web/sites/functions/write | 更新 Web Apps 函式。 |
> | 動作 | microsoft.web/sites/hostnamebindings/delete | 刪除 Web Apps 的主機名稱繫結。 |
> | 動作 | microsoft.web/sites/hostnamebindings/read | 取得 Web Apps 的主機名稱繫結。 |
> | 動作 | microsoft.web/sites/hostnamebindings/write | 更新 Web Apps 的主機名稱繫結。 |
> | 動作 | Microsoft.Web/sites/hostruntime/host/_master/read | 取得系統管理作業的函式應用程式主要金鑰 |
> | 動作 | Microsoft.Web/sites/hostruntime/host/action | 執行函式應用程式的執行階段動作，例如同步觸發程序、新增函式、叫用函式、刪除函式等動作。 |
> | 動作 | microsoft.web/sites/hybridconnection/delete | 刪除 Web Apps 的混合式連線。 |
> | 動作 | microsoft.web/sites/hybridconnection/read | 取得 Web Apps 的混合式連線。 |
> | 動作 | microsoft.web/sites/hybridconnection/write | 更新 Web Apps 的混合式連線。 |
> | 動作 | microsoft.web/sites/hybridconnectionnamespaces/relays/delete | 刪除 Web Apps 的混合式連線命名空間轉送。 |
> | 動作 | microsoft.web/sites/hybridconnectionnamespaces/relays/listkeys/action | 列出 Web Apps 的混合式連線命名空間轉送。 |
> | 動作 | microsoft.web/sites/hybridconnectionnamespaces/relays/read | 取得 Web Apps 的混合式連線命名空間轉送。 |
> | 動作 | microsoft.web/sites/hybridconnectionnamespaces/relays/write | 更新 Web Apps 的混合式連線命名空間轉送。 |
> | 動作 | microsoft.web/sites/hybridconnectionrelays/read | 取得 Web Apps 的混合式連線轉送。 |
> | 動作 | microsoft.web/sites/instances/deployments/delete | 刪除 Web Apps 的執行個體部署。 |
> | 動作 | microsoft.web/sites/instances/deployments/read | 取得 Web Apps 的執行個體部署。 |
> | 動作 | microsoft.web/sites/instances/extensions/log/read | 取得 Web Apps 的執行個體擴充功能記錄。 |
> | 動作 | microsoft.web/sites/instances/extensions/read | 取得 Web Apps 的執行個體擴充功能。 |
> | 動作 | microsoft.web/sites/instances/processes/delete | 刪除 Web Apps 的執行個體處理序。 |
> | 動作 | microsoft.web/sites/instances/processes/read | 取得 Web Apps 的執行個體處理序。 |
> | 動作 | microsoft.web/sites/instances/processes/threads/read | 取得 Web Apps 執行個體處理程序的執行緒。 |
> | 動作 | microsoft.web/sites/instances/read | 取得 Web Apps 執行個體。 |
> | 動作 | microsoft.web/sites/listsyncfunctiontriggerstatus/action | 列出同步函式觸發程序狀態 Web Apps。 |
> | 動作 | microsoft.web/sites/metricdefinitions/read | 取得 Web Apps 計量定義。 |
> | 動作 | microsoft.web/sites/metrics/read | 取得 Web Apps 計量。 |
> | 動作 | microsoft.web/sites/metricsdefinitions/read | 取得 Web Apps 計量定義。 |
> | 動作 | microsoft.web/sites/migratemysql/action | 移轉 MySql Web Apps。 |
> | 動作 | microsoft.web/sites/migratemysql/read | 取得 Web Apps 的移轉 MySql。 |
> | 動作 | microsoft.web/sites/networktrace/action | 網路追蹤 Web Apps。 |
> | 動作 | microsoft.web/sites/newpassword/action | Newpassword Web Apps。 |
> | 動作 | microsoft.web/sites/operationresults/read | 取得 Web Apps 的作業結果。 |
> | 動作 | microsoft.web/sites/operations/read | 取得 Web Apps 作業。 |
> | 動作 | microsoft.web/sites/perfcounters/read | 取得 Web Apps 的效能計數器。 |
> | 動作 | microsoft.web/sites/premieraddons/delete | 刪除 Web Apps 的頂級附加元件。 |
> | 動作 | microsoft.web/sites/premieraddons/read | 取得 Web Apps 的頂級附加元件。 |
> | 動作 | microsoft.web/sites/premieraddons/write | 更新 Web Apps 的頂級附加元件。 |
> | 動作 | microsoft.web/sites/privateaccess/read | 取得私人網站存取啟用和已授權虛擬網路 (可存取該網站) 的資料。 |
> | 動作 | microsoft.web/sites/processes/read | 取得 Web Apps 處理序。 |
> | 動作 | microsoft.web/sites/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | microsoft.web/sites/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | microsoft.web/sites/providers/Microsoft.Insights/logDefinitions/read | 取得 Web 應用程式的可用記錄 |
> | 動作 | Microsoft.Web/sites/providers/Microsoft.Insights/metricDefinitions/Read | 取得 Web 應用程式的可用計量 |
> | 動作 | microsoft.web/sites/publiccertificates/delete | 刪除 Web Apps 的公開憑證。 |
> | 動作 | microsoft.web/sites/publiccertificates/read | 取得 Web Apps 的公開憑證。 |
> | 動作 | microsoft.web/sites/publiccertificates/write | 更新 Web Apps 的公開憑證。 |
> | 動作 | Microsoft.Web/sites/publish/Action | 發佈 Web 應用程式 |
> | 動作 | Microsoft.Web/sites/publishxml/Action | 取得 Web 應用程式的發行設定檔 xml |
> | 動作 | microsoft.web/sites/publishxml/read | 取得 Web Apps 的發佈 XML。 |
> | 動作 | Microsoft.Web/sites/Read | 取得 Web 應用程式的屬性 |
> | 動作 | microsoft.web/sites/recommendationhistory/read | 取得 Web Apps 的建議歷程記錄。 |
> | 動作 | microsoft.web/sites/recommendations/disable/action | 停用 Web Apps 建議。 |
> | 動作 | Microsoft.Web/sites/recommendations/Read | 取得 Web 應用程式的建議清單。 |
> | 動作 | microsoft.web/sites/recover/action | 復原 Web Apps。 |
> | 動作 | Microsoft.Web/sites/resetSlotConfig/Action | 重設 Web 應用程式組態 |
> | 動作 | microsoft.web/sites/resourcehealthmetadata/read | 取得 Web Apps 的資源健康狀態中繼資料。 |
> | 動作 | Microsoft.Web/sites/restart/Action | 重新啟動 Web 應用程式 |
> | 動作 | microsoft.web/sites/restore/read | 取得 Web Apps 還原。 |
> | 動作 | microsoft.web/sites/restore/write | 還原 Web Apps。 |
> | 動作 | microsoft.web/sites/restorefrombackupblob/action | 從備份 Blob 還原 Web 應用程式。 |
> | 動作 | microsoft.web/sites/restorefromdeletedwebapp/action | 從已刪除的應用程式還原 Web Apps。 |
> | 動作 | microsoft.web/sites/restoresnapshot/action | 還原 Web Apps 快照集。 |
> | 動作 | microsoft.web/sites/siteextensions/delete | 刪除 Web Apps 的網站擴充功能。 |
> | 動作 | microsoft.web/sites/siteextensions/read | 取得 Web Apps 的網站擴充功能。 |
> | 動作 | microsoft.web/sites/siteextensions/write | 更新 Web Apps 的網站擴充功能。 |
> | 動作 | microsoft.web/sites/slots/analyzecustomhostname/read | 取得 Web Apps 位置的分析自訂主機名稱。 |
> | 動作 | Microsoft.Web/sites/slots/applySlotConfig/Action | 將目標位置的 Web 應用程式位置組態套用到目前的位置。 |
> | 動作 | Microsoft.Web/sites/slots/backup/Action | 建立新的 Web 應用程式位置備份。 |
> | 動作 | microsoft.web/sites/slots/backup/read | 取得 Web Apps 位置備份。 |
> | 動作 | microsoft.web/sites/slots/backup/write | 更新 Web Apps 位置的備份。 |
> | 動作 | microsoft.web/sites/slots/backups/action | 探索 Web Apps 位置備份。 |
> | 動作 | microsoft.web/sites/slots/backups/delete | 刪除 Web Apps 位置備份。 |
> | 動作 | microsoft.web/sites/slots/backups/list/action | 列出 Web Apps 位置的備份。 |
> | 動作 | Microsoft.Web/sites/slots/backups/Read | 取得 Web 應用程式位置備份的屬性 |
> | 動作 | microsoft.web/sites/slots/backups/restore/action | 還原 Web Apps 位置的備份。 |
> | 動作 | microsoft.web/sites/slots/config/delete | 刪除 Web Apps 的位置設定。 |
> | 動作 | Microsoft.Web/sites/slots/config/list/Action | 列出 Web 應用程式位置的安全性機密設定，例如發佈認證、應用程式設定和連接字串 |
> | 動作 | Microsoft.Web/sites/slots/config/Read | 取得 Web 應用程式位置的組態設定 |
> | 動作 | Microsoft.Web/sites/slots/config/Write | 更新 Web 應用程式位置的組態設定 |
> | 動作 | microsoft.web/sites/slots/containerlogs/action | 取得 Web 應用程式位置的壓縮容器記錄。 |
> | 動作 | microsoft.web/sites/slots/continuouswebjobs/delete | 刪除 Web Apps 位置的連續 Web 作業。 |
> | 動作 | microsoft.web/sites/slots/continuouswebjobs/read | 取得 Web Apps 位置的連續 Web 作業。 |
> | 動作 | microsoft.web/sites/slots/continuouswebjobs/start/action | 啟動 Web Apps 位置的連續 Web 作業。 |
> | 動作 | microsoft.web/sites/slots/continuouswebjobs/stop/action | 停止 Web Apps 位置的連續 Web 作業。 |
> | 動作 | Microsoft.Web/sites/slots/Delete | 刪除現有的 Web 應用程式位置 |
> | 動作 | microsoft.web/sites/slots/deployments/delete | 刪除 Web Apps 的位置部署。 |
> | 動作 | microsoft.web/sites/slots/deployments/log/read | 取得 Web Apps 的位置部署記錄。 |
> | 動作 | microsoft.web/sites/slots/deployments/read | 取得 Web Apps 的位置部署。 |
> | 動作 | microsoft.web/sites/slots/deployments/write | 更新 Web Apps 的位置部署。 |
> | 動作 | microsoft.web/sites/slots/detectors/read | 取得 Web Apps 位置偵測器。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/analyses/execute/Action | 執行 Web Apps 位置診斷的分析。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/analyses/read | 取得 Web Apps 位置診斷的分析。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/aspnetcore/read | 取得 ASP.NET Core 應用程式的 Web Apps 位置診斷。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/autoheal/read | 取得 Web Apps 位置診斷的 Autoheal。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/deployment/read | 取得 Web Apps 位置診斷的部署。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/deployments/read | 取得 Web Apps 位置診斷的部署。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/detectors/execute/Action | 執行 Web Apps 位置診斷的偵測器。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/detectors/read | 取得 Web Apps 位置診斷的偵測器。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/frebanalysis/read | 取得 Web Apps 位置診斷的 FREB 分析。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/loganalyzer/read | 取得 Web Apps 位置診斷的記錄分析器。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/read | 取得 Web Apps 位置診斷。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/runtimeavailability/read | 取得 Web Apps 位置診斷的執行階段可用性。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/servicehealth/read | 取得 Web Apps 位置診斷的服務健康狀態。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/sitecpuanalysis/read | 取得 Web Apps 位置診斷的網站 CPU 分析。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/sitecrashes/read | 取得 Web Apps 位置診斷的網站損毀。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/sitelatency/read | 取得 Web Apps 位置診斷的網站延遲。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/sitememoryanalysis/read | 取得 Web Apps 位置診斷的網站記憶體分析。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/siterestartsettingupdate/read | 取得 Web Apps 位置診斷的網站重新啟動設定更新。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/siterestartuserinitiated/read | 取得 Web Apps 位置診斷中使用者初始的網站重新啟動。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/siteswap/read | 取得 Web Apps 位置診斷的網站交換。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/threadcount/read | 取得 Web Apps 位置診斷的執行緒計數。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/workeravailability/read | 取得 Web Apps 位置診斷的 Workeravailability。 |
> | 動作 | microsoft.web/sites/slots/diagnostics/workerprocessrecycle/read | 取得 Web Apps 位置診斷的背景工作處理序回收。 |
> | 動作 | microsoft.web/sites/slots/domainownershipidentifiers/read | 取得 Web Apps 位置的網域擁有權識別碼。 |
> | 動作 | microsoft.web/sites/slots/functions/read | 取得 Web Apps 位置函式。 |
> | 動作 | microsoft.web/sites/slots/hostnamebindings/delete | 刪除 Web Apps 位置的主機名稱繫結。 |
> | 動作 | microsoft.web/sites/slots/hostnamebindings/read | 取得 Web Apps 位置的主機名稱繫結。 |
> | 動作 | microsoft.web/sites/slots/hostnamebindings/write | 更新 Web Apps 位置的主機名稱繫結。 |
> | 動作 | microsoft.web/sites/slots/hybridconnection/delete | 刪除 Web Apps 位置的混合式連線。 |
> | 動作 | microsoft.web/sites/slots/hybridconnection/read | 取得 Web Apps 位置的混合式連線。 |
> | 動作 | microsoft.web/sites/slots/hybridconnection/write | 更新 Web Apps 位置的混合式連線。 |
> | 動作 | microsoft.web/sites/slots/hybridconnectionnamespaces/relays/delete | 刪除 Web Apps 位置的混合式連線命名空間轉送。 |
> | 動作 | microsoft.web/sites/slots/hybridconnectionnamespaces/relays/write | 更新 Web Apps 位置的混合式連線命名空間轉送。 |
> | 動作 | microsoft.web/sites/slots/hybridconnectionrelays/read | 取得 Web Apps 位置的混合式連線轉送。 |
> | 動作 | microsoft.web/sites/slots/instances/deployments/read | 取得 Web Apps 位置的執行個體部署。 |
> | 動作 | microsoft.web/sites/slots/instances/processes/delete | 刪除 Web Apps 位置的執行個體處理序。 |
> | 動作 | microsoft.web/sites/slots/instances/processes/read | 取得 Web Apps 位置的執行個體處理序。 |
> | 動作 | microsoft.web/sites/slots/instances/read | 取得 Web Apps 位置的執行個體。 |
> | 動作 | microsoft.web/sites/slots/metricdefinitions/read | 取得 Web Apps 位置的計量定義。 |
> | 動作 | microsoft.web/sites/slots/metrics/read | 取得 Web Apps 位置的計量。 |
> | 動作 | microsoft.web/sites/slots/migratemysql/read | 取得 Web Apps 位置的移轉 MySql。 |
> | 動作 | microsoft.web/sites/slots/networktrace/action | 網路追蹤 Web Apps 位置。 |
> | 動作 | microsoft.web/sites/slots/newpassword/action | Newpassword Web Apps 位置。 |
> | 動作 | microsoft.web/sites/slots/operationresults/read | 取得 Web Apps 位置的作業結果。 |
> | 動作 | microsoft.web/sites/slots/operations/read | 取得 Web Apps 位置的作業。 |
> | 動作 | microsoft.web/sites/slots/perfcounters/read | 取得 Web Apps 位置的效能計數器。 |
> | 動作 | microsoft.web/sites/slots/phplogging/read | 取得 Web Apps 位置的 Phplogging。 |
> | 動作 | microsoft.web/sites/slots/premieraddons/delete | 刪除 Web Apps 位置的頂級附加元件。 |
> | 動作 | microsoft.web/sites/slots/premieraddons/read | 取得 Web Apps 位置的頂級附加元件。 |
> | 動作 | microsoft.web/sites/slots/premieraddons/write | 更新 Web Apps 位置的頂級附加元件。 |
> | 動作 | microsoft.web/sites/slots/providers/Microsoft.Insights/diagnosticSettings/read | 取得資源的診斷設定 |
> | 動作 | microsoft.web/sites/slots/providers/Microsoft.Insights/diagnosticSettings/write | 建立或更新資源的診斷設定 |
> | 動作 | microsoft.web/sites/slots/providers/Microsoft.Insights/logDefinitions/read | 取得 Web 應用程式位置的可用記錄 |
> | 動作 | Microsoft.Web/sites/slots/providers/Microsoft.Insights/metricDefinitions/Read | 取得 Web 應用程式位置的可用計量 |
> | 動作 | microsoft.web/sites/slots/publiccertificates/delete | 刪除 Web Apps 位置的公開憑證。 |
> | 動作 | microsoft.web/sites/slots/publiccertificates/read | 取得 Web Apps 位置的公開憑證。 |
> | 動作 | microsoft.web/sites/slots/publiccertificates/write | 建立或更新 Web Apps 位置的公開憑證。 |
> | 動作 | Microsoft.Web/sites/slots/publish/Action | 發佈 Web 應用程式位置 |
> | 動作 | Microsoft.Web/sites/slots/publishxml/Action | 取得 Web 應用程式位置的發行設定檔 xml |
> | 動作 | Microsoft.Web/sites/slots/Read | 取得 Web 應用程式部署位置的屬性 |
> | 動作 | microsoft.web/sites/slots/recover/action | 復原 Web Apps 位置。 |
> | 動作 | Microsoft.Web/sites/slots/resetSlotConfig/Action | 重設 Web 應用程式位置組態 |
> | 動作 | microsoft.web/sites/slots/resourcehealthmetadata/read | 取得 Web Apps 位置的資源健康狀態中繼資料。 |
> | 動作 | Microsoft.Web/sites/slots/restart/Action | 重新啟動 Web 應用程式位置 |
> | 動作 | microsoft.web/sites/slots/restore/read | 取得 Web Apps 位置的還原。 |
> | 動作 | microsoft.web/sites/slots/restore/write | 還原 Web Apps 位置。 |
> | 動作 | microsoft.web/sites/slots/restorefrombackupblob/action | 從備份 Blob 還原 Web 應用程式位置。 |
> | 動作 | microsoft.web/sites/slots/restorefromdeletedwebapp/action | 從已刪除的應用程式還原 Web 應用程式位置。 |
> | 動作 | microsoft.web/sites/slots/restoresnapshot/action | 還原 Web Apps 位置快照集。 |
> | 動作 | microsoft.web/sites/slots/siteextensions/delete | 刪除 Web Apps 位置的網站擴充功能。 |
> | 動作 | microsoft.web/sites/slots/siteextensions/read | 取得 Web Apps 位置的網站擴充功能。 |
> | 動作 | microsoft.web/sites/slots/siteextensions/write | 更新 Web Apps 位置的網站擴充功能。 |
> | 動作 | Microsoft.Web/sites/slots/slotsdiffs/Action | 取得 Web 應用程式和位置之間的組態差異 |
> | 動作 | Microsoft.Web/sites/slots/slotsswap/Action | 交換 Web 應用程式部署位置 |
> | 動作 | microsoft.web/sites/slots/snapshots/read | 取得 Web Apps 位置的快照集。 |
> | 動作 | Microsoft.Web/sites/slots/sourcecontrols/Delete | 刪除 Web 應用程式位置的原始檔控制組態設定 |
> | 動作 | Microsoft.Web/sites/slots/sourcecontrols/Read | 取得 Web 應用程式位置的原始檔控制組態設定 |
> | 動作 | Microsoft.Web/sites/slots/sourcecontrols/Write | 更新 Web 應用程式位置的原始檔控制組態設定 |
> | 動作 | Microsoft.Web/sites/slots/start/Action | 啟動 Web 應用程式位置 |
> | 動作 | Microsoft.Web/sites/slots/stop/Action | 停止 Web 應用程式位置 |
> | 動作 | microsoft.web/sites/slots/sync/action | 同步處理 Web Apps 的位置。 |
> | 動作 | microsoft.web/sites/slots/triggeredwebjobs/delete | 刪除 Web Apps 位置所觸發的 WebJobs。 |
> | 動作 | microsoft.web/sites/slots/triggeredwebjobs/read | 取得 Web Apps 位置所觸發的 WebJobs。 |
> | 動作 | microsoft.web/sites/slots/triggeredwebjobs/run/action | 執行 Web Apps 位置所觸發的 WebJobs。 |
> | 動作 | microsoft.web/sites/slots/usages/read | 取得 Web Apps 位置的使用量。 |
> | 動作 | microsoft.web/sites/slots/virtualnetworkconnections/delete | 刪除 Web Apps 位置的虛擬網路連線。 |
> | 動作 | microsoft.web/sites/slots/virtualnetworkconnections/gateways/write | 更新 Web Apps 位置的虛擬網路連線閘道器。 |
> | 動作 | microsoft.web/sites/slots/virtualnetworkconnections/read | 取得 Web Apps 位置的虛擬網路連線。 |
> | 動作 | microsoft.web/sites/slots/virtualnetworkconnections/write | 更新 Web Apps 位置的虛擬網路連線。 |
> | 動作 | microsoft.web/sites/slots/webjobs/read | 取得 Web Apps 位置的 WebJobs。 |
> | 動作 | Microsoft.Web/sites/slots/Write | 建立新的 Web 應用程式位置，或更新現有的 Web 應用程式位置 |
> | 動作 | Microsoft.Web/sites/slotsdiffs/Action | 取得 Web 應用程式和位置之間的組態差異 |
> | 動作 | Microsoft.Web/sites/slotsswap/Action | 交換 Web 應用程式部署位置 |
> | 動作 | microsoft.web/sites/snapshots/read | 取得 Web Apps 快照集。 |
> | 動作 | Microsoft.Web/sites/sourcecontrols/Delete | 刪除 Web 應用程式的原始檔控制組態設定 |
> | 動作 | Microsoft.Web/sites/sourcecontrols/Read | 取得 Web 應用程式的原始檔控制組態設定 |
> | 動作 | Microsoft.Web/sites/sourcecontrols/Write | 更新 Web 應用程式的原始檔控制組態設定 |
> | 動作 | Microsoft.Web/sites/start/Action | 啟動 Web 應用程式 |
> | 動作 | Microsoft.Web/sites/stop/Action | 停止 Web 應用程式 |
> | 動作 | microsoft.web/sites/sync/action | 同步處理 Web Apps。 |
> | 動作 | microsoft.web/sites/syncfunctiontriggers/action | 同步處理 Web Apps 的函式觸發程序。 |
> | 動作 | microsoft.web/sites/triggeredwebjobs/delete | 刪除 Web Apps 所觸發的 WebJobs。 |
> | 動作 | microsoft.web/sites/triggeredwebjobs/history/read | 取得 Web Apps 所觸發的 WebJobs 歷程記錄。 |
> | 動作 | microsoft.web/sites/triggeredwebjobs/read | 取得 Web Apps 所觸發的 WebJobs。 |
> | 動作 | microsoft.web/sites/triggeredwebjobs/run/action | 執行 Web Apps 所觸發的 WebJobs。 |
> | 動作 | microsoft.web/sites/usages/read | 取得 Web Apps 使用量。 |
> | 動作 | microsoft.web/sites/virtualnetworkconnections/delete | 刪除 Web Apps 的虛擬網路連線。 |
> | 動作 | microsoft.web/sites/virtualnetworkconnections/gateways/read | 取得 Web Apps 的虛擬網路連線閘道器。 |
> | 動作 | microsoft.web/sites/virtualnetworkconnections/gateways/write | 更新 Web Apps 的虛擬網路連線閘道器。 |
> | 動作 | microsoft.web/sites/virtualnetworkconnections/read | 取得 Web Apps 的虛擬網路連線。 |
> | 動作 | microsoft.web/sites/virtualnetworkconnections/write | 更新 Web Apps 的虛擬網路連線。 |
> | 動作 | microsoft.web/sites/webjobs/read | 取得 Web Apps WebJobs。 |
> | 動作 | Microsoft.Web/sites/Write | 建立新的 Web 應用程式，或更新現有 Web 應用程式 |
> | 動作 | microsoft.web/skus/read | 取得 SKU。 |
> | 動作 | microsoft.web/sourcecontrols/read | 取得原始檔控制。 |
> | 動作 | microsoft.web/sourcecontrols/write | 更新原始檔控制。 |
> | 動作 | microsoft.web/unregister/action | 針對訂用帳戶取消註冊 Microsoft.Web 資源提供者。 |
> | 動作 | microsoft.web/validate/action | 驗證。 |
> | 動作 | microsoft.web/verifyhostingenvironmentvnet/action | 確認裝載環境 Vnet。 |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tdCol2BreakAll"]
> | 動作類型 | 作業 | 說明 |
> | --- | --- | --- |
> | 動作 | Microsoft.WorkloadMonitor/components/read | 取得資源的元件 |
> | 動作 | Microsoft.WorkloadMonitor/componentsSummary/read | 取得元件的摘要 |
> | 動作 | Microsoft.WorkloadMonitor/monitorInstances/read | 取得資源的監視器執行個體 |
> | 動作 | Microsoft.WorkloadMonitor/monitorInstancesSummary/read | 取得監視器執行個體的摘要 |
> | 動作 | Microsoft.WorkloadMonitor/monitors/read | 取得資源的監視器 |
> | 動作 | Microsoft.WorkloadMonitor/monitors/write | 設定資源的監視器 |
> | 動作 | Microsoft.WorkloadMonitor/notificationSettings/read | 取得資源的通知設定 |
> | 動作 | Microsoft.WorkloadMonitor/notificationSettings/write | 設定資源的通知設定 |
> | 動作 | Microsoft.WorkloadMonitor/operations/read | 取得支援的作業 |

## <a name="next-steps"></a>後續步驟

- [自訂角色](custom-roles.md)
- [內建角色](built-in-roles.md)
