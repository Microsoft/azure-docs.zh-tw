---
title: 在 Azure Blockchain Workbench 中使用應用程式
description: 如何在 Azure Blockchain Workbench 中使用應用程式合約。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: 4fe6f164882ffce7bf22ec0c0b94107abcf6a20e
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48241224"
---
# <a name="using-applications-in-azure-blockchain-workbench"></a>在 Azure Blockchain Workbench 中使用應用程式

您可以使用 Blockchain Workbench 建立合約並對其採取動作。 您也可以檢視合約詳細資料，例如狀態和交易歷程記錄。

## <a name="prerequisites"></a>必要條件

* Blockchain Workbench 部署。 如需詳細資訊，請參閱 [Azure Blockchain Workbench 部署](deploy.md)，以取得部署的詳細資訊
* 已在 Blockchain Workbench 中部署的區塊鏈應用程式。 請參閱[在 Azure Blockchain Workbench 中建立區塊鏈應用程式](create-app.md)

在瀏覽器中[開啟 Blockchain Workbench](deploy.md#blockchain-workbench-web-url)。

![Blockchain Workbench](./media/use/workbench.png)

您必須以 Blockchain Workbench 成員的身分進行登入。 如果未列出應用程式，代表您是 Blockchain Workbench 的成員，但不是任何應用程式的成員。 Blockchain Workbench 系統管理員可以將成員指派至應用程式。

## <a name="create-new-contract"></a>建立新的合約 

若要建立新合約，您必須是指定為合約**啟動者**的成員。 如需有關定義應用程式角色和合約啟動者的資訊，請參閱[組態概觀中的工作流程](configuration.md#workflows)。 如需將成員指派給應用程式角色的資訊，請參閱[將成員新增至應用程式](manage-users.md#add-member-to-application)。

1. 在 Blockchain Workbench 應用程式區段中，選取您想要建立的合約所在的應用程式圖格。 隨即會顯示作用中合約的清單。

2. 若要建立新的合約，請選取 [新增合約]。

    ![[新增合約] 按鈕](./media/use/contract-list.png)

3. [新增合約] 窗格隨即顯示。 指定初始參數值。 選取 [建立] 。

    ![[新增合約] 窗格](./media/use/new-contract.png)

    新建立的合約會與其他作用中合約一起顯示在清單中。

    ![作用中合約清單](./media/use/active-contracts.png)

## <a name="take-action-on-contract"></a>對合約採取動作

根據合約所處狀態，成員可以採取動作來將合約轉換至下一個狀態。 這些動作會在[狀態](configuration.md#states)內定義為[轉換](configuration.md#transitions)。 若成員屬於轉換所允許的應用程式或執行個體角色，則該成員可以採取動作。 

1. 在 Blockchain Workbench 應用程式區段中，選取要採取動作的合約所在的應用程式圖格。
2. 選取清單中的合約。 合約的詳細資料會顯示在不同的區段。 

    ![合約詳細資料](./media/use/contract-details.png)

    | 區段  | 說明  |
    |---------|---------|
    | 狀態 | 列出合約階段內的目前進度 |
    | 詳細資料 | 合約目前的值 |
    | 動作 | 最後一個動作的詳細資料 |
    | 活動 | 合約的交易歷程記錄 |
    
3. 在 [動作] 區段中，選取 [採取動作]。

4. 合約目前狀態的詳細資料會顯示在窗格中。 在下拉式清單中選擇您想要採取的動作。 

    ![選擇動作](./media/use/choose-action.png)

5. 選取 [採取動作] 以起始動作。
6. 若動作需要參數，請指定動作的值。

    ![採取動作](./media/use/take-action.png)

7. 選取 [採取動作] 以執行動作。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [如何針對 Azure Blockchain Workbench 進行疑難排解](troubleshooting.md)
