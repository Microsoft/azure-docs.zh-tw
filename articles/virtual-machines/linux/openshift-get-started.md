---
title: Azure 上的 OpenShift 概觀 | Microsoft Docs
description: Azure 中的 OpenShift 概觀。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: haroldw
manager: najoshi
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: ''
ms.author: haroldw
ms.openlocfilehash: e3ab060c1cea28f83c18dc89aeea7716ec86572a
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43190338"
---
# <a name="openshift-in-azure"></a>Azure 中的 OpenShift

OpenShift 為企業帶來 Docker 和 Kubernetes 開放且可延伸的容器應用程式平台。  

OpenShift 包含容器協調流程與管理的 Kubernetes。 它加入以開發人員和以作業為中心的工具，可達成：

- 應用程式快速開發。
- 部署及調整更容易。
- 小組和應用程式的長期生命週期維護。

有多個版本的 OpenShift 可用：

- OKD (先前稱為 OpenShift Origin)
- Red Hat OpenShift 容器平台
- OpenShift 線上
- OpenShift 專用

在本文涵蓋的四個版本中，只有兩個可供客戶部署至 Azure：OpenShift Origin 和 OpenShift 容器平台。

## <a name="okd-formerly-openshift-origin"></a>OKD (先前稱為 OpenShift Origin)

OKD 是由社群支援的 OpenShift [開放原始碼](https://www.okd.io/)上游專案。 OKD 可以安裝在 CentOS 或 Red Hat Enterprise Linux (RHEL) 上。

## <a name="openshift-container-platform"></a>Red Hat OpenShift 容器平台

「容器平台」是由 Red Hat 提供及支援、符合企業需求的[商業版](https://www.openshift.com)。 使用這個版本，客戶要購買 OpenShift 容器平台的必要權利，並負責安裝與管理整個基礎結構。

因為客戶「擁有」整個平台，因此他們可以將其安裝在內部部署資料中心或公用雲端 (如 Azure、AWS、Google)。

## <a name="openshift-online"></a>OpenShift 線上

「線上」使用容器平台，是由 Red Hat 管理的*多租用戶* OpenShift。 Red Hat 會管理所有的基礎結構 (例如 VM、OpenShift 叢集、網路、儲存體)。 

使用這個版本，客戶要部署容器，但是對於執行容器的主機沒有控制權。 因為「線上」為多租用戶，容器可能會和其他客戶的容器放在相同 VM 主機上。 成本按每個容器計算。

## <a name="openshift-dedicated"></a>OpenShift 專用

「專用」使用容器平台，是由 Red Hat 管理的*單一租用戶* OpenShift。 Red Hat 會管理所有的基礎結構 (VM、OpenShift 叢集、網路、儲存體等)。 叢集由一位客戶所特有，並在公用雲端 (如 AWS 或 Google，搭配 2018 年初推出的 Azure) 中執行。 初始叢集包含四個應用程式節點，每年 48000 美元 (先付款)。

## <a name="next-steps"></a>後續步驟

- [在 Azure 中設定 OpenShift 的一般必要條件](./openshift-prerequisites.md)
- [在 Azure 中部署 OpenShift Origin](./openshift-origin.md)
- [在 Azure 中部署 OpenShift 容器平台](./openshift-container-platform.md)
- [部署後工作](./openshift-post-deployment.md)
- [針對 OpenShift 部署進行疑難排解](./openshift-troubleshooting.md)
