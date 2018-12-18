---
title: 指定 Azure DevTest Labs 的必要構件 | Microsoft Docs
description: 了解如何指定在實驗室中的虛擬機器 (VM) 上安裝任何使用者所選構件前，必須先安裝的必要構件。
services: devtest-lab,virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: devtest-lab
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/23/2018
ms.author: spelluru
ms.openlocfilehash: a739b958ad60e39c38e81ce887edf68349340bb0
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39295211"
---
# <a name="specify-mandatory-artifacts-for-your-lab-in-azure-devtest-labs"></a>針對 Azure DevTest Labs 中的實驗室指定必要構件
身為實驗室的擁有者，您可以指定必要構件，以套用至實驗室中建立的每個機器。 假設您想讓實驗室中的每一部機器都連線至您的公司網路。 在此情況下，每位實驗室使用者都必須在虛擬機器建立期間，新增網域加入構件，以確保他們的機器可連線到公司網域。 換句話說，實驗室使用者基本上必須重新建立機器，以免他們忘記在機器上套用必要的構件。 身為實驗室擁有者，您可以讓網域加入構件成為實驗室中的必要構件。 此步驟可確保每部機器都會連線到公司網路，並且可為您的實驗室使用者節省時間和勞力。
 
其他必要構件可以是您團隊使用的常見工具，或是每部機器必須預先具備的平台相關安全性套件等等。簡單地說，您實驗室中每部機器必須具備的任何常見軟體，都會成為必要構件。 如果您從已套用必要構件的機器中建立自訂映像，然後從該映像中建立全新的機器，則這些必要構件會在機器建立期間重新套用到此機器上。 此行為也表示，即使是舊的自訂映像，每次您從中建立機器時，最新版的必要構件還是會在建立流程期間套用到機器上。 
 
沒有任何參數的構件才可成為必要構件。 實驗室使用者不需要在建立實驗室期間輸入其他參數，這樣可簡化 VM 建立程序。 

## <a name="specify-mandatory-artifacts"></a>指定必要的成品
您可以分別為 Windows 和 Linux 機器選取必要構件。 您也可以根據想要的構件套用順序，來重新排序這些構件。 

1. 在實驗室首頁上，選取 [設定] 下方的 [組態和原則]。 
3. 在 [外部資源] 下方選取 [必要構件]。 
4. 在 [Windows] 區段或 [Linux] 區段中選取 [編輯]。 此範例使用 [Windows] 選項。 

    ![必要構件頁面 - 編輯按鈕](media/devtest-lab-mandatory-artifacts/mandatory-artifacts-edit-button.png)
4. 選取構件。 此範例使用 [7-zip] 選項。 
5. 在 [新增構件] 頁面上，選取 [新增]。 

    ![必要構件頁面 - 新增 7-zip](media/devtest-lab-mandatory-artifacts/add-seven-zip.png)
6. 若要新增其他構件，選取發行項，然後選取 [新增]。 此範例會將 **Chrome** 新增為第二個必要構件。

    ![必要構件頁面 - 新增 Chrome](media/devtest-lab-mandatory-artifacts/add-chrome.png)
7. 在 [必要構件] 頁面上，會有訊息顯示已選取的構件數量。 如果您按一下該訊息，您會看到選取的構件。 選取 [儲存] 來進行儲存。 

    ![必要構件頁面 - 儲存構件](media/devtest-lab-mandatory-artifacts/save-artifacts.png)
8. 重複步驟以指定適用於 Linux VM 的必要構件。 
    
    ![必要構件頁面 - Windows 和 Linux 的構件](media/devtest-lab-mandatory-artifacts/windows-linux-artifacts.png)
9. 若要從清單中**刪除**構件，請選取資料列末端的 [...]\(省略符號\)，然後選取 [刪除]。 
10. 若要**重新排序**清單中的構件，請將滑鼠停留在構件的上方，選取顯示在資料列開頭的 [...]\(省略符號\) ，然後將項目拖曳到新位置。 
11. 若要在實驗室中儲存必要構件，請選取 [儲存]。 

    ![必要構件頁面 - 在實驗室中儲存構件](media/devtest-lab-mandatory-artifacts/save-to-lab.png)
12. 關閉 [組態和原則]頁面 (選取右上角的 [X])，即可回到實驗室首頁。  

## <a name="delete-a-mandatory-artifact"></a>刪除必要構件
若要從實驗室刪除必要構件，請執行下列動作： 

1. 選取 [設定] 下方的 [組態和原則]。 
2. 在 [外部資源] 下方選取 [必要構件]。 
3. 在 [Windows] 區段或 [Linux] 區段中選取 [編輯]。 此範例使用 [Windows] 選項。 
4. 在頂端選取包含必要構件數目的訊息。 

    ![必要構件頁面 - 選取訊息](media/devtest-lab-mandatory-artifacts/select-message-artifacts.png)
5. 在 [選取的構件] 頁面上，針對要刪除的構件選取 [...]\(省略符號\)，然後選取 [移除]。 
    
    ![必要構件頁面 - 移除構件](media/devtest-lab-mandatory-artifacts/remove-artifact.png)
6. 選取 [確定] 以關閉 [選取的構件] 頁面。 
7. 在 [必要構件] 頁面上選取 [儲存]。
8. 如有需要，針對 **Linux** 映像重複這些步驟。 
9. 選取 [儲存] 以儲存對實驗室所做的所有變更。 

## <a name="view-mandatory-artifacts-when-creating-a-vm"></a>在建立 VM 時檢視必要構件
現在，您如果是實驗室使用者，在實驗室中建立 VM 時，即可檢視必要構件的清單。 您無法編輯或刪除實驗室擁有者在實驗室中設定的必要構件。

1. 在您的實驗室首頁上，從功能表中選取 [概觀]。
2. 若要將 VM 新增至實驗室中，請選取 [+新增]。 
3. 選取 [基礎映像]。 此範例使用 **Windows Server 1709 版**。
4. 請注意，您會看到有關**構件**的訊息，其中顯示已選取的必要構件數目。 
5. 選取 [構件]。 
6. 確認實驗室的 [組態與原則] 中，有顯示您指定的**必要構件**。 

    ![建立 VM - 必要構件](media/devtest-lab-mandatory-artifacts/create-vm-artifacts.png)

## <a name="next-steps"></a>後續步驟
* 了解如何 [將 Git 構件儲存機制加入實驗室](devtest-lab-add-artifact-repo.md)。

