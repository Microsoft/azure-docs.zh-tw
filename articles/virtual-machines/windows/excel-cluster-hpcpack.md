---
title: Excel 和 SOA 適用的 HPC Pack 叢集 | Microsoft Docs
description: 開始在 Azure 中的 HPC Pack 叢集上執行大規模 Excel 和 SOA 工作負載
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager,hpc-pack
ms.assetid: cb6a9abe-caf3-44da-b911-849a50f6cfb3
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: big-compute
ms.date: 06/01/2017
ms.author: danlep
ms.openlocfilehash: 4a6327fcfe6f6e6f3b8b5c6ecbd14b832b4134c5
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39421207"
---
# <a name="get-started-running-excel-and-soa-workloads-on-an-hpc-pack-cluster-in-azure"></a>開始在 Azure 中的 HPC Pack 叢集上執行 Excel 和 SOA 工作負載
此文章說明如何在 Azure 虛擬機器上使用 Azure 快速入門範本或 Azure PowerShell 部署指令碼 (選擇性) 部署 Microsoft HPC Pack 2012 R2 叢集。 此叢集使用 Azure Marketplace VM 映像，其設計目的為使用 HPC Pack 執行 Microsoft Excel 或服務導向架構 (SOA) 工作負載。 您可以從內部部署用戶端電腦使用叢集來執行 Excel HPC 和 SOA 服務。 Excel HPC 服務包括 Excel 活頁簿卸載和 Excel 使用者定義函數或 UDF。

> [!IMPORTANT] 
> 此文章是以 HPC Pack 2012 R2 的功能、範本與指令碼為基礎。 HPC Pack 2016 目前不支援此案例。
>

[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

在較高層級上，下圖顯示您建立的 HPC Pack 叢集。

![HPC 叢集與執行 Excel 工作負載的節點][scenario]

## <a name="prerequisites"></a>必要條件
* **用戶端電腦** - 您需要 Windows 用戶端電腦，以將範例 Excel 和 SOA 工作提交至叢集。 您也需要 Windows 電腦來執行 Azure PowerShell 叢集部署指令碼 (如果您選擇該部署方法)。
* **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，只需要幾分鐘就可以建立 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/) 。
* **核心配額** - 您可能需要增加核心的配額，特別是如果您部署多核心 VM 大小的數個叢集節點。 如果您使用 Azure 快速入門範本，則 Resource Manager 中的核心配額是根據 Azure 區域而定。 在此情況下，您可能需要增加特定區域中的配額。 請參閱 [Azure 訂用帳戶限制、配額與限制](../../azure-subscription-service-limits.md)。 若要增加配額，請 [開立線上客戶支援要求](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/) (免費)。
* **Microsoft Office 授權** - 如果您使用包含 Microsoft Excel 的 Marketplace HPC Pack 2012 R2 VM 映像部署計算節點，會安裝 Microsoft Excel Professional Plus 2013 的 30 天評估版。 評估期過後，您需要提供有效的 Microsoft Office 授權來啟用 Excel，才能繼續執行工作負載。 請參閱在本文章稍候的 [啟用 Excel](#excel-activation) 。 

## <a name="step-1-set-up-an-hpc-pack-cluster-in-azure"></a>步驟 1. 在 Azure 中設定 HPC Pack 叢集
我們將說明設定 HPC Pack 2012 R2 叢集的兩種選項：第一，使用 Azure 快速入門範本和 Azure 入口網站；第二，使用 Azure PowerShell 部署指令碼。

### <a name="option-1-use-a-quickstart-template"></a>選項 1。 使用快速入門範本
使用 Azure 快速入門範本在 Azure 入口網站中快速地部署 HPC Pack 叢集。 當您在入口網站中開啟範本時，您會取得一個簡單的 UI 讓您在其中輸入叢集的設定。 步驟如下： 

> [!TIP]
> 如果您願意的話，可以使用 [Azure Marketplace 範本](https://portal.azure.com/?feature.relex=*%2CHubsExtension#create/microsofthpc.newclusterexcelcn) 來專為 Excel 工作負載建立類似的叢集。 其步驟與下文中的內容稍有不同。
> 
> 

1. 造訪 [在 GitHub 上建立 HPC 叢集範本頁面](https://github.com/Azure/azure-quickstart-templates/tree/master/create-hpc-cluster)。 您可根據意願檢視範本和原始碼的相關資訊。
1. 按一下 [部署至 Azure]  ，以在 Azure 入口網站中利用範本開始部署。
   
   ![將範本部署到 Azure][github]
1. 在入口網站中，遵循下列步驟輸入 HPC 叢集範本的參數。
   
   a. 在 [參數] 頁面上，輸入或修改範本參數的值。 (按一下說明資訊的每個設定旁邊的圖示。)下列畫面顯示範例值。 此範例會在 *hpc.local* 網域中建立名為 *hpc01* 的叢集，由一個前端節點和 2 個計算節點組成。 計算節點是從包括 Microsoft Excel 的 HPC Pack VM 映像建立。
   
   ![輸入參數][parameters-new-portal]
   
   > [!NOTE]
   > 前端節點 VM 會在 Windows Server 2012 R2 上從 HPC Pack 2012 R2 的 [最新 Marketplace 映像](https://azure.microsoft.com/marketplace/partners/microsoft/hpcpack2012r2onwindowsserver2012r2/) 自動建立。 目前此映像以 HPC Pack 2012 R2 Update 3 為基礎。
   > 
   > 計算節點 VM 會從選取之計算節點系列的最新映像建立。 選取 [ComputeNodeWithExcel] 選項做為最新的 HPC Pack 計算節點映像，包含評估版 Microsoft Excel Professional Plus 2013。 如果要部署一般 SOA 工作階段或 Excel UDF 卸載的叢集，請選擇 **ComputeNode** 選項 (不需安裝 Excel)。
   > 
   > 
   
   b. 選擇訂用帳戶。
   
   c. 建立叢集的資源群組，例如 *hpc01RG*。
   
   d. 選擇資源群組的位置，例如美國中部。
   
   e. 在 [法律條款] 頁面上檢閱條款。 如果您同意，請按一下 [購買]。 接著，當您完成範本值的設定時，請按一下 [建立]。
1. 當部署完成時 (通常需要約 30 分鐘)，從叢集前端節點匯出叢集憑證檔。 在稍後的步驟中，您要在用戶端電腦上匯入此公開憑證以提供安全 HTTP 繫結的伺服器端驗證。
   
   a. 在 Azure 入口網站中，依序移至儀表板，選取前端節點，然後按一下頁面頂端的 [連線]，即可使用遠端桌面連線。
   
    <!-- ![Connect to the head node][connect] -->
   
   b. 在憑證管理員中，使用標準程序來匯出前端節點憑證 (位於 Cert: \LocalMachine\My 之下) 而不需私密金鑰。 在此範例中，匯出 *CN = hpc01.eastus.cloudapp.azure.com*。
   
   ![匯出憑證][cert]

### <a name="option-2-use-the-hpc-pack-iaas-deployment-script"></a>選項 2。 使用 HPC Pack IaaS 部署指令碼
HPC Pack IaaS 部署指令碼提供靈活的另一種方式部署 HPC Pack 叢集。 它會在傳統部署模型中建立叢集，而範本則會使用 Azure Resource Manager 部署模型。 指令碼也和 Azure 全域或 Azure China 服務中的訂用帳戶相容。

**其他必要條件**

* **Azure PowerShell** - [安裝和設定 Azure PowerShell](/powershell/azure/overview) (0.8.10 版或更新版本)。
* **HPC Pack IaaS 部署指令碼** - 從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=44949)下載並解壓縮最新版的指令碼。 執行 `New-HPCIaaSCluster.ps1 –Version`以檢查指令碼的版本。 這篇文章根據 4.5.0 版或更新版本的指令碼。

**建立組態檔**

 HPC Pack IaaS 部署指令碼會使用 XML 組態檔做為輸入，可描述 HPC 叢集的基礎結構。 若要部署由一個前端節點和從包含 Microsoft Excel 之運算節點映像所建立的 18 個運算節點組成的叢集，請將環境的值取代為下列範例組態檔。 如需組態檔的詳細資訊，請參閱指令碼資料夾中的 Manual.rtf 檔案和 [使用 HPC Pack IaaS 部署指令碼建立 HPC 叢集](classic/hpcpack-cluster-powershell-script.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。

```
<?xml version="1.0" encoding="utf-8"?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionName>MySubscription</SubscriptionName>
    <StorageAccount>hpc01</StorageAccount>
  </Subscription>
  <Location>West US</Location>
  <VNet>
    <VNetName>hpc-vnet01</VNetName>
    <SubnetName>Subnet-1</SubnetName>
  </VNet>
  <Domain>
    <DCOption>NewDC</DCOption>
    <DomainFQDN>hpc.local</DomainFQDN>
    <DomainController>
      <VMName>HPCExcelDC01</VMName>
      <ServiceName>HPCExcelDC01</ServiceName>
      <VMSize>Medium</VMSize>
    </DomainController>
  </Domain>
   <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>HPCExcelHN01</VMName>
    <ServiceName>HPCExcelHN01</ServiceName>
    <VMSize>Large</VMSize>
    <EnableRESTAPI/>
    <EnableWebPortal/>
    <PostConfigScript>C:\tests\PostConfig.ps1</PostConfigScript>
  </HeadNode>
  <ComputeNodes>
    <VMNamePattern>HPCExcelCN%00%</VMNamePattern>
    <ServiceName>HPCExcelCN01</ServiceName>
    <VMSize>Medium</VMSize>
    <NodeCount>18</NodeCount>
    <ImageName>HPCPack2012R2_ComputeNodeWithExcel</ImageName>
  </ComputeNodes>
</IaaSClusterConfig>
```

**組態檔的相關注意事項**

* 前端節點的 **VMName** **必須**和 **ServiceName** 相同，否則 SOA 工作會無法執行。
* 請確定您會指定 **EnableWebPortal** ，所以已經產生並匯出前端節點憑證。
* 這個檔案會指定在前端節點上執行的後續組態 PowerShell 指令碼 PostConfig.ps1。 下列範例指令碼會設定 Azure 儲存體連接字串、從前端節點中移除計算節點角色，並在所有節點部署後讓所有節點上線。 

```
    # add the HPC Pack powershell cmdlets
        Add-PSSnapin Microsoft.HPC

    # set the Azure storage connection string for the cluster
        Set-HpcClusterProperty -AzureStorageConnectionString 'DefaultEndpointsProtocol=https;AccountName=<yourstorageaccountname>;AccountKey=<yourstorageaccountkey>'

    # remove the compute node role for head node to make sure the Excel workbook won’t run on head node
        Get-HpcNode -GroupName HeadNodes | Set-HpcNodeState -State offline | Set-HpcNode -Role BrokerNode

    # total number of nodes in the deployment including the head node and compute nodes, which should match the number specified in the XML configuration file
        $TotalNumOfNodes = 19

        $ErrorActionPreference = 'SilentlyContinue'

    # bring nodes online when they are deployed until all nodes are online
        while ($true)
        {
          Get-HpcNode -State Offline | Set-HpcNodeState -State Online -Confirm:$false
          $OnlineNodes = @(Get-HpcNode -State Online)
          if ($OnlineNodes.Count -eq $TotalNumOfNodes)
          {
             break
          }
          sleep 60
        }
```

**執行指令碼**

1. 在用戶端電腦上以系統管理員身分開啟 PowerShell 主控台。
1. 將目錄變更為指令碼資料夾 (在本範例中為 E:\IaaSClusterScript)。
   
   ```
   cd E:\IaaSClusterScript
   ```
1. 若要部署 HPC Pack 叢集，請執行下列命令。 這個範例假設組態檔位於 E:\HPCDemoConfig.xml。
   
   ```
   .\New-HpcIaaSCluster.ps1 –ConfigFile E:\HPCDemoConfig.xml –AdminUserName MyAdminName
   ```

HPC Pack 部署指令碼會執行一段時間。 指令碼會匯出和下載叢集憑證，並將它儲存在用戶端電腦上目前使用者的文件資料夾中。 指令碼會產生類似下方的訊息。 在下列步驟中，您將在適當的憑證存放區中匯入憑證。    

    You have enabled REST API or web portal on HPC Pack head node. Please import the following certificate in the Trusted Root Certification Authorities certificate store on the computer where you are submitting job or accessing the HPC web portal:
    C:\Users\hpcuser\Documents\HPCWebComponent_HPCExcelHN004_20150707162011.cer

## <a name="step-2-offload-excel-workbooks-and-run-udfs-from-an-on-premises-client"></a>步驟 2. 卸載 Excel 活頁簿並從內部部署用戶端執行 UDF
### <a name="excel-activation"></a>啟用 Excel
使用 ComputeNodeWithExcel VM 映像做為生產工作負載時，您需要提供有效的 Microsoft Office 授權金鑰才能啟用計算節點上的 Excel。 否則，Excel 評估版會在 30 天後到期，且執行 Excel 活頁簿會失敗並出現 COMException (0x800AC472)。 

您可以重新取得額外 30 天的 Excel 評估時間：登入前端節點，並透過 HPC 叢集管理員在所有 Excel 計算節點上執行 `%ProgramFiles(x86)%\Microsoft Office\Office15\OSPPREARM.exe` 。 您最多可以重新取得兩次。 之後，您就必須提供有效的 Office 授權金鑰。

安裝在 VM 映像上的 Office Professional Plus 2013 是含有一般大量授權金鑰 (GVLK) 的大量授權版本。 您可以透過金鑰管理服務 (KMS)/Active Directory 型啟用 (AD-BA) 或多重啟用金鑰 (MAK) 來啟用它。 

    * 若要使用 KMS/AD-BA，請使用現有的 KMS 伺服器，或使用 Microsoft Office 2013 大量授權套件設定新伺服器。 (您可視需要設定前端節點上伺服器。)接著，透過網際網路或電話啟用 KMS 主機金鑰。 然後再 clusrun `ospp.vbs` 以設定 KMS 伺服器和連接埠，以及啟用所有 Excel 計算節點上的 Office。 

    * 若要使用 MAK，請先 clusrun `ospp.vbs` 以輸入金鑰，然後再透過網際網路或電話啟用所有 Excel 計算節點。 

> [!NOTE]
> Office Professsional Plus 2013 的零售產品金鑰不適用於此 VM 映像。 如果您擁有非此 Office Professional Plus 2013 大量授權版本之 Office 或 Excel 版本的有效金鑰和安裝媒體，您也可以改用它們。 先解除安裝此大量授權版本，然後安裝您所擁有的版本。 您可以將重新安裝的 Excel 計算節點擷取成自訂 VM 映像，以在大規模部署時使用。
> 
> 

### <a name="offload-excel-workbooks"></a>卸載 Excel 活頁簿
遵循下列步驟來卸載 Excel 活頁簿，以在 Azure 的 HPC Pack 叢集上執行。 若要這樣做，您必須在用戶端電腦上安裝 Excel 2010 或 2013。

1. 使用步驟 1 中的其中一個選項，來部署具有 Excel 計算節點映像的 HPC Pack 叢集。 取得叢集憑證檔 (.cer) 以及叢集的使用者名稱和密碼。
1. 在用戶端電腦上，在 Cert:\CurrentUser\Root 下匯入叢集憑證。
1. 請確定已安裝 Excel。 在與用戶端電腦上的 Excel.exe 相同的資料夾中，建立包含下列內容的 Excel.exe.config 檔案。 這樣可確保 HPC Pack 2012 R2 Excel COM 增益集順利載入。
   
    ```
    <?xml version="1.0"?>
    <configuration>
        <startup useLegacyV2RuntimeActivationPolicy="true">
            <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0"/>
        </startup>
    </configuration>
    ```
1. 設定用戶端以將工作提交到 HPC Pack 叢集。 其中一個選項是下載完整的 [HPC Pack 2012 R2 Update 3 安裝](http://www.microsoft.com/download/details.aspx?id=49922) ，並安裝 HPC Pack 用戶端。 或者，為您的電腦 ([x64](http://www.microsoft.com/download/details.aspx?id=14632)、[x86](https://www.microsoft.com/download/details.aspx?id=5555)) 下載並安裝 [HPC Pack 2012 R2 Update 3 用戶端公用程式](https://www.microsoft.com/download/details.aspx?id=49923)及適當的 Visual C++ 2010 可轉散發套件。
1. 在此範例中，我們使用名為 ConvertiblePricing_Complete.xlsb 的範例 Excel 活頁簿。 您可以從 [這裡](https://www.microsoft.com/en-us/download/details.aspx?id=2939)下載。
1. 將 Excel 活頁簿複製到工作資料夾，例如 D:\Excel\Run。
1. 開啟 Excel 活頁簿。 在 [開發] 功能區上，按一下 [COM 增益集] 並確認 HPC Pack Excel COM 增益集已成功載入。
   
   ![HPC Pack 的 Excel 增益集][addin]
1. 藉由變更加上註解的行，編輯 Excel 中的 VBA 巨集 HPCControlMacros，如下列指令碼所示。 請將您的環境取代為適當的值。
   
   ![HPC Pack 的 Excel 巨集][macro]
   
   ```
   'Private Const HPC_ClusterScheduler = "HEADNODE_NAME"
   Private Const HPC_ClusterScheduler = "hpc01.eastus.cloudapp.azure.com"
   
   'Private Const HPC_NetworkShare = "\\PATH\TO\SHARE\DIRECTORY"
   Private Const HPC_DependFiles = "D:\Excel\Upload\ConvertiblePricing_Complete.xlsb=ConvertiblePricing_Complete.xlsb"
   
   'HPCExcelClient.Initialize ActiveWorkbook
   HPCExcelClient.Initialize ActiveWorkbook, HPC_DependFiles
   
   'HPCWorkbookPath = HPC_NetworkShare & Application.PathSeparator & ActiveWorkbook.name
   HPCWorkbookPath = "ConvertiblePricing_Complete.xlsb"
   
   'HPCExcelClient.OpenSession headNode:=HPC_ClusterScheduler, remoteWorkbookPath:=HPCWorkbookPath
   HPCExcelClient.OpenSession headNode:=HPC_ClusterScheduler, remoteWorkbookPath:=HPCWorkbookPath, UserName:="hpc\azureuser", Password:="<YourPassword>"
   ```
1. 將 Excel 活頁簿複製到上傳目錄，例如 D:\Excel\Upload。 此目錄是在 VBA 巨集的 HPC_DependsFiles 常數中指定。
1. 若要在 Azure 的叢集中執行活頁簿，請按一下工作表上的 [叢集]  按鈕。

### <a name="run-excel-udfs"></a>執行 Excel UDF
若要執行 Excel UDF，請遵循上述的步驟 1 - 3 來設定用戶端電腦。 對於 Excel UDF，您不需在計算節點上安裝 Excel 應用程式。 因此，在建立您的叢集計算節點時，您可以選擇一般計算節點映像，而不是含有 Excel 的計算節點映像。

> [!NOTE]
> [Excel 2010 和 2013 叢集連接器] 對話方塊中有 34 個字元的限制。 您可以使用此對話方塊來指定執行 UDF 的叢集。 如果完整的叢集名稱過長 (例如 hpcexcelhn01.southeastasia.cloudapp.azure.com)，該名稱就無法放入對話方塊中。 解決方法是使用完整叢集名稱的值設定電腦全域變數，例如 *CCP_IAASHN*。 然後，在對話方塊中輸入 *%CCP_IAASHN%* 做為叢集前端節點名稱。 
> 
> 

成功部署叢集之後，繼續進行下列步驟來執行內建的範例 Excel UDF。 關於自訂的 Excel UDF，請參閱這些 [資源](http://social.technet.microsoft.com/wiki/contents/articles/1198.windows-hpc-and-microsoft-excel-resources-for-building-cluster-ready-workbooks.aspx) 以建置 XLL 並將其部署在 IaaS 叢集上。

1. 開啟新的 Excel 活頁簿。 在 [開發] 功能區上，按一下 [增益集]。然後在對話方塊中按一下 [瀏覽]、瀏覽至 %CCP_HOME%Bin\XLL32 資料夾並選取範例 ClusterUDF32.xll。 如果 ClusterUDF32 不存在於用戶端電腦上，您可以從前端節點上的 %CCP_HOME%Bin\XLL32 資料夾複製它。
   
   ![選取 UDF][udf]
1. 按一下 [檔案] > [選項] > [進階]。 在 [公式] 下，核取 [允許使用者定義的 XLL 函數執行計算叢集]。 然後按一下 [選項] 並在 [叢集前端節點名稱] 中輸入完整叢集名稱。 (如先前所述，這個輸入方塊限制為 34 個字元，因此較長的叢集名稱可能不適合。 您在這裡可以對完整叢集名稱使用電腦全域變數。)
   
   ![設定 UDF][options]
1. 若要在叢集上執行 UDF 運算，請按一下值 =XllGetComputerNameC() 的儲存格並按 Enter。 函數只會擷取 UDF 執行所在的計算節點名稱。 初次執行時，認證對話方塊會提示輸入使用者名稱和密碼以連接到 IaaS 叢集。
   
   ![執行 UDF][run]
   
   有大量儲存格要計算時，請按 Alt-Shift-Ctrl + F9 以在所有儲存格上執行計算。

## <a name="step-3-run-a-soa-workload-from-an-on-premises-client"></a>步驟 3. 從內部部署用戶端執行 SOA 工作負載
若要在 HPC Pack IaaS 叢集上執行一般 SOA 應用程式，請先使用步驟 1 的其中一個方法部署叢集。 在此案例中請指定一般計算節點映像，因為在計算節點上您不需要 Excel。 接著，遵循下列步驟。

1. 擷取叢集憑證之後，在 Cert:\CurrentUser\Root 下的用戶端電腦上匯入叢集憑證。
1. 安裝 [HPC Pack 2012 R2 Update 3 SDK](http://www.microsoft.com/download/details.aspx?id=49921) 和 [HPC Pack 2012 R2 Update 3 用戶端公用程式](https://www.microsoft.com/download/details.aspx?id=49923)。 這些工具可讓您開發和執行 SOA 用戶端應用程式。
1. 下載 HelloWorldR2 [範例程式碼](https://www.microsoft.com/download/details.aspx?id=41633)。 在 Visual Studio 2010 或 2012 中開啟 HelloWorldR2.sln。 (此範例目前與較新的 Visual Studio 版本不相容。)
1. 首先建置 EchoService 專案。 接著以您部署至內部部署叢集的相同方式，將服務部署到 IaaS 叢集。 如需詳細步驟，請參閱 HelloWordR2 中的 Readme.doc。 以下一節所述的方式修改並建置 HelloWorldR2 和其他專案，以產生執行於 Azure IaaS 叢集上的 SOA 用戶端應用程式。

### <a name="use-http-binding-with-azure-storage-queue"></a>搭配使用 Http 繫結和 Azure 儲存體佇列
若要搭配使用 Http 繫結與 Azure 儲存體佇列，請對範例程式碼進行一些變更。

* 更新叢集名稱。
  
    ```
  // Before
  const string headnode = "[headnode]";
  // After e.g.
  const string headnode = "hpc01.eastus.cloudapp.azure.com";
  or
  const string headnode = "hpc01.cloudapp.net";
  ```
* (選擇性) 在 SessionStartInfo 中使用預設 TransportScheme 或明確地將它設為 Http。

```
    info.TransportScheme = TransportScheme.Http;
```

* 使用預設繫結做為 BrokerClient。
  
    ```
  // Before
  using (BrokerClient<IService1> client = new BrokerClient<IService1>(session, binding))
  // After
  using (BrokerClient<IService1> client = new BrokerClient<IService1>(session))
  ```
  
    或者明確使用 basicHttpBinding 設定。
  
    ```
  BasicHttpBinding binding = new BasicHttpBinding(BasicHttpSecurityMode.TransportWithMessageCredential);
  binding.Security.Message.ClientCredentialType = BasicHttpMessageCredentialType.UserName;    binding.Security.Transport.ClientCredentialType = HttpClientCredentialType.None;
  ```
* 選擇性地，在 SessionStartInfo 中將 UseAzureQueue 旗標設為 true。 如果未設定，它會根據預設，在叢集名稱具有 Azure 網域尾碼且 TransportScheme 為 Http 時設為 true。
  
    ```
    info.UseAzureQueue = true;
  ```

### <a name="use-http-binding-without-azure-storage-queue"></a>使用 Http 繫結而不使用 Azure 儲存體佇列
若要使用不含 Azure 儲存體佇列的 Http 繫結，請在 SessionStartInfo 中將 UseAzureQueue 旗標明確設為 false。

```
    info.UseAzureQueue = false;
```

### <a name="use-nettcp-binding"></a>使用 NetTcp 繫結
若要使用 NetTcp 繫結，組態就像是連接至內部部署叢集。 您必須在前端節點 VM 上開啟幾個端點。 舉例來說，如果您使用 HPC Pack IaaS 部署指令碼來建立叢集，請依下列方式在 Azure 入口網站中設定端點。

1. 停止 VM。
1. 新增 TCP 連接埠 9090、9087、9091、9094 分別做為工作階段、訊息代理程式、訊息代理程式背景工作和資料服務
   
    ![設定端點][endpoint-new-portal]
1. 啟動 VM。

SOA 用戶端應用程式不需要變更，除了將標頭名稱改變為 IaaS 叢集的完整名稱。

## <a name="next-steps"></a>後續步驟
* 請參閱 [這些資源](http://social.technet.microsoft.com/wiki/contents/articles/1198.windows-hpc-and-microsoft-excel-resources-for-building-cluster-ready-workbooks.aspx) 以取得使用 HPC Pack 執行 Excel 工作負載的詳細資訊。
* 請參閱 [管理 Microsoft HPC Pack 中的 SOA 服務](https://technet.microsoft.com/library/ff919412.aspx) 以取得使用 HPC Pack 部署和管理 SOA 服務的詳細資訊。

<!--Image references-->
[scenario]: ./media/excel-cluster-hpcpack/scenario.png
[github]: ./media/excel-cluster-hpcpack/github.png
[template]: ./media/excel-cluster-hpcpack/template.png
[parameters]: ./media/excel-cluster-hpcpack/parameters.png
[parameters-new-portal]: ./media/excel-cluster-hpcpack/parameters-new-portal.png
[create]: ./media/excel-cluster-hpcpack/create.png
[connect]: ./media/excel-cluster-hpcpack/connect.png
[cert]: ./media/excel-cluster-hpcpack/cert.png
[addin]: ./media/excel-cluster-hpcpack/addin.png
[macro]: ./media/excel-cluster-hpcpack/macro.png
[options]: ./media/excel-cluster-hpcpack/options.png
[run]: ./media/excel-cluster-hpcpack/run.png
[endpoint]: ./media/excel-cluster-hpcpack/endpoint.png
[endpoint-new-portal]: ./media/excel-cluster-hpcpack/endpoint-new-portal.png
[udf]: ./media/excel-cluster-hpcpack/udf.png
