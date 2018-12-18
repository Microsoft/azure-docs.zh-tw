---
title: 針對 Azure 中的 OpenShift 部署進行疑難排解 | Microsoft Docs
description: 針對 Azure 中的 OpenShift 部署進行疑難排解
services: virtual-machines-linux
documentationcenter: virtual-machines
author: haroldw
manager: najoshi
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: ''
ms.author: haroldw
ms.openlocfilehash: 35e554d3a9c7e7d56546ae9723c33eb59e906472
ms.sourcegitcommit: 6a22af82b88674cd029387f6cedf0fb9f8830afd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/11/2017
ms.locfileid: "24139445"
---
# <a name="troubleshoot-openshift-deployment-in-azure"></a>針對 Azure 中的 OpenShift 部署進行疑難排解

如果 OpenShift 叢集部署不成功，可以進行一些疑難排解工作來縮小問題的範圍。 檢視部署狀態，並與下列的結束代碼清單比較：

- 結束代碼 3：您的 Red Hat 訂用帳戶使用者名稱 / 密碼或組織識別碼 / 啟用金鑰不正確
- 結束代碼 4：您的 Red Hat 集區識別碼不正確，或沒有可用的權利
- 結束代碼 5：無法佈建 Docker 精簡集區磁碟區
- 結束代碼 6：OpenShift 叢集安裝失敗
- 結束代碼 7：OpenShift 叢集安裝成功，但是 Azure 雲端解決方案提供者設定失敗 - 主要節點上的主要機器設定問題
- 結束代碼 8：OpenShift 叢集安裝成功，但是 Azure 雲端解決方案提供者設定失敗 - 主要節點上的節點設定問題
- 結束代碼 9：OpenShift 叢集安裝成功，但是 Azure 雲端解決方案提供者設定失敗 - 基礎結構或應用程式節點上的主要機器設定問題
- 結束代碼 10：OpenShift 叢集安裝成功，但是 Azure 雲端解決方案提供者設定失敗 - 更正主要節點或無法將主機設為無法排程
- 結束代碼 11：計量無法部署
- 結束代碼 12：記錄無法部署

結束代碼 7-10 是 OpenShift 叢集已安裝，但是 Azure 雲端解決方案提供者設定失敗。 您可以使用 SSH 連線到主要節點 (OpenShift Origin) 或防禦節點 (OpenShift 容器平台)，然後從該處使用 SSH 連線到每一個叢集節點以修正問題。

結束代碼 7-9 常見的失敗原因是，服務主體對於訂用帳戶或資源群組沒有適當的權限。 如果是這個問題，請指派正確的權限，並手動重新執行使所有後續指令碼失敗的指令碼。

請務必重新啟動失敗的服務 (例如，systemctl restart atomic-openshift-node.service)，然後再次執行指令碼。

如需進一步疑難排解，在連接埠 2200 (Origin) 上使用 SSH 連線到您的主要節點，或在連接埠 22 (容器平台) 上使用 SSH 連線到防禦節點。 您必須位在根目錄 (sudo su-)，然後瀏覽至下列目錄：/var/lib/waagent/custom-script/download。

您在這裡會看到名為 "0" 和 "1" 的資料夾。 在這些資料夾中，您都會看到 "stderr" 和 "stdout" 兩個檔案。 查看這些檔案，以判斷失敗發生的位置。
