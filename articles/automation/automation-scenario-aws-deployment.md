---
title: 在 Amazon Web Services 中自動部署 VM
description: 本文示範如何使用 Azure 自動化來自動化 Amazon Web 服務 VM 的建立
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 03/16/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 4f49adf006e8d55337220fad9ee84de65209880b
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34193476"
---
# <a name="azure-automation-scenario---provision-an-aws-virtual-machine"></a>Azure 自動化案例 - 佈建 AWS 虛擬機器
在本文中，您將了解如何利用「Azure 自動化」在 Amazon Web Service (AWS) 訂用帳戶中佈建虛擬機器，並為該 VM 提供一個特定名稱 (AWS 稱之為為 VM 「加上標籤」)。

## <a name="prerequisites"></a>先決條件
基於本文的目的，您需要擁有 Azure 自動化帳戶和 AWS 訂用帳戶。 如需設定 Azure 自動化帳戶以及使用 AWS 訂用帳戶認證進行設定的詳細資訊，請檢閱[使用 Amazon Web Services 設定驗證](automation-config-aws-account.md)。 繼續之前，應該先使用您的 AWS 訂用帳戶認證來建立或更新此帳戶，因為您會在下面的步驟中參考此帳戶。

## <a name="deploy-amazon-web-services-powershell-module"></a>部署 Amazon Web Services PowerShell 模組
您的 VM 佈建 Runbook 會利用 AWS PowerShell 模組來執行其工作。 執行下列步驟，將模組新增至使用 AWS 訂用帳戶認證所設定的自動化帳戶。  

1. 開啟網頁瀏覽器，並瀏覽至 [PowerShell 資源庫](http://www.powershellgallery.com/packages/AWSPowerShell/)，然後按一下 [部署至 Azure 自動化] 按鈕。<br><br> ![AWS PS 模組匯入](./media/automation-scenario-aws-deployment/powershell-gallery-download-awsmodule.png)
2. 系統會帶您前往 Azure 登入頁面，而在驗證之後，則會帶您到 Azure 入口網站並顯示下列頁面：<br><br> ![匯入模組頁面](./media/automation-scenario-aws-deployment/deploy-aws-powershell-module-parameters.png)
3. 選取所要使用的「自動化帳戶」，然後按一下 [確定] 來開始進行部署。

   > [!NOTE]
   > 將 PowerShell 模組匯入到「Azure 自動化」時，也會擷取 Cmdlet，而必須等到模組完全完成匯入並擷取 Cmdlet 之後，才會顯示這些活動。 此程序需要數分鐘的時間。  
   > <br>

1. 在 Azure 入口網站中，開啟步驟 3 中所參考的自動化帳戶。
2. 按一下 [資產] 磚，然後在 [資產] 窗格上選取 [模組] 磚。
3. 在 [模組] 頁面上，您會在清單中看到 **AWSPowerShell** 模組。

## <a name="create-aws-deploy-vm-runbook"></a>建立 AWS 部署 VM Runbook
部署 AWS PowerShell 模組之後，您現在即可撰寫 Runbook，以使用 PowerShell 指令碼自動在 AWS 中佈建虛擬機器。 下面的步驟將示範如何在「Azure 自動化」中利用原生 PowerShell 指令碼。  

> [!NOTE]
> 如需進一步選項以及此指令碼的相關資訊，請瀏覽 [PowerShell 資源庫](https://www.powershellgallery.com/packages/New-AwsVM/DisplayScript)。
> 

1. 開啟 PowerShell 工作階段並且輸入下列指令，以從 PowerShell 資源庫下載 PowerShell 指令碼 New-AwsVM：<br>
   ```
   Save-Script -Name New-AwsVM -Path <path>
   ```
   <br>
2. 從 Azure 入口網站中，開啟您的自動化帳戶，然後在左側的 [流程自動化] 區段底下選取 [Runbook]。  
3. 從 [Runbook] 頁面中，選取 [新增 Runbook]。
4. 在 [新增 Runbook] 窗格上，選取 [快速建立] \(建立新的 Runbook\)。
5. 在 [Runbook] 屬性窗格上，於 Runbook 的 [名稱] 方塊中輸入名稱，並從 [Runbook 類型] 下拉式清單中選取 [PowerShell]，然後按一下 [建立]。<br><br> ![建立 Runbook 窗格](./media/automation-scenario-aws-deployment/runbook-quickcreate-properties.png)
6. [編輯 PowerShell Runbook] 頁面出現時，將 PowerShell 指令碼複製並貼入 Runbook 撰寫畫布。<br><br> ![Runbook PowerShell 指令碼](./media/automation-scenario-aws-deployment/runbook-powershell-script.png)<br>
   
    > [!NOTE]
    > 使用範例 PowerShell 指令碼時，請注意下列事項：
    > 
    > * Runbook 包含一些預設參數值。 請評估所有預設值，並視需要更新。
    > * 如果您已將 AWS 認證儲存為名稱與 **AWScred** 不同的認證資產，則需要更新指令碼的第 57 行，使其相符。  
    > * 在 PowerShell 中使用 AWS CLI 命令 (特別是使用此範例 Runbook) 時，您必須指定 AWS 區域。 否則，Cmdlet 會失敗。 如需進一步詳細資訊，請檢視 AWS Tools for PowerShell 文件中的 AWS 主題： [指定 AWS 區域](http://docs.aws.amazon.com/powershell/latest/userguide/pstools-installing-specifying-region.html) 。  
    >

7. 若要從 AWS 訂用帳戶擷取映像名稱清單，請啟動 PowerShell ISE 並匯入 AWS PowerShell 模組。 將 ISE 環境中的 **Get-AutomationPSCredential** 取代為 **AWScred = Get-Credential**，以驗證 AWS。 這會提示您輸入認證，您可以提供您的**存取金鑰識別碼**作為使用者名稱，以及**祕密存取金鑰**作為密碼。 請參閱下面的範例：  

        #Sample to get the AWS VM available images
        #Please provide the path where you have downloaded the AWS PowerShell module
        Import-Module AWSPowerShell
        $AwsRegion = "us-west-2"
        $AwsCred = Get-Credential
        $AwsAccessKeyId = $AwsCred.UserName
        $AwsSecretKey = $AwsCred.GetNetworkCredential().Password
   
        # Set up the environment to access AWS
        Set-AwsCredentials -AccessKey $AwsAccessKeyId -SecretKey $AwsSecretKey -StoreAs AWSProfile
        Set-DefaultAWSRegion -Region $AwsRegion
   
        Get-EC2ImageByName -ProfileName AWSProfile

    會傳回下列輸出：<br><br>
   ![取得 AWS 映像](./media/automation-scenario-aws-deployment/powershell-ise-output.png)<br>  
8. 在自動化變數中複製並貼上其中一個映像名稱，在 Runbook 中參考為 **$InstanceType**。 由於在此範例中您使用免費的 AWS 分層訂用帳戶，因此您將使用 **t2.micro** 作為 Runbook 範例。  
9. 儲存 Runbook，然後按一下 [發佈] 來發佈 Runbook，並在出現提示時按一下 [是]。

### <a name="testing-the-aws-vm-runbook"></a>測試 AWS VM Runbook
繼續測試 Runbook 之前，您必須先驗證一些事項。 具體而言：  

* 驗證 AWS 的資產已建立並命名為 **AWScred** ，或指令碼已更新成參考您認證資產的名稱。    
* AWS PowerShell 模組已匯入 Azure 自動化  
* 已建立新的 Runbook，已驗證參數值並且視需要更新  
* Runbook 設定 [記錄和追蹤] 下的 [記錄詳細資訊記錄] 和選擇性的 [記錄進度記錄] 已設為 [開啟]。<br><br> ![Runbook 記錄和追蹤](./media/automation-scenario-aws-deployment/runbook-settings-logging-and-tracing.png)  

1. 您想要啟動 Runbook，因此按一下 [啟動]，然後在 [啟動 Runbook] 窗格開啟時，按一下 [確定]。
2. 在 [啟動 Runbook] 窗格上，提供 **VMname**。 接受您稍早已在指令碼中為其他參數預先設定的預設值。 按一下 [確定] 啟動 Runbook 工作。<br><br> ![啟動 New-AwsVM Runbook](./media/automation-scenario-aws-deployment/runbook-start-job-parameters.png)
3. 作業窗格會針對您所建立的 Runbook 作業開啟。 關閉此窗格。
4. 從 Runbook 作業頁面中選取 [所有記錄] 圖格，即可檢視作業進度及檢視輸出**串流**。<br><br> ![串流輸出](./media/automation-scenario-aws-deployment/runbook-job-streams-output.png)
5. 請登入 AWS 管理主控台 (如果目前尚未登入)，以確認正在佈建 VM。<br><br> ![AWS 主控台部署 VM](./media/automation-scenario-aws-deployment/aws-instances-status.png)

## <a name="next-steps"></a>後續步驟
* 若要開始使用圖形化 Runbook，請參閱 [我的第一個圖形化 Runbook](automation-first-runbook-graphical.md)
* 若要開始使用 PowerShell 工作流程 Runbook，請參閱 [我的第一個 PowerShell 工作流程 Runbook](automation-first-runbook-textual.md)
* 若要深入了解 Runbook 類型、其優點和限制，請參閱 [Azure 自動化 Runbook 類型](automation-runbook-types.md)
* 如需 PowerShell 指令碼支援功能的詳細資訊，請參閱 [Azure 自動化中的原生 PowerShell 指令碼支援](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/)

