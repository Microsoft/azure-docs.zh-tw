---
title: Azure 虛擬機器擴充功能和功能 | Microsoft Docs
description: 了解 Azure VM 擴充功能，以及如何搭配 Azure 虛擬機器使用
services: virtual-machines-linux
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/30/2018
ms.author: roiyz
ms.openlocfilehash: 38f885cc230df6e0edb1e5b5254faf0cd519847a
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451438"
---
# <a name="azure-virtual-machine-extensions-and-features"></a>Azure 虛擬機器擴充功能和功能
Azure 虛擬機器 (VM) 擴充功能是在 Azure VM 上提供部署後設定和自動化工作的小型應用程式。您可以使用現有的映像，然後將其自訂為部署的一部分，就不必建置自訂映像。

Azure 平台裝載許多擴充功能，包括 VM 設定、監視、安全性和公用程式應用程式等等。 發行者會將應用程式包裝成擴充功能，並簡化安裝，因此您只需要提供必要的參數即可。 

 有許多第一方和第三方的擴充功能可供選擇。如果擴充功能存放庫中沒有想要的應用程式，則您可以使用「自訂指令碼」擴充功能，並使用自己的指令碼和命令來設定 VM。

使用擴充功能的主要案例包括下列範例：
* VM 設定。您可以使用 Powershell DSC (預期狀態設定)、Chef、Puppet 和「自訂指令碼」擴充功能來安裝 VM 設定代理程式並設定您的 VM。 
* AV 產品。例如 Symantec、ESET。
* VM 弱點工具。例如 Qualys、Rapid7、HPE。
* VM 和應用程式監視工具。例如 DynaTrace、Azure 網路監看員、Site24x7 和 Stackify。

擴充功能可以與新的 VM 部署配套。 例如，這些擴充功能可以是較大部署的一部分、可以在 VM 佈建時設定應用程式，或者可以在部署之後針對任何支援的擴充功能操作系統來執行。

## <a name="how-can-i-find-what-extensions-are-available"></a>如何找到可用的擴充功能？
您可以在入口網站中 VM 刀鋒視窗的擴充功能之下，檢視可用的擴充功能。此處僅為一小部分，如需完整清單，可以使用 CLI 工具，請參閱[探索適用於 Linux 的 VM 擴充功能](features-linux.md)和[探索適用於 Windows 的 VM 擴充功能](features-windows.md)。

## <a name="how-can-i-install-an-extension"></a>如何安裝擴充功能？
您可以使用 Azure CLI、Azure PowerShell、Azure Resource Manager 範本及 Azure 入口網站來管理 Azure VM 擴充功能。 若要試用擴充功能，您可以前往 Azure 入口網站、選取「自訂指令碼」擴充功能，然後傳入命令/指令碼並執行擴充功能。

如果想要在入口網站中透過 CLI 或 Resource Manager 範本所新增的相同延伸模組，請參閱不同的延伸模組文件，例如 [Windows 自訂指令碼延伸模組](custom-script-windows.md)和 [Linux 自訂指令碼延伸模組](custom-script-linux.md)。

## <a name="how-do-i-manage-extension-application-lifecycle"></a>如何管理擴充功能應用程式週期？
您不需要直接連線至 VM，就可以安裝或刪除擴充功能。 由於 Azure 擴充功能應用程式週期是在 VM 外部管理，並整合到 Azure 平台，因此您也可以取得擴充功能的整合狀態。

## <a name="anything-else-i-should-be-thinking-about-for-extensions"></a>對於擴充功能，我還應該考慮其他任何事項嗎？
擴充程式會安裝應用程式。就如同任何應用程式，擴充功能也會有一些需求、會有支援的 Windows 和 Linux 作業系統清單，而且您必須先安裝 Azure VM 代理程式。 有些個別的 VM 擴充功能應用程式可能會有自己的環境必要條件，例如對端點的存取。

## <a name="next-steps"></a>後續步驟
* 如需有關 Linux 代理程式和擴充功能運作方式的詳細資訊，請參閱[適用於 Linux 的 Azure VM 擴充功能和功能](features-linux.md)。
* 如需有關 Windows 客體代理程式和擴充功能運作方式的詳細資訊，請參閱[適用於 Windows 的 Azure VM 擴充功能和功能](features-windows.md)。  
* 若要安裝 Windows 客體代理程式，請參閱 [Azure Windows 虛擬機器代理程式概觀](agent-windows.md)。  
* 若要安裝 Linux 代理程式，請參閱 [Azure Linux 虛擬機器代理程式概觀](agent-linux.md)。  

