---
title: Azure Service Fabric 容器應用程式資訊清單範例 | Microsoft Docs
description: 了解如何設定 Service Fabric 應用程式的應用程式和服務資訊清單設定。
services: service-fabric
documentationcenter: na
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: xml
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 06/11/2018
ms.author: ryanwi
ms.openlocfilehash: 8336f0a63f74aa0db176adbb3baf462c903ba2ec
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47095574"
---
# <a name="service-fabric-application-and-service-manifest-examples"></a>Service Fabric 應用程式和服務資訊清單範例
本節包含應用程式和服務資訊清單的範例。 這些範例並非用來說明重要案例，而是要說明可用的不同設定及其使用方式。 

以下是所示功能的索引，以及其所屬的資訊清單範例。

|功能|資訊清單|
|---|---|
|[資源管理](service-fabric-resource-governance.md)|[Reliable Services 應用程式資訊清單](service-fabric-manifest-example-reliable-services-app.md#application-manifest)、[容器應用程式資訊清單](service-fabric-manifest-example-container-app.md#application-manifest)|
|[以本機系統管理員帳戶身分執行服務](service-fabric-application-runas-security.md)|[Reliable Services 應用程式資訊清單](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[將預設原則套用到所有的服務程式碼套件](service-fabric-application-runas-security.md#apply-a-default-policy-to-all-service-code-packages)|[Reliable Services 應用程式資訊清單](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[建立使用者和群組主體](service-fabric-application-runas-security.md)|[Reliable Services 應用程式資訊清單](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|在服務執行個體之間共用資料套件|[Reliable Services 應用程式資訊清單](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[覆寫服務端點](service-fabric-service-manifest-resources.md#overriding-endpoints-in-servicemanifestxml)|[Reliable Services 應用程式資訊清單](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[在服務啟動時執行指令碼](service-fabric-run-script-at-service-startup.md)|[VotingWeb 服務資訊清單](service-fabric-manifest-example-reliable-services-app.md#votingweb-service-manifest)|
|[定義 HTTPS 端點](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#define-an-https-endpoint-in-the-service-manifest)|[VotingWeb 服務資訊清單](service-fabric-manifest-example-reliable-services-app.md#votingweb-service-manifest)|
|[宣告設定套件](service-fabric-application-and-service-manifests.md)|[VotingData 服務資訊清單](service-fabric-manifest-example-reliable-services-app.md#votingdata-service-manifest)|
|[宣告資料套件](service-fabric-application-and-service-manifests.md)|[VotingData 服務資訊清單](service-fabric-manifest-example-reliable-services-app.md#votingdata-service-manifest)|
|[覆寫環境變數](service-fabric-get-started-containers.md#configure-and-set-environment-variables)|[容器應用程式資訊清單](service-fabric-manifest-example-container-app.md#application-manifest)|
|[設定容器連接埠至主機的對應](service-fabric-get-started-containers.md#configure-container-port-to-host-port-mapping-and-container-to-container-discovery)| [容器應用程式資訊清單](service-fabric-manifest-example-container-app.md#application-manifest)|
|[設定容器登錄驗證](service-fabric-get-started-containers.md#configure-container-registry-authentication)|[容器應用程式資訊清單](service-fabric-manifest-example-container-app.md#application-manifest)|
|[設定隔離模式](service-fabric-get-started-containers.md#configure-isolation-mode)|[容器應用程式資訊清單](service-fabric-manifest-example-container-app.md#application-manifest)|
|[指定作業系統組建專屬的容器映像](service-fabric-get-started-containers.md#specify-os-build-specific-container-images)|[容器應用程式資訊清單](service-fabric-manifest-example-container-app.md#application-manifest)|
|[設定環境變數](service-fabric-get-started-containers.md#configure-and-set-environment-variables)|[容器 FrontEndService 服務資訊清單](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest)、[容器 BackEndService 服務資訊清單](service-fabric-manifest-example-container-app.md#backendservice-service-manifest)|
|[設定端點](service-fabric-get-started-containers.md#configure-communication)|[容器 FrontEndService 服務資訊清單](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest)、[容器 BackEndService 服務資訊清單](service-fabric-manifest-example-container-app.md#backendservice-service-manifest)、[VotingData 服務資訊清單](service-fabric-manifest-example-reliable-services-app.md#votingdata-service-manifest)|
|將命令傳遞至容器|[容器 FrontEndService 服務資訊清單](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest)|
|[將憑證匯入容器](service-fabric-securing-containers.md)|[容器 FrontEndService 服務資訊清單](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest)|
|[設定磁碟區驅動程式](service-fabric-containers-volume-logging-drivers.md)|[容器 BackEndService 服務資訊清單](service-fabric-manifest-example-container-app.md#backendservice-service-manifest)|

