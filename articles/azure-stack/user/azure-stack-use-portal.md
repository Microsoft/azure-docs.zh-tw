---
title: 使用 Azure Stack 入口網站 | Microsoft Docs
description: 了解如何存取和使用 Azure Stack 中的使用者入口網站。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/01/2018
ms.author: mabrigg
ms.reviewer: efemmano
ms.openlocfilehash: 7ca29ee359349f69c3d5ff21bd9db3f93358206a
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/04/2018
ms.locfileid: "34724307"
---
# <a name="use-the-azure-stack-portal"></a>使用 Azure Stack 入口網站

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以使用 Azure Stack 入口網站來訂閱公用供應項目，以及使用這些供應項目所提供的服務。 如果您已使用全域 Azure 入口網站，便已熟悉該網站的運作方式。

## <a name="access-the-portal"></a>存取入口網站

您的 Azure Stack 操作員 (服務提供者或您組織中的系統管理員) 將會告訴您正確的 URL 來存取入口網站。

- 就整合系統而言，URL 會因操作員的區域和外部網域名稱而有所不同，且格式會是 https://portal.&lt.region&gt;&lt;.FQDN&gt;。
- 如果您使用 Azure Stack 開發套件，則入口網站位址為 https://portal.local.azurestack.external。

![Azure Stack 使用者入口網站的螢幕擷取](media/azure-stack-use-portal/UserPortal.png)

## <a name="customize-the-dashboard"></a>自訂儀表板

儀表板包含一組預設圖格。 您可以選取 [編輯儀表板] 來修改預設儀表板，或是選取 [新增儀表板] 來建立自訂儀表板。 您可以透過新增或移除圖格，輕鬆地自訂儀表板。 例如，若要新增 [計算] 圖格，請選取 [新增]。 在 [計算] 上按一下滑鼠右鍵，然後選取 [釘選到儀表板]。

## <a name="create-subscription-and-browse-available-resources"></a>建立訂用帳戶及瀏覽可用的資源

如果您還沒有訂用帳戶，則要做的第一件事就是訂閱供應項目。 之後，您就可以瀏覽可用的資源。 若要瀏覽和建立資源，請使用下列任何一個方法：

- 選取儀表板上的 [Marketplace] 圖格。
- 在 [所有資源] 圖格上，選取 [建立資源]。
- 在左導覽窗格中，選取 [新增]。

## <a name="learn-how-to-use-available-services"></a>了解如何使用可用的服務

如果您需要如何使用可用服務的指引，可能有各種不同選項可供您使用。

- 您的組織或服務提供者可能會提供自己的文件，如果他們提供自訂的服務或應用程式，通常就會如此。
- 協力廠商應用程式會有自己的文件。
- 針對與 Azure 一致的服務，強烈建議您先檢閱 Azure Stack 文件。 若要存取 Azure Stack 使用者文件，請選取 [說明] 圖示，然後選取 [說明 + 支援]。

    ![UI 中的 [說明及支援] 選項](media/azure-stack-use-portal/HelpAndSupport.png)

    特別是，建議您檢閱下列文章來開始作業：

    - [關鍵考量：使用 Azure Stack 的服務或組建 Azure Stack 應用程式](azure-stack-considerations.md)
    - 在此文件的〈使用服務〉一節中，有針對每個服務所撰寫的〈考量〉文章。 〈考量〉頁面會說明 Azure 所提供的服務與 Azure Stack 所提供的相同服務彼此之間的差異。 如需範例，請參閱 [VM 考量](azure-stack-vm-considerations.md)。 〈使用服務〉一節中可能有 Azure Stack 專屬的其他資訊。

      您可以使用 Azure 文件作為服務的一般參考，但必須注意這些差異。 請注意，[快速入門教學課程] 圖格上的文件連結會指向 Azure 文件。

## <a name="get-support"></a>取得支援

如果您需要支援，請連絡您的組織或服務提供者以取得協助。

如果您使用 Azure Stack 開發套件，則 [Azure Stack 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)是唯一的支援來源。

## <a name="next-steps"></a>後續步驟

[關鍵考量：使用 Azure Stack 的服務或組建 Azure Stack 應用程式](azure-stack-considerations.md)
