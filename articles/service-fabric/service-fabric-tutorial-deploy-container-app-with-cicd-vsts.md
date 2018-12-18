---
title: 將具有 CI/CD 的容器應用程式部署到 Service Fabric 叢集
description: 在本教學課程中，您會了解如何使用 Visual Studio Team Services (VSTS) 設定 Azure Service Fabric 容器應用程式的持續整合和部署。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/29/2018
ms.author: ryanwi,v-steg
ms.custom: mvc
ms.openlocfilehash: 5ec898465b550a136db71ee731be5943c7441dcd
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44381894"
---
# <a name="tutorial-deploy-a-container-application-with-cicd-to-a-service-fabric-cluster"></a>教學課程：將具有 CI/CD 的容器應用程式部署到 Service Fabric 叢集

本教學課程是系列中的第二部分，說明如何使用 Visual Studio Team Services (VSTS) 設定 Azure Service Fabric 容器應用程式的持續整合和部署。  需要現有的 Service Fabric 應用程式，在[將 Windows 容器中的 .NET 應用程式部署到 Azure Service Fabric](service-fabric-host-app-in-a-container.md) 中建立的應用程式會作為範例。

在本系列的第二部分中，您將瞭解如何：

> [!div class="checklist"]
> * 將原始檔控制新增至專案
> * 在 Team Services 中建立組建定義
> * 在 Team Services 中建立發行定義
> * 自動部署和升級應用程式

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 具有 Azure 上的叢集，或[使用本教學課程建立一個](service-fabric-tutorial-create-vnet-and-windows-cluster.md)
* [將容器化應用程式部署至該叢集](service-fabric-host-app-in-a-container.md)

## <a name="prepare-a-publish-profile"></a>下載發行設定檔

您現在已經[部署容器應用程式](service-fabric-host-app-in-a-container.md)，接下來可以設定持續整合。  首先，在應用程式中準備部署程序使用的發行設定檔 (於 Team Services 內執行)。  發行設定檔應設定為以先前建立的叢集為目標。  啟動 Visual Studio，並開啟現有的 Service Fabric 應用程式專案。  在 [方案總管] 中，以滑鼠右鍵按一下應用程式並選取 [發佈...]。

在您的應用程式專案內選擇要使用於持續整合工作流程 (例如雲端) 的目標設定檔。  指定叢集連線端點。  勾選**升級應用程式**核取方塊，讓您的應用程式對於 Team Services 中的每個部署升級。  按一下 [儲存] 超連結，將設定儲存至發行設定檔，然後按一下 [取消] 關閉對話方塊。

![推送設定檔][publish-app-profile]

## <a name="share-your-visual-studio-solution-to-a-new-team-services-git-repo"></a>向新的 Team Services Git 儲存機制共用 Visual Studio 解決方案

向 Team Services 中的小組專案共用應用程式原始檔，以便產生組建。

Visual Studio 右上角的狀態列上，選取 [新增至原始檔控制] -> [Git]，建立專案的新本機 Git 儲存機制。

在 [Team Explorer] 的 [推送] 檢視中，選取 [推送至 Visual Studio Team Services] 下的 [發佈 Git 儲存機制] 按鈕。

![推送 Git 儲存機制][push-git-repo]

確認您的電子郵件，並在 [Team Services 網域] 下拉式清單選取您的帳戶。 輸入您的儲存機制名稱，並選取 [發佈儲存機制]。

![推送 Git 儲存機制][publish-code]

發佈存放庫將在帳戶中建立與本機存放庫名稱相同的新 Team 專案。 若要在現有的 Team 專案中建立存放庫，請按一下**存放庫名稱**旁邊的 [進階]，並選取 Team 專案。 您可以選取**在網路上檢視**，在網路上檢視您的程式碼。

## <a name="configure-continuous-delivery-with-vsts"></a>設定 VSTS 的持續傳遞

Team Services 組建定義描述由一組循序執行的組建步驟所組成的工作流程。 建立產生 Service Fabric 應用程式封裝的組建定義，以及其他構件，以便部署到 Service Fabric 叢集。 深入了解 [Team Services 組建定義](https://www.visualstudio.com/docs/build/define/create)。 

Team Services 發行定義描述將應用程式封裝部署到叢集的工作流程。 一起使用時，組建定義和發行定義可以執行整個工作流程，從來源檔案開始，並以叢集中的執行中應用程式結束。 深入了解 Team Services [發行定義](https://www.visualstudio.com/docs/release/author-release-definition/more-release-definition)。

### <a name="create-a-build-definition"></a>建立組建定義

開啟網頁瀏覽器並瀏覽至新的 Team 專案，網址為：[https://&lt;myaccount&gt;.visualstudio.com/Voting/Voting%20Team/_git/Voting](https://myaccount.visualstudio.com/Voting/Voting%20Team/_git/Voting)。

選取 [組建與發行] 索引標籤，再選取 [組建]，然後按一下 [新增管線]。

![新增管線][new-pipeline]

選取 **VSTS Git** 作為來源，然後依序選取**投票** Team 專案、**投票**存放庫、**主要**預設分支或手動和排程組建。  然後按一下 [繼續]。

在 [選取範本] 中，選取 [含 Docker 支援的 Azure Service Fabric 應用程式] 範本，然後按一下 [套用]。

![選擇組建範本][select-build-template]

在 [工作] 中，選取 [Hosted VS2017] 作為 [代理程式集區]。

![選取工作][task-agent-pool]

按一下 [標記映像]。

在 [容器登錄類型] 中，選取 [Azure Container Registry]。 選取 [Azure 訂用帳戶]，然後按一下 [授權]。 選取 [Azure Container Registry]。

![選取 Docker Tag 映像][select-tag-images]

按一下 [推送映像]。

在 [容器登錄類型] 中，選取 [Azure Container Registry]。 選取 [Azure 訂用帳戶]，然後按一下 [授權]。 選取 [Azure Container Registry]。

![選取 Docker Push 映像][select-push-images]

在 [觸發程序] 下方，透過核取 [啟用持續整合] 來啟用持續整合。 在 [分支篩選] 內按一下 [+ 新增]，[分支規格] 將會預設為 [主要]。

在 [儲存組建管線和佇列] 對話方塊中，按一下 [儲存並加入佇列] 以手動啟動組建。

![選取觸發程序][save-and-queue]

推送或簽入時也會觸發組建。 若要檢查組建進度，請切換到 [組建] 索引標籤。您確認組建執行成功，請定義將應用程式部署至叢集的發行定義。

### <a name="create-a-release-definition"></a>建立發行定義

依序選取 [組建與發行] 索引標籤、[版本]，以及 [+ 新增管線]。  在 [選取範本] 中，從清單中選取 [Azure Service Fabric 部署] 範本，然後選取 [套用]。

![選擇發行範本][select-release-template]

選取 [工作] 和 [環境 1]，然後選取 [+ 新增] 來新增叢集連線。

![新增叢集連線][add-cluster-connection]

在 [新增 Service Fabric 連線] 檢視中，選取 [憑證式] 或 [Azure Active Directory] 驗證。  指定連線名稱 "mysftestcluster" 和叢集端點 "tcp://mysftestcluster.southcentralus.cloudapp.azure.com:19000" (或您要部署的目標叢集端點)。

若是憑證式驗證，請新增伺服器憑證的**伺服器憑證指紋** (用來建立叢集的伺服器憑證)。  在 [用戶端憑證] 中，新增 Base-64 編碼的用戶端憑證檔案。 請參閱該欄位的快顯說明，了解如何取得以 Base-64 編碼表示的憑證。 以及新增憑證的**密碼**。  如果您沒有個別的用戶端憑證，您可以使用叢集或伺服器憑證。

若是 Azure Active Directory 認證，請新增伺服器憑證的**伺服器憑證指紋** (用來建立叢集的伺服器憑證)，以及在 [使用者名稱] 和 [密碼] 欄位中新增要用來連線到叢集的認證。

按一下 [新增] 以儲存叢集連線。



在 [代理程式階段] 底下，按一下 [部署 Service Fabric 應用程式]。
按一下 [Docker 設定]，然後按一下 [設定 Docker 設定]。 在 [登錄認證來源] 中，選取 [Azure Resource Manager 服務連線]。 然後選取 [Azure 訂用帳戶]。

![發行管線代理程式][release-pipeline-agent]

接下來，將組建成品新增至管線，讓發行定義可以從組建中找到輸出。 選取 [管線] 和 [成品]->[+ 新增]。  在 [來源 (組建定義)] 中，選取您先前建立的組建定義。  按一下 [新增] 以儲存組建成品。

![新增成品][add-artifact]

啟用持續部署觸發程序，以便在組建完成時自動建立發行。 按一下成品中的閃電圖示、啟用觸發程序，然後按一下 [儲存] 以儲存發行定義。

![啟用觸發程序][enable-trigger]

選取 [+ 發行] -> [建立發行] -> [建立] 可手動建立發行。 您可以在 [發行] 索引標籤中監視發行進度。

確認部署成功，而且應用程式在叢集中執行。  開啟瀏覽器並巡覽至 [http://mysftestcluster.southcentralus.cloudapp.azure.com:19080/Explorer/](http://mysftestcluster.southcentralus.cloudapp.azure.com:19080/Explorer/)。  請注意應用程式版本，在此範例中是「1.0.0.20170616.3」。

## <a name="commit-and-push-changes-trigger-a-release"></a>認可並推送變更，觸發發行程序

簽入某些程式碼變更至 Team Services，確認持續整合管線正常運作。

您撰寫程式碼時，Visual Studio 會自動追蹤您的變更。 認可本機 Git 儲存機制的變更，方法是從右下方的狀態列選取暫止變更圖示 (![Pending][pending])。

在 Team Explorer 的**變更**檢視中，加入描述更新的訊息，並認可變更。

![全部認可][changes]

選取 [未發行的變更] 狀態列圖示 (![未發行的變更][unpublished-changes]) 或 Team Explorer 中的 [同步] 檢視。 選取 [推送] 更新 Team Services/TFS 中的程式碼。

![推送變更][push]

自動將變更推送至 Team Services 觸發組建。  組建定義成功完成時，發行就會自動建立，並開始升級叢集上的應用程式。

若要檢查組建進度，請切換到 **Team Explorer** Visual Studio 中的 [組建] 索引標籤。  您確認組建執行成功，請定義將應用程式部署至叢集的發行定義。

確認部署成功，而且應用程式在叢集中執行。  開啟瀏覽器並巡覽至 [http://mysftestcluster.southcentralus.cloudapp.azure.com:19080/Explorer/](http://mysftestcluster.southcentralus.cloudapp.azure.com:19080/Explorer/)。  請注意應用程式版本，在此範例中是 "1.0.0.20170815.3"。

![Service Fabric Explorer][sfx1]

## <a name="update-the-application"></a>更新應用程式

在應用程式中進行程式碼變更。  在先前的步驟儲存並認可變更。

應用程式的升級開始後，即可注意 Service Fabric Explorer 中的升級進度：

![Service Fabric Explorer][sfx2]

應用程式升級可能需要幾分鐘的時間。 升級完成時，應用程式將執行下一個版本。  在此範例中為 "1.0.0.20170815.4"。

![Service Fabric Explorer][sfx3]

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 將原始檔控制新增至專案
> * 建立組建定義
> * 建立發行定義
> * 自動部署和升級應用程式

在本教學課程的下一個部分，了解如何設定[監視您的容器](service-fabric-tutorial-monitoring-wincontainers.md)。

<!-- Image References -->
[publish-app-profile]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/PublishAppProfile.png
[push-git-repo]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/PublishGitRepo.png
[publish-code]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/PublishCode.png
[new-pipeline]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/NewPipeline.png
[select-build-template]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SelectBuildTemplate.png
[task-agent-pool]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/TaskAgentPool.png
[save-and-queue]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SaveAndQueue.png
[select-tag-images]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/DockerTagImages.png
[select-push-images]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/DockerPushImages.png
[select-release-template]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SelectReleaseTemplate.png
[set-continuous-integration]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SetContinuousIntegration.png
[add-cluster-connection]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/AddClusterConnection.png
[release-pipeline-agent]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/ReleasePipelineAgent.png
[add-artifact]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/AddArtifact.png
[enable-trigger]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/EnableTrigger.png
[sfx1]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SFX1.png
[sfx2]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SFX2.png
[sfx3]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SFX3.png
[pending]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/Pending.png
[changes]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/Changes.png
[unpublished-changes]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/UnpublishedChanges.png
[push]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/Push.png
[continuous-delivery-with-VSTS]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/VSTS-Dialog.png
[new-service-endpoint]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/NewServiceEndpoint.png
[new-service-endpoint-dialog]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/NewServiceEndpointDialog.png
