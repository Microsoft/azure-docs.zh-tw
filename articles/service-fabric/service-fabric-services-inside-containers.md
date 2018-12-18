---
title: 在 Windows 上將 Azure Service Fabric 服務容器化
description: 了解如何在 Windows 上將 Service Fabric Reliable Services 和 Reliable Actors 容器化。
services: service-fabric
documentationcenter: .net
author: anmolah
manager: anmolah
editor: roroutra
ms.assetid: 0b41efb3-4063-4600-89f5-b077ea81fa3a
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 5/23/2018
ms.author: anmola
ms.openlocfilehash: d3ed1ff46bf4c82a172954828ec74bae80241288
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44056693"
---
# <a name="containerize-your-service-fabric-reliable-services-and-reliable-actors-on-windows"></a>將 Windows 上的 Service Fabric Reliable Services 和 Reliable Actors 容器化

Service Fabric 支援將 Service Fabric 微服務 (Reliable Services 和 Reliable Actors 型服務) 容器化。 如需詳細資訊，請參閱 [Service Fabric 容器](service-fabric-containers-overview.md)。

本文件可指引您如何讓服務在 Windows 容器內執行。

> [!NOTE]
> 此功能目前僅適用於 Windows。 若要執行容器，叢集必須執行於具有容器的 Windows Server 2016 上。

## <a name="steps-to-containerize-your-service-fabric-application"></a>用來將 Service Fabric 應用程式容器化的步驟

1. 在 Visual Studio 中開啟 Service Fabric 應用程式。

2. 將 [SFBinaryLoader.cs](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/code/SFBinaryLoaderForContainers/SFBinaryLoader.cs) 類別新增至您的專案。 此類別的程式碼是協助程式，用以在容器內執行應用程式時，於應用程式內正確載入 Service Fabric 執行階段二進位檔。

3. 針對每個想要容器化的程式碼套件，於程式進入點初始化載入器。 將下列程式碼片段所示的靜態建構函式新增至程式的進入點檔案。

  ```csharp
  namespace MyApplication
  {
      internal static class Program
      {
          static Program()
          {
              SFBinaryLoader.Initialize();
          }

          /// <summary>
          /// This is the entry point of the service host process.
          /// </summary>
          private static void Main()
          {
  ```

4. 建置和[封裝](service-fabric-package-apps.md#Package-App)您的專案。 若要建置和建立套件，請以滑鼠右鍵按一下方案總管中的應用程式專案，然後選擇 [封裝] 命令。

5. 針對每個需要容器化的程式碼套件，執行 PowerShell 指令碼 [CreateDockerPackage.ps1](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/scripts/CodePackageToDockerPackage/CreateDockerPackage.ps1)。 使用方式如下：

    完整的 .NET
      ```powershell
        $codePackagePath = 'Path to the code package to containerize.'
        $dockerPackageOutputDirectoryPath = 'Output path for the generated docker folder.'
        $applicationExeName = 'Name of the Code package executable.'
        CreateDockerPackage.ps1 -CodePackageDirectoryPath $codePackagePath -DockerPackageOutputDirectoryPath $dockerPackageOutputDirectoryPath -ApplicationExeName $applicationExeName
      ```
    .NET Core
      ```powershell
        $codePackagePath = 'Path to the code package to containerize.'
        $dockerPackageOutputDirectoryPath = 'Output path for the generated docker folder.'
        $dotnetCoreDllName = 'Name of the Code package dotnet Core Dll.'
        CreateDockerPackage.ps1 -CodePackageDirectoryPath $codePackagePath -DockerPackageOutputDirectoryPath $dockerPackageOutputDirectoryPath -DotnetCoreDllName $dotnetCoreDllName
      ```
      指令碼會在 $dockerPackageOutputDirectoryPath 建立具有 Docker 構件的資料夾。 根據您的需求修改所產生的 Dockerfile，以 `expose` (公開) 任何連接埠、執行安裝指令碼等等。 based on your needs.

6. 接下來，您需要[建置](service-fabric-get-started-containers.md#Build-Containers) Docker 容器套件並將其[推送](service-fabric-get-started-containers.md#Push-Containers)至您的存放庫。

7. 修改 ApplicationManifest.xml 和 ServiceManifest.xml 以新增容器映像、存放庫資訊、登錄驗證和「連接埠與主機的對應」。 若要修改資訊清單，請參閱[建立 Azure Service Fabric 容器應用程式](service-fabric-get-started-containers.md)。 服務資訊清單中的程式碼套件定義需以對應的容器映像來加以取代。 請務必要將 EntryPoint 變更為 ContainerHost 類型。

  ```xml
<!-- Code package is your service executable. -->
<CodePackage Name="Code" Version="1.0.0">
  <EntryPoint>
    <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
    <ContainerHost>
      <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
    </ContainerHost>
  </EntryPoint>
  <!-- Pass environment variables to your container: -->
</CodePackage>
  ```

8. 為您的複寫器和服務端點新增「連接埠與主機的對應」。 這兩個連接埠都會在執行階段由 Service Fabric 加以指派，因此 ContainerPort 會設定為零以使用指派的連接埠來進行對應。

 ```xml
<Policies>
  <ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="0" EndpointRef="ServiceEndpoint"/>
    <PortBinding ContainerPort="0" EndpointRef="ReplicatorEndpoint"/>
  </ContainerHostPolicies>
</Policies>
 ```

9. 若要設定容器隔離模式，請參閱[設定隔離模式]( https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-get-started-containers#configure-isolation-mode)。 Windows 支援兩種容器隔離模式：分別為處理序和 Hyper-V。 下列程式碼片段顯示如何在應用程式資訊清單檔中指定隔離模式。

 ```xml
<Policies>
  <ContainerHostPolicies CodePackageRef="Code" Isolation="process">
  ...
  </ContainerHostPolicies>
</Policies>
 ```
  ```xml
<Policies>
  <ContainerHostPolicies CodePackageRef="Code" Isolation="hyperv">
  ...
  </ContainerHostPolicies>
</Policies>
 ```

10. 若要測試此應用程式，您必須將它部署至執行 5.7 版或更高版本的叢集。 若為執行階段 6.1 版或更低版本，您還需要編輯和更新叢集設定，以啟用此預覽功能。 請遵循這篇[文章](service-fabric-cluster-fabric-settings.md)中的步驟，以新增接下來顯示的設定。
```
      {
        "name": "Hosting",
        "parameters": [
          {
            "name": "FabricContainerAppsEnabled",
            "value": "true"
          }
        ]
      }
```

11. 接著，將編輯過的應用程式套件[部署](service-fabric-deploy-remove-applications.md)至此叢集。

現在，您的叢集中應該會有正在執行的容器化 Service Fabric 應用程式。

## <a name="next-steps"></a>後續步驟
* 深入了解如何[在 Service Fabric 上執行容器](service-fabric-get-started-containers.md)。
* 深入了解 Service Fabric [應用程式生命週期](service-fabric-application-lifecycle.md)。
