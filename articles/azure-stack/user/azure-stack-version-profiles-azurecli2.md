---
title: 使用 CLI 連線至 Azure Stack | Microsoft 文件
description: 了解如何使用跨平台命令列介面 (CLI) 在 Azure Stack 上管理及部署資源
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/08/2018
ms.author: sethm
ms.reviewer: sijuman
ms.openlocfilehash: 59b637e6887a645430d902cd846cacda13b14cfe
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46972805"
---
# <a name="use-api-version-profiles-with-azure-cli-in-azure-stack"></a>在 Azure Stack 中搭配 Azure CLI 使用 API 版本設定檔

您可以遵循本文中的步驟從 Linux、Mac 和 Windows 用戶端平台設定 Azure 命令列介面 (CLI)，來管理 Azure Stack 開發套件資源。

## <a name="install-cli"></a>安裝 CLI

登入您的開發工作站並安裝 CLI。 Azure Stack 需要有 Azure CLI 2.0 版或更新版本。 您可以使用[安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) 一文中所述的步驟來安裝它。 若要確認安裝是否成功，請開啟終端機或命令提示字元視窗，並執行下列命令：

```azurecli
az --version
```

您應該會看到 Azure CLI 的號碼和您電腦上安裝的其他相依程式庫。

## <a name="trust-the-azure-stack-ca-root-certificate"></a>信任 Azure Stack CA 根憑證

1. 從 [Azure Stack 操作員](..\azure-stack-cli-admin.md#export-the-azure-stack-ca-root-certificate)取得 Azure Stack CA 根憑證，並信任該憑證。 若要信任 Azure Stack CA 根憑證，請將它附加到現有的 Python 憑證。

1. 尋找您機器上的憑證位置。 此位置可能會根據您安裝 Python 的位置不同而有所差異。 您必須安裝 [pip](https://pip.pypa.io) 和 [certifi](https://pypi.org/project/certifi/) 模組。 您可以從 Bash 提示中使用下列 Python 命令：

  ```bash  
    python -c "import certifi; print(certifi.where())"
  ```

  記下憑證位置。 例如： `~/lib/python3.5/site-packages/certifi/cacert.pem`。 您的特定路徑取決於安裝的 OS 和 Python 版本。

### <a name="set-the-path-for-a-development-machine-inside-the-cloud"></a>為雲端內的開發機器設定路徑

如果您是從在 Azure Stack 環境內建立的 Linux 機器執行 CLI，請以您的憑證路徑執行下列 Bash 命令。

```bash
sudo cat /var/lib/waagent/Certificates.pem >> ~/<yourpath>/cacert.pem
```

### <a name="set-the-path-for-a-development-machine-outside-the-cloud"></a>為雲端外的開發機器設定路徑

如果您是從 Azure Stack 環境**以外**的機器執行 CLI：  

1. 您必須設定 [VPN 連線來連線到 Azure Stack](azure-stack-connect-azure-stack.md)。

1. 複製從 Azure Stack 操作員那取得的 PEM 憑證，並記下檔案位置 (PATH_TO_PEM_FILE)。

1. 根據您開發工作站的 OS，執行下列命令。

#### <a name="linux"></a>Linux

```bash
sudo cat PATH_TO_PEM_FILE >> ~/<yourpath>/cacert.pem
```

#### <a name="macos"></a>macOS

```bash
sudo cat PATH_TO_PEM_FILE >> ~/<yourpath>/cacert.pem
```

#### <a name="windows"></a>Windows

```powershell
$pemFile = "<Fully qualified path to the PEM certificate Ex: C:\Users\user1\Downloads\root.pem>"

$root = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
$root.Import($pemFile)

Write-Host "Extracting needed information from the cert file"
$md5Hash    = (Get-FileHash -Path $pemFile -Algorithm MD5).Hash.ToLower()
$sha1Hash   = (Get-FileHash -Path $pemFile -Algorithm SHA1).Hash.ToLower()
$sha256Hash = (Get-FileHash -Path $pemFile -Algorithm SHA256).Hash.ToLower()

$issuerEntry  = [string]::Format("# Issuer: {0}", $root.Issuer)
$subjectEntry = [string]::Format("# Subject: {0}", $root.Subject)
$labelEntry   = [string]::Format("# Label: {0}", $root.Subject.Split('=')[-1])
$serialEntry  = [string]::Format("# Serial: {0}", $root.GetSerialNumberString().ToLower())
$md5Entry     = [string]::Format("# MD5 Fingerprint: {0}", $md5Hash)
$sha1Entry    = [string]::Format("# SHA1 Fingerprint: {0}", $sha1Hash)
$sha256Entry  = [string]::Format("# SHA256 Fingerprint: {0}", $sha256Hash)
$certText = (Get-Content -Path $pemFile -Raw).ToString().Replace("`r`n","`n")

$rootCertEntry = "`n" + $issuerEntry + "`n" + $subjectEntry + "`n" + $labelEntry + "`n" + `
$serialEntry + "`n" + $md5Entry + "`n" + $sha1Entry + "`n" + $sha256Entry + "`n" + $certText

Write-Host "Adding the certificate content to Python Cert store"
Add-Content "${env:ProgramFiles(x86)}\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem" $rootCertEntry

Write-Host "Python Cert store was updated for allowing the azure stack CA root certificate"
```

## <a name="get-the-virtual-machine-aliases-endpoint"></a>取得虛擬機器別名端點

使用者在使用 CLI 建立虛擬機器之前，必須連絡 Azure Stack 操作員，並取得虛擬機器別名端點 URI。 例如，Azure 會使用下列 URI： https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json。 雲端系統管理員應該使用 Azure Stack 市集中可用的映像，為 Azure Stack 設定類似的端點。 使用者必須將端點 URI 的 `endpoint-vm-image-alias-doc` 參數傳遞至 `az cloud register` 命令，如下一節中所示。 
   

## <a name="connect-to-azure-stack"></a>連線至 Azure Stack

使用下列步驟連線至 Azure Stack：

1. 執行 `az cloud register` 命令來註冊 Azure Stack 環境。
   
   a. 若要註冊「雲端系統管理」環境，請使用：

      ```azurecli
      az cloud register \ 
        -n AzureStackAdmin \ 
        --endpoint-resource-manager "https://adminmanagement.local.azurestack.external" \ 
        --suffix-storage-endpoint "local.azurestack.external" \ 
        --suffix-keyvault-dns ".adminvault.local.azurestack.external" \ 
        --endpoint-vm-image-alias-doc <URI of the document which contains virtual machine image aliases>
      ```

   b. 若要註冊「使用者」環境，請使用：

      ```azurecli
      az cloud register \ 
        -n AzureStackUser \ 
        --endpoint-resource-manager "https://management.local.azurestack.external" \ 
        --suffix-storage-endpoint "local.azurestack.external" \ 
        --suffix-keyvault-dns ".vault.local.azurestack.external" \ 
        --endpoint-vm-image-alias-doc <URI of the document which contains virtual machine image aliases>
      ```

1. 使用下列命令來設定作用中環境。

   a. 針對「雲端系統管理」環境，請使用：

      ```azurecli
      az cloud set \
        -n AzureStackAdmin
      ```

   b. 針對「使用者」環境，請使用：

      ```azurecli
      az cloud set \
        -n AzureStackUser
      ```

1. 將您的環境組態更新成使用 Azure Stack 特定的 API 版本設定檔。 若要更新組態，請執行下列命令：

   ```azurecli
   az cloud update \
     --profile 2018-03-01-hybrid
   ```

    >[!NOTE]  
    >如果您執行正在 Azure Stack 1808 組建之前的版本，則將必須使用 API 版本設定檔 **2017-03-09-profile** 而不是 API 版本設定檔 **2018-03-01-hybrid**。

1. 使用 `az login` 命令來登入 Azure Stack 環境。 您可以以使用者身分或以[服務主體](https://docs.microsoft.com/azure/active-directory/develop/active-directory-application-objects)形式登入 Azure Stack 環境。 

   * 以「使用者」身分登入：您可以直接在 `az login` 命令內指定使用者名稱和密碼，或使用瀏覽器進行驗證。 如果您的帳戶已啟用多重要素驗證，則必須採用後者方式。

      ```azurecli
      az login \
        -u <Active directory global administrator or user account. For example: username@<aadtenant>.onmicrosoft.com> \
        --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com>
      ```

      > [!NOTE]
      > 如果您的使用者帳戶已啟用多重要素驗證，則可以使用 `az login command` 命令，而不需提供 `-u` 參數。 執行此命令可提供您一個 URL 以及必須用來進行驗證的代碼。
   
   * 以*服務主體*形式登入：在登入之前，請[透過 Azure 入口網站](azure-stack-create-service-principals.md)或 CLI 建立服務主體，並為它指派角色。 現在，請使用下列命令登入：

      ```azurecli
      az login \
        --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com> \
        --service-principal \
        -u <Application Id of the Service Principal> \
        -p <Key generated for the Service Principal>
      ```

## <a name="test-the-connectivity"></a>測試連線

一切都已準備就緒，接下來我們要使用 CLI 在 Azure Stack 中建立資源。 例如，您可以建立應用程式的資源群組，並新增虛擬機器。 若要建立名為 "MyResourceGroup" 的資源群組，請使用下列命令：

```azurecli
az group create \
  -n MyResourceGroup -l local
```

如果資源群組成功建立，先前的命令會輸出新建立資源的下列內容：

![資源群組建立輸出](media/azure-stack-connect-cli/image1.png)

## <a name="known-issues"></a>已知問題
使用 Azure Stack 中的 CLI 時，有一些已知問題必須留意：

 - CLI 互動模式 (例如 `az interactive` 命令) 在 Azure Stack 中尚未支援。
 - 若要取得 Azure Stack 中的可用虛擬機器映像清單，請使用 `az vm images list --all` 命令，而非 `az vm image list` 命令。 指定 `--all` 選項可確保回應只傳回您 Azure Stack 環境中的可用映像。
 - Azure 中可用的虛擬機器映像別名可能不適用於 Azure Stack。 使用虛擬機器映像時，您必須使用整個 URN 參數 (Canonical:UbuntuServer:14.04.3-LTS:1.0.0)，而非映像別名。 此 URN 必須符合從 `az vm images list` 命令衍生的映像規格。

## <a name="next-steps"></a>後續步驟

[使用 Azure CLI 部署範本](azure-stack-deploy-template-command-line.md)

[為 Azure Stack 使用者 (操作員) 啟用 Azure CLI](..\azure-stack-cli-admin.md)

[管理使用者權限](azure-stack-manage-permissions.md)
