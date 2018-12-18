---
title: 重新部署 Azure Stack 開發套件 (ASDK) | Microsoft Docs
description: 在本文中，您會了解如何安裝 ASDK。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.custom: ''
ms.date: 08/01/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: d166916ca54f3b8c26a418ff83093e53dcdbe515
ms.sourcegitcommit: 96f498de91984321614f09d796ca88887c4bd2fb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39413727"
---
# <a name="redeploy-the-asdk"></a>重新部署 ASDK
在本文中，您會了解如何在非生產環境中重新部署 Azure Stack 開發套件 (ASDK)。 因為不支援升級 ASDK，您需要將其完全重新部署到較新的版本。 若您想要從頭開始，也可以隨時重新部署 ASDK。

> [!IMPORTANT]
> 不支援將 ASDK 升級到新版本。 每當您想要評估新版本的 Azure Stack，都必須在開發套件主機電腦上重新部署 ASDK。

## <a name="remove-azure-registration"></a>移除 Azure 註冊 
若您先前已向 Azure 註冊您的 ASDK 安裝，請先移除註冊資源，再重新部署 ASDK。 重新部署 ASDK 時，請重新註冊 ASDK 以啟用市集中的項目可用性。 若您先前未使用您的 Azure 訂用帳戶註冊過 ASDK，可略過本節。

若要移除註冊資源，請使用 **Remove-AzsRegistration** Cmdlet 來取消註冊 Azure Stack。 然後，使用 **Remove-AzureRMRsourceGroup** Cmdlet，從您的 Azure 訂用帳戶中刪除 Azure Stack 資源群組：

1. 在可以存取特殊權限端點的電腦上，以系統管理員身分開啟 PowerShell 主控台。 對於 ASDK 而言，就是指開發套件主機電腦。

2. 請執行下列 PowerShell 命令來取消註冊 ASDK 安裝，並從您的 Azure 訂用帳戶中刪除 **azurestack** 資源群組：

  ```Powershell    
  #Import the registration module that was downloaded with the GitHub tools
  Import-Module C:\AzureStack-Tools-master\Registration\RegisterWithAzure.psm1

  # Provide Azure subscription admin credentials
  Add-AzureRmAccount

  # Provide ASDK admin credentials
  $CloudAdminCred = Get-Credential -UserName AZURESTACK\CloudAdmin -Message "Enter the cloud domain credentials to access the privileged endpoint"

  # Unregister Azure Stack
  Remove-AzsRegistration `
      -PrivilegedEndpointCredential $CloudAdminCred `
      -PrivilegedEndpoint AzS-ERCS01

  # Remove the Azure Stack resource group
  Remove-AzureRmResourceGroup -Name azurestack -Force
  ```

3. 指令碼執行時，系統會提示您登入您的 Azure 訂用帳戶和本機 ASDK 安裝。
4. 指令碼完成時，您應該會看到類似下列範例的訊息：

    ` De-Activating Azure Stack (this may take up to 10 minutes to complete).` ` Your environment is now unable to syndicate items and is no longer reporting usage data.` ` Remove registration resource from Azure...` ` "Deleting the resource..." on target "/subscriptions/<subscription information>"` ` ********** End Log: Remove-AzsRegistration ********* `



現在系統應該已成功從您的 Azure 訂用帳戶中取消註冊了 Azure Stack。 此外，您向 Azure 註冊 ASDK 時所建立的 azurestack 資源群組，應該也已經刪除。

## <a name="deploy-the-asdk"></a>部署 ASDK
若要重新部署 Azure Stack，您必須從頭開始進行，如下所述。 您是否使用了 Azure Stack 安裝程式 (asdk-installer.ps1) 指令碼來安裝 ASDK，會讓接下來的步驟有所不同。

### <a name="redeploy-the-asdk-using-the-installer-script"></a>使用安裝程式指令碼重新部署 ASDK
1. 在 ASDK 電腦上，開啟提升權限的 PowerShell 主控台，並瀏覽到位於非系統磁碟機上 **AzureStack_Installer** 目錄中的 asdk installer.ps1 指令碼。 執行指令碼，然後按一下 [重新開機]。

   ![執行 asdk installer.ps1 指令碼](media/asdk-redeploy/1.png)

2. 選取基礎作業系統 (非 Azure Stack) 並按一下 [下一步]。

   ![重新開機進入主機作業系統](media/asdk-redeploy/2.png)

3. 開發套件主機重新開機進入基礎作業系統後，以本機系統管理員身分登入。 找出並刪除先前部署過程中所使用的 **C:\CloudBuilder.vhdx** 檔案。 

4. 重複第一次[部署 ASDK](asdk-install.md) 時的相同步驟。

### <a name="redeploy-the-asdk-without-using-the-installer"></a>不使用安裝程式重新部署 ASDK
若您未使用 asdk-installer.ps1 指令碼來安裝 ASDK，就必須先手動重新設定開發套件主機電腦，再重新部署 ASDK。

1. 在 ASDK 電腦上執行 **msconfig.exe**，啟動 [系統設定] 公用程式。 在 [開機] 索引標籤上，選取主機電腦的作業系統 (而不是 Azure Stack)、按一下 [設為預設值]，然後按一下 [確定]。 出現提示時，請按一下 [重新啟動]。

      ![設定開機組態](media/asdk-redeploy/4.png)

2. 開發套件主機重新開機進入基礎作業系統後，以開發套件主機電腦的本機系統管理員身分登入。 找出並刪除先前部署過程中所使用的 **C:\CloudBuilder.vhdx** 檔案。 

3. 重複第一次[使用 PowerShell 部署 ASDK](asdk-deploy-powershell.md) 時的相同步驟。


## <a name="next-steps"></a>後續步驟
[ASDK 安裝後設定工作](asdk-post-deploy.md)




