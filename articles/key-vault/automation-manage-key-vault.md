---
title: 使用 Azure 自動化管理 Azure 金鑰保存庫 | Microsoft Docs
description: 了解如何使用 Azure 自動化服務來管理 Azure 金鑰保存庫。
services: Key-Vault, automation
documentationcenter: ''
author: mgoedtel
manager: jwhit
editor: ''
ms.assetid: 4e780762-19b6-4ca6-b894-ebb44c538f35
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/29/2016
ms.author: magoedte
ms.openlocfilehash: b5f8a1b826717d51729b0bb621bf26e35d4bdd36
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31406869"
---
# <a name="managing-azure-key-vault-using-azure-automation"></a>使用 Azure 自動化管理 Azure 金鑰保存庫
本指南將為您介紹 Azure 自動化服務，以及如何使用它來簡化管理 Azure 金鑰保存庫中的金鑰和密碼。

## <a name="what-is-azure-automation"></a>什麼是 Azure 自動化？
[Azure 自動化](../automation/automation-intro.md) 是一項 Azure 服務，可經由程序自動化和所要的狀態組態簡化雲端管理。 使用 Azure 自動化，可以自動執行手動、重複、長時間執行及容易出錯的工作，以提高您的組織的可靠性、效率和時間價值。

Azure 自動化提供高度可靠、高度可用的工作流程執行引擎，可加以調整以符合您的需求。 在 Azure 自動化中，程序可透過手動方式、經由協力廠商系統，或依照排程的間隔啟動，讓工作精準地在需要時執行。

將您的雲端管理工作交由「Azure 自動化」自動執行，以減少營運負擔並釋出 IT 和開發維運人力，使其專注於能夠為企業創造價值的工作上。

## <a name="how-can-azure-automation-help-manage-azure-key-vault"></a>Azure 自動化如何協助管理 Azure 金鑰保存庫？
您可以使用 [AzureRM 金鑰保存庫 Cmdlet](https://www.powershellgallery.com/packages/AzureRM.KeyVault/1.1.4) 和 [Azure 傳統金鑰保存庫 Cmdlet](https://msdn.microsoft.com/library/azure/dn868052.aspx)，在 Azure 自動化中管理金鑰保存庫。 管理傳統金鑰保存庫的 Azure 模組會自動出現在 Azure 自動化中供您使用，而且您可以將 [AzureRM-KeyVault 模組](https://www.powershellgallery.com/packages/AzureRM.KeyVault/1.1.4) 匯入 Azure 自動化，以便在服務中執行許多金鑰保存庫管理工作。 您也可以在 Azure 自動化中將這些 Cmdlet 與其他 Azure 服務的 Cmdlet 配對，將跨 Azure 服務和協力廠商系統的複雜工作自動化。

使用 Azure 金鑰保存庫 Cmdlet，您可以執行以下工作和其他工作︰ 

* 建立和設定金鑰保存庫
* 建立或匯入金鑰
* 建立或更新密碼
* 更新金鑰的屬性
* 取得金鑰或密碼
* 刪除金鑰或密碼

以下是使用 PowerShell 來管理金鑰保存庫的一些範例︰  

* [Azure 金鑰保存庫 - 逐步解說](https://blogs.technet.microsoft.com/kv/2015/06/02/azure-key-vault-step-by-step)
* [設定 Azure 金鑰保存庫](https://www.simple-talk.com/cloud/platform-as-a-service/setting-up-and-configuring-an-azure-key-vault)

## <a name="next-steps"></a>後續步驟
了解 Azure 自動化的基本概念以及如何用它來管理 Azure 金鑰保存庫之後，請參考下列連結，以深入了解 Azure 自動化。

* 請參閱 Azure 自動化 [快速入門教學課程](../automation/automation-first-runbook-graphical.md)。
* 請參閱 [Azure 金鑰保存庫 PowerShell 指令碼](https://gallery.technet.microsoft.com/scriptcenter/site/search?query=azure%20key%20vault&f%5B0%5D.Value=azure%20key%20vault&f%5B0%5D.Type=SearchText&ac=5)。

