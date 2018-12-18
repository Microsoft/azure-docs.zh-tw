---
title: 在 Windows Server 和 Linux 上建立 Service Fabric 叢集 | Microsoft Docs
description: Service Fabric 叢集會在 Windows Server 或 Linux 上執行，這表示您能夠在任何您可以執行 Windows Server 和 Linux 的環境中部署和裝載 Service Fabric 應用程式。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: 19ca51e8-69b9-4952-b4b5-4bf04cded217
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/28/2018
ms.author: dekapur
ms.openlocfilehash: 80b331d32fe1e7bb4eb331bd981106968bc73bed
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44163204"
---
# <a name="create-service-fabric-clusters-on-windows-server-or-linux"></a>在 Windows Server 或 Linux 上建立 Service Fabric 叢集
Service Fabric 叢集是一組由網路連接的虛擬或實體機器，可用來將您的微服務部署到其中並進行管理。 隸屬於叢集的機器或 VM 稱為叢集模式。 叢集可擴充至數千個節點。 若您新增節點至叢集，則 Service Fabric 會重新平衡全體增加節點數的服務資料分割複本和執行個體。 整體應用程式效能會有所改善，改善，並減少爭用記憶體的存取權。 若未有效率地使用叢集中的節點，您可減少叢集中的節點數目。 Service Fabric 會再次重新平衡全體減少節點數的資料分割複本和執行個體，以善加使用每個節點上的硬體。

Service Fabric 可讓您在執行 Windows Server 或 Linux 的任何 VM 或電腦上建立 Service Fabric 叢集。 這表示在任何環境中，只要有一組互連式 Windows Server 或 Linux 電腦，不論是在內部部署、Microsoft Azure 或透過任何雲端提供者，您都能夠部署和執行 Service Fabric 應用程式。

## <a name="create-service-fabric-clusters-on-azure"></a>在 Azure 上建立 Service Fabric 叢集
在 Azure 上建立叢集要透過資源模型範本或 [Azure 入口網站](https://portal.azure.com)。 如需詳細資訊，請參閱[使用 Resource Manager 範本來建立 Service Fabric 叢集](service-fabric-cluster-creation-via-arm.md)或[從 Azure 入口網站建立 Service Fabric 叢集](service-fabric-cluster-creation-via-portal.md)。

## <a name="supported-operating-systems-for-clusters-on-azure"></a>Azure 上的叢集支援的作業系統
您可以在執行下列作業系統的虛擬機器上建立叢集：

* Windows Server 2012 R2
* Windows Server 2016 
* Windows Server 1709
* Linux Ubuntu 16.04

> [!NOTE]
> 如果您決定在 Windows Server 1709 上部署 Service Fabric，請注意 (1) 它不是長期維護分支，因此您必須在未來移轉版本，以及 (2) 如果您部署容器，則建置於 Windows Server 2016 上的容器無法在 Windows Server 1709 上運作，反之亦然 (您必須重建容器才能部署它們)。
>

## <a name="create-service-fabric-standalone-clusters-on-premises-or-with-any-cloud-provider"></a>在內部部署或透過任何雲端提供者建立 Service Fabric 獨立叢集
Service Fabric 提供安裝套件，可讓您在內部部署或任何雲端提供者上建立獨立 Service Fabric 叢集。

如需在 Windows Server 上設定獨立 Service Fabric 叢集的詳細資訊，請參閱[建立適用於 Windows Server 的 Service Fabric 叢集](service-fabric-cluster-creation-for-windows-server.md)

  > [!NOTE]
  > 目前不支援 Linux 的獨立叢集。 在開發與 Azure Linux 多電腦叢集的同一系統上支援 Linux。
  >

### <a name="any-cloud-deployments-vs-on-premises-deployments"></a>任何雲端部署與內部部署之比較
在內部部署環境建立 Service Fabric 叢集的程序會與在您所選擇、具有一組 VM 的任何雲端建立叢集的程序類似。 佈建 VM 的初始步驟取決於您要使用的雲端提供者或內部部署環境。 在您有一組彼此之間已啟用網路連線的 VM 之後，則安裝 Service Fabric 套件、編輯叢集設定，以及執行叢集建立與管理指令碼的步驟將會相同。 這可確保當您選擇以新裝載環境做為目標時，可將您操作和管理 Service Fabric 叢集方面的知識與經驗轉移過去。

### <a name="benefits-of-creating-standalone-service-fabric-clusters"></a>建立獨立 Service Fabric 叢集的優點
* 您可以自由選擇任何雲端提供者來裝載您的叢集。
* Service Fabric 應用程式一旦撰寫完成，只需進行最低限度的變更或不需任何變更，即可在多個裝載環境中執行。
* 建置 Service Fabric 應用程式的知識可以從一個裝載環境轉移到其他裝載環境。
* 執行和管理 Service Fabric 叢集的操作經驗可以從一個環境轉移到其他環境。
* 不受裝載環境條件約束束縛的廣大客戶群。
* 有一層額外的可靠性和保護可防止發生廣泛的中斷狀況，因為您可以在資料中心或雲端提供發生中斷時，將服務移到另一個部署環境。

## <a name="supported-operating-systems-for-standalone-clusters"></a>獨立叢集支援的作業系統
您可以在執行下列作業系統的 VM 或電腦上建立叢集 (尚不支援 Linux)：

* Windows Server 2012 R2
* Windows Server 2016 

## <a name="advantages-of-service-fabric-clusters-on-azure-over-standalone-service-fabric-clusters-created-on-premises"></a>Azure 上的 Service Fabric 叢集的優點勝於在內部部署建立的獨立 Service Fabric 叢集
在 Azure 上執行 Service Fabric 叢集可提供勝於內部部署選項的優點，因此如果您對叢集的執行位置沒有特定的需求，則建議您將在 Azure 上執行叢集。 在 Azure 中，我們與其他的 Azure 功能和服務整合，因此能輕鬆可靠地操作與管理叢集。

* **Azure 入口網站：** Azure 入口網站能輕鬆建立和管理叢集。
* **Azure 資源管理員：** 使用 Azure Resource Manager 可輕鬆管理叢集做為單位使用的所有資源，並簡化成本追蹤與付費作業。
* **Service Fabric 叢集做為 Azure 資源** Service Fabric 叢集是 Azure 資源，因此可以像 Azure 中的其他資源進行模型化。
* **與 Azure 基礎結構整合** Service Fabric 會協調作業系統的 Azure 基礎結構、網路和其他升級，以改善應用程式的可用性和可靠性。  
* **診斷：** 在 Azure 上，我們提供與 Azure 診斷及 Log Analytics 的整合。
* **自動調整：** 對於 Azure 上的叢集，我們會提供虛擬機器調整集產生的內建自動調整功能。 在內部部署與其他雲端環境中，您必須建置您自己的自動調整規模功能，或使用 Service Fabric 針對調整叢集規模顯示的 API 來手動調整規模。

## <a name="next-steps"></a>後續步驟

* 在執行 Windows Server 的 VM 或電腦上建立叢集： [建立適用於 Windows Server 的 Service Fabric 叢集](service-fabric-cluster-creation-for-windows-server.md)
* 在 VM 或執行 Linux 的電腦上建立叢集：[建立 Linux 叢集](service-fabric-cluster-creation-via-portal.md)
* 了解 [Service Fabric 支援選項](service-fabric-support.md)

