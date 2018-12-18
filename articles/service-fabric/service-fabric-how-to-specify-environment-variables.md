---
title: 如何在 Azure Service Fabric 中指定服務的環境變數 | Microsoft Docs
description: 示範如何在 Service Fabric 中使用應用程式的環境變數
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
ms.openlocfilehash: 4325b3acd3cbc73ee5976021bebe96c267b2a6dd
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34206023"
---
# <a name="how-to-specify-environment-variables-for-services-in-service-fabric"></a>如何在 Service Fabric 中指定服務的環境變數

本文會示範如何在 Service Fabric 中指定服務或容器的環境變數。

## <a name="procedure-for-specifying-environment-variables-for-services"></a>指定服務環境變數的程序

在此範例中，您可以設定容器的環境變數。 本文假設您已有應用程式和服務資訊清單。

1. 開啟 ServiceManifest.xml 檔案。
1. 在 `CodePackage` 元素中，為每個環境變數新增新的 `EnvironmentVariables` 元素和 `EnvironmentVariable` 元素。

    ```xml
      <CodePackage Name="MyCode" Version="CodeVersion1">
      ...
        <EnvironmentVariables>
          <EnvironmentVariable Name="MyEnvVariable" Value="DeafultValue"/>
          <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
        </EnvironmentVariables>
      </CodePackage>
    ```

    在應用程式資訊清單中可以覆寫環境變數。

1. 若要在應用程式資訊清單中覆寫環境變數，請使用 `EnvironmentOverrides` 元素。

    ```xml
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="FrontEndServicePkg" ServiceManifestVersion="1.0.0" />
        <EnvironmentOverrides CodePackageRef="MyCode">
          <EnvironmentVariable Name="MyEnvVariable" Value="OverrideValue"/>
        </EnvironmentOverrides>
      </ServiceManifestImport>
    ```

## <a name="next-steps"></a>後續步驟
若要深入了解這篇文章所討論的某些核心概念，請參閱[管理多個環境發行項的應用程式](service-fabric-manage-multiple-environment-app-configuration.md)。

如需 Visual Studio 中其他可用的應用程式管理功能的相關資訊，請參閱 [在 Visual Studio 中管理 Service Fabric 應用程式](service-fabric-manage-application-in-visual-studio.md)。