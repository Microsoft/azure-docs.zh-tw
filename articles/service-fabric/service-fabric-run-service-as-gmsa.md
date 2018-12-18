---
title: 在 gMSA 帳戶之下執行 Azure Service Fabric 服務 | Microsoft Docs
description: 了解如何以 gMSA 身分在 Service Fabric Windows 標準叢集上執行 Service Fabric。
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: ''
ms.assetid: 4242a1eb-a237-459b-afbf-1e06cfa72732
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/29/2018
ms.author: mfussell
ms.openlocfilehash: 5f93285061708172b9b6ac40dc97fce08f7b2a86
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
---
# <a name="run-a-service-as-a-group-managed-service-account"></a>以群組受控服務帳戶身分執行服務
在 Windows Server 獨立叢集上，您可以使用 RunAs 原則，以群組受控服務帳戶 (gMSA) 身分執行服務。  根據預設，Service Fabric 應用程式會在用以執行 Fabric.exe 程序的帳戶之下執行。 即使在共用主控環境中，以不同帳戶執行應用程式能避免彼此干擾。 請注意，這會使用網域內部部署的 Active Directory，不是 Azure Active Directory (Azure AD)。 使用 gMSA，就不需將密碼或加密的密碼儲存於應用程式資訊清單中。  您也可以利用 [Active Directory 使用者或群組](service-fabric-run-service-as-ad-user-or-group.md)身分執行服務。

下列範例示範如何建立名為 *svc-Test$* 的 gMSA 帳戶；如何將受控服務帳戶部署至叢集節點；以及如何設定使用者主體。

先決條件：
- 網域需要一個 KDS 根金鑰。
- 網域必須位於 Windows Server 2012 或更新版本的功能層級上。

1. 讓 Active Directory 網域系統管理員能夠使用 `New-ADServiceAccount` 指令程式來建立群組受控服務帳戶，並確定 `PrincipalsAllowedToRetrieveManagedPassword` 包括所有 Service Fabric 叢集節點。 `AccountName`、`DnsHostName` 和 `ServicePrincipalName` 必須是唯一的。

    ```poweshell
    New-ADServiceAccount -name svc-Test$ -DnsHostName svc-test.contoso.com  -ServicePrincipalNames http/svc-test.contoso.com -PrincipalsAllowedToRetrieveManagedPassword SfNode0$,SfNode1$,SfNode2$,SfNode3$,SfNode4$
    ```

2. 在每個 Service Fabric 叢集節點 (例如 `SfNode0$,SfNode1$,SfNode2$,SfNode3$,SfNode4$`) 上，安裝並測試 gMSA。
    
    ```powershell
    Add-WindowsFeature RSAT-AD-PowerShell
    Install-AdServiceAccount svc-Test$
    Test-AdServiceAccount svc-Test$
    ```

3. 設定使用者主體，並設定 RunAsPolicy 來參考使用者。
    
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyApplicationType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
        <ServiceManifestImport>
          <ServiceManifestRef ServiceManifestName="MyServiceTypePkg" ServiceManifestVersion="1.0.0" />
          <ConfigOverrides />
          <Policies>
              <RunAsPolicy CodePackageRef="Code" UserRef="DomaingMSA"/>
          </Policies>
        </ServiceManifestImport>
      <Principals>
        <Users>
          <User Name="DomaingMSA" AccountType="ManagedServiceAccount" AccountName="domain\svc-Test$"/>
        </Users>
      </Principals>
    </ApplicationManifest>
    ```

> [!NOTE] 
> 如果您將 RunAs 原則套用到服務，而服務資訊清單宣告具有 HTTP 通訊協定的端點資源，您就必須指定 **SecurityAccessPolicy**。  如需詳細資訊，請參閱[為 HTTP 和 HTTPS 端點指派安全性存取原則](service-fabric-assign-policy-to-endpoint.md)。 
>

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
若要了解下一個步驟，請閱讀下列文章：
* [了解應用程式模型](service-fabric-application-model.md)
* [在服務資訊清單中指定資源](service-fabric-service-manifest-resources.md)
* [部署應用程式](service-fabric-deploy-remove-applications.md)

[image1]: ./media/service-fabric-application-runas-security/copy-to-output.png
