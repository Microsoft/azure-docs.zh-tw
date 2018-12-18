---
title: 如何在 Azure Dev Spaces 中使用自訂 NuGet 摘要 | Microsoft Docs
titleSuffix: Azure Dev Spaces
services: azure-dev-spaces
ms.service: azure-dev-spaces
ms.component: azds-kubernetes
author: johnsta
ms.author: johnsta
ms.date: 05/11/2018
ms.topic: article
description: 在 Azure Dev Spaces 中使用自訂 NuGet 摘要以存取及使用 NuGet 套件。
keywords: Docker, Kubernetes, Azure, AKS, Azure Container Service, 容器
manager: ghogen
ms.openlocfilehash: 04b89f6d12c58e2f4915a84d3e0a7988d0e3192f
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45579187"
---
#  <a name="use-a-custom-nuget-feed-in-an-azure-dev-space"></a>在 Azure Dev Spaces 中使用自訂 NuGet 摘要

NuGet 摘要會提供在專案中包含套件來源的便利方式。 Azure Dev Spaces 必須能夠存取此摘要，才能在 Docker 容器中正確地安裝相依性。

## <a name="set-up-a-nuget-feed"></a>設定 NuGet 摘要

若要設定 NuGet 摘要：
1. 在 `PackageReference` 節點底下的 `*.csproj` 檔案中新增[套件參考](https://docs.microsoft.com/nuget/consume-packages/package-references-in-project-files)。

   ```xml
   <ItemGroup>
       <!-- ... -->
       <PackageReference Include="Contoso.Utility.UsefulStuff" Version="3.6.0" />
       <!-- ... -->
   </ItemGroup>
   ```

2. 在專案資料夾中建立 [NuGet.Config](https://docs.microsoft.com/nuget/reference/nuget-config-file) 檔案。
     * 使用 `packageSources` 區段以參考 NuGet 摘要位置。 重要事項：NuGet 摘要必須可以公開存取。
     * 使用 `packageSourceCredentials` 區段來設定使用者名稱和密碼認證。 

   ```xml
   <packageSources>
       <add key="Contoso" value="https://contoso.com/packages/" />
   </packageSources>

   <packageSourceCredentials>
       <Contoso>
           <add key="Username" value="user@contoso.com" />
           <add key="ClearTextPassword" value="33f!!lloppa" />
       </Contoso>
   </packageSourceCredentials>
   ```

3. 如果您使用原始程式碼控制：
    - 請在您的 `.gitignore` 檔案中參考 `NuGet.Config`，您就不會不小心將認證認可到來源存放庫。
    - 開啟專案中的 `azds.yaml` 檔案，並尋找 `build` 區段，然後插入下列程式碼片段，以確保 `NuGet.Config` 檔案會同步至 Azure，以便在容器映像建置程序期間使用。 (根據預設，Azure Dev Spaces 不會同步處理符合 `.gitignore` 和 `.dockerignore` 規則的檔案。)

        ```yaml
        build:
        useGitIgnore: true
        ignore:
        - “!NuGet.Config”
        ```


## <a name="next-steps"></a>後續步驟

完成上述步驟之後，下一次您執行 `azds up` (或者在 VSCode 或 Visual Studio 中點擊 `F5`) 時，Azure Dev Spaces 會將 `NuGet.Config` 檔案同步到 Azure，然後 `dotnet restore` 會利用該檔案以在容器中安裝套件相依性。

