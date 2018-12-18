---
title: Azure 的 RBAC 疑難排解 | Microsoft Docs
description: 針對 Azure 角色型存取控制 (RBAC) 的各種問題進行疑難排解。
services: azure-portal
documentationcenter: na
author: rolyon
manager: mtillman
ms.assetid: df42cca2-02d6-4f3c-9d56-260e1eb7dc44
ms.service: role-based-access-control
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/23/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: seohack1
ms.openlocfilehash: d1a0e46fe348bbc60a4d02a4727a9bb27cb26742
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39223291"
---
# <a name="troubleshoot-rbac-in-azure"></a>針對 Azure 的各種 RBAC 問題進行疑難排解

本文將回答常見的角色型存取控制 (RBAC) 問題，讓您知道在 Azure 入口網站中使用角色時會有什麼樣的結果，以及如何解決存取權相關問題。

## <a name="web-app-features-that-require-write-access"></a>需要寫入存取權的 Web 應用程式功能

如果您授與對單一 Web 應用程式授與使用者唯讀存取權限，部分功能可能會在未預期的情況下停用。 以下管理功能需要 Web 應用程式的**寫入**存取權限 (參與者或擁有者)，而且無法在任何唯讀情況中使用。

* 命令 (像是開始、停止等)
* 變更像是一般組態的設定、調整設定、備份設定與監視設定
* 存取發佈認證與其他機密，像是應用程式設定與連接字串
* 串流記錄
* 診斷記錄組態
* 主控台 (命令提示字元)
* 有效的近期部署項目 (以便本機 git 持續部署)
* 預估的花費
* Web 測試
* 虛擬網路 (只有當虛擬網路先前是由具備寫入存取權限的使用者所設定，才會顯示出來)。

如果您無法存取上述任何一個磚，請洽詢您的系統管理員，取得 Web 應用程式的參與者存取權限。

## <a name="web-app-resources-that-require-write-access"></a>需要寫入存取權的 Web 應用程式資源

Web 應用程式因為幾個互有關聯的資源而顯得複雜。 以下是具有多個網站的典型資源群組：

![Web 應用程式資源群組](./media/troubleshooting/website-resource-model.png)

如此一來，如果您只授予某人 Web 應用程式存取權限，Azure 入口網站的網站刀鋒視窗上的諸多功能即會停用。

這些項目需要對應至您網站的「應用程式服務方案」的**寫入**權：  

* 檢視 Web 應用程式的定價層 (免費或標準)  
* 調整組態 (執行個體數量、虛擬機器大小、自動調整設定)  
* 配額 (儲存容量、頻寬、CPU)  

這些項目都需要包含您網站的整個「資源群組」的**寫入**權：  

* SSL 憑證與繫結 (相同資源群組與地理位置中的各個網站之間，可共用 SSL 憑證)  
* 警示規則  
* 自動調整設定  
* 應用程式見解元件  
* Web 測試  

## <a name="virtual-machine-features-that-require-write-access"></a>需要寫入存取權的虛擬機器功能

類似於 Web 應用程式，虛擬機器刀鋒視窗上的某些功能需要具備虛擬機器 (或是資源群組中的其他資源) 的寫入存取權限。

虛擬機器與網域名稱、虛擬網路、儲存體帳戶及警示規則相關。

這些項目都需要具備「虛擬機器」的**寫入**權：

* 端點  
* IP 位址  
* 磁碟  
* 擴充功能  

這些項目都需要同時具備「虛擬機器」與所屬之「資源群組」(連同網域名稱) 的**寫入**權：  

* 可用性設定組  
* 負載平衡集合  
* 警示規則  

如果您無法存取上述任何一個磚，請洽詢您的系統管理員，以取得資源群組的參與者存取權限。

## <a name="azure-functions-and-write-access"></a>Azure Functions 和寫入權限

[Azure Functions](../azure-functions/functions-overview.md) 的某些功能需要寫入存取權。 例如，如果指派使用者讀者角色，他們將無法檢視函數應用程式內的函數。 入口網站將顯示 **(無存取權)**。

![函數應用程式無存取權](./media/troubleshooting/functionapps-noaccess.png)

讀者可以按一下 [平台功能] 索引標籤，然後按一下 [所有設定] 以檢視與函數應用程式相關的一些設定 (類似於 Web 應用程式)，但他們不能修改這些設定。

## <a name="rbac-changes-are-not-being-detected"></a>偵測不到 RBAC 的變更

Azure Resource Manager 有時候會快取組態和資料來改善效能。 建立或刪除角色指派時，這些變更可能需要 30 分鐘的時間才會生效。 如果您使用 Azure 入口網站、Azure PowerShell 或 Azure CLI，您可以藉由登出再登入，來強制重新整理角色指派變更。 如果您使用 REST API 呼叫來變更角色指派，您可以重新整理存取權杖來強制重新整理。

## <a name="next-steps"></a>後續步驟
* [使用 RBAC 和 Azure 入口網站來管理存取權](role-assignments-portal.md)
* [檢視活動記錄檔中的各種 RBAC 變更](change-history-report.md)

