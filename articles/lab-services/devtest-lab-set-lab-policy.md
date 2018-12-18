---
title: 在 Azure DevTest Labs 中管理實驗室原則 | Microsoft Docs
description: 了解如何定義實驗室原則，例如 VM 大小、每位使用者的 VM 數目上限，以及自動關機。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
ms.assetid: 7756aa64-49ca-45a0-9f90-0fd101c7be85
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: 212afbd605e3a16da7be2c04492ec41875ff5b75
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38666834"
---
# <a name="manage-all-policies-for-a-lab-in-azure-devtest-labs"></a>在 Azure DevTest Labs 中管理實驗室的所有原則

Azure DevTest Labs 讓您管理每個實驗室的原則 (設定)，以控制實驗室的成本並儘可能避免浪費。 本文逐步地詳細說明如何設定每個原則。  

## <a name="set-allowed-virtual-machine-sizes"></a>設定允許的虛擬機器大小
設定允許的 VM 大小的原則有助於將實驗室的成本浪費降至最低，方式是讓您指定實驗室中允許的 VM 大小。 如果啟用此原則，就只能使用此清單中的 VM 大小來建立 VM。

1. 在 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)中，選取實驗室，然後選取 [設定與原則]。

    ![存取實驗室的設定與原則](./media/devtest-lab-set-lab-policy/policies-menu.png)

1. 在實驗室的 [設定與原則] 窗格上，選取 [允許的虛擬機器大小]。
   
    ![允許的虛擬機器大小](./media/devtest-lab-set-lab-policy/allowed-vm-sizes.png)

1. 選取 [開啟] 來啟用此原則，以及選取 [關閉] 來停用它。

1. 如果啟用這個原則，請選取一或多個可在實驗室中建立的 VM 大小。

1. 選取 [ **儲存**]。

## <a name="set-virtual-machines-per-user"></a>設定每位使用者的虛擬機器數目
[每位使用者的虛擬機器數] 原則可讓您指定個別使用者可以建立的 VM 數目上限。 當達到使用者限制時，如果使用者嘗試建立或宣告 VM，就會顯示錯誤訊息，指出無法建立/宣告 VM。 

1. 在實驗室的 [設定與原則] 窗格上，選取 [每位使用者的虛擬機器數]。
   
    ![每位使用者的虛擬機器數目](./media/devtest-lab-set-lab-policy/max-vms-per-user.png)

1. 選取 [是]，以限制每個使用者的 VM 數目。 如果不想限制每個使用者的 VM 數目，請選取 [否]。 如果您選取 [是]，請輸入數值，指出使用者可建立的 VM 數目上限。 

1. 選取 [是]，以限制可使用 SSD (固態硬碟) 的 VM 數目。 如果不想限制可使用 SSD 的 VM 數目，請選取 [否]。 如果您選取 [是]，請輸入一個值，指出可使用 SSD 建立的 VM 數目上限。 

1. 選取 [ **儲存**]。

## <a name="set-virtual-machines-per-lab"></a>設定每個實驗室的虛擬機器數目
[每個實驗室的虛擬機器數] 原則可讓您指定可為目前實驗室建立的 VM 數目上限。 當達到實驗室限制時，如果使用者嘗試建立 VM，就會顯示錯誤訊息，指出無法建立 VM。 

1. 在實驗室的 [設定與原則] 窗格上，選取 [每個實驗室的虛擬機器數]。
   
    ![每個實驗室的虛擬機器數目](./media/devtest-lab-set-lab-policy/max-vms-per-lab.png)

1. 選取 [是]，以限制每個實驗室的 VM 數目。 如果不想限制每個實驗室的 VM 數目，請選取 [否]。 如果您選取 [是]，請輸入數值，指出使用者可建立的 VM 數目上限。 

1. 選取 [是]，以限制可使用 SSD (固態硬碟) 的 VM 數目。 如果不想限制可使用 SSD 的 VM 數目，請選取 [否]。 如果您選取 [是]，請輸入一個值，指出可使用 SSD 建立的 VM 數目上限。 

1. 選取 [ **儲存**]。

## <a name="set-auto-shutdown"></a>設定自動關機
[自動關機] 原則有助於將實驗室的成本浪費降至最低，方式是讓您指定這個實驗室之 VM 的關機時間。

1. 在實驗室的 [設定與原則] 窗格上，選取 [自動關機]。
   
    ![自動關機](./media/devtest-lab-set-lab-policy/auto-shutdown.png)

1. 選取 [開啟] 來啟用此原則，以及選取 [關閉] 來停用它。

1. 如果您啟用這個原則，請指定時間 (和時區) 以關閉目前實驗室中的所有 VM。

1. 對於在指定的自動關機時間之前 15 分鐘傳送通知的選項中，指定 [是] 或 [否]。 如果您選擇 [是]，請輸入 Webhook URL 端點或電子郵件地址，指定您要在哪裡張貼或傳送通知。 使用者會收到通知，並且給予延遲關機的選項。

   如需 webhook 的詳細資訊，請參閱[建立 webhook 或 API Azure 函式](../azure-functions/functions-create-a-web-hook-or-api-function.md)。 

1. 選取 [ **儲存**]。

根據預設，這個原則一經啟用，就會套用到目前實驗室的所有 VM。 若要從特定的 VM 移除這項設定，請開啟 VM 的管理窗格並變更其 [自動關機] 設定。

## <a name="set-auto-start"></a>設定自動啟動
[自動啟動] 原則可讓您指定目前實驗室中的 VM 應該於何時啟動。  

1. 在實驗室的 [設定與原則] 窗格上，選取 [自動啟動]。
   
    ![自動啟動](./media/devtest-lab-set-lab-policy/auto-start.png)

2. 選取 [開啟] 來啟用此原則，以及選取 [關閉] 來停用它。

3. 如果您啟用這個原則，請指定排定的啟動時間、時區和該時間適用於星期幾。 

4. 選取 [ **儲存**]。

這個原則一經啟用，就不會自動套用到目前實驗室中的任何 VM。 若要將這項設定套用至特定的 VM，請開啟 VM 的管理窗格並變更其 [自動啟動] 設定。

## <a name="set-expiration-date"></a>設定到期日期
當您[建立 VM](devtest-lab-add-vm.md)時，可以設定到期日期。 在 [進階設定] 中，選擇行事曆圖示來指定要自動刪除 VM 的日期。 根據預設，VM 永遠不會到期。

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>後續步驟
實驗室一旦定義並套用了各種 VM 原則設定，接下來就要嘗試一些作業：

* [了解共用 IP 位址](devtest-lab-shared-ip.md) - 說明 DevTest Labs 中如何使用共用 IP 位址，將需要連線至您的實驗室 VM 的公用 IP 位址數目減到最少。
* [設定成本管理](devtest-lab-configure-cost-management.md) - 示範如何使用 [每月估計成本趨勢] 圖表，  
  來檢視本月到目前為止的估計成本，以及預計的月底成本。
* [建立自訂映像](devtest-lab-create-template.md) - 當您建立 VM 時，您要指定一個基本映像，它可以是自訂映像或 Marketplace 映像。 本文會示範如何從 VHD 檔案建立自訂的映像。
* [設定 Marketplace 映像](devtest-lab-configure-marketplace-images.md) - Azure DevTest Labs 支援根據 Azure Marketplace 映像建立 VM。 本文會示範在實驗室中建立 VM 時，如何指定可以使用哪些 Azure Marketplace 映像 (如果有的話)。
* [在實驗室中建立 VM](devtest-lab-add-vm.md) - 示範如何從基本映像 (自訂或 Marketplace) 建立 VM，以及如何使用 VM 中的構件。

