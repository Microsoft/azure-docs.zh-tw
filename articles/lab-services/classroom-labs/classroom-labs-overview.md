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
ms.date: 05/21/2018
ms.author: spelluru
ms.openlocfilehash: e9c3cae7c7129cc489ddd38b5b2de18dd6f52e58
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34660480"
---
# <a name="introduction-to-classroom-labs"></a>教室實驗室簡介
Azure 實驗室服務可讓您在雲端中快速設定教室實驗室環境。 教育工作者可建立教室實驗室、佈建 Windows 或 Linux 虛擬機器、在教室中安裝必要的軟體和工具實驗室，然後使其可供學生使用。 教室中的學生可連線至實驗室中的虛擬機器 (VM)，並將這些機器用於專案、指派、教室練習等。 

教室實驗室受到 Azure 管理的受控實驗室。 服務本身會處理受控實驗室的所有基礎結構管理，從啟動虛擬機器 (VM) 以處理錯誤，到調整基礎結構的規模。 您指定您需要何種基礎結構，並安裝課堂所需的任何工具或軟體。 受控實驗室目前為預覽狀態。  

## <a name="scenarios"></a>案例
以下是 Azure 實驗室服務的教室實驗室支援的主要案例： 

### <a name="set-up-a-resizable-computer-lab-in-the-cloud-for-your-classroom"></a>在雲端為您的教室設定可調整大小的電腦實驗室  

- 建立受控教室實驗室。 您只需將自己的需求告訴實驗室服務，它便會為您建立並管理實驗室的基礎結構，讓您可以專注在課堂的課程上，而不需處理實驗室的技術性細節。 
- 為學生提供具有虛擬機器的實驗室，且虛擬機器已針對課堂需求來設定。 給予每個學生有限的時間，以使用 VM 來完成課堂作業。  
- 將學校的實體電腦實驗室移至雲端。 以您在實驗室上設定的使用量上限和成本閾值為限，以自動調整 VM 數目。 
- 在結束之後，只需按一下便可以刪除實驗室。 

## <a name="user-profiles"></a>使用者設定檔
本文描述 Azure 實驗室服務中各種不同的使用者設定檔。 

### <a name="lab-account-owner"></a>實驗室帳戶擁有者
通常會是擁有 Azure 訂用帳戶的組織雲端資源 IT 系統管理員，並會執行下列工作：   

- 為組織設定實驗室帳戶。
- 管理並設定所有實驗室的原則。
- 給予組織中人員在實驗室帳戶下建立實驗室的權限。

### <a name="educator"></a>教育工作者
通常，教師或線上訓練人員等使用者會在實驗室帳戶下建立教室實驗室。 教育工作者會執行下列工作： 

- 建立教室實驗室。
- 在實驗室中建立虛擬機器。 
- 在虛擬機器上安裝適當的軟體。
- 指定可以存取實驗室的人員。
- 為學生提供實驗室的註冊連結。

### <a name="student"></a>學生
學生會執行下列工作：

- 使用實驗室使用者從實驗室建立者處收到的註冊連結來註冊至實驗室。 
- 連線至實驗室中的虛擬機器，並用它來執行課堂作業、指定作業和專案。 

## <a name="next-steps"></a>後續步驟
開始使用 Azure 實驗室服務設定建立教室實驗室所需的實驗室帳戶：

- [設定實驗室帳戶](tutorial-setup-lab-account.md)
