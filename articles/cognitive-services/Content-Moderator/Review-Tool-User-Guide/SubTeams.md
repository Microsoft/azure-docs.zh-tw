---
title: Content Moderator API 中的小組與子小組 | Microsoft Docs
description: 了解如何在認知服務的 Content Moderator API 中使用小組與子小組。
services: cognitive-services
author: sanjeev3
manager: mikemcca
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: article
ms.date: 06/22/2017
ms.author: sajagtap
ms.openlocfilehash: 161c7cd8bac07d5ffc138297d98a40317a8d88fc
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367947"
---
# <a name="team-and-subteams"></a>小組與子小組 #

使用 Content Moderator 之前，您必須先建立一個小組。 當您註冊並替您的小組命名時，這個小組將成為您的預設小組。 您在審查工具中只能有一個小組。 不過，您可以建立多個子小組。 子小組適合用於建立擴大小組，或專門用來審核特定內容類別的小組。 例如，您可以將成人內容傳送到不同的小組進一步審核。

本主題說明如何建立子小組，以及快速地指派審核。 不過，您可以使用[工作流程](workflows.md)，根據特定準則來指派審核。

## <a name="go-to-the-teams-setting"></a>移至小組設定 ##

若要開始建立子小組，請選取 [設定] 之下的 [小組] 選項。

![小組設定](images/0-teams-1.png)

## <a name="create-subteams"></a>建立子小組 ##

預設小組包含所有潛在審查者；子小組是預設小組的子集。 如果有人還不是預設小組的成員，您就無法將他指派給子小組，所以您現在必須將任何審查者新增至預設小組。 按一下 [小組] 頁面上的 [邀請] 按鈕。

![邀請使用者](images/invite-users.png)

### <a name="1-create-a-subteam"></a>1.建立子小組。
向下捲動至 [子小組] 區段的 [小組] 頁面。 按一下 [新增子小組] 按鈕。 

![新增子小組](images/1-teams-1.png)

### <a name="2-name-your-subteam"></a>2.為您的子小組命名。
在對話方塊中輸入您的子小組名稱，然後按一下 [儲存]。

![子小組名稱](images/1-Teams-2.PNG)

### <a name="3-assign-members-from-your-default-team"></a>3.指派預設小組中的成員。
按一下 [新增成員] 按鈕，將預設小組中的成員指派給一或多個子小組。 您只能將現有使用者新增至子小組。 如需新增不在審查工具中的新使用者，請使用 [小組設定] 頁面上的 [邀請] 按鈕來邀請他們。

![指派子小組成員](images/1-Teams-3.PNG)

## <a name="assign-reviews-to-your-subteams"></a>將審核指派給您的子小組 ##

在您建立子小組並指派小組成員之後，您可以開始將影像和文字審核指派給這些子小組。 這項工作是從 [審核] 視窗進行。
如果您想要將個別影像指派給此小組，請按一下影像右上角的省略符號，選取 [移至]，然後選取子小組。

![將影像審核指派給子小組](images/3-review-image-subteam-1.png)

## <a name="switch-between-subteams-to-review-assigned-content"></a>切換子小組來審核指派的內容 ##

如果您是一或多個子小組的成員，您可以從 [審查工具儀表板] 切換這些子小組。 若要查看屬於子小組的所有目前擱置審核，請從 [影像] 索引標籤選取 [選擇子小組]。

![切換子小組](images/3-review-image-subteam-2.png)
