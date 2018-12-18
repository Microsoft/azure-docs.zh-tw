---
title: Azure Active Directory Domain Services：更新 Azure 虛擬網路的 DNS 設定 | Microsoft Docs
description: 開始使用 Azure Active Directory Domain Services
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: d4f3e82c-6807-4690-b298-4eabad2b7927
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/30/2018
ms.author: maheshu
ms.openlocfilehash: f683eeee05f264ca239b8f1da2bc5078e0146a17
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39503480"
---
# <a name="enable-azure-active-directory-domain-services"></a>啟用 Azure Active Directory Domain Services

## <a name="task-4-update-dns-settings-for-the-azure-virtual-network"></a>工作 4：更新 Azure 虛擬網路的 DNS 設定
在先前的組態工作中，您已成功為目錄啟用 Azure Active Directory Domain Services。 接下來，讓虛擬網路內的電腦可以連線並取用這些服務。 在本文中，您可以更新虛擬網路的 DNS 伺服器設定，以指向虛擬網路上可以使用 Azure Active Directory Domain Services 的兩個 IP 位址。

若要為已啟用 Azure Active Directory Domain Services 的虛擬網路更新 DNS 伺服器設定，請完成下列步驟︰


1. [概觀] 索引標籤會列出在完整佈建受控網域之後所要執行的一組**必要設定步驟**。 第一個設定步驟是**更新虛擬網路的 DNS 伺服器設定**。

    ![網域服務 - 概觀索引標籤](./media/getting-started/domain-services-provisioned-overview.png)

    > [!TIP]
    > 沒看到這個組態步驟嗎？ 如果虛擬網路的 DNS 伺服器設定是最新的，您就不會在 [概觀] 索引標籤上看到 [更新虛擬網路的 DNS 伺服器設定] 圖格。
    >
    >

2. 按一下 [設定] 按鈕，以更新虛擬網路的 DNS 伺服器設定。

> [!NOTE]
> 網路中的虛擬機器只會在重新啟動後取得新的 DNS 設定。 如果您希望它們立即取得更新後的 DNS 設定，請透過入口網站、PowerShell 或 CLI 觸發重新啟動。
>
>

## <a name="next-step"></a>後續步驟
[工作 5：啟用 Azure Active Directory Domain Services 的密碼雜湊同步](active-directory-ds-getting-started-password-sync.md)
