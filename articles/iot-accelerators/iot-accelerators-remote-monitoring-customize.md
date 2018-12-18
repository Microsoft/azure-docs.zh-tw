---
title: 自訂遠端監視解決方案 UI - Azure | Microsoft Docs
description: 本文提供如何存取遠端監視解決方案加速器 UI 的原始程式碼並進行某些自訂的相關資訊。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 01/17/2018
ms.topic: conceptual
ms.openlocfilehash: 7971a5aeadd2b2edaa43da8bacae9743937ce30e
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46127341"
---
# <a name="customize-the-remote-monitoring-solution-accelerator"></a>自訂遠端監視解決方案加速器

本文提供如何存取原始程式碼並自訂遠端監視解決方案加速器 UI 的相關資訊。 本文章說明：

## <a name="prepare-a-local-development-environment-for-the-ui"></a>準備適用於 UI 的本機開發環境

遠端監視解決方案加速器 UI 程式碼是使用 React.js 架構來實作。 您可以在 [azure-iot-pcs-remote-monitoring-webui](https://github.com/Azure/azure-iot-pcs-remote-monitoring-webui) \(英文\) GitHub 存放庫中找到此原始程式碼。

若要對 UI 做出變更，您可以在本機執行它的複本。 本機複本會連線到解決方案的已部署執行個體，以執行像是擷取遙測等動作。

下列步驟會概述設定本機環境以進行 UI 開發的程序：

1. 使用 **pcs** CLI 來部署解決方案加速器的**基本**執行個體。 記下部署的名稱以及您為虛擬機器提供的認證。 如需詳細資訊，請參閱[使用 CLI 進行部署](iot-accelerators-remote-monitoring-deploy-cli.md)。

1. 使用 Azure 入口網站或 [az CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 來針對裝載解決方案中微服務的虛擬機器啟用 SSH 存取。 例如︰

    ```sh
    az network nsg rule update --name SSH --nsg-name {your solution name}-nsg --resource-group {your solution name} --access Allow
    ```

    您只應該在測試和開發期間啟用 SSH 存取。 如果您啟用 SSH，[您應儘速再次加以停用](../security/azure-security-network-security-best-practices.md#disable-rdpssh-access-to-virtual-machines)。

1. 使用 Azure 入口網站或 [az CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 來尋找虛擬機器的名稱和公用 IP 位址。 例如︰

    ```sh
    az resource list --resource-group {your solution name} -o table
    az vm list-ip-addresses --name {your vm name from previous command} --resource-group {your solution name} -o table
    ```

1. 使用 SSH 來利用上一個步驟的 IP 位址，以及您執行 **pcs** 來部署解決方案時所提供的認證，以連線到您的虛擬機器。

1. 若要讓本機 UX 能夠連線，請在虛擬機器中的 Bash 殼層中執行下列命令：

    ```sh
    cd /app
    sudo ./start.sh --unsafe
    ```

1. 當您看到命令完成且網站啟動之後，就可以從虛擬機器中斷連線。

1. 在 [azure-iot-pcs-remote-monitoring-webui](https://github.com/Azure/azure-iot-pcs-remote-monitoring-webui) \(英文\) 存放庫的本機複本中，編輯 **.env** 檔案以新增已部署解決方案的 URL：

    ```config
    NODE_PATH = src/
    REACT_APP_BASE_SERVICE_URL=https://{your solution name}.azurewebsites.net/
    ```

1. 在 `azure-iot-pcs-remote-monitoring-webui` 資料夾本機複本的命令提示字元中，執行下列命令以安裝必要的程式庫，並在本機執行 UI：

    ```cmd/sh
    npm install
    npm start
    ```

1. 上一個命令會在本機於 http://localhost:3000/dashboard 執行 UI。 您可以在網站執行期間編輯程式碼，並看到它動態更新。

## <a name="customize-the-layout"></a>自訂版面配置

遠端監視解決方案中的每個頁面都是由一組控制項所組成，這在原始程式碼中稱為*面板*。 例如，[儀表板] 頁面是由五個面板所組成：概觀、地圖、警示、遙測及 KPI。 您可以在 [pcs-remote-monitoring-webui](https://github.com/Azure/pcs-remote-monitoring-webui) \(英文\) GitHub 存放庫中找到定義每個頁面及其面板的原始程式碼。 例如，定義 [儀表板] 頁面、其版面配置，以及頁面上面板的程式碼位於 [src/components/pages/dashboard](https://github.com/Azure/pcs-remote-monitoring-webui/tree/master/src/components/pages/dashboard) \(英文\) 資料夾中。

由於面板會管理自己的版面配置及大小，因此您可以輕易地修改頁面的版面配置。 例如，下列針對 `src/components/pages/dashboard/dashboard.js` 檔案中 **PageContent** 元素所做的變更，會調換地圖與遙測面板的位置，然後變更地圖與 KPI 面板的相對寬度：

```nodejs
<PageContent className="dashboard-container" key="page-content">
  <Grid>
    <Cell className="col-1 devices-overview-cell">
      <OverviewPanel
        openWarningCount={openWarningCount}
        openCriticalCount={openCriticalCount}
        onlineDeviceCount={onlineDeviceCount}
        offlineDeviceCount={offlineDeviceCount}
        isPending={kpisIsPending || devicesIsPending}
        error={devicesError || kpisError}
        t={t} />
    </Cell>
    <Cell className="col-5">
      <TelemetryPanel
        telemetry={telemetry}
        isPending={telemetryIsPending}
        error={telemetryError}
        colors={chartColorObjects}
        t={t} />
    </Cell>
    <Cell className="col-3">
      <CustAlarmsPanel
        alarms={currentActiveAlarmsWithName}
        isPending={kpisIsPending || rulesIsPending}
        error={rulesError || kpisError}
        t={t} />
    </Cell>
    <Cell className="col-4">
    <PanelErrorBoundary msg={t('dashboard.panels.map.runtimeError')}>
        <MapPanel
          azureMapsKey={azureMapsKey}
          devices={devices}
          devicesInAlarm={devicesInAlarm}
          mapKeyIsPending={azureMapsKeyIsPending}
          isPending={devicesIsPending || kpisIsPending}
          error={azureMapsKeyError || devicesError || kpisError}
          t={t} />
      </PanelErrorBoundary>
    </Cell>
    <Cell className="col-6">
      <KpisPanel
        topAlarms={topAlarmsWithName}
        alarmsPerDeviceId={alarmsPerDeviceType}
        criticalAlarmsChange={criticalAlarmsChange}
        warningAlarmsChange={warningAlarmsChange}
        isPending={kpisIsPending || rulesIsPending || devicesIsPending}
        error={devicesError || rulesError || kpisError}
        colors={chartColorObjects}
        t={t} />
    </Cell>
  </Grid>
</PageContent>
```

![變更面板版面配置](./media/iot-accelerators-remote-monitoring-customize/layout.png)

> [!NOTE]
> 本機部署中並未設定地圖。

您也可以新增相同面板的多個執行個體，或在[複製和自訂面板](#duplicate-and-customize-an-existing-control)的情況下新增相同面板的多個版本。 下列範例示範如何藉由編輯 `src/components/pages/dashboard/dashboard.js` 檔案來新增遙測面板的兩個執行個體：

```nodejs
<PageContent className="dashboard-container" key="page-content">
  <Grid>
    <Cell className="col-1 devices-overview-cell">
      <OverviewPanel
        openWarningCount={openWarningCount}
        openCriticalCount={openCriticalCount}
        onlineDeviceCount={onlineDeviceCount}
        offlineDeviceCount={offlineDeviceCount}
        isPending={kpisIsPending || devicesIsPending}
        error={devicesError || kpisError}
        t={t} />
    </Cell>
    <Cell className="col-3">
      <TelemetryPanel
        telemetry={telemetry}
        isPending={telemetryIsPending}
        error={telemetryError}
        colors={chartColorObjects}
        t={t} />
    </Cell>
    <Cell className="col-3">
      <TelemetryPanel
        telemetry={telemetry}
        isPending={telemetryIsPending}
        error={telemetryError}
        colors={chartColorObjects}
        t={t} />
    </Cell>
    <Cell className="col-2">
      <CustAlarmsPanel
        alarms={currentActiveAlarmsWithName}
        isPending={kpisIsPending || rulesIsPending}
        error={rulesError || kpisError}
        t={t} />
    </Cell>
    <Cell className="col-4">
    <PanelErrorBoundary msg={t('dashboard.panels.map.runtimeError')}>
        <MapPanel
          azureMapsKey={azureMapsKey}
          devices={devices}
          devicesInAlarm={devicesInAlarm}
          mapKeyIsPending={azureMapsKeyIsPending}
          isPending={devicesIsPending || kpisIsPending}
          error={azureMapsKeyError || devicesError || kpisError}
          t={t} />
      </PanelErrorBoundary>
    </Cell>
    <Cell className="col-6">
      <KpisPanel
        topAlarms={topAlarmsWithName}
        alarmsPerDeviceId={alarmsPerDeviceType}
        criticalAlarmsChange={criticalAlarmsChange}
        warningAlarmsChange={warningAlarmsChange}
        isPending={kpisIsPending || rulesIsPending || devicesIsPending}
        error={devicesError || rulesError || kpisError}
        colors={chartColorObjects}
        t={t} />
    </Cell>
  </Grid>
</PageContent>
```

您接著可以在每個面板中檢視不同的遙測：

![多個遙測面板](./media/iot-accelerators-remote-monitoring-customize/multiple-telemetry.png)

> [!NOTE]
> 本機部署中並未設定地圖。

## <a name="duplicate-and-customize-an-existing-control"></a>複製和自訂現有的控制項

下列步驟會概述如何使用**警示**面板，來示範如何複製現有的面板、對它進行修改，以及使用修改後的版本：

1. 在存放庫的本機複本中，複製 `src/components/pages/dashboard/panels` 資料夾中的 **alarms** 資料夾。 將新的複本命名為 **cust_alarms**。

1. 在 **cust_alarms** 資料夾的 **alarmsPanel.js** 檔案中，將類別的名稱編輯為 **CustAlarmsPanel**：

    ```nodejs
    export class CustAlarmsPanel extends Component {
    ```

1. 在 `src/components/pages/dashboard/panels/index.js` 檔案中新增以下文字行：

    ```nodejs
    export * from './cust_alarms';
    ```

1. 在 `src/components/pages/dashboard/dashboard.js` 檔案中，使用 `CustAlarmsPanel` 來取代 `AlarmsPanel`：

    ```nodejs
    import {
      OverviewPanel,
      CustAlarmsPanel,
      TelemetryPanel,
      KpisPanel,
      MapPanel,
      transformTelemetryResponse,
      chartColors
    } from './panels';

    ...

    <Cell className="col-3">
      <CustAlarmsPanel
        alarms={currentActiveAlarmsWithName}
        isPending={kpisIsPending || rulesIsPending}
        error={rulesError || kpisError}
        t={t} />
    </Cell>
    ```

您現在已使用名為 **CustAlarms** 的複本來取代原始的**警示**面板。 此複本與原始版本完全相同。 您現在可以修改此複本。 例如，若要在**警示**面板中變更資料行順序：

1. 開啟 `src/components/pages/dashboard/panels/cust_alarms/alarmsPanel.js` 檔案。

1. 修改資料行定義，如下列程式碼片段所示：

    ```nodejs
    this.columnDefs = [
      rulesColumnDefs.severity,
      {
        headerName: 'rules.grid.count',
        field: 'count'
      },
      {
        ...rulesColumnDefs.ruleName,
        minWidth: 200
      },
      rulesColumnDefs.explore
    ];
    ```

下列螢幕擷取畫面會顯示新版本的**警示**面板：

![警示面板已更新](./media/iot-accelerators-remote-monitoring-customize/reorder-columns.png)

## <a name="customize-the-telemetry-chart"></a>自訂遙測圖表

[儀表板] 頁面上的遙測圖表是由 `src/components/pages/dashboard/panels/telemtry` 資料夾中的資料所定義。 UI 會從解決方案後端的 `src/services/telemetryService.js` 檔案中擷取遙測。 下列步驟示範如何將遙測圖表上顯示的時間週期從 15 分鐘變更為 5 分鐘：

1. 在 `src/services/telemetryService.js` 檔案中，找出名為 **getTelemetryByDeviceIdP15M** 的函式。 複製此函式並修改複本，如下所示：

    ```nodejs
    static getTelemetryByDeviceIdP5M(devices = []) {
      return TelemetryService.getTelemetryByMessages({
        from: 'NOW-PT5M',
        to: 'NOW',
        order: 'desc',
        devices
      });
    }
    ```

1. 若要使用這個新的函式來填入遙測圖表，請開啟 `src/components/pages/dashboard/dashboard.js` 檔案。 找到將遙測資料流初始化的那一行並進行修改，如下所示：

    ```node.js
    const getTelemetryStream = ({ deviceIds = [] }) => TelemetryService.getTelemetryByDeviceIdP5M(deviceIds)
    ```

遙測圖表現在會顯示五分鐘的遙測資料：

![顯示一天的遙測圖表](./media/iot-accelerators-remote-monitoring-customize/telemetry-period.png)

## <a name="add-a-new-kpi"></a>新增 KPI

[儀表板] 頁面會在 [系統 KPI] 面板中顯示 KPI。 這些 KPI 是在 `src/components/pages/dashboard/dashboard.js` 檔案中計算的。 KPI 會由 `src/components/pages/dashboard/panels/kpis/kpisPanel.js` 檔案來呈現。 下列步驟說明如何在 [儀表板] 頁面上計算並呈現新的 KPI 值。 顯示的範例會在警告警示 KPI 中新增百分比變更：

1. 開啟 `src/components/pages/dashboard/dashboard.js` 檔案。 修改 **initialState** 物件以包含 **warningAlarmsChange** 屬性，如下所示：

    ```nodejs
    const initialState = {
      ...

      // Kpis data
      currentActiveAlarms: [],
      topAlarms: [],
      alarmsPerDeviceId: {},
      criticalAlarmsChange: 0,
      warningAlarmsChange: 0,
      kpisIsPending: true,
      kpisError: null,

      ...
    };
    ```

1. 修改 **currentAlarmsStats** 物件以包含 **totalWarningCount** 作為屬性：

    ```nodejs
    return {
      openWarningCount: (acc.openWarningCount || 0) + (isWarning && isOpen ? 1 : 0),
      openCriticalCount: (acc.openCriticalCount || 0) + (isCritical && isOpen ? 1 : 0),
      totalWarningCount: (acc.totalWarningCount || 0) + (isWarning ? 1 : 0),
      totalCriticalCount: (acc.totalCriticalCount || 0) + (isCritical ? 1 : 0),
      alarmsPerDeviceId: updatedAlarmsPerDeviceId
    };
    ```

1. 計算新的 KPI。 尋找適用於重要警示計數的計算。 複製程式碼並修改複本，如下所示：

    ```nodejs
    // ================== Warning Alarms Count - START
    const currentWarningAlarms = currentAlarmsStats.totalWarningCount;
    const previousWarningAlarms = previousAlarms.reduce(
      (cnt, { severity }) => severity === 'warning' ? cnt + 1 : cnt,
      0
    );
    const warningAlarmsChange = ((currentWarningAlarms - previousWarningAlarms) / currentWarningAlarms * 100).toFixed(2);
    // ================== Warning Alarms Count - END
    ```

1. 在 KPI 資料流中包含新的 **warningAlarmsChange** KPI：

    ```nodejs
    return ({
      kpisIsPending: false,

      // Kpis data
      currentActiveAlarms,
      topAlarms,
      criticalAlarmsChange,
      warningAlarmsChange,
      alarmsPerDeviceId: currentAlarmsStats.alarmsPerDeviceId,

      ...
    });
    ```

1. 在用於呈現 UI 的狀態資料中包含新的 **warningAlarmsChange** KPI：

    ```nodejs
    const {
      ...

      currentActiveAlarms,
      topAlarms,
      alarmsPerDeviceId,
      criticalAlarmsChange,
      warningAlarmsChange,
      kpisIsPending,
      kpisError,

      ...
    } = this.state;
    ```

1. 更新傳送到 KPI 面板的資料：

    ```node.js
    <KpisPanel
      topAlarms={topAlarmsWithName}
      alarmsPerDeviceId={alarmsPerDeviceType}
      criticalAlarmsChange={criticalAlarmsChange}
      warningAlarmsChange={warningAlarmsChange}
      isPending={kpisIsPending || rulesIsPending || devicesIsPending}
      error={devicesError || rulesError || kpisError}
      colors={chartColorObjects}
      t={t} />
    ```

您現在已完成 `src/components/pages/dashboard/dashboard.js` 檔案中的變更。 下列步驟說明如何在 `src/components/pages/dashboard/panels/kpis/kpisPanel.js` 檔案中進行變更，以顯示新的 KPI：

1. 修改下列程式碼行以擷取新的 KPI 值，如下所示：

    ```nodejs
    const { t, isPending, criticalAlarmsChange, warningAlarmsChange, error } = this.props;
    ```

1. 修改標記以顯示新的 KPI 值，如下所示：

    ```nodejs
    <div className="kpi-cell">
      <div className="kpi-header">{t('dashboard.panels.kpis.criticalAlarms')}</div>
      <div className="critical-alarms">
        {
          criticalAlarmsChange !== 0 &&
            <div className="kpi-percentage-container">
              <div className="kpi-value">{ criticalAlarmsChange }</div>
              <div className="kpi-percentage-sign">%</div>
            </div>
        }
      </div>
      <div className="kpi-header">{t('Warning alarms')}</div>
      <div className="critical-alarms">
        {
          warningAlarmsChange !== 0 &&
            <div className="kpi-percentage-container">
              <div className="kpi-value">{ warningAlarmsChange }</div>
              <div className="kpi-percentage-sign">%</div>
            </div>
        }
      </div>
    </div>
    ```

[儀表板] 頁面現在會顯示新的 KPI 值：

![警告 KPI](./media/iot-accelerators-remote-monitoring-customize/new-kpi.png)

## <a name="customize-the-map"></a>自訂地圖

如需解決方案中地圖元件的詳細資料，請參閱 GitHub 中的[自訂地圖](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Developer-Reference-Guide#upgrade-map-key-to-see-devices-on-a-dynamic-map)頁面。

<!--
### Connect an external visualization tool

See the [Connect an external visualization tool](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/) page in GitHub for details of how to connect an external visualization tool.

-->

## <a name="other-customization-options"></a>其他自訂選項

若要進一步修改遠端監視解決方案中的呈現方式和視覺效果層級，您可以編輯程式碼。 相關的 GitHub 存放庫為：

* [適用於 Azure IoT 解決方案的設定微服務 (.NET)](https://github.com/Azure/remote-monitoring-services-dotnet/tree/master/config) \(英文\)
* [適用於 Azure IoT 解決方案的設定微服務 (Java)](https://github.com/Azure/remote-monitoring-services-java/tree/master/config) \(英文\)
* [Azure IoT PCS 遠端監視 Web UI](https://github.com/Azure/pcs-remote-monitoring-webui) \(英文\)

## <a name="next-steps"></a>後續步驟

在本文中，您已了解可用來協助您在遠端監視解決方案加速器中自訂 Web UI 的資源。

如需關於遠端監視解決方案加速器的詳細概念資訊，請參閱[遠端監視架構](iot-accelerators-remote-monitoring-sample-walkthrough.md)

如需關於自訂遠端監視解決方案的詳細資訊，請參閱[自訂和重新部署微服務](iot-accelerators-microservices-example.md)
<!-- Next tutorials in the sequence -->
