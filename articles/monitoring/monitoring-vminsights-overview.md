---
title: 適用於 VM 的 Azure 監視器是什麼？ | Microsoft Docs
description: 適用於 VM 的 Azure 監視器是結合了 Azure VM 作業系統健康情況與效能監控的 Azure 監視器功能，其會自動探索應用程式元件和與其他資源的相依性，並且會對應它們之間的通訊。 本文提供概觀。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: ''
ms.service: azure-monitor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: magoedte
ms.openlocfilehash: 26fcc3eb78af53360cca57382b4c06b017f36c0e
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063263"
---
# <a name="what-is-azure-monitor-for-vms"></a>適用於 VM 的 Azure 監視器是什麼？

適用於 VM 的 Azure 監視器可藉由分析 Windows 和 Linux VM 的效能和健康情況 (包括其不同的程序以及其他資源和外部程序的互連相依性)，大規模監視 Azure 虛擬機器 (VM)。 此解決方案包含針對在內部部署環境或其他雲端提供者上裝載的 VM，監視效能和應用程式相依性的支援。  它包含三個主要功能，以提供此深入解析：

* 執行 Windows 和 Linux 作業系統的 Azure VM 的邏輯元件，是在滿足評估條件時，依據一組預先設定的健康情況準則與警示所測量的。  
* 在預先定義的趨勢效能圖表中，會收集並顯示客體 VM 作業系統的核心效能計量，從處理器、記憶體、磁碟和網路介面卡都包含在內。
* 相依性地圖會顯示在多個資源群組與訂用帳戶中已找到與 VM 互連的元件。  

這些功能分成三個檢視方塊：

* 健康情況
* 效能
* 對應

>[!NOTE]
>目前，僅針對 Azure 虛擬機器提供健康情況功能。
>

與 Log Analytics 整合，可提供強大的彙總、篩選，以及隨著時間對資料執行趨勢分析的能力。 單獨使用 Azure 監視器、服務對應，或是 Log Analytics，並無法達到對您工作負載的全面性監視。  

您可以直接在虛擬機器的單一 VM 內容中檢視此資料，或是使用 Azure 監視器，它根據每個功能的下列檢視方塊，為您的 VM 提供彙總檢視：

* 健康情況 - 與資源群組有關的 VM
* 對應與效能 - 設定為回報給特定 Log Analytics 工作區的 VM

![從入口網站查看虛擬機器深入見解檢視方塊](./media/monitoring-vminsights-overview/vminsights-azmon-directvm-01.png)

透過找出關鍵作業系統事件和效能瓶頸、網路問題，以及了解問題是否與其他相依性有關，DevOps 可以有效地提供重要應用程式的可預測效能和可用性。  

## <a name="data-usage"></a>資料使用量 

當您將適用於 VM 的 Azure 監視器上線時，您的 VM 所收集的資料會內嵌及儲存在 Azure 監視器中。  適用於 VM 的 Azure 監視器的收費是依據內嵌及保留的資料、監視到的健康情況準則計量時間序列數、所建立的警示規則、已傳送的通知、Azure 監視器[定價頁面](https://azure.microsoft.com/pricing/details/monitor/)上發佈的每個定價

記錄大小會因計數器的字串長度而異，而且可以增加邏輯磁碟和網路介面卡數目。  如果您已經有工作區，並且收集這些計數器，則不會收取任何重複費用。  如果您已經使用服務對應，您會看到的唯一變更是傳送到 Azure 監視器的額外連線資料。

## <a name="next-steps"></a>後續步驟
檢閱[將適用於 VM 的 Azure 監視器上線](monitoring-vminsights-onboard.md)，了解啟用虛擬機器監視的需求和方法。
