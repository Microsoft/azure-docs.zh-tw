---
title: 適用於 MySQL 的 Azure 資料庫的資料庫應用程式開發概觀
description: 介紹開發人員在撰寫應用程式程式碼以連接到適用於 MySQL 的 Azure 資料庫時所應遵循的設計考量
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: b733468d41afacb616c95f0628e7bad6b0c837f0
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35264158"
---
# <a name="application-development-overview-for-azure-database-for-mysql"></a>適用於 MySQL 的 Azure 資料庫的應用程式開發概觀 
本文討論開發人員在撰寫應用程式程式碼以連接到適用於 MySQL 的 Azure 資料庫時所應遵循的設計考量。 

> [!TIP]
> 如需示範如何建立伺服器、建立伺服器型防火牆、檢視伺服器屬性、建立資料庫，以及使用 Workbench 和 mysql.exe 進行連接和查詢的教學課程，請參閱[設計您第一個適用於 MySQL 資料庫的 Azure 資料庫](tutorial-design-database-using-portal.md)

## <a name="language-and-platform"></a>語言和平台
有一些程式碼範例可供各種程式設計語言和平台使用。 您可以在下列文章中找到程式碼範例的連結：[用來連接到適用於 MySQL 的 Azure 資料庫的連線庫](concepts-connection-libraries.md)

## <a name="tools"></a>工具
適用於 MySQL 的 Azure 資料庫會使用 MySQL 社群版本，此版本可與 MySQL 一般管理工具相容，例如 Workbench 或 MySQL 公用程式 (例如 mysql.exe、[phpMyAdmin](https://www.phpmyadmin.net/)、[Navicat](https://www.navicat.com/products/navicat-for-mysql) 等)。 您也可以使用 Azure 入口網站、Azure CLI 和 REST API，來與資料庫服務互動。

## <a name="resource-limitations"></a>資源限制
適用於 MySQL 的 Azure 資料庫會使用兩個種不同機制來管理可供伺服器使用的資源： 
- 資源管理。
- 強制執行限制。

## <a name="security"></a>安全性
適用於 MySQL 的 Azure 資料庫提供資源，以便在 MySQL 資料庫上限制存取、保護資料、設定使用者和角色，以及監視活動。

## <a name="authentication"></a>驗證
適用於 MySQL 的 Azure 資料庫支援使用者與登入的伺服器驗證。

## <a name="resiliency"></a>復原功能
當連接到 MySQL 資料庫發生暫時性錯誤時，您的程式碼應該重試呼叫。 我們建議重試邏輯使用輪詢邏輯，如此它就不會同時重試多個用戶端而讓 SQL Database 超過負荷。

- 程式碼範例︰如需示範重試邏輯的程式碼範例，請參閱以下文章中您所選擇語言的範例︰[用來連接到適用於 MySQL 的 Azure 資料庫的連線庫](concepts-connection-libraries.md)

## <a name="managing-connections"></a>管理連線
資料庫連接是一項有限的資源，因此我們建議在存取您的 MySQL 資料庫時合理地使用連接，以達到更佳的效能。
- 使用連接共用或持續性連接來存取資料庫。
- 使用簡短的連接存留時間範圍來存取資料庫。 
- 在嘗試連接期間，於您的應用程式中使用重試邏輯，會因為並行連接數已達到允許的最大值而導致失敗。 在重試邏輯中，設定短暫的延遲，接著等待一段隨機的時間，然後再嘗試其他連接。