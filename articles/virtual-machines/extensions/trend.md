---
title: 在 VM 上安裝 Trend Micro Deep Security | Microsoft Docs
description: 本文說明如何在 Azure 中，在以傳統部署模型建立的 VM 上安裝和設定 Trend Micro 安全性。
services: virtual-machines-windows
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: e991b635-f1e2-483f-b7ca-9d53e7c22e2a
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-multiple
ms.devlang: na
ms.topic: article
ms.date: 04/20/2018
ms.author: roiyz
ms.openlocfilehash: 134105ee2446f3c7bf0f19e2f9611597ee9bedfe
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451947"
---
# <a name="how-to-install-and-configure-trend-micro-deep-security-as-a-service-on-a-windows-vm"></a>如何在 Windows VM 上安裝和設定 Trend Micro Deep Security as a Service
[!INCLUDE [virtual-machines-extensions-deprecation-statement](../../../includes/virtual-machines-extensions-deprecation-statement.md)]
本文說明如何在執行 Windows Server 的新或現有虛擬機器 (VM) 上，安裝和設定 Trend Micro Deep Security as a Service。 Deep Security as a Service 包括反惡意程式碼防護、防火牆、入侵防禦系統及完整監視。

透過 VM 代理程式，用戶端會安裝為安全性延伸模組。 在新的虛擬機器上，您將安裝 Deep Security Agent，因為 Azure 入口網站會自動建立 VM 代理程式。

使用 Azure 傳統入口網站、Azure CLI 或 PowerShell 建立的現有 VM 可能沒有 VM 代理程式。 對於沒有 VM 代理程式的現有虛擬機器，您必須先下載與安裝 VM 代理程式。 本文將探討這兩種狀況。

如果您已有 Trend Micro 的內部部署解決方案的目前訂用帳戶，您可以用它來協助保護 Azure 虛擬機器的安全。 如果您還不是 Symantec 客戶，您可以註冊試用訂用帳戶。 如需有關此解決方案的詳細資訊，請參閱 Trend Micro 部落格文章 [適用於 Deep Security 的 Microsoft Azure VM 代理程式延伸模組](http://go.microsoft.com/fwlink/p/?LinkId=403945)。

## <a name="install-the-deep-security-agent-on-a-new-vm"></a>在新的 VM 上安裝 Deep Security 代理程式

當您使用來自 **Marketplace** 的映像建立虛擬機器時，[Azure 入口網站](http://portal.azure.com)可讓您安裝 Trend Micro 安全性擴充功能。 如果您打算建立單一虛擬機器，使用此入口網站可輕易地新增 Trend Micro 的防護。

使用來自 **Marketplace** 的項目會開啟能協助您設定虛擬機器的精靈。 您會使用精靈中的第三個面板 [設定] 刀鋒視窗，來安裝 Trend Micro 安全性擴充功能。  如需一般指示，請參閱[在 Azure 入口網站中建立執行 Windows 的虛擬機器](../windows/classic/tutorial.md)。

當您進入精靈的**設定**刀鋒視窗時，請執行下列步驟：

1. 按一下 [擴充功能]，然後在下一個窗格中按一下 [新增擴充功能]。

   ![開始新增擴充功能][1]

2. 在 [新增資源] 窗格中選取 [Deep Security 代理程式]。 在 [Deep Security 代理程式] 窗格中，按一下 [建立]。

   ![識別 Deep Security Agent][2]

3. 輸入擴充功能的**租用戶識別碼**和**租用戶啟用密碼**。 您可以選擇性地輸入**安全性原則識別碼**。 然後按一下 [確定] 來新增用戶端。

   ![提供擴充功能詳細資料][3]

## <a name="install-the-deep-security-agent-on-an-existing-vm"></a>在現有 VM 上安裝 Deep Security 代理程式
若要在現有的 VM 上安裝代理程式，您需要下列項目：

* 在本機電腦上安裝 Azure PowerShell 模組 0.8.2 版或更新版本。 您可以使用 **Get-Module azure | format-table version** 命令來檢查已安裝的 Azure PowerShell 版本。 如需最新版本的指示與連結，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。 使用 `Add-AzureAccount`登入您的 Azure 訂用帳戶。
* 在目標虛擬機器上安裝 VM 代理程式。

首先，確認已安裝 VM 代理程式。 填寫雲端服務名稱和虛擬機器名稱，然後在系統管理員層級 Azure PowerShell 命令提示字元上執行下列命令。 取代括弧內 (包括 < 和 > 字元) 的所有項目。

    $CSName = "<cloud service name>"
    $VMName = "<virtual machine name>"
    $vm = Get-AzureVM -ServiceName $CSName -Name $VMName
    write-host $vm.VM.ProvisionGuestAgent

如果您不知道雲端服務和虛擬機器名稱，請執行 **Get-AzureVM** ，來顯示目前訂用帳戶中所有虛擬機器的該項資訊。

如果 **write-host** 命令傳回 **True**，則會安裝 VM 代理程式。 如果傳回 **False**,，請參閱 Azure 部落格文章 [VM 代理程式與延伸模組 - 第 2 部分](http://go.microsoft.com/fwlink/p/?LinkId=403947)中的指示和下載連結。

如果已安裝 VM 代理程式，請執行這些命令。

    $Agent = Get-AzureVMAvailableExtension TrendMicro.DeepSecurity -ExtensionName TrendMicroDSA

    Set-AzureVMExtension -Publisher TrendMicro.DeepSecurity –Version $Agent.Version -ExtensionName TrendMicroDSA -VM $vm | Update-AzureVM

## <a name="next-steps"></a>後續步驟
讓代理程式安裝並開始執行需要幾分鐘的時間。 之後，您必須在虛擬機器上啟用 Deep Security，才能由 Deep Security Manager 進行管理。 如需其他指示，請參閱下列文章：

* 與此解決方案相關的 Trend 文章： [Microsoft Azure 的即時雲端安全性](http://go.microsoft.com/fwlink/?LinkId=404101)
* 設定虛擬機器的 [Windows PowerShell 指令碼範例](http://go.microsoft.com/fwlink/?LinkId=404100)
* [指示](http://go.microsoft.com/fwlink/?LinkId=404099) 

## <a name="additional-resources"></a>其他資源
[如何登入執行 Windows Server 的虛擬機器]

[Azure VM 延伸模組與功能]

<!-- Image references -->
[1]: ./media/trend/new_vm_Blade3.png
[2]: ./media/trend/find_SecurityAgent.png
[3]: ./media/trend/SecurityAgentDetails.png

<!-- Link references -->
[如何登入執行 Windows Server 的虛擬機器]:../windows/classic/connect-logon.md
[Azure VM 延伸模組與功能]: http://go.microsoft.com/fwlink/p/?linkid=390493&clcid=0x409
