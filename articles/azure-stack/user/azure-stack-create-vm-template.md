---
title: 在此教學課程中，您要使用範本來建立 Azure Stack 虛擬機器 | Microsoft Docs
description: 說明如何使用 ASDK，來建立會使用預先定義範本和 GitHub 自訂範本的虛擬機器。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 09/12/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: 0b3ba5c3a091cf673d8b3dbc413d36cb5fb75de5
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44713403"
---
# <a name="tutorial-create-a-vm-using-a-community-template"></a>教學課程：建立使用社群範本的虛擬機器
身為 Azure Stack 操作人員或使用者，您可以建立會使用[自訂 GitHub 快速入門的範本](https://github.com/Azure/AzureStack-QuickStart-Templates)的虛擬機器，而無須從 Azure Stack 市集手動部署虛擬機器。

在此教學課程中，您了解如何：

> [!div class="checklist"]
> * 了解 Azure Stack 快速入門範本 
> * 建立會使用自訂 GitHub 範本的虛擬機器
> * 啟動 minikube 並安裝應用程式

## <a name="learn-about-azure-stack-quickstart-templates"></a>了解 Azure Stack 快速入門範本
Azure Stack 快速入門範本會儲存在 GitHub 上的[公用 AzureStack 快速入門範本存放庫](https://github.com/Azure/AzureStack-QuickStart-Templates)中。 此存放庫包含已經過 Microsoft Azure Stack 開發套件 (ASDK) 測試的 Azure Resource Manager 部署範本。 您可以使用這些範本，輕鬆評估 Azure Stack 及使用 ASDK 環境。 

隨著時間的推移，GitHub 使用者對存放庫貢獻良多，目前其中已堂堂累積了超過 400 種部署範本。 想要了解如何將各式各樣的環境部署到 Azure Stack，此存放庫是絕佳的出發點。 

>[!IMPORTANT]
> 這些範本中有部分是社群成員所創造的，而不是由 Microsoft。 每個範本都是由其擁有者 (而非 Microsoft) 依據授權合約進行授權。 Microsoft 對於這些範本概不負責，且不會過濾其安全性、相容性或效能。 社群範本並未依據任何 Microsoft 支援方案或服務來提供支援，而且會在沒有任何擔保的情況下「依現況」提供。

若您想將 Azure Resource Manager 範本貢獻給 GitHub，請貢獻到 [azure-quickstart-templates 存放庫](https://github.com/Azure/AzureStack-QuickStart-Templates)中。

若要深入了解 GitHub 存放庫以及如何貢獻，請參閱[讀我檔案](https://github.com/Azure/AzureStack-QuickStart-Templates/blob/master/README.md) \(英文\)。 


## <a name="create-a-vm-using-a-custom-github-template"></a>建立會使用自訂 GitHub 範本的虛擬機器
在此範例教學課程中，會使用 [101-vm-linux-minikube](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/101-vm-linux-minikube) Azure Stack 快速入門範本，將 Ubuntu 16.04 虛擬機器部署在執行 Minikube 的 Azure Stack 上以管理 Kubenetes 叢集。

Minikube 是種可讓您在本機輕鬆執行 Kubernetes 的工具。 Minikube 會在 VM 內執行單一節點 Kubernetes 叢集，可讓您試用 Kubernetes 或使用它進行日常開發。 它支援在 Linux VM 上執行的簡單、單一節點 Kubernetes 叢集。 Minikube 是可讓完整功能的 Kubernetes 叢集運作的最快速、最直接方式。 它可讓開發人員在其本機電腦上，開發及測試以 Kubernetes 為基礎的應用程式部署。 在架構方面，Minikube VM 會在本機執行主要節點和代理程式節點元件：

- 主要節點元件 (例如 API 伺服器、排程器及 [etcd 伺服器](https://coreos.com/etcd/)) 會在稱為 LocalKube 的單一 Linux 程序中執行。
- 代理程式節點元件則會在 docker 容器中執行，正如這些元件會在正常的代理程式節點上執行一般。 從應用程式部署的觀點而言，在 Minikube 或一般 Kubernetes 叢集上部署應用程式，並沒有任何差異。

此範本會安裝下列元件：

- Ubuntu 16.04 LTS 虛擬機器
- [Docker-CE](https://download.docker.com/linux/ubuntu) 
- [Kubectl](https://storage.googleapis.com/kubernetes-release/release/v1.8.0/bin/linux/amd64/kubectl) \(英文\)
- [Minikube](https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64)
- xFCE4
- xRDP

> [!IMPORTANT]
> 開始這些步驟之前，必須已將 Ubuntu 虛擬機器映像 (在此範例中為 Ubuntu Server 16.04 LTS) 新增到 Azure Stack 市集中。

1.  按一下 [+ 建立資源] > [自訂] > [範本部署]。

    ![](media/azure-stack-create-vm-template/1.PNG) 

2. 按一下 [編輯範本]。

    ![](media/azure-stack-create-vm-template/2.PNG) 

3.  按一下 [快速入門範本]。

    ![](media/azure-stack-create-vm-template/3.PNG)

4. 使用 [選取範本] 下拉式清單，從可用的範本中選取 **101-vm-linux-minikube**，然後再按一下 [確定]。  

    ![](media/azure-stack-create-vm-template/4.PNG)

5. 若您想要修改範本 JSON，可以執行此操作；若否，完成時請按一下 [儲存] 以關閉 [編輯範本] 對話方塊。

    ![](media/azure-stack-create-vm-template/5.PNG) 

6.  按一下 [參數]、視需要填入或修改可用欄位，然後再按一下 [確定]。 選擇要使用的訂用帳戶、建立或選擇現有的資源群組名稱，然後按一下 [建立] 來起始範本部署。

    ![](media/azure-stack-create-vm-template/6.PNG)

7. 選擇要使用的訂用帳戶、建立或選擇現有的資源群組名稱，然後按一下 [建立] 來起始範本部署。

    ![](media/azure-stack-create-vm-template/7.PNG)

8. 資源群組部署需要數分鐘，才能建立以自訂範本為基礎的虛擬機器。 您可以透過通知或從資源群組屬性，監視安裝狀態。 

    ![](media/azure-stack-create-vm-template/8.PNG)

    >[!NOTE]
    > 部署完成後，虛擬機器就會開始執行。 

## <a name="start-minikube-and-install-an-application"></a>啟動 minikube 並安裝應用程式
Linux 虛擬機器現已成功建立，您可以登入以啟動 Minikube 並安裝應用程式。 

1. 部署完成後，請按一下 [連線] 以檢視用來連線到 Linux 虛擬機器的公用 IP 位址。 

    ![](media/azure-stack-create-vm-template/9.PNG)

2. 從提升權限的命令提示字元執行 **mstsc.exe**，開啟 [遠端桌面連線]，並連線到上一個步驟中所探索到的 Linux 虛擬機器公用 IP 位址。 當系統提示您登入 xRDP 時，請使用您在建立虛擬機器時指定的認證。

    ![](media/azure-stack-create-vm-template/10.PNG)

3. 開啟 [終端機模擬器] 並輸入下列命令，啟動 Minikube：

    ```shell
    sudo minikube start --vm-driver=none
    sudo minikube addons enable dashboard
    sudo minikube dashboard --url
    ```

    ![](media/azure-stack-create-vm-template/11.PNG)

4. 開啟網頁瀏覽器，然後瀏覽 Kubernetes 儀表板位址。 恭喜，您現在已擁有使用 Minikube 且具備完整功能的 Kubernetes 安裝了！

    ![](media/azure-stack-create-vm-template/12.PNG)

5. 若您想要部署範例應用程式，請造訪 Kubernetes 的官方文件頁面，並略過「建立 Minikube 叢集」一節 (因為您在以上步驟中已建立過)。 請直接跳到「建立 Node.js 應用程式」一節，網址是： https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/。

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何：

> [!div class="checklist"]
> * 了解 Azure Stack 快速入門範本 
> * 建立會使用自訂 GitHub 範本的虛擬機器
> * 啟動 minikube 並安裝應用程式

