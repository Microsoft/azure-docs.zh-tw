---
title: 以資料科學虛擬機器為基礎的團隊環境簡介 - Azure | Microsoft Docs
description: 在企業團隊環境中部署「資料科學 VM」的模式。
keywords: 深度學習, 資料科學工具, 資料科學虛擬機器, 地理空間分析, 小組資料科學流程
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/08/2018
ms.author: gokuma
ms.openlocfilehash: ea8d53ee71e9272167a045e2ea1780828f974d30
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45573586"
---
# <a name="data-science-virtual-machine-based-team-analytics-and-ai-environment"></a>以資料科學虛擬機器為基礎的團隊分析和 AI 環境 
[資料科學虛擬機器](overview.md) (DSVM) 在 Azure 平台中提供豐富的環境，其中包含了適用於人工智慧 (AI) 和資料分析的預先建立軟體。 

傳統上，DSVM 已用來作為個別的分析電腦。 個別資料科學家可以對其預先建立的分析環境持有共同看法，藉以提升生產力。 當大型分析團隊在規劃其資料科學家和 AI 開發人員的分析環境時，其中一個重複出現的主題便是用於開發與實驗的共用分析基礎結構。 此基礎結構的管理方式與企業 IT 原則一致，這些原則同時也協助跨資料科學/分析團隊的共同作業和一致性。 

共用的基礎結構也可讓 IT 能夠更充分地利用分析環境。 有些組織將團隊型資料科學/分析基礎結構稱為「分析沙箱」。 它可讓資料科學家存取各種資料資產來快速了解資料、執行實驗、驗證假設及建置預測模型，而不會影響到生產環境。 

由於 DSVM 在 Azure 基礎結構層級運作，因此 IT 系統管理員立即就可以將 DSVM 的運作方式設定成符合企業的 IT 原則。 DSVM 既提供可實作各種共用基礎結構的完整彈性，又能控管對公司資料資產的存取。 

本節將討論可用來將 DSVM 部署成團隊型資料科學基礎結構的一些模式與指導方針。 這些模式的構成要素來自 Azure 基礎結構即服務 (IaaS)，因此可適用於任何 Azure VM。 此系列文章主要著重在將這些標準 Azure 基礎結構功能應用在「資料科學 VM」上。 

企業團隊分析環境的部分重要建構元素是：

* [資料科學虛擬機器的自動調整規模集區](dsvm-pools.md)
* [從集區中的任何 DSVM 使用通用身分識別並存取工作區](dsvm-common-identity.md)
* [保護對資料來源的存取](dsvm-secure-access-keys.md)


此系列文章提供前述各個項目的指引和指標。 它並未涵蓋在大型企業設定中部署 DSVM 的所有考量和需求。 以下是您在企業中實作 DSVM 執行個體時可使用的其他 Azure 文件： 

* [網路安全性](https://docs.microsoft.com/azure/security/azure-network-security)
* [監視](https://docs.microsoft.com/azure/virtual-machines/windows/monitor)和[管理](https://docs.microsoft.com/azure/virtual-machines/windows/maintenance-and-updates)
* [記錄與稽核](https://docs.microsoft.com/azure/security/azure-log-audit)
* [角色型存取控制](https://docs.microsoft.com/azure/role-based-access-control/overview)
* [原則設定和強制執行](https://docs.microsoft.com/azure/azure-policy/azure-policy-introduction)
* [反惡意程式碼](https://docs.microsoft.com/azure/security/azure-security-antimalware)
* [加密](https://docs.microsoft.com/azure/virtual-machines/windows/encrypt-disks)
* [資料探索和控管](https://docs.microsoft.com/azure/data-catalog/)

[Azure 架構中心](https://docs.microsoft.com/azure/architecture/)提供用於建置和管理雲端式分析基礎結構的詳細端對端架構和模式。 
