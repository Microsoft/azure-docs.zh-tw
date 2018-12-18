---
title: 使用 Azure 容器執行個體作為 Jenkins 建置代理程式
description: 了解如何使用 Azure 容器執行個體作為 Jenkins 建置代理程式。
services: container-instances
author: mmacy
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 08/31/2018
ms.author: marsma
ms.openlocfilehash: 6419753a48e1356b2d94592fca72cee1f848dcdb
ms.sourcegitcommit: e3d5de6d784eb6a8268bd6d51f10b265e0619e47
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39390648"
---
# <a name="use-azure-container-instances-as-a-jenkins-build-agent"></a>使用 Azure 容器執行個體作為 Jenkins 建置代理程式

Azure 容器執行個體 (ACI) 可提供隨選且高載的隔離環境，以便執行容器化工作負載。 ACI 因為這些特性，而成為大規模執行 Jenkins 建置作業的絕佳平台。 本文逐步說明如何部署和使用透過 ACI 預先設定為建置目標的 Jenkins 伺服器。

如需 Azure 容器執行個體的詳細資訊，請參閱[關於 Azure 容器執行個體][about-aci]。

## <a name="deploy-a-jenkins-server"></a>部署 Jenkins 伺服器

1. 在 Azure 入口網站中，選取 [建立資源] 並搜尋 [Jenkins]。 選取發行者為 **Microsoft** 的 Jenkins 供應項目，然後選取 [建立]。

2. 在 [基本資料] 表單上輸入下列資訊，然後選取 [確定]。

   - **名稱**：輸入 Jenkins 部署的名稱。
   - **使用者名稱**：輸入 Jenkins 虛擬機器管理員使用者的名稱。
   - **驗證類型**：我們建議使用 SSH 公開金鑰進行驗證。 如果您選取此選項，請貼上用來登入 Jenkins 虛擬機器的 SSH 公開金鑰。
   - **訂用帳戶**：選取 Azure 訂用帳戶。
   - **資源群組**：建立資源群組，或選取現有的資源群組。
   - **位置**：選取 Jenkins 伺服器的位置。

   ![Jenkins 入口網站部署的基本設定](./media/container-instances-jenkins/jenkins-portal-01.png)

3. 在 [其他設定] 表單上，完成下列項目：

   - **大小**：為您的 Jenkins 虛擬機器選取適當大小選項。
   - **VM 磁碟類型**：針對 Jenkins 伺服器指定 **HDD** (硬碟) 或 **SSD** (固態硬碟)。
   - **虛擬網路**：如果您想要修改預設設定，請選取箭號。
   - **子網路**：選取箭號、確認資訊，然後選取 [確定]。
   - **公用 IP 位址**：選取箭號來為公用 IP 位址提供自訂名稱、設定 SKU 和指派方法。
   - **網域名稱標籤**：指定一個值，為 Jenkins 虛擬機器建立完整 URL。
   - **Jenkins 版本類型**：從選項中選取所需的版本類型：[LTS]、[每週建置] 或 [Azure 已驗證]。

   ![Jenkins 入口網站部署的額外設定](./media/container-instances-jenkins/jenkins-portal-02.png)

4. 針對 [服務主體整合]，選取 [自動 (MSI)]，讓 [Azure 受控服務識別][managed-service-identity] 為 Jenkins 執行個體自動建立驗證身分識別。 選取 [手動] 以提供您自己的服務主體認證。

5. 雲端代理程式可為 Jenkins 建置作業設定雲端式平台。 基於本文的緣故，請選取 **ACI**。 利用 ACI 雲端代理程式，每個 Jenkins 建置作業都是在容器執行個體中執行。

   ![Jenkins 入口網站部署的雲端整合設定](./media/container-instances-jenkins/jenkins-portal-03.png)

6. 完成整合設定之後，請選取 [確定]，然後在驗證摘要上再次選取 [確定]。 選取 [使用規定] 摘要上的 [建立]。 部署 Jenkins 伺服器需要幾分鐘的時間。

## <a name="configure-jenkins"></a>設定 Jenkins

1. 在 Azure 入口網站中，瀏覽至 Jenkins 資源群組、選取 Jenkins 虛擬機器，並且記下 DNS 名稱。

   ![Jenkins 虛擬機器的詳細 DNS 名稱](./media/container-instances-jenkins/jenkins-portal-fqdn.png)

2. 瀏覽至 Jenkins VM 的 DNS 名稱並複製所傳回的 SSH 字串。

   ![使用 SSH 字串的 Jenkins 登入指示](./media/container-instances-jenkins/jenkins-portal-04.png)

3. 在您的開發系統上開啟終端機工作階段，然後貼入上一個步驟中的 SSH 字串。 將 `username` 更新為在部署 Jenkins 伺服器時指定的使用者名稱。

4. 工作階段連線之後，請執行下列命令來擷取初始管理員密碼：

   ```
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

5. 讓 SSH 工作階段和通道保持執行狀態，並且在瀏覽器中移至 http://localhost:8080。 將初始管理密碼貼到方塊中，然後選取 [繼續]。

   ![「解除鎖定 Jenkins」畫面與系統管理員密碼方塊](./media/container-instances-jenkins/jenkins-portal-05.png)

6. 選取 [安裝建議的外掛程式] 以安裝所有建議的 Jenkins 外掛程式。

   ![已選取 [安裝建議的外掛程式] 的「自訂 Jenkins」畫面](./media/container-instances-jenkins/jenkins-portal-06.png)

7. 建立管理員使用者帳戶。 此帳戶用來登入和使用您的 Jenkins 執行個體。

   ![已填入認證的「建立第一位管理使用者」畫面](./media/container-instances-jenkins/jenkins-portal-07.png)

8. 選取 [儲存並完成]，然後選取 [開始使用 Jenkins] 來完成設定。

Jenkins 現在已設定並準備要建置和部署程式碼。 在此範例中，簡單的 Java 應用程式用來示範在 Azure 容器執行個體上建置 Jenkins。

## <a name="create-a-build-job"></a>建立建置作業

現在，Jenkins 建置任務建立好了，可以在 Azure 容器執行個體上示範 Jenkins 建置。

1. 選取 [新增項目]，提供組件專案名稱 (例如 **aci-demo**)，選取 [Freestyle 專案]，然後選取 [確定]。

   ![建置作業的名稱方塊和專案類型清單](./media/container-instances-jenkins/jenkins-new-job.png)

2. 在 [一般] 之下，確定已選取 [限制可以執行這個專案的位置]。 針對 [標籤運算式] 輸入 **linux**。 此組態可確保在 ACI 雲端上執行此建置作業。

   ![包含組態詳細資訊的 [一般] 索引標籤](./media/container-instances-jenkins/jenkins-job-01.png)

3. 在 [建置] 之下，選取 [新增建置步驟]，然後選取 [執行殼層]。 輸入 `echo "aci-demo"` 當作命令。

   ![包含建置步驟選項的 [組建] 索引標籤](./media/container-instances-jenkins/jenkins-job-02.png)

5. 選取 [ **儲存**]。

## <a name="run-the-build-job"></a>執行建置作業

若要測試建置作業並將 Azure 容器執行個體當作建置平台觀測，請以手動方式開始建置。

1. 選取 [立即建置] 以開始建置作業。 系統需要花幾分鐘的時間來啟動作業。 您應該會看到類似下圖的狀態：

   ![包含作業狀態的「建置記錄」資訊](./media/container-instances-jenkins/jenkins-job-status.png)

2. 在執行作業時，開啟 Azure 入口網站並查看 Jenkins 資源群組。 您應會看到已建立一個容器執行個體。 Jenkins 作業正在這個執行個體內部執行。

   ![資源群組中的容器執行個體](./media/container-instances-jenkins/jenkins-aci.png)

3. 由於 Jenkins 執行的作業數目超過已設定的 Jenkins 執行程式數目 (預設值為 2)，所以會建立多個容器執行個體。

   ![新建立的容器執行個體](./media/container-instances-jenkins/jenkins-aci-multi.png)

4. 完成所有建置作業後，就會移除容器執行個體。

   ![已移除容器執行個體的資源群組](./media/container-instances-jenkins/jenkins-aci-none.png)

## <a name="troubleshooting-the-jenkins-plugin"></a>針對 Jenkins 外掛程式進行疑難排解

如果您遇到任何有關 Jenkins 外掛程式的錯誤，請在 [Jenkins JIRA](https://issues.jenkins-ci.org/) 的特定元件中提交問題。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure 上的 Jenkins，請參閱 [Azure 與 Jenkins][jenkins-azure]。

<!-- LINKS - internal -->
[about-aci]: ./container-instances-overview.md
[jenkins-azure]: ../jenkins/overview.md
[managed-service-identity]: ../active-directory/managed-service-identity/overview.md