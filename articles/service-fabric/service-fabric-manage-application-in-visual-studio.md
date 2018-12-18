---
title: 在 Visual Studio 中管理 Azure Servic Fabric 應用程式 | Microsoft Docs
description: 使用 Visual Studio 來建立、開發、封裝、部署和偵錯 Azure Service Fabric 應用程式和服務。
services: service-fabric
documentationcenter: .net
author: mikkelhegn
manager: timlt
editor: ''
ms.assetid: c317cb7e-7eae-466e-ba41-6aa2518be5cf
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.custom: vs-azure
ms.workload: azure-vs
ms.date: 03/26/2018
ms.author: mikhegn
ms.openlocfilehash: 2e7b7d9b0a194b13de9bdf759f4f3be645ed7c2e
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42442164"
---
# <a name="use-visual-studio-to-simplify-writing-and-managing-your-service-fabric-applications"></a>使用 Visual Studio 簡化撰寫及管理 Service Fabric 應用程式
您可以透過 Visual Studio 管理 Azure Service Fabric 應用程式與服務。 [設定開發環境](service-fabric-get-started.md)後，您可以使用 Visual Studio 在本機開發叢集中建立 Service Fabric 應用程式、新增服務，或是封裝、註冊及部署應用程式。

## <a name="deploy-your-service-fabric-application"></a>部署 Service Fabric 應用程式
根據預設，部署應用程式會將以下幾個步驟結合成一個簡單的作業：

1. 建立應用程式封裝
2. 將應用程式封裝上傳至映像存放區
3. 註冊應用程式類型
4. 移除任何執行中的應用程式執行個體
5. 建立應用程式執行個體

在 Visual Studio 中按 **F5** 會部署應用程式，並將偵錯工具附加到所有應用程式執行個體。 您可以使用 **Ctrl + F5** 來部署應用程式而不進行偵錯，或是使用發佈設定檔來發佈至本機或遠端叢集。

### <a name="application-debug-mode"></a>應用程式偵錯模式
Visual Studio 提供的 **Application Debug Mode** 屬性可讓您控制 Visual Studio 要如何處理應用程式部署作為偵錯的一部分。

#### <a name="to-set-the-application-debug-mode-property"></a>設定應用程式偵錯模式屬性
1. 在 Service Fabric 應用程式專案 (*.sfproj) 的捷徑功能表上，選擇 [屬性]\(或按 **F4** 按鍵)。
2. 在 [屬性] 視窗中，設定 [應用程式偵錯模式] 屬性。

![設定應用程式偵錯模式屬性][debugmodeproperty]

#### <a name="application-debug-modes"></a>應用程式偵錯模式

1. **重新整理應用程式** 這個模式可讓您快速變更和偵錯您的程式碼，並支援在偵錯時編輯靜態網頁檔案。 這種模式僅適用於您的本機開發叢集為 [1 個節點模式] 的情況。 此為預設的應用程式偵錯模式。(/service-fabric-get-started-with-a-local-cluster.md#one-node-and-five-node-cluster-mode)。
2. **移除應用程式** 會在偵錯工作階段結束時移除應用程式。
3. **自動升級** 偵錯工作階段結束時，應用程式繼續執行。 下一個偵錯工作階段會將部署視為升級。 此升級程序會保留您在前一個偵錯工作階段中輸入的所有資料。
4. **保留應用程式** 偵錯工作階段結束時，應用程式會在叢集中繼續執行。 在下一個偵錯工作階段開始時，會移除應用程式。

如果使用 [自動升級]，資料會藉由套用 Service Fabric 的應用程式升級功能得以保留。 如需有關升級應用程式和如何在實際環境中執行升級的詳細資訊，請參閱 [Service Fabric 應用程式升級](service-fabric-application-upgrade.md)。

## <a name="add-a-service-to-your-service-fabric-application"></a>將服務加入 Service Fabric 應用程式
您可以將新的服務新增至應用程式以擴充其功能。 若要確保應用程式封裝中包含該服務，請透過 [新增 Fabric 服務]  功能表項目新增服務。

![新增 Service Fabric 服務][newservice]

選取要新增至應用程式的 Service Fabric 專案類型，並指定服務的名稱。  請參閱 [選擇服務架構](service-fabric-choose-framework.md) ，以協助您決定要使用的服務類型。

![選取要新增至應用程式的 Service Fabric 服務類型][addserviceproject]

新服務會新增到解決方案與現有的應用程式套件中。 服務參照與預設的服務執行個體將會新增到應用程式資訊清單，然後在您下次部署應用程式時建立並啟動服務。

![新服務會新增到應用程式資訊清單][newserviceapplicationmanifest]

## <a name="package-your-service-fabric-application"></a>封裝 Service Fabric 應用程式
若要將應用程式及其服務部署到叢集，您需要建立應用程式封裝。  套件會以特定的配置來組織應用程式資訊清單、服務資訊清單和其他必要的檔案。  Visual Studio 會在 'pkg' 目錄的應用程式專案資料夾中設定與管理封裝。  從 [應用程式] 操作功能表中按一下 [封裝] 會建立或更新應用程式封裝。

## <a name="remove-applications-and-application-types-using-cloud-explorer"></a>使用 Cloud Explorer 移除應用程式和應用程式類型
您可以從 Visual Studio 使用 Cloud Explorer (可從 [檢視]  功能表啟動) 執行基本的叢集管理作業。 例如，您可以在本機或遠端叢集上刪除應用程式和解除佈建應用程式類型。

![移除應用程式][removeapplication]

> [!TIP]
> 如需更豐富的叢集管理功能，請參閱 [使用 Service Fabric Explorer 視覺化叢集](service-fabric-visualizing-your-cluster.md)。
>
>

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>後續步驟
* [Service Fabric 應用程式模型](service-fabric-application-model.md)
* [Service Fabric 應用程式部署](service-fabric-deploy-remove-applications.md)
* [管理多個環境的應用程式參數](service-fabric-manage-multiple-environment-app-configuration.md)
* [偵錯 Service Fabric 應用程式](service-fabric-debugging-your-application.md)
* [使用 Service Fabric 總管視覺化叢集](service-fabric-visualizing-your-cluster.md)

<!--Image references-->
[addserviceproject]:./media/service-fabric-manage-application-in-visual-studio/addserviceproject.png
[manageservicefabric]: ./media/service-fabric-manage-application-in-visual-studio/manageservicefabric.png
[newservice]:./media/service-fabric-manage-application-in-visual-studio/newservice.png
[newserviceapplicationmanifest]:./media/service-fabric-manage-application-in-visual-studio/newserviceapplicationmanifest.png
[debugmodeproperty]:./media/service-fabric-manage-application-in-visual-studio/debugmodeproperty.png
[removeapplication]:./media/service-fabric-manage-application-in-visual-studio/removeapplication.png