---
title: 在 Azure 中設定管理解決方案目標 | Microsoft Docs
description: 設定管理解決方案目標，可讓您將管理解決方案限制為一組特定的代理程式。  本文說明如何建立範圍設定，並將它套用至解決方案。
services: monitoring
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.assetid: 1f054a4e-6243-4a66-a62a-0031adb750d8
ms.service: monitoring
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/27/2017
ms.author: bwren
ms.openlocfilehash: 65585e6c09def23101d9735c8b9c719d213938ac
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
ms.locfileid: "33887398"
---
# <a name="targeting-management-solutions-in-azure-preview"></a>在 Azure 中設定管理解決方案目標 (預覽)
當您將管理解決方案新增至訂用帳戶時，預設會將該解決方案自動部署到與您 Log Analytics 工作區連線的所有 Windows 與 Linux 代理程式。  您可能會想要管理您的成本，並透過將解決方案限制在一組特定的代理程式，來限制針對該解決方案所收集的資料量。  本文說明如何使用**設定解決方案目標**，此功能可讓您將範圍套用至解決方案。

## <a name="how-to-target-a-solution"></a>如何為解決方案設定目標
為解決方案設定目標有三個步驟，如下列各節中所述。 


### <a name="1-create-a-computer-group"></a>1.建立電腦群組
您可以透過在 Log Analytics 中建立[電腦群組](../log-analytics/log-analytics-computer-groups.md)，來指定您想要包含在範圍內的電腦。  電腦群組可以是根據記錄搜尋，或者從其他來源匯入 (例如 Active Directory 或 WSUS 群組)。 [如同下面所述](#solutions-and-agents-that-cant-be-targeted)，只有直接連線至 Log Analytics 的電腦才會包含在範圍內。

一旦您在工作區中建立電腦群組，便可將它包含在可套用至一或多個解決方案的範圍設定中。
 
 
 ### <a name="2-create-a-scope-configuration"></a>2.建立範圍設定
 「範圍設定」包含一或多個電腦群組，可以套用至一或多個解決方案。 
 
 使用下列程序建立範圍設定。  

 1. 在 Azure 入口網站中，瀏覽至 [Log Analytics] 並選取您的工作區。
 2. 在工作區內容中的 [工作區資料來源] 底下，選取 [範圍設定]。
 3. 按一下 [新增] 以建立新的範圍設定。
 4. 輸入範圍設定的 [名稱]。
 5. 按一下 [選取電腦群組]。
 6. 選取您所建立的電腦群組 (並選擇性地選取其他群組) 以新增至設定。  按一下 [選取] 。  
 6. 按一下 [確定] 以建立範圍設定。 


 ### <a name="3-apply-the-scope-configuration-to-a-solution"></a>3.將範圍設定套用至解決方案。
一旦您有範圍設定，便可以將它套用至一或多個解決方案。  請注意，雖然單一範圍設定可以搭配多個解決方案使用，但每個解決方案僅可以使用一個範圍設定。

使用下列程序套用範圍設定。  

 1. 在 Azure 入口網站中，瀏覽至 [Log Analytics] 並選取您的工作區。
 2. 在工作區內容中，選取 [解決方案]。
 3. 按一下您想要設定範圍的解決方案。
 4. 在解決方案內容中的 [工作區資料來源] 底下，選取 [設定解決方案目標]。  如果該選項無法使用，便代表[此解決方案無法設定目標](#solutions-and-agents-that-cant-be-targeted)。
 5. 按一下 [新增範圍設定]。  如果您已經將設定套用到此解決方案，則此選項將無法使用。  您必須移除現有設定，才能新增其他設定。
 6. 按一下您所建立的範圍設定。
 7. 查看設定的 [狀態]，以確定它顯示 [成功]。  如果狀態顯示發生錯誤，請按一下設定右側的省略符號，並選取 [編輯範圍設定] 以進行變更。

## <a name="solutions-and-agents-that-cant-be-targeted"></a>無法設定目標的解決方案和代理程式
下列是無法搭配「設定解決方案目標」使用的代理程式和解決方案準則。

- 「設定解決方案目標」僅適用於會部署至代理程式的解決方案。
- 「設定解決方案目標」僅適用於 Microsoft 提供的解決方案。  不適用於[由您或合作夥伴所建立的](monitoring-solutions-creating.md)解決方案。
- 您只能篩選掉直接連線至 Log Analytics 的代理程式。  解決方案會自動部署到屬於已連線 Operations Manager 管理群組之一部分的所有代理程式，不論它們是否包含在範圍設定內。

### <a name="exceptions"></a>例外狀況
「設定解決方案目標」無法搭配下列解決方案使用，即使它們符合上述準則。

- 代理程式健康狀態評估

## <a name="next-steps"></a>後續步驟
- 在[將 Azure Log Analytics 管理解決方案新增至您的工作區](../log-analytics/log-analytics-add-solutions.md)中深入了解管理解決方案，包含可在您的環境中安裝的可用解決方案。
- 在 [Log Analytics 記錄檔搜尋中的電腦群組](../log-analytics/log-analytics-computer-groups.md)中深入了解建立電腦群組。