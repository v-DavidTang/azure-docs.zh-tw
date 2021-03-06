---
title: "使用 Azure 入口網站設定檔案上傳 | Microsoft Docs"
description: "如何使用 Azure 入口網站設定檔案上傳的概觀"
services: iot-hub
documentationcenter: 
author: dominicbetts
manager: timlt
editor: 
ms.assetid: 915f1597-272d-4fd4-8c5b-a0ccb1df0d91
ms.service: iot-hub
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/30/2016
ms.author: dobett
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: e3ac3e8cee2724b76f51423d1a6757382cca04f0


---
# <a name="configure-file-uploads-using-the-azure-portal"></a>使用 Azure 入口網站設定檔案上傳
## <a name="file-upload"></a>檔案上傳
若要使用 [IoT 中樞中的檔案上傳功能][lnk-upload]，您必須先將 Azure 儲存體帳戶與您的中樞建立關聯。 選取 [檔案上傳]  設定，即可顯示正在修改之 IoT 中樞的檔案上傳屬性清單。

![][13]

**儲存體容器**︰使用 Azure 入口網站在您目前的 Azure 訂用帳戶中選取 Azure 儲存體帳戶中的 blob 容器，來與您的 IoT 中樞建立關聯。 如果有需要，您可以在 [儲存體帳戶] 刀鋒視窗上建立 Azure 儲存體帳戶，並在 [容器] 刀鋒視窗上建立 Blob 容器。 IoT 中樞會自動產生具有此 Blob 容器寫入權限的 SAS URI，以供裝置上傳檔案時使用。

![][14]

**接收已上傳檔案的通知**︰透過切換來啟用或停用檔案上傳通知。

**SAS TTL**︰這個設定是「IoT 中樞」傳回給裝置之 SAS URI 的存留時間。 預設為 1 小時，但可以使用滑桿來自訂成其他值。

**檔案通知設定預設 TTL**：檔案上傳通知到期前的存留時間。 預設為 1 天，但可以使用滑桿來自訂成其他值。

**檔案通知最大傳遞計數**︰IoT 中樞可嘗試傳遞檔案上傳通知的次數。 預設為 10，但可以使用滑桿來自訂成其他值。

![][15]

## <a name="next-steps"></a>後續步驟
如需 IoT 中樞檔案上傳功能的詳細資訊，請參閱開發人員指南中的[從裝置上傳檔案][lnk-upload]。

遵循下列連結以深入了解如何管理 Azure IoT 中樞：

* [大量管理 IoT 裝置][lnk-bulk]
* [用量度量][lnk-metrics]
* [作業監視][lnk-monitor]

若要進一步探索 IoT 中樞的功能，請參閱︰

* [開發人員指南][lnk-devguide]
* [使用 IoT 閘道 SDK 來模擬裝置][lnk-gateway]
* [徹底保護您的 IoT 解決方案][lnk-securing]

[13]: ./media/iot-hub-configure-file-upload/file-upload-settings.png
[14]: ./media/iot-hub-configure-file-upload/file-upload-container-selection.png
[15]: ./media/iot-hub-configure-file-upload/file-upload-selected-container.png

[lnk-upload]: iot-hub-devguide-file-upload.md

[lnk-bulk]: iot-hub-bulk-identity-mgmt.md
[lnk-metrics]: iot-hub-metrics.md
[lnk-monitor]: iot-hub-operations-monitoring.md

[lnk-devguide]: iot-hub-devguide.md
[lnk-gateway]: iot-hub-linux-gateway-sdk-simulated-device.md
[lnk-securing]: iot-hub-security-ground-up.md



<!--HONumber=Nov16_HO3-->


