---
title: 比較 Azure 實驗室服務中不同類型的實驗室 | Microsoft Docs
description: 說明及比較您可以使用 Azure 實驗室服務所建立的不同類型實驗室
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
ms.date: 05/17/2018
ms.author: spelluru
ms.openlocfilehash: 8cf779f203850ca03942ba2395baf07412712610
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47092964"
---
# <a name="compare-managed-labs-in-azure-lab-services-and-devtest-labs"></a>比較 Azure 實驗室服務與 DevTest Labs 中的受控實驗室
您可以建立的實驗室有兩種，使用 Azure 實驗室服務的**受控實驗室**，和使用 Azure DevTest Labs 的**自訂實驗室**。 如果您要在實驗室中只放置需要的項目，並讓服務設定及管理實驗室所需的基礎結構，請選擇其中一個**受控實驗室**。 目前，**教室實驗室**是您使用 Azure 實驗室服務可以建立的唯一受控實驗室類型。 如果您要管理自己的基礎結構，請使用 Azure DevTest Labs 建立實驗室。

下節提供更多這些實驗室的詳細資料。 

## <a name="managed-lab-types"></a>受控實驗室類型
Azure 實驗室服務可讓您建立基礎結構由 Azure 所管理的實驗室。 此文章將它們稱為受控實驗室。 受控實驗室提供符合您特定需求的不同實驗室類型。 目前唯一支援的受控實驗室類型是**教室實驗室**。 

受控實驗室讓您能夠進行最少的設定便立即開始使用。 服務本身會處理所有實驗室的基礎結構管理，從啟動 VM 來處理錯誤，至縮放基礎結構。 若要建立受控實驗室 (例如教室實驗室)，您需要先為您的組織建立實驗室帳戶。 實驗室帳戶是作為中心帳戶，並在其中管理組織內的所有實驗室。 

當您在這些受控實驗室中建立和使用 Azure 資源時，服務會在內部 Microsoft 訂用帳戶中建立和管理資源。 它們不是建立在您自己的 Azure 訂用帳戶中。 服務會在內部 Microsoft 訂用帳戶中繼續追蹤這些資源的使用量。 這些使用量會計費回到包含該實驗室帳戶的 Azure 訂用帳戶。   

以下是一些**受控實驗室的使用案例**： 

- 為學生提供具有虛擬機器的實驗室，且虛擬機器已針對課堂需求來設定。 提供每個學生使用 VM 處理作業或個人專案的有限時數。
- 設定高效能計算 VM 集區，以執行計算密集研究或圖形密集研究。 視需要執行 VM，並在完成時清除機器。 
- 將學校的實體電腦實驗室移至雲端。 以您在實驗室上設定的使用量上限和成本閾值為限，以自動調整 VM 數目。  
- 為裝載 Hackathon 快速佈建虛擬機器實驗室。 在結束之後，只需按一下便可以刪除實驗室。 


## <a name="devtest-labs"></a>DevTest Labs
在某些情況下您可能想要自行管理所有基礎結構和設定 (在您擁有的訂用帳戶內)。 若要這麼做，您可以在 Azure 入口網站中使用 Azure DevTest Labs 建立實驗室。 對於這些實驗室，您不需要建立實驗室帳戶。 這些實驗室不會顯示在實驗室帳戶 (為受控實驗室而存在) 中。  

以下是一些**使用 DevTest Labs 的使用案例**： 

- 快速佈建用於裝載 Hackathon 或會議實作課程的虛擬機器實驗室。 在結束之後，只需按一下便可以刪除實驗室。 
- 建立設定好您應用程式的 VM 集區，並讓您的小組輕鬆使用虛擬機器進行群眾挑錯。  
- 提供包含開發人員所需所有工具的虛擬機器給他們。 排定自動啟動和關機，以將成本降到最低。 
- 重複建立測試機器作為您部署的一部分。 測試您最新的位元，並於完成時清除測試機器。 
- 設定各種不同組態的虛擬機器和多個測試代理程式，以進行縮放和效能測試。 
- 使用以您的最新版產品設定的實驗室，為您的客戶提供訓練課程。 提供每位客戶使用實驗室的有限時數。 


## <a name="managed-lab-types-vs-devtest-labs"></a>受控實驗室類型與DevTest Labs
下表比較 Azure 實驗室服務支援的兩種實驗室類型： 

| 特性 | 受控實驗室 | DevTest Labs |
| -------- | ----------------- | ---------- |
| 在實驗室中管理 Azure 基礎結構。 |  由服務自動管理 | 您自行管理  |
| 基礎結構問題的內建恢復 | 由服務自動處理 | 您自行管理  |
| 訂用帳戶管理 | 服務處理支援服務的 Microsoft 訂用帳戶內的資源配置。 由服務自動處理縮放。 | 您使用自己的 Azure 訂用帳戶自行管理。 不會自動縮放訂用帳戶。 |
| 實驗室內的 Azure Resource Manager 部署 | 尚未提供 | 可用 |

## <a name="next-steps"></a>後續步驟
使用 Azure 實驗室服務以設定實驗室的入門：

- [設定教室實驗室](classroom-labs/tutorial-setup-classroom-lab.md)
- [設定實驗室](tutorial-create-custom-lab.md)
