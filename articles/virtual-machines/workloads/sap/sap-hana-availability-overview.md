---
title: Azure 虛擬機器上的SAP Hana 可用性 - 概觀 | Microsoft Docs
description: 說明 Azure 原生虛擬機器上的 SAP Hana 作業。
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/05/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7049a4b5159687ab928cda7ddc6b1a35959529ac
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38972102"
---
# <a name="sap-hana-high-availability-for-azure-virtual-machines"></a>Azure 虛擬機器的 SAP Hana 高可用性

您可以使用許多 Azure 功能，在 Azure 虛擬機器上部署像是 SAP Hana 的任務關鍵性資料庫。 本文提供的指引會引導您達成 Azure 虛擬機器中裝載之 SAP Hana 執行個體的可用性。 本文說明可以使用 Azure 基礎結構實作以提高 Azure 中 SAP Hana 可用性的幾個案例。 

## <a name="prerequisites"></a>先決條件

本文假設您已熟悉 Azure 上的基礎結構即服務 (IaaS) 基本知識，包括： 

- 如何透過 Azure 入口網站或 PowerShell 部署虛擬機器或虛擬網路。
- 使用 Azure 跨平台命令列介面 (Azure CLI)，包括使用 JavaScript 物件通知 (JSON) 範本的選項。

本文也假設您已熟悉安裝 SAP Hana 執行個體和管理及操作 SAP Hana 執行個體。 務必要熟悉 HANA 系統複寫的設定和作業。 包括像是 SAP Hana 資料庫備份和還原等工作。

下列文章提供在 Azure 中使用 SAP Hana 的良好概觀：

- [在 Azure VM 上手動安裝單一執行個體 SAP HANA](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-get-started)
- [在 Azure 虛擬機器中設定 SAP Hana 系統複寫](sap-hana-high-availability.md)
- [在 Azure虛擬機器上備份 SAP Hana](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-guide)

利用下列文章來熟悉 SAP Hana 也是很好的想法：

- [SAP Hana 的高可用性](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/en-US/6d252db7cdd044d19ad85b46e6c294a4.html)
- [常見問題集：SAP Hana 的高可用性](https://archive.sap.com/documents/docs/DOC-66702)
- [執行 SAP Hana 的系統複寫](https://archive.sap.com/documents/docs/DOC-47702)
- [SAP Hana 2.0 SPS 01 新增功能：高可用性](https://blogs.sap.com/2017/05/15/sap-hana-2.0-sps-01-whats-new-high-availability-by-the-sap-hana-academy/)
- [SAP Hana 系統複寫的網路建議](https://www.sap.com/documents/2016/06/18079a1c-767c-0010-82c7-eda71af511fa.html)
- [SAP Hana 系統複寫](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html)
- [SAP Hana 服務自動重新啟動](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/cf10efba8bea4e81b1dc1907ecc652d3.html)
- [設定 SAP Hana 系統複寫](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/676844172c2442f0bf6c8b080db05ae7.html)

除了熟悉在 Azure 中部署虛擬機器之外，在您於 Azure 中定義可用性架構之前，我們建議您閱讀[在 Azure 中管理 Windows 虛擬機器的可用性](https://docs.microsoft.com/azure/virtual-machines/windows/manage-availability)。

## <a name="service-level-agreements-for-azure-components"></a>適用於 Azure 元件的服務等級協定

Azure 對不同的元件 (例如網路、儲存體和虛擬機器) 有不同的可用性 SLA。 所有的 SLA 都會記錄下來。 如需詳細資訊，請參閱 [Microsoft Azure 服務等級協定](https://azure.microsoft.com/support/legal/sla/)。 

[虛擬機器的 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_6/) 說明兩個不同組態的兩個不同 SLA：

- 針對作業系統磁碟和所有資料磁碟使用 [Azure 進階儲存體](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage)的單一虛擬機器。 此選項提供每月 99.9% 的執行時間。
- 在 [Azure 可用性設定組](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets)中組織的多個 (至少兩個) 虛擬機器。 此選項提供每月 99.95% 的執行時間。

針對 Azure 元件可以提供的 SLA 測量您的可用性需求。 然後，針對 SAP Hana 選擇您的案例，以達到所需的可用性層級。

## <a name="next-steps"></a>後續步驟

- 深入了解[單一 Azure 區域中的 SAP Hana 可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-one-region)。
- 深入了解[跨 Azure 區域的 SAP Hana 可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-across-regions)。 















  


