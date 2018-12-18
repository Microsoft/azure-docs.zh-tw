---
title: Azure Service Fabric 應用程式部署 | Microsoft Docs
description: 使用 FabricClient API 來部署和移除 Service Fabric 中的應用程式。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: b120ffbf-f1e3-4b26-a492-347c29f8f66b
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/19/2018
ms.author: ryanwi
ms.openlocfilehash: 50c487e6bad2a009b4f719d44b129ba860432e28
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34207463"
---
# <a name="deploy-and-remove-applications-using-fabricclient"></a>使用 FabricClient 來部署和移除應用程式
> [!div class="op_single_selector"]
> * [Resource Manager](service-fabric-application-arm-resource.md)
> * [PowerShell](service-fabric-deploy-remove-applications.md)
> * [Service Fabric CLI](service-fabric-application-lifecycle-sfctl.md)
> * [FabricClient API](service-fabric-deploy-remove-applications-fabricclient.md)
> 
> 

<br/>

[封裝應用程式類型][10]後，即可將它部署至 Azure Service Fabric 叢集中。 部署涉及下列三個步驟：

1. 將應用程式封裝上傳至映像存放區
2. 註冊應用程式類型
3. 移除映像存放區中的應用程式封裝
4. 建立應用程式執行個體

在應用程式中部署且個體開始在叢集中執行之後，您就可以刪除應用程式執行個體和其應用程式類型。 從叢集完全移除應用程式需要執行下列步驟︰

1. 移除 (或刪除) 執行中的應用程式執行個體
2. 取消註冊不再需要的應用程式類型

如果您在本機開發叢集上使用 Visual Studio 來針對應用程式進行部署和偵錯，則先前所有步驟都會透過 PowerShell 指令碼自動處理。  在應用程式專案的 [指令碼] 資料夾中可找到這個指令碼。 本文提供該指令碼的背景資料，讓您可以在 Visual Studio 之外執行相同的作業。 
 
## <a name="connect-to-the-cluster"></a>連接到叢集
執行本文中任何程式碼範例之前，請先建立 [FabricClient](/dotnet/api/system.fabric.fabricclient) 執行個體來連線到叢集。 針對連線至本機開發叢集或遠端叢集，或是連線至使用 Azure Active Directory、X509 憑證或 Windows Active Directory 保護的叢集，如需相關的範例，請參閱[連線至安全的叢集](service-fabric-connect-to-secure-cluster.md#connect-to-a-cluster-using-the-fabricclient-apis)。 若要連線至本機開發叢集，請執行下列命令︰

```csharp
// Connect to the local cluster.
FabricClient fabricClient = new FabricClient();
```

## <a name="upload-the-application-package"></a>上傳應用程式封裝
假設您在 Visual Studio 中組建並封裝名為 *MyApplication* 的應用程式。 根據預設，ApplicationManifest.xml 中列出的應用程式類型名稱會是 "MyApplicationType"。  應用程式封裝 (其中包含必要的應用程式資訊清單、服務資訊清單和程式碼/組態/資料封裝) 位於 *C:\Users\&lt;username&gt;\Documents\Visual Studio 2017\Projects\MyApplication\MyApplication\pkg\Debug*。

上傳應用程式套件會將它放在內部 Service Fabric 元件可以存取的位置。 Service Fabric 會在應用程式套件的註冊期間驗證應用程式套件。 不過，如果您想要在本機確認應用程式套件 (亦即，在上傳前)，使用 [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage?view=azureservicefabricps) cmdlet。

[Copy-ServiceFabricApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.copyapplicationpackage) API 會將應用程式套件上傳至叢集映像存放區。 

如果是大型的應用程式套件且/或具有許多檔案，您可以[將它壓縮](service-fabric-package-apps.md#compress-a-package)，並使用 PowerShell 將它複製到 ImageStore。 壓縮會減少檔案大小和數目。

請參閱[了解映像存放區連接字串](service-fabric-image-store-connection-string.md)，以取得有關映像存放區和映像存放區連接字串的補充資訊。

## <a name="register-the-application-package"></a>註冊應用程式封裝
註冊應用程式套件時，應用程式類型和應用程式資訊清單中宣告的版本可供使用。 系統會讀取在上一個步驟上傳的套件，請確認套件、處理套件內容，然後將處理過的套件複製至內部系統位置。  

[ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) API 會註冊叢集中的應用程式類型，並使其可供部署。

[GetApplicationTypeListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationtypelistasync) API 會提供所有註冊成功之應用程式類型的相關資訊。 您可以使用此 API 來判斷何時會完成註冊。

## <a name="remove-an-application-package-from-the-image-store"></a>從映像存放區移除應用程式封裝
建議您在成功註冊應用程式之後，移除應用程式套件。  從映像存放區刪除應用程式套件會釋放系統資源。  保留未使用的應用程式套件會耗用磁碟儲存空間，並會導致應用程式效能問題。 使用 [RemoveApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.removeapplicationpackage) API 從映像存放區刪除應用程式套件。

## <a name="create-an-application-instance"></a>建立應用程式執行個體
您可以使用 [CreateApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.createapplicationasync) API，從任何已成功註冊的應用程式類型，將應用程式具現化。 每個應用程式名稱的開頭必須為 "fabric:" 配置，而且必須是 (叢集內) 每個應用程式執行個體的唯一名稱。 如果已在目標應用程式類型的應用程式資訊清單中定義預設服務，也會一併建立這些服務。

多個應用程式執行個體可以針對任何指定的已註冊應用程式類型版本來建立。 每個應用程式執行個體將在隔離狀態下執行，包含本身的工作目錄和程序集。

若要查看哪些具名的應用程式和服務正在叢集中執行，請執行 [GetApplicationListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationlistasync) 和 [GetServiceListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getservicelistasync) API。

## <a name="create-a-service-instance"></a>建立服務執行個體
您可以使用 [CreateServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync) API，從服務類型將服務具現化。  如果服務宣告為應用程式資訊清單中的預設服務，服務會在應用程式具現化時進行具現化。  對已具現化的服務呼叫 [CreateServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync) API，會傳回 FabricException 類型的例外狀況，其中包含值為 FabricErrorCode.ServiceAlreadyExists 的錯誤碼。

## <a name="remove-a-service-instance"></a>移除服務執行個體
當不再需要服務執行個體時，可以呼叫 [DeleteServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.deleteserviceasync) API，將它從執行的應用程式執行個體中移除。  

> [!WARNING]
> 此作業無法回復，且服務狀態無法復原。

## <a name="remove-an-application-instance"></a>移除應用程式執行個體
當不再需要應用程式執行個體時，您可以使用 [DeleteApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.deleteapplicationasync) API，依名稱將它永久移除。 [DeleteApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.deleteapplicationasync) 會將屬於應用程式的所有服務自動移除，以及將所有服務狀態永久移除。

> [!WARNING]
> 此作業無法回復，且應用程式狀態無法復原。

## <a name="unregister-an-application-type"></a>取消註冊應用程式類型
不再需要特定版本的應用程式類型時，您應該使用 [Unregister-ServiceFabricApplicationType](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.unprovisionapplicationasync) API 將該特定版本的應用程式類型取消註冊。 取消註冊未使用的應用程式類型版本會釋放映像存放區已使用的儲存空間。 只要沒有針對某個版本的應用程式類型進行具現化，且沒有擱置中的應用程式升級參考該版本的應用程式類型，便可取消註冊該版本的應用程式類型。

## <a name="troubleshooting"></a>疑難排解
### <a name="copy-servicefabricapplicationpackage-asks-for-an-imagestoreconnectionstring"></a>Copy-ServiceFabricApplicationPackage 要求 ImageStoreConnectionString
Service Fabric SDK 環境應已正確設定預設值。 但若有需要，所有命令的 ImageStoreConnectionString都應符合 Service Fabric 叢集正在使用的值。 您可以在使用 [Get-ServiceFabricClusterManifest](/powershell/module/servicefabric/get-servicefabricclustermanifest?view=azureservicefabricps) 和 Get-ImageStoreConnectionStringFromClusterManifest 命令擷取的叢集資訊清單 ImageStoreConnectionString 中找到此值：

```powershell
PS C:\> Get-ImageStoreConnectionStringFromClusterManifest(Get-ServiceFabricClusterManifest)
```

**Get ImageStoreConnectionStringFromClusterManifest** Cmdlet 是 Service Fabric SDK PowerShell 模組的一部分，可用來取得映像存放區連接字串。  若要匯入 SDK 模組，請執行︰

```powershell
Import-Module "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1"
```


會在叢集資訊清單中找到 ImageStoreConnectionString：

```xml
<ClusterManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Name="Server-Default-SingleNode" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">

    [...]

    <Section Name="Management">
      <Parameter Name="ImageStoreConnectionString" Value="file:D:\ServiceFabric\Data\ImageStore" />
    </Section>

    [...]
```

請參閱[了解映像存放區連接字串](service-fabric-image-store-connection-string.md)，以取得有關映像存放區和映像存放區連接字串的補充資訊。

### <a name="deploy-large-application-package"></a>部署大型應用程式封裝
問題︰大型應用程式套件 (GB 的順序) 的 [CopyApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.copyapplicationpackage) API 逾時。
請嘗試︰
- 使用 `timeout` 參數指定 [CopyApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.copyapplicationpackage) 方法的較大逾時。 此逾時預設為 30 分鐘。
- 檢查來源電腦與叢集之間的網路連線。 如果連線速度變慢，請考慮使用更佳網路連線的電腦。
如果用戶端電腦與叢集在不同的區域中，請考慮使用與叢集接近或相同區域中的用戶端電腦。
- 請檢查是否到達外部節流。 例如，當映像存放區設定為使用 Azure 儲存體時，上傳可能受到節流控制。

問題︰上傳套件順利完成，但 [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) API 逾時。請嘗試︰
- 複製到映像存放區之前[壓縮封裝](service-fabric-package-apps.md#compress-a-package)。
壓縮會減少檔案的大小和數目，而後者則可減少資料傳輸量和 Service Fabric 必須執行的工作。 上傳作業可能會變慢 (尤其是如果您包含壓縮時間)，但註冊和取消註冊應用程式類型會比較快。
- 使用 `timeout` 參數為 [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) API 指定較大的逾時。

### <a name="deploy-application-package-with-many-files"></a>部署具有很多檔案的應用程式封裝
問題︰具有很多檔案的應用程式套件 (以千計的順序) [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) 的逾時。
請嘗試︰
- 複製到映像存放區之前[壓縮封裝](service-fabric-package-apps.md#compress-a-package)。 壓縮會減少檔案的數目。
- 使用 `timeout` 參數指定 [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) 的較大逾時。

## <a name="code-example"></a>程式碼範例
下列範例會將應用程式套件複製到映像存放區、佈建應用程式類型、建立應用程式執行個體、建立服務執行個體、移除應用程式執行個體、取消佈建應用程式類型，以及將應用程式套件從映像存放區中刪除。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Reflection;
using System.Threading.Tasks;

using System.Fabric;
using System.Fabric.Description;
using System.Threading;

namespace ServiceFabricAppLifecycle
{
class Program
{
static void Main(string[] args)
{

    string clusterConnection = "localhost:19000";
    string appName = "fabric:/MyApplication";
    string appType = "MyApplicationType";
    string appVersion = "1.0.0";
    string serviceName = "fabric:/MyApplication/Stateless1";
    string imageStoreConnectionString = "file:C:\\SfDevCluster\\Data\\ImageStoreShare";
    string packagePathInImageStore = "MyApplication";
    string packagePath = "C:\\Users\\username\\Documents\\Visual Studio 2017\\Projects\\MyApplication\\MyApplication\\pkg\\Debug";
    string serviceType = "Stateless1Type";

    // Connect to the cluster.
    FabricClient fabricClient = new FabricClient(clusterConnection);

    // Copy the application package to a location in the image store
    try
    {
        fabricClient.ApplicationManager.CopyApplicationPackage(imageStoreConnectionString, packagePath, packagePathInImageStore);
        Console.WriteLine("Application package copied to {0}", packagePathInImageStore);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Application package copy to Image Store failed: ");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Provision the application.  "MyApplicationV1" is the folder in the image store where the application package is located. 
    // The application type with name "MyApplicationType" and version "1.0.0" (both are found in the application manifest) 
    // is now registered in the cluster.            
    try
    {
        fabricClient.ApplicationManager.ProvisionApplicationAsync(packagePathInImageStore).Wait();

        Console.WriteLine("Provisioned application type {0}", packagePathInImageStore);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Provision Application Type failed:");

        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Delete the application package from a location in the image store.
    try
    {
        fabricClient.ApplicationManager.RemoveApplicationPackage(imageStoreConnectionString, packagePathInImageStore);
        Console.WriteLine("Application package removed from {0}", packagePathInImageStore);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Application package removal from Image Store failed: ");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    //  Create the application instance.
    try
    {
        ApplicationDescription appDesc = new ApplicationDescription(new Uri(appName), appType, appVersion);
        fabricClient.ApplicationManager.CreateApplicationAsync(appDesc).Wait();
        Console.WriteLine("Created application instance of type {0}, version {1}", appType, appVersion);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("CreateApplication failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Create the stateless service description.  For stateful services, use a StatefulServiceDescription object.
    StatelessServiceDescription serviceDescription = new StatelessServiceDescription();
    serviceDescription.ApplicationName = new Uri(appName);
    serviceDescription.InstanceCount = 1;
    serviceDescription.PartitionSchemeDescription = new SingletonPartitionSchemeDescription();
    serviceDescription.ServiceName = new Uri(serviceName);
    serviceDescription.ServiceTypeName = serviceType;

    // Create the service instance.  If the service is declared as a default service in the ApplicationManifest.xml,
    // the service instance is already running and this call will fail.
    try
    {
        fabricClient.ServiceManager.CreateServiceAsync(serviceDescription).Wait();
        Console.WriteLine("Created service instance {0}", serviceName);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("CreateService failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Delete a service instance.
    try
    {
        DeleteServiceDescription deleteServiceDescription = new DeleteServiceDescription(new Uri(serviceName));

        fabricClient.ServiceManager.DeleteServiceAsync(deleteServiceDescription);
        Console.WriteLine("Deleted service instance {0}", serviceName);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("DeleteService failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Delete an application instance from the application type.
    try
    {
        DeleteApplicationDescription deleteApplicationDescription = new DeleteApplicationDescription(new Uri(appName));

        fabricClient.ApplicationManager.DeleteApplicationAsync(deleteApplicationDescription).Wait();
        Console.WriteLine("Deleted application instance {0}", appName);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("DeleteApplication failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Un-provision the application type.
    try
    {
        fabricClient.ApplicationManager.UnprovisionApplicationAsync(appType, appVersion).Wait();
        Console.WriteLine("Un-provisioned application type {0}, version {1}", appType, appVersion);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Un-provision application type failed: ");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    Console.WriteLine("Hit enter...");
    Console.Read();
}        
}
}

```

## <a name="next-steps"></a>後續步驟
[Service Fabric 應用程式升級](service-fabric-application-upgrade.md)

[Service Fabric 健康狀態簡介](service-fabric-health-introduction.md)

[診斷和疑難排解 Service Fabric 服務](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[在 Service Fabric 中模型化應用程式](service-fabric-application-model.md)

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-package-apps.md
[11]: service-fabric-application-upgrade.md
