---
title: 建立 Azure Service Fabric 叢集 | Microsoft Docs
description: 了解如何使用 Azure Resource Manager 在 Azure 中設定安全的 Service Fabric 叢集。  您可以使用預設範本或您自己的叢集範本來建立叢集。
services: service-fabric
documentationcenter: .net
author: aljo-microsoft
manager: timlt
editor: chackdan
ms.assetid: 15d0ab67-fc66-4108-8038-3584eeebabaa
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/16/2018
ms.author: aljo
ms.openlocfilehash: b71c1d75dc9d988c7b26f9720cc03d9fee9109b1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970306"
---
# <a name="create-a-service-fabric-cluster-using-azure-resource-manager"></a>使用 Azure Resource Manager 來建立 Service Fabric 叢集 
> [!div class="op_single_selector"]
> * [Azure Resource Manager](service-fabric-cluster-creation-via-arm.md)
> * [Azure 入口網站](service-fabric-cluster-creation-via-portal.md)
>
>

[Azure Service Fabric 叢集](service-fabric-deploy-anywhere.md)是一組透過網路連線的虛擬機器，您可以將微服務部署到其中並進行管理。  在 Azure 中執行的 Service Fabric 叢集是 Azure 資源，而且是使用 Azure Resource Manager 部署的。 本文說明如何使用 Resource Manager 在 Azure 中部署安全的 Service Fabric 叢集。 您可以使用預設叢集範本或自訂範本。  如果您還沒有自訂範本，您可以[了解如何建立自訂範本](service-fabric-cluster-creation-create-template.md)。

叢集安全性是在第一次設定叢集時所設定，而且稍後無法變更。 設定叢集之前，請閱讀 [Service Fabric 叢集安全性案例][service-fabric-cluster-security]。 在 Azure 中，Service Fabric 使用 x509 憑證來保護您的叢集與其端點、驗證用戶端，以及加密資料。 也建議您使用 Azure Active Directory 來保護對管理端點的存取。 必須在建立叢集之前先建立 Azure AD 租用戶與使用者。  如需詳細資訊，請閱讀[設定 Azure AD 以驗證用戶端](service-fabric-cluster-creation-setup-aad.md)。

部署生產叢集以執行生產工作負載之前，請務必先閱讀[生產整備檢查清單](service-fabric-production-readiness-checklist.md)。

## <a name="prerequisites"></a>必要條件 
在本文中使用 Service Fabric RM Powershell 或 Azure CLI 模組來部署叢集：

* [Azure PowerShell 4.1 和更新版本][azure-powershell]
* [Azure CLI 2.0 版與更新版本][azure-CLI]

您可以在此找到 Service Fabric 模組的參考文件：
* [AzureRM.ServiceFabric](https://docs.microsoft.com/powershell/module/azurerm.servicefabric)
* [az SF CLI module](https://docs.microsoft.com/cli/azure/sf?view=azure-cli-latest)

### <a name="sign-in-to-azure"></a>登入 Azure

在執行本文中的任何命令之前，請先登入 Azure。

```powershell
Connect-AzureRmAccount
Set-AzureRmContext -SubscriptionId <subscriptionId>
```

```azurecli
az login
az account set --subscription $subscriptionId
```

## <a name="create-a-new-cluster-using-a-system-generated-self-signed-certificate"></a>使用系統產生的自我簽署憑證建立新叢集

若使用下列命令來建立叢集，此叢集會受到系統產生的自我簽署憑證保護。 此命令會設定主要叢集憑證，該憑證可用於叢集安全性，以及用來設定管理員存取權，以便使用該憑證來執行管理作業。  自我簽署憑證可用於保護測試叢集。  應使用憑證授權單位 (CA) 提供的憑證來保護生產叢集。

### <a name="use-the-default-cluster-template-that-ships-in-the-module"></a>使用模組中隨附的預設叢集範本

使用預設範本與下列命令，以最少的參數數目快速建立叢集。

所用範本可於 [Azure Service Fabric 範本範例：Windows 範本](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure-NSG)和 [Ubuntu 範本](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Ubuntu-1-NodeTypes-Secure)上取得

下列命令可以建立 Windows 或 Linux 叢集，您需要指定適用的作業系統。 PowerShell/CLI 命令也會在指定的 CertificateOutputFolder 中輸出憑證，但是請確定已建立憑證資料夾。 此命令也需要其他參數如 VM SKU。

> [!NOTE]
> 下列 PowerShell 命令僅適用於 Azure Resource Manager PowerShell 6.1 以上的版本。 若要檢查 Azure Resource Manager PowerShell 的目前版本，請執行下列 PowerShell 命令 "Get-Module AzureRM"。 遵循[此連結](/powershell/azure/install-azurerm-ps?view=azurermps-6.3.0)即可升級 Azure Resource Manager PowerShell 版本。 
>
>

使用 PowerShell 部署叢集：

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$vaultName="myvault"
$vaultResourceGroupName="myvaultrg"
$CertSubjectName="mycluster.westus.cloudapp.azure.com"
$certPassword="Password123!@#" | ConvertTo-SecureString -AsPlainText -Force 
$vmpassword="Password4321!@#" | ConvertTo-SecureString -AsPlainText -Force
$vmuser="myadmin"
$os="WindowsServer2016DatacenterwithContainers"
$certOutputFolder="c:\certificates"

New-AzureRmServiceFabricCluster -ResourceGroupName $resourceGroupName -Location $resourceGroupLocation -CertificateOutputFolder $certOutputFolder -CertificatePassword $certpassword -CertificateSubjectName $CertSubjectName -OS $os -VmPassword $vmpassword -VmUserName $vmuser
```

使用 Azure CLI 部署叢集：

```azurecli
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare vaultResourceGroupName="myvaultrg"
declare vaultName="myvault"
declare CertSubjectName="mylinux.westus.cloudapp.azure.com"
declare vmpassword="Password!1"
declare certpassword="Password!4321"
declare vmuser="myadmin"
declare vmOs="UbuntuServer1604"
declare certOutputFolder="c:\certificates"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-output-folder $certOutputFolder --certificate-password $certpassword  \
    --vault-name $vaultName --vault-resource-group $resourceGroupName  \
    --template-file $templateFilePath --parameter-file $parametersFilePath --vm-os $vmOs  \
    --vm-password $vmpassword --vm-user-name $vmuser
```

### <a name="use-your-own-custom-template"></a>使用您自己的自訂範本

如果您需要撰寫自訂範本以符合需求，強烈建議您使用 [Azure Service Fabric 範本範例](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master)上提供的其中一個範本。 了解如何[自訂您的叢集範本][customize-your-cluster-template]。

如果您已有自訂範本，則請再次確認範本及參數檔案中三個與憑證相關的參數，其命名如下所示，且值也如下所示為 Null：

```json
   "certificateThumbprint": {
      "value": ""
    },
    "sourceVaultValue": {
      "value": ""
    },
    "certificateUrlValue": {
      "value": ""
    },
```

使用 PowerShell 部署叢集：

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$CertSubjectName="mycluster.westus.cloudapp.azure.com"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$certOutputFolder="c:\certificates"

$parameterFilePath="c:\mytemplates\mytemplateparm.json"
$templateFilePath="c:\mytemplates\mytemplate.json"

New-AzureRmServiceFabricCluster -ResourceGroupName $resourceGroupName -CertificateOutputFolder $certOutputFolder -CertificatePassword $certpassword -CertificateSubjectName $CertSubjectName -TemplateFile $templateFilePath -ParameterFile $parameterFilePath 
```

使用 Azure CLI 部署叢集：

```azurecli
declare certPassword=""
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare certSubjectName="mylinuxsecure.westus.cloudapp.azure.com"
declare parameterFilePath="c:\mytemplates\linuxtemplateparm.json"
declare templateFilePath="c:\mytemplates\linuxtemplate.json"
declare certOutputFolder="c:\certificates"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-output-folder $certOutputFolder --certificate-password $certPassword  \
    --certificate-subject-name $certSubjectName \
    --template-file $templateFilePath --parameter-file $parametersFilePath
```

## <a name="create-a-new-cluster-using-your-own-x509-certificate"></a>使用您自己的 X.509 憑證建立新叢集

如果您有想要用來保護叢集的憑證，請使用下列命令建立叢集。

如果這是 CA 簽署的憑證，而您最後會將其用於其他用途，建議您特別針對金鑰保存庫提供不同的資源群組。 建議您將 Key Vault 放入其自己的資源群組中。 此動作可讓您移除計算和儲存體資源群組 (包括含有 Service Fabric 叢集的資源群組)，而不會遺失您的金鑰和密碼。 **含有您金鑰保存庫的資源群組*必須與正在使用它的叢集位於相同區域*。**

### <a name="use-the-default-five-node-one-node-type-template-that-ships-in-the-module"></a>使用預設 5 節點 (模組中隨附的節點類型範本)
所用範本可於 [Azure 範例：Windows 範本](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure-NSG)和 [Ubuntu 範本](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Ubuntu-1-NodeTypes-Secure)上取得

使用 PowerShell 部署叢集：

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mylinux"
$vaultName="myvault"
$vaultResourceGroupName="myvaultrg"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$vmpassword=("Password!4321" | ConvertTo-SecureString -AsPlainText -Force) 
$vmuser="myadmin"
$os="WindowsServer2016DatacenterwithContainers"

New-AzureRmServiceFabricCluster -ResourceGroupName $resourceGroupName -Location $resourceGroupLocation -KeyVaultResouceGroupName $vaultResourceGroupName -KeyVaultName $vaultName -CertificateFile C:\MyCertificates\chackocertificate3.pfx -CertificatePassword $certPassword -OS $os -VmPassword $vmpassword -VmUserName $vmuser 
```

使用 Azure CLI 部署叢集：

```azurecli
declare vmPassword="Password!1"
declare certPassword="Password!1"
declare vmUser="myadmin"
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare vaultResourceGroupName="myvaultrg"
declare vaultName="myvault"
declare certificate-file="c:\certificates\mycert.pem"
declare vmOs="UbuntuServer1604"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-file $certificate-file --certificate-password $certPassword  \
    --vault-name $vaultName --vault-resource-group $vaultResourceGroupName  \
    --vm-os vmOs \
    --vm-password $vmPassword --vm-user-name $vmUser
```

### <a name="use-your-own-custom-cluster-template"></a>使用您自己的自訂叢集範本
如果您需要撰寫自訂範本以符合需求，強烈建議您使用 [Azure Service Fabric 範本範例](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master)上提供的其中一個範本。 了解如何[自訂您的叢集範本][customize-your-cluster-template]。

如果您已有自訂範本，則請再次確認範本及參數檔案中三個與憑證相關的參數，其命名如下所示，且值也如下所示為 Null。

```json
   "certificateThumbprint": {
      "value": ""
    },
    "sourceVaultValue": {
      "value": ""
    },
    "certificateUrlValue": {
      "value": ""
    },
```

使用 PowerShell 部署叢集：

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mylinux"
$vaultName="myvault"
$vaultResourceGroupName="myvaultrg"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$os="WindowsServer2016DatacenterwithContainers"
$parameterFilePath="c:\mytemplates\mytemplateparm.json"
$templateFilePath="c:\mytemplates\mytemplate.json"
$certificateFile="C:\MyCertificates\chackonewcertificate3.pem"

New-AzureRmServiceFabricCluster -ResourceGroupName $resourceGroupName -Location $resourceGroupLocation -TemplateFile $templateFilePath -ParameterFile $parameterFilePath -KeyVaultResouceGroupName $vaultResourceGroupName -KeyVaultName $vaultName -CertificateFile $certificateFile -CertificatePassword $certPassword
```

使用 Azure CLI 部署叢集：

```azurecli
declare certPassword="Password!1"
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare vaultResourceGroupName="myvaultrg"
declare vaultName="myvault"
declare parameterFilePath="c:\mytemplates\linuxtemplateparm.json"
declare templateFilePath="c:\mytemplates\linuxtemplate.json"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-file $certificate-file --certificate-password $password  \
    --vault-name $vaultName --vault-resource-group $vaultResourceGroupName  \
    --template-file $templateFilePath --parameter-file $parametersFilePath 
```

### <a name="use-a-pointer-to-a-secret-uploaded-into-a-key-vault"></a>使用指標以指向上傳至金鑰保存庫的祕密

若要使用現有的金鑰保存庫，其必須[啟用為可供部署使用](../key-vault/key-vault-manage-with-cli2.md#bkmk_KVperCLI)，才能讓計算資源提供者從中取得憑證，然後將它安裝在叢集節點上。

使用 PowerShell 部署叢集：

```powershell
Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -EnabledForDeployment

$parameterFilePath="c:\mytemplates\mytemplate.json"
$templateFilePath="c:\mytemplates\mytemplateparm.json"
$secretID="https://test1.vault.azure.net:443/secrets/testcertificate4/55ec7c4dc61a462bbc645ffc9b4b225f"

New-AzureRmServiceFabricCluster -ResourceGroupName $resourceGroupName -SecretIdentifier $secretId -TemplateFile $templateFilePath -ParameterFile $parameterFilePath 
```

使用 Azure CLI 部署叢集：

```azurecli
declare $resourceGroupName = "testRG"
declare $parameterFilePath="c:\mytemplates\mytemplate.json"
declare $templateFilePath="c:\mytemplates\mytemplateparm.json"
declare $secertId="https://test1.vault.azure.net:443/secrets/testcertificate4/55ec7c4dc61a462bbc645ffc9b4b225f"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --secret-identifier az $secretID  \
    --template-file $templateFilePath --parameter-file $parametersFilePath 
```

## <a name="next-steps"></a>後續步驟
此時，Azure 中已有安全的叢集在執行。 接下來，請[連線到您的叢集](service-fabric-connect-to-secure-cluster.md)並了解如何[管理應用程式密碼](service-fabric-application-secret-management.md)。

<!-- Links -->
[azure-powershell]:https://docs.microsoft.com/powershell/azure/install-azurerm-ps
[azure-CLI]:https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[customize-your-cluster-template]: service-fabric-cluster-creation-via-arm.md#create-a-service-fabric-cluster-resource-manager-template
