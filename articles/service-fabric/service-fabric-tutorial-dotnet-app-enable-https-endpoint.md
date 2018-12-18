---
title: 使用 Kestrel 將 HTTPS 端點新增至 Azure 中的 Service Fabric 應用程式 | Microsoft Docs
description: 在本教學課程中，您將了解如何使用 Kestrel 將 HTTPS 端點新增至 ASP.NET Core 前端 Web 服務，以及將應用程式部署到叢集。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/12/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 4333a234efe96f32541254819c9c5f21bb031757
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49115071"
---
# <a name="tutorial-add-an-https-endpoint-to-an-aspnet-core-web-api-front-end-service-using-kestrel"></a>教學課程：使用 Kestrel 將 HTTPS 端點新增至 ASP.NET Core Web API 前端服務

本教學課程是一個系列課程的第三部分。  您將了解如何在 Service Fabric 上執行的 ASP.NET Core 服務中啟用 HTTPS。 當您完成時，您會有一個投票應用程式，其啟用 HTTPS 的 ASP.NET Core Web 前端會在連接埠 443 上接聽。 如果您不想要在[建置 .NET Service Fabric 應用程式](service-fabric-tutorial-deploy-app-to-party-cluster.md)中手動建立投票應用程式，可以針對已完成的應用程式[下載原始程式碼](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/)。

在系列的第三部分中，您將了解如何：

> [!div class="checklist"]
> * 在服務中定義 HTTPS 端點
> * 將 Kestrel 設定為使用 HTTPS
> * 在遠端叢集節點上安裝 SSL 憑證
> * 將憑證的私密金鑰存取權給予網路服務
> * 在 Azure Load Balancer 中開啟連接埠 443
> * 將應用程式部署到遠端叢集

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [建置 .NET Service Fabric 應用程式](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * [將應用程式部署到遠端叢集](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * 將 HTTPS 端點新增至 ASP.NET Core 前端服務
> * [使用 Azure Pipelines 設定 CI/CD](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * [設定應用程式的監視和診斷](service-fabric-tutorial-monitoring-aspnet.md)

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [安裝 Visual Studio 2017](https://www.visualstudio.com/) 15.5 版或更新版本，其中包含 **Azure 開發**及 **ASP.NET 和 Web 開發**工作負載。
* [安裝 Service Fabric SDK](service-fabric-get-started.md)

## <a name="obtain-a-certificate-or-create-a-self-signed-development-certificate"></a>取得憑證或建立自我簽署的開發憑證

對於生產應用程式，使用[憑證授權單位 (CA)](https://wikipedia.org/wiki/Certificate_authority) 提供的憑證。 基於開發和測試目的，您可以建立和使用自我簽署憑證。 Service Fabric SDK 會提供 CertSetup.ps1指令碼，該指令碼可建立自我簽署的憑證並將它匯入到 `Cert:\LocalMachine\My` 憑證存放區中。 以系統管理員身分開啟命令提示字元，然後執行下列命令來建立主體為 "CN=localhost" 的憑證：

```powershell
PS C:\program files\microsoft sdks\service fabric\clustersetup\secure> .\CertSetup.ps1 -Install -CertSubjectName CN=localhost
```

如果您已經有憑證 PFX 檔案，請執行下列命令，將憑證匯入 `Cert:\LocalMachine\My` 憑證存放區：

```powershell

PS C:\mycertificates> Import-PfxCertificate -FilePath .\mysslcertificate.pfx -CertStoreLocation Cert:\LocalMachine\My -Password (ConvertTo-SecureString "!Passw0rd321" -AsPlainText -Force)


   PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\My

Thumbprint                                Subject
----------                                -------
3B138D84C077C292579BA35E4410634E164075CD  CN=zwin7fh14scd.westus.cloudapp.azure.com
```

## <a name="define-an-https-endpoint-in-the-service-manifest"></a>在服務資訊清單中定義 HTTPS 端點

以**系統管理員**身分啟動 Visual Studio，並開啟投票解決方案。 在 [方案總管] 中，開啟 VotingWeb/PackageRoot/ServiceManifest.xml。 服務資訊清單會定義服務端點。  尋找 **Endpoints** 區段並編輯現有的 "ServiceEndpoint" 端點。  將名稱變更為 "EndpointHttps"、將通訊協定設定為 https，將類型設定為 Input，以即將連接埠設定為 443。  儲存您的變更。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="configure-kestrel-to-use-https"></a>將 Kestrel 設定為使用 HTTPS

在 [方案總管] 中開啟 VotingWeb/VotingWeb.cs 檔案。  將 Kestrel 設定為使用 HTTPS，並且在 `Cert:\LocalMachine\My` 存放區中查閱憑證。 加入下列 using 陳述式：

```csharp
using System.Net;
using Microsoft.Extensions.Configuration;
using System.Security.Cryptography.X509Certificates;
```

更新 `ServiceInstanceListener` 以使用新的 EndpointHttps 端點，並且在通訊埠 443 上接聽。 將 Web 主機設定為使用 Kestrel 伺服器時，您必須設定 Kestrel 以接聽所有網路介面上的 IPv6 位址：`opt.Listen(IPAddress.IPv6Any, port, listenOptions => {...}`。

```csharp
new ServiceInstanceListener(
serviceContext =>
    new KestrelCommunicationListener(
        serviceContext,
        "EndpointHttps",
        (url, listener) =>
        {
            ServiceEventSource.Current.ServiceMessage(serviceContext, $"Starting Kestrel on {url}");

            return new WebHostBuilder()
                .UseKestrel(opt =>
                {
                    int port = serviceContext.CodePackageActivationContext.GetEndpoint("EndpointHttps").Port;
                    opt.Listen(IPAddress.IPv6Any, port, listenOptions =>
                    {
                        listenOptions.UseHttps(GetCertificateFromStore());
                        listenOptions.NoDelay = true;
                    });
                })
                .ConfigureAppConfiguration((builderContext, config) =>
                {
                    config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
                })

                .ConfigureServices(
                    services => services
                        .AddSingleton<HttpClient>(new HttpClient())
                        .AddSingleton<FabricClient>(new FabricClient())
                        .AddSingleton<StatelessServiceContext>(serviceContext))
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseStartup<Startup>()
                .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                .UseUrls(url)
                .Build();
        }))
```

此外，新增下列方法，以便 Kestrel 在 `Cert:\LocalMachine\My` 存放區中使用主體來尋找憑證。  如果您使用上一個 PowerShell 命令建立了自我簽署的憑證，則以 "localhost" 取代 "&lt;your_CN_value&gt;"，或使用您憑證的 CN。

```csharp
private X509Certificate2 GetCertificateFromStore()
{
    var store = new X509Store(StoreName.My, StoreLocation.LocalMachine);
    try
    {
        store.Open(OpenFlags.ReadOnly);
        var certCollection = store.Certificates;
        var currentCerts = certCollection.Find(X509FindType.FindBySubjectDistinguishedName, "CN=<your_CN_value>", false);
        return currentCerts.Count == 0 ? null : currentCerts[0];
    }
    finally
    {
        store.Close();
    }
}
```

## <a name="give-network-service-access-to-the-certificates-private-key"></a>將憑證的私密金鑰存取權給予網路服務

在上一個步驟中，您已將憑證匯入到開發電腦上的 `Cert:\LocalMachine\My` 存放區中。  您也必須將憑證的私密金鑰存取權明確地授與執行服務 (預設為「網路服務」) 的帳戶。 您可以手動執行 (使用 certlm.msc 工具)，但是最好在服務資訊清單的 **SetupEntryPoint** 中 [設定啟動指令碼](service-fabric-run-script-at-service-startup.md)，以便自動執行 PowerShell 指令碼。

### <a name="configure-the-service-setup-entry-point"></a>設定服務安裝程式進入點

在 [方案總管] 中，開啟 VotingWeb/PackageRoot/ServiceManifest.xml。  在 **CodePackage** 區段中，新增 **SetupEntryPoint** 節點，然後新增 **ExeHost** 節點。  在 **ExeHost** 中，將 **Program** 設定為 "Setup.bat" 以即將 **WorkingFolder** 設定為 "CodePackage"。  當 VotingWeb 服務啟動時，在 VotingWeb.exe 啟動之前，Setup.bat 指令碼會在 CodePackage 資料夾中執行。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <CodePackage Name="Code" Version="1.0.0">
    <SetupEntryPoint>
      <ExeHost>
        <Program>Setup.bat</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </SetupEntryPoint>

    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

### <a name="add-the-batch-and-powershell-setup-scripts"></a>新增批次和 PowerShell 設定指令碼

若要從 **SetupEntryPoint** 點執行 PowerShell，您可以在指向 PowerShell 檔案的批次檔中執行 PowerShell.exe。 首先，新增服務專案的批次檔案。  在 [方案總管] 中，以滑鼠右鍵按一下 **VotingWeb**，選取 [新增]-> [新項目]，然後新增名為 "Setup.bat" 的檔案。  編輯 Setup.bat 檔案並新增以下組態：

```bat
powershell.exe -ExecutionPolicy Bypass -Command ".\SetCertAccess.ps1"
```

修改 Setup.bat 檔案屬性，以將 [複製到輸出目錄] 設為 [有更新時才複製]。

![設定檔案屬性][image1]

在 [方案總管] 中，以滑鼠右鍵按一下 **VotingWeb**，選取 [新增]-> [新項目]，然後新增名為 "SetCertAccess.ps1" 的檔案。  編輯 SetCertAccess.ps1 檔案，然後新增下列指令碼：

```powershell
$subject="localhost"
$userGroup="NETWORK SERVICE"

Write-Host "Checking permissions to certificate $subject.." -ForegroundColor DarkCyan

$cert = (gci Cert:\LocalMachine\My\ | where { $_.Subject.Contains($subject) })[-1]

if ($cert -eq $null)
{
    $message="Certificate with subject:"+$subject+" does not exist at Cert:\LocalMachine\My\"
    Write-Host $message -ForegroundColor Red
    exit 1;
}elseif($cert.HasPrivateKey -eq $false){
    $message="Certificate with subject:"+$subject+" does not have a private key"
    Write-Host $message -ForegroundColor Red
    exit 1;
}else
{
    $keyName=$cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName

    $keyPath = "C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys\"
    $fullPath=$keyPath+$keyName
    $acl=(Get-Item $fullPath).GetAccessControl('Access')


    $hasPermissionsAlready = ($acl.Access | where {$_.IdentityReference.Value.Contains($userGroup.ToUpperInvariant()) -and $_.FileSystemRights -eq [System.Security.AccessControl.FileSystemRights]::FullControl}).Count -eq 1

    if ($hasPermissionsAlready){
        Write-Host "Account $userGroup already has permissions to certificate '$subject'." -ForegroundColor Green
        return $false;
    } else {
        Write-Host "Need add permissions to '$subject' certificate..." -ForegroundColor DarkYellow

        $permission=$userGroup,"Full","Allow"
        $accessRule=new-object System.Security.AccessControl.FileSystemAccessRule $permission
        $acl.AddAccessRule($accessRule)
        Set-Acl $fullPath $acl

        Write-Output "Permissions were added"

        return $true;
    }
}

```

修改 SetCertAccess.ps1 檔案屬性，以將 [複製到輸出目錄] 設為 [有更新時才複製]。

### <a name="run-the-setup-script-as-a-local-administrator"></a>以本機系統管理員身分執行設定指令碼

依預設，服務安裝程式進入點的可執行檔會在與 Service Fabric 相同的認證 (通常為 NetworkService 帳戶) 下執行。 SetCertAccess.ps1 必須有系統管理員權限。 在應用程式資訊清單中，您可以變更安全性權限，在本機系統管理員帳戶下執行啟動指令碼。

在 [方案總管] 中，開啟 Voting/ApplicationPackageRoot/ApplicationManifest.xml。 首先，建立 **Principals** 區段並新增使用者 (例如 "SetupAdminUser")。 將 SetupAdminUser 使用者帳戶新增至 Administrators 系統群組。
接下來，在 VotingWebPkg **ServiceManifestImport**區段中，設定 **RunAsPolicy** 以將 SetupAdminUser 主體套用至安裝程式進入點。 此原則會告知 Service Fabric：Setup.bat 檔案會以 SetupAdminUser (具備系統管理員權限) 身分執行。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="VotingData_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingData_PartitionCount" DefaultValue="1" />
    <Parameter Name="VotingData_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingWeb_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingDataPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="VotingData">
      <StatefulService ServiceTypeName="VotingDataType" TargetReplicaSetSize="[VotingData_TargetReplicaSetSize]" MinReplicaSetSize="[VotingData_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[VotingData_PartitionCount]" LowKey="0" HighKey="25" />
      </StatefulService>
    </Service>
    <Service Name="VotingWeb" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="VotingWebType" InstanceCount="[VotingWeb_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="SetupAdminUser">
        <MemberOf>
          <SystemGroup Name="Administrators" />
        </MemberOf>
      </User>
    </Users>
  </Principals>
</ApplicationManifest>
```

## <a name="run-the-application-locally"></a>在本機執行應用程式

在 [方案總管] 中，選取 [投票] 應用程式，並將 [應用程式 URL] 屬性設定為 " https://localhost:443 " 。

儲存所有檔案，然後按 F5 在本機執行應用程式。  在應用程式部署之後，Web 瀏覽器會開啟至 [https://localhost:443](https://localhost:443)。 如果您使用自我簽署的憑證，您會看到您的電腦不信任此網站安全性的警告。  繼續在網頁上執行。

![投票應用程式][image2]

## <a name="install-certificate-on-cluster-nodes"></a>在叢集節點上安裝憑證

將應用程式部署至 Azure 之前，將憑證安裝到遠端叢集節點的 `Cert:\LocalMachine\My` 存放區中。  當前端 Web 服務在叢集節點上啟動時，啟動指令碼會查閱憑證並設定存取權限。

首先，將憑證匯出至 PFX 檔案。 開啟 certlm.msc 應用程式，並巡覽至 **Personal**>**Certificates**。  在 localhost 憑證上按一下滑鼠右鍵，然後選取 [所有工作]>[匯出]。

![匯出憑證][image4]

在匯出精靈中，選擇 [是，匯出私密金鑰]，然後選擇個人資訊交換 (PFX) 格式。  將檔案匯出至 C:\Users\sfuser\votingappcert.pfx。

接下來，使用 [Add-AzureRmServiceFabricApplicationCertificate](/powershell/module/azurerm.servicefabric/Add-AzureRmServiceFabricApplicationCertificate) Cmdlet，在遠端叢集上安裝憑證。

> [!Warning]
> 自我簽署的憑證已足夠用於開發和測試應用程式。 對於生產應用程式，使用[憑證授權單位 (CA)](https://wikipedia.org/wiki/Certificate_authority) 提供的憑證，而非自我簽署的憑證。

```powershell
Connect-AzureRmAccount

$vaultname="sftestvault"
$certname="VotingAppPFX"
$certpw="!Password321#"
$groupname="voting_RG"
$clustername = "votinghttps"
$ExistingPfxFilePath="C:\Users\sfuser\votingappcert.pfx"

$appcertpwd = ConvertTo-SecureString -String $certpw -AsPlainText -Force

Write-Host "Reading pfx file from $ExistingPfxFilePath"
$cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2 $ExistingPfxFilePath, $certpw

$bytes = [System.IO.File]::ReadAllBytes($ExistingPfxFilePath)
$base64 = [System.Convert]::ToBase64String($bytes)

$jsonBlob = @{
   data = $base64
   dataType = 'pfx'
   password = $certpw
   } | ConvertTo-Json

$contentbytes = [System.Text.Encoding]::UTF8.GetBytes($jsonBlob)
$content = [System.Convert]::ToBase64String($contentbytes)

$secretValue = ConvertTo-SecureString -String $content -AsPlainText -Force

# Upload the certificate to the key vault as a secret
Write-Host "Writing secret to $certname in vault $vaultname"
$secret = Set-AzureKeyVaultSecret -VaultName $vaultname -Name $certname -SecretValue $secretValue

# Add a certificate to all the VMs in the cluster.
Add-AzureRmServiceFabricApplicationCertificate -ResourceGroupName $groupname -Name $clustername -SecretIdentifier $secret.Id -Verbose
```

## <a name="open-port-443-in-the-azure-load-balancer"></a>在 Azure Load Balancer 中開啟連接埠 443

在負載平衡器中開啟連接埠 443 (如果尚未開啟的話)。

```powershell
$probename = "AppPortProbe6"
$rulename="AppPortLBRule6"
$RGname="voting_RG"
$port=443

# Get the load balancer resource
$resource = Get-AzureRmResource | Where {$_.ResourceGroupName –eq $RGname -and $_.ResourceType -eq "Microsoft.Network/loadBalancers"}
$slb = Get-AzureRmLoadBalancer -Name $resource.Name -ResourceGroupName $RGname

# Add a new probe configuration to the load balancer
$slb | Add-AzureRmLoadBalancerProbeConfig -Name $probename -Protocol Tcp -Port $port -IntervalInSeconds 15 -ProbeCount 2

# Add rule configuration to the load balancer
$probe = Get-AzureRmLoadBalancerProbeConfig -Name $probename -LoadBalancer $slb
$slb | Add-AzureRmLoadBalancerRuleConfig -Name $rulename -BackendAddressPool $slb.BackendAddressPools[0] -FrontendIpConfiguration $slb.FrontendIpConfigurations[0] -Probe $probe -Protocol Tcp -FrontendPort $port -BackendPort $port

# Set the goal state for the load balancer
$slb | Set-AzureRmLoadBalancer
```

## <a name="deploy-the-application-to-azure"></a>將應用程式部署至 Azure

儲存所有檔案、從 [偵錯] 切換至 [發行]，然後按 F6 重建。  在 [方案總管] 中，以滑鼠右鍵按一下 [投票] 並選取 [發佈]。 選取在[將應用程式部署到叢集](service-fabric-tutorial-deploy-app-to-party-cluster.md)中建立之叢集的連線端點。，或選取另一個叢集。  按一下 [發佈]，將應用程式發佈至遠端叢集。

當應用程式部署時，開啟網頁瀏覽器並巡覽至 [https://mycluster.region.cloudapp.azure.com:443](https://mycluster.region.cloudapp.azure.com:443) (以您叢集的連線端點更新此 URL)。 如果您使用自我簽署的憑證，您會看到您的電腦不信任此網站安全性的警告。  繼續在網頁上執行。

![投票應用程式][image3]

## <a name="next-steps"></a>後續步驟

在教學課程的這個部分中，您已了解如何：

> [!div class="checklist"]
> * 在服務中定義 HTTPS 端點
> * 將 Kestrel 設定為使用 HTTPS
> * 在遠端叢集節點上安裝 SSL 憑證
> * 將憑證的私密金鑰存取權給予網路服務
> * 在 Azure Load Balancer 中開啟連接埠 443
> * 將應用程式部署到遠端叢集

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [使用 Azure Pipelines 設定 CI/CD](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)

[image1]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/SetupBatProperties.png
[image2]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/VotingAppLocal.png
[image3]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/VotingAppAzure.png
[image4]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/ExportCert.png
