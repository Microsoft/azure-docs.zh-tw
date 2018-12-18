---
title: Azure 中的 HPC Pack 2016 叢集 | Microsoft Docs
description: 了解如何在 Azure 中部署 HPC Pack 2016 叢集
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 3dde6a68-e4a6-4054-8b67-d6a90fdc5e3f
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-multiple
ms.workload: big-compute
ms.date: 03/09/2018
ms.author: danlep
ms.openlocfilehash: 91f067de33d1ff4bc272773e3db49de47fac2feb
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32190007"
---
# <a name="deploy-an-hpc-pack-2016-cluster-in-azure"></a>在 Azure 中部署 HPC Pack 2016 叢集

遵循本文中的步驟，在 Azure 虛擬機器中部署 [Microsoft HPC Pack 2016 更新 1](https://technet.microsoft.com/library/cc514029) 叢集。 HPC Pack 是建置在 Microsoft Azure 和 Windows Server 技術上的 Microsoft 免費 HPC 解決方案，並支援各種 HPC 工作負載。

使用其中一種 [Azure Resource Manager 範本](https://github.com/MsHpcPack/HPCPack2016) 部署 HPC Pack 2016 叢集。 您可以選擇多種叢集拓撲，當中包含不同數目的叢集前端節點和計算節點類型。

## <a name="prerequisites"></a>先決條件

### <a name="pfx-certificate"></a>PFX 憑證

Microsoft HPC Pack 2016 叢集需要個人資訊交換 (PFX) 憑證來保護 HPC 節點之間的通訊安全。 憑證必須符合下列要求：

* 它必須有能夠進行金鑰交換的私密金鑰
* 金鑰使用方法包含數位簽章和金鑰加密
* 增強金鑰使用方法包含用戶端驗證和伺服器驗證

如果您還沒有符合這些需求的憑證，可以從憑證授權單位要求憑證。 或者，根據您用來執行命令的作業系統，使用下列命令產生自我簽署的憑證。 然後，以私密金鑰將憑證匯出為受密碼保護的 PFX 檔案。

* **Windows 10 或 Windows Server 2016**，執行內建 **New-SelfSignedCertificate** PowerShell cmdlet，如下所示︰

  ```PowerShell
  New-SelfSignedCertificate -Subject "CN=HPC Pack 2016 Communication" -KeySpec KeyExchange -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1,1.3.6.1.5.5.7.3.2") -CertStoreLocation cert:\CurrentUser\My -KeyExportPolicy Exportable -NotAfter (Get-Date).AddYears(5)
  ```
* **若為 Windows 10 或 Windows Server 2016 以前的作業系統**，則必須從 Microsoft 指令碼中心下載[自我簽署憑證產生器](https://gallery.technet.microsoft.com/scriptcenter/Self-signed-certificate-5920a7c6/)。 將其內容解壓縮，並在 PowerShell 提示字元執行下列命令︰

    ```PowerShell 
    Import-Module -Name c:\ExtractedModule\New-SelfSignedCertificateEx.ps1
  
    New-SelfSignedCertificateEx -Subject "CN=HPC Pack 2016 Communication" -KeySpec Exchange -KeyUsage "DigitalSignature,KeyEncipherment" -EnhancedKeyUsage "Server Authentication","Client Authentication" -StoreLocation CurrentUser -Exportable -NotAfter (Get-Date).AddYears(5)
    ```

在「目前使用者」存放區中建立憑證之後，使用憑證嵌入式管理單元以私密金鑰將憑證匯出為受密碼保護的 PFX 檔案。 您也可以使用 [Export-Pfxcertificate](/powershell/module/pkiclient/export-pfxcertificate?view=win10-ps) PowerShell Cmdlet 來匯出憑證。

### <a name="upload-certificate-to-an-azure-key-vault"></a>將憑證上傳至 Azure 金鑰保存庫

在部署 HPC 叢集之前，將 PFX 憑證上傳至 [Azure 金鑰保存庫](../../key-vault/index.yml)作為秘密，並記錄下列資訊以在部署期間使用︰**保存庫名稱**、**保存庫資源群組**、**憑證 URL** 及**憑證指紋**。

PowerShell 指令碼範例會用來上傳憑證、建立金鑰保存庫及產生後續的必要資訊。 如需將憑證上傳至 Azure 金鑰保存庫的詳細資訊，請參閱[開始使用 Azure 金鑰保存庫](../../key-vault/key-vault-get-started.md)。

```powershell
#Give the following values
$VaultName = "mytestvault"
$SecretName = "hpcpfxcert"
$VaultRG = "myresourcegroup"
$location = "westus"
$PfxFile = "c:\Temp\mytest.pfx"
$Password = "yourpfxkeyprotectionpassword"
#Validate the pfx file
try {
    $pfxCert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList $PfxFile, $Password
}
catch [System.Management.Automation.MethodInvocationException]
{
    throw $_.Exception.InnerException
}
$thumbprint = $pfxCert.Thumbprint
$pfxCert.Dispose()
# Create and encode the JSON object
$pfxContentBytes = Get-Content $PfxFile -Encoding Byte
$pfxContentEncoded = [System.Convert]::ToBase64String($pfxContentBytes)
$jsonObject = @"
{
"data": "$pfxContentEncoded",
"dataType": "pfx",
"password": "$Password"
}
"@
$jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
$jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)
#Create an Azure key vault and upload the certificate as a secret
$secret = ConvertTo-SecureString -String $jsonEncoded -AsPlainText -Force
$rg = Get-AzureRmResourceGroup -Name $VaultRG -Location $location -ErrorAction SilentlyContinue
if($null -eq $rg)
{
    $rg = New-AzureRmResourceGroup -Name $VaultRG -Location $location
}
$hpcKeyVault = New-AzureRmKeyVault -VaultName $VaultName -ResourceGroupName $VaultRG -Location $location -EnabledForDeployment -EnabledForTemplateDeployment
$hpcSecret = Set-AzureKeyVaultSecret -VaultName $VaultName -Name $SecretName -SecretValue $secret
"The following Information will be used in the deployment template"
"Vault Name             :   $VaultName"
"Vault Resource Group   :   $VaultRG"
"Certificate URL        :   $($hpcSecret.Id)"
"Certificate Thumbprint :   $thumbprint"

```


## <a name="supported-topologies"></a>支援的拓撲

選取其中一種 [Azure Resource Manager 範本](https://github.com/MsHpcPack/HPCPack2016) 部署 HPC Pack 2016 叢集。 以下是三個叢集拓撲範例的高階架構。 高可用性拓撲包含多個叢集前端節點。

1. 包含 Active Directory 網域的高可用性叢集

    ![AD 網域中的 HA 叢集](./media/hpcpack-2016-cluster/haad.png)


2. 不包含 Active Directory 網域的高可用性叢集

    ![不包含 AD 網域的 HA 叢集](./media/hpcpack-2016-cluster/hanoad.png)

3. 包含單一前端節點的叢集

   ![包含單一前端節點的叢集](./media/hpcpack-2016-cluster/singlehn.png)


## <a name="deploy-a-cluster"></a>部署叢集

若要建立叢集，請選擇範本並按一下 [部署至 Azure]。 在 [Azure 入口網站](https://portal.azure.com)中，如下列步驟中所述指定範本的參數。 每個範本會建立 HPC 叢集基礎結構所需的所有 Azure 資源。 資源包括 Azure 虛擬網路、公用 IP 位址、負載平衡器 (僅適用於高可用性叢集)、網路介面、可用性設定組、儲存體帳戶和虛擬機器。

### <a name="step-1-select-the-subscription-location-and-resource-group"></a>步驟 1︰選取訂用帳戶、位置和資源群組

**訂用帳戶**和**位置**必須與您上傳 PFX 憑證 (請參閱必要條件) 時所指定的相同。 建議您為部署建立不同的**資源群組**。

### <a name="step-2-specify-the-parameter-settings"></a>步驟 2︰指定參數設定

輸入或修改範本參數的值。 按一下說明資訊的每個參數旁邊的圖示。 另請參閱[可用 VM 大小](sizes.md)的指引。

指定您在必要條件中記錄之下列參數的值︰**保存庫名稱**、**保存庫資源群組**、**憑證 URL** 和 **憑證指紋**。

### <a name="step-3-review-terms-and-create"></a>步驟 3. 檢閱條款並建立
檢閱與範本相關聯的條款和條件。 如果您同意，按一下 [購買] 以啟動部署。

根據叢集拓撲而定，部署需要 30 分鐘或更長時間才能完成。

## <a name="connect-to-the-cluster"></a>連接到叢集
1. 部署 HPC Pack 叢集之後，請移至 [Azure 入口網站](https://portal.azure.com)。 按一下 [資源群組]，並尋找已在當中部署叢集的資源群組。 您可以找到前端節點虛擬機器。

    ![入口網站中的叢集前端節點](./media/hpcpack-2016-cluster/clusterhns.png)

2. 按一個前端節點 (在高可用性叢集中，按一下任一個前端節點)。 在 [概觀] 中，您可以找到叢集的公用 IP 位址或完整 DNS 名稱。

    ![叢集連線設定](./media/hpcpack-2016-cluster/clusterconnect.png)

3. 按一下 [連接]，使用遠端桌面搭配您指定的系統管理員使用者名稱登入任何前端節點。 如果您部署的叢集在 Active Directory 網域中，使用者名稱是表單 \<privateDomainName>\\\<adminUsername> (例如，hpc.local\hpcadmin)。

## <a name="next-steps"></a>後續步驟
* 將工作提交至叢集。 請參閱 [提交工作至 Azure 中 HPC Pack 叢集的 HPC](hpcpack-cluster-submit-jobs.md) 和 [使用 Azure Active Directory 管理 Azure 中的 HPC Pack 2016 叢集](hpcpack-cluster-active-directory.md)。

