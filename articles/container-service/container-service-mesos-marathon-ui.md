---
title: "透過 Web UI 進行 Azure Container Service 容器管理 | Microsoft Docs"
description: "使用 Marathon Web UI 將容器部署到 Azure 容器服務叢集服務。"
services: container-service
documentationcenter: 
author: neilpeterson
manager: timlt
editor: 
tags: acs, azure-container-service
keywords: "Docker、容器、微服務、Mesos、Azure"
ms.assetid: d148ed1e-b582-4d51-944f-1ac7ae3c4fd6
ms.service: container-service
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/19/2016
ms.author: timlt
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: ae2945b0025da3bc933a1e4a8f10f21fd35cb51c


---
# <a name="container-management-through-the-web-ui"></a>透過 Web UI 來管理容器
DC/OS 提供環境來部署及調整叢集工作負載，同時將基礎硬體抽象化。 在 DC/OS 之上有架構會管理排程和執行計算工作負載。

雖然許多常見的工作負載都有可用的架構，但這份文件只說明如何使用 Marathon 來建立及調整容器部署。 在練習這些範例之前，您需要 Azure 容器服務中設定的 DC/OS 叢集。 您也需要有此叢集的遠端連線。 如需這些項目的詳細資訊，請參閱下列文章。

* [部署 Azure 容器服務叢集](container-service-deployment.md)
* [連接到 Azure 容器服務叢集](container-service-connect.md)

## <a name="explore-the-dcos-ui"></a>探索 DC/OS UI
在建立安全殼層 (SSH) 通道之後，瀏覽至 http://localhost/。 這會載入 DC/OS Web UI 並顯示叢集的相關資訊，例如使用的資源、作用中代理程式和執行中的服務。

![DC/OS UI](media/dcos/dcos2.png)

## <a name="explore-the-marathon-ui"></a>瀏覽 Marathon UI
若要查看 Marathon UI，請瀏覽至 http://localhost/Marathon。 在此畫面中，您可以啟動 Azure 容器服務 DC/OS 叢集上的新容器或其他應用程式。 您也可以看到有關執行容器和應用程式的資訊。  

![Marathon UI](media/dcos/dcos3.png)

## <a name="deploy-a-dockerformatted-container"></a>部署 Docker 格式化容器
若要使用 Marathon 部署新容器，請按一下 [建立應用程式]  按鈕，並在表單中輸入下列資訊：

| 欄位 | 值 |
| --- | --- |
| ID |nginx |
| 映像 |nginx |
| 網路 |橋接 |
| 主機連接埠 |80 |
| 通訊協定 |TCP |

![新增應用程式 UI--一般](media/dcos/dcos4.png)

![新增應用程式 UI--Docker 容器](media/dcos/dcos5.png)

![新增應用程式 UI--連接埠和服務探索](media/dcos/dcos6.png)

如果您想要以靜態方式將容器的連接埠對應至代理程式上的連接埠，您需要使用 JSON 模式。 若要這樣做，請使用切換功能將「新增應用程式」精靈切換為 **JSON 模式** 。 然後在應用程式定義的 `portMappings` 區段底下輸入下列內容。 此範例會將容器的連接埠 80 繫結至 DC/OS 代理程式的連接埠 80。 在進行這項變更之後，您可以將此精靈切換離開 JSON 模式。

```none
"hostPort": 80,
```

![新增應用程式 UI--連接埠 80 範例](media/dcos/dcos13.png)

DC/OS 叢集會使用一組私人和公用代理程式來進行部署。 若要讓叢集能夠從網際網路存取應用程式，您需要將應用程式部署至公用代理程式。 若要這樣做，請選取「新增應用程式」精靈的 [選擇性] 索引標籤，並在 [接受的資源角色] 中輸入 **slave_public**。

![新增應用程式 UI--公用代理程式設定](media/dcos/dcos14.png)

回到 Marathon 主頁面，您可以看到容器的部署狀態。

![Marathon 主頁面 UI--容器部署狀態](media/dcos/dcos7.png)

當您切換回 DC/OS Web UI (http://localhost/)，您將會看到 DC/OS 叢集上正在執行一項工作 (在此案例中是 Docker 格式化容器)。

![DC/OS Web UI--在叢集上執行的工作](media/dcos/dcos8.png)

您也可以看到正在執行工作的叢集節點。

![DC/OS Web UI--工作叢集節點](media/dcos/dcos9.png)

## <a name="scale-your-containers"></a>調整容器的大小
您可以使用 Marathon UI 來調整容器的執行個體計數。 若要這樣做，請瀏覽至 [Marathon] 頁面，選取您想要調整的容器，然後按一下 [調整] 按鈕。 在 [調整應用程式] 對話方塊中，輸入您想要的容器執行個體數目，然後選取 [調整應用程式]。

![Marathon UI--調整應用程式對話方塊](media/dcos/dcos10.png)

調整作業完成後，您會看到相同工作的多個執行個體分散到各 DC/OS 代理程式。

![DC/OS Web UI 儀表板--分散於代理程式的工作](media/dcos/dcos11.png)

![DC/OS Web UI--節點](media/dcos/dcos12.png)

## <a name="next-steps"></a>後續步驟
* [使用 DC/OS 和 Marathon API](container-service-mesos-marathon-rest.md)

透過 Mesos 在 Azure Container Service 上深入探索

> [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON203/player]
> 
> 




<!--HONumber=Nov16_HO2-->


