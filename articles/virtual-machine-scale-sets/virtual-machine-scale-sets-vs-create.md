---
title: 使用 Visual Studio 部署虛擬機器擴展集 | Microsoft Docs
description: 使用 Visual Studio 和 Resource Manager 範本部署虛擬機器調整集
services: virtual-machine-scale-sets
ms.custom: H1Hack27Feb2017
ms.workload: na
documentationcenter: ''
author: gatneil
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ed0786b8-34b2-49a8-85b5-2a628128ead6
ms.service: virtual-machine-scale-sets
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/13/2017
ms.author: negat
ms.openlocfilehash: f0bd57836d6d54fe3e3460af27ebea4cffe3ee40
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42443927"
---
# <a name="how-to-create-a-virtual-machine-scale-set-with-visual-studio"></a>如何使用 Visual Studio 建立虛擬機器擴展集
本文說明如何使用 Visual Studio 資源群組部署，部署 Azure 虛擬機器調整集。

[Azure 虛擬機器擴展集 (英文)](https://azure.microsoft.com/blog/azure-vm-scale-sets-public-preview/) 是一個 Azure 計算資源，可使用自動調整規模和負載平衡來部署和管理類似虛擬機器的集合。 您可以使用 [Azure Resource Manager 範本 (英文)](https://github.com/Azure/azure-quickstart-templates) 來佈建和部署虛擬機器擴展集。 您可以使用 Azure CLI、PowerShell、REST 部署 Azure Resource Manager 範本，也可以直接從 Visual Studio 部署。 Visual Studio 會提供一組範例範本，這些範本可部署為 Azure 資源群組部署專案的一部分。

Azure 資源群組部署是一種方式，可在單一部署作業中將一組相關的 Azure 資源群組在一起並加以發佈。 您可以在以下位置深入了解： [透過 Visual Studio 建立與部署 Azure 資源群組](../vs-azure-tools-resource-groups-deployment-projects-create-deploy.md)。

## <a name="pre-requisites"></a>先決條件
若要開始在 Visual Studio 中部署虛擬機器擴展集，您需要下列項目：

* Visual Studio 2013 或更新版本
* Azure SDK 2.7、2.8 或 2.9

>[!NOTE]
>這些指示假設您使用 Visual Studio 搭配 [Azure SDK 2.8 (英文)](https://azure.microsoft.com/blog/announcing-the-azure-sdk-2-8-for-net/)。

## <a name="creating-a-project"></a>建立專案
1. 選擇 [檔案 | 新增 | 專案]，在 Visual Studio 中建立新專案。
   
    ![新增檔案][file_new]

2. 在 [Visual C# | 雲端] 底下，選擇 [Azure Resource Manager] 來建立專案，以部署 Azure Resource Manager 範本。
   
    ![建立專案][create_project]

3. 從範本清單中，選取 Linux 或 Windows 虛擬機器調整集範本。
   
   ![選取範本][select_Template]

4. 建立專案之後，您會看到 PowerShell 部署指令碼、Azure Resource Manager 範本，以及虛擬機器擴展集的參數檔。
   
    ![Controllers\HomeController.cs][solution_explorer]

## <a name="customize-your-project"></a>自訂您的專案
現在您可以編輯範本，以針對您的應用程式需求自訂，例如新增 VM 延伸模組屬性或編輯負載平衡規則。 預設會設定虛擬機器擴展集範本來部署 AzureDiagnostics 延伸模組，以便更容易新增自動調整規模規則。 它也會部署具有公用 IP 位址的負載平衡器 (使用輸入 NAT 規則所設定)。 

負載平衡器可讓您透過 SSH (Linux) 或 RDP (Windows) 連接到 VM 執行個體。 前端連接埠範圍是從 50000 開始。 針對 Linux，這表示如果您透過 SSH 連接到通訊埠 50000，系統就會將您路由傳送到擴展集中第一個 VM 的連接埠 22。 連接到連接埠 50001，將路由傳送到第二個 VM 的連接埠 22，依此類推。

 使用 Visual Studio 編輯範本的好方法是使用 JSON 大綱來組織參數、變數和資源。 了解結構描述 Visual Studio 可以在部署範本之前在其中指出錯誤。

![JSON 總管][json_explorer]

## <a name="deploy-the-project"></a>部署專案
1. 部署 Azure Resource Manager 範本來建立虛擬機器擴展集資源。 以滑鼠右鍵按一下專案節點，然後選擇 [部署 | 新增部署]。
   
    ![部署範本][5deploy_Template]
    
2. 在 [部署到資源群組] 對話方塊中選取您的訂用帳戶。
   
    ![部署範本][6deploy_Template]

3. 您可以從這裡建立新的 Azure 資源群組，以將範本部署到其中。
   
    ![新增資源群組][new_resource]

4. 接著，按一下 [編輯參數] 以輸入要傳遞到範本的參數。 提供建立部署所需的 OS 使用者名稱和密碼。 如果尚未安裝 PowerShell Tools for Visual Studio，建議核取 [儲存密碼] 以避免隱藏的 PowerShell 命令列提示字元，或使用[金鑰保存庫支援 (英文)](https://azure.microsoft.com/blog/keyvault-support-for-arm-templates/)。
   
    ![編輯參數][edit_parameters]

5. 現在，按一下 [部署]。 [輸出] 視窗會顯示部署進度。 請注意，動作正在執行 **Deploy-AzureResourceGroup.ps1** 指令碼。
   
   ![輸出視窗][output_window]

## <a name="exploring-your-virtual-machine-scale-set"></a>探索您的虛擬機器擴展集
部署完成之後，您可以在 Visual Studio 的 [Cloud Explorer] 中檢視新的虛擬機器擴展集 (重新整理清單)。 雲端總管可讓您在開發應用程式的同時，於 Visual Studio 中管理 Azure 資源。 您也可以在 [Azure 入口網站](https://portal.azure.com)和 [Azure 資源總管 (英文)](https://resources.azure.com/) 中檢視虛擬機器擴展集。

![雲端總管][cloud_explorer]

 入口網站提供使用網頁瀏覽器以視覺化方式管理 Azure 基礎結構的最佳方式，而 Azure 資源總管則提供一個簡單的方式來探索 Azure 資源及進行偵錯，其中提供視窗來深入「執行個體檢視」，同時也顯示您要尋找之資源的 PowerShell 命令。

## <a name="next-steps"></a>後續步驟
一旦透過 Visual Studio 成功部署虛擬機器擴展集之後，就可進一步自訂專案以符合應用程式的需求。 例如，設定自動調整規模，方法是新增 **Insights** 資源、將基礎結構新增至範本 (例如獨立 VM)，或是使用自訂指令碼延伸模組部署應用程式。 您可以在 [Azure 快速入門範本 (英文)](https://github.com/Azure/azure-quickstart-templates) GitHub 儲存機制中找到良好的範例範本 (搜尋 "vmss")。

[file_new]: ./media/virtual-machine-scale-sets-vs-create/1-FileNew.png
[create_project]: ./media/virtual-machine-scale-sets-vs-create/2-CreateProject.png
[select_Template]: ./media/virtual-machine-scale-sets-vs-create/3b-SelectTemplateLin.png
[solution_explorer]: ./media/virtual-machine-scale-sets-vs-create/4-SolutionExplorer.png
[json_explorer]: ./media/virtual-machine-scale-sets-vs-create/10-JsonExplorer.png
[5deploy_Template]: ./media/virtual-machine-scale-sets-vs-create/5-DeployTemplate.png
[6deploy_Template]: ./media/virtual-machine-scale-sets-vs-create/6-DeployTemplate.png
[new_resource]: ./media/virtual-machine-scale-sets-vs-create/7-NewResourceGroup.png
[edit_parameters]: ./media/virtual-machine-scale-sets-vs-create/8-EditParameter.png
[output_window]: ./media/virtual-machine-scale-sets-vs-create/9-Output.png
[cloud_explorer]: ./media/virtual-machine-scale-sets-vs-create/12-CloudExplorer.png
