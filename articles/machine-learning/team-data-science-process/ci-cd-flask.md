---
title: 人工智慧 (AI) 應用程式的 DevOps：使用 Docker、Kubernetes 及 Python Flask 應用程式在 Azure 上建立持續整合管線
description: 人工智慧 (AI) 應用程式的 DevOps：使用 Docker 和 Kubernetes 在 Azure 上建立持續整合管線
services: machine-learning, team-data-science-process
documentationcenter: ''
author: jainr
manager: deguhath
editor: cgronlun
ms.assetid: b8fbef77-3e80-4911-8e84-23dbf42c9bee
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/22/2018
ms.author: jainr
ms.openlocfilehash: b0368e742c990feed626a1c4982bfedc35785b49
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304283"
---
# <a name="devops-for-artificial-intelligence-ai-applications-creating-continuous-integration-pipeline-on-azure-using-docker-and-kubernetes"></a>人工智慧 (AI) 應用程式的 DevOps：使用 Docker 和 Kubernetes 在 Azure 上建立持續整合管線
就 AI 應用程式而言，有兩個經常性的工作資料流：資料科學家建置機器學習模型，而應用程式開發人員則建置應用程式並將其公開給使用者取用。 在本文中，我們將示範如何實作 AI 應用程式的持續整合 (CI)/持續部署 (CD) 管線。 AI 應用程式是內嵌了預先定型機器學習 (ML) 模型的應用程式碼組合。 針對本文，我們會從私用的 Azure Blob 儲存體帳戶 (也可以是 AWS S3 帳戶) 中擷取一個預先定型的模型。 我們將針對本文使用一個簡單的 Python Flask Web 應用程式。

> [!NOTE]
> 這是數種執行 CI/CD 的方式其中之一。 下面所提到的工具及其他先決條件皆有替代方案。 隨著我們對額外內容的開發，我們將會發佈這些替代方案。
>
>

## <a name="github-repository-with-document-and-code"></a>含有文件和程式碼的 GitHub 存放庫
您可以從 [GitHub](https://github.com/Azure/DevOps-For-AI-Apps)下載原始程式碼。 此外，也有提供[詳細的教學課程](https://github.com/Azure/DevOps-For-AI-Apps/blob/master/Tutorial.md)。

## <a name="pre-requisites"></a>先決條件
以下是依循下述 CI/CD 管線的先決條件：
* [Azure DevOps 組織](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization-msa-or-work-student)
* [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)
* [執行 Kubernetes 的 Azure Container Service (AKS) 叢集](https://docs.microsoft.com/azure/container-service/kubernetes/container-service-tutorial-kubernetes-deploy-cluster)
* [Azure Container Registy (ACR) 帳戶](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-portal)
* [安裝 Kubectl 以針對 Kubernetes 叢集執行命令](https://kubernetes.io/docs/tasks/tools/install-kubectl/)。 我們將需要此工具，才能從 ACS 叢集擷取設定。 
* 在您的 GitHub 帳戶建立該存放庫的分叉。

## <a name="description-of-the-cicd-pipeline"></a>CI/CD 管線的描述
管線會針對每個新的認可開始執行、執行測試套件，如果測試通過，便採用最新的組建、將它封裝在 Docker 容器中。 接著，會使用 Azure Container Service (ACS) 來部署該容器，並將映像安全地儲存在 Azure Container Registry (ACR) 中。 ACS 執行 Kubernetes 來管理容器叢集，但您可以選擇 Docker Swarm 或 Mesos。

應用程式會從 Azure 儲存體帳戶中安全地提取最新的模型，然後將其封裝成應用程式的一部分。 所部署應用程式的應用程式程式碼和 ML 模型會封裝成單一容器。 這可讓應用程式開發人員與資料科學家分開，以確保其生產環境應用程式一律搭配最新的 ML 模型來執行最新的程式碼。

管線架構如下所示。 

![架構](./media/ci-cd-flask/Architecture.PNG?raw=true)

## <a name="steps-of-the-cicd-pipeline"></a>CI/CD 管線的步驟
1. 開發人員在其選擇的 IDE 上處理應用程式程式碼。
2. 他們將程式碼認可至其選擇的原始檔控制 (Azure DevOps 可對各種原始檔控制提供良好的支援)
3. 資料科學家則是個別進行其模型開發。
4. 在對模型滿意之後，他們便將模型發佈到存放庫，在此案例中，我們使用 Blob 儲存體帳戶。 您可以使用 Azure ML Workbench 的模型管理服務 (透過其 REST API) 來輕鬆取代此做法。
5. 系統會根據 GitHub 中的認可開始在 Azure DevOps 中執行建置。
6. Azure DevOps 組建管線會從 Blob 容器中提取最新的模型，然後建立容器。
7. Azure DevOps 會將映像推送至 Azure Container Registry 中的私人映像存放庫
8. 發行管線會根據已設定的排程 (每晚) 開始執行。
9. 系統會從 ACR 中提取最新的映像，並部署至 ACS 上的整個 Kubernetes 叢集。
10. 使用者對應用程式發出的要求會經過 DNS 伺服器。
11. DNS 伺服器會將該要求傳遞給負載平衡器，然後將回應傳回給使用者。

## <a name="next-steps"></a>後續步驟
* 請參考[教學課程](https://github.com/Azure/DevOps-For-AI-Apps/blob/master/Tutorial.md)來依照詳細資料進行操作，並實作您自己的應用程式 CI/CD 管線。

## <a name="references"></a>參考
* [Team Data Science Process (TDSP)](https://aka.ms/tdsp)
* [Azure Machine Learning (AML)](https://docs.microsoft.com/azure/machine-learning/service/)
* [Visual Studio Team Services (VSTS)](https://www.visualstudio.com/vso/)
* [Azure Kubernetes Services (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes)
