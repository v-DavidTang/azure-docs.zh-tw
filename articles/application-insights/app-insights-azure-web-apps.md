---
title: "監視 Azure Web 應用程式效能 |Microsoft Docs"
description: "Azure Web 應用程式的應用程式效能監視。 圖表載入和回應時間、相依性資訊以及設定效能警示。"
services: application-insights
documentationcenter: .net
author: alancameronwills
manager: douge
ms.assetid: 0b2deb30-6ea8-4bc4-8ed0-26765b85149f
ms.service: azure-portal
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/24/2016
ms.author: awills
translationtype: Human Translation
ms.sourcegitcommit: 41ce9b0e323c0938b6db98b99d8d687d1ed0f0ef
ms.openlocfilehash: c5869c2f4f593d8ffd1992ec2a7dbc473898f3ad


---
# <a name="monitor-azure-web-app-performance"></a>監視 Azure Web 應用程式效能
在 [Azure 入口網站](https://portal.azure.com)中，您可以為 [Azure Web 應用程式](../app-service-web/app-service-web-overview.md)設定應用程式效能監視。 [Azure Application Insights](app-insights-overview.md) 會檢測您的應用程式，將其活動的相關遙測傳送給 Application Insights 服務，以在其中儲存和分析遙測。 該處的度量圖表和搜尋工具可用於協助診斷問題、改善效能，以及評估使用方式。

## <a name="run-time-or-build-time"></a>執行階段或建置階段
您可以用任一種方式檢測應用程式，進而設定監視︰

* **執行階段** - 您可以在 Web 應用程式已經上線時，選取效能監視延伸模組。 不需要重建或重新安裝您的應用程式。 您會取得一組標準封裝，用以監視回應時間、成功率、例外狀況、相依性等。 
* **建置階段** - 您可以在開發應用程式中安裝封裝。 此選項比較靈活。 除了相同的標準封裝以外，您可以撰寫程式碼以自訂遙測，或傳送自己的遙測。 您可以根據您的應用程式網域，記錄特定活動或記錄事件。 

## <a name="run-time-instrumentation-with-application-insights"></a>使用 Application Insights 執行時間檢測
如果您已在 Azure 中執行 Web 應用程式，您便已得到一些監視︰要求率和錯誤率。 新增 Application Insights 可得到更多監視，例如回應時間、監視相依性呼叫、智慧偵測，以及強大的分析查詢語言。 

1. 在 Web 應用程式的 Azure 控制台中**選取 Application Insights**。
   
    ![在 [監視] 之下，選擇 Application Insights](./media/app-insights-azure-web-apps/05-extend.png)
   
   * 除非您已經由其他途徑為此應用程式設定 Application Insights 資源，否則請選擇建立新的資源。
2. 安裝 Application Insights 之後，**檢測您的 Web 應用程式**。 
   
    ![檢測 Web 應用程式](./media/app-insights-azure-web-apps/restart-web-app-for-insights.png)
3. **監視 Web 應用程式**。  [探索資料](#explore-the-data)。

稍後，您可以視需要使用 Application Insights 來建置並重新部署應用程式。

如何移除 Application Insights，或切換成傳送到另一個資源？

* 在 Azure 中，開啟 Web 應用程式控制刀鋒視窗，在 [開發工具] 下開啟 [擴充功能]。 刪除 Application Insights 延伸模組。 然後在 [監視] 之下，選擇 Application Insights 並建立或選取您想要的資源。

## <a name="build-the-app-with-application-insights"></a>使用 Application Insights 建置應用程式
Application Insights 可以提供更詳細的遙測，方法是將 SDK 安裝至您的 App。 特別是，您可以收集追蹤記錄檔、[撰寫自訂遙測](app-insights-api-custom-events-metrics.md)，並取得更詳細的例外狀況報告。

1. **在 Visual Studio** (2013 Update 2 或更新版本) 中，將 Application Insights SDK 新增至您的專案。
   
    ![以滑鼠右鍵按一下 Web 專案，然後選擇 [加入 Application Insights]。](./media/app-insights-azure-web-apps/03-add.png)
   
    系統要求您登入時，請使用 Azure 帳戶的認證。
   
    此作業有兩種效果︰
   
   1. 在 Azure 中建立 Application Insights 資源，這是存放、分析和顯示遙測資料的位置。
   2. 將 Application Insights NuGet 封裝加入至您的程式碼，並加以設定以將遙測傳送至 Azure 資源。
2. 在開發電腦中執行應用程式 (F5)，以**測試遙測**。
3. 如常**將應用程式發佈**至 Azure。 

*如何切換為傳送至不同的 Application Insights 資源？*

* 在 Visual Studio 中，在專案上按一下滑鼠右鍵，選擇 [Application Insights] > [設定] 並選擇您想要的資源。 您可取得建立新資源的選項。 重建並重新部署。

## <a name="explore-the-data"></a>探索資料
1. 在 Web 應用程式控制台的 Application Insights 刀鋒視窗中，您會看到「即時計量」，其顯示在一兩秒內發生的要求和失敗。 當您要重新發佈應用程式時，這就非常有用 - 您可以立即看到任何問題。
2. 點選完整的 Application Insights 資源。

    ![點選](./media/app-insights-azure-web-apps/view-in-application-insights.png)

    您也可以直接從 Azure 資源導覽前往該處。

1. 逐一點選任何圖表以取得更詳細的資料：
   
    ![按一下 Application Insights 概觀刀鋒視窗上的圖表](./media/app-insights-azure-web-apps/07-dependency.png)
   
    您可以 [自訂計量刀鋒視窗](app-insights-metrics-explorer.md)。
2. 進一步逐一點選以查看個別事件與其屬性︰
   
    ![按一下事件類型以開啟依該類型篩選的搜尋](./media/app-insights-azure-web-apps/08-requests.png)
   
    請注意 [...] 連結以開啟所有的屬性。
   
    您可以 [自訂搜尋](app-insights-diagnostic-search.md)。

透過您的遙測功能更強大的搜尋，請使用 [Analytics 查詢語言](app-insights-analytics-tour.md)。

## <a name="next-steps"></a>後續步驟
* [能夠讓 Azure 診斷](app-insights-azure-diagnostics.md) 傳送至 Application Insights。
* [監視服務健康狀態計量](../monitoring-and-diagnostics/insights-how-to-customize-monitoring.md)，確保您的服務可用且回應正常。
* 每當發生作業事件或計量超過臨界值時，[接收警示通知](../monitoring-and-diagnostics/insights-receive-alert-notifications.md)。
* 使用 [JavaScript 應用程式和網頁適用的 Application Insights](app-insights-web-track-usage.md) ，以從造訪網頁的瀏覽器取得用戶端遙測。
* [設定可用性 Web 測試](app-insights-monitor-web-app-availability.md) ，以在您的網站關閉時發出警示。




<!--HONumber=Nov16_HO3-->


