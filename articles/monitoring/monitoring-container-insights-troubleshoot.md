---
title: 如何對適用於容器的 Azure 監視器進行疑難排解 | Microsoft Docs
description: 本文說明如何對適用於容器的 Azure 監視器問題進行疑難排解並解決問題。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/14/2018
ms.author: magoedte
ms.openlocfilehash: de7ae5788224b83105e4dc9a24aea35c8b841c88
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986722"
---
# <a name="troubleshooting-azure-monitor-for-containers"></a>對適用於容器的 Azure 監視器進行疑難排解

當您設定使用適用於容器的 Azure 監視器來監視 Azure Kubernetes Service (AKS) 叢集時，您可能會遇到阻止資料收集或報告狀態的問題。 本文將詳細說明一些常見問題與疑難排解步驟。

## <a name="azure-monitor-for-containers-is-enabled-but-not-reporting-any-information"></a>適用於容器的 Azure 監視器已啟用但未報告任何資訊
如果適用於容器的 Azure 監視器已成功啟用並設定，但您無法檢視狀態資訊，或者未從 Log Analytics 記錄查詢中傳回任何結果，您可以遵循下列步驟來診斷此問題： 

1. 執行下列命令來檢查代理程式的狀態： 

    `kubectl get ds omsagent --namespace=kube-system`

    輸出應該像下面這樣，這表示它已正確部署：

    ```
    User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system 
    NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
    omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
    ```  
2. 使用下列命令，利用代理程式版本 *06072018* 或更新版本來檢查部署狀態：

    `kubectl get deployment omsagent-rs -n=kube-system`

    輸出應該會像下列範例，這表示它已正確部署：

    ```
    User@aksuser:~$ kubectl get deployment omsagent-rs -n=kube-system 
    NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
    omsagent   1         1         1            1            3h
    ```

3. 執行下列命令來檢查 Pod 的狀態，以確認其正在執行：`kubectl get pods --namespace=kube-system`

    輸出應該會像下列範例，且 omsagent 的狀態為「執行中」：

    ```
    User@aksuser:~$ kubectl get pods --namespace=kube-system 
    NAME                                READY     STATUS    RESTARTS   AGE 
    aks-ssh-139866255-5n7k5             1/1       Running   0          8d 
    azure-vote-back-4149398501-7skz0    1/1       Running   0          22d 
    azure-vote-front-3826909965-30n62   1/1       Running   0          22d 
    omsagent-484hw                      1/1       Running   0          1d 
    omsagent-fkq7g                      1/1       Running   0          1d 
    ```

4. 檢查代理程式記錄。 容器化的代理程式完成部署時，它會執行 OMI 命令並顯示代理程式與提供者版本，以執行快速檢查。 

5. 執行下列命令，以確認代理程式已順利上線：`kubectl logs omsagent-484hw --namespace=kube-system`

    狀態應該會像下列範例：

    ```
    User@aksuser:~$ kubectl logs omsagent-484hw --namespace=kube-system
    :
    :
    instance of Container_HostInventory
    {
        [Key] InstanceID=3a4407a5-d840-4c59-b2f0-8d42e07298c2
        Computer=aks-nodepool1-39773055-0
        DockerVersion=1.13.1
        OperatingSystem=Ubuntu 16.04.3 LTS
        Volume=local
        Network=bridge host macvlan null overlay
        NodeRole=Not Orchestrated
        OrchestratorType=Kubernetes
    }
    Primary Workspace: b438b4f6-912a-46d5-9cb1-b44069212abc    Status: Onboarded(OMSAgent Running)
    omi 1.4.2.2
    omsagent 1.6.0.23
    docker-cimprov 1.0.0.31
    ```

## <a name="next-steps"></a>後續步驟
藉由啟用監視來擷取 AKS 叢集節點和 Pod 的健康情況計量，這些健康情況計量都可在 Azure 入口網站中取得。 若要了解如何使用適用於容器的 Azure 監視器，請參閱[檢視 Azure Kubernetes Service 健康情況](monitoring-container-insights-analyze.md)。