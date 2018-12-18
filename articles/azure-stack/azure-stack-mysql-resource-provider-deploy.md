---
title: 在 Azure Stack 上使用 MySQL 資料庫 | Microsoft Docs
description: 了解如何在 Azure Stack 上部署 MySQL 資料庫即服務，並了解部署 MySQL Server 資源提供者配接器的快速步驟。
services: azure-stack
documentationCenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/05/2018
ms.author: jeffgilb
ms.reviewer: jeffgo
ms.openlocfilehash: 722df244135d045e18b9f2d0dd88066ba00b7d49
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43841874"
---
# <a name="deploy-the-mysql-resource-provider-on-azure-stack"></a>在 Azure Stack 上部署 MySQL 資源提供者

使用 MySQL Server 資源提供者將 MySQL 資料庫公開成 Azure Stack 服務。 MySQL 資源提供者在 Windows Server 2016 Server Core 虛擬機器 (VM) 上會以服務的形式執行。

> [!IMPORTANT]
> 僅支援資源提供者在裝載 SQL 或 MySQL 的伺服器上建立項目。 在不是由資源提供者建立的主機伺服器上建立項目，可能會導致不相符的狀態。

## <a name="prerequisites"></a>必要條件

您必須先滿足數個先決條件，才能部署 Azure Stack MySQL 資源提供者。 為了滿足這些需求，請在能夠存取具特殊權限端點 VM 的電腦上完成本文中的步驟。

* 如果您尚未這麼做，請向 Azure [註冊 Azure Stack](.\azure-stack-registration.md)，以便下載 Azure Marketplace 項目。
* 您必須在將執行此安裝所在的系統上，安裝 Azure 和 Azure Stack PowerShell 模組。 該系統必須是包含最新版 .NET 執行階段的 Windows 10 或 Windows Server 2016 映像。 請參閱[安裝適用於 Azure Stack 的 PowerShell](.\azure-stack-powershell-install.md)。
* 透過下載 **Windows Server 2016 Datacenter - Server Core** 映像，將必要的 Windows Server Core VM 新增到 Azure Stack Marketplace。

* 下載 MySQL 資源提供者二進位檔，然後執行自我解壓縮程式，以將內容解壓縮到至暫存目錄。

  >[!NOTE]
  >若要在無法存取網際網路的系統上部署 MySQL 提供者，請將 [mysql-connector-net-6.10.5.msi](https://dev.mysql.com/get/Downloads/Connector-Net/mysql-connector-net-6.10.5.msi) 檔案複製到本機路徑。 使用 **DependencyFilesLocalPath** 參數提供路徑名稱。

* 資源提供者會有最低限度的相對應 Azure Stack 組建。

    | 最低 Azure Stack 版本 | MySQL RP 版本|
    | --- | --- |
    | 1804 版 (1.0.180513.1)|[MySQL RP 1.1.24.0 版](https://aka.ms/azurestackmysqlrp1804) |
    |     |     |

* 請確定已符合資料中心整合必要條件：

    |必要條件|參考|
    |-----|-----|
    |條件式 DNS 轉送已正確設定。|[Azure Stack 資料中心整合 - DNS](azure-stack-integrate-dns.md)|
    |資源提供者的輸入連接埠已開啟。|[Azure Stack 資料中心整合 - 發佈端點](azure-stack-integrate-endpoints.md#ports-and-protocols-inbound)|
    |PKI 憑證主體和 SAN 已正確設定。|[Azure Stack 部署必要 PKI 必要條件](azure-stack-pki-certs.md#mandatory-certificates)[Azure Stack 部署 PaaS 憑證必要條件](azure-stack-pki-certs.md#optional-paas-certificates)|
    |     |     |

### <a name="certificates"></a>憑證

_僅適用於整合式系統安裝_。 您必須提供 [Azure Stack 部署 PKI 需求](.\azure-stack-pki-certs.md#optional-paas-certificates)中選擇性 PaaS 憑證一節所述的 SQL PaaS PKI 憑證。 請將 .pfx 檔案放在 **DependencyFilesLocalPath** 參數所指定的位置中。 不要提供 ASDK 系統的憑證。

## <a name="deploy-the-resource-provider"></a>部署資源提供者

安裝妥所有先決條件項目之後，請執行 **DeployMySqlProvider.ps1** 指令碼來部署 MYSQL 資源提供者。 DeployMySqlProvider.ps1 指令碼是從您針對 Azure Stack 版本下載的 MySQL 資源提供者二進位檔中解壓縮而來。

若要部署 MySQL 資源提供者，請開啟新的已提升權限 PowerShell 視窗 (不是 PowerShell ISE)，然後變更至您解壓縮 MySQL 資源提供者二進位檔的目錄。 建議您使用新的 PowerShell 視窗，以避免已載入的 PowerShell 模組可能造成的問題。

請執行 **DeployMySqlProvider.ps1** 指令碼，這會完成下列工作：

* 將憑證和其他成品上傳到 Azure Stack 上的儲存體帳戶。
* 發佈資源庫套件，以便使用資源庫來部署 MySQL 資料庫。
* 發佈用於部署主控伺服器的資源庫套件。
* 使用您已下載的 Windows Server 2016 核心映像來部署 VM，然後安裝 MySQL 資源提供者。
* 註冊與您資源提供者 VM 對應的本機 DNS 記錄。
* 針對操作員帳戶，向本機 Azure Resource Manager 註冊您的資源提供者。

> [!NOTE]
> 當 MySQL 資源提供者部署開始時，會建立 **system.local.mysqladapter** 資源群組。 最多可能需要 75 分鐘的時間，才能完成對此資源群組的必要部署。

### <a name="deploymysqlproviderps1-parameters"></a>DeployMySqlProvider.ps1 參數

您可以從命令列指定這些參數。 如果未指定參數，或任何參數驗證失敗，系統就會提示您提供所需的參數。

| 參數名稱 | 說明 | 註解或預設值 |
| --- | --- | --- |
| **CloudAdminCredential** | 雲端管理員的認證，這是存取具特殊權限端點所需的認證。 | _必要_ |
| **AzCredential** | Azure Stack 服務管理帳戶的認證。 使用與部署 Azure Stack 時所用認證相同的認證。 | _必要_ |
| **VMLocalCredential** | MySQL 資源提供者 VM 之本機系統管理員帳戶的認證。 | _必要_ |
| **PrivilegedEndpoint** | 具特殊權限端點的 IP 位址或 DNS 名稱。 |  _必要_ |
| **AzureEnvironment** | 您用來部署 Azure Stack 的服務管理員帳戶所屬的 Azure 環境。 只有在不是 ADFS 時才需要。 支援的環境名稱為 **AzureCloud**、**AzureUSGovernment**，或如果使用中國 Azure Active Directory，則為 **AzureChinaCloud**。 | AzureCloud |
| **DependencyFilesLocalPath** | 您的憑證 .pfx 檔案必須放在這個目錄中 (僅適用於整合式系統)。 若是已中斷連線的環境，請將 [mysql-connector-net-6.10.5.msi](https://dev.mysql.com/get/Downloads/Connector-Net/mysql-connector-net-6.10.5.msi) 下載到這個目錄。 您可以在這裡選擇性地複製一個 Windows Update MSU 套件。 | _選擇性_ (對於整合式系統或已中斷連線的環境是_強制_的) |
| **DefaultSSLCertificatePassword** | .pfx 憑證的密碼。 | _必要_ |
| **MaxRetryCount** | 作業失敗時，您想要重試每個作業的次數。| 2 |
| **RetryDuration** | 重試之間的逾時間隔 (秒)。 | 120 |
| **解除安裝** | 移除資源提供者和所有關聯的資源 (請參閱下面的附註)。 | 否 |
| **DebugMode** | 防止在失敗時自動清除。 | 否 |
| **AcceptLicense** | 略過接受 GPL 授權的提示。  <http://www.gnu.org/licenses/old-licenses/gpl-2.0.html> | |

## <a name="deploy-the-mysql-resource-provider-using-a-custom-script"></a>使用自訂指令碼來部署 MySQL 資源提供者

若要在部署資源提供者時免除任何手動設定，您可以自訂下列指令碼。 請視需要針對您的 Azure Stack 部署，變更預設帳戶資訊和密碼。

```powershell
# Install the AzureRM.Bootstrapper module, set the profile and install the AzureStack module
Install-Module -Name AzureRm.BootStrapper -Force
Use-AzureRmProfile -Profile 2017-03-09-profile
Install-Module -Name AzureStack -RequiredVersion 1.4.0

# Use the NetBIOS name for the Azure Stack domain. On the Azure Stack SDK, the default is AzureStack but could have been changed at install time.
$domain = "AzureStack"  

# For integrated systems, use the IP address of one of the ERCS virtual machines
$privilegedEndpoint = "AzS-ERCS01"

# Point to the directory where the resource provider installation files were extracted.
$tempDir = 'C:\TEMP\MYSQLRP'

# The service admin account (can be Azure Active Directory or Active Directory Federation Services).
$serviceAdmin = "admin@mydomain.onmicrosoft.com"
$AdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$AdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $AdminPass)

# Set the credentials for the new resource provider VM local administrator account
$vmLocalAdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ("mysqlrpadmin", $vmLocalAdminPass)

# And the cloudadmin credential required for privileged endpoint access.
$CloudAdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$CloudAdminCreds = New-Object System.Management.Automation.PSCredential ("$domain\cloudadmin", $CloudAdminPass)

# Change the following as appropriate.
$PfxPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force

# Change to the directory folder where you extracted the installation files. Do not provide a certificate on ASDK!
. $tempDir\DeployMySQLProvider.ps1 `
    -AzCredential $AdminCreds `
    -VMLocalCredential $vmLocalAdminCreds `
    -CloudAdminCredential $cloudAdminCreds `
    -PrivilegedEndpoint $privilegedEndpoint `
    -DefaultSSLCertificatePassword $PfxPass `
    -DependencyFilesLocalPath $tempDir\cert `
    -AcceptLicense

```

當資源提供者安裝指令碼完成時，請重新整理您的瀏覽器，以確定可以看到最新的更新。

## <a name="verify-the-deployment-by-using-the-azure-stack-portal"></a>使用 Azure Stack 入口網站來確認部署是否成功

1. 以服務管理員身分登入管理入口網站。
2. 選取 [資源群組]
3. 選取 [**system.\<位置\>.mysqladapter**] 資源群組。
4. 在資源群組概觀的摘要頁面上，應該沒有失敗的部署。
5. 最後，在管理入口網站中選取 [虛擬機器]，以驗證 MySQL 資源提供者 VM 已成功建立並執行。

## <a name="next-steps"></a>後續步驟

[新增主控伺服器](azure-stack-mysql-resource-provider-hosting-servers.md)
