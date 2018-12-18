---
title: 在 Azure Content Moderator 中使用標記 | Microsoft Docs
description: Content Moderator 包含預設標記，您可以建立自訂標記，以便審查您企業特有的內容。
services: cognitive-services
author: sanjeev3
manager: mikemcca
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: article
ms.date: 06/25/2017
ms.author: sajagtap
ms.openlocfilehash: add4c685c07c63944ae89f48a47ac78df28c1623
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367935"
---
# <a name="about-tags"></a>關於標記 #

除了兩個預設標記 (a – isadult 和 r – isracy)，您可以建立自訂標記，以便進行更有目標性的掃描。 人力審查者接著可將這些自訂標記指派給影像或文字。

## <a name="create-tags"></a>建立標記 ##

1.  從 [設定] 索引標籤中選取 [標記]。

  ![內容審核標記](images/tags-1.png)

2.  輸入標記的兩個字母簡短代碼。
3.  輸入標記的名稱。 名稱應簡短且具有描述性。 例如，"isNudity"。
4.  輸入 [描述]。
5.  按一下 [加入]。
6.  建立標記完成時，請按一下 [儲存]。

![定義內容審核標記](images/tags-2-define.png)

## <a name="using-custom-tags"></a>使用自訂標記 ##

自訂在人力審查期間使用的標記。 它們會顯示於預覽上，且審查者在其上按一下即可加以選取。

![使用內容審核標記](images/tags-3-use.png)

勾選或取消核取 [可見]，即可關閉不同審查的不同標記。
 
![停用內容審核標記](images/tags-4-disable.png)

雖然您無法刪除兩個預設標記 (isadult 和 isracy)，您可以刪除任何已定義的自訂標記。 按一下所要刪除標記旁邊的垃圾桶。

![刪除內容審核標記](images/tags-5-delete.png)

## <a name="next-steps"></a>後續步驟 ##

若要了解如何使用標記進行影像審核，請參閱[如何審查已審核的影像](Review-Moderated-Images.md)。
