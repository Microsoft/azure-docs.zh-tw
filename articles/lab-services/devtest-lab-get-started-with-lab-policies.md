---
title: 在 Azure DevTest Labs 中管理基本實驗室原則 | Microsoft Docs
description: 了解如何在 DevTest Labs 中設定實驗室的一些基本原則 (設定)
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: 40b8fb360be7b08540e25886aaebe7f911607b6d
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33781827"
---
# <a name="manage-basic-policies-for-a-lab-in-azure-devtest-labs"></a>在 Azure DevTest Labs 中管理實驗室的基本原則

Azure DevTest Labs 可讓您管理每個實驗室的原則 (設定)，以控制實驗室的成本並儘可能避免浪費。 在本文中，您將從學習如何設定兩個最重要的原則來開始使用原則：限制單一使用者可建立或宣告的虛擬機器 (VM) 數目，以及設定自動關機。 若要了解如何設定每個實驗室原則，請參閱[在 Azure DevTest Labs 中定義實驗室原則](devtest-lab-set-lab-policy.md)。  

## <a name="accessing-a-labs-policies-in-azure-devtest-labs"></a>在 Azure DevTest Labs 中存取實驗室的原則
下列步驟將引導您完成在 Azure DevTest Labs 中設定實驗室原則的步驟︰

若要檢視 (及變更) 實驗室的原則，請依照下列步驟操作：

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。

1. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。

1. 從實驗室清單中，選取所需的實驗室。   

1. 選取 [組態和原則]。

    ![原則設定窗格](./media/devtest-lab-set-lab-policy/policies-menu.png)

1. [設定和原則] 窗格包含一個功能表，內含您可以指定的設定。 本文章只涵蓋 [每位使用者的虛擬機器數目]、[自動關機] 和 [自動啟動] 的設定。 若要深入了解其餘設定，請參閱[在 Azure DevTest Labs 中管理實驗室的所有原則](./devtest-lab-set-lab-policy.md)。 
   
## <a name="set-virtual-machines-per-user"></a>設定每位使用者的虛擬機器數目
[每位使用者的虛擬機器數目]  原則可讓您指定個別使用者可以建立的 VM 數目上限。 當達到使用者限制時，如果使用者嘗試建立或宣告 VM，就會顯示錯誤訊息，指出無法建立/宣告 VM。 

1. 在實驗室的 [組態和原則] 功能表上，選取 [每位使用者的虛擬機器數目]。
   
    ![每位使用者的虛擬機器數目](./media/devtest-lab-set-lab-policy/max-vms-per-user.png)

1. 選取 [是]，以限制每個使用者的 VM 數目。 如果不想限制每個使用者的 VM 數目，請選取 [否]。 如果您選取 [是]，請輸入數值，指出使用者可建立的 VM 數目上限。 

1. 選取 [是]，以限制可使用 SSD (固態硬碟) 的 VM 數目。 如果不想限制可使用 SSD 的 VM 數目，請選取 [否]。 如果您選取 [是]，請輸入一個值，指出可使用 SSD 建立的 VM 數目上限。 

1. 選取 [ **儲存**]。

## <a name="set-auto-shutdown"></a>設定自動關機
自動關機原則有助於將實驗室的成本浪費降至最低，方式是讓您指定這個實驗室中 VM 的關機時間。

1. 在實驗室的 [設定與原則] 窗格上，選取 [自動關機]。
   
    ![自動關機](./media/devtest-lab-set-lab-policy/auto-shutdown.png)

1. 選取 [開啟] 來啟用此原則，以及選取 [關閉] 來停用它。

1. 如果您啟用這個原則，請指定時間 (和時區) 以關閉目前實驗室中的所有 VM。

1. 對於在指定的自動關機時間之前 15 分鐘傳送通知的選項中，指定 [是] 或 [否]。 如果您選擇 [是]，請輸入 Webhook URL 端點或電子郵件地址，指定您要在哪裡張貼或傳送通知。 使用者會收到通知，並且給予延遲關機的選項。

   如需 webhook 的詳細資訊，請參閱[建立 webhook 或 API Azure 函式](../azure-functions/functions-create-a-web-hook-or-api-function.md)。 

1. 選取 [ **儲存**]。

根據預設，這個原則一經啟用，就會套用到目前實驗室的所有 VM。 若要從特定的 VM 移除這項設定，請開啟 VM 的管理窗格並變更其 [自動關機] 設定。

## <a name="set-auto-start"></a>設定自動啟動
自動啟動原則可讓您指定目前實驗室 VM 應該啟動的時間。  

1. 在實驗室的 [設定與原則] 窗格上，選取 [自動啟動]。
   
    ![自動啟動](./media/devtest-lab-set-lab-policy/auto-start.png)

2. 選取 [開啟] 來啟用此原則，以及選取 [關閉] 來停用它。

3. 如果您啟用這個原則，請指定排定的啟動時間、時區和該時間適用於星期幾。 

4. 選取 [ **儲存**]。

這個原則一經啟用，就不會自動套用到目前實驗室中的任何 VM。 若要將這項設定套用至現有的 VM，請開啟 VM 的管理窗格並變更其 [自動啟動] 設定。

## <a name="next-steps"></a>後續步驟

- [在 Azure DevTest Labs 中定義實驗室原則](devtest-lab-set-lab-policy.md) - 了解如何修改其他實驗室原則。
