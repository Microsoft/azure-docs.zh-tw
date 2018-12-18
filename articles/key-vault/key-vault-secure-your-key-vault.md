---
title: 保護您的金鑰保存庫 | Microsoft Docs
description: 管理金鑰保存庫的存取權限，來管理保存庫以及金鑰和密碼。 金鑰保存庫的驗證和授權模型以及如何保護金鑰保存庫
services: key-vault
documentationcenter: ''
author: amitbapat
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: e5b4e083-4a39-4410-8e3a-2832ad6db405
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 08/31/2018
ms.author: ambapat
ms.openlocfilehash: 9b8b0da6e1572ab79ffb369497f64aad2cd249b9
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43343457"
---
# <a name="secure-your-key-vault"></a>保護您的金鑰保存庫
Azure 金鑰保存庫是用來保護雲端應用程式加密金鑰和密碼 (例如憑證、連接字串、密碼) 的雲端服務。 由於這項資料相當敏感且攸關業務，您會想要保護金鑰保存庫的存取權，讓只有獲得授權的應用程式和使用者可以存取金鑰保存庫。 本文提供金鑰保存庫存取模型概觀、說明驗證和授權，並透過範例描述如何保護雲端應用程式金鑰保存庫的存取權。

## <a name="overview"></a>概觀
金鑰保存庫的存取權可透過兩個不同介面來控制︰管理平面和資料平面。 這兩個平面都需要先經過適當的驗證和授權，呼叫者 (使用者或應用程式) 才能取得金鑰保存庫的存取權。 驗證會建立呼叫者的身分識別，授權則會判斷呼叫者可以執行的作業。

管理平面和資料平面都會使用 Azure Active Directory 來進行驗證。 至於授權，管理平面使用角色型存取控制 (RBAC)，資料平面則使用金鑰保存庫存取原則。

以下是所涵蓋主題的簡短概觀︰

[使用 Azure Active Directory 進行驗證](#authentication-using-azure-active-directory) - 本節說明呼叫者如何使用 Azure Active Directory 進行驗證，以透過管理平面和資料平面存取金鑰保存庫。 

[管理平面和資料平面](#management-plane-and-data-plane) - 管理平面和資料平面是用來存取金鑰保存庫的兩個存取平面。 這兩個存取平面各支援特定作業。 本節描述各個平面所使用的存取端點、所支援作業和存取控制方法。 

[管理平面存取控制](#management-plane-access-control) - 在這一節，我們將探討如何使用角色型存取控制來允許管理平面作業的存取權。

[資料平面存取控制](#data-plane-access-control) - 本節描述如何使用金鑰保存庫存取原則來控制資料平面存取。

[範例](#example) - 此範例描述如何設定金鑰保存庫的存取控制，允許三個不同小組 (安全性小組、開發人員/操作員和稽核員) 執行特定工作，以在 Azure 中開發、管理和監視應用程式。

## <a name="authentication-using-azure-active-directory"></a>使用 Azure Active Directory 進行驗證
當您在 Azure 訂用帳戶中建立金鑰保存庫時，它會自動與該訂用帳戶的 Azure Active Directory 租用戶建立關聯。 所有呼叫者 (使用者和應用程式) 都必須在此租用戶中註冊，才能存取此金鑰保存庫。 應用程式或使用者必須使用 Azure Active Directory 進行驗證，才能存取金鑰保存庫。 管理平面和資料平面的存取皆適用此原則。 在這兩種情況中，應用程式可透過兩種方式存取金鑰保存庫︰

* **使用者 + 應用程式的存取** - 這通常適用於代表登入之使用者存取金鑰保存庫的應用程式。 舉例來說，Azure PowerShell 和 Azure 入口網站便是這類存取。 有兩種方式可對使用者授與存取權︰一種方式是對使用者授與存取權，讓他們可以從任何應用程式存取金鑰保存庫，另一種方式則是，只在使用者使用特定應用程式 (稱為複合身分識別) 時，才對其授與金鑰保存庫的存取權。 
* **僅應用程式的存取** - 適用於執行精靈服務、背景作業等項目的應用程式。該應用程式的身分識別會被授與金鑰保存庫的存取權。

在這兩種應用程式中，應用程式會使用任何[支援的驗證方法](../active-directory/develop/authentication-scenarios.md)向 Azure Active Directory 進行驗證，並取得權杖。 使用何種驗證方法取決於應用程式類型。 然後，應用程式會使用此權杖，對金鑰保存庫傳送 REST API 要求。 如果是管理平面存取，要求會透過 Azure Resource Manager 端點路由傳送。 在存取資料平面時，應用程式則會直接與金鑰保存庫端點通訊。 如需詳細資訊，請參閱[整個驗證流程](../active-directory/develop/v1-protocols-oauth-code.md)。 

根據應用程式是存取管理平面還是資料平面，受到應用程式要求權杖的資源名稱會有所不同。 因此，視 Azure 環境而定，資源名稱將是稍後章節所含資料表中所述的管理平面或資料平面端點。

管理平面和資料平面各有一個驗證機制會有它自己的優點︰

* 組織可以集中控制其組織內所有金鑰保存庫的存取權
* 使用者如果離職，就會立即失去組織中所有金鑰保存庫的存取權
* 組織可以透過 Azure Active Directory 中的選項 (例如，啟用 Multi-Factor Authentication 以提升安全性) 自訂驗證

## <a name="management-plane-and-data-plane"></a>管理平面和資料平面
Azure 金鑰保存庫是一項可透過 Azure Resource Manager 部署模型取得的 Azure 服務。 當您建立金鑰保存庫時，就會獲得虛擬容器以供您在其中建立其他物件，像是金鑰、密碼和憑證。 然後，您可以使用管理平面和資料平面存取金鑰保存庫，以執行特定作業。 管理平面介面可用來管理金鑰保存庫本身，例如建立、刪除、更新金鑰保存庫屬性和設定資料平面的存取原則。 資料平面介面可用來新增、刪除、修改和使用金鑰保存庫中儲存的金鑰、密碼和憑證。

管理平面和資料平面介面需透過不同端點存取 (請參閱資料表)。 資料表中的第二個資料行描述不同 Azure 環境中這些端點的 DNS 名稱。 第三個資料行描述可從每個存取平面執行的作業。 每個存取平面也有自己的存取控制機制︰管理平面使用 Azure Resource Manager 角色型存取控制 (RBAC) 來設定存取控制，資料平面則使用金鑰保存庫存取原則來設定存取控制。

| 存取平面 | 存取端點 | 作業 | 存取控制機制 |
| --- | --- | --- | --- |
| 管理平面 |**全域：**<br> management.azure.com:443<br><br> **Azure 中國︰**<br> management.chinacloudapi.cn:443<br><br> **Azure 美國政府︰**<br> management.usgovcloudapi.net:443<br><br> **Azure 德國︰**<br> management.microsoftazure.de:443 |建立/讀取/更新/刪除金鑰保存庫 <br> 設定金鑰保存庫的存取原則<br>設定金鑰保存庫的標籤 |Azure Resource Manager 角色型存取控制 (RBAC) |
| 資料平面 |**全域：**<br> &lt;vault-name&gt;.vault.azure.net:443<br><br> **Azure 中國︰**<br> &lt;vault-name&gt;.vault.azure.cn:443<br><br> **Azure 美國政府︰**<br> &lt;vault-name&gt;.vault.usgovcloudapi.net:443<br><br> **Azure 德國︰**<br> &lt;vault-name&gt;.vault.microsoftazure.de:443 |針對金鑰︰解密、加密、解除包裝金鑰、包裝金鑰、驗證、簽署、取得、列出、更新、建立、匯入、刪除、備份、還原<br><br> 針對密碼︰取得、列出、設定、刪除 |金鑰保存庫存取原則 |

管理平面和資料平面的存取控制在運作上互不相關。 比方說，如果您想要對應用程式授與金鑰保存庫中金鑰的使用權限，您只需要使用金鑰保存庫存取原則對資料平面授與存取權限，此應用程式完全不需要管理平面的存取權。 相反地，如果您想要讓使用者能夠讀取保存庫屬性和標籤，但不讓他擁有金鑰、密碼或憑證的任何存取權，您可以使用 RBAC 對這位使用者授與「讀取」權限，但不需要授與資料平面的存取權。

## <a name="management-plane-access-control"></a>管理平面存取控制
管理平面是由會影響金鑰保存庫本身的作業所組成。 例如，您可以建立或刪除金鑰保存庫。 您可以取得訂用帳戶中的保存庫清單。 您可以擷取金鑰保存庫屬性 (例如 SKU、標籤)，並設定金鑰保存庫存取原則來控制可存取金鑰保存庫中金鑰和密碼的使用者和應用程式。 管理平面存取控制會使用 RBAC。 請參閱上一節的資料表中，可透過管理平面執行的金鑰保存庫作業完整清單。 

### <a name="role-based-access-control-rbac"></a>角色型存取控制 (RBAC)
每一個 Azure 訂用帳戶都具有 Azure Active Directory。 您可以為來自該目錄的使用者、群組和應用程式授與存取權，以便在使用 Azure Resource Manager 部署模型的 Azure 訂用帳戶中管理資源。 這種存取控制稱為角色型存取控制 (RBAC)。 若要管理此存取權，您可以使用 [Azure 入口網站](https://portal.azure.com/)、[Azure CLI 工具](../cli-install-nodejs.md)、[PowerShell](/powershell/azureps-cmdlets-docs) 或 [Azure Resource Manager REST API](https://msdn.microsoft.com/library/azure/dn906885.aspx)。

透過 Azure Resource Manager 模型，您可以在資源群組中建立金鑰保存庫，並使用 Azure Active Directory 來控制此金鑰保存庫之管理平面的存取權。 例如，您可以對使用者或群組授與在特定資源群組中管理金鑰保存庫的能力。

您可以藉由指派適當的 RBAC 角色，在特定範圍對使用者、群組和應用程式授與存取權。 例如，若要對使用者授與管理金鑰保存庫的權限，您會在特定範圍對此使用者指派預先定義的角色「金鑰保存庫參與者」。 在此案例中，範圍會是訂用帳戶、資源群組或只是特定金鑰保存庫。 在訂用帳戶層級指派的角色，會套用至該訂用帳戶內的所有資源群組和資源。 在資源群組層級指派的角色，會套用至該資源群組內的所有資源。 針對特定資源指派的角色，則只會套用至該資源。 有數個預先定義的角色 (請參閱 [RBAC︰內建角色](../role-based-access-control/built-in-roles.md))，如果預先定義的角色不符合您的需求，您也可以定義自己的角色。

> [!IMPORTANT]
> 請注意，如果使用者具有金鑰保存庫管理平面的參與者權限 (RBAC)，她可以藉由設定可控制資料平面存取權的金鑰保存庫存取原則，對自己授與資料平面的存取權。 因此，建議您嚴格控制擁有金鑰保存庫「參與者」權限的人員，以確保只有獲得授權的人員可以存取和管理金鑰保存庫、金鑰、密碼和憑證。
> 
> 

## <a name="data-plane-access-control"></a>資料平面存取控制
金鑰保存庫資料平面是由會影響金鑰保存庫物件 (例如金鑰、密碼和憑證) 的作業所組成。  這包括金鑰作業 (例如建立、匯入、更新、列出、備份和還原金鑰)、密碼編譯作業 (例如簽署、驗證、加密、解密、包裝和解除包裝)，以及為金鑰設定標籤和其他屬性。 同樣地，針對密碼所包含的是取得、設定、列出、刪除。

資料平面的存取權是藉由設定金鑰保存庫的存取原則來授與。 使用者、群組或應用程式必須具有管理平面的參與者權限 (RBAC)，金鑰保存庫才能設定該金鑰保存庫的存取原則。 您可以對使用者、群組或應用程式授與權限，來為金鑰保存庫中的金鑰或密碼執行特定作業。 金鑰保存庫最多可支援 1024 個存取原則項目。 建立 Azure Active Directory 安全性群組並在該群組中新增使用者，即可對數名使用者授與金鑰保存庫的資料平面存取權。

### <a name="key-vault-access-policies"></a>金鑰保存庫存取原則
金鑰保存庫存取原則可分別授與金鑰、密碼和憑證的權限。 例如，您可以只提供金鑰存取權給使用者，但不提供密碼的權限。 不過，金鑰、密碼或憑證的存取權限是在保存庫層級。 換句話說，金鑰保存庫存取原則不支援物件層級權限。 您可以使用 [Azure 入口網站](https://portal.azure.com/)、[Azure CLI 工具](../cli-install-nodejs.md)、[PowerShell](/powershell/azureps-cmdlets-docs) 或[金鑰保存庫管理 REST API](https://msdn.microsoft.com/library/azure/mt620024.aspx) 來設定金鑰保存庫的存取原則。

> [!IMPORTANT]
> 請注意，金鑰保存庫存取原則會套用在保存庫層級。 例如，對使用者授與金鑰的建立和刪除權限時，她就可以對該金鑰保存庫的所有金鑰執行這些作業。

除了存取原則，您也可以設定[防火牆和虛擬網路規則](key-vault-network-security.md)，利用 [Azure Key Vault 的虛擬網路服務端點](key-vault-overview-vnet-service-endpoints.md)來限制資料層存取，以多加一道安全性。

## <a name="example"></a>範例
假設您正在開發的應用程式使用 SSL 憑證、使用 Azure 儲存體來儲存資料，並使用 RSA 2048 位元金鑰來進行簽署作業。 假設此應用程式正在 VM (或 VM 擴展集) 中執行。 您可以使用金鑰保存庫來儲存所有應用程式密碼，並使用金鑰保存庫來儲存應用程式用來向 Azure Active Directory 進行驗證的啟動程序憑證。

那麼，以下是金鑰保存庫中儲存之所有金鑰和密碼的摘要。

* **SSL 憑證** - 用於 SSL
* **儲存體金鑰** - 用來取得儲存體帳戶的存取權
* **RSA 2048 位元金鑰** - 用於簽署作業
* **啟動程序憑證** - 用來向 Azure Active Directory 進行驗證，取得金鑰保存庫的存取權以擷取儲存體金鑰並使用 RSA 金鑰來進行簽署。

現在讓我們看看負責管理、部署和稽核此應用程式的人員。 在此範例中，我們會使用三種角色。

* **安全性小組** - 這些人通常是「CSO (資訊安全長) 辦公室」的 IT 人員或地位相當的人員，需負責妥善保管機密資料，像是 SSL 憑證、用於進行簽署的 RSA 金鑰、資料庫的連接字串和儲存體帳戶金鑰。
* **開發人員/操作員** - 這些人需負責開發此應用程式，然後將它部署在 Azure 中。 一般而言，它們不屬於安全性小組，因此不應該擁有任何機密資料 (像是 SSL 憑證、RSA 金鑰) 的存取權，但他們所部署的應用程式應該擁有這些項目的存取權。
* **稽核員** - 這通常是一組不同人員，與開發人員和一般 IT 人員有所區隔。 他們的責任是檢閱憑證、金鑰等項目的使用和維護情形是否適當，並確保各項作業符合資料安全性標準。 

還有一種角色不在此應用程式的涵蓋範圍內，但有所相關所以在此提出，該角色就是訂用帳戶 (或資源群組) 管理員。 訂用帳戶管理員會設定安全性小組的初始存取權限。 這裡我們假設，訂用帳戶管理員已對安全性小組授與資源群組的存取權，該群組中含有此應用程式所需的所有資源。

現在我們來看看，每個角色會在此應用程式內容中執行的動作。

* **安全性小組**
  * 建立金鑰保存庫
  * 開啟金鑰保存庫記錄
  * 新增金鑰/密碼
  * 建立金鑰備份以用於災害復原
  * 設定金鑰保存庫存取原則，對使用者和應用程式授與執行特定作業的權限
  * 定期輪替金鑰/密碼
* **開發人員/操作員**
  * 從安全性小組取得啟動程序和 SSL 憑證 (指紋)、儲存體金鑰 (密碼 URI) 和簽署金鑰 (金鑰 URI) 的參考
  * 開發和部署以程式設計方式存取金鑰和密碼的應用程式
* **稽核員**
  * 檢閱使用記錄，確認金鑰/密碼的使用是否適當，以及是否符合資料安全性標準

現在讓我們看看，每個角色 (和應用程式) 需要哪些金鑰保存庫存取權限才能執行對其指派的工作。 

| 使用者角色 | 管理平面權限 | 資料平面權限 |
| --- | --- | --- |
| 安全性小組 |金鑰保存庫參與者 |金鑰︰備份、建立、刪除、取得、匯入、列出、還原 <br> 密碼︰全部 |
| 開發人員/操作員 |金鑰保存庫部署權限，以便其所部署的 VM 可以擷取金鑰保存庫中的密碼 |None |
| 稽核員 |None |金鑰︰列出<br>密碼︰列出 |
| 應用程式 |None |金鑰︰簽署<br>密碼︰取得 |

> [!NOTE]
> 稽核員需要金鑰和密碼的列出權限，以便可以檢查記錄檔中不會顯現之金鑰和密碼的屬性，例如標籤、啟用日和到期日。
> 
> 

除了金鑰保存庫的權限外，這三個角色還需要其他資源的存取權。 例如，為了能夠部署 VM (或 Web Apps 等)。開發人員/操作員還需要這些資源類型的「參與者」存取權。 稽核員需要金鑰保存庫記錄檔儲存所在之儲存體帳戶的讀取權限。

本文的重點是保護金鑰保存庫的存取權，因此只說明與本主題相關的部分，並略過有關以程式設計方式部署憑證、存取金鑰和密碼等等的詳細資料。這些詳細資料已在其他地方做過說明。 將金鑰保存庫中儲存的憑證部署至 VM 是在[部落格文章](https://blogs.technet.microsoft.com/kv/2016/09/14/updated-deploy-certificates-to-vms-from-customer-managed-key-vault/)中說明，裡面有提供[範例程式碼](https://www.microsoft.com/download/details.aspx?id=45343)來說明，如何使用啟動程序憑證向 Azure AD 進行驗證以取得金鑰保存庫的存取權。

大多數存取權限皆可使用 Azure 入口網站來授與，但若要授與更細微的權限，您可能需要使用 Azure PowerShell (或 Azure CLI) 來達到所需結果。 

下列 PowerShell 程式碼片段假設︰

* Azure Active Directory 系統管理員已建立安全性群組來代表三種角色，分別是 Contoso Security Team、Contoso App Devops、Contoso App Auditors。 系統管理員也已將使用者新增至所屬群組。
* **ContosoAppRG** 是所有資源所在的資源群組。 **contosologstorage** 是記錄檔的儲存位置。 
* 金鑰保存庫 **ContosoKeyVault** 和用於金鑰保存庫記錄檔的儲存體帳戶 **contosologstorage** 必須位在相同的 Azure 位置

首先，訂用帳戶管理員會對安全性小組指派「金鑰保存庫參與者」和「使用者存取系統管理員」角色。 這可讓安全性小組管理其他資源的存取權，以及管理資源群組 ContosoAppRG 中的金鑰保存庫。

```
New-AzureRmRoleAssignment -ObjectId (Get-AzureRmADGroup -SearchString 'Contoso Security Team')[0].Id -RoleDefinitionName "key vault Contributor" -ResourceGroupName ContosoAppRG
New-AzureRmRoleAssignment -ObjectId (Get-AzureRmADGroup -SearchString 'Contoso Security Team')[0].Id -RoleDefinitionName "User Access Administrator" -ResourceGroupName ContosoAppRG
```

下列指令碼說明安全性小組要如何建立金鑰保存庫、設定記錄，以及設定其他角色和應用程式的存取權限。 

```
# Create key vault and enable logging
$sa = Get-AzureRmStorageAccount -ResourceGroup ContosoAppRG -Name contosologstorage
$kv = New-AzureRmKeyVault -VaultName ContosoKeyVault -ResourceGroup ContosoAppRG -SKU premium -Location 'westus' -EnabledForDeployment
Set-AzureRmDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $true -Categories AuditEvent

# Data plane permissions for Security team
Set-AzureRmKeyVaultAccessPolicy -VaultName ContosoKeyVault -ObjectId (Get-AzureRmADGroup -SearchString 'Contoso Security Team')[0].Id -PermissionsToKeys backup,create,delete,get,import,list,restore -PermissionsToSecrets all

# Management plane permissions for Dev/ops
# Create a new role from an existing role
$devopsrole = Get-AzureRmRoleDefinition -Name "Virtual Machine Contributor"
$devopsrole.Id = $null
$devopsrole.Name = "Contoso App Devops"
$devopsrole.Description = "Can deploy VMs that need secrets from key vault"
$devopsrole.AssignableScopes = @("/subscriptions/<SUBSCRIPTION-GUID>")

# Add permission for dev/ops so they can deploy VMs that have secrets deployed from key vaults
$devopsrole.Actions.Add("Microsoft.KeyVault/vaults/deploy/action")
New-AzureRmRoleDefinition -Role $devopsrole

# Assign this newly defined role to Dev ops security group
New-AzureRmRoleAssignment -ObjectId (Get-AzureRmADGroup -SearchString 'Contoso App Devops')[0].Id -RoleDefinitionName "Contoso App Devops" -ResourceGroupName ContosoAppRG

# Data plane permissions for Auditors
Set-AzureRmKeyVaultAccessPolicy -VaultName ContosoKeyVault -ObjectId (Get-AzureRmADGroup -SearchString 'Contoso App Auditors')[0].Id -PermissionsToKeys list -PermissionsToSecrets list
```

所定義的自訂角色只能指派給 ContosoAppRG 資源群組建立所在的訂用帳戶。 如果相同的自訂角色會用於其他訂用帳戶中的其他專案，則其範圍可以新增更多訂用帳戶。

可為開發人員/操作員指派「部署/動作」權限的自訂角色侷限於資源群組。 如此一來，只有在資源群組「ContosoAppRG」中建立的 VM 會取得機密資料 (SSL 憑證和啟動程序憑證)。 開發人員/操作員小組成員在其他資源群組中建立的任何 VM 即使知道機密資料 URI，也無法取得這些機密資料。

此範例描述的是簡單案例。 真實案例可能會更複雜，您可能需要根據需求調整金鑰保存庫的權限。 例如，在我們所舉的範例中，我們假設該安全性小組會提供開發人員/操作員小組需要在其應用程式中參考的金鑰和密碼參考 (URI 和指紋)。 因此，他們不需要對開發人員/操作員授與任何資料平面存取權。 另請注意，此範例的內容著重在保護金鑰保存庫。 若要保護[您的 VM](https://azure.microsoft.com/services/virtual-machines/security/)、[儲存體帳戶](../storage/common/storage-security-guide.md)和其他 Azure 資源，也應提供類似考量。

> [!NOTE]
> 注意︰此範例示範生產環境中會如何鎖定金鑰保存庫的存取權。 開發人員應該有自己的訂用帳戶或資源群組，並擁有完整權限以便管理他們用來開發應用程式的保存庫、VM 和儲存體帳戶。

強烈建議[設定 Key Vault 防火牆和虛擬網路](key-vault-network-security.md)，以進一步保護對金鑰保存庫的存取。

## <a name="resources"></a>資源
* [Azure Active Directory 角色型存取控制](../role-based-access-control/role-assignments-portal.md)
  
  本文說明 Azure Active Directory 角色型存取控制及其運作方式。
* [RBAC：內建角色](../role-based-access-control/built-in-roles.md)
  
  本文詳細說明 RBAC 中所有可用的內建角色。
* [了解資源管理員部署和傳統部署](../azure-resource-manager/resource-manager-deployment-model.md)
  
  本文說明資源管理員部署和傳統部署模型，並說明使用資源管理員和資源群組的優點
* [使用 Azure PowerShell 管理角色型存取控制](../role-based-access-control/role-assignments-powershell.md)
  
  本文說明如何使用 Azure PowerShell 管理角色型存取控制
* [使用 REST API 管理角色型存取控制](../role-based-access-control/role-assignments-rest.md)
  
  本文說明如何使用 REST API 來管理 RBAC。
* [來自Ignite 且適用於 Microsoft Azure 的角色型存取控制](https://channel9.msdn.com/events/Ignite/2015/BRK2707)
  
  這個連結會連到 2015 MS Ignite 會議的 Channel 9 上的視訊。 在這個研討會中，他們討論 Azure 中的存取管理和報告功能，並探索使用 Azure Active Directory 安全存取 Azure 訂用帳戶的最佳做法。
* [使用 OAuth 2.0 和 Azure Active Directory 授權存取 Web 應用程式](../active-directory/develop/v1-protocols-oauth-code.md)
  
  本文說明向 Azure Active Directory 進行驗證的完整 OAuth 2.0 流程。
* [金鑰保存庫管理 REST API](https://msdn.microsoft.com/library/azure/mt620024.aspx)
  
  本文件是可供以程式設計方式管理金鑰保存庫 (包括設定金鑰保存庫存取原則) 的 REST API 參考。
* [金鑰保存庫 REST API](https://msdn.microsoft.com/library/azure/dn903609.aspx)
  
  金鑰保存庫 REST API 參考文件的連結。
* [金鑰存取控制](https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_KeyAccessControl)
  
  密碼存取控制參考文件的連結。
* [密碼存取控制](https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_SecretAccessControl)
  
  金鑰存取控制參考文件的連結。
* 使用 PowerShell [設定](https://docs.microsoft.com/powershell/module/azurerm.keyvault/Set-AzureRmKeyVaultAccessPolicy)和[移除](https://docs.microsoft.com/powershell/module/azurerm.keyvault/Remove-AzureRmKeyVaultAccessPolicy)金鑰保存庫存取原則
  
  用來管理金鑰保存庫存取原則之 PowerShell Cmdlet 參考文件的連結。

## <a name="next-steps"></a>後續步驟
[設定 Key Vault 防火牆和虛擬網路](key-vault-network-security.md)

如需適用於系統管理員的開始使用教學課程，請參閱[開始使用 Azure 金鑰保存庫](key-vault-get-started.md)。

如需金鑰保存庫使用方法記錄的詳細資訊，請參閱 [Azure 金鑰保存庫記錄](key-vault-logging.md)。

如需搭配 Azure 金鑰保存庫使用金鑰和密碼的詳細資訊，請參閱[關於金鑰和密碼](https://msdn.microsoft.com/library/azure/dn903623.aspx)。

如果您有關於金鑰保存庫的問題，請造訪 [Azure 金鑰保存庫論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=AzureKeyVault)

