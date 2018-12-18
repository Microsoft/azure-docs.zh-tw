---
title: 在 LUIS 應用程式中管理版本
titleSuffix: Azure Cognitive Services
description: 版本可讓您建置及發佈不同的模型。 理想的做法是先將目前作用中的模型複製到不同的應用程式版本，再對模型進行變更。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: db461191b70aabc322e570ecc814a076c21206f2
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47033782"
---
# <a name="manage-versions"></a>管理版本

版本可讓您建置及發佈不同的模型。 理想的做法是先將目前作用中的模型複製到不同的應用程式[版本](luis-concept-version.md)，再對模型進行變更。 

若要使用版本，請在 [我的應用程式] 頁面上選取應用程式的名稱來開啟應用程式，接著選取頂端列中的 [管理]，然後選取左側導覽中的 [版本]。 

版本清單會顯示已發佈哪些版本，其發佈位置，以及目前作用中的版本。 

[![](./media/luis-how-to-manage-versions/versions-import.png "[版本] 頁面的 [管理] 區段")](./media/luis-how-to-manage-versions/versions-import.png#lightbox)

## <a name="clone-a-version"></a>複製版本

1. 選取您要複製的版本，然後從工具列中選取 [複製]。 

2. 在 [複製版本] 對話方塊中，輸入新版本的名稱，例如 "0.2"。

   ![[複製版本] 對話方塊](./media/luis-how-to-manage-versions/version-clone-version-dialog.png)
 
     > [!NOTE]
     > 版本識別碼只能包含字元、數字或 '.'，且長度不能超過 10 個字元。
 
 隨即會建立具有指定名稱的新版本，並會設定為使用中版本。

## <a name="set-active-version"></a>設定使用中版本

從清單中選取版本，然後從工具列中選取 [設成作用中]。 

[![](./media/luis-how-to-manage-versions/versions-other.png "[版本] 頁面的 [管理] 區段")](./media/luis-how-to-manage-versions/versions-other.png#lightbox)

## <a name="import-version"></a>匯入版本

1. 從工具列中選取 [匯入版本]。 

2. 在 [匯入新版本] 快顯視窗中，輸入新的版本名稱 (10 個字元)。 如果應用程式中已有 JSON 檔案中的版本，您就只需要設定版本識別碼。

    ![[版本] 頁面的 [管理] 區段](./media/luis-how-to-manage-versions/versions-import-pop-up.png)

    匯入版本之後，新版本就會變成使用中的版本。

<a name = "export-version"></a>

## <a name="other-actions"></a>其他動作

* 若要**刪除**版本，請從清單中選取版本，然後從工具列中選取 [刪除]。 選取 [確定]。 
* 若要**重新命名**版本，請從清單中選取版本，然後從工具列中選取 [重新命名]。 輸入新的名稱，然後選取 [完成]。 
* 若要**匯出**版本，請從清單中選取版本，然後從工具列中選取 [匯出應用程式]。 檔案會下載到您的本機電腦。 

