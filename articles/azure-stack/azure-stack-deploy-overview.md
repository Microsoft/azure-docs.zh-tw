---
title: 評估 Azure Stack 開發套件 | Microsoft Docs
description: 了解如何部署評估用的 Azure Stack 開發套件。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 10/15/2018
ms.author: jeffgilb
ms.custom: mvc
ms.openlocfilehash: bf07fe56c65e53f5485b9927e0d704f80842cf3a
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49338690"
---
# <a name="quickstart-evaluate-the-azure-stack-development-kit"></a>快速入門：評估 Azure Stack 開發套件

[Azure Stack 開發套件 (ASDK)](.\asdk\asdk-what-is.md) 是一個測試和部署環境，可供您部署以評估及示範 Azure Stack 的功能和服務。 若要開始使用 ASDK，您需要準備主機電腦硬體，然後再執行一些指令碼 (安裝須花費數小時)。 之後，您就可以登入系統管理員或使用者入口網站，開始使用 Azure Stack。

## <a name="prerequisites"></a>必要條件

### <a name="asdk-host-computer-requirements"></a>ASDK 主機電腦需求

安裝 ASDK前，您需要準備將裝載開發套件的電腦。 開發套件主機電腦必須符合**[檢閱 ASDK 部署規劃考量](.\asdk\asdk-deploy-considerations.md)** 中所述的硬體、軟體及網路需求。

> [!TIP]
> 安裝作業系統之後，您可以在開發套件主機電腦上使用 [Azure Stack 部署需求檢查工具](https://gallery.technet.microsoft.com/Deployment-Checker-for-50e0f51b)，來確認您的硬體符合所有需求。

### <a name="account-requirements"></a>帳戶需求

您也需要選擇使用 Azure Active Directory (Azure AD) 或 Active Directory Federation Services (AD FS) 作為部署的識別解決方案。 檢閱**[部署規劃考量](.\asdk\asdk-deploy-considerations.md#account-requirements)** 中的帳戶需求

## <a name="download-and-extract-the-deployment-package"></a>下載部署套件並解壓縮

在準備好開發套件主機電腦後，請下載 ASDK 部署套件並予以解壓縮。 開發套件包括 Cloudbuilder.vhdx 檔案，這是具有可開機作業系統和 Azure Stack 安裝檔的虛擬硬碟 (VHD)。

您可以將部署套件下載到開發套件主機或另一部電腦。 解壓縮後的部署檔案會佔用 60 GB 的可用磁碟空間，因此使用另一部電腦可協助降低開發套件主機上的儲存體需求。

**[下載 Azure Stack 開發套件 (ASDK) 並解壓縮](.\asdk\asdk-download.md)**

## <a name="prepare-the-host-computer"></a>準備主機電腦

必須先備妥主機環境，並將系統設定為從開發套件 VHD 開機，才可以安裝 ASDK。 當您重新啟動主機時，它會從 CloudBuilder.vhdx 開機，然後您就可以開始部署 ASDK。

**[準備 ASDK 主機電腦](.\asdk\asdk-prepare-host.md)**

## <a name="install-the-asdk-on-the-host-computer"></a>在主機電腦上安裝 ASDK

主機電腦從 VHD 開機之後，您就可以在 Cloudbuilder 虛擬環境中部署開發套件。 您可以使用藉由執行 asdk-installer.ps1 PowerShell 指令碼所提供的圖形化使用者介面 (GUI) 來部署 ASDK，也可以從 [PowerShell 命令列](.\asdk\asdk-deploy-powershell.md)進行部署。

> [!NOTE]
> 在主機從 Cloudbuilder.vhdx 映像開機後，您可以選擇在安裝 ASDK 前，先設定 [Azure Stack 遙測設定](.\asdk\asdk-telemetry.md#set-telemetry-level-in-the-windows-registry)。

**[安裝 Azure Stack 開發套件 (ASDK)](.\asdk\asdk-install.md)**

## <a name="perform-post-deployment-configurations"></a>執行部署後設定

安裝 ASDK 後，建議您進行一些安裝後的檢查，並變更一些設定。

**工具**

安裝 Azure Stack PowerShell 和 GitHub 工具，並使用 test-AzureStack Cmdlet 來檢查安裝是否成功。

**身分識別解決方案**

針對使用 Azure AD 的部署，您必須同時啟用 Azure Stack 系統管理員和租用戶入口網站。 此啟用同意將所有目錄使用者的正確權限 (列在同意頁面上) 提供給 Azure Stack 入口網站和 Azure Resource Manager。

**密碼到期**

您應該重設密碼到期原則，以確保開發套件主機的密碼不會在評估期間結束前到期。

> [!NOTE]
> 您也可以選擇在安裝 ASDK 後，再設定 [Azure Stack 遙測設定](.\asdk\asdk-telemetry.md#enable-or-disable-telemetry-after-deployment)。

**[部署 ASDK 後的工作](.\asdk\asdk-post-deploy.md)**

## <a name="register-with-azure"></a>向 Azure 註冊

您必須向 Azure 註冊 Azure Stack，如此您才能夠[將 Azure 市集項目下載到 Azure Stack](.\asdk\asdk-marketplace-item.md)。

**[向 Azure 註冊 Azure Stack](.\asdk\asdk-register.md)**

## <a name="next-steps"></a>後續步驟

恭喜！ 在完成本快速入門中的步驟後，您所擁有的 ASDK 環境已具備[管理員](https://adminportal.local.azurestack.external)入口網站和[使用者](https://portal.local.azurestack.external)入口網站。
