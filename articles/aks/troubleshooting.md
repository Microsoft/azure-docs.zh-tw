---
title: 針對常見的 Azure Kubernetes Service 問題進行疑難排解
description: 了解在使用 Azure Kubernetes Service (AKS) 時，如何針對常見問題進行疑難排解並加以解決
services: container-service
author: sauryadas
ms.service: container-service
ms.topic: troubleshooting
ms.date: 08/13/2018
ms.author: saudas
ms.openlocfilehash: 9f082c5f198ebd7123058bd250d3fef55494d553
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43287534"
---
# <a name="aks-troubleshooting"></a>AKS 疑難排解
當您建立或管理 AKS 叢集時，可能偶爾會遇到問題。 本文將詳細說明一些常見問題與疑難排解步驟。

### <a name="in-general-where-do-i-find-information-about-debugging-kubernetes-issues"></a>我通常可以在哪裡找到針對 Kubernetes 進行偵錯問題的相關資訊？

[這裡](https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/) 是針對 Kubernetes 叢集進行疑難排解的官方連結。
[這裡](https://github.com/feiskyer/kubernetes-handbook/blob/master/en/troubleshooting/index.md)是由 Microsoft 工程師所發佈之疑難排解指南的連結，該指南可用來針對 Pod、節點、叢集等進行疑難排解。

### <a name="i-am-getting-a-quota-exceeded-error-during-create-or-upgrade-what-should-i-do"></a>我在建立或升級期間收到已超出配額的錯誤。 我該怎麼辦？ 

您將需要在[這裡](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request)要求核心。

### <a name="what-is-the-max-pods-per-node-setting-for-aks"></a>對於 AKS，每個節點的最大 Pod 數設定為何？

如果您在 Azure 入口網站中部署 AKS 叢集，每個節點的最大 Pod 數預設會設為 30。
如果您在 Azure CLI 中部署 AKS 叢集，每個節點的最大 Pod 數預設會設為 110 (請確定您使用的是最新版的 Azure CLI)。 您可以在 az aks create 命令中，使用 –max-nodes-per-pod 旗標來變更這個預設設定。

### <a name="i-am-getting-insufficientsubnetsize-error-while-deploying-an-aks-cluster-with-advanced-networking-what-should-i-do"></a>當我使用進階網路設定部署 AKS 叢集時收到 “insufficientSubnetSize” 錯誤。 我該怎麼辦？

在建立 AKS 期間選取來進行網路設定的自訂 VNET 選項中，會針對 IPAM 使用 Azure CNI。 AKS 叢集中的節點數可以是介於 1 到 100 之間的任何數字。 根據上述的 2)，子網路大小應大於節點數和每個節點的最大 Pod 數之乘積，子網路大小 > 叢集中的節點數 * 每個節點的最大 Pod 數。

### <a name="my-pod-is-stuck-in-crashloopbackoff-mode-what-should-i-do"></a>我的 Pod 會在 'CrashLoopBackOff' 模式中停滯。 我該怎麼辦？

有各種不同的原因可能會讓 Pod 在該模式中停滯。 您可能想要 
* 使用 `kubectl describe pod <pod-name>` 了解 Pod 本身
* 使用 `kubectl log <pod-name>` 了解記錄

### <a name="i-am-trying-to-enable-rbac-on-an-existing-cluster-can-you-tell-me-how-i-can-do-that"></a>我正嘗試在現有叢集上啟用 RBAC。 您可以告訴我如何辦到嗎？

很遺憾，目前不支援在現有叢集上啟用 RBAC。 您將需要明確地建立新的叢集。 如果您使用 CLI，預設會啟用 RBAC，否則，可在 AKS 入口網站建立工作流程中使用切換按鈕來啟用它。

### <a name="i-created-a-cluster-using-the-azure-cli-with-defaults-or-the-azure-portal-with-rbac-enabled-and-numerous-warnings-in-the-kubernetes-dashboard-the-dashboard-used-to-work-before-without-any-warnings-what-should-i-do"></a>我已使用 Azure CLI 搭配預設值或已啟用 RBAC 的 Azure 入口網站來建立叢集，但 Kubernetes 儀表板中產生了大量警告。 該儀表板過去一直可正常運作，而且不會產生任何警告。 我該怎麼辦？

儀表板上產生警告的原因是，因為根據預設，現在會透過 RBAC 啟用它，並已停用對它的存取權。 這種方法通常會被視為良好的做法，因為預設向叢集的所有使用者公開該儀表板可能會導致安全性威脅。 如果您仍然想要啟用該儀表板，請依照這個[部落格](https://pascalnaber.wordpress.com/2018/06/17/access-dashboard-on-aks-with-rbac-enabled/) \(英文\) 來啟用。

### <a name="i-cant-seem-to-connect-to-the-dashboard-what-should-i-do"></a>我似乎無法連線到儀表板。 我該怎麼辦？

存取叢集外部服務的最簡單方式是執行 kubectl proxy，它會將您 localhost 連接埠 8001 的要求 Proxy 到 Kubernetes API 伺服器。 Apiserver 可以從該處 Proxy 到您的服務： http://localhost:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/#!/node?namespace=default

如果您沒有看到 Kubernetes 儀表板，請檢查 kube-proxy Pod 是否正在 kube-system 命名空間中執行。 如果它並未處於執行中狀態，請刪除該 Pod，而它將會重新啟動。

### <a name="i-could-not-get-logs-using-kubectl-logs-or-cannot-connect-to-the-api-server-getting-the-error-from-server-error-dialing-backend-dial-tcp-what-should-i-do"></a>我無法使用 Kubectl 記錄來取得記錄，或者無法連線到 API 伺服器並發生下列錯誤：「來自伺服器的錯誤: 撥號後端錯誤: dial tcp…」。 我該怎麼辦？

請確定並未修改預設的 NSG，並已針對與 API 伺服器的連線開啟連接埠 22。 檢查 tunnelfront Pod 是否正在 kube-system 命名空間中執行。 如果沒有，請強制刪除它，而它將會重新啟動。

### <a name="i-am-trying-to-upgrade-or-scale-and-am-getting-message-changing-property-imagereference-is-not-allowed-error--how-do-i-fix-this-issue"></a>我正嘗試升級或進行調整，但收到「訊息」：「不允許變更屬性 'imageReference'。」 錯誤。  如何修正此問題？

您可能會因為修改了 AKS 叢集內代理程式節點中的標記而收到此錯誤。 修改和刪除 MC_* 資源群組中資源的標記和其他屬性可能會導致非預期的結果。 在 AKS 叢集中修改 MC_* 下的資源會中斷 SLO。


