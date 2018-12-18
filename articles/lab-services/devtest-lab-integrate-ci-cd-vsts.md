---
title: 將 Azure DevTest Labs 整合到 VSTS 持續整合和傳遞管線 | Microsoft Docs
description: 瞭解如何將 Azure DevTest Labs 整合到 VSTS 持續整合和傳遞管線
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: a26df85e-2a00-462b-aac1-dd3539532569
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: b7ce07547eccd52a8b10d4cffecaf1456778da4a
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44301203"
---
# <a name="integrate-azure-devtest-labs-into-your-azure-devops-continuous-integration-and-delivery-pipeline"></a>將 Azure DevTest Labs 整合到 Azure DevOps 持續整合和傳遞管線
您可以使用安裝在 Azure DevOps 中的 *Azure DevTest Labs 工作*擴充功能，輕鬆地整合您的 CI/CD 組建和發行管線與 Azure DevTest Labs。 此延伸模組會安裝三項工作： 
* 建立 VM
* 從 VM 建立自訂映像
* 刪除 VM 

舉例來說，此程序可協助您快速地為 特定測試工作部署「黃金映像」，並在測試完成後加以刪除。

本文說明如何在一個完整的管線中建立並部署 VM、建立自訂映像，然後刪除 VM。 您通常會在自己的自訂「建置-測試-部署」管線中個別執行每項工作。

## <a name="before-you-begin"></a>開始之前
整合您的 CI/CD 管線與 Azure DevTest Labs 之前，您必須從 Visual Studio Marketplace 安裝延伸模組。
1. 移至 [Azure DevTest Labs 工作](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks)。
1. 選取 [安裝]。
1. 完成精靈。

## <a name="create-a-resource-manager-template"></a>建立 Resource Manager 範本
本節說明如何建立您用來隨需建立 Azure 虛擬機器的 Azure Resource Manager 範本。

1. 若要在您的定用帳戶中建立 Resource Manager 範本，請完成[使用 Resource Manager 範本](devtest-lab-use-resource-manager-template.md)中的程序。
1. 產生 Resource Manager 範本之前，請先在建立 VM 的過程中新增 [WinRM 構件](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts/windows-winrm)。

   必須具有 WinRM 存取，才能使用 *Azure 檔案複製*和*目標電腦上的 PowerShell* 之類的部署工作。

   > [!NOTE]
   > 當您透過共用 IP 位址使用 WinRM 時，您必須新增將外部連接埠對應至 WinRM 連接埠的 NAT 規則。 如果您使用公用 IP 位址建立 VM，則無須執行此步驟。
   >
   >

1. 將範本儲存為電腦上的檔案。 請將此檔案命名為 **CreateVMTemplate.json**。
1. 將範本簽入您的來源控制系統中。
1. 開啟文字編輯器，並在其中貼上下列程式碼。

   ```powershell
   Param( [string] $labVmId)

   $labVmComputeId = (Get-AzureRmResource -Id $labVmId).Properties.ComputeId

   # Get lab VM resource group name
   $labVmRgName = (Get-AzureRmResource -Id $labVmComputeId).ResourceGroupName

   # Get the lab VM Name
   $labVmName = (Get-AzureRmResource -Id $labVmId).Name

   # Get lab VM public IP address
   $labVMIpAddress = (Get-AzureRmPublicIpAddress -ResourceGroupName $labVmRgName
                   -Name $labVmName).IpAddress

   # Get lab VM FQDN
   $labVMFqdn = (Get-AzureRmPublicIpAddress -ResourceGroupName $labVmRgName
              -Name $labVmName).DnsSettings.Fqdn

   # Set a variable labVmRgName to store the lab VM resource group name
   Write-Host "##vso[task.setvariable variable=labVmRgName;]$labVmRgName"

   # Set a variable labVMIpAddress to store the lab VM Ip address
   Write-Host "##vso[task.setvariable variable=labVMIpAddress;]$labVMIpAddress"

   # Set a variable labVMFqdn to store the lab VM FQDN name
   Write-Host "##vso[task.setvariable variable=labVMFqdn;]$labVMFqdn"
   ```

1. 將指令碼簽入您的來源控制系統中。 請將其命名為 **GetLabVMParams.ps1** 之類的名稱。

   當您在建立發行管線的過程中對代理程式執行此指令碼時，如果您使用 *Azure 檔案複製*或*目標機器上的 PowerShell* 之類的工作步驟，指令碼將會收集您將應用程式部署至 VM 所需的值。 您通常會使用這些工作將應用程式部署至 Azure VM。 這些工作需要 VM 資源群組名稱、IP 位址和完整的網域名稱 (FDQN) 之類的值。

## <a name="create-a-release-pipeline-in-release-management"></a>在 Release Management 中建立發行管線
若要建立發行管線，請執行下列動作：

1. 在 [建置及發行] 中樞的 [發行] 索引標籤上，選取加號 (+) 按鈕。
1. 在 [建立發行定義] 視窗中，選取 [空白] 範本，然後選取 [下一步]。
1. 選取 [稍後選擇]，然後選取 [建立]，以使用一個預設的環境、但不使用任何連結的成品來建立新的發行管線。
1. 若要開啟捷徑功能表，請在新的發行管線中選取環境名稱旁邊的省略符號 (...)，然後選取 [設定變數]。 
1. 在 [設定 - 環境] 視窗中，為您在發行管線工作中使用的變數輸入下列值：

   a. 針對 [VM 名稱]，輸入您在 Azure 入口網站中建立 Resource Manager 範本時指派給 VM 的名稱。

   b. 針對 [使用者名稱]，輸入您在 Azure 入口網站中建立 Resource Manager 範本時指派給 VM 的使用者名稱。

   c. 針對 [密碼]，輸入您在 Azure 入口網站中建立 Resource Manager 範本時指派給 VM 的密碼。 使用「掛鎖」圖示可隱藏及保護密碼。

### <a name="create-a-vm"></a>建立 VM

部署的下一個階段，是建立 VM 以作為後續部署的「黃金映像」。 您可以使用特別為此目的開發的工作，在您的 Azure DevTest Lab 執行個體中建立此 VM。 

1. 在發行管線中，選取 [新增工作]。
1. 在 [部署] 索引標籤上，新增 [Azure DevTest Labs 建立 VM] 工作。 請依照下列方式設定工作：

   > [!NOTE]
   > 若要建立用於後續部署的 VM，請參閱 [Azure DevTest Labs 工作](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks)。

   a. 針對 [Azure RM 訂用帳戶]，選取 [可用的 Azure 服務連線] 清單中的連線，或對您的 Azure 訂用帳戶建立權限更具限制性的連線。 如需詳細資訊，請參閱 [Azure Resource Manager 服務端點](https://docs.microsoft.com/azure/devops/pipelines/library/service-endpoints#sep-azure-rm)。

   b. 針對 [實驗室名稱]，選取您先前建立之執行個體的名稱。

   c. 針對 [範本名稱]，輸入您儲存到原始程式碼存放庫之範本檔案的完整路徑和名稱。 您可以使用 Release Management 的內建屬性來簡化路徑，例如：

   ```
   $(System.DefaultWorkingDirectory)/Contoso/ARMTemplates/CreateVMTemplate.json
   ```

   d. 針對 [範本參數]，輸入在範本中定義之變數的參數。 請使用您在環境中定義的變數名稱，例如：

   ```
   -newVMName '$(vmName)' -userName '$(userName)' -password (ConvertTo-SecureString -String '$(password)' -AsPlainText -Force)
   ```

   e. 針對 [輸出變數 - 實驗室 VM 識別碼]，您必須提供新建立之 VM 的識別碼，供後續步驟使用。 您必須在 [輸出變數] 區段中設定會以此識別碼自動填入之環境變數的預設名稱。 如有必要，您可以編輯變數，但請記得在後續工作中使用正確的名稱。 實驗室 VM 識別碼會以下列形式寫入：

   ```
   /subscriptions/{subId}/resourceGroups/{rgName}/providers/Microsoft.DevTestLab/labs/{labName}/virtualMachines/{vmName}
   ```

1. 執行您先前建立的指令碼，以收集 DevTest Labs VM 的詳細資料。 
1. 在發行管線中選取 [新增工作]，然後在 [部署] 索引標籤上新增 [Azure PowerShell] 工作。 請依照下列方式設定工作：

   > [!NOTE]
   > 若要收集 DevTest Labs VM 的詳細資料，請參閱[部署：Azure PowerShell](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/AzurePowerShell)，並執行指令碼。

   a. 針對 [Azure 連線類型]，選取 [Azure Resource Manager]。

   b. 針對 [Azure RM 訂用帳戶]，從 [可用的 Azure 服務連線] 下方的清單中選取連線，或對您的 Azure 訂用帳戶建立權限更具限制性的連線。 如需詳細資訊，請參閱 [Azure Resource Manager 服務端點](https://docs.microsoft.com/azure/devops/pipelines/library/service-endpoints#sep-azure-rm)。

   c. 針對 [指令碼類型]，選取 [指令碼檔案]。
 
   d. 針對 [指令碼路徑]，輸入您儲存到原始程式碼存放庫之指令碼的完整路徑和名稱。 您可以使用 Release Management 的內建屬性來簡化路徑，例如：
      ```
      $(System.DefaultWorkingDirectory/Contoso/Scripts/GetLabVMParams.ps1
      ```
   e. 針對 [指令碼引數]，輸入由上一項工作以實驗室 VM 的識別碼自動填入的環境變數名稱，例如： 
      ```
      -labVmId '$(labVMId)'
      ```
    此指令碼會收集所需的值，並將其儲存在發行管線內的環境變數中，以便您在後續的步驟中加以參考。

1. 將您的應用程式部署到新的 DevTest Labs VM。 您通常會使用 *Azure 檔案複製*和*目標電腦上的 PowerShell* 工作來部署應用程式。
   這些工作的參數所需的 VM 相關資訊，會儲存在發行管線內名為 **labVmRgName**、**labVMIpAddress** 和 **labVMFqdn** 的三個設定變數中。 如果您只想要試驗性地建立 DevTest Labs VM 和自訂映像，而不為其部署應用程式，則可以略過此步驟。

### <a name="create-an-image"></a>建立映像

下一個階段是在您的 Azure DevTest Labs 執行個體中建立新部署之 VM 的映像。 後續當您想要執行開發工作或執行某些測試時，您即可使用此映像隨需建立 VM 的複本。 

1. 在發行管線中，選取 [新增工作]。
1. 在 [部署] 索引標籤上，新增 [Azure DevTest Labs 建立自訂映像] 工作。 進行下列設定：

   > [!NOTE]
   > 若要建立映像，請參閱 [Azure DevTest Labs 工作](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks)。

   a. 針對 [Azure RM 訂用帳戶]，在 [可用的 Azure 服務連線] 清單中選取連線，或對您的 Azure 訂用帳戶建立權限更具限制性的連線。 如需詳細資訊，請參閱 [Azure Resource Manager 服務端點](https://docs.microsoft.com/azure/devops/pipelines/library/service-endpoints#sep-azure-rm)。

   b. 針對 [實驗室名稱]，選取您先前建立之執行個體的名稱。

   c. 針對 [自訂映像名稱]，輸入自訂映像的名稱。

   d. (選擇性) 針對 [說明]，輸入有助於您後續選取正確映像的說明。

   e. 針對 [來源實驗室 VM - 來源實驗室 VM 識別碼]，如果您變更了先前的工作以實驗室 VM 的識別碼自動填入的環境變數預設名稱，請在此處加以編輯。 預設值為 **$(labVMId)**。

   f. 針對 [輸出變數 - 自訂映像識別碼]，您必須提供新建立之映像的識別碼，以便您在需要管理或刪除該映像時使用。 以此識別碼自動填入之環境變數的預設名稱會設定於 [輸出變數] 區段中。 如有必要，您可以編輯該變數。

### <a name="delete-the-vm"></a>刪除 VM

最後一個階段是刪除您在 Azure DevTest Labs 執行個體中部署的 VM。 在已部署的 VM 上執行您所需的開發工作或測試之後，您通常會刪除 VM。 

1. 在發行管線中選取 [新增工作]，然後在 [部署] 索引標籤上新增 [Azure DevTest Labs 刪除 VM] 工作。 進行下列設定：

      > [!NOTE]
      > 若要刪除 VM，請參閱 [Azure DevTest Labs 工作](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks)。

   a. 針對 [Azure RM 訂用帳戶]，選取 [可用的 Azure 服務連線] 清單中的連線，或對您的 Azure 訂用帳戶建立權限更具限制性的連線。 如需詳細資訊，請參閱 [Azure Resource Manager 服務端點](https://docs.microsoft.com/azure/devops/pipelines/library/service-endpoints#sep-azure-rm)。
 
   b. 針對 [實驗室 VM 識別碼]，如果您變更了先前的工作以實驗室 VM 的識別碼自動填入的環境變數預設名稱，請在此處加以編輯。 預設值為 **$(labVMId)**。

1. 輸入發行管線的名稱，並加以儲存。
1. 建立新版本，選取最新的組建，然後將其部署到管線中的單一環境。

在每個階段中，重新整理您的 DevTest Labs 執行個體在 Azure 入口網站中的檢視，以檢視要建立的 VM 和映像，以及要再次刪除的 VM。

您現在已可在必要時使用自訂映像建立 VM。


[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>後續步驟
* 了解如何[使用 Resource Manager 範本建立多個 VM 環境](devtest-lab-create-environment-from-arm.md)。
* 從[公用 DevTest Labs GitHub 存放庫](https://github.com/Azure/azure-quickstart-templates)探索更多 DevTest Labs 自動化的快速入門 Resource Manager 範本。
* 如有必要，請參閱 [Azure DevOps 疑難排解](https://docs.microsoft.com/azure/devops/pipelines/troubleshooting)頁面。
 
