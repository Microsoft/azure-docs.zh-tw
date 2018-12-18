---
title: 建立 Machine Learning Studio 工作區 | Microsoft Docs
description: 如何建立 Azure Machine Learning Studio 的工作區
services: machine-learning
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: aa96b784-ac6c-44bc-a28a-85d49fbe90a2
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.topic: article
ms.date: 12/07/2017
ms.openlocfilehash: 94502cbb0946ad1568cf33716480406b17fd57ac
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34833949"
---
# <a name="create-and-share-an-azure-machine-learning-workspace"></a>建立和共用 Azure Machine Learning 工作區
此功能表會連結至說明如何設定 Cortana Analytics 程序 (CAP) 所用的各種資料科學環境主題。

[!INCLUDE [data-science-environment-setup](../../../includes/cap-setup-environments.md)]

若要使用 Azure Machine Learning Studio，您需要有機器學習服務工作區。 此工作區包含您建立、管理及發行實驗所需的工具。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

### <a name="to-create-a-workspace"></a>建立工作區
1. 登入 [Azure 入口網站](https://portal.azure.com/)

    > [!NOTE]
    > 若要登入並建立工作區，您必須是 Azure 訂用帳戶管理員。 
    >
    > 

2. 按一下 [+ 新增]

3. 在搜尋方塊中，輸入 **Machine Learning Studio 工作區**，並選取相符的項目。 然後按一下頁面底部的 [建立]。

4. 輸入您的工作區資訊：

    - 工作區名稱可能最多 260 個字元，結尾不可為空格。 名稱不能包含下列字元︰`< > * % & : \ ? + /`
    - 會使用您選擇 (或建立) 的 Web 服務方案，以及您選取的相關聯定價層，如果您從此工作區中部署 web 服務。

    ![建立新的工作區](./media/create-workspace/create-new-workspace.png)

5. 按一下頁面底部的 [新增] 。

一旦部署工作區之後，您可以在 Machine Learning Studio 中開啟它。

1. 瀏覽至 [https://studio.azureml.net/](https://studio.azureml.net/) 的 Machine Learning Studio。

2. 選取您右上角的工作區。

    ![選取工作區](./media/create-workspace/open-workspace.png)

3. 按一下 [我的實驗]。

    ![開啟實驗](./media/create-workspace/my-experiments.png)

如需管理您的工作區的詳細資訊，請參閱 [管理 Azure Machine Learning 工作區](manage-workspace.md)。
如果您對於建立您的工作區遇到問題，請參閱 [疑難排解指南：建立及連線至 Machine Learning 工作區](troubleshooting-creating-ml-workspace.md)。


## <a name="sharing-an-azure-machine-learning-workspace"></a>共用 Azure 機器學習服務工作區
建立 Machine Learning 工作區 之後，您就可以邀請使用者加入您的工作區，並與您的工作區和其所有實驗、資料集、筆記型電腦等共用存取權。您可以將使用者新增至兩個角色之一︰

* **使用者** - 工作區使用者可以在工作區中建立、開啟、修改和刪除實驗、資料集等。
* **擁有者** - 除了使用者可以執行的動作以外，擁有者可以邀請和移除工作區中的使用者。

> [!NOTE]
> 建立工作區的系統管理員帳戶會自動以工作區擁有者身分新增至工作區。 不過，其他的系統管理員或該訂用帳戶中的使用者不會自動授與工作區的存取權 - 您必須先明確邀請他們。
> 
> 

### <a name="to-share-a-workspace"></a>共用工作區

1. 登入 [https://studio.azureml.net/Home](https://studio.azureml.net/Home) 的 Machine Learning Studio

2. 按一下左面板中的 [設定]

3. 按一下 [使用者] 索引標籤

4. 按一下頁面底部的 [邀請更多使用者]

    ![Studio 設定](./media/create-workspace/settings.png)

5. 輸入一或多個電子郵件地址。 使用者需要有效的 Microsoft 帳戶或組織帳戶 (來自 Azure Active Directory)。

6. 選取您是否想要將使用者新增為擁有者或使用者。

7. 按一下 [確定] 核取記號按鈕。

您新增的每個使用者會收到一封電子郵件，其中含有如何登入共用工作區的指示。

> [!NOTE]
> 若要讓使用者能夠部署或管理此工作區中的 web 服務，它們必須為參與者或 Azure 訂用帳戶中的系統管理員。 



