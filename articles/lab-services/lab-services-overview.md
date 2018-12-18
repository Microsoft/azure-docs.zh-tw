---
title: 關於 Azure 實驗室服務 | Microsoft Azure
description: 了解實驗室服務如何協助您輕鬆搭配虛擬機器建立、管理及保護實驗室，以供開發人員、測試人員、教育工作者、學生及其他人員使用。
services: lab-services
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 07/13/2018
ms.author: spelluru
ms.openlocfilehash: cb957346cb0e003e9aff19026ed81495abf432c5
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39049372"
---
# <a name="an-introduction-to-azure-lab-services"></a>Azure 實驗室服務簡介
Azure 實驗室服務可讓您在雲端為團隊快速設定環境 (例如：開發環境、測試環境、教室實驗室環境)。 實驗室擁有者會建立實驗室，佈建 Windows 或 Linux 虛擬機器，安裝必要的軟體和工具，然後將它們提供給實驗室使用者使用。 實驗室使用者會連線至實驗室中的虛擬機器 (VM)，並使用它們來進行日常工作、短期專案，或是教室練習。 當使用者開始運用實驗室中的資源後，實驗室管理員便可以跨多個實驗室分析成本和使用情形，並設定整體性的原則來對組織或團隊的成本進行最佳化。

> [!IMPORTANT]
> 我們正在為 **Azure DevTest Labs** 擴展新的實驗室類型 (Azure 實驗室服務)！ 
> 
> Azure 實驗室服務可讓您建立受控實驗室，例如教室實驗室。 服務本身會處理受控實驗室的所有基礎結構管理，從啟動 VM 以處理錯誤，到調整基礎結構的規模。 受控實驗室目前為預覽狀態。 預覽結束之後，新的實驗室類型和現有的 DevTest Labs 都會被納入 Azure 實驗室服務底下，且所有實驗室類型都會在其中持續進化。 

## <a name="key-capabilities"></a>主要功能
Azure 實驗室服務支援下列主要能力/功能： 

- **快速且彈的實驗室設定**。 透過使用 Azure 實驗室服務，實驗室擁有者可以快速地依需求設定實驗室。 該服務提供由系統處理受控實驗室所有 Azure 基礎結構工作的選項，也可以讓實驗室擁有者在實驗室擁有者的訂用帳戶中自行管理及自訂基礎結構。 該服務針對實驗室提供由服務為您管理的內建基礎結構規模調整及復原功能。 
- **簡化的實驗室使用者體驗**。 在受控實驗室 (例如教室實驗室) 中，實驗室使用者可以使用註冊代碼來註冊至實驗室，並隨時存取該實驗室以使用實驗室的資源。 在於 DevTest Labs 中建立的實驗室中，實驗室擁有者可以給予實驗室使用者建立及存取虛擬機器、管理及重新使用資料磁碟，以及設定可重複使用祕密的權限。  
- **成本最佳化和分析**。 實驗室擁有者可以設定實驗室排程，以自動關閉及啟動虛擬機器。 實驗室擁有者可以設定排程以指定實驗室虛擬機器可供使用者存取的時段、設定每個使用者或實驗室的使用原則來將成本最佳化，以及分析實驗室中的使用情況和活動趨勢。 針對如教室實驗室等的受控實驗室，目前有一些成本最佳化和分析選項子集可供使用。 
- **內嵌安全性**。 實驗室使用者可以針對實驗室設定私人虛擬網路和子集，並提供共用的公用 IP 位址。 實驗室使用者可以使用已設定 ExpressRoute 或站對站 VPN 的虛擬網路，安全地存取資源。 (目前僅於 DevTest Labs 中提供)
- **與您的工作流程和工具整合**。 Azure 實驗室服務可讓您將實驗室整合至您組織的網站和管理系統。 您可以從您的持續整合/持續部署 (CI/CD) 工具內自動佈建環境。 (目前僅於 DevTest Labs 中提供)

> [!NOTE]
> 目前，Azure 實驗室服務僅支援從 Azure Marketplace 映像建立的 VM。 如果您想要在實驗室環境中使用自訂映像或建立其他 PaaS 資源，請使用 DevTest 實驗室。 如需詳細資訊，請參閱[在 DevTest 實驗室中建立自訂映像](devtest-lab-create-custom-image-from-vm-using-portal.md)及[使用 Resource Manager 範本建立實驗室環境](devtest-lab-create-environment-from-arm.md)。 

## <a name="scenarios"></a>案例
以下為一些 Azure 實驗服務所支援的案例： 

### <a name="set-up-a-resizable-computer-lab-in-the-cloud-for-your-classroom"></a>在雲端為您的教室設定可調整大小的電腦實驗室  

- 建立受控教室實驗室。 您只需將自己的需求告訴實驗室服務，它便會為您建立並管理實驗室的基礎結構，讓您可以專注在課堂的課程上，而不需處理實驗室的技術性細節。 
- 為學生提供具有虛擬機器的實驗室，且虛擬機器已針對課堂需求來設定。 給予每個學生有限的時間，以使用 VM 來完成課堂作業。  
- 將學校的實體電腦實驗室移至雲端。 以您在實驗室上設定的使用量上限和成本閾值為限，以自動調整 VM 數目。 
- 在結束之後，只需按一下便可以刪除實驗室。 

### <a name="use-devtest-labs-for-development-environments"></a>針對開發環境使用 DevTest Labs 
Azure DevTest Labs 可以用來實作許多重要案例，但是其中一個主要案例是使用 DevTest Labs 來託管開發人員的開發電腦。 在此案例中，DevTest Labs 提供以下優點： 

- 可讓開發人員快速視需要佈建開發電腦。
- 使用可重複使用的範本及構件來佈建 Windows 和 Linux 環境。
- 開發人員可以隨時輕鬆自訂他們的開發電腦。
- 系統管理員可以透過確保開發人員無法取得超過其開發需求的 VM 數目，並使 VM 在未使用的情況下關閉來控制成本。 

如需詳細資訊，請參閱[將 DevTest Labs 用於開發](devtest-lab-developer-lab.md)。 

### <a name="use-devtest-labs-for-test-environments"></a>針對測試環境使用 DevTest Labs
您可以使用 Azure DevTest Labs 實作許多重要案例，但是主要案例要有關使用 DevTest Labs 來託管測試人員的電腦。 在此案例中，DevTest Labs 提供以下優點：

- 測試人員可以利用可重複使用的範本和構件，快速佈建 Windows 和 Linux 環境，來測試其最新版的應用程式。
- 測試人員可以佈建多個測試代理程式，相應增加其負載測試。
- 系統管理員可以透過確保測試人員無法取得超過其測試需求的 VM 數目，並使 VM 在未使用的情況下關閉來控制成本。

如需詳細資訊，請參閱[將 DevTest Labs 用於測試](devtest-lab-test-env.md)。

## <a name="user-profiles"></a>使用者設定檔
本文描述 Azure 實驗室服務中各種不同的使用者設定檔。 

### <a name="lab-account-owner"></a>實驗室帳戶擁有者
通常會是擁有 Azure 訂用帳戶的組織雲端資源 IT 系統管理員，並會執行下列工作：   

- 為組織設定實驗室帳戶。
- 管理並設定所有實驗室的原則。
- 給予組織中人員在實驗室帳戶下建立實驗室的權限。

### <a name="lab-creator"></a>實驗室建立者 
通常，開發領導人/主管、教師、Hackathon (駭客松) 主持人、線上訓練人員等使用者，會在實驗室帳戶下建立實驗室。 實驗室建立者會執行下列工作： 

- 建立實驗室。
- 在實驗室中建立虛擬機器。 
- 在虛擬機器上安裝適當的軟體。
- 指定可以存取實驗室的人員。
- 將實驗室的連結提供給實驗室使用者。

### <a name="lab-user"></a>實驗室使用者
實驗室使用者會執行下列工作：

- 使用實驗室使用者從實驗室建立者處收到的註冊連結來註冊至實驗室。 
- 連線至實驗室中的虛擬機器，並使用它進行開發、測試或完成課堂作業。 

## <a name="next-steps"></a>後續步驟
使用 Azure 實驗室服務以設定實驗室的入門：

- [設定教室實驗室](classroom-labs/tutorial-setup-classroom-lab.md)
- [設定實驗室](tutorial-create-custom-lab.md)
