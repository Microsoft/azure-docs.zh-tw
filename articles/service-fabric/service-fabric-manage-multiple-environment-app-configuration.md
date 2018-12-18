---
title: 在 Azure Service Fabric 中管理多個環境的應用程式 | Microsoft Docs
description: Azure Service Fabric 應用程式可以在任意大小 (從一部機器至數千部機器) 的叢集上執行。 在某些情況下，您會想要針對各種環境以不同的方式設定應用程式。 本文說明如何定義每個環境的不同應用程式參數。
services: service-fabric
documentationcenter: .net
author: mikkelhegn
manager: msfussell
editor: ''
ms.assetid: f406eac9-7271-4c37-a0d3-0a2957b60537
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/23/2018
ms.author: mikhegn
ms.openlocfilehash: dac96ef6fce38a0557444e181fa6eccb649cfb9a
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44298330"
---
# <a name="manage-applications-for-multiple-environments"></a>管理多個環境的應用程式

Azure Service Fabric 叢集可讓您使用任意數量的機器 (從一部至數千部) 建立叢集。 在大部分情況下，您會發現需要在多個叢集設定中部署您的應用程式：您的本機開發叢集、共用的開發叢集和生產叢集。 所有這些叢集會視為不同的環境，您的程式碼皆必須在其中執行。 雖然不需針對各種環境進行修改，即可執行應用程式二進位檔，但您通常會以不同的方式設定應用程式。

請看兩個簡單的範例：
  - 您的服務會在定義的連接埠上接聽，但該連接埠在各環境之間必須不同
  - 您需要在不同環境間為資料庫提供不同的繫結認證

## <a name="specifying-configuration"></a>指定設定

您提供的設定可以分成兩個類別：

- 適用於服務執行方式的設定
  - 例如，端點的連接埠號碼或服務的執行個體數目
  - 此設定會在應用程式或服務資訊清單檔中指定
- 適用於應用程式程式碼的設定
  - 例如，資料庫的繫結資訊
  - 此設定可透過設定檔或環境變數來提供

> [!NOTE]
> 並非應用程式及服務資訊清單檔中的所有屬性皆支援參數。
> 在這些案例中，您必須依賴將字串取代為部署工作流程的一部分。 在 Azure DevOps 中，您可以使用「取代權杖」之類的擴充功能： https://marketplace.visualstudio.com/items?itemName=qetza.replacetokens，或可在 Jenkins 中執行指令碼工作來取代該值。
>

## <a name="specifying-parameters-during-application-creation"></a>在應用程式建立期間指定參數

在 Service Fabric 中建立具名的應用程式執行個體時，您可以傳入參數。 執行的方式取決於您如何建立應用程式執行個體。

  - 在 PowerShell 中，[`New-ServiceFabricApplication`](https://docs.microsoft.com/powershell/module/servicefabric/new-servicefabricapplication?view=azureservicefabricps) Cmdlet 會將應用程式參數作為雜湊表。
  - 使用 sfctl，[`sfctl application create`](https://docs.microsoft.com/azure/service-fabric/service-fabric-sfctl-application#sfctl-application-create) 命令會將參數作為 JSON 字串。 install.sh 指令碼會使用 sfctl。
  - Visual Studio 會在應用程式專案的 [參數] 資料夾中為您提供一組參數檔案。 從 Visual Studio 使用 Azure DevOps Services 或 Team Foundation Server 發佈時，會使用這些參數檔案。 在 Visual Studio 中，參數檔案會傳遞至 Deploy-FabricApplication.ps1 指令碼。

## <a name="next-steps"></a>後續步驟
下列文章將示範如何使用此處所述的一些概念：

- [如何在 Service Fabric 中指定服務的環境變數](service-fabric-how-to-specify-environment-variables.md)
- [如何使用 Service Fabric 中的參數指定服務的連接埠號碼](service-fabric-how-to-specify-port-number-using-parameters.md)
- [如何參數化設定檔](service-fabric-how-to-parameterize-configuration-files.md)

- [環境變數參考](service-fabric-environment-variables-reference.md)
