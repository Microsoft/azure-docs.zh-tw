---
title: 準備 Azure Stack 開發套件 (ASDK) 主機電腦 | Microsoft Docs
description: 說明如何準備 Azure Stack 開發套件 (ASDK) 主機電腦以進行 ASDK 安裝。
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
ms.date: 03/22/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: 5de25f574cb876701ffce74f1dca8c4bb9764157
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30165665"
---
# <a name="prepare-the-asdk-host-computer"></a>準備 ASDK 主機電腦
必須先讓 ASDK 環境做好安裝準備，才可以在主機電腦上安裝 ASDK。 當開發套件主機電腦做好準備時，它會從 CloudBuilder.vhdx 虛擬機器硬碟開機，以開始進行 ASDK 部署。

## <a name="prepare-the-development-kit-host-computer"></a>準備開發套件主機電腦
必須先備妥 ASDK 主機電腦環境，才可以在主機電腦上安裝 ASDK。
1. 以本機系統管理員的身分來登入開發套件主機電腦。
2. 確保 CloudBuilder.vhdx 檔案已移至 C:\ 磁碟機的根目錄 (C:\CloudBuilder.vhdx)。
3. 執行下列指令碼，將開發套件安裝程式檔案 (asdk-installer.ps1) 從 [Azure Stack GitHub 工具存放庫](https://github.com/Azure/AzureStack-Tools)下載至開發套件主機電腦上的 **C:\AzureStack_Installer** 資料夾：

  ```powershell
  # Variables
  $Uri = 'https://raw.githubusercontent.com/Azure/AzureStack-Tools/master/Deployment/asdk-installer.ps1'
  $LocalPath = 'C:\AzureStack_Installer'
  # Create folder
  New-Item $LocalPath -Type directory
  # Enforce usage of TLSv1.2 to download from GitHub
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
  # Download file
  Invoke-WebRequest $uri -OutFile ($LocalPath + '\' + 'asdk-installer.ps1')
  ```

4. 從已提升權限的 PowerShell 主控台，啟動 **C:\AzureStack_Installer\asdk-installer.ps1** 指令碼，然後按一下 [準備環境]。

    ![](media/asdk-prepare-host/1.PNG) 

5. 在安裝程式的 [選取 Cloudbuilder vhdx] 頁面上，瀏覽至您在[先前步驟中](asdk-download.md)下載並解壓縮的 **cloudbuilder.vhdx** 檔案並加以選取。 如果您需要在開發套件主機電腦上新增其他驅動程式，您也可以在此頁面上選擇性地啟用 [新增驅動程式] 核取方塊。 按 [下一步] 。  

    ![](media/asdk-prepare-host/2.PNG)

6. 在 [選擇性設定] 頁面上，提供開發套件主機電腦的本機系統管理員帳戶資訊，然後按 [下一步]。 您也可以為下列選用設定提供值：
  - **電腦名稱**：此選項會設定開發套件主機的名稱。 名稱必須符合 FQDN 需求，且長度必須等於或少於 15 個字元。 預設值是由 Windows 產生的隨機電腦名稱。
  - **時區**：設定開發套件主機的時區。 預設值是 (UTC-8:00) 太平洋時間 (美國和加拿大)。
  - **靜態 IP 組態**：將您的部署設定為使用靜態 IP 位址。 否則，當安裝程式重新開機進入 cloudbuilder.vhx 時，會使用 DHCP 來設定網路介面。

    ![](media/asdk-prepare-host/3.PNG)

  > [!IMPORTANT]
  > 如果您未在此步驟提供本機系統管理員認證，則在設定開發套件期間重新啟動電腦之後，您需要主機的直接或 KVM 存取權。

7. 如果您在上一個步驟中選擇靜態 IP 組態，您現在必須：
    - 選取網路介面卡。 請先確定您可以連線到介面卡，然後再按一下 [下一步]。
    - 請確定 [IP 位址]、[閘道] 和 [DNS] 值正確無誤，然後按一下 [下一步]。
13. 按一下 [下一步]，以開始準備程序。
14. 當準備指出 [已完成] 時，按一下 [下一步]。
15. 按一下 [立即重新開機]，以將開發套件主機電腦開機進入 cloudbuilder.vhdx 並[繼續部署程序](asdk-install.md)。

    ![](media/asdk-prepare-host/4.PNG)


## <a name="next-steps"></a>後續步驟
[安裝 ASDK](asdk-install.md)