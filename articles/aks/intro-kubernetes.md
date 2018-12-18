---
title: 介紹 Azure Kubernetes Service
description: 了解 Azure Kubernetes Service 的功能與優點，以在 Azure 中部署及管理容器型應用程式。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: overview
ms.date: 09/26/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: ac18b8099b14f29579b35eb8dd2124d2d63f80d7
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49390069"
---
# <a name="azure-kubernetes-service-aks"></a>Azure Kubernetes Service (AKS)

Azure Kubernetes Service (AKS) 可讓您輕鬆地在 Azure 中部署受控 Kubernetes 叢集。 AKS 可降低管理 Kubernetes 的複雜性和作業負荷，因為是由 Azure 負責大部分的工作。 以主控的 Kubernetes 服務形式，Azure 會為您處理像是健康狀態監視和維護等重要工作。 Kubernetes 主機是由 Azure 管理。 您只管理及維護代理程式節點。 由於是受控 Kubernetes 服務，AKS 是免費的，您只需要針對叢集中的代理程式節點付費，而不用為主機付費。

您可以使用 Azure CLI 或 Resource Manager 範本與 Terraform 等以範本為基礎的部署選項，在 Azure 入口網站中建立 AKS 叢集。 部署 AKS 叢集時，系統會為您部署及設定 Kubernetes 主機與所有節點。 部署程序中也可以設定其他功能，例如進階網路功能、Azure Active Directory 整合及監視功能。

若要開始使用，請在 [Azure 入口網站][aks-portal]中或使用 [Azure CLI][aks-cli] 來完成 AKS 快速入門。

## <a name="access-security-and-monitoring"></a>存取、安全性和監視

針對改善的安全性和管理，AKS 可讓您與 Azure Active Directory 整合，並使用 Kubernetes 角色型存取控制。 您也可以監視叢集與資源的健康情況。

### <a name="identity-and-security-management"></a>身分識別和安全性管理

為限制對叢集資源的存取，AKS 支援 [Kubernetes 角色型存取控制 (RBAC)][kubernetes-rbac]。 RBAC 可讓您控制 Kubernetes 資源與命名空間的存取，以及這些資源的權限。 您也可以將 AKS 叢集設定成與 Azure Active Directory (AD) 整合。 若與 Azure AD 整合，您可以根據現有的身分識別與群組成員資格來設定 Kubernetes 存取。 可以為您現有的 Azure AD 使用者與群組提供 AKS 資源的存取權，且包含整合式登入體驗。

若要保護您的 AKS 叢集，請參閱[整合 Azure Active Directory 與 AKS][aks-aad]。

### <a name="integrated-logging-and-monitoring"></a>整合的記錄和監視功能

若要了解您的 AKS 叢集與已部署應用程式的執行情況，適用於容器健康情況的 Azure 監視器可從容器、節點與控制站收集記憶體與處理器計量。 您可以取得容器記錄檔，而且也可以[檢閱 Kubernetes 主機記錄檔][aks-master-logs]。 此監視資料會儲存在 Azure Log Analytics 工作區，而且可透過 Azure 入口網站、Azure CLI 或 REST 端點取得。

如需詳細資訊，請參閱[監視 Azure Kubernetes Service 的容器健康情況][container-health]。

## <a name="cluster-and-node"></a>叢集與節點

AKS 節點是在 Azure 虛擬機器上執行的。 您可以將儲存體連接到節點和 Pod、升級叢集節點，以及使用 GPU。

### <a name="cluster-node-and-pod-scaling"></a>調整叢集節點與 Pod

視資源變更的需求，執行服務的叢集節點或 Pod 數目可以自動相應增加或減少。 您可以同時使用水平 Pod 自動調整程式或叢集自動調整程式。 此調整方式讓 AKS 叢集視需要自動調整，並且只執行需要的資源。

如需詳細資訊，請參閱[縮放 Azure Kubernetes Service (AKS) 叢集][aks-scale]。

### <a name="cluster-node-upgrades"></a>叢集節點升級

Azure Kubernetes Service 提供多個 Kubernetes 版本。 當 AKS 有新版本可用時，您可以使用 Azure 入口網站或 Azure CLI 來升級您的叢集。 在升級過程中，會將節點仔細地隔離並清空，將中斷執行中應用程式的情況降到最低。

若要深入了解生命週期的版本，請參閱 [AKS 中支援的 Kubernetes 版本][aks-supported versions]。 如需如何升級的步驟，請參閱[升級 Azure Kubernetes Service (AKS) 叢集][aks-upgrade]。

### <a name="gpu-enabled-nodes"></a>已啟用 GPU 的節點

AKS 支援建立已啟用 GPU 的節點集區。 Azure 目前提供單一或多個已啟用 GPU 的虛擬機器。 已啟用 GPU 的虛擬機器是專門針對計算密集型、圖形密集型及視覺效果的工作負載所設計。

如需詳細資訊，請參閱[在 AKS 上使用 GPU][aks-gpu]。

### <a name="storage-volume-support"></a>存放磁碟區支援

若要支援應用程式工作負載，您可以掛接存放永續性資料的儲存體磁碟區。 可以使用靜態與動態磁碟區。 您可以根據要共用儲存體的已連線 Pod 數目，來決定要使用 Azure 磁碟所提供適用於單一 Pod 存取的儲存體，或使用 Azure 檔案服務所提供適用於多個 Pod 並行存取的儲存體。

開始以 [Azure 磁碟][azure-disk]或 [Azure 檔案服務][azure-files]來使用動態永續性磁碟區。

## <a name="virtual-networks-and-ingress"></a>虛擬網路與輸入

AKS 叢集可以部署到現有的虛擬網路中。 在此設定中，叢集中的每個 Pod 都會獲指派虛擬網路中的 IP 位址，而且可以直接與叢集中的其他 Pod 以及虛擬網路中的其他節點通訊。 Pod 也可以連線到對等互連虛擬網路中的其他服務，也可以透過 ExpressRoute 或站對站 (S2S) VPN 連線來連線到內部部署網路。

如需詳細資訊，請參閱 [AKS 網路功能概觀][aks-networking]。

### <a name="ingress-with-http-application-routing"></a>使用 HTTP 應用程式路由的輸入

HTTP 應用程式路由附加元件可讓您輕鬆存取已部署到 AKS 叢集的應用程式。 啟用時，HTTP 應用程式路由解決方案會設定 AKS 叢集中的輸入控制器。 部署應用程式時，系統會自動設定可公開存取的 DNS 名稱。 HTTP 應用程式路由會設定 DNS 區域，並將它與 AKS 叢集整合。 接著，您可以如往常一樣部署 Kubernetes 輸入資源。

若要開始使用輸入流量，請參閱 [HTTP 應用程式路由][aks-http-routing]。

## <a name="development-tooling-integration"></a>開發工具整合

Kubernetes 具有豐富的開發和管理工具生態系統，例如 Helm、Draft 和 Visual Studio Code 的 Kubernetes 擴充功能。 這些工具可與 AKS 完美搭配。

此外，Azure Dev Spaces 可為小組提供快速、疊代的 Kubernetes 開發經驗。 使用基本設定，您就可以直接在 AKS 中執行容器並進行偵錯。 若要開始使用，請參閱 [Azure Dev Spaces][azure-dev-spaces]。

Azure DevOps 專案提供簡單的解決方案，可讓您將現有的程式碼與 Git 存放庫帶入 Azure。 DevOps 專案會自動建立 ASK (Azure DevOps Services 中的發行管線，其中包含 CI 建置管線) 等 Azure 資源、設定 CD 發行管線，然後建立監視用的 Azure Application Insights 資源。

如需詳細資訊，請參閱 [Azure DevOps 專案][azure-devops]。

## <a name="docker-image-support-and-private-container-registry"></a>Docker 映像支援和私人容器登錄

AKS 支援 Docker 映像格式。 針對您 Docker 映像的私人儲存體，您可以將 AKS 與 Azure Container Registry (ACR) 整合。

若要建立私人映像存放區，請參閱 [Azure Container Registry][acr-docs]。

## <a name="kubernetes-certification"></a>Kubernetes 憑證

Azure Kubernetes Service (AKS) 已經由 CNCF 認證，符合 Kubernetes 相關規範。

## <a name="regulatory-compliance"></a>法規遵循

Azure Kubernetes Service (AKS) 符合 SOC、ISO 和 PCI DSS 的規範。

## <a name="next-steps"></a>後續步驟

深入了解使用 Azure CLI 部署及管理 AKS 的快速入門。

> [!div class="nextstepaction"]
> [AKS 快速入門][aks-cli]

<!-- LINKS - external -->
[acs-engine]: https://github.com/Azure/acs-engine
[draft]: https://github.com/Azure/draft
[helm]: https://helm.sh/
[kubectl-overview]: https://kubernetes.io/docs/user-guide/kubectl-overview/
[kubernetes-rbac]: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

<!-- LINKS - internal -->
[acr-docs]: ../container-registry/container-registry-intro.md
[aks-aad]: ./aad-integration.md
[aks-cli]: ./kubernetes-walkthrough.md
[aks-gpu]: ./gpu-cluster.md
[aks-http-routing]: ./http-application-routing.md
[aks-networking]: ./concepts-network.md
[aks-portal]: ./kubernetes-walkthrough-portal.md
[aks-scale]: ./tutorial-kubernetes-scale.md
[aks-upgrade]: ./upgrade-cluster.md
[azure-dev-spaces]: https://docs.microsoft.com/azure/dev-spaces/azure-dev-spaces
[azure-devops]: https://docs.microsoft.com/azure/devops-project/overview
[azure-disk]: ./azure-disks-dynamic-pv.md
[azure-files]: ./azure-files-dynamic-pv.md
[container-health]: ../monitoring/monitoring-container-health.md
[aks-master-logs]: view-master-logs.md
[aks-supported versions]: supported-kubernetes-versions.md
