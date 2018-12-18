---
title: 將 Git存放庫加入 Azure DevTest Labs 中的實驗室 | Microsoft Docs
description: 了解如何在 Azure DevTest Labs 中為自訂成品來源新增 GitHub 或 Azure DevOps Services Git 存放庫。
services: devtest-lab,virtual-machines,visual-studio-online
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.assetid: 01b459f7-eaf2-45a8-b4b5-2c0a821b33c8
ms.service: devtest-lab
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: e6c6f36d877f220bcc8d83b75750f54f85dcc198
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44299877"
---
# <a name="add-a-git-repository-to-store-custom-artifacts-and-resource-manager-templates"></a>新增可以存放自訂構件和 Resource Manager 範本的 Git 存放庫

您可以在實驗室中為 VM [建立自訂構件](devtest-lab-artifact-author.md)，或[使用 Azure Resource Manager 範本來建立自訂測試環境](devtest-lab-create-environment-from-arm.md)。 您必須為小組所建立的構件或 Resource Manager 範本新增私人 Git 存放庫。 存放庫可以裝載在 [GitHub](https://github.com) 或 [Azure DevOps Services](https://visualstudio.com) 上。

我們提供了[構件的 Github 存放庫](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts)供您依原樣部署，或者，您也可以加以自訂以供實驗室使用。 當您自訂或建立構件時，您無法將構件儲存在公用存放庫。 您必須為自訂構件和您建立的構件建立您自己的私人存放庫。 

建立 VM 時，您可以儲存 Resource Manager 範本，並依您的需求自訂，並在稍後使用範本來建立更多 VM。 您必須建立您自己的私人存放庫，來儲存您自訂的 Resource Manager 範本。  

* 若要了解如何建立 GitHub 儲存機制，請參閱 [GitHub Bootcamp](https://help.github.com/categories/bootcamp/)。
* 若要了解如何建立具有 Git 存放庫的 Azure DevOps Services 專案，請參閱[連線到 Azure DevOps Services](https://www.visualstudio.com/get-started/setup/connect-to-visual-studio-online)。

下圖顯示具有構件的存放庫在 GitHub 中的可能外觀範例：  

![GitHub 構件儲存機制範例](./media/devtest-lab-add-repo/devtestlab-github-artifact-repo-home.png)

## <a name="get-the-repository-information-and-credentials"></a>取得儲存機制資訊和認證
若要將存放庫新增至實驗室，請取得存放庫中的重要資訊。 以下幾節將說明如何取得在 GitHub 或 Azure DevOps Services 上裝載的存放庫所需的資訊。

### <a name="get-the-github-repository-clone-url-and-personal-access-token"></a>取得 GitHub 儲存機制複製 URL 和個人存取權杖

1. 移至包含構件或 Resource Manager 範本定義的 GitHub 存放庫首頁。
2. 選取 [複製或下載] 。
3. 若要將 URL 複製到剪貼簿，請選取 [HTTPS 複製 URL] 按鈕。 儲存 URL 以供稍後使用。
4. 選取 GitHub 右上角的設定檔影像，然後選取 [設定]。
5. 在左側的 [個人設定] 功能表上，選取 [個人存取權杖]。
6. 選取 [產生新的權杖] 。
7. 在 [新增個人存取權杖] 頁面上，於 [權杖描述] 底下輸入描述。 接受 [選取範圍] 底下的預設項目，然後選取 [產生權杖]。
8. 儲存產生的權杖。 您稍後會用到該權杖。
9. 關閉 GitHub。   
10. 繼續 [將您的實驗室連接至存放庫](#connect-your-lab-to-the-repository) 一節。

### <a name="get-the-azure-repos-clone-url-and-personal-access-token"></a>取得 Azure Repos 複製 URL 和個人存取權杖

1. 移至您的小組集合首頁 (例如 https://contoso-web-team.visualstudio.com) ，然後選取專案。
2. 在專案首頁上，選取 [程式碼] 。
3. 若要檢視複製 URL，可在專案 [程式碼] 頁面上，選取 [複製]。
4. 儲存 URL。 您稍後會用到該 URL。
5. 若要建立個人存取權杖，請選取使用者帳戶下拉式功能表中的 [我的設定檔]。
6. 在 [設定檔資訊] 頁面上，選取 [安全性] 。
7. 在 [安全性] 索引標籤上，選取 [新增]。
8. 在 [建立個人存取權杖] 頁面上：
   1. 輸入權杖的 **描述** 。
   2. 在 [到期日] 清單中選取 [180 天]。
   3. 在 [帳戶] 清單中選取 [所有可存取的帳戶]。
   4. 選取 [所有範圍] 選項。
   5. 選擇 [建立權杖]。
9. 新的權杖會出現在 [個人存取權杖] 清單中。 選取 [複製權杖] ，然後儲存權杖值供稍後使用。
10. 繼續 [將您的實驗室連接至存放庫](#connect-your-lab-to-the-repository) 一節。

## <a name="connect-your-lab-to-the-repository"></a>將實驗室連接至存放庫
1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
2. 選取 [更多服務]，然後從服務清單中選取 [DevTest Labs]。
3. 從實驗室清單中選取您的實驗室。 
4. 選取 [組態和原則] > [存放庫] > [+ 新增]。

    ![[新增存放庫] 按鈕](./media/devtest-lab-add-repo/devtestlab-add-repo.png)
5. 在第二個 [存放庫] 頁面上，指定下列資訊︰
  1. **名稱**。 輸入存放庫的名稱。
  2. **Git Clone URL**。 輸入您先前從 GitHub 或 Azure DevOps Services 複製的 Git HTTPS 複製 URL。
  3. **分支**。 若要取得您的定義，請輸入分支。
  4. **個人存取權杖**。 輸入您先前從 GitHub 或 Azure DevOps Services 取得的個人存取權杖。
  5. **資料夾路徑**。 輸入至少與複製 URL 相關的一個資料夾路徑，其中包含構件或 Resource Manager 範本。 指定子目錄時，請確定您有在資料夾路徑中包含斜線。

     ![存放庫區域](./media/devtest-lab-add-repo/devtestlab-repo-blade.png)
6. 選取 [ **儲存**]。

### <a name="related-blog-posts"></a>相關部落格文章
* [在 DevTest Labs 中針對失敗的構件進行疑難排解](devtest-lab-troubleshoot-artifact-failure.md)
* [Join a VM to an existing Active Directory domain by using a Resource Manager template in DevTest Labs](http://www.visualstudiogeeks.com/blog/DevOps/Join-a-VM-to-existing-AD-domain-using-ARM-template-AzureDevTestLabs) (在 DevTest Labs 中使用 Resource Manager 範本將 VM 加入至現有 Active Directory 網域)

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>後續步驟
建立私人 Git 存放庫之後，您可以執行下列一或兩個動作，視您的需求而定︰
* 儲存[自訂構件](devtest-lab-artifact-author.md)。 您稍候可以使用它們來建立新的 VM。
* [使用 Resource Manager 範本建立多 VM 環境和 PaaS 資源](devtest-lab-create-environment-from-arm.md)。 然後，您可以將範本儲存在私人存放庫中。

建立 VM 時，您可以確認構件或範本已新增至 Git存放庫。 它們會立即出現在構件或範本的清單中。 私人存放庫的名稱會顯示在可指定來源的資料行中。 
