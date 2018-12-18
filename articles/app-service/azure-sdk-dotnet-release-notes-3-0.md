---
title: Azure SDK for .NET 3.0 版本資訊 | Microsoft 文件
description: Azure SDK for .NET 3.0 版本資訊
services: app-service\web
documentationcenter: .net
author: chrissfanos
editor: ''
ms.assetid: c83d815b-fc19-4260-821e-7d2a7206dffc
ms.service: app-service
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 03/07/2017
ms.author: juliako
ms.openlocfilehash: 207aa137b25e44baf73e7f481ebc8b6362dfa245
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30162089"
---
# <a name="azure-sdk-for-net-30-release-notes"></a>Azure SDK for .NET 3.0 版本資訊

本主題包含 Azure SDK for .NET 3.0 版的版本資訊。

## <a name="azure-sdk-for-net-30-release-summary"></a>Azure SDK for .NET 3.0 發行摘要

發行日期︰2017/03/07
 
Azure SDK 3.0 在此版本中沒有重大變更。 在現有的雲端服務專案上使用這套 SDK 也不需要升級程序。 為了在無需升級程序的情況下允許使用 Azure SDK 3.0，Azure SDK 3.0 會安裝至與 Azure SDK 2.9 相同的目錄中。 大部分元件的主要版本並未自 2.9 變更，而是只更新組建編號。

## <a name="visual-studio-2017-rtw"></a>Visual Studio 2017 RTW

- 在 Visual Studio 2017 中，這個版本的 Azure SDK for .NET 內建於 Azure 工作負載。 未來將會在 Visual Studio 2017 中提供您開發 Azure 所需的一切工具。 在 Visual Studio 2015 中，將仍然可以透過 WebPI 使用這套 SDK。 現在既然已釋出 Visual Studio 2017，我們已不再提供 Visual Studio 2013 適用的 Azure SDK .NET 版本。

### <a name="azure-diagnostics"></a>Azure 診斷

- 已變更為只儲存部分連接字串，並以雲端服務儲存體連接字串的權杖取代金鑰。 實際的儲存體金鑰現在儲存在使用者設定檔資料夾中，方便控制其存取權。 Visual Studio 會從使用者設定檔資料夾中讀取儲存體金鑰，以執行本機偵錯和發佈程序。 
- 為了因應上述變更，Visual Studio Online 小組已增強 Azure 雲端服務部署工作範本，當使用者在持續整合及部署中發佈至 Azure 時，可以指定儲存體金鑰來設定診斷擴充。
- 我們已可讓您儲存安全連接字串和 Azure 診斷 (WAD) 的 Token 化，協助您解決跨環境組態的問題。
 
### <a name="windows-server-2016-virtual-machines"></a>Windows Server 2016 虛擬機器

- Visual Studio 現在支援將雲端服務部署到作業系統系列 5 (Windows Server 2016) 虛擬機器。 對於現有的雲端服務，您可以變更設定，以新的作業系統系列為目標。 建立新的雲端服務時，如果您選擇使用 .net 4.6 或更高版本建立服務，服務會預設為使用作業系統系列 5。  如需詳細資訊，您可以檢閱[客體 OS 系列支援資料表](../cloud-services/cloud-services-guestos-update-matrix.md)。

### <a name="known-issues"></a>已知問題

- Azure .NET SDK 3.0 中造成了在 Visual Studio 2015 的並存組態中移除 Visual Studio 2017 時的問題。  如果已安裝 Visual Studio 2015 的 Azure SDK，如果解除安裝 Visual Studio 2017，將會移除 Microsoft Azure 儲存體模擬器和 Microsoft Azure 計算模擬器。  這會在 Visual Studio 2015 建立和偵錯新雲端服務專案時產生錯誤。 若要修正這個問題，請透過 Web Platform Installer 重新安裝 Azure SDK。  此問題將在未來的 Visual Studio 2017 更新中獲得解決。  .

 
### <a name="azure-in-role-cache"></a>Azure In-Role Cache 

- 2016 年 11 月 30 日終止支援 Azure In-Role Cache。 如需詳細資訊，請按一下[這裡](https://azure.microsoft.com/blog/azure-managed-cache-and-in-role-cache-services-to-be-retired-on-11-30-2016/)。




