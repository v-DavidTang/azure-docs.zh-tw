---
title: "在 Azure App Service 中使用 Azure Redis 快取的工作階段狀態"
description: "了解如何使用 Azure 快取服務來支援 ASP.NET 工作階段狀態快取。"
services: app-service\web
documentationcenter: .net
author: Rick-Anderson
manager: wpickett
editor: none
ms.assetid: 4f98d289-2698-464d-85cd-99ec40fad1f2
ms.service: app-service-web
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: get-started-article
ms.date: 06/27/2016
ms.author: riande
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 7ac0fde19d61a9c3a9f54b42982c7096434cb965


---
# <a name="session-state-with-azure-redis-cache-in-azure-app-service"></a>在 Azure App Service 中使用 Azure Redis 快取的工作階段狀態
本主題說明如何將 Azure Redis 快取服務用於工作階段狀態。

如果您的 ASP.NET Web 應用程式使用工作階段狀態，則將需要設定外部工作階段狀態提供者 (可為 Redis 快取服務或 SQL Server 工作階段狀態提供者)。 如果您使用工作階段狀態，但並未使用外部提供者，則您的 Web 應用程式只能限定一個執行個體。 Redis 快取服務是最快最簡單的啟用方式。

[!INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="a-idcreatecacheacreate-the-cache"></a><a id="createcache"></a>建立快取
遵循[這些指示](../redis-cache/cache-dotnet-how-to-use-azure-redis-cache.md#create-cache)以建立快取。

## <a name="a-idconfigureprojectaadd-the-redissessionstateprovider-nuget-package-to-your-web-app"></a><a id="configureproject"></a>將 RedisSessionStateProvider NuGet 封裝加入 Web 應用程式
安裝 NuGet `RedisSessionStateProvider` 封裝。  從封裝管理員主控台 ([工具] > [NuGet 封裝管理員] > [封裝管理員主控台]) 中使用下列命令來安裝：

  `PM> Install-Package Microsoft.Web.RedisSessionStateProvider`

若要從 [工具] > [NuGet 封裝管理員] > [管理解決方案的 NuGet 封裝] 安裝，可搜尋 `RedisSessionStateProvider`。

如需詳細資訊，請參閱 [NuGet RedisSessionStateProvider 頁面](http://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider/)和[設定快取用戶端](../redis-cache/cache-dotnet-how-to-use-azure-redis-cache.md#NuGet)。

## <a name="a-idconfigurewebconfigamodify-the-webconfig-file"></a><a id="configurewebconfig"></a>修改 Web.Config 檔案
除了對快取進行組件參考之外，NuGet 封裝還能在 *web.config* 檔案中加入虛設常式項目。 

1. 開啟 *web.config* ，然後尋找 **sessionState** 元素。
2. 輸入 `host`、`accessKey`、`port` (SSL 連接埠應為 6380)，然後將 `SSL` 設定為 `true`。 這些值可以從快取執行個體的 [Azure 入口網站](http://go.microsoft.com/fwlink/?LinkId=529715) 分頁中取得。 如需詳細資訊，請參閱[連線至快取](../redis-cache/cache-dotnet-how-to-use-azure-redis-cache.md#connect-to-cache)。 請注意，預設會為新的快取停用非 SSL 連接埠。 如需啟用非 SSL 連接埠的詳細資訊，請參閱[在 Azure Redis 快取中設定快取](https://msdn.microsoft.com/library/azure/dn793612.aspx)主題中的[存取連接埠](https://msdn.microsoft.com/library/azure/dn793612.aspx#AccessPorts)一節。 下列標記會顯示 web.config 檔案的變更結果，特別是 port、host、accessKey* 和 *ssl* 的變更結果。
   
          <system.web>;
            <customErrors mode="Off" />;
            <authentication mode="None" />;
            <compilation debug="true" targetFramework="4.5" />;
            <httpRuntime targetFramework="4.5" />;
            <sessionState mode="Custom" customProvider="RedisSessionProvider">;
              <providers>;  
                  <!--<add name="RedisSessionProvider" 
                    host = "127.0.0.1" [String]
                    port = "" [number]
                    accessKey = "" [String]
                    ssl = "false" [true|false]
                    throwOnError = "true" [true|false]
                    retryTimeoutInMilliseconds = "0" [number]
                    databaseId = "0" [number]
                    applicationName = "" [String]
                  />;-->;
                 <add name="RedisSessionProvider" 
                      type="Microsoft.Web.Redis.RedisSessionStateProvider" 
                      port="6380"
                      host="movie2.redis.cache.windows.net" 
                      accessKey="m7PNV60CrvKpLqMUxosC3dSe6kx9nQ6jP5del8TmADk=" 
                      ssl="true" />;
              <!--<add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider" host="127.0.0.1" accessKey="" ssl="false" />;-->;
              </providers>;
            </sessionState>;
          </system.web>;

## <a name="a-idusesessionobjecta-use-the-session-object-in-code"></a><a id="usesessionobject"></a> 在程式碼中使用工作階段物件
最後一步，則是開始在您的 ASP.NET 程式碼中使用工作階段物件。 您可以使用 **Session.Add** 方法，將物件新增至工作階段狀態。 此方法使用機碼值組，將項目儲存到工作階段狀態快取。

    string strValue = "yourvalue";
    Session.Add("yourkey", strValue);

以下程式碼會從工作階段狀態擷取此值。

    object objValue = Session["yourkey"];
    if (objValue != null)
       strValue = (string)objValue;    

您也可以使用 Redis 快取來快取 Web 應用程式中物件。 如需詳細資訊，請參閱 [15 分鐘學會包含 Azure Redis 快取的 MVC 影片應用程式](https://azure.microsoft.com/blog/2014/06/05/mvc-movie-app-with-azure-redis-cache-in-15-minutes/)。
如需如何使用 ASP.NET 工作階段狀態的詳細資訊，請參閱 [ASP.NET 工作階段狀態概觀][ASP.NET 工作階段狀態概觀]。

> [!NOTE]
> 如果您想在註冊 Azure 帳戶前開始使用 Azure App Service，請移至 [試用 App Service](http://go.microsoft.com/fwlink/?LinkId=523751)，即可在 App Service 中立即建立短期入門 Web 應用程式。 不需要信用卡；沒有承諾。
> 
> 

## <a name="whats-changed"></a>變更的項目
* 如需從網站變更為 App Service 的指南，請參閱： [Azure App Service 及其對現有 Azure 服務的影響](http://go.microsoft.com/fwlink/?LinkId=529714)
  
  *作者：[Rick Anderson](https://twitter.com/RickAndMSFT)*

[安裝最新版]: http://www.windowsazure.com/downloads/?sdk=net  
[ASP.NET 工作階段狀態概觀]: http://msdn.microsoft.com/library/ms178581.aspx

[NewIcon]: ./media/web-sites-dotnet-session-state-caching/CacheScreenshot_NewButton.png
[NewCacheDialog]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_CreateOptions.png
[CacheIcon]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_CacheIcon.png
[NuGetDialog]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_NuGet.png
[OutputConfig]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_OC_WebConfig.png
[CacheConfig]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_CacheConfig.png
[EndpointURL]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_EndpointURL.png
[Managekeys]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_ManageAccessKeys.png




<!--HONumber=Nov16_HO2-->


