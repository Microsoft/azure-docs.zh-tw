---
title: 在 IoT 核心版上安裝 Azure IoT Edge | Microsoft Docs
description: 在 Windows IoT 核心版裝置上安裝 Azure IoT Edge 執行階段
author: kgremban
manager: timlt
ms.author: kgremban
ms.reviewer: veyalla
ms.date: 03/05/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: f57db00894dab80f96f45111331d47a173520ced
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39575993"
---
# <a name="install-the-iot-edge-runtime-on-windows-iot-core---preview"></a>在 Windows IoT 核心版上安裝 IoT Edge 執行階段 - 預覽

Azure IoT Edge 和 [Windows IoT Core](https://docs.microsoft.com/windows/iot-core/) 會一起運作，甚至可以在小型裝置上啟用邊緣計算。 Azure IoT Edge 執行階段即使在微型單板電腦 (SBC) 裝置上也能執行，這類裝置在 IoT 產業中非常普遍。 

本文將逐步解說如何在執行 Windows IoT Core 的開發板上佈建執行階段。 

**目前，Windows IoT Core 僅在 Intel x64 型處理器上支援 Azure IoT Edge。**

## <a name="install-the-container-runtime"></a>安裝容器執行階段

1. 使用**組建 17134 (RS4)** IoT Core 映像設定您的開發板。 
1. 開啟裝置電源，然後[使用 PowerShell 從遠端登入][lnk-powershell]。
1. 在 PowerShell 主控台中，安裝容器執行階段： 

   ```powershell
   Invoke-WebRequest https://master.dockerproject.org/windows/x86_64/docker-0.0.0-dev.zip -o temp.zip
   Expand-Archive .\temp.zip $env:ProgramFiles -f
   Remove-Item .\temp.zip
   $env:Path += ";$env:programfiles\docker"
   SETX /M PATH "$env:Path"
   dockerd --register-service
   start-service docker
   ```

   >[!NOTE]
   >此容器執行階段來自於「白鯨」專案建置伺服器，且僅供評估之用。 Docker 並未加以測試、背書或支援。

## <a name="finish-installing"></a>完成安裝

安裝 IoT Edge 安全性精靈，並使用[本文][lnk-install-windows-on-windows]中的指示加以設定

## <a name="next-steps"></a>後續步驟

現在，您的裝置正在執行 IoT Edge 執行階段，請了解如何[大規模部署和監視 IoT Edge 模組][lnk-deploy]。

<!--Links-->
[lnk-install-windows-on-windows]: how-to-install-iot-edge-windows-with-windows.md
[lnk-powershell]: https://docs.microsoft.com/windows/iot-core/connect-your-device/powershell
[lnk-deploy]: how-to-deploy-monitor.md
[lnk-docker-install]: https://docs.docker.com/engine/installation/linux/docker-ce/binaries#install-server-and-client-binaries-on-windows
[lnk-docker-containers]: https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10#2-switch-to-windows-containers
