---
title: 將 Azure AD Connect 資料庫從 SQL Server Express 移至 SQL Server。 | Microsoft Docs
description: 本文件說明如何將 Azure AD Connect 資料庫從本機 SQL Server Express 伺服器移至遠端 SQL Server。
services: active-directory
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: get-started-article
ms.date: 03/19/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: a42deb6bc039602c4bda523d2ab1ee5a69bb5644
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46303531"
---
# <a name="move-azure-ad-connect-database-from-sql-server-express-to-sql-server"></a>將 Azure AD Connect 資料庫從 SQL Server Express 移至 SQL Server 

本文件說明如何將 Azure AD Connect 資料庫從本機 SQL Server Express 伺服器移至遠端 SQL Server。  您可以使用下列程序來完成此工作。

## <a name="about-this-scenario"></a>有關此案例
以下是一些有關此案例的簡短資訊。  在此案例中，Azure AD Connect 版本 (1.1.819.0) 會安裝於單一 Windows Server 2016 網域控制站上。  它會使用內建 SQL Server 2012 Express Edition 作為其資料庫。  資料庫會移到 SQL Server 2017 伺服器。

![](media/how-to-connect-install-move-db/move1.png)

## <a name="move-the-azure-ad-connect-database"></a>移動 Azure AD Connect 資料庫
使用下列步驟將 Azure AD Connect 資料庫移至遠端 SQL Server。

1.  在 Azure AD Connect 伺服器上，移至 [服務] 並停止 [Microsoft Azure AD 同步] 服務。
2. 找到 **%Program Files%\Microsoft Azure AD Sync/Data/** 資料夾，然後將 **ADSync.mdf** 和 **ADSync_log.mdf** 檔案複製到遠端 SQL Server。
3. 在 Azure AD Connect 伺服器上重新啟動 [Microsoft Azure AD 同步] 服務。
4. 移至 [控制台] -> [程式] -> [程式和功能]，將 Azure AD Connect 解除安裝。  選取 Microsoft Azure AD Connect 並按一下最上方的 [解除安裝]。
5. 在遠端 SQL Server 上，開啟 SQL Server Management Studio。
6. 在 [資料庫] 上，按一下滑鼠右鍵並選取 [連結]。
7. 在 [連結資料庫] 畫面上，按一下 [新增] 並瀏覽至 ADSync.mdf 檔案。  按一下 [確定]。
![](media/how-to-connect-install-move-db/move2.png)

8. 連結資料庫之後，請回到 Azure AD Connect 伺服器並安裝 Azure AD Connect。
9. 一旦 MSI 安裝完成後，Azure AD Connect 精靈就會開始使用快速模式安裝。 按一下 [結束] 圖示將畫面關閉。
![歡迎使用](./media/how-to-connect-install-move-db/db1.png)
10. 啟動新的命令提示字元或 PowerShell 工作階段。 瀏覽至 <drive>\program files\Microsoft Azure AD Connect 資料夾。 執行 .\AzureADConnect.exe /useexistingdatabase 命令，可在「使用現有資料庫」安裝模式中啟動 Azure AD Connect 精靈。
![PowerShell](./media/how-to-connect-install-move-db/db2.png)
11. [歡迎使用 Azure AD Connect] 畫面隨即迎接您。 一旦您同意授權條款及隱私權注意事項後，請按一下 [繼續]。
![歡迎使用](./media/how-to-connect-install-move-db/db3.png)
12. 在 [安裝必要元件] 畫面上，會啟用 [使用現有的 SQL Server] 選項。 指定裝載 ADSync 資料庫的 SQL Server 名稱。 如果用來主控 ADSync 資料庫的 SQL 引擎執行個體不是 SQL Server 上的預設執行個體，您就必須指定 SQL 引擎執行個體名稱。 此外，如果未啟用 SQL 瀏覽，就必須指定 SQL 引擎執行個體的連接埠號碼。 例如︰         
![歡迎使用](./media/how-to-connect-install-move-db/db4.png)           

13. 在 [連線到 Azure AD] 畫面上，您必須提供 Azure AD 目錄的全域管理員認證。 建議使用預設 onmicrosoft.com 網域中的帳戶。 此帳戶只會用來在 Azure AD 中建立服務帳戶，而且在精靈完成後便不會使用。
![連線](./media/how-to-connect-install-move-db/db5.png)
 
14. 在 [連線您的目錄] 畫面上，會列出針對目錄同步作業設定的現有 AD 樹系，旁邊會有紅色十字圖示。 若要同步處理內部部署 AD 樹系的變更，需要 AD DS 帳戶。 Azure AD Connect 精靈無法擷取儲存在 ADSync 資料庫的 AD DS 帳戶認證，因為認證會予以加密，且只能由先前的 Azure AD Connect 伺服器進行解密。 按一下 [變更認證] 可指定 AD 樹系的 AD DS 帳戶。
![Directories](./media/how-to-connect-install-move-db/db6.png)
 
 
15. 在快顯對話方塊中，您可以 (i) 提供企業管理員認證，並且讓 Azure AD Connect 為您建立 AD DS 帳戶，或 (ii) 自行建立 AD DS 帳戶，並且為 Azure AD Connect 提供其認證。 一旦您選取選項並提供必要的認證後，請按一下 [確定] 以關閉快顯對話方塊。
![歡迎使用](./media/how-to-connect-install-move-db/db7.png)
 
 
16. 一旦提供認證後，紅色十字圖示會取代為綠色勾號圖示。 按 [下一步] 。
![歡迎使用](./media/how-to-connect-install-move-db/db8.png)
 
 
17. 在 [準備好設定] 畫面中，按一下 [安裝]。
![歡迎使用](./media/how-to-connect-install-move-db/db9.png)
 
 
18. 安裝完成之後，Azure AD Connect 伺服器會自動啟用暫存模式。 建議您先檢閱伺服器組態和擱置的未預期變更匯出，然後再停用暫存模式。 

## <a name="next-steps"></a>後續步驟

- 深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
- [使用現有的 ADSync 資料庫安裝 Azure AD Connect](how-to-connect-install-existing-database.md)
- [使用 SQL 委派管理員權限安裝 Azure AD Connect](how-to-connect-install-sql-delegation.md)

