---
title: 使用 Habitat 將應用程式部署至 Azure
description: 了解如何以一致的方式將應用程式部署至 Azure 虛擬機器和容器
keywords: azure, chef, devops, 虛擬機器, 概觀, 自動化, habitat
ms.service: virtual-machines-linux
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.date: 05/15/2018
ms.topic: article
ms.openlocfilehash: 73c6834eb53c0496fa673dd25ab90abc18a6a139
ms.sourcegitcommit: 96089449d17548263691d40e4f1e8f9557561197
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/17/2018
ms.locfileid: "34267006"
---
# <a name="use-habitat-to-deploy-your-application-to-azure"></a>使用 Habitat 將應用程式部署至 Azure
[Habitat](https://www.habitat.sh/) 是此類開放原始碼的先驅，提供了管理應用程式的全新方法。 Habitat 可以讓應用程式及其自動化成為部署的單位。 以輕量型的 "habitat" 包裝應用程式後，執行階段環境 (無論是容器、裸機或 PaaS) 就不再是重點，也不會再限制應用程式。 

本文說明使用 Habitat 的優點。

## <a name="support-for-the-modern-application"></a>對於現代應用程式的支援
Habitat 套件包含應用程式在其整個生命週期執行所需的所有項目。 Habitat 的核心元件是：
- 封裝格式 - Habitat 套件中的應用程式是不可部分完成的、固定的及可稽核的。
- Habitat 監督員會在知曉套件的對等關係、升級策略及安全性原則的情形下，執行應用程式套件。 Habitat 監督員可在任何環境下設定及管理應用程式。
- Habitat 生態系統也會提供建置服務 (該服務採用 Habitat 建置計劃)、建立 Habitat 套件並且將其發行至 Depot。

## <a name="run-any-application-anywhere"></a>在任何位置執行任何應用程式
有了 Habitat，應用程式就可以不經修改，在任何執行階段環境下執行。 這包括從裸機和虛擬機器到容器 (例如 Docker)、叢集管理系統 (例如 Mesosphere 或 Kubernetes) 及 PaaS 系統 (例如 Pivotal Cloud Foundry) 的所有項目。

## <a name="easily-port-legacy-applications"></a>輕鬆地連接繼承應用程式
當繼承應用程式是包裝在 Habitat 套件中時，應用程式就與它們原始設計所在的環境無關。 套件可以快速地移到更現代的環境，例如雲端或容器。 此外，因為 Habitat 套件有標準、向外的介面，所以繼承應用程式會變得更容易管理。

## <a name="improve-the-container-experience"></a>改善容器體驗
Habitat 會降低在生產環境中管理容器的複雜度。 藉由讓容器內的應用程式組態自動化，Habitat 可以解決開發人員將容器型應用程式從開發環境移至生產環境時，所面對的挑戰。

## <a name="integrate-into-the-chef-devops-workflow"></a>整合至 Chef DevOps 工作流程
Habitat 專案是由 Chef 協辦。 Habitat 會利用 Chef 對基礎結構自動化的深厚經驗，為應用程式帶來所未有的自動化功能。 Chef 可提供對於 Habitat 的商務支援，並且確保 Habitat 與 Chef 傳遞之間的無縫整合，讓應用程式發行週期 (從開發到部署) 完全自動化。

## <a name="next-steps"></a>後續步驟
* [使用 Chef 在 Azure 上建立 Windows 虛擬機器](/azure/virtual-machines/windows/chef-automation)