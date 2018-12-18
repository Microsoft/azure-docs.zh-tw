---
title: 將容器中的 .NET 應用程式部署到 Azure Service Fabric | Microsoft Docs
description: 了解如何使用 Visual Studio 將現有 .NET 應用程式容器化，並在 Service Fabric 本機為容器偵錯。 需將容器化的應用程式推送至 Azure 容器登錄，並部署到 Service Fabric 叢集。 部署到 Azure 時，應用程式會使用 Azure SQL 資料庫保存資料。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/18/2018
ms.author: ryanwi
ms.openlocfilehash: 7573746b91f057d83a299d54801785118cc1c878
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44380126"
---
# <a name="tutorial-deploy-a-net-application-in-a-windows-container-to-azure-service-fabric"></a>教學課程：將 Windows 容器中的 .NET 應用程式部署到 Azure Service Fabric

本教學課程示範如何將現有的 ASP.NET 應用程式容器化，並封裝為 Service Fabric 應用程式。  在 Service Fabric 開發叢集上本機執行容器，然後將此應用程式部署到 Azure。  應用程式的資料會保存在 [Azure SQL Database](/azure/sql-database/sql-database-technical-overview) 中。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 使用 Visual Studio 將現有應用程式容器化
> * 建立 Azure SQL 資料庫
> * 建立 Azure 容器登錄
> * 將 Service Fabric 應用程式部署至 Azure

## <a name="prerequisites"></a>必要條件

1. 如果您沒有 Azure 訂用帳戶，請[建立免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
2. 安裝 [Docker CE for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description)，以便在 Windows 10 上執行容器。
3. 安裝 [Service Fabric 執行階段 6.2 版或更新版本](service-fabric-get-started.md)和 [Service Fabric SDK 3.1 版](service-fabric-get-started.md)或更新版本。
4. 安裝 [Visual Studio 2017 15.7 版](https://www.visualstudio.com/)或更新版本，其中包含 **Azure 開發**及 **ASP.NET 和 Web 開發**工作負載。
5. 安裝 [Azure PowerShell][link-azure-powershell-install]
 

## <a name="download-and-run-fabrikam-fiber-callcenter"></a>下載並執行 Fabrikam Fiber CallCenter
下載 [Fabrikam Fiber CallCenter][link-fabrikam-github] 範例應用程式。  按一下 [下載封存] 連結。  從 fabrikam.zip檔案中的 sourceCode目錄 ，解壓縮 sourceCode.zip檔案，然後將 VS2015 目錄解壓縮到電腦中。

確認 Fabrikam Fiber CallCenter 應用程式建置及執行正常。  以**系統管理員**身分啟動 Visual Studio，開啟 [FabrikamFiber.CallCenter.sln][link-fabrikam-github] 檔案。  按 F5 進行偵錯並執行應用程式。

![Fabrikam Web 範例][fabrikam-web-page]

## <a name="containerize-the-application"></a>將應用程式容器化
在 [FabrikamFiber.Web] 專案上按一下滑鼠右鍵 > [新增]  >  [容器協調器支援]。  選取 [Service Fabric] 作為容器協調器，然後按一下 [確定]。

按一下 [是]，可立即將 Docker 切換到 Windows 容器。

現在在方案中已建立新的 Service Fabric 應用程式專案 **FabrikamFiber.CallCenterApplication**。  並已在現有 **FabrikamFiber.Web** 專案中新增 Dockerfile。  **PackageRoot** 目錄也已新增至 **FabrikamFiber.Web** 專案，其中包含新 FabrikamFiber.Web 服務的服務資訊清單和設定。 

現在準備好在 Service Fabric 應用程式中建置及封裝此容器。 在您的電腦上建置完容器映像之後，您便可以將它推送到任何容器登錄，然後下拉到任何主機來執行。

## <a name="create-an-azure-sql-db"></a>建立 Azure SQL 資料庫
在生產環境中執行 Fabrikam Fiber CallCenter 應用程式時，資料必須保存在資料庫中。 目前無法保證資料能保存在容器中，因此請不要將 SQL Server 生產環境的資料儲存在容器中。

建議使用 [Azure SQL Database](/azure/sql-database/sql-database-get-started-powershell)。 若要在 Azure 中設定及執行受控 SQL Server 資料庫，執行下列指令碼。  視需要修改指令碼變數。 clientIP 是開發電腦的 IP 位址。  如果您在公司防火牆後方，開發電腦的 IP 位址可能是無法向網際網路公開的 IP 位址。  您也可以透過 [Azure 入口網站](https://portal.azure.com)設定 SQL Database 的伺服器防火牆規則，入口網站中會列出您的電腦 IP 位址。

```powershell
$subscriptionID="<subscription ID>"

# Sign in to your Azure account and select your subscription.
Login-AzureRmAccount -SubscriptionId $subscriptionID 

# The data center and resource name for your resources.
$dbresourcegroupname = "fabrikam-fiber-db-group"
$location = "southcentralus"

# The logical server name: Use a random value or replace with your own value (do not capitalize).
$servername = "fab-fiber-$(Get-Random)"

# Set an admin login and password for your database.
# The login information for the server.
$adminlogin = "ServerAdmin"
$password = "Password@123"

# The IP address of your development computer that accesses the SQL DB.
$clientIP = "24.18.117.76"

# The database name.
$databasename = "call-center-db"

# Create a new resource group for your deployment and give it a name and a location.
New-AzureRmResourceGroup -Name $dbresourcegroupname -Location $location

# Create the SQL server.
New-AzureRmSqlServer -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -Location $location `
    -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

# Create the firewall rule to allow your development computer to access the server.
New-AzureRmSqlServerFirewallRule -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -FirewallRuleName "AllowClient" -StartIpAddress $clientIP -EndIpAddress $clientIP

# Creeate the database in the server.
New-AzureRmSqlDatabase  -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -DatabaseName $databasename `
    -RequestedServiceObjectiveName "S0"

Write-Host "Server name is $servername"
```

## <a name="update-the-web-config"></a>更新 Web 設定
回到 **FabrikamFiber.Web** 專案，更新 **web.config** 檔案中的連接字串以指向容器中的 SQL Server。  更新連接字串的 Server 這個部分，改為之前的指令碼所建立的伺服器。 

```xml
<add name="FabrikamFiber-Express" connectionString="Server=tcp:fab-fiber-1300282665.database.windows.net,1433;Initial Catalog=call-center-db;Persist Security Info=False;User ID=ServerAdmin;Password=Password@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" providerName="System.Data.SqlClient" />
<add name="FabrikamFiber-DataWarehouse" connectionString="Server=tcp:fab-fiber-1300282665.database.windows.net,1433;Initial Catalog=call-center-db;Persist Security Info=False;User ID=ServerAdmin;Password=Password@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" providerName="System.Data.SqlClient" />
  
```
>[!NOTE]
>您可以使用任何慣用的 SQL Server 來進行本機偵錯，只要能夠從您的主機連線到該 SQL Server 即可。 不過，**localdb** 不支援 `container -> host` 通訊。 在建置 Web 應用程式的發行組建時，如果您想要使用不同的 SQL 資料庫，請在 web.release.config 檔案中新增另一個連接字串。

## <a name="run-the-containerized-application-locally"></a>在本機執行容器化的應用程式
按 **F5** 在本機 Service Fabric 開發叢集中執行容器中的應用程式並進行偵錯。 如果出現訊息方塊，詢問您是否要將 Visual Studio 專案目錄的讀取和執行權限授與 'ServiceFabricAllowedUsers' 群組，請按一下 [是]。

## <a name="create-a-container-registry"></a>建立容器登錄庫
現在應用程式是在本機執行，可以開始準備部署至 Azure。  容器映像需要存放在容器登錄中。  使用下列指令碼建立 [Azure 容器登錄](/azure/container-registry/container-registry-intro)。 其他 Azure 訂用帳戶會看到容器登錄名稱，因此此名稱必須是唯一的。
將應用程式部署到 Azure 前，需將容器映像推送至此登錄。  當應用程式部署到 Azure 中的叢集時，會從此登錄提取容器映像。

```powershell
# Variables
$acrresourcegroupname = "fabrikam-acr-group"
$location = "southcentralus"
$registryname="fabrikamregistry$(Get-Random)"

New-AzureRmResourceGroup -Name $acrresourcegroupname -Location $location

$registry = New-AzureRMContainerRegistry -ResourceGroupName $acrresourcegroupname -Name $registryname -EnableAdminUser -Sku Basic
```

## <a name="create-a-service-fabric-cluster-on-azure"></a>在 Azure 中建立 Service Fabric 叢集
Service Fabric 應用程式執行於叢集，也就是一組連接網路的虛擬或實體機器。  您需要先在 Azure 中建立 Service Fabric 叢集，才能將應用程式部署至 Azure。

您可以：
- 從 Visual Studio 建立測試叢集。 此選項可讓您使用慣用的組態，直接從 Visual Studio 建立安全的叢集。 
- [從範本建立安全叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)

本教學課程是從 Visual Studio建立叢集，這非常適合用於測試案例。 如果您用其他方式建立叢集或使用現有叢集，可以複製和貼上您的連線端點，或從訂用帳戶中選擇它。 

在建立叢集時，選擇支援執行容器的 SKU。 在叢集節點上的 Windows Server 作業系統必須相容於您容器的 Windows Server 作業系統。 若要深入了解，請參閱 [Windows Server 容器作業系統和主機作業系統的相容性](service-fabric-get-started-containers.md#windows-server-container-os-and-host-os-compatibility)。 根據預設，本教學課程會建立以 Windows Server 2016 LTSC 為基礎的 Docker 映像。 以此映像為基礎的容器將會在叢集上執行，而叢集會透過具有容器的 Windows Server 2016 Datacenter 來建立。 不過，如果您建立的叢集或使用的現有叢集，是以具有容器的 Windows Server Datacenter Core 1709 為基礎，您必須變更容器所依據的 Windows Server 作業系統映像。 開啟 **FabrikamFiber.Web** 專案中的 [Dockerfile]，為現有的 `FROM` 陳述式加上註解 (以 `windowsservercore-ltsc` 為基礎)，並對以 `windowsservercore-1709` 為基礎的 `FROM` 陳述式取消註解。 

1. 以滑鼠右鍵按一下 [方案總管] 中的 [FabrikamFiber.CallCenterApplication] 應用程式專案，然後選擇 [發佈]。

2. 使用您的 Azure 帳戶登入，以便存取您的訂用帳戶。 

3. 選取 [連線端點] 下拉式清單，然後選取 [建立新叢集...] 選項。    
        
4. 在 [建立叢集] 對話方塊中，修改下列設定：

    1. 在 [叢集名稱] 欄位中指定叢集的名稱，以及您想要使用的訂用帳戶和位置。
    2. 您可以選擇是否修改節點的數目。 根據預設，您有三個節點，這是測試 Service Fabric 案例所需的最少節點數。
    3. 選取 [憑證] 索引標籤。在此索引標籤中，輸入要用來保護叢集憑證的密碼。 此憑證可協助保護您的叢集。 您也可以修改您要儲存憑證的路徑。 Visual Studio 也可以為您匯入憑證，因為這是要將應用程式發佈至叢集所需的項目。
    4. 選取 [VM 詳細資料] 索引標籤。指定您想用於組成叢集之虛擬機器 (VM) 的密碼。 使用者名稱和密碼可用來從遠端連線到 VM。 您也必須選取 VM 機器大小，並可視需要變更 VM 映像。
    5. 在 [進階] 索引標籤中，列出應用程式連接埠，這是叢集部署時要在負載平衡器中開啟的連接埠。 在 [方案總管] 中，開啟 FabrikamFiber.Web -> PackageRoot -> ServiceManifest.xml。  [端點] 會列出 Web 前端的連接埠。  您也可以新增現有的 Application Insights 金鑰，此金鑰會用於路由傳送應用程式記錄檔。
    6. 當您完成設定修改時，請選取 [建立] 按鈕。 
5. 建立作業需要幾分鐘才能完成；輸出視窗會指出叢集何時建立完成。
    

## <a name="allow-your-application-running-in-azure-to-access-the-sql-db"></a>允許在 Azure 中執行的應用程式存取 SQL 資料庫
您先前已建立 SQL 防火牆規則，讓在本機執行的應用程式有存取權。  接著，您需要讓在 Azure 中執行的應用程式存取 SQL 資料庫。  為 Service Fabric 叢集建立[虛擬網路服務端點](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview)，然後再建立規則以允許該端點存取 SQL 資料庫。

```powershell
# Create a virtual network service endpoint
$clusterresourcegroup = "fabrikamfiber.callcenterapplication_RG"
$resource = Get-AzureRmResource -ResourceGroupName $clusterresourcegroup -ResourceType Microsoft.Network/virtualNetworks | Select-Object -first 1
$vnetName = $resource.Name

Write-Host 'Virtual network name: ' $vnetName 

# Get the virtual network by name.
$vnet = Get-AzureRmVirtualNetwork `
  -ResourceGroupName $clusterresourcegroup `
  -Name              $vnetName

Write-Host "Get the subnet in the virtual network:"

# Get the subnet, assume the first subnet contains the Service Fabric cluster.
$subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet | Select-Object -first 1

$subnetName = $subnet.Name
$subnetID = $subnet.Id
$addressPrefix = $subnet.AddressPrefix

Write-Host "Subnet name: " $subnetName " Address prefix: " $addressPrefix " ID: " $subnetID

# Assign a Virtual Service endpoint 'Microsoft.Sql' to the subnet.
$vnet = Set-AzureRmVirtualNetworkSubnetConfig `
  -Name            $subnetName `
  -AddressPrefix   $addressPrefix `
  -VirtualNetwork  $vnet `
  -ServiceEndpoint Microsoft.Sql | Set-AzureRmVirtualNetwork

$vnet.Subnets[0].ServiceEndpoints;  # Display the first endpoint.

# Add a SQL DB firewall rule for the virtual network service endpoint
$subnet = Get-AzureRmVirtualNetworkSubnetConfig `
  -Name           $subnetName `
  -VirtualNetwork $vnet;

$VNetRuleName="ServiceFabricClusterVNetRule"
$vnetRuleObject1 = New-AzureRmSqlServerVirtualNetworkRule `
  -ResourceGroupName      $dbresourcegroupname `
  -ServerName             $servername `
  -VirtualNetworkRuleName $VNetRuleName `
  -VirtualNetworkSubnetId $subnetID;
```
## <a name="deploy-the-application-to-azure"></a>將應用程式部署至 Azure
現在應用程式已備妥，您可以直接從 Visual Studio 將其部署到 Azure 中的叢集。  以滑鼠右鍵按一下 [方案總管] 中的 [FabrikamFiber.CallCenterApplication] 應用程式專案，然後選擇 [發佈]。  在 [連線端點] 中選取您先前建立的叢集端點。  在 [Azure 容器登錄] 中，選取您先前建立的容器登錄。  按一下 [發佈] 將應用程式發佈至 Azure 中的叢集。

![發佈應用程式][publish-app]

輸出視窗會顯示部署進度。  部署應用程式後，開啟瀏覽器並鍵入叢集位址和應用程式連接埠。 例如： http://fabrikamfibercallcenter.southcentralus.cloudapp.azure.com:8659/。

![Fabrikam Web 範例][fabrikam-web-page-deployed]

## <a name="set-up-continuous-integration-and-deployment-cicd-with-a-service-fabric-cluster"></a>設定 Service Fabric 叢集的持續整合和部署 (CI/CD)
若要了解如何使用 Azure DevOps 設定對 Service Fabric 叢集的 CI/CD 應用程式部署，請參閱[教學課程：透過 CI/CD 將應用程式部署至 Service Fabric 叢集](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)。 該教學課程中說明的程序與此 (FabrikamFiber) 專案的相同，只要略過下載投票範例的步驟，並將 FabrikamFiber 取代為存放庫名稱 (而不是「投票」) 即可。

## <a name="clean-up-resources"></a>清除資源
完成時，請務必移除您建立的所有資源。  最簡單的方式是移除資源群組，包括 Service Fabric 叢集、Azure SQL 資料庫、Azure 容器登錄。

```powershell
$dbresourcegroupname = "fabrikam-fiber-db-group"
$acrresourcegroupname = "fabrikam-acr-group"
$clusterresourcegroupname="fabrikamcallcentergroup"

# Remove the Azure SQL DB
Remove-AzureRmResourceGroup -Name $dbresourcegroupname

# Remove the container registry
Remove-AzureRmResourceGroup -Name $acrresourcegroupname

# Remove the Service Fabric cluster
Remove-AzureRmResourceGroup -Name $clusterresourcegroupname
```

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 使用 Visual Studio 將現有應用程式容器化
> * 建立 Azure SQL 資料庫
> * 建立 Azure 容器登錄
> * 將 Service Fabric 應用程式部署至 Azure

在本教學課程的下一個部分，了解如何[將具有 CI/CD 的容器應用程式部署到 Service Fabric 叢集](service-fabric-tutorial-deploy-container-app-with-cicd-vsts.md)。

[link-fabrikam-github]: https://aka.ms/fabrikamcontainer
[link-azure-powershell-install]: /powershell/azure/install-azurerm-ps
[link-servicefabric-create-secure-clusters]: service-fabric-cluster-creation-via-arm.md
[link-visualstudio-cd-extension]: https://aka.ms/cd4vs
[link-servicefabric-containers]: service-fabric-get-started-containers.md
[link-servicefabric-createapp]: service-fabric-create-your-first-application-in-visual-studio.md
[link-azure-portal]: https://portal.azure.com
[link-sf-clustertemplate]: https://aka.ms/securepreviewonelineclustertemplate
[link-azure-pricing-calculator]: https://azure.microsoft.com/pricing/calculator/
[link-azure-subscription]: https://azure.microsoft.com/free/
[link-vsts-account]: https://www.visualstudio.com/team-services/pricing/
[link-azure-sql]: /azure/sql-database/

[fabrikam-web-page]: media/service-fabric-host-app-in-a-container/fabrikam-web-page.png
[fabrikam-web-page-deployed]: media/service-fabric-host-app-in-a-container/fabrikam-web-page-deployed.png
[publish-app]: media/service-fabric-host-app-in-a-container/publish-app.png
