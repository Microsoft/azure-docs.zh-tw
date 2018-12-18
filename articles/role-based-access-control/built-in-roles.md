---
title: 適用於 Azure 資源的內建角色 | Microsoft Docs
description: 說明角色型存取控制 (RBAC) 和 Azure 資源的內建角色。 列出 Actions、NotActions、DataActions 和 NotDataActions。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: reference
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 09/27/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: it-pro
ms.openlocfilehash: 6fe9a106975a03fabc9d674ede694e683dc3cd94
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47410267"
---
# <a name="built-in-roles-for-azure-resources"></a>適用於 Azure 資源的內建角色
[角色型存取控制 (RBAC)](overview.md) 具有數個內建角色定義，可供您指派給使用者、群組和服務主體。 角色指派是您控制 Azure 資源存取權的方式。 如果內建的角色無法滿足您組織的特定需求，您可以建立自己的[自訂角色](custom-roles.md)。

內建角色不斷發展。 若要取得最新角色定義，請使用 [Get-AzureRmRoleDefinition](/powershell/module/azurerm.resources/get-azurermroledefinition) 或 [az role definition list](/cli/azure/role/definition#az-role-definition-list)。

## <a name="built-in-role-descriptions"></a>內建角色描述
下表提供內建角色的簡短描述。 按一下角色名稱，即可查看每個角色的 `Actions`、`NotActions`、`DataActions` 及 `NotDataActions` 清單。


| 內建角色 | 說明 |
| --- | --- |
| [擁有者](#owner) | 可讓您管理一切，包括對資源的存取。 |
| [參與者](#contributor) | 可讓您管理一切，但無法管理對資源的存取。 |
| [讀取者](#reader) | 可讓您檢視所有項目，但是無法進行變更。 |
| [AcrImageSigner](#acrimagesigner) | ACR 影像簽署者 |
| [AcrQuarantineReader](#acrquarantinereader) | ACR 隔離資料讀取者 |
| [AcrQuarantineWriter](#acrquarantinewriter) | ACR 隔離資料寫入者 |
| [API 管理服務參與者](#api-management-service-contributor) | 可讓您管理 API 管理服務，但無法加以存取。 |
| [API 管理服務操作員角色](#api-management-service-operator-role) | 可管理服務，但無法管理 API |
| [API 管理服務讀取者角色](#api-management-service-reader-role) | 具有服務與 API 的唯讀存取權 |
| [Application Insights 元件參與者](#application-insights-component-contributor) | 可以管理 Application Insights 元件 |
| [Application Insights 快照集偵錯工具](#application-insights-snapshot-debugger) | 給予使用者權限，以便檢視及下載使用 Application Insights 快照偵錯工具所收集的偵錯快照。 請注意，[擁有者](#owner)或[參與者](#contributor)角色未包含這些權限。 |
| [自動化作業運算子](#automation-job-operator) | 使用「自動化 Runbook」來建立及管理作業。 |
| [自動化運算子](#automation-operator) | 「自動化運算子」能夠啟動、停止、暫止及繼續作業 |
| [自動化 Runbook 運算子](#automation-runbook-operator) | 讀取 Runbook 屬性 - 以便能夠建立 Runbook 的作業。 |
| [Azure Kubernetes Service 叢集管理員角色](#azure-kubernetes-service-cluster-admin-role) | 列出叢集管理員認證動作。 |
| [Azure Kubernetes Service 叢集使用者角色](#azure-kubernetes-service-cluster-user-role) | 列出叢集使用者認證動作。 |
| [Azure Stack 註冊擁有者](#azure-stack-registration-owner) | 可讓您管理 Azure Stack 註冊。 |
| [備份參與者](#backup-contributor) | 可讓您管理備份服務，但無法建立保存庫及為其他人提供存取權 |
| [備份操作員](#backup-operator) | 可讓您管理備份服務，但無法移除備份、建立保存庫及為其他人提供存取權 |
| [備份讀取者](#backup-reader) | 可以檢視備份服務，但無法進行變更 |
| [帳單讀取器](#billing-reader) | 可讓您讀取計費資料 |
| [BizTalk 參與者](#biztalk-contributor) | 可讓您管理 BizTalk 服務，但無法存取它們。 |
| [CDN 端點參與者](#cdn-endpoint-contributor) | 可管理 CDN 端點，但無法對其他使用者授與存取權。 |
| [CDN 端點讀者](#cdn-endpoint-reader) | 可檢視 CDN 端點，但無法進行變更。 |
| [CDN 設定檔參與者](#cdn-profile-contributor) | 可管理 CDN 設定檔及其端點，但無法對其他使用者授與存取權。 |
| [CDN 設定檔讀者](#cdn-profile-reader) | 可檢視 CDN 設定檔及其端點，但無法進行變更。 |
| [傳統網路參與者](#classic-network-contributor) | 可讓您管理傳統網路，但無法存取它們。 |
| [傳統儲存體帳戶參與者](#classic-storage-account-contributor) | 可讓您管理傳統儲存體帳戶，但無法存取它們。 |
| [傳統儲存體帳戶金鑰操作員服務角色](#classic-storage-account-key-operator-service-role) | 「傳統儲存體帳戶金鑰操作員」可以列出及重新產生「傳統儲存體帳戶」的金鑰 |
| [傳統虛擬機器參與者](#classic-virtual-machine-contributor) | 可讓您管理傳統虛擬機器 (不含虛擬機器所連線的虛擬網路或儲存體帳戶)，但無法存取它們。 |
| [認知服務參與者](#cognitive-services-contributor) | 可讓您建立、讀取、更新、刪除及管理認知服務的金鑰。 |
| [認知服務使用者](#cognitive-services-user) | 可讓您讀取和列出認知服務的金鑰。 |
| [Cosmos DB 帳戶讀者角色](#cosmos-db-account-reader-role) | 可以讀取 Azure Cosmos DB 帳戶資料。 請參閱 [DocumentDB 帳戶參與者](#documentdb-account-contributor)以管理 Azure Cosmos DB 帳戶。 |
| [成本管理參與者](#cost-management-contributor) | 可檢視成本和管理成本組態 (例如預算、匯出) |
| [成本管理讀者](#cost-management-reader) | 可檢視成本資料和組態 (例如預算、匯出) |
| [資料箱參與者](#data-box-contributor) | 可讓您管理資料箱服務下的所有項目，為他人賦予存取權除外。 |
| [資料箱讀者](#data-box-reader) | 可讓您管理資料箱服務，建立訂單或編輯訂單詳細資料和為他人賦予存取權除外。 |
| [Data Factory 參與者](#data-factory-contributor) | 可讓您管理 Data Factory，但無法加以存取。 |
| [Data Lake Analytics 開發人員](#data-lake-analytics-developer) | 可讓您提交、監視及管理您自己的作業，但無法建立或刪除 Data Lake Analytics 帳戶。 |
| [資料清除者](#data-purger) | 可清除分析資料 |
| [DevTest Labs 使用者](#devtest-labs-user) | 可讓您連線、啟動、重新啟動及關閉 Azure DevTest Labs 中的虛擬機器。 |
| [DNS 區域參與者](#dns-zone-contributor) | 可讓您管理 Azure DNS 中的 DNS 區域與記錄集，但無法讓您控制誰可存取它們。 |
| [DocumentDB 帳戶參與者](#documentdb-account-contributor) | 可以管理 Azure Cosmos DB 帳戶。 Azure Cosmos DB 先前稱為 DocumentDB。 |
| [HDInsight 網域服務參與者](#hdinsight-domain-services-contributor) | 可讀取、建立、修改和刪除 HDInsight 企業安全性套件所需的網域服務相關作業 |
| [Intelligent Systems 帳戶參與者](#intelligent-systems-account-contributor) | 可讓您管理「智慧型系統」帳戶，但無法存取它們。 |
| [Key Vault 參與者](#key-vault-contributor) | 可讓您管理金鑰保存庫，但無法存取它們。 |
| [實驗室建立者](#lab-creator) | 可讓您在「Azure 實驗室帳戶」下建立、管理、刪除您的受控實驗室。 |
| [Log Analytics 參與者](#log-analytics-contributor) | 「Log Analytics 參與者」角色可以讀取所有監視資料和編輯監視設定。 編輯監視設定包括將 VM 延伸模組新增至 VM、讀取儲存體帳戶金鑰以便能夠設定從「Azure 儲存體」收集記錄、建立及設定「自動化」帳戶、新增解決方案，以及設定所有 Azure 資源上的 Azure 診斷。 |
| [Log Analytics 讀者](#log-analytics-reader) | 「Log Analytics 讀者」可以檢視和搜尋所有監視資料，以及檢視監視設定，包括檢視所有 Azure 資源上的 Azure 診斷設定。 |
| [邏輯應用程式參與者](#logic-app-contributor) | 可讓您管理邏輯應用程式，但無法存取它們。 |
| [邏輯應用程式運算子](#logic-app-operator) | 可讓您讀取、啟用及停用邏輯應用程式。 |
| [受控應用程式操作員角色](#managed-application-operator-role) | 可讓您讀取受控應用程式資源及對其執行動作 |
| [受控應用程式讀者](#managed-applications-reader) | 可讓您讀取受控應用程式中的資源及要求 JIT 存取權。 |
| [受控身分識別參與者](#managed-identity-contributor) | 建立、讀取、更新及刪除使用者指派的身分識別 |
| [受控身分識別操作員](#managed-identity-operator) | 讀取及指派使用者指派的身分識別 |
| [管理群組參與者](#management-group-contributor) | 管理群組參與者角色 |
| [管理群組讀者](#management-group-reader) | 管理群組讀者角色 |
| [監視參與者](#monitoring-contributor) | 可以讀取所有監視資料並編輯監視設定。 請參閱[開始使用 Azure 監視器的角色、權限和安全性](../monitoring-and-diagnostics/monitoring-roles-permissions-security.md#built-in-monitoring-roles)。 |
| [監視計量發行者](#monitoring-metrics-publisher) | 針對 Azure 資源啟用發佈計量 |
| [監視讀取器](#monitoring-reader) | 可以讀取所有監視資料 (計量、記錄檔等等)。 請參閱[開始使用 Azure 監視器的角色、權限和安全性](../monitoring-and-diagnostics/monitoring-roles-permissions-security.md#built-in-monitoring-roles)。 |
| [網路參與者](#network-contributor) | 可讓您管理網路，但無法存取它們。 |
| [New Relic APM 帳戶參與者](#new-relic-apm-account-contributor) | 可讓您管理 New Relic Application Performance Management 帳戶及應用程式，但無法存取它們。 |
| [PowerApps 系統管理員測試](#powerapps-administrator-test) | 可讓您管理 Azure DNS 中的 DNS 區域與記錄集，但無法讓您控制誰可存取它們。 |
| [讀取者及資料存取](#reader-and-data-access) | 可讓您檢視所有內容，但無法讓您刪除或建立儲存體帳戶或內含的資源。 也可透過存取儲存體帳戶金鑰，對儲存體帳戶中內含的所有資料進行讀取/寫入存取。 |
| [Redis 快取參與者](#redis-cache-contributor) | 可讓您管理 Redis 快取，但無法存取它們。 |
| [資源原則參與者 (預覽)](#resource-policy-contributor-preview) | (預覽) 從 EA 回填的使用者，有權建立/修改資源原則、建立支援票證及讀取資源/階層。 |
| [排程器工作集合參與者](#scheduler-job-collections-contributor) | 可讓您管理「排程器」工作集合，但無法存取它們。 |
| [搜尋服務參與者](#search-service-contributor) | 可讓您管理「搜尋」服務，但無法存取它們。 |
| [安全性系統管理員](#security-admin) | 僅限資訊安全中心：可檢視安全性原則、檢視安全性狀態、編輯安全性原則、檢視警示和建議、關閉警示和建議 |
| [安全性管理員](#security-manager) | 可讓您管理安全性元件、安全性原則及虛擬機器 |
| [安全性讀取者](#security-reader) | 僅限資訊安全中心：可檢視建議和警示、檢視安全性原則、檢視安全性狀態，但無法進行變更 |
| [Site Recovery 參與者](#site-recovery-contributor) | 可讓您管理 Site Recovery 服務，但無法建立保存庫和指派角色 |
| [Site Recovery 操作員](#site-recovery-operator) | 可讓您容錯移轉及容錯回復，但無法執行其他 Site Recovery 管理作業 |
| [Site Recovery 讀取者](#site-recovery-reader) | 可讓您檢視 Site Recovery 狀態，但無法執行其他管理作業 |
| [SQL DB 參與者](#sql-db-contributor) | 可讓您管理 SQL 資料庫，但無法存取它們。 此外，您也無法管理其安全性相關原則或其父 SQL 伺服器。 |
| [SQL 安全性管理員](#sql-security-manager) | 可讓您管理 SQL 伺服器及資料庫的安全性相關原則，但無法存取它們。 |
| [SQL Server 參與者](#sql-server-contributor) | 可讓您管理 SQL 伺服器及資料庫，但無法存取它們，也無法存取其安全性相關原則。 |
| [儲存體帳戶參與者](#storage-account-contributor) | 可讓您管理儲存體帳戶，但無法存取它們。 |
| [儲存體帳戶金鑰操作員服務角色](#storage-account-key-operator-service-role) | 「儲存體帳戶金鑰操作員」可以列出及重新產生「儲存體帳戶」的金鑰 |
| [儲存體 Blob 資料參與者 (預覽)](#storage-blob-data-contributor-preview) | 可讀取、寫入及刪除 Azure 儲存體 Blob 容器與資料 |
| [儲存體 Blob 資料讀者 (預覽)](#storage-blob-data-reader-preview) | 可讀取 Azure 儲存體 Blob 容器與資料 |
| [儲存體佇列資料參與者 (預覽)](#storage-queue-data-contributor-preview) | 可讀取、寫入及刪除 Azure 儲存體佇列與佇列訊息 |
| [儲存體佇列資料讀者 (預覽)](#storage-queue-data-reader-preview) | 可讀取 Azure 儲存體佇列與佇列訊息 |
| [支援要求參與者](#support-request-contributor) | 可讓您建立及管理支援要求 |
| [流量管理員參與者](#traffic-manager-contributor) | 可讓您管理「流量管理員」設定檔，但無法控制誰可以存取它們。 |
| [使用者存取系統管理員](#user-access-administrator) | 可讓您管理 Azure 資源的使用者存取。 |
| [虛擬機器系統管理員登入](#virtual-machine-administrator-login) | 在入口網站中檢視虛擬機器並以系統管理員身分登入 |
| [虛擬機器參與者](#virtual-machine-contributor) | 可讓您管理虛擬機器 (不含虛擬機器所連接的虛擬網路或儲存體帳戶)，但無法存取它們。 |
| [虛擬機器使用者登入](#virtual-machine-user-login) | 在入口網站中檢視虛擬機器並以一般使用者身分登入。 |
| [Web 方案參與者](#web-plan-contributor) | 可讓您管理網站的 Web 方案，但無法存取它們。 |
| [網站參與者](#website-contributor) | 可讓您管理網站 (非 Web 方案)，但無法存取它們。 |


## <a name="owner"></a>擁有者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理一切，包括對資源的存取。 |
> | **Id** | 8e3af657-a8ff-443c-a75c-2fe8c4bcb635 |
> | **動作** |  |
> | * | 建立和管理所有類型的資源 |

## <a name="contributor"></a>參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理一切，但無法管理對資源的存取。 |
> | **Id** | b24988ac-6180-42a0-ab88-20f7382dd24c |
> | **動作** |  |
> | * | 建立和管理所有類型的資源 |
> | **NotActions** |  |
> | Microsoft.Authorization/*/Delete | 無法刪除角色和角色指派 |
> | Microsoft.Authorization/*/Write | 無法建立角色和角色指派 |
> | Microsoft.Authorization/elevateAccess/Action | 對呼叫者授與租用戶範圍的使用者存取系統管理員存取權 |
> | Microsoft.Blueprint/blueprintAssignments/write |  |
> | Microsoft.Blueprint/blueprintAssignments/delete |  |

## <a name="reader"></a>讀取者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您檢視所有項目，但是無法進行變更。 |
> | **Id** | acdd72a7-3385-48ef-bd42-f606fba81ae7 |
> | **動作** |  |
> | */read | 讀取密碼以外的所有類型的資源。 |

## <a name="acrimagesigner"></a>AcrImageSigner
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | ACR 影像簽署者 |
> | **Id** | 6cef56e8-d556-48e5-a04f-b8e64114680f |
> | **動作** |  |
> | Microsoft.ContainerRegistry/registries/*/read |  |
> | Microsoft.ContainerRegistry/registries/*/write |  |

## <a name="acrquarantinereader"></a>AcrQuarantineReader
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | ACR 隔離資料讀取者 |
> | **Id** | cdda3590-29a3-44f6-95f2-9f980659eb04 |
> | **動作** |  |
> | Microsoft.ContainerRegistry/registries/*/read |  |

## <a name="acrquarantinewriter"></a>AcrQuarantineWriter
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | ACR 隔離資料寫入者 |
> | **Id** | c8d4ff99-41c3-41a8-9f60-21dfdad59608 |
> | **動作** |  |
> | Microsoft.ContainerRegistry/registries/*/write |  |
> | Microsoft.ContainerRegistry/registries/*/read |  |

## <a name="api-management-service-contributor"></a>API 管理服務參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 API 管理服務，但無法加以存取。 |
> | **Id** | 312a565d-c81f-4fd8-895a-4e21e48d571c |
> | **動作** |  |
> | Microsoft.ApiManagement/service/* | 建立和管理 API 管理服務 |
> | Microsoft.Authorization/*/read | 讀取授權 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="api-management-service-operator-role"></a>API 管理服務操作員角色
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可管理服務，但無法管理 API |
> | **Id** | e022efe7-f5ba-4159-bbe4-b44f577e9b61 |
> | **動作** |  |
> | Microsoft.ApiManagement/service/*/read | 讀取 API 管理服務執行個體 |
> | Microsoft.ApiManagement/service/backup/action | 將 API 管理服務備份到使用者所提供之儲存體帳戶中的指定容器 |
> | Microsoft.ApiManagement/service/delete | 刪除 API 管理服務執行個體 |
> | Microsoft.ApiManagement/service/managedeployments/action | 變更 SKU/單位、新增/移除 API 管理服務的區域部署 |
> | Microsoft.ApiManagement/service/read | 讀取 API 管理服務執行個體的中繼資料 |
> | Microsoft.ApiManagement/service/restore/action | 從使用者所提供之儲存體帳戶中的指定容器來還原 API 管理服務 |
> | Microsoft.ApiManagement/service/updatecertificate/action | 上傳 API 管理服務的 SSL 憑證 |
> | Microsoft.ApiManagement/service/updatehostname/action | 設定、更新或移除 API 管理服務的自訂網域名稱 |
> | Microsoft.ApiManagement/service/write | 建立 API 管理服務的新執行個體 |
> | Microsoft.Authorization/*/read | 讀取授權 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | **NotActions** |  |
> | Microsoft.ApiManagement/service/users/keys/read | 取得使用者金鑰的清單 |

## <a name="api-management-service-reader-role"></a>API 管理服務讀取者角色
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 具有服務與 API 的唯讀存取權 |
> | **Id** | 71522526-b88f-4d52-b57f-d31fc3546d0d |
> | **動作** |  |
> | Microsoft.ApiManagement/service/*/read | 讀取 API 管理服務執行個體 |
> | Microsoft.ApiManagement/service/read | 讀取 API 管理服務執行個體的中繼資料 |
> | Microsoft.Authorization/*/read | 讀取授權 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | **NotActions** |  |
> | Microsoft.ApiManagement/service/users/keys/read | 取得使用者金鑰的清單 |

## <a name="application-insights-component-contributor"></a>Application Insights 元件參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可以管理 Application Insights 元件 |
> | **Id** | ae349356-3a1b-4a5e-921d-050484c6347e |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.Insights/components/* | 建立和管理 Insights 元件 |
> | Microsoft.Insights/webtests/* | 建立和管理 Web 測試 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="application-insights-snapshot-debugger"></a>Application Insights 快照集偵錯工具
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 給予使用者權限，以便檢視及下載使用 Application Insights 快照偵錯工具所收集的偵錯快照。 請注意，[擁有者](#owner)或[參與者](#contributor)角色未包含這些權限。 |
> | **Id** | 08954f03-6346-4c2e-81c0-ec3a5cfae23b |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Insights/components/*/read |  |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="automation-job-operator"></a>自動化作業運算子
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 使用「自動化 Runbook」來建立及管理作業。 |
> | **Id** | 4fe576fe-1146-4730-92eb-48519fa6bf9f |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Automation/automationAccounts/hybridRunbookWorkerGroups/read | 讀取混合式 Runbook 背景工作角色資源 |
> | Microsoft.Automation/automationAccounts/jobs/read | 取得 Azure 自動化作業 |
> | Microsoft.Automation/automationAccounts/jobs/resume/action | 繼續 Azure 自動化作業 |
> | Microsoft.Automation/automationAccounts/jobs/stop/action | 停止 Azure 自動化作業 |
> | Microsoft.Automation/automationAccounts/jobs/streams/read | 取得 Azure 自動化作業串流 |
> | Microsoft.Automation/automationAccounts/jobs/suspend/action | 暫止 Azure 自動化作業 |
> | Microsoft.Automation/automationAccounts/jobs/write | 建立 Azure 自動化作業 |
> | Microsoft.Automation/automationAccounts/jobs/output/read | 取得作業的輸出 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="automation-operator"></a>自動化運算子
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 「自動化運算子」能夠啟動、停止、暫止及繼續作業 |
> | **Id** | d3881f73-407a-4167-8283-e981cbba0404 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Automation/automationAccounts/hybridRunbookWorkerGroups/read | 讀取混合式 Runbook 背景工作角色資源 |
> | Microsoft.Automation/automationAccounts/jobs/read | 取得 Azure 自動化作業 |
> | Microsoft.Automation/automationAccounts/jobs/resume/action | 繼續 Azure 自動化作業 |
> | Microsoft.Automation/automationAccounts/jobs/stop/action | 停止 Azure 自動化作業 |
> | Microsoft.Automation/automationAccounts/jobs/streams/read | 取得 Azure 自動化作業串流 |
> | Microsoft.Automation/automationAccounts/jobs/suspend/action | 暫止 Azure 自動化作業 |
> | Microsoft.Automation/automationAccounts/jobs/write | 建立 Azure 自動化作業 |
> | Microsoft.Automation/automationAccounts/jobSchedules/read | 取得 Azure 自動化作業排程 |
> | Microsoft.Automation/automationAccounts/jobSchedules/write | 建立 Azure 自動化作業排程 |
> | Microsoft.Automation/automationAccounts/linkedWorkspace/read | 取得連結至自動化帳戶的工作區 |
> | Microsoft.Automation/automationAccounts/read | 取得 Azure 自動化帳戶 |
> | Microsoft.Automation/automationAccounts/runbooks/read | 取得 Azure 自動化 Runbook |
> | Microsoft.Automation/automationAccounts/schedules/read | 取得 Azure 自動化排程資產 |
> | Microsoft.Automation/automationAccounts/schedules/write | 建立或更新 Azure 自動化排程資產 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Automation/automationAccounts/jobs/output/read | 取得作業的輸出 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="automation-runbook-operator"></a>自動化 Runbook 運算子
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 讀取 Runbook 屬性 - 以便能夠建立 Runbook 的作業。 |
> | **Id** | 5fb5aef8-1081-4b8e-bb16-9d5d0385bab5 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Automation/automationAccounts/runbooks/read | 取得 Azure 自動化 Runbook |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="azure-kubernetes-service-cluster-admin-role"></a>Azure Kubernetes Service 叢集管理員角色
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 列出叢集管理員認證動作。 |
> | **Id** | 0ab0b1a8-8aac-4efd-b8c2-3ee1fb270be8 |
> | **動作** |  |
> | Microsoft.ContainerService/managedClusters/listClusterAdminCredential/action | 列出受控叢集的 clusterAdmin 認證 |

## <a name="azure-kubernetes-service-cluster-user-role"></a>Azure Kubernetes Service 叢集使用者角色
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 列出叢集使用者認證動作。 |
> | **Id** | 4abbcc35-e782-43d8-92c5-2d3f1bd2253f |
> | **動作** |  |
> | Microsoft.ContainerService/managedClusters/listClusterUserCredential/action | 列出受控叢集的 clusterUser 認證 |

## <a name="azure-stack-registration-owner"></a>Azure Stack 註冊擁有者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 Azure Stack 註冊。 |
> | **Id** | 6f12a6df-dd06-4f3e-bcb1-ce8be600526a |
> | **動作** |  |
> | Microsoft.AzureStack/registrations/products/listDetails/action | 擷取 Azure Stack Marketplace 產品的延伸詳細資料 |
> | Microsoft.AzureStack/registrations/products/read | 取得 Azure Stack Marketplace 產品的屬性 |
> | Microsoft.AzureStack/registrations/read | 取得 Azure Stack 註冊的屬性 |

## <a name="backup-contributor"></a>備份參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理備份服務，但無法建立保存庫及為其他人提供存取權 |
> | **Id** | 5e467623-bb1f-42f4-a55d-6e525e11384b |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Network/virtualNetworks/read | 取得虛擬網路定義 |
> | Microsoft.RecoveryServices/locations/* |  |
> | Microsoft.RecoveryServices/locations/allocatedStamp/read | GetAllocatedStamp 是服務所使用的內部作業 |
> | Microsoft.RecoveryServices/Vaults/backupconfig/vaultconfig/* |  |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/operationResults/* | 管理備份管理上作業的結果 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/* | 在復原服務保存庫的備份網狀架構內建立和管理備份容器 |
> | Microsoft.RecoveryServices/Vaults/backupJobs/* | 建立和管理備份作業 |
> | Microsoft.RecoveryServices/Vaults/backupJobsExport/action | 匯出作業 |
> | Microsoft.RecoveryServices/Vaults/backupJobsExport/operationResults/read | 傳回匯出作業的作業結果。 |
> | Microsoft.RecoveryServices/Vaults/backupManagementMetaData/* | 建立和管理與備份管理相關的中繼資料 |
> | Microsoft.RecoveryServices/Vaults/backupOperationResults/* | 建立和管理備份管理作業的結果 |
> | Microsoft.RecoveryServices/Vaults/backupPolicies/* | 建立和管理備份原則 |
> | Microsoft.RecoveryServices/Vaults/backupProtectableItems/* | 建立和管理可以備份的項目 |
> | Microsoft.RecoveryServices/Vaults/backupProtectedItems/* | 建立和管理備份項目 |
> | Microsoft.RecoveryServices/Vaults/backupProtectionContainers/* | 建立和管理保存備份項目的容器 |
> | Microsoft.RecoveryServices/Vaults/backupSecurityPIN/* |  |
> | Microsoft.RecoveryServices/Vaults/backupUsageSummaries/read | 傳回復原服務之受保護項目和受保護伺服器的摘要。 |
> | Microsoft.RecoveryServices/Vaults/certificates/* | 建立和管理備份復原服務保存庫中與備份相關的憑證 |
> | Microsoft.RecoveryServices/Vaults/extendedInformation/* | 建立和管理與保存庫相關的擴充資訊 |
> | Microsoft.RecoveryServices/Vaults/monitoringAlerts/read | 取得復原服務保存庫的警示。 |
> | Microsoft.RecoveryServices/Vaults/monitoringConfigurations/* |  |
> | Microsoft.RecoveryServices/Vaults/read | 「取得保存庫」作業會取得物件，此物件代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/refreshContainers/action | 重新整理容器清單 |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/* | 建立和管理註冊的身分識別 |
> | Microsoft.RecoveryServices/Vaults/storageConfig/* |  |
> | Microsoft.RecoveryServices/Vaults/usages/* | 建立和管理復原服務保存庫的使用方式 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Storage/storageAccounts/read | 傳回儲存體帳戶清單，或取得指定儲存體帳戶的屬性。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="backup-operator"></a>備份操作員
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理備份服務，但無法移除備份、建立保存庫及為其他人提供存取權 |
> | **Id** | 00c29273-979b-4161-815c-10b084fb9324 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Network/virtualNetworks/read | 取得虛擬網路定義 |
> | Microsoft.RecoveryServices/locations/allocatedStamp/read | GetAllocatedStamp 是服務所使用的內部作業 |
> | Microsoft.RecoveryServices/Vaults/backupconfig/vaultconfig/* |  |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/operationResults/read | 傳回作業的狀態 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/operationResults/read | 取得對保護容器執行之作業的結果。 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/backup/action | 對受保護的項目執行備份。 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/operationResults/read | 取得對受保護項目執行之作業的結果。 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/operationsStatus/read | 傳回對受保護項目執行之作業的狀態。 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/read | 傳回受保護項目的物件詳細資料 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/provisionInstantItemRecovery/action | 為受保護的項目佈建即時項目復原 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/read | 取得受保護項目的復原點。 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/restore/action | 還原受保護項目的復原點。 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/revokeInstantItemRecovery/action | 為受保護的項目撤銷即時項目復原 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/write | 建立備用的受保護項目 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/read | 傳回所有已註冊的容器 |
> | Microsoft.RecoveryServices/Vaults/backupJobs/* | 建立和管理備份作業 |
> | Microsoft.RecoveryServices/Vaults/backupJobs/cancel/action | 取消作業 |
> | Microsoft.RecoveryServices/Vaults/backupJobs/operationResults/read | 傳回作業的作業結果。 |
> | Microsoft.RecoveryServices/Vaults/backupJobs/read | 傳回所有作業物件 |
> | Microsoft.RecoveryServices/Vaults/backupJobsExport/action | 匯出作業 |
> | Microsoft.RecoveryServices/Vaults/backupJobsExport/operationResults/read | 傳回匯出作業的作業結果。 |
> | Microsoft.RecoveryServices/Vaults/backupManagementMetaData/read | 傳回復原服務保存庫的備份管理中繼資料。 |
> | Microsoft.RecoveryServices/Vaults/backupOperationResults/* | 建立和管理備份管理作業的結果 |
> | Microsoft.RecoveryServices/Vaults/backupPolicies/operationResults/read | 取得原則作業的結果。 |
> | Microsoft.RecoveryServices/Vaults/backupPolicies/operationStatus/read |  |
> | Microsoft.RecoveryServices/Vaults/backupPolicies/read | 傳回所有保護原則 |
> | Microsoft.RecoveryServices/Vaults/backupProtectableItems/* | 建立和管理可以備份的項目 |
> | Microsoft.RecoveryServices/Vaults/backupProtectableItems/read | 傳回所有可保護項目的清單。 |
> | Microsoft.RecoveryServices/Vaults/backupProtectedItems/read | 傳回所有受保護項目的清單。 |
> | Microsoft.RecoveryServices/Vaults/backupProtectionContainers/read | 傳回屬於訂用帳戶的所有容器 |
> | Microsoft.RecoveryServices/Vaults/backupUsageSummaries/read | 傳回復原服務之受保護項目和受保護伺服器的摘要。 |
> | Microsoft.RecoveryServices/Vaults/certificates/write | 「更新資源憑證」作業會更新資源/保存庫的認證憑證。 |
> | Microsoft.RecoveryServices/Vaults/extendedInformation/read | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/extendedInformation/write | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/monitoringAlerts/read | 取得復原服務保存庫的警示。 |
> | Microsoft.RecoveryServices/Vaults/monitoringConfigurations/* |  |
> | Microsoft.RecoveryServices/Vaults/read | 「取得保存庫」作業會取得物件，此物件代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/refreshContainers/action | 重新整理容器清單 |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/operationResults/read | 「取得作業結果」作業可用來取得以非同步方式提交之作業的作業狀態和結果 |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/read | 「取得容器」作業可用來取得為資源註冊的容器。 |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/write | 「註冊服務容器」作業可用來向復原服務註冊容器。 |
> | Microsoft.RecoveryServices/Vaults/storageConfig/* |  |
> | Microsoft.RecoveryServices/Vaults/usages/read | 傳回復原服務保存庫的使用量詳細資料。 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Storage/storageAccounts/read | 傳回儲存體帳戶清單，或取得指定儲存體帳戶的屬性。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="backup-reader"></a>備份讀取者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可以檢視備份服務，但無法進行變更 |
> | **Id** | a795c7a0-d4a2-40c1-ae25-d81f01202912 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/operationResults/read | 傳回作業的狀態 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/operationResults/read | 取得對保護容器執行之作業的結果。 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/operationResults/read | 取得對受保護項目執行之作業的結果。 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/operationsStatus/read | 傳回對受保護項目執行之作業的狀態。 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/read | 傳回受保護項目的物件詳細資料 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/read | 傳回所有已註冊的容器 |
> | Microsoft.RecoveryServices/Vaults/backupJobs/operationResults/read | 傳回作業的作業結果。 |
> | Microsoft.RecoveryServices/Vaults/backupJobs/read | 傳回所有作業物件 |
> | Microsoft.RecoveryServices/Vaults/backupJobsExport/action | 匯出作業 |
> | Microsoft.RecoveryServices/Vaults/backupManagementMetaData/read | 傳回復原服務保存庫的備份管理中繼資料。 |
> | Microsoft.RecoveryServices/Vaults/backupOperationResults/read | 傳回復原服務保存庫的備份作業結果。 |
> | Microsoft.RecoveryServices/Vaults/backupPolicies/operationResults/read | 取得原則作業的結果。 |
> | Microsoft.RecoveryServices/Vaults/backupPolicies/read | 傳回所有保護原則 |
> | Microsoft.RecoveryServices/Vaults/backupProtectedItems/read | 傳回所有受保護項目的清單。 |
> | Microsoft.RecoveryServices/Vaults/backupProtectionContainers/read | 傳回屬於訂用帳戶的所有容器 |
> | Microsoft.RecoveryServices/Vaults/backupUsageSummaries/read | 傳回復原服務之受保護項目和受保護伺服器的摘要。 |
> | Microsoft.RecoveryServices/Vaults/extendedInformation/read | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/read | 「取得保存庫」作業會取得物件，此物件代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/refreshContainers/read |  |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/operationResults/read | 「取得作業結果」作業可用來取得以非同步方式提交之作業的作業狀態和結果 |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/read | 「取得容器」作業可用來取得為資源註冊的容器。 |
> | Microsoft.RecoveryServices/locations/allocatedStamp/read | GetAllocatedStamp 是服務所使用的內部作業 |
> | Microsoft.RecoveryServices/Vaults/monitoringConfigurations/notificationConfiguration/read |  |
> | Microsoft.RecoveryServices/Vaults/monitoringAlerts/read | 取得復原服務保存庫的警示。 |
> | Microsoft.RecoveryServices/Vaults/storageConfig/read |  |
> | Microsoft.RecoveryServices/Vaults/backupconfig/vaultconfig/read |  |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/read | 取得受保護項目的復原點。 |
> | Microsoft.RecoveryServices/Vaults/backupJobsExport/operationResults/read | 傳回匯出作業的作業結果。 |
> | Microsoft.RecoveryServices/Vaults/usages/read | 傳回復原服務保存庫的使用量詳細資料。 |

## <a name="billing-reader"></a>帳單讀取器
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您讀取計費資料 |
> | **Id** | fa23ad8b-c56e-40d8-ac0c-ce449e1d2c64 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Billing/*/read | 讀取帳單資訊 |
> | Microsoft.Commerce/*/read |  |
> | Microsoft.Consumption/*/read |  |
> | Microsoft.Management/managementGroups/read | 列出已驗證之使用者的管理群組。 |
> | Microsoft.CostManagement/*/read |  |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="biztalk-contributor"></a>BizTalk 參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 BizTalk 服務，但無法存取它們。 |
> | **Id** | 5e3c6656-6cfa-4708-81fe-0de47ac73342 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.BizTalkServices/BizTalk/* | 建立和管理 BizTalk 服務 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="cdn-endpoint-contributor"></a>CDN 端點參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可管理 CDN 端點，但無法對其他使用者授與存取權。 |
> | **Id** | 426e0c7f-0c7e-4658-b36f-ff54d6c29b45 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Cdn/edgenodes/read |  |
> | Microsoft.Cdn/operationresults/* |  |
> | Microsoft.Cdn/profiles/endpoints/* |  |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="cdn-endpoint-reader"></a>CDN 端點讀者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可檢視 CDN 端點，但無法進行變更。 |
> | **Id** | 871e35f6-b5c1-49cc-a043-bde969a0f2cd |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Cdn/edgenodes/read |  |
> | Microsoft.Cdn/operationresults/* |  |
> | Microsoft.Cdn/profiles/endpoints/*/read |  |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="cdn-profile-contributor"></a>CDN 設定檔參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可管理 CDN 設定檔及其端點，但無法對其他使用者授與存取權。 |
> | **Id** | ec156ff8-a8d1-4d15-830c-5b80698ca432 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Cdn/edgenodes/read |  |
> | Microsoft.Cdn/operationresults/* |  |
> | Microsoft.Cdn/profiles/* |  |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="cdn-profile-reader"></a>CDN 設定檔讀者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可檢視 CDN 設定檔及其端點，但無法進行變更。 |
> | **Id** | 8f96442b-4075-438f-813d-ad51ab4019af |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Cdn/edgenodes/read |  |
> | Microsoft.Cdn/operationresults/* |  |
> | Microsoft.Cdn/profiles/*/read |  |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="classic-network-contributor"></a>傳統網路參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理傳統網路，但無法存取它們。 |
> | **Id** | b34d265f-36f7-4a0d-a4d4-e158ca92e90f |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取授權 |
> | Microsoft.ClassicNetwork/* | 建立和管理傳統網路 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="classic-storage-account-contributor"></a>傳統儲存體帳戶參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理傳統儲存體帳戶，但無法存取它們。 |
> | **Id** | 86e8f5dc-a6e9-4c67-9d15-de283e8eac25 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取授權 |
> | Microsoft.ClassicStorage/storageAccounts/* | 建立及管理儲存體帳戶 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="classic-storage-account-key-operator-service-role"></a>傳統儲存體帳戶金鑰操作員服務角色
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 「傳統儲存體帳戶金鑰操作員」可以列出及重新產生「傳統儲存體帳戶」的金鑰 |
> | **Id** | 985d6b00-f706-48f5-a6fe-d0ca12fb668d |
> | **動作** |  |
> | Microsoft.ClassicStorage/storageAccounts/listkeys/action | 列出儲存體帳戶的存取金鑰。 |
> | Microsoft.ClassicStorage/storageAccounts/regeneratekey/action | 重新產生儲存體帳戶的現有存取金鑰。 |

## <a name="classic-virtual-machine-contributor"></a>傳統虛擬機器參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理傳統虛擬機器 (不含虛擬機器所連線的虛擬網路或儲存體帳戶)，但無法存取它們。 |
> | **Id** | d73bb868-a0df-4d4d-bd69-98a00b01fccb |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取授權 |
> | Microsoft.ClassicCompute/domainNames/* | 建立和管理傳統運算網域名稱 |
> | Microsoft.ClassicCompute/virtualMachines/* | 建立和管理虛擬機器 |
> | Microsoft.ClassicNetwork/networkSecurityGroups/join/action |  |
> | Microsoft.ClassicNetwork/reservedIps/link/action | 連結保留的 IP |
> | Microsoft.ClassicNetwork/reservedIps/read | 取得保留的 IP |
> | Microsoft.ClassicNetwork/virtualNetworks/join/action | 加入虛擬網路。 |
> | Microsoft.ClassicNetwork/virtualNetworks/read | 取得虛擬網路。 |
> | Microsoft.ClassicStorage/storageAccounts/disks/read | 傳回儲存體帳戶磁碟。 |
> | Microsoft.ClassicStorage/storageAccounts/images/read | 傳回儲存體帳戶映像。 (已淘汰。 使用 'Microsoft.ClassicStorage/storageAccounts/vmImages') |
> | Microsoft.ClassicStorage/storageAccounts/listKeys/action | 列出儲存體帳戶的存取金鑰。 |
> | Microsoft.ClassicStorage/storageAccounts/read | 傳回具有給定帳戶的儲存體帳戶。 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="cognitive-services-contributor"></a>認知服務參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您建立、讀取、更新、刪除及管理認知服務的金鑰。 |
> | **Id** | 25fbc0a9-bd7c-42a3-aa1a-3b75d497ee68 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.CognitiveServices/* |  |
> | Microsoft.Features/features/read | 取得訂用帳戶的功能。 |
> | Microsoft.Features/providers/features/read | 取得給定資源提供者中某個訂用帳戶的功能。 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Insights/diagnosticSettings/* | 建立、更新或讀取 Analysis Server 的診斷設定 |
> | Microsoft.Insights/logDefinitions/read | 讀取記錄定義 |
> | Microsoft.Insights/metricdefinitions/read | 讀取計量定義 |
> | Microsoft.Insights/metrics/read | 讀取計量 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/deployments/operations/read | 取得或列出部署作業。 |
> | Microsoft.Resources/subscriptions/operationresults/read | 取得訂用帳戶作業結果。 |
> | Microsoft.Resources/subscriptions/read | 取得訂用帳戶清單。 |
> | Microsoft.Resources/subscriptions/resourcegroups/deployments/* |  |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="cognitive-services-user"></a>認知服務使用者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您讀取和列出認知服務的金鑰。 |
> | **Id** | a97b65f3-24c7-4388-baec-2e87135dc908 |
> | **動作** |  |
> | Microsoft.CognitiveServices/*/read |  |
> | Microsoft.CognitiveServices/accounts/listkeys/action | 列出金鑰 |
> | Microsoft.Insights/metricdefinitions/read | 讀取計量定義 |
> | Microsoft.Insights/metrics/read | 讀取計量 |
> | Microsoft.Insights/alertRules/read | 讀取傳統計量警示 |
> | Microsoft.Insights/diagnosticSettings/read | 讀取資源診斷設定 |
> | Microsoft.Insights/logDefinitions/read | 讀取記錄定義 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/operations/read | 取得或列出部署作業。 |
> | Microsoft.Resources/subscriptions/operationresults/read | 取得訂用帳戶作業結果。 |
> | Microsoft.Resources/subscriptions/read | 取得訂用帳戶清單。 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="cosmos-db-account-reader-role"></a>Cosmos DB 帳戶讀者角色
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可以讀取 Azure Cosmos DB 帳戶資料。 請參閱 [DocumentDB 帳戶參與者](#documentdb-account-contributor)以管理 Azure Cosmos DB 帳戶。 |
> | **Id** | fbdf93bf-df7d-467e-a4d2-9458aa1360c8 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派，可以讀取提供給每個使用者的權限 |
> | Microsoft.DocumentDB/*/read | 讀取任何集合 |
> | Microsoft.DocumentDB/databaseAccounts/readonlykeys/action | 讀取資料庫帳戶的唯讀金鑰。 |
> | Microsoft.Insights/MetricDefinitions/read | 讀取計量定義 |
> | Microsoft.Insights/Metrics/read | 讀取計量 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="cost-management-contributor"></a>成本管理參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可檢視成本和管理成本組態 (例如預算、匯出) |
> | **Id** | 434105ed-43f6-45c7-a02f-909b2ba83430 |
> | **動作** |  |
> | Microsoft.Consumption/* |  |
> | Microsoft.CostManagement/* |  |
> | Microsoft.Billing/billingPeriods/read | 列出可用的計費週期 |
> | Microsoft.Resources/subscriptions/read | 取得訂用帳戶清單。 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="cost-management-reader"></a>成本管理讀者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可檢視成本資料和組態 (例如預算、匯出) |
> | **Id** | 72fafb9e-0641-4937-9268-a91bfd8191a3 |
> | **動作** |  |
> | Microsoft.Consumption/*/read |  |
> | Microsoft.CostManagement/*/read |  |
> | Microsoft.Billing/billingPeriods/read | 列出可用的計費週期 |
> | Microsoft.Resources/subscriptions/read | 取得訂用帳戶清單。 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="data-box-contributor"></a>資料箱參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理資料箱服務下的所有項目，為他人賦予存取權除外。 |
> | **Id** | add466c9-e687-43fc-8d98-dfcf8d720be5 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | Microsoft.Databox/* |  |

## <a name="data-box-reader"></a>資料箱讀者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理資料箱服務，建立訂單或編輯訂單詳細資料和為他人賦予存取權除外。 |
> | **Id** | 028f4ed7-e2a9-465e-a8f4-9c0ffdfdc027 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Databox/*/read |  |
> | Microsoft.Databox/jobs/listsecrets/action |  |
> | Microsoft.Databox/jobs/listcredentials/action | 列出與訂單相關的未加密認證。 |
> | Microsoft.Databox/locations/availableSkus/action | 此方法會傳回可用的 SKU 清單。 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="data-factory-contributor"></a>Data Factory 參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 Data Factory，但無法加以存取。 |
> | **Id** | 673868aa-7521-48a0-acc6-0f60742d39f5 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.DataFactory/dataFactories/* | 建立和管理 Data Factory 以及其中的子資源。 |
> | Microsoft.DataFactory/factories/* | 建立和管理 Data Factory 以及其中的子資源。 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="data-lake-analytics-developer"></a>Data Lake Analytics 開發人員
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您提交、監視及管理您自己的作業，但無法建立或刪除 Data Lake Analytics 帳戶。 |
> | **Id** | 47b7735b-770e-4598-a7da-8b91488b4c88 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.BigAnalytics/accounts/* |  |
> | Microsoft.DataLakeAnalytics/accounts/* |  |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | **NotActions** |  |
> | Microsoft.BigAnalytics/accounts/Delete |  |
> | Microsoft.BigAnalytics/accounts/TakeOwnership/action |  |
> | Microsoft.BigAnalytics/accounts/Write |  |
> | Microsoft.DataLakeAnalytics/accounts/Delete | 刪除 DataLakeAnalytics 帳戶。 |
> | Microsoft.DataLakeAnalytics/accounts/TakeOwnership/action | 授與權限以取消其他使用者所提交的作業。 |
> | Microsoft.DataLakeAnalytics/accounts/Write | 建立或更新 DataLakeAnalytics 帳戶。 |
> | Microsoft.DataLakeAnalytics/accounts/dataLakeStoreAccounts/Write | 建立或更新 DataLakeAnalytics 帳戶所連結的 DataLakeStore 帳戶。 |
> | Microsoft.DataLakeAnalytics/accounts/dataLakeStoreAccounts/Delete | 取消 DataLakeStore 帳戶與 DataLakeAnalytics 帳戶的連結。 |
> | Microsoft.DataLakeAnalytics/accounts/storageAccounts/Write | 建立或更新 DataLakeAnalytics 帳戶所連結的儲存體帳戶。 |
> | Microsoft.DataLakeAnalytics/accounts/storageAccounts/Delete | 取消儲存體帳戶與 DataLakeAnalytics 帳戶的連結。 |
> | Microsoft.DataLakeAnalytics/accounts/firewallRules/Write | 建立或更新防火牆規則。 |
> | Microsoft.DataLakeAnalytics/accounts/firewallRules/Delete | 刪除防火牆規則。 |
> | Microsoft.DataLakeAnalytics/accounts/computePolicies/Write | 建立或更新計算原則。 |
> | Microsoft.DataLakeAnalytics/accounts/computePolicies/Delete | 刪除計算原則。 |

## <a name="data-purger"></a>資料清除者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可清除分析資料 |
> | **Id** | 150f5e0c-0603-4f03-8c7f-cf70034c4e90 |
> | **動作** |  |
> | Microsoft.Insights/components/*/read |  |
> | Microsoft.Insights/components/purge/action | 從 Application Insights 清除資料 |
> | Microsoft.OperationalInsights/workspaces/*/read |  |
> | Microsoft.OperationalInsights/workspaces/purge/action | 從工作區刪除指定的資料 |

## <a name="devtest-labs-user"></a>DevTest Labs 使用者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您連線、啟動、重新啟動及關閉 Azure DevTest Labs 中的虛擬機器。 |
> | **Id** | 76283e04-6283-4c54-8f91-bcf1374a3c64 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Compute/availabilitySets/read | 取得可用性設定組的屬性 |
> | Microsoft.Compute/virtualMachines/*/read | 讀取虛擬機器的屬性 (VM 大小、執行階段狀態、VM 擴充功能等) |
> | Microsoft.Compute/virtualMachines/deallocate/action | 關閉虛擬機器的電源，並將計算資源釋出 |
> | Microsoft.Compute/virtualMachines/read | 取得虛擬機器的屬性 |
> | Microsoft.Compute/virtualMachines/restart/action | 重新啟動虛擬機器 |
> | Microsoft.Compute/virtualMachines/start/action | 啟動虛擬機器 |
> | Microsoft.DevTestLab/*/read | 讀取實驗室的屬性 |
> | Microsoft.DevTestLab/labs/createEnvironment/action | 在實驗室中建立虛擬機器。 |
> | Microsoft.DevTestLab/labs/claimAnyVm/action | 在實驗室中宣告隨機的可宣告虛擬機器。 |
> | Microsoft.DevTestLab/labs/formulas/delete | 刪除公式。 |
> | Microsoft.DevTestLab/labs/formulas/read | 讀取公式。 |
> | Microsoft.DevTestLab/labs/formulas/write | 新增或修改公式。 |
> | Microsoft.DevTestLab/labs/policySets/evaluatePolicies/action | 評估實驗室原則。 |
> | Microsoft.DevTestLab/labs/virtualMachines/claim/action | 取得現有虛擬機器的擁有權 |
> | Microsoft.Network/loadBalancers/backendAddressPools/join/action | 加入負載平衡器的後端位址集區 |
> | Microsoft.Network/loadBalancers/inboundNatRules/join/action | 加入負載平衡器的輸入 NAT 規則 |
> | Microsoft.Network/networkInterfaces/*/read | 讀取網路介面的屬性 (例如網路介面所屬的所有負載平衡器) |
> | Microsoft.Network/networkInterfaces/join/action | 將虛擬機器加入網路介面 |
> | Microsoft.Network/networkInterfaces/read | 取得網路介面定義。  |
> | Microsoft.Network/networkInterfaces/write | 建立網路介面，或更新現有的網路介面。  |
> | Microsoft.Network/publicIPAddresses/*/read | 讀取公用 IP 位址的屬性 |
> | Microsoft.Network/publicIPAddresses/join/action | 加入公用 IP 位址 |
> | Microsoft.Network/publicIPAddresses/read | 取得公用 IP 位址定義。 |
> | Microsoft.Network/virtualNetworks/subnets/join/action | 加入虛擬網路 |
> | Microsoft.Resources/deployments/operations/read | 取得或列出部署作業。 |
> | Microsoft.Resources/deployments/read | 取得或列出部署。 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Storage/storageAccounts/listKeys/action | 傳回指定儲存體帳戶的存取金鑰。 |
> | **NotActions** |  |
> | Microsoft.Compute/virtualMachines/vmSizes/read | 列出虛擬機器所能更新成的大小 |

## <a name="dns-zone-contributor"></a>DNS 區域參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 Azure DNS 中的 DNS 區域與記錄集，但無法讓您控制誰可存取它們。 |
> | **Id** | befefa01-2a29-4197-83a8-272ff33ce314 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.Network/dnsZones/* | 建立和管理 DNS 區域和記錄 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="documentdb-account-contributor"></a>DocumentDB 帳戶參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可以管理 Azure Cosmos DB 帳戶。 Azure Cosmos DB 先前稱為 DocumentDB。 |
> | **Id** | 5bd9cd88-fe45-4216-938b-f97437e15450 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.DocumentDb/databaseAccounts/* | 建立及管理 Azure Cosmos DB 帳戶 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="hdinsight-domain-services-contributor"></a>HDInsight 網域服務參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讀取、建立、修改和刪除 HDInsight 企業安全性套件所需的網域服務相關作業 |
> | **Id** | 8d8d5a11-05d3-4bda-a417-a08778121c7c |
> | **動作** |  |
> | Microsoft.AAD/*/read |  |
> | Microsoft.AAD/domainServices/*/read |  |
> | Microsoft.AAD/domainServices/oucontainer/* |  |

## <a name="intelligent-systems-account-contributor"></a>Intelligent Systems 帳戶參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理「智慧型系統」帳戶，但無法存取它們。 |
> | **Id** | 03a6d094-3444-4b3d-88af-7477090a9e5e |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.IntelligentSystems/accounts/* | 建立及管理 Intelligent Systems 帳戶 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="key-vault-contributor"></a>Key Vault 參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理金鑰保存庫，但無法存取它們。 |
> | **Id** | f25e0fa2-a7c8-4377-a976-54943a77a395 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.KeyVault/* |  |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | **NotActions** |  |
> | Microsoft.KeyVault/locations/deletedVaults/purge/action | 清除虛刪除的 Key Vault |
> | Microsoft.KeyVault/hsmPools/* |  |

## <a name="lab-creator"></a>實驗室建立者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您在「Azure 實驗室帳戶」下建立、管理、刪除您的受控實驗室。 |
> | **Id** | b97fb8bc-a8b2-4522-a38b-dd33c7e65ead |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.LabServices/labAccounts/*/read |  |
> | Microsoft.LabServices/labAccounts/createLab/action | 在實驗室帳戶中建立實驗室。 |
> | Microsoft.LabServices/labAccounts/sizes/getRegionalAvailability/action |  |
> | Microsoft.LabServices/labAccounts/getRegionalAvailability/action | 取得在實驗室帳戶下設定的每個大小類別的區域可用性資訊 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="log-analytics-contributor"></a>Log Analytics 參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 「Log Analytics 參與者」角色可以讀取所有監視資料和編輯監視設定。 編輯監視設定包括將 VM 延伸模組新增至 VM、讀取儲存體帳戶金鑰以便能夠設定從「Azure 儲存體」收集記錄、建立及設定「自動化」帳戶、新增解決方案，以及設定所有 Azure 資源上的 Azure 診斷。 |
> | **Id** | 92aaf0da-9dab-42b6-94a3-d43ce8d16293 |
> | **動作** |  |
> | */read | 讀取密碼以外的所有類型的資源。 |
> | Microsoft.Automation/automationAccounts/* |  |
> | Microsoft.ClassicCompute/virtualMachines/extensions/* |  |
> | Microsoft.ClassicStorage/storageAccounts/listKeys/action | 列出儲存體帳戶的存取金鑰。 |
> | Microsoft.Compute/virtualMachines/extensions/* |  |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Insights/diagnosticSettings/* | 建立、更新或讀取 Analysis Server 的診斷設定 |
> | Microsoft.OperationalInsights/* |  |
> | Microsoft.OperationsManagement/* |  |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourcegroups/deployments/* |  |
> | Microsoft.Storage/storageAccounts/listKeys/action | 傳回指定儲存體帳戶的存取金鑰。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="log-analytics-reader"></a>Log Analytics 讀者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 「Log Analytics 讀者」可以檢視和搜尋所有監視資料，以及檢視監視設定，包括檢視所有 Azure 資源上的 Azure 診斷設定。 |
> | **Id** | 73c42c96-874c-492b-b04d-ab87d138a893 |
> | **動作** |  |
> | */read | 讀取密碼以外的所有類型的資源。 |
> | Microsoft.OperationalInsights/workspaces/analytics/query/action | 使用新的引擎進行搜尋。 |
> | Microsoft.OperationalInsights/workspaces/search/action | 執行搜尋查詢 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | **NotActions** |  |
> | Microsoft.OperationalInsights/workspaces/sharedKeys/read | 擷取工作區的共用金鑰。 這些金鑰可用來將 Microsoft Operational Insights 代理程式連線到工作區。 |

## <a name="logic-app-contributor"></a>邏輯應用程式參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理邏輯應用程式，但無法存取它們。 |
> | **Id** | 87a39d53-fc1b-424a-814c-f7e04687dc9e |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.ClassicStorage/storageAccounts/listKeys/action | 列出儲存體帳戶的存取金鑰。 |
> | Microsoft.ClassicStorage/storageAccounts/read | 傳回具有給定帳戶的儲存體帳戶。 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Insights/diagnosticSettings/* | 建立、更新或讀取 Analysis Server 的診斷設定 |
> | Microsoft.Insights/logdefinitions/* | 此為使用者需要透過入口網站存取活動記錄檔時所需的權限。 列出活動記錄檔中的記錄檔分類。 |
> | Microsoft.Insights/metricDefinitions/* | 讀取度量定義 (可用資源的度量類型清單)。 |
> | Microsoft.Logic/* | 管理 Logic Apps 資源。 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/operationresults/read | 取得訂用帳戶作業結果。 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Storage/storageAccounts/listkeys/action | 傳回指定儲存體帳戶的存取金鑰。 |
> | Microsoft.Storage/storageAccounts/read | 傳回儲存體帳戶清單，或取得指定儲存體帳戶的屬性。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | Microsoft.Web/connectionGateways/* | 建立及管理「連線閘道」。 |
> | Microsoft.Web/connections/* | 建立及管理「連線」。 |
> | Microsoft.Web/customApis/* | 建立及管理「自訂 API」。 |
> | Microsoft.Web/serverFarms/join/action |  |
> | Microsoft.Web/serverFarms/read | 取得 App Service 方案的屬性 |
> | Microsoft.Web/sites/functions/listSecrets/action | 列出祕密 Web Apps 函式。 |

## <a name="logic-app-operator"></a>邏輯應用程式操作員
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您讀取、啟用及停用邏輯應用程式。 |
> | **Id** | 515c2055-d9d4-4321-b1b9-bd0c9a0f79fe |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/*/read | 讀取 Insights 警示規則 |
> | Microsoft.Insights/diagnosticSettings/*/read | 取得 Logic Apps 的診斷設定 |
> | Microsoft.Insights/metricDefinitions/*/read | 取得 Logic Apps 的可用計量。 |
> | Microsoft.Logic/*/read | 讀取 Logic Apps 資源。 |
> | Microsoft.Logic/workflows/disable/action | 停用工作流程。 |
> | Microsoft.Logic/workflows/enable/action | 啟用工作流程。 |
> | Microsoft.Logic/workflows/validate/action | 驗證工作流程。 |
> | Microsoft.Resources/deployments/operations/read | 取得或列出部署作業。 |
> | Microsoft.Resources/subscriptions/operationresults/read | 取得訂用帳戶作業結果。 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | Microsoft.Web/connectionGateways/*/read | 讀取「連線閘道」。 |
> | Microsoft.Web/connections/*/read | 讀取「連線」。 |
> | Microsoft.Web/customApis/*/read | 讀取「自訂 API」。 |
> | Microsoft.Web/serverFarms/read | 取得 App Service 方案的屬性 |

## <a name="managed-application-operator-role"></a>受控應用程式操作員角色
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您讀取受控應用程式資源及對其執行動作 |
> | **Id** | c7393b34-138c-406f-901b-d8cf2b17e6ae |
> | **動作** |  |
> | Microsoft.Solutions/applications/read | 擷取應用程式清單。 |

## <a name="managed-applications-reader"></a>受控應用程式讀者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您讀取受控應用程式中的資源及要求 JIT 存取權。 |
> | **Id** | b9331d33-8a36-4f8c-b097-4f54124fdb44 |
> | **動作** |  |
> | */read | 讀取密碼以外的所有類型的資源。 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Solutions/jitRequests/* |  |

## <a name="managed-identity-contributor"></a>受控身分識別參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 建立、讀取、更新及刪除使用者指派的身分識別 |
> | **Id** | e40ec5ca-96e0-45a2-b4ff-59039f2c2b59 |
> | **動作** |  |
> | Microsoft.ManagedIdentity/userAssignedIdentities/*/read |  |
> | Microsoft.ManagedIdentity/userAssignedIdentities/*/write |  |
> | Microsoft.ManagedIdentity/userAssignedIdentities/*/delete |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="managed-identity-operator"></a>受控身分識別操作員
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 讀取及指派使用者指派的身分識別 |
> | **Id** | f1a07417-d97a-45cb-824c-7a7467783830 |
> | **動作** |  |
> | Microsoft.ManagedIdentity/userAssignedIdentities/*/read |  |
> | Microsoft.ManagedIdentity/userAssignedIdentities/*/assign/action |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="management-group-contributor"></a>管理群組參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 管理群組參與者角色 |
> | **Id** | 5d58bcaf-24a5-4b20-bdb6-eed9f69fbe4c |
> | **動作** |  |
> | Microsoft.Management/managementGroups/delete | 刪除管理群組。 |
> | Microsoft.Management/managementGroups/read | 列出已驗證之使用者的管理群組。 |
> | Microsoft.Management/managementGroups/subscriptions/delete | 從管理群組中取消訂用帳戶的關聯。 |
> | Microsoft.Management/managementGroups/subscriptions/write | 將現有的訂用帳戶關聯至管理群組。 |
> | Microsoft.Management/managementGroups/write | 建立或更新管理群組。 |

## <a name="management-group-reader"></a>管理群組讀者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 管理群組讀者角色 |
> | **Id** | ac63b705-f282-497d-ac71-919bf39d939d |
> | **動作** |  |
> | Microsoft.Management/managementGroups/read | 列出已驗證之使用者的管理群組。 |

## <a name="monitoring-contributor"></a>監視參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可以讀取所有監視資料並編輯監視設定。 請參閱[開始使用 Azure 監視器的角色、權限和安全性](../monitoring-and-diagnostics/monitoring-roles-permissions-security.md#built-in-monitoring-roles)。 |
> | **Id** | 749f88d5-cbae-40b8-bcfc-e573ddc772fa |
> | **動作** |  |
> | */read | 讀取密碼以外的所有類型的資源。 |
> | Microsoft.AlertsManagement/alerts/* |  |
> | Microsoft.AlertsManagement/alertsSummary/* |  |
> | Microsoft.Insights/actiongroups/* |  |
> | Microsoft.Insights/AlertRules/* | 讀取/寫入/刪除警示規則。 |
> | Microsoft.Insights/components/* | 讀取/寫入/刪除 Application Insights 元件。 |
> | Microsoft.Insights/DiagnosticSettings/* | 讀取/寫入/刪除診斷設定。 |
> | Microsoft.Insights/eventtypes/* | 列出訂用帳戶中的活動記錄檔事件 (管理事件)。 此權限適用於以程式設計方式存取和入口網站存取活動記錄檔。 |
> | Microsoft.Insights/LogDefinitions/* | 此為使用者需要透過入口網站存取活動記錄檔時所需的權限。 列出活動記錄檔中的記錄檔分類。 |
> | Microsoft.Insights/metricalerts/* |  |
> | Microsoft.Insights/MetricDefinitions/* | 讀取度量定義 (可用資源的度量類型清單)。 |
> | Microsoft.Insights/Metrics/* | 讀取資源的度量。 |
> | Microsoft.Insights/Register/Action | 註冊 Microsoft Insights 提供者 |
> | Microsoft.Insights/scheduledqueryrules/* |  |
> | Microsoft.Insights/webtests/* | 讀取/寫入/刪除 Application Insights Web 測試。 |
> | Microsoft.OperationalInsights/workspaces/intelligencepacks/* | 讀取/寫入/刪除 Log Analytics 解決方案套件。 |
> | Microsoft.OperationalInsights/workspaces/savedSearches/* | 讀取/寫入/刪除 Log Analytics 已儲存的搜尋。 |
> | Microsoft.OperationalInsights/workspaces/search/action | 執行搜尋查詢 |
> | Microsoft.OperationalInsights/workspaces/sharedKeys/action | 擷取工作區的共用金鑰。 這些金鑰可用來將 Microsoft Operational Insights 代理程式連線到工作區。 |
> | Microsoft.OperationalInsights/workspaces/storageinsightconfigs/* | 讀取/寫入/刪除 Log Analytics 儲存體深入解析設定。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | Microsoft.WorkloadMonitor/monitors/* |  |
> | Microsoft.WorkloadMonitor/notificationSettings/* |  |
> | Microsoft.WorkloadMonitor/workloadInsights/* |  |

## <a name="monitoring-metrics-publisher"></a>監視計量發行者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 針對 Azure 資源啟用發佈計量 |
> | **Id** | 3913510d-42f4-4e42-8a64-420c390055eb |
> | **動作** |  |
> | Microsoft.Insights/Register/Action | 註冊 Microsoft Insights 提供者 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | **DataActions** |  |
> | Microsoft.Insights/Metrics/Write | 寫入計量 |

## <a name="monitoring-reader"></a>監視讀取器
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可以讀取所有監視資料 (計量、記錄檔等等)。 請參閱[開始使用 Azure 監視器的角色、權限和安全性](../monitoring-and-diagnostics/monitoring-roles-permissions-security.md#built-in-monitoring-roles)。 |
> | **Id** | 43d0d8ad-25c7-4714-9337-8ba259a9fe05 |
> | **動作** |  |
> | */read | 讀取密碼以外的所有類型的資源。 |
> | Microsoft.OperationalInsights/workspaces/search/action | 執行搜尋查詢 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="network-contributor"></a>網路參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理網路，但無法存取它們。 |
> | **Id** | 4d97b98b-1d4f-4787-a291-c67834d212e7 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.Network/* | 建立和管理網路 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="new-relic-apm-account-contributor"></a>New Relic APM 帳戶參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 New Relic Application Performance Management 帳戶及應用程式，但無法存取它們。 |
> | **Id** | 5d28c62d-5b37-4476-8438-e587778df237 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | NewRelic.APM/accounts/* |  |

## <a name="powerapps-administrator-test"></a>PowerApps 系統管理員測試
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 Azure DNS 中的 DNS 區域與記錄集，但無法讓您控制誰可存取它們。 |
> | **Id** | befefa01-2a29-4897-83a8-272ff33ce314 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Network/dnsZones/* |  |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="reader-and-data-access"></a>讀取者及資料存取
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您檢視所有內容，但無法讓您刪除或建立儲存體帳戶或內含的資源。 也可透過存取儲存體帳戶金鑰，對儲存體帳戶中內含的所有資料進行讀取/寫入存取。 |
> | **Id** | c12c1c16-33a1-487b-954d-41c89c60f349 |
> | **動作** |  |
> | Microsoft.Storage/storageAccounts/listKeys/action | 傳回指定儲存體帳戶的存取金鑰。 |
> | Microsoft.Storage/storageAccounts/read | 傳回儲存體帳戶清單，或取得指定儲存體帳戶的屬性。 |

## <a name="redis-cache-contributor"></a>Redis 快取參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 Redis 快取，但無法存取它們。 |
> | **Id** | e0f68234-74aa-48ed-b826-c38b57376e17 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Cache/redis/* | 建立和管理 Redis 快取 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="resource-policy-contributor-preview"></a>資源原則參與者 (預覽)
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | (預覽) 從 EA 回填的使用者，有權建立/修改資源原則、建立支援票證及讀取資源/階層。 |
> | **Id** | 36243c78-bf99-498c-9df9-86d9f8d28608 |
> | **動作** |  |
> | */read | 讀取密碼以外的所有類型的資源。 |
> | Microsoft.Authorization/policyassignments/* | 建立及管理原則指派 |
> | Microsoft.Authorization/policydefinitions/* | 建立及管理原則定義 |
> | Microsoft.Authorization/policysetdefinitions/* | 建立及管理原則集合 |
> | Microsoft.PolicyInsights/* |  |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="scheduler-job-collections-contributor"></a>排程器工作集合參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理「排程器」工作集合，但無法存取它們。 |
> | **Id** | 188a0f2f-5c9e-469b-ae67-2aa5ce574b94 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Scheduler/jobcollections/* | 建立和管理工作集合 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="search-service-contributor"></a>搜尋服務參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理「搜尋」服務，但無法存取它們。 |
> | **Id** | 7ca78c08-252a-4471-8644-bb5ff32d4ba0 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Search/searchServices/* | 建立和管理搜尋服務 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="security-admin"></a>安全性系統管理員
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 僅限資訊安全中心：可檢視安全性原則、檢視安全性狀態、編輯安全性原則、檢視警示和建議、關閉警示和建議 |
> | **Id** | fb1c8493-542b-48eb-b624-b4c8fea62acd |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Authorization/policyAssignments/* | 建立及管理原則指派 |
> | Microsoft.Authorization/policyDefinitions/* | 建立及管理原則定義 |
> | Microsoft.Authorization/policySetDefinitions/* | 建立及管理原則集合 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.Management/managementGroups/read | 列出已驗證之使用者的管理群組。 |
> | Microsoft.operationalInsights/workspaces/*/read | 檢視 Log Analytics 資料 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Security/*/read | 讀取安全性元件和原則 |
> | Microsoft.Security/locations/alerts/activate/action | 啟動安全性警訊 |
> | Microsoft.Security/locations/alerts/dismiss/action | 關閉安全性警示 |
> | Microsoft.Security/locations/tasks/activate/action | 啟用安全性建議 |
> | Microsoft.Security/locations/tasks/dismiss/action | 關閉安全性建議 |
> | Microsoft.Security/policies/write | 更新安全性原則 |
> | Microsoft.Security/securityContacts/write | 更新安全性連絡人 |
> | Microsoft.Security/securityContacts/delete | 刪除安全性連絡人 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="security-manager"></a>安全性管理員
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理安全性元件、安全性原則及虛擬機器 |
> | **Id** | e3d13bf0-dd5a-482e-ba6b-9b8433878d10 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.ClassicCompute/*/read | 讀取傳統虛擬機器的設定資訊 |
> | Microsoft.ClassicCompute/virtualMachines/*/write | 撰寫傳統虛擬機器的設定 |
> | Microsoft.ClassicNetwork/*/read | 讀取傳統網路的組態資訊 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Security/* | 建立和管理安全性元件和原則 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="security-reader"></a>安全性讀取者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 僅限資訊安全中心：可檢視建議和警示、檢視安全性原則、檢視安全性狀態，但無法進行變更 |
> | **Id** | 39bc4728-0917-49c7-9d2c-d95423bc2eb4 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.operationalInsights/workspaces/*/read | 檢視 Log Analytics 資料 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Security/*/read | 讀取安全性元件和原則 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | Microsoft.Management/managementGroups/read | 列出已驗證之使用者的管理群組。 |

## <a name="site-recovery-contributor"></a>Site Recovery 參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 Site Recovery 服務，但無法建立保存庫和指派角色 |
> | **Id** | 6670b86e-a3f7-4917-ac9b-5d6ab1be4567 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.Network/virtualNetworks/read | 取得虛擬網路定義 |
> | Microsoft.RecoveryServices/locations/allocatedStamp/read | GetAllocatedStamp 是服務所使用的內部作業 |
> | Microsoft.RecoveryServices/locations/allocateStamp/action | AllocateStamp 是服務所使用的內部作業 |
> | Microsoft.RecoveryServices/Vaults/certificates/write | 「更新資源憑證」作業會更新資源/保存庫的認證憑證。 |
> | Microsoft.RecoveryServices/Vaults/extendedInformation/* | 建立和管理與保存庫相關的擴充資訊 |
> | Microsoft.RecoveryServices/Vaults/read | 「取得保存庫」作業會取得物件，此物件代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/refreshContainers/read |  |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/* | 建立和管理註冊的身分識別 |
> | Microsoft.RecoveryServices/vaults/replicationAlertSettings/* | 建立或更新複寫警示設定 |
> | Microsoft.RecoveryServices/vaults/replicationEvents/read | 讀取任何事件 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/* | 建立和管理複寫網狀架構 |
> | Microsoft.RecoveryServices/vaults/replicationJobs/* | 建立和管理複寫作業 |
> | Microsoft.RecoveryServices/vaults/replicationPolicies/* | 建立和管理複寫原則 |
> | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/* | 建立和管理復原計劃 |
> | Microsoft.RecoveryServices/Vaults/storageConfig/* | 建立和管理復原服務保存庫的儲存體設定 |
> | Microsoft.RecoveryServices/Vaults/tokenInfo/read | 傳回復原服務保存庫的權杖資訊。 |
> | Microsoft.RecoveryServices/Vaults/usages/read | 傳回復原服務保存庫的使用量詳細資料。 |
> | Microsoft.RecoveryServices/Vaults/vaultTokens/read | 「保存庫權杖」作業可用來取得保存庫層級後端作業的保存庫權杖。 |
> | Microsoft.RecoveryServices/Vaults/monitoringAlerts/* | 讀取復原服務保存庫的警示 |
> | Microsoft.RecoveryServices/Vaults/monitoringConfigurations/notificationConfiguration/read |  |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Storage/storageAccounts/read | 傳回儲存體帳戶清單，或取得指定儲存體帳戶的屬性。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="site-recovery-operator"></a>Site Recovery 操作員
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您容錯移轉及容錯回復，但無法執行其他 Site Recovery 管理作業 |
> | **Id** | 494ae006-db33-4328-bf46-533a6560a3ca |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.Network/virtualNetworks/read | 取得虛擬網路定義 |
> | Microsoft.RecoveryServices/locations/allocatedStamp/read | GetAllocatedStamp 是服務所使用的內部作業 |
> | Microsoft.RecoveryServices/locations/allocateStamp/action | AllocateStamp 是服務所使用的內部作業 |
> | Microsoft.RecoveryServices/Vaults/extendedInformation/read | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/read | 「取得保存庫」作業會取得物件，此物件代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/refreshContainers/read |  |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/operationResults/read | 「取得作業結果」作業可用來取得以非同步方式提交之作業的作業狀態和結果 |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/read | 「取得容器」作業可用來取得為資源註冊的容器。 |
> | Microsoft.RecoveryServices/vaults/replicationAlertSettings/read | 讀取任何警示設定 |
> | Microsoft.RecoveryServices/vaults/replicationEvents/read | 讀取任何事件 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/checkConsistency/action | 檢查網狀架構的一致性 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/read | 讀取任何網狀架構 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/reassociateGateway/action | 重新關聯閘道 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/renewcertificate/action | 更新網狀架構的憑證 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationNetworks/read | 讀取任何網路 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationNetworks/replicationNetworkMappings/read | 讀取任何網路對應 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/read | 讀取任何保護容器 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectableItems/read | 讀取任何可保護的項目 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/applyRecoveryPoint/action | 套用復原點 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/failoverCommit/action | 容錯移轉認可 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/plannedFailover/action | 計劃性容錯移轉 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/read | 讀取任何受保護的項目 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/recoveryPoints/read | 讀取任何複寫復原點 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/repairReplication/action | 修復複寫 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/reProtect/action | 重新保護受保護的項目 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/testFailover/action | Test Failover |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/testFailoverCleanup/action | 測試容錯移轉清理 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/unplannedFailover/action | 容錯移轉 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/updateMobilityService/action | 更新行動服務 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectionContainerMappings/read | 讀取任何保護容器對應 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationRecoveryServicesProviders/read | 讀取任何復原服務提供者 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationRecoveryServicesProviders/refreshProvider/action | 重新整理提供者 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationStorageClassifications/read | 讀取任何存放裝置分類 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationStorageClassifications/replicationStorageClassificationMappings/read | 讀取任何存放裝置分類對應 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationvCenters/read | 讀取任何 vCenter |
> | Microsoft.RecoveryServices/vaults/replicationJobs/* | 建立和管理複寫作業 |
> | Microsoft.RecoveryServices/vaults/replicationPolicies/read | 讀取任何原則 |
> | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/failoverCommit/action | 容錯移轉認可復原方案 |
> | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/plannedFailover/action | 計劃性容錯移轉復原方案 |
> | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/read | 讀取任何復原方案 |
> | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/reProtect/action | 重新保護復原方案 |
> | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/testFailover/action | 測試容錯移轉復原方案 |
> | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/testFailoverCleanup/action | 測試容錯移轉清理復原方案 |
> | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/unplannedFailover/action | 容錯移轉復原方案 |
> | Microsoft.RecoveryServices/Vaults/monitoringAlerts/* | 讀取復原服務保存庫的警示 |
> | Microsoft.RecoveryServices/Vaults/monitoringConfigurations/notificationConfiguration/read |  |
> | Microsoft.RecoveryServices/Vaults/storageConfig/read |  |
> | Microsoft.RecoveryServices/Vaults/tokenInfo/read | 傳回復原服務保存庫的權杖資訊。 |
> | Microsoft.RecoveryServices/Vaults/usages/read | 傳回復原服務保存庫的使用量詳細資料。 |
> | Microsoft.RecoveryServices/Vaults/vaultTokens/read | 「保存庫權杖」作業可用來取得保存庫層級後端作業的保存庫權杖。 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Storage/storageAccounts/read | 傳回儲存體帳戶清單，或取得指定儲存體帳戶的屬性。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="site-recovery-reader"></a>Site Recovery 讀取者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您檢視 Site Recovery 狀態，但無法執行其他管理作業 |
> | **Id** | dbaa88c4-0c30-4179-9fb3-46319faa6149 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.RecoveryServices/locations/allocatedStamp/read | GetAllocatedStamp 是服務所使用的內部作業 |
> | Microsoft.RecoveryServices/Vaults/extendedInformation/read | 「取得延伸資訊」作業會取得物件的延伸資訊，此延伸資訊代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/monitoringAlerts/read | 取得復原服務保存庫的警示。 |
> | Microsoft.RecoveryServices/Vaults/monitoringConfigurations/notificationConfiguration/read |  |
> | Microsoft.RecoveryServices/Vaults/read | 「取得保存庫」作業會取得物件，此物件代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/refreshContainers/read |  |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/operationResults/read | 「取得作業結果」作業可用來取得以非同步方式提交之作業的作業狀態和結果 |
> | Microsoft.RecoveryServices/Vaults/registeredIdentities/read | 「取得容器」作業可用來取得為資源註冊的容器。 |
> | Microsoft.RecoveryServices/vaults/replicationAlertSettings/read | 讀取任何警示設定 |
> | Microsoft.RecoveryServices/vaults/replicationEvents/read | 讀取任何事件 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/read | 讀取任何網狀架構 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationNetworks/read | 讀取任何網路 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationNetworks/replicationNetworkMappings/read | 讀取任何網路對應 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/read | 讀取任何保護容器 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectableItems/read | 讀取任何可保護的項目 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/read | 讀取任何受保護的項目 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems/recoveryPoints/read | 讀取任何複寫復原點 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectionContainerMappings/read | 讀取任何保護容器對應 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationRecoveryServicesProviders/read | 讀取任何復原服務提供者 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationStorageClassifications/read | 讀取任何存放裝置分類 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationStorageClassifications/replicationStorageClassificationMappings/read | 讀取任何存放裝置分類對應 |
> | Microsoft.RecoveryServices/vaults/replicationFabrics/replicationvCenters/read | 讀取任何 vCenter |
> | Microsoft.RecoveryServices/vaults/replicationJobs/read | 讀取任何作業 |
> | Microsoft.RecoveryServices/vaults/replicationPolicies/read | 讀取任何原則 |
> | Microsoft.RecoveryServices/vaults/replicationRecoveryPlans/read | 讀取任何復原方案 |
> | Microsoft.RecoveryServices/Vaults/storageConfig/read |  |
> | Microsoft.RecoveryServices/Vaults/tokenInfo/read | 傳回復原服務保存庫的權杖資訊。 |
> | Microsoft.RecoveryServices/Vaults/usages/read | 傳回復原服務保存庫的使用量詳細資料。 |
> | Microsoft.RecoveryServices/Vaults/vaultTokens/read | 「保存庫權杖」作業可用來取得保存庫層級後端作業的保存庫權杖。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="sql-db-contributor"></a>SQL DB 參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 SQL 資料庫，但無法存取它們。 此外，您也無法管理其安全性相關原則或其父 SQL 伺服器。 |
> | **Id** | 9b7fa17d-e63e-47b0-bb0a-15c516ac86ec |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立及管理警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Sql/locations/*/read |  |
> | Microsoft.Sql/servers/databases/* | 建立和管理 SQL 資料庫 |
> | Microsoft.Sql/servers/read | 傳回伺服器清單，或取得指定伺服器的屬性。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | **NotActions** |  |
> | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/* |  |
> | Microsoft.Sql/servers/databases/auditingPolicies/* | 無法編輯稽核原則 |
> | Microsoft.Sql/servers/databases/auditingSettings/* | 無法編輯稽核設定 |
> | Microsoft.Sql/servers/databases/auditRecords/read | 擷取資料庫 Blob 稽核記錄 |
> | Microsoft.Sql/servers/databases/connectionPolicies/* | 無法編輯連接原則 |
> | Microsoft.Sql/servers/databases/dataMaskingPolicies/* | 無法編輯資料遮罩原則 |
> | Microsoft.Sql/servers/databases/extendedAuditingSettings/* |  |
> | Microsoft.Sql/servers/databases/schemas/tables/columns/sensitivityLabels/* |  |
> | Microsoft.Sql/servers/databases/securityAlertPolicies/* | 無法編輯安全性警示原則 |
> | Microsoft.Sql/servers/databases/securityMetrics/* | 無法編輯安全性度量 |
> | Microsoft.Sql/servers/databases/sensitivityLabels/* |  |
> | Microsoft.Sql/servers/databases/vulnerabilityAssessments/* |  |
> | Microsoft.Sql/servers/databases/vulnerabilityAssessmentScans/* |  |
> | Microsoft.Sql/servers/databases/vulnerabilityAssessmentSettings/* |  |

## <a name="sql-security-manager"></a>SQL 安全性管理員
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 SQL 伺服器及資料庫的安全性相關原則，但無法存取它們。 |
> | **Id** | 056cd41c-7e88-42e1-933e-88ba6a50c9c3 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取 Microsoft 授權 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action | 將資源 (例如，儲存體帳戶或 SQL Database) 加入至子網路。 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/* |  |
> | Microsoft.Sql/servers/auditingPolicies/* | 建立和管理 SQL Server 稽核原則 |
> | Microsoft.Sql/servers/auditingSettings/* | 建立和管理 SQL Server 稽核設定 |
> | Microsoft.Sql/servers/databases/auditingPolicies/* | 建立和管理 SQL Server 資料庫稽核原則 |
> | Microsoft.Sql/servers/databases/auditingSettings/* | 建立和管理 SQL Server 資料庫稽核設定 |
> | Microsoft.Sql/servers/databases/auditRecords/read | 讀取稽核記錄 |
> | Microsoft.Sql/servers/databases/connectionPolicies/* | 建立和管理 SQL Server 資料庫連接原則 |
> | Microsoft.Sql/servers/databases/dataMaskingPolicies/* | 建立和管理 SQL Server 資料庫資料遮罩原則 |
> | Microsoft.Sql/servers/databases/read | 傳回資料庫清單，或取得指定資料庫的屬性。 |
> | Microsoft.Sql/servers/databases/schemas/read | 擷取資料庫的結構描述清單 |
> | Microsoft.Sql/servers/databases/schemas/tables/columns/read | 擷取資料表的資料行清單 |
> | Microsoft.Sql/servers/databases/schemas/tables/columns/sensitivityLabels/* |  |
> | Microsoft.Sql/servers/databases/schemas/tables/read | 擷取資料庫的資料表清單 |
> | Microsoft.Sql/servers/databases/securityAlertPolicies/* | 建立和管理 SQL Server 資料庫安全性警示原則 |
> | Microsoft.Sql/servers/databases/securityMetrics/* | 建立和管理 SQL Server 資料庫安全性度量 |
> | Microsoft.Sql/servers/databases/sensitivityLabels/* |  |
> | Microsoft.Sql/servers/databases/vulnerabilityAssessments/* |  |
> | Microsoft.Sql/servers/databases/vulnerabilityAssessmentScans/* |  |
> | Microsoft.Sql/servers/databases/vulnerabilityAssessmentSettings/* |  |
> | Microsoft.Sql/servers/firewallRules/* |  |
> | Microsoft.Sql/servers/read | 傳回伺服器清單，或取得指定伺服器的屬性。 |
> | Microsoft.Sql/servers/securityAlertPolicies/* | 建立和管理 SQL Server 安全性警示原則 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="sql-server-contributor"></a>SQL Server 參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 SQL 伺服器及資料庫，但無法存取它們，也無法存取其安全性相關原則。 |
> | **Id** | 6d8ee4ec-f05a-4a1d-8b00-a9b17e38b437 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Sql/locations/*/read |  |
> | Microsoft.Sql/servers/* | 建立和管理 SQL Server |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | **NotActions** |  |
> | Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/* |  |
> | Microsoft.Sql/servers/auditingPolicies/* | 無法編輯 SQL Server 稽核原則 |
> | Microsoft.Sql/servers/auditingSettings/* | 無法編輯 SQL Server 稽核設定 |
> | Microsoft.Sql/servers/databases/auditingPolicies/* | 無法編輯 SQL Server 資料庫稽核原則 |
> | Microsoft.Sql/servers/databases/auditingSettings/* | 無法編輯 SQL Server 資料庫稽核設定 |
> | Microsoft.Sql/servers/databases/auditRecords/read | 無法讀取稽核記錄 |
> | Microsoft.Sql/servers/databases/connectionPolicies/* | 無法編輯 SQL Server 資料庫連接原則 |
> | Microsoft.Sql/servers/databases/dataMaskingPolicies/* | 無法編輯 SQL Server 資料庫資料遮罩原則 |
> | Microsoft.Sql/servers/databases/extendedAuditingSettings/* |  |
> | Microsoft.Sql/servers/databases/schemas/tables/columns/sensitivityLabels/* |  |
> | Microsoft.Sql/servers/databases/securityAlertPolicies/* | 無法編輯 SQL Server 資料庫安全性警示原則 |
> | Microsoft.Sql/servers/databases/securityMetrics/* | 無法編輯 SQL Server 資料庫安全性度量 |
> | Microsoft.Sql/servers/databases/sensitivityLabels/* |  |
> | Microsoft.Sql/servers/databases/vulnerabilityAssessments/* |  |
> | Microsoft.Sql/servers/databases/vulnerabilityAssessmentScans/* |  |
> | Microsoft.Sql/servers/databases/vulnerabilityAssessmentSettings/* |  |
> | Microsoft.Sql/servers/extendedAuditingSettings/* |  |
> | Microsoft.Sql/servers/securityAlertPolicies/* | 無法編輯 SQL Server 安全性警示原則 |

## <a name="storage-account-contributor"></a>儲存體帳戶參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理儲存體帳戶，但無法存取它們。 |
> | **Id** | 17d1049b-9a84-46fb-8f53-869881c3d3ab |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取所有授權 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Insights/diagnosticSettings/* | 管理診斷設定 |
> | Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action | 將資源 (例如，儲存體帳戶或 SQL Database) 加入至子網路。 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Storage/storageAccounts/* | 建立及管理儲存體帳戶 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="storage-account-key-operator-service-role"></a>儲存體帳戶金鑰操作員服務角色
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 「儲存體帳戶金鑰操作員」可以列出及重新產生「儲存體帳戶」的金鑰 |
> | **Id** | 81a9662b-bebf-436f-a333-f67b29880f12 |
> | **動作** |  |
> | Microsoft.Storage/storageAccounts/listkeys/action | 傳回指定儲存體帳戶的存取金鑰。 |
> | Microsoft.Storage/storageAccounts/regeneratekey/action | 重新產生指定儲存體帳戶的存取金鑰。 |

## <a name="storage-blob-data-contributor-preview"></a>儲存體 Blob 資料參與者 (預覽)
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讀取、寫入及刪除 Azure 儲存體 Blob 容器與資料 |
> | **Id** | ba92f5b4-2d11-453d-a403-e96b0029c9fe |
> | **動作** |  |
> | Microsoft.Storage/storageAccounts/blobServices/containers/delete | 傳回刪除容器的結果 |
> | Microsoft.Storage/storageAccounts/blobServices/containers/read | 傳回容器或容器清單 |
> | Microsoft.Storage/storageAccounts/blobServices/containers/write | 傳回放置或租用 Blob 容器的結果 |
> | **DataActions** |  |
> | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete | 傳回刪除 Blob 的結果 |
> | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read | 傳回 Blob 或 Blob 清單 |
> | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write | 傳回寫入 Blob 的結果 |

## <a name="storage-blob-data-reader-preview"></a>儲存體 Blob 資料讀者 (預覽)
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讀取 Azure 儲存體 Blob 容器與資料 |
> | **Id** | 2a2b9908-6ea1-4ae2-8e65-a410df84e7d1 |
> | **動作** |  |
> | Microsoft.Storage/storageAccounts/blobServices/containers/read | 傳回容器或容器清單 |
> | **DataActions** |  |
> | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read | 傳回 Blob 或 Blob 清單 |

## <a name="storage-queue-data-contributor-preview"></a>儲存體佇列資料參與者 (預覽)
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讀取、寫入及刪除 Azure 儲存體佇列與佇列訊息 |
> | **Id** | 974c5e8b-45b9-4653-ba55-5f855dd0fb88 |
> | **動作** |  |
> | Microsoft.Storage/storageAccounts/queueServices/queues/delete | 傳回刪除佇列的結果 |
> | Microsoft.Storage/storageAccounts/queueServices/queues/read | 傳回佇列或佇列清單。 |
> | Microsoft.Storage/storageAccounts/queueServices/queues/write | 傳回寫入佇列的結果 |
> | **DataActions** |  |
> | Microsoft.Storage/storageAccounts/queueServices/queues/messages/delete | 傳回刪除訊息的結果 |
> | Microsoft.Storage/storageAccounts/queueServices/queues/messages/read | 傳回訊息 |
> | Microsoft.Storage/storageAccounts/queueServices/queues/messages/write | 傳回寫入訊息的結果 |

## <a name="storage-queue-data-reader-preview"></a>儲存體佇列資料讀者 (預覽)
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讀取 Azure 儲存體佇列與佇列訊息 |
> | **Id** | 19e7f393-937e-4f77-808e-94535e297925 |
> | **動作** |  |
> | Microsoft.Storage/storageAccounts/queueServices/queues/read | 傳回佇列或佇列清單。 |
> | **DataActions** |  |
> | Microsoft.Storage/storageAccounts/queueServices/queues/messages/read | 傳回訊息 |

## <a name="support-request-contributor"></a>支援要求參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您建立及管理支援要求 |
> | **Id** | cfd33db0-3dd1-45e3-aa9d-cdbdf3b6f24e |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取授權 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="traffic-manager-contributor"></a>流量管理員參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理「流量管理員」設定檔，但無法控制誰可以存取它們。 |
> | **Id** | a4b10055-b0c7-44c2-b00f-c7b5b3550cf7 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取角色和角色指派 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Network/trafficManagerProfiles/* |  |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="user-access-administrator"></a>使用者存取系統管理員
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理 Azure 資源的使用者存取。 |
> | **Id** | 18d7d88d-d35e-4fb5-a5c3-7773c20a72d9 |
> | **動作** |  |
> | */read | 讀取密碼以外的所有類型的資源。 |
> | Microsoft.Authorization/* | 管理授權 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="virtual-machine-administrator-login"></a>虛擬機器系統管理員登入
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 在入口網站中檢視虛擬機器並以系統管理員身分登入 |
> | **Id** | 1c0163c0-47e6-4577-8991-ea5c82e286e4 |
> | **動作** |  |
> | Microsoft.Network/publicIPAddresses/read | 取得公用 IP 位址定義。 |
> | Microsoft.Network/virtualNetworks/read | 取得虛擬網路定義 |
> | Microsoft.Network/loadBalancers/read | 取得負載平衡器定義 |
> | Microsoft.Network/networkInterfaces/read | 取得網路介面定義。  |
> | Microsoft.Compute/virtualMachines/*/read |  |
> | **DataActions** |  |
> | Microsoft.Compute/virtualMachines/login/action | 以一般使用者身分登入虛擬機器 |
> | Microsoft.Compute/virtualMachines/loginAsAdmin/action | 以 Windows 系統管理員或 Linux 根使用者權限登入虛擬機器 |

## <a name="virtual-machine-contributor"></a>虛擬機器參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理虛擬機器 (不含虛擬機器所連接的虛擬網路或儲存體帳戶)，但無法存取它們。 |
> | **Id** | 9980e02c-c2be-4d73-94e8-173b1dc7cf3c |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取授權 |
> | Microsoft.Compute/availabilitySets/* | 建立和管理運算可用性集合 |
> | Microsoft.Compute/locations/* | 建立和管理運算位置 |
> | Microsoft.Compute/virtualMachines/* | 建立和管理虛擬機器 |
> | Microsoft.Compute/virtualMachineScaleSets/* | 建立和管理虛擬機器擴展集 |
> | Microsoft.DevTestLab/schedules/* |  |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Network/applicationGateways/backendAddressPools/join/action | 加入應用程式閘道後端位址集區 |
> | Microsoft.Network/loadBalancers/backendAddressPools/join/action | 加入負載平衡器的後端位址集區 |
> | Microsoft.Network/loadBalancers/inboundNatPools/join/action | 加入負載平衡器的輸入 NAT 集區 |
> | Microsoft.Network/loadBalancers/inboundNatRules/join/action | 加入負載平衡器的輸入 NAT 規則 |
> | Microsoft.Network/loadBalancers/probes/join/action | 允許使用負載平衡器的探查。 例如，使用此權限，VM 擴展集的 healthProbe 屬性就可以參考探查。 |
> | Microsoft.Network/loadBalancers/read | 取得負載平衡器定義 |
> | Microsoft.Network/locations/* | 建立和管理網路位置 |
> | Microsoft.Network/networkInterfaces/* | 建立和管理網路介面 |
> | Microsoft.Network/networkSecurityGroups/join/action | 加入網路安全性群組 |
> | Microsoft.Network/networkSecurityGroups/read | 取得網路安全性群組定義 |
> | Microsoft.Network/publicIPAddresses/join/action | 加入公用 IP 位址 |
> | Microsoft.Network/publicIPAddresses/read | 取得公用 IP 位址定義。 |
> | Microsoft.Network/virtualNetworks/read | 取得虛擬網路定義 |
> | Microsoft.Network/virtualNetworks/subnets/join/action | 加入虛擬網路 |
> | Microsoft.RecoveryServices/locations/* |  |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/backupProtectionIntent/write | 建立備份保護用途 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/*/read |  |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/read | 傳回受保護項目的物件詳細資料 |
> | Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/write | 建立備用的受保護項目 |
> | Microsoft.RecoveryServices/Vaults/backupPolicies/read | 傳回所有保護原則 |
> | Microsoft.RecoveryServices/Vaults/backupPolicies/write | 建立保護原則 |
> | Microsoft.RecoveryServices/Vaults/read | 「取得保存庫」作業會取得物件，此物件代表 'vault' 類型的 Azure 資源 |
> | Microsoft.RecoveryServices/Vaults/usages/read | 傳回復原服務保存庫的使用量詳細資料。 |
> | Microsoft.RecoveryServices/Vaults/write | 「建立保存庫」作業會建立 'vault' 類型的 Azure 資源 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.SqlVirtualMachine/* |  |
> | Microsoft.Storage/storageAccounts/listKeys/action | 傳回指定儲存體帳戶的存取金鑰。 |
> | Microsoft.Storage/storageAccounts/read | 傳回儲存體帳戶清單，或取得指定儲存體帳戶的屬性。 |
> | Microsoft.Support/* | 建立和管理支援票證 |

## <a name="virtual-machine-user-login"></a>虛擬機器使用者登入
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 在入口網站中檢視虛擬機器並以一般使用者身分登入。 |
> | **Id** | fb879df8-f326-4884-b1cf-06f3ad86be52 |
> | **動作** |  |
> | Microsoft.Network/publicIPAddresses/read | 取得公用 IP 位址定義。 |
> | Microsoft.Network/virtualNetworks/read | 取得虛擬網路定義 |
> | Microsoft.Network/loadBalancers/read | 取得負載平衡器定義 |
> | Microsoft.Network/networkInterfaces/read | 取得網路介面定義。  |
> | Microsoft.Compute/virtualMachines/*/read |  |
> | **DataActions** |  |
> | Microsoft.Compute/virtualMachines/login/action | 以一般使用者身分登入虛擬機器 |

## <a name="web-plan-contributor"></a>Web 方案參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理網站的 Web 方案，但無法存取它們。 |
> | **Id** | 2cc479cb-7b4d-49a8-b449-8c00fd0f0a4b |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取授權 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | Microsoft.Web/serverFarms/* | 建立和管理伺服器陣列 |

## <a name="website-contributor"></a>網站參與者
> [!div class="mx-tableFixed"]
> | | |
> | --- | --- |
> | **說明** | 可讓您管理網站 (非 Web 方案)，但無法存取它們。 |
> | **Id** | de139f84-1756-47ae-9be6-808fbbe84772 |
> | **動作** |  |
> | Microsoft.Authorization/*/read | 讀取授權 |
> | Microsoft.Insights/alertRules/* | 建立和管理 Insights 警示規則 |
> | Microsoft.Insights/components/* | 建立和管理 Insights 元件 |
> | Microsoft.ResourceHealth/availabilityStatuses/read | 取得指定範圍中所有資源的可用性狀態 |
> | Microsoft.Resources/deployments/* | 建立和管理資源群組部署 |
> | Microsoft.Resources/subscriptions/resourceGroups/read | 取得或列出資源群組。 |
> | Microsoft.Support/* | 建立和管理支援票證 |
> | Microsoft.Web/certificates/* | 建立和管理網站憑證 |
> | Microsoft.Web/listSitesAssignedToHostName/read | 取得指派給主機名稱之網站的名稱。 |
> | Microsoft.Web/serverFarms/join/action |  |
> | Microsoft.Web/serverFarms/read | 取得 App Service 方案的屬性 |
> | Microsoft.Web/sites/* | 建立和管理網站 (建立網站也需要相關聯應用程式服務方案的寫入權限) |

## <a name="next-steps"></a>後續步驟

- [自訂角色](custom-roles.md)
- [使用 Azure 入口網站管理角色指派](role-assignments-portal.md)
- [Azure 資訊安全中心的權限](../security-center/security-center-permissions.md)
