---
title: 安裝 Visual Studio 並連線至 Azure Stack | Microsoft Docs
description: 了解安裝 Visual Studio 並連線至 Azure Stack 的必要逐步執行
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.custom: vs-azure
ms.workload: azure-vs
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2018
ms.author: sethm
ms.reviewer: unknown
ms.openlocfilehash: 2afbea68c017805e9bd7db43b03face0705608b7
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42358742"
---
# <a name="install-visual-studio-and-connect-to-azure-stack"></a>安裝 Visual Studio 並連線至 Azure Stack

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以使用 Visual Studio 在 Azure Stack 中撰寫和部署 Azure Resource Manager [範本](azure-stack-arm-templates.md)。 本文中的步驟會逐步引導您在 [Azure Stack](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-remote-desktop) 上安裝 Visual Studio，或是在外部電腦上安裝 (如果您計劃透過 [VPN](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn) 連線至 Azure Stack)。

## <a name="install-visual-studio"></a>安裝 Visual Studio

1. 下載並執行 [Web Platform Installer](https://www.microsoft.com/web/downloads/platform.aspx)。  

2. 開啟 **Microsoft Web Platform Installer**。

3. 搜尋 **Visual Studio Community 2015 與 Microsoft Azure SDK - 2.9.6**。 按一下 [新增] 和 [檢閱]。

4. 解除安裝作為 Azure SDK 安裝一部分的 **Microsoft Azure PowerShell**。

    ![WebPI 安裝逐步執行的螢幕擷取畫面](./media/azure-stack-install-visual-studio/image1.png) 

5. [安裝 Azure Stack 適用的 PowerShell](azure-stack-powershell-install.md)

6. 在安裝完成之後，請重新啟動作業系統。

## <a name="connect-to-azure-stack-with-azure-ad"></a>使用 Azure AD 來連線到 Azure Stack

1. 啟動 Visual Studio。

2. 從 [檢視] 功能表選取 [Cloud Explorer]。

3. 在新的窗格中，選取 [新增帳戶] 並以您的 Azure Active Directory (Azure AD) 認證登入。  

    ![登入 Cloud Explorer 後並連接到 Azure Stack 的螢幕擷取畫面](./media/azure-stack-install-visual-studio/image2.png)

登入之後，您可以[部署範本](azure-stack-deploy-template-visual-studio.md)，或瀏覽可用的資源型別和資源群組來建立您自己的範本。  

## <a name="connect-to-azure-stack-with-ad-fs"></a>使用 AD FS 來連線到 Azure Stack

1. 啟動 Visual Studio。

2. 從 [工具] 選取 [選項]。

3. 展開 [瀏覽窗格] 中的 [環境]，然後選取 [帳戶]。

4. 選取 [新增]，然後輸入使用者 Azure Resource Manager 端點。  
  若為 Azure Stack 開發套件，URL 是：`https://management.local.azurestack/external`。  
  若為 Azure Stack 整合式系統，URL 是：`https://management.[Region}.[External FQDN]`。

    ![X](./media/azure-stack-install-visual-studio/image5.png)

5. 選取 [新增] 。  

    Visual Studio 會呼叫 Azure Resource Manager 並探索端點 (包括 Active Directory Federation Services (AD FS) 的驗證端點)。

    ![登入 Cloud Explorer 後並連接到 Azure Stack 的螢幕擷取畫面](./media/azure-stack-install-visual-studio/image6.png)

6. 從 [檢視] 功能表選取 [Cloud Explorer]。
7. 選取 [新增帳戶] 並使用 AD FS 認證來登入。  

    ![X](./media/azure-stack-install-visual-studio/image7.png)

    Cloud Explorer 會查詢可用訂用帳戶。 您可以選取其中一個可用訂用帳戶來進行管理。

    ![X](./media/azure-stack-install-visual-studio/image8.png)

8. 瀏覽您現有的資源、資源群組，或部署範本。

## <a name="next-steps"></a>後續步驟

 - 深入了解關於其他 Visual Studio 版本的[共存](https://msdn.microsoft.com/library/ms246609.aspx)。
 - [開發適用於 Azure Stack 的範本](azure-stack-develop-templates.md)