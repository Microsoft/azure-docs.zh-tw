---
title: Service Fabric 專案建立後續步驟 |Microsoft Docs
description: 了解您剛才在 Visual Studio 中建立的應用程式專案。  了解如何使用教學課程建置服務，並深入了解為 Service Fabric 開發服務。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: 299d1f97-1ca9-440d-9f81-d1d0dd2bf4df
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 12/07/2017
ms.author: rwike77
ms.openlocfilehash: a87dd6f4afa152aebafdde24defcabe841ae2e9c
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34206460"
---
# <a name="your-service-fabric-application-and-next-steps"></a>您的 Service Fabric 應用程式和後續步驟
您的 Azure Service Fabric 應用程式已經建立。 本文說明一些可嘗試的教學課程、專案的組成、您感興趣的一些其他資訊，以及潛在後續步驟。

## <a name="get-started-with-tutorials-walk-throughs-and-samples"></a>開始使用教學課程、逐步解說和範例
準備好開始了嗎？  

逐步查看 .NET 應用程式教學課程。 了解如何使用 ASP.NET Core 前端和具狀態後端來[建置應用程式](service-fabric-tutorial-create-dotnet-app.md)、[部署應用程式](service-fabric-tutorial-deploy-app-to-party-cluster.md)至叢集、[設定 CI/CD](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)，以及[設定監視和診斷](service-fabric-tutorial-monitoring-aspnet.md)。

或者，嘗試其中一個下列逐步解說，然後建立您的第一個...
- [Windows 上的 C# Reliable Services 服務](service-fabric-reliable-services-quick-start.md) 
- [Windows 上的 C# Reliable Actors 服務](service-fabric-reliable-actors-get-started.md) 
- [Windows 上可供來賓執行的服務](quickstart-guest-app.md) 
- [Windows 容器應用程式](service-fabric-get-started-containers.md) 

您可以試用我們的[範例應用程式](http://aka.ms/servicefabricsamples)。

## <a name="have-questions-or-feedback--need-to-report-an-issue"></a>有任何問題或意見嗎？  需要回報問題？
閱讀[常見問題](service-fabric-common-questions.md)並尋找有關 Service Fabric 可以執行的工作和其使用方式的答案。

[支援選項](service-fabric-support.md)列出 StackOverflow 和 MSDN 上用於詢問問題的論壇，以及用於報告問題、取得支援及提交產品意見反應的選項。

## <a name="the-application-project"></a>應用程式專案
每個新應用程式都包含一個應用程式專案。 視所選擇的服務類型而定，可能會有一個或兩個額外的專案。

應用程式專案包含：

* 一組對構成應用程式之服務的參考。
* 三個發行設定檔 (1-Node Local、5-Node Local、Cloud)，可用來維護喜好設定，以搭配不同的環境使用，例如叢集端點的相關喜好設定，以及是否預設要執行升級部署。
* 三個應用程式參數檔案 (同上)，可用來維護環境特定應用程式組態，例如為服務建立之分割區的數目。 了解如何[針對多種環境設定您的應用程式](service-fabric-manage-multiple-environment-app-configuration.md)。
* 部署指令碼，可用來從命令列部署您的應用程式，或透過自動化連續整合和部署管線中部署您的應用程式。 深入了解[使用 PowerShell 部署應用程式](service-fabric-deploy-remove-applications.md)。
* 應用程式資訊清單，描述應用程式。 您可以在 ApplicationPackageRoot 資料夾下方找到資訊清單。 深入了解[應用程式及服務資訊清單](service-fabric-application-model.md)。



## <a name="learn-more-about-the-programming-models"></a>深入了解有關程式設計模型
Service Fabric 提供多種撰寫和管理服務的方式。  以下的概觀和概念性資訊是關於[無狀態與具狀態的 Reliable Services](service-fabric-reliable-services-introduction.md)、[Reliable Actors](service-fabric-reliable-actors-introduction.md)、[容器](service-fabric-containers-overview.md)、[可供來賓執行](service-fabric-guest-executables-introduction.md)和[無狀態與具狀態的 ASP.NET Core 服務](service-fabric-reliable-services-communication-aspnetcore.md)。

## <a name="learn-about-service-communication"></a>深入了解服務通訊
Service Fabric 應用程式是由不同的服務組成，每一個服務用來執行特定的工作。 這些服務可能會彼此通訊，且連接到服務並與服務通訊的叢集外可能會有用戶端應用程式。 了解如何在 Service Fabric 中[設定您的服務之間的通訊](service-fabric-connect-and-communicate-with-services.md)。 

## <a name="learn-about-configuring-application-security"></a>了解有關設定應用程式安全性
您可以保護在叢集中以不同使用者帳戶執行的應用程式。 在以該使用者帳戶部署時，Service Fabric 也會協助保護應用程式所使用的資源，例如檔案、目錄和憑證。 如此一來，即使在共用主控環境中，執行中的應用程式就不會彼此干擾。  了解如何[設定應用程式的安全性原則](service-fabric-application-runas-security.md)。

應用程式可能包含機密資訊，例如儲存體連接字串、密碼或其他不會以純文字處理的值。 了解如何[管理您的應用程式中的祕密](service-fabric-application-secret-management.md)。

## <a name="learn-about-the-application-lifecycle"></a>了解應用程式生命週期
如同其他平台，Service Fabric 應用程式通常會經歷下列階段：設計、開發、測試、部署、升級、維護和移除。 [本文章](service-fabric-application-lifecycle.md)提供 API 的概觀以及其使用方式，這些 API 是由不同的角色在 Service Fabric 應用程式生命週期的各個階段使用。

## <a name="next-steps"></a>後續步驟
- [在 Azure 中建立 Windows 叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)。
- 使用 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 將叢集視覺化，包括已部署的應用程式和實體配置。
- [進行服務版本設定和升級](service-fabric-application-upgrade-tutorial.md)


