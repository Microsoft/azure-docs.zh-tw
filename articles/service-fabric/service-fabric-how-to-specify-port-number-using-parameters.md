---
title: 如何使用參數在 Azure Service Fabric 中指定服務的連接埠號碼 | Microsoft Docs
description: 示範如何使用參數在 Service Fabric 中指定應用程式的連接埠
documentationcenter: .net
author: mikkelhegn
manager: markfuss
editor: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 12/06/2017
ms.author: mikhegn
ms.openlocfilehash: 06cfb375c6c18082a0d0316cfcb742a7779fc8a8
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34206373"
---
# <a name="how-to-specify-the-port-number-of-a-service-using-parameters-in-service-fabric"></a>如何在 Service Fabric 中使用參數指定服務的連接埠號碼

本文會示範如何使用 Visual Studio，在 Service Fabric 中使用參數指定服務的連接埠號碼。

## <a name="procedure-for-specifying-the-port-number-of-a-service-using-parameters"></a>使用參數指定服務的連接埠號碼的程序

在此範例中，您會使用參數設定您的 ASP.NET Core Web API 的連接埠號碼。

1. 開啟 Visual Studio 並建立一個新的 Service Fabric 應用程式。
1. 選擇無狀態的 ASP.NET Core 範本。
1. 選擇 Web API。
1. 開啟 ServiceManifest.xml 檔案。
1. 請注意針對您的服務所指定的端點名稱。 預設值為 `ServiceEndpoint`。
1. 開啟 ApplicationManifest.xml 檔案
1. 在 `ServiceManifestImport` 元素中，新增新的 `RessourceOverrides` 元素與您的 ServiceManifest.xml 檔案中端點的參考。

    ```xml
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
        <ResourceOverrides>
          <Endpoints>
            <Endpoint Name="ServiceEndpoint"/>
          </Endpoints>
        </ResourceOverrides>
        <ConfigOverrides />
      </ServiceManifestImport>
    ```

1. 在 `Endpoint` 元素中，您現在可以使用參數覆寫任何屬性。 在此範例中，您可以指定 `Port`，並使用方括號將它設為參數名稱 - 例如，`[MyWebAPI_PortNumber]`

    ```xml
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
        <ResourceOverrides>
          <Endpoints>
            <Endpoint Name="ServiceEndpoint" Port="[MyWebAPI_PortNumber]"/>
          </Endpoints>
        </ResourceOverrides>
        <ConfigOverrides />
      </ServiceManifestImport>
    ```

1. 然後仍舊在 ApplicationManifest.xml 中，指定 `Parameters` 元素中的參數

    ```xml
      <Parameters>
        <Parameter Name="MyWebAPI_PortNumber" />
      </Parameters>
    ```

1. 並定義 `DefaultValue`

    ```xml
      <Parameters>
        <Parameter Name="MyWebAPI_PortNumber" DefaultValue="8080" />
      </Parameters>
    ```

1. 開啟 ApplicationParameters 資料夾和 `Cloud.xml` 檔案
1. 若要指定發行至遠端叢集時要使用不同的通訊埠，將包含連接埠號碼的參數新增到此檔案。

    ```xml
      <Parameters>
        <Parameter Name="MyWebAPI_PortNumber" DefaultValue="80" />
      </Parameters>
    ```

當使用 Cloud.xml 發行設定檔從 Visual Studio 發行您的應用程式時，您的服務會設定為使用連接埠 80。 如果部署應用程式未指定 MyWebAPI_PortNumber 參數，則服務會使用連接埠 8080。

## <a name="next-steps"></a>後續步驟
若要深入了解這篇文章所討論的某些核心概念，請參閱[管理多個環境發行項的應用程式](service-fabric-manage-multiple-environment-app-configuration.md)。

如需 Visual Studio 中其他可用的應用程式管理功能的相關資訊，請參閱 [在 Visual Studio 中管理 Service Fabric 應用程式](service-fabric-manage-application-in-visual-studio.md)。