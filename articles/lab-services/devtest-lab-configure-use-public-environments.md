---
title: 在 Azure DevTest Labs 中設定及使用公用環境 | Microsoft Docs
description: 了解如何在 Azure DevTest Labs 中設定及使用公用環境。
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
ms.date: 08/13/2018
ms.author: spelluru
ms.openlocfilehash: d93818cd875c4050b1b35f21ce580933776c5bc5
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "40234686"
---
# <a name="configure-and-use-public-environments-in-azure-devtest-labs"></a>在 Azure DevTest Labs 中設定及使用公用環境
Azure DevTest Labs 有 [Azure Resource Manager 範本的公用存放庫](https://github.com/Azure/azure-devtestlab/tree/master/Environments)，您可以用來建立環境，而不需要自行連線到外部 GitHub 來源。 此存放庫包括常用範本，例如 Azure Web Apps、Service Fabric 叢集與開發 SharePoint 伺服器陣列環境。 此功能類似您建立之每個實驗室包含之成品的公用存放庫。 環境存放庫可讓您使用預先製作的環境範本在輸入參數最少的情況下開始，以在實驗室內為您提供順暢的 PaaS 資源開始使用體驗。 

## <a name="configuring-public-environments"></a>設定公用環境
做為實驗室擁有者，您可以在實驗事建立期間為您的實驗室啟用公用環境存放庫。 若要為您的實驗室啟用公用環境，請在建立實驗室時為 [公用環境] 欄位選取 [開啟]。 

![為新實驗室啟用公用環境](media/devtest-lab-configure-use-public-environments/enable-public-environment-new-lab.png)


針對現有的實驗室，公用環境存放庫並未啟用。 手動啟用它以使用存放庫中的範本。 針對使用 Resource Manager 範本建立的實驗室，預設也會停用存放庫。

您可以為您的實驗室啟用/停用公用環境，以及使用下列步驟只將特定環境提供給實驗室使用者： 

1. 為您的實驗室選取 [設定與原則]。 
2. 在 [虛擬機器基底] 區段中，選取 [公用環境]。
3. 若要為實驗室啟用公用環境，請選取 [是]。 否則，請選取 [否]。 
4. 若您已啟用公用環境，預設也會啟用該存放庫中的所有環境。 您可以取消選取環境，這樣該環境就不會提供給您的實驗室使用者使用。 

![公用環境頁面](media/devtest-lab-configure-use-public-environments/public-environments-page.png)

## <a name="use-environment-templates-as-a-lab-user"></a>以實驗室使用者的身分使用環境範本
身為實驗室使用者，您可以從實驗室頁面的工具列選取 [+新增]，以從已啟用的環境範本清單建立新環境。 基底清單頂端包括由您的實驗室系統管理員所啟用的公用環境範本。

![公用環境範本](media/devtest-lab-configure-use-public-environments/public-environment-templates.png)

## <a name="next-steps"></a>後續步驟
此存放庫是開放原始碼存放庫，您可以將自己的常用與實用 Resource Manager 範本加到其中以對它做出貢獻。 若要做出貢獻，只要對存放庫提交提取要求即可。  
