---
title: 將容器中的 .NET 應用程式部署到 Azure Service Fabric
description: 了解如何使用 Visual Studio 將現有 .NET 應用程式容器化，並在 Service Fabric 本機為容器偵錯。 需將容器化的應用程式推送至 Azure 容器登錄，並部署到 Service Fabric 叢集。 部署到 Azure 時，應用程式會使用 Azure SQL 資料庫保存資料。
ms.topic: tutorial
ms.date: 07/08/2019
ms.openlocfilehash: 85e9b553000c52131c04502d496aa050b73d6d8a
ms.sourcegitcommit: a055089dd6195fde2555b27a84ae052b668a18c7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98791657"
---
# <a name="tutorial-deploy-a-net-application-in-a-windows-container-to-azure-service-fabric"></a>教學課程：將 Windows 容器中的 .NET 應用程式部署到 Azure Service Fabric

本教學課程示範如何將現有的 ASP.NET 應用程式容器化，並封裝為 Service Fabric 應用程式。  在 Service Fabric 開發叢集上本機執行容器，然後將此應用程式部署到 Azure。  應用程式的資料會保存在 [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md) 中。

在本教學課程中，您會了解如何：

> [!div class="checklist"]
>
> * 使用 Visual Studio 將現有應用程式容器化
> * 在 Azure SQL Database 中建立資料庫
> * 建立 Azure 容器登錄
> * 將 Service Fabric 應用程式部署至 Azure

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>必要條件

1. 如果您沒有 Azure 訂用帳戶，請[建立免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
2. 啟用 Windows 功能 **hyper-v** 和 **容器**。
3. 安裝 [Docker CE for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description)，以便在 Windows 10 上執行容器。
4. 安裝 [Service Fabric 執行階段 6.2 版或更新版本](service-fabric-get-started.md)和 [Service Fabric SDK 3.1 版](service-fabric-get-started.md)或更新版本。
5. 安裝 [Visual Studio 2019 16.1 版](https://www.visualstudio.com/)或更新版本，其中包含 **Azure 開發** 及 **ASP.NET 和 Web 開發** 工作負載。
6. 安裝 [Azure PowerShell][link-azure-powershell-install]

## <a name="download-and-run-fabrikam-fiber-callcenter"></a>下載並執行 Fabrikam Fiber CallCenter

1. 下載 [Fabrikam Fiber CallCenter][link-fabrikam-github] 範例應用程式。  按一下 [下載封存] 連結。  從 fabrikam.zip檔案中的 sourceCode目錄 ，解壓縮 sourceCode.zip檔案，然後將 VS2015 目錄解壓縮到電腦中。

2. 確認 Fabrikam Fiber CallCenter 應用程式建置及執行正常。  以 **系統管理員** 身分啟動 Visual Studio，並開啟 [FabrikamFiber.CallCenter.sln][link-fabrikam-github] 檔案。  按 F5 進行偵錯並執行應用程式。

   ![Fabrikam Fiber CallCenter 應用程式首頁 (執行於本機主機上) 的螢幕擷取畫面。 此頁面會顯示儀表板，其中包含支援電話的清單。][fabrikam-web-page]

## <a name="create-an-azure-sql-db"></a>建立 Azure SQL 資料庫

在生產環境中執行 Fabrikam Fiber CallCenter 應用程式時，資料必須保存在資料庫中。 目前無法保證資料能保存在容器中，因此請不要將 SQL Server 生產環境的資料儲存在容器中。

建議使用 [Azure SQL Database](../azure-sql/database/powershell-script-content-guide.md)。 若要在 Azure 中設定及執行受控 SQL Server 資料庫，執行下列指令碼。  視需要修改指令碼變數。 clientIP 是開發電腦的 IP 位址。 記下由指令碼輸出的伺服器名稱。

```powershell
$subscriptionID="<subscription ID>"

# Sign in to your Azure account and select your subscription.
Login-AzAccount -SubscriptionId $subscriptionID

# The data center and resource name for your resources.
$dbresourcegroupname = "fabrikam-fiber-db-group"
$location = "southcentralus"

# The server name: Use a random value or replace with your own value (do not capitalize).
$servername = "fab-fiber-$(Get-Random)"

# Set an admin login and password for your database.
# The login information for the server.
$adminlogin = "ServerAdmin"
$password = "Password@123"

# The IP address of your development computer that accesses the SQL DB.
$clientIP = "<client IP>"

# The database name.
$databasename = "call-center-db"

# Create a new resource group for your deployment and give it a name and a location.
New-AzResourceGroup -Name $dbresourcegroupname -Location $location

# Create the SQL server.
New-AzSqlServer -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -Location $location `
    -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

# Create the firewall rule to allow your development computer to access the server.
New-AzSqlServerFirewallRule -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -FirewallRuleName "AllowClient" -StartIpAddress $clientIP -EndIpAddress $clientIP

# Create the database in the server.
New-AzSqlDatabase  -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -DatabaseName $databasename `
    -RequestedServiceObjectiveName "S0"

Write-Host "Server name is $servername"
```

> [!TIP]
> 如果您在公司防火牆後方，開發電腦的 IP 位址可能是無法向網際網路公開的 IP 位址。 若要確認用於防火牆規則的資料庫 IP 位址是否正確，請前往 [Azure 入口網站](https://portal.azure.com)，並在 SQL Database 區段中尋找您的資料庫。 按一下其名稱，然後在 [概觀] 區段中按一下 [設定伺服器防火牆]。 「用戶端 IP 位址」是您開發機器的 IP 位址。 請確定它符合 "AllowClient" 規則中的 IP 位址。

## <a name="update-the-web-config"></a>更新 Web 設定

回到 **FabrikamFiber.Web** 專案，更新 **web.config** 檔案中的連接字串以指向容器中的 SQL Server。  更新連接字串的 Server 這個部分，改為之前指令碼所建立的伺服器名稱。 它應該會類似 "fab-fiber-751718376.database.windows.net"。 在下列 XML 中，您只需要更新 `connectionString` 屬性；`providerName` 和 `name` 屬性不需要變更。

```xml
<add name="FabrikamFiber-Express" connectionString="Server=<server name>,1433;Initial Catalog=call-center-db;Persist Security Info=False;User ID=ServerAdmin;Password=Password@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" providerName="System.Data.SqlClient" />
<add name="FabrikamFiber-DataWarehouse" connectionString="Server=<server name>,1433;Initial Catalog=call-center-db;Persist Security Info=False;User ID=ServerAdmin;Password=Password@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" providerName="System.Data.SqlClient" />
  
```

>[!NOTE]
>您可以使用任何慣用的 SQL Server 來進行本機偵錯，只要能夠從您的主機連線到該 SQL Server 即可。 不過，**localdb** 不支援 `container -> host` 通訊。 在建置 Web 應用程式的發行組建時，如果您想要使用不同的 SQL 資料庫，請在 web.release.config 檔案中新增另一個連接字串。

## <a name="containerize-the-application"></a>將應用程式容器化

1. 在 [FabrikamFiber.Web] 專案上按一下滑鼠右鍵 > [新增] >  [容器協調器支援]。  選取 [Service Fabric] 作為容器協調器，然後按一下 [確定]。

2. 若出現提示，請按一下 [是]，立即將 Docker 切換到 Windows 容器。

   現在在方案中已建立新的 Service Fabric 應用程式專案 **FabrikamFiber.CallCenterApplication**。  並已在現有 **FabrikamFiber.Web** 專案中新增 Dockerfile。  **PackageRoot** 目錄也已新增至 **FabrikamFiber.Web** 專案，其中包含新 FabrikamFiber.Web 服務的服務資訊清單和設定。

   現在準備好在 Service Fabric 應用程式中建置及封裝此容器。 在您的電腦上建置完容器映像之後，您便可以將它推送到任何容器登錄，然後下拉到任何主機來執行。

## <a name="run-the-containerized-application-locally"></a>在本機執行容器化的應用程式

按 **F5** 在本機 Service Fabric 開發叢集中執行容器中的應用程式並進行偵錯。 如果出現訊息方塊，詢問您是否要將 Visual Studio 專案目錄的讀取和執行權限授與 'ServiceFabricAllowedUsers' 群組，請按一下 [是]。

如果 F5 run 擲回例外狀況（如下所示），則表示正確的 IP 尚未新增至 Azure 資料庫防火牆。

```text
System.Data.SqlClient.SqlException
HResult=0x80131904
Message=Cannot open server 'fab-fiber-751718376' requested by the login. Client with IP address '123.456.789.012' is not allowed to access the server.  To enable access, use the Windows Azure Management Portal or run sp_set_firewall_rule on the master database to create a firewall rule for this IP address or address range.  It may take up to five minutes for this change to take effect.
Source=.Net SqlClient Data Provider
StackTrace:
<Cannot evaluate the exception stack trace>
```

若要將適當的 IP 新增至 Azure 資料庫防火牆，請執行下列命令。

```powershell
# The IP address of your development computer that accesses the SQL DB.
$clientIPNew = "<client IP from the Error Message>"

# Create the firewall rule to allow your development computer to access the server.
New-AzSqlServerFirewallRule -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -FirewallRuleName "AllowClientNew" -StartIpAddress $clientIPNew -EndIpAddress $clientIPNew
```

## <a name="create-a-container-registry"></a>建立容器登錄庫

現在應用程式是在本機執行，可以開始準備部署至 Azure。  容器映像需要存放在容器登錄中。  使用下列指令碼建立 [Azure 容器登錄](../container-registry/container-registry-intro.md)。 其他 Azure 訂用帳戶會看到容器登錄名稱，因此此名稱必須是唯一的。
將應用程式部署到 Azure 前，需將容器映像推送至此登錄。  當應用程式部署到 Azure 中的叢集時，會從此登錄提取容器映像。

```powershell
# Variables
$acrresourcegroupname = "fabrikam-acr-group"
$location = "southcentralus"
$registryname="fabrikamregistry$(Get-Random)"

New-AzResourceGroup -Name $acrresourcegroupname -Location $location

$registry = New-AzContainerRegistry -ResourceGroupName $acrresourcegroupname -Name $registryname -EnableAdminUser -Sku Basic
```

## <a name="create-a-service-fabric-cluster-on-azure"></a>在 Azure 中建立 Service Fabric 叢集

Service Fabric 應用程式執行於叢集，也就是一組連接網路的虛擬或實體機器。  您需要先在 Azure 中建立 Service Fabric 叢集，才能將應用程式部署至 Azure。

您可以：

* 從 Visual Studio 建立測試叢集。 此選項可讓您使用慣用的組態，直接從 Visual Studio 建立安全的叢集。
* [從範本建立安全叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)

本教學課程是從 Visual Studio建立叢集，這非常適合用於測試案例。 如果您用其他方式建立叢集或使用現有叢集，可以複製和貼上您的連線端點，或從訂用帳戶中選擇它。

開始之前，請開啟 [FabrikamFiber] > PackageRoot] > ServiceManifest.xml 的方案總管。 記下 [端點] 中列出的 Web 前端連接埠。

建立叢集時：

1. 以滑鼠右鍵按一下 [方案總管] 中的 [FabrikamFiber.CallCenterApplication] 應用程式專案，然後選擇 [發佈]。
2. 使用您的 Azure 帳戶登入，以便存取您的訂用帳戶。
3. 在 [連線端點] 下拉式清單下方，選取 [建立新叢集...] 選項。
4. 在 [建立叢集] 對話方塊中，修改下列設定：

    a. 在 [叢集名稱] 欄位中指定叢集的名稱，以及您想要使用的訂用帳戶和位置。 記下叢集資源群組的名稱。

    b. 選擇性：您可以修改節點數目。 根據預設，您有三個節點，這是測試 Service Fabric 案例所需的最少節點數。

    c. 選取 [憑證] 索引標籤。在此索引標籤中，輸入要用來保護叢集憑證的密碼。 此憑證可協助保護您的叢集。 您也可以修改您要儲存憑證的路徑。 Visual Studio 也可以為您匯入憑證，因為這是要將應用程式發佈至叢集所需的項目。

    >[!NOTE]
    >請記住匯入此憑證的資料夾路徑。 叢集建立之後的下一個步驟是匯入此憑證。

    d. 選取 [VM 詳細資料] 索引標籤。指定您想用於組成叢集之虛擬機器 (VM) 的密碼。 使用者名稱和密碼可用來從遠端連線到 VM。 您也必須選取 VM 機器大小，並可視需要變更 VM 映像。

    > [!IMPORTANT]
    > 選擇支援執行容器的 SKU。 在叢集節點上的 Windows Server 作業系統必須相容於您容器的 Windows Server 作業系統。 若要深入了解，請參閱 [Windows Server 容器作業系統和主機作業系統的相容性](service-fabric-get-started-containers.md#windows-server-container-os-and-host-os-compatibility)。 根據預設，本教學課程會建立以 Windows Server 2016 LTSC 為基礎的 Docker 映像。 以此映像為基礎的容器將會在叢集上執行，而叢集會透過具有容器的 Windows Server 2016 Datacenter 來建立。 不過，如果您建立叢集或使用以不同版本的 Windows Server 為基礎的現有叢集，則必須變更容器所依據的 OS 映像。 開啟 **FabrikamFiber.Web** 專案中的 **dockerfile**，根據舊版 Windows Server 將任何現有的 `FROM` 陳述式註解化，然後根據 [Windows Server Core DockerHub](https://hub.docker.com/_/microsoft-windows-servercore) 頁面中的所需版本標籤來新增 `FROM` 陳述式。 如需有關 Windows Server Core 版本、支援時程表和版本設定的詳細資訊，請參閱 [Windows Server Core 版本資訊](/windows-server/get-started/windows-server-release-info)。 

    e. 在 [進階] 索引標籤中，列出應用程式連接埠，這是叢集部署時要在負載平衡器中開啟的連接埠。 這是您在開始建立叢集之前所記下的連接埠。 您也可以新增現有的 Application Insights 金鑰，此金鑰會用於路由傳送應用程式記錄檔。

    f. 當您完成設定修改時，請選取 [建立] 按鈕。

5. 建立作業需要幾分鐘才能完成；輸出視窗會指出叢集何時建立完成。

## <a name="install-the-imported-certificate"></a>安裝匯入的憑證

將匯入的憑證作為叢集建立步驟的一部分，安裝到 **目前的使用者** 存放區位置，並提供您提供的私密金鑰密碼。

您可以從 [控制台] 開啟 [**管理使用者憑證**]，並確認憑證是安裝在 [**憑證-目前的使用者**  ->  **個人**  ->  **憑證**] 下，以確認安裝。 憑證應該類似 *[Cluster Name]*。*[Cluster Location]*. cloudapp.azure.com，例如 *fabrikamfibercallcenter.southcentralus.cloudapp.azure.com*。 

## <a name="allow-your-application-running-in-azure-to-access-sql-database"></a>允許在 Azure 中執行的應用程式存取 SQL Database

您先前已建立 SQL 防火牆規則，讓在本機執行的應用程式有存取權。  接著，您需要讓在 Azure 中執行的應用程式存取 SQL 資料庫。  為 Service Fabric 叢集建立[虛擬網路服務端點](../azure-sql/database/vnet-service-endpoint-rule-overview.md)，然後再建立規則以允許該端點存取 SQL 資料庫。 請務必指定建立叢集時，您所記下的叢集資源群組變數。

```powershell
# Create a virtual network service endpoint
$clusterresourcegroup = "<cluster resource group>"
$resource = Get-AzResource -ResourceGroupName $clusterresourcegroup -ResourceType Microsoft.Network/virtualNetworks | Select-Object -first 1
$vnetName = $resource.Name

Write-Host 'Virtual network name: ' $vnetName

# Get the virtual network by name.
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName $clusterresourcegroup `
  -Name              $vnetName

Write-Host "Get the subnet in the virtual network:"

# Get the subnet, assume the first subnet contains the Service Fabric cluster.
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet | Select-Object -first 1

$subnetName = $subnet.Name
$subnetID = $subnet.Id
$addressPrefix = $subnet.AddressPrefix

Write-Host "Subnet name: " $subnetName " Address prefix: " $addressPrefix " ID: " $subnetID

# Assign a Virtual Service endpoint 'Microsoft.Sql' to the subnet.
$vnet = Set-AzVirtualNetworkSubnetConfig `
  -Name            $subnetName `
  -AddressPrefix   $addressPrefix `
  -VirtualNetwork  $vnet `
  -ServiceEndpoint Microsoft.Sql | Set-AzVirtualNetwork

$vnet.Subnets[0].ServiceEndpoints;  # Display the first endpoint.

# Add a SQL DB firewall rule for the virtual network service endpoint
$subnet = Get-AzVirtualNetworkSubnetConfig `
  -Name           $subnetName `
  -VirtualNetwork $vnet;

$VNetRuleName="ServiceFabricClusterVNetRule"
$vnetRuleObject1 = New-AzSqlServerVirtualNetworkRule `
  -ResourceGroupName      $dbresourcegroupname `
  -ServerName             $servername `
  -VirtualNetworkRuleName $VNetRuleName `
  -VirtualNetworkSubnetId $subnetID;
```

## <a name="deploy-the-application-to-azure"></a>將應用程式部署至 Azure

現在應用程式已備妥，您可以直接從 Visual Studio 將其部署到 Azure 中的叢集。  以滑鼠右鍵按一下 [方案總管] 中的 [FabrikamFiber.CallCenterApplication] 應用程式專案，然後選擇 [發佈]。  在 [連線端點] 中選取您先前建立的叢集端點。  在 [Azure 容器登錄] 中，選取您先前建立的容器登錄。  按一下 [發佈] 將應用程式發佈至 Azure 中的叢集。

![發佈應用程式][publish-app]

輸出視窗會顯示部署進度。 部署應用程式後，開啟瀏覽器並鍵入叢集位址和應用程式連接埠。 例如： `http://fabrikamfibercallcenter.southcentralus.cloudapp.azure.com:8659/` 。

![Fabrikam Fiber CallCenter 應用程式首頁 (執行於 azure.com 上) 的螢幕擷取畫面。 此頁面會顯示儀表板，其中包含支援電話的清單。][fabrikam-web-page-deployed]

如果頁面無法載入，或無法提示輸入憑證，請嘗試開啟 Explorer 路徑， `https://fabrikamfibercallcenter.southcentralus.cloudapp.azure.com:19080/Explorer` 然後選取新安裝的憑證。

## <a name="set-up-continuous-integration-and-deployment-cicd-with-a-service-fabric-cluster"></a>設定 Service Fabric 叢集的持續整合和部署 (CI/CD)

若要了解如何使用 Azure DevOps 設定對 Service Fabric 叢集的 CI/CD 應用程式部署，請參閱[教學課程：透過 CI/CD 將應用程式部署至 Service Fabric 叢集](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)。 該教學課程中說明的程序與此 (FabrikamFiber) 專案的相同，只要略過下載投票範例的步驟，並將 FabrikamFiber 取代為存放庫名稱 (而不是「投票」) 即可。

## <a name="clean-up-resources"></a>清除資源

完成時，請務必移除您建立的所有資源。  最簡單的方式是移除資源群組，包括 Service Fabric 叢集、Azure SQL 資料庫、Azure 容器登錄。

```powershell
$dbresourcegroupname = "fabrikam-fiber-db-group"
$acrresourcegroupname = "fabrikam-acr-group"
$clusterresourcegroupname="fabrikamcallcentergroup"

# Remove the Azure SQL DB
Remove-AzResourceGroup -Name $dbresourcegroupname

# Remove the container registry
Remove-AzResourceGroup -Name $acrresourcegroupname

# Remove the Service Fabric cluster
Remove-AzResourceGroup -Name $clusterresourcegroupname
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
>
> * 使用 Visual Studio 將現有應用程式容器化
> * 在 Azure SQL Database 中建立資料庫
> * 建立 Azure 容器登錄
> * 將 Service Fabric 應用程式部署至 Azure

在本教學課程的下一個部分，了解如何[將具有 CI/CD 的容器應用程式部署到 Service Fabric 叢集](service-fabric-tutorial-deploy-container-app-with-cicd-vsts.md)。

[link-fabrikam-github]: https://aka.ms/fabrikamcontainer
[link-azure-powershell-install]: /powershell/azure/install-Az-ps
[link-servicefabric-create-secure-clusters]: service-fabric-cluster-creation-via-arm.md
[link-visualstudio-cd-extension]: https://aka.ms/cd4vs
[link-servicefabric-containers]: service-fabric-get-started-containers.md
[link-azure-portal]: https://portal.azure.com
[link-sf-clustertemplate]: https://aka.ms/securepreviewonelineclustertemplate
[link-azure-pricing-calculator]: https://azure.microsoft.com/pricing/calculator/
[link-azure-subscription]: https://azure.microsoft.com/free/
[link-vsts-account]: https://www.visualstudio.com/team-services/pricing/
[link-azure-sql]: /azure/sql-database/

[fabrikam-web-page]: media/service-fabric-host-app-in-a-container/fabrikam-web-page.png
[fabrikam-web-page-deployed]: media/service-fabric-host-app-in-a-container/fabrikam-web-page-deployed.png
[publish-app]: media/service-fabric-host-app-in-a-container/publish-app.png