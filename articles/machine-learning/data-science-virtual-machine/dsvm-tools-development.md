---
title: 資料科學虛擬機器開發工具 - Azure | Microsoft Docs
description: 資料科學虛擬機器開發工具。
keywords: 資料科學工具、資料科學虛擬機器、資料科學工具、linux 資料科學
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/11/2017
ms.author: gokuma
ms.openlocfilehash: b8b0b8934b51080c3583281673183c1498c26417
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31408304"
---
# <a name="development-tools-on-the-data-science-virtual-machine"></a>資料科學虛擬機器的開發工具

資料科學虛擬機器 (DSVM) 提供組合數個常用工具和 IDE 的開發生產環境。 以下是 DSVM 提供的一些工具。 

## <a name="visual-studio-2017"></a>Visual Studio 2017  
|    |           |
| ------------- | ------------- |
| 這是什麼？   | 一般用途 IDE      |
| 支援的 DSVM 版本      | Windows      |
| 典型的使用案例      | 軟體開發    |
| 它是如何在 DSVM 上設定/安裝的？      | 資料科學工作負載 (Python 和 R 工具)、Azure 工作負載 (Hadoop、Data Lake)、Node.js、SQL Server 工具、[Visual Studio Tools for AI](https://github.com/Microsoft/vs-tools-for-ai)    |
| 如何使用/執行它？      | 桌面捷徑 (`C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\IDE\devenv.exe`)    |
| DSVM 上的相關工具      |     Visual Studio Code、RStudio、Juno  |

## <a name="visual-studio-code"></a>Visual Studio Code 
|    |           |
| ------------- | ------------- |
| 這是什麼？   | 一般用途 IDE      |
| 支援的 DSVM 版本      | Windows、Linux     |
| 典型的使用案例      | 程式碼編輯器和 Git 整合   |
| 如何使用/執行它？      | Windows 的桌面捷徑 (`C:\Program Files (x86)\Microsoft VS Code\Code.exe`)、Linux 的桌面捷徑或終端機 (`code`)    |
| DSVM 上的相關工具      |     Visual Studio 2017、RStudio、Juno  |

## <a name="rstudio--desktop"></a>RStudio Desktop 
|    |           |
| ------------- | ------------- |
| 這是什麼？   | 適用於 R 的用戶端 IDE    |
| 支援的 DSVM 版本      | Windows、Linux      |
| 典型的使用案例      |  R 開發     |
| 如何使用/執行它？      | Windows 的桌面捷徑 (`C:\Program Files\RStudio\bin\rstudio.exe`)、Linux 的桌面捷徑 (`/usr/bin/rstudio`)      |
| DSVM 上的相關工具      |   Visual Studio 2017、Visual Studio Code、Juno      |

## <a name="rstudio--server"></a>RStudio  Server 
|    |           |
| ------------- | ------------- |
| 這是什麼？   | 適用於 R 的網頁型 IDE    |
| 支援的 DSVM 版本      | Linux      |
| 典型的使用案例      |  R 開發     |
| 如何使用/執行它？      | 使用 systemctl enable rstudio-server 來啟用服務，然後使用 systemctl start rstudio-server 來啟動服務。 接著，您可以登入位於 http://your-vm-ip:8787 的 RStudio Server。       |
| DSVM 上的相關工具      |   Visual Studio 2017、Visual Studio Code、RStudio Desktop      |

## <a name="juno"></a>Juno 
|    |           |
| ------------- | ------------- |
| 這是什麼？   | Julia 語言的用戶端 IDE   |
| 支援的 DSVM 版本      | Windows、Linux      |
| 典型的使用案例      |  Julia 開發     |
| 如何使用/執行它？      | Windows 的桌面捷徑 (`C:\JuliaPro-0.5.1.1\Juno.bat`)、Linux 的桌面捷徑 (`/opt/JuliaPro-VERSION/Juno`)      |
| DSVM 上的相關工具      |   Visual Studio 2017、Visual Studio Code、RStudio      |

## <a name="pycharm"></a>Pycharm
|    |           |
| ------------- | ------------- |
| 這是什麼？   | Python 語言的用戶端 IDE    |
| 支援的 DSVM 版本      | Linux      |
| 典型的使用案例      |  R 開發     |
| 如何使用/執行它？      | Linux 的桌面捷徑 (`/usr/bin/pycharm`)      |
| DSVM 上的相關工具      |   Visual Studio 2017、Visual Studio Code、RStudio      |



## <a name="powerbi-desktop"></a>PowerBI Desktop 
|    |           |
| ------------- | ------------- |
| 這是什麼？   | 互動式資料視覺效果和 BI 工具    |
| 支援的 DSVM 版本      | Windows  |
| 典型的使用案例      |  資料視覺效果和建立儀表板   |
| 如何使用/執行它？      | 桌面捷徑 (`C:\Program Files\Microsoft Power BI Desktop\bin\PBIDesktop.exe`)      |
| DSVM 上的相關工具      |   Visual Studio 2017、Visual Studio Code、Juno      |

