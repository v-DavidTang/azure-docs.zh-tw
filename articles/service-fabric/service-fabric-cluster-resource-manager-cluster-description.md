---
title: "資源平衡器叢集描述 | Microsoft Docs"
description: "將容錯網域、升級網域、節點屬性和節點容量指定給叢集資源管理員以描述 Service Fabric 叢集。"
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: 
ms.assetid: 55f8ab37-9399-4c9a-9e6c-d2d859de6766
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/19/2016
ms.author: masnider
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 5c8da2a9fcb824deb2a15a661a5ea355984e4fab


---
# <a name="describing-a-service-fabric-cluster"></a>描述 Service Fabric 叢集
Service Fabric 叢集資源管理員提供數種機制，來描述叢集。 在執行階段，叢集資源管理員會使用此資訊以確保叢集中執行之服務的高可用性，同時也確保能適當地使用叢集中的資源。

## <a name="key-concepts"></a>重要概念
叢集資源管理員支援數個描述叢集的功能︰

* 容錯網域
* 升級網域
* 節點屬性
* 節點容量

## <a name="fault-domains"></a>容錯網域
容錯網域是協調失敗的任何區域。 單一機器是容錯網域 (因為它有很多會造成當機的理由，從電源供應器故障，到磁碟機失敗，到 NIC 韌體不正確)。 連接到相同乙太網路交換器的許多機器會位於相同的容錯網域，連接到單一電力來源的機器也是如此。 由於這些項目本身就很容易重疊，容錯網域本身就具備階層性，並在 Service Fabric 中以 URI 表示。

如果您已設定您自己的叢集，您需要考慮失敗的所有不同區域，以確保您的容錯網域已正確設定，讓 Service Fabric 知道可以安全放置服務的位置。 我們所謂的「安全」實際上是指具智慧性。我們不想要將服務放置在只要失去容錯網域 (例如上面所列出任何元件的失敗)，就會造成服務中斷的位置。  在 Azure 環境中，我們運用環境所提供的容錯網域資訊，以正確地代表您設定叢集中的節點。
在下圖 (圖 7) 中，我們會為所有實體上色，讓容錯網域成為簡單的範例，並且列出產生的所有不同的容錯網域。 在此範例中，我們有資料中心 (DC)、機架 (R) 和刀鋒視窗 (B)。 如果每個刀鋒視窗包含一個以上的虛擬機器，容錯網域階層中可能會有另一個階層。

![透過容錯網域組織的節點][Image1]

 在執行階段期間，Service Fabric 叢集資源管理員會考慮叢集中的容錯網域，並嘗試針對指定的服務分散複本，以免將它們全都放在不同的容錯網域中。 在任何一個容錯網域 (於階層中任何層級) 發生錯誤時，此程序有助確保該服務的可用性沒有遭到破壞。

 Service Fabric 的叢集資源管理員不在意階層中有多少層級，不過因為它會嘗試確保遺失階層的任何一部分並不會影響在其上執行的叢集或服務，通常最好是在容錯網域的每一個深度層級有相同的機器數目。 這可防止階層的某一個部分在一天的結尾包含比其他部分還要多的服務。

 以會造成容錯網域的「樹系」不平衡的方式設定您的叢集，會讓叢集資源管理員難以釐清最佳的複本配置，特別是因為遺失特定網域可能會嚴重影響叢集的可用性。叢集資源管理員將會在透過將服務放置到機器上以在該「繁重」的網域中有效使用機器，以及在放置服務以使網域遺失不會造成問題之間左右為難。

 在下圖中我們會示範兩個不同的範例叢集配置，其中一個叢集的節點完善地散佈到容錯網域，而另一個叢集的其中一個容錯網域會多出許多的節點。  請注意，在 Azure 中會為您處理哪個節點位於哪個容錯和升級網域的選擇，因此您應該不會看見這類失衡。 不過，如果您曾經在內部部署或另一個環境中維護您自己的叢集，您需要考慮一些事項。

 ![兩個不同的叢集配置][Image2]

## <a name="upgrade-domains"></a>升級網域
升級網域是另一項功能，可協助 Service Fabric 資源管理員了解叢集的版面配置，讓它可以針對失敗事先計劃。 升級網域會定義區域 (實際上是節點集)，在升級期間會在相同的時間停機。

升級網域非常類似容錯網域，但是有幾個主要的差異。 首先，升級網域通常會定義原則；而容錯網域是由協調失敗區域嚴格定義 (因此通常是環境的硬體配置)。 但是，在升級網域的情況下，您必須決定您要的數量。 另一個差異是 (截至目前為止)，升級網域不是階層式，它們較像是簡單的標記。

下圖顯示虛構的設定，其中有等量分散在三個容錯網域的三個升級網域。 它也會示範一個可能的情況，具狀態服務的三個不同複本。 請注意，它們全部都在不同的容錯和升級網域。 這表示我們可能會在服務升級途中遺失容錯網域，而且叢集中仍然有一個程式碼和資料的執行中複本。 根據您的需求，這樣可能已經足夠，不過您可能會發現這個複本是舊的 (因為 Service Fabric 使用以仲裁為基礎的複寫）。 若要真正突破兩個失敗，您需要多個複本 (最少五個)。

![放置在容錯和升級網域][Image3]

擁有大量升級網域有利有弊 – 優點是升級的每個步驟更細微，因此會影響較少的節點或服務。 這會導致每次需要移動的服務較少，讓系統的流失較少，並且整體改善可靠性 (因為受到任何問題所影響的服務較少)。 擁有許多升級網域的缺點是，Service Fabric 會在升級網域升級時確認每個升級網域的健康狀態，並且確保升級網域狀況良好，再進行至下一個升級網域。 這項檢查的目的是確保服務可以穩定，其健康狀態會在繼續進行升級之前進行驗證，以便偵測任何問題。 缺點是可接受的，因為它會防止不良的變更一次影響到太多服務。

太少升級網域也有它的副作用，當每個個別升級網域關閉並且進行升級時，您的大部分整體容量無法使用。 例如，如果您只有三個升級網域，您一次只能採用整體服務或叢集容量的 1/3。 這並非您想要的結果，因為您必須在其餘的叢集中擁有足夠的容量以涵蓋工作負載，這表示正常的情況下這些節點比起增加 COGS 時負載較少。

環境中容錯或升級網域的總數沒有實際限制，對於它們如何重疊也沒有條件約束。 我們看到的一般結構是 1:1 (其中每個唯一的容錯網域對應至其本身的升級網域)、每個節點 (實體或虛擬作業系統執行個體) 一個升級網域，形成具有機器的矩陣的容錯網域和升級網域中的「等量」或「矩陣」模型通常會在對角線下。

![容錯和升級網域配置][Image4]

對於選擇哪個配置並沒有最佳答案，每個答案都各有優缺點。 例如，1FD:1UD 模型相當容易設定，而每個節點 1 UD 模型是最接近以前用來管理少量機器的模型 (其中每個機器會獨立卸下)。

最常見的模型 (我們用於裝載 Azure Service Fabric 叢集的模型) 是 FD/UD 矩陣，其中 FD 和 UD 會形成資料表，且節點是沿著對角線啟動。 最後是疏鬆或壓縮取決於相較於 FD 和 UD 數目的節點總數 (針對大型叢集以不同的方式放置，幾乎所有項目最終看起來就像是密集矩陣模式，如 [圖 10] 的右下角選項所示)。

## <a name="fault-and-upgrade-domain-constraints-and-resulting-behavior"></a>容錯和升級網域條件約束及產生的行為
叢集資源管理員會將想要在容錯和升級網域之間保持服務的平衡視為條件約束。 您可以在 [這篇文章](service-fabric-cluster-resource-manager-management-integration.md)中深入了解條件約束。 容錯和升級網域條件約束的定義如下：「針對特定服務資料分割，兩個網域之間的複本數目的差異應該永遠不能大於一」。  這基本上代表對於指定服務來說，特定的移動或排列方式在叢集中可能是無效的，因為那麼做將會違反容錯和升級網域的條件約束。

讓我們來看看一個範例。 假設我們有一個具有 6 個節點的叢集，並已設定有 5 個容錯網域和 5 個升級網域。

|  | FD0 | FD1 | FD2 | FD3 | FD4 |
| --- |:---:|:---:|:---:|:---:|:---:|
| UD0 |N1 | | | | |
| UD1 |N6 |N2 | | | |
| UD2 | | |N3 | | |
| UD3 | | | |N4 | |
| UD4 | | | | |N5 |

假設我們建立一個 TargetReplicaSetSize 為 5 的服務。 複本將會落在 N1-N5 上。 而 N6 則永遠不會被使用。 原因為何？ 讓我們看看目前配置，以及改為選擇 N6 時所會發生的情況之間的差異，並思考那和我們對 FD 和 UD 條件約束的定義之間的關係。

以下是我們的配置，以及每個容錯和升級網域的複本總數。

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| UD0 |R1 | | | | |1 |
| UD1 | |R2 | | | |1 |
| UD2 | | |R3 | | |1 |
| UD3 | | | |R4 | |1 |
| UD4 | | | | |R5 |1 |
| FDTotal |1 |1 |1 |1 |1 |- |

請注意到，此配置在每個容錯網域和升級網域的節點數上是平衡的，而在每個容錯和升級網域的複本數目上也一樣是平衡的。 每個網域都擁有相同數量的節點，以及相同數量的複本。

現在讓我們看看如果我們不使用 N2 並改為使用 N6 會發生什麼情況。 複本將會如何散佈？ 它們看起來應該會類似這樣：

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| UD0 |R1 | | | | |1 |
| UD1 |R5 | | | | |1 |
| UD2 | | |R2 | | |1 |
| UD3 | | | |R3 | |1 |
| UD4 | | | | |R4 |1 |
| FDTotal |2 |0 |1 |1 |1 |- |

這將會違反我們針對容錯網域條件約束的定義，因為 FD0 具有 2 個複本，而 FD1 則有 0 個，使得總差異為 2，因此叢集資源管理員將不會允許這種排列方式。 類似地，如果我們選擇 N2-6，便會得到：

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| UD0 | | | | | |0 |
| UD1 |R5 |R1 | | | |2 |
| UD2 | | |R2 | | |1 |
| UD3 | | | |R3 | |1 |
| UD4 | | | | |R4 |1 |
| FDTotal |1 |1 |1 |1 |1 |- |

雖然這就容錯網域而言是平衡的，但這卻違反了升級網域的條件約束 (因為 UD0 具有 0 個複本，而 UD1 具有 2 個)，因此這也是無效的。

## <a name="configuring-fault-and-upgrade-domains"></a>設定容錯和升級網域
定義容錯網域和升級網域是在 Azure 託管的 Service Fabric 部署中自動完成；Service Fabric 只會從 Azure 拾取環境資訊。 在 Azure 中容錯和升級網域資訊看起來是「單一層級」，但是實際上是從 Azure Stack 的較低層級封裝資訊，並且僅從使用者的角度顯示邏輯容錯和升級網域。

如果您維護自己的叢集 (或只是想要在您的開發機器上嘗試執行特定拓撲)，必須自行提供容錯網域和升級網域資訊。 在此範例中，我們會定義一個 9 節點的本機開發叢集，它會跨越三個「資料中心」(各有三個機架)，並且跨越那三個資料中心等量分散三個升級網域。 在叢集資訊清單範本中，它看起來將會如下︰

ClusterManifest.xml

```xml
  <Infrastructure>
    <!-- IsScaleMin indicates that this cluster runs on one-box /one single server -->
    <WindowsServer IsScaleMin="true">
      <NodeList>
        <Node NodeName="Node01" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType01" FaultDomain="fd:/DC01/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
        <Node NodeName="Node02" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType02" FaultDomain="fd:/DC01/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
        <Node NodeName="Node03" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType03" FaultDomain="fd:/DC01/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
        <Node NodeName="Node04" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType04" FaultDomain="fd:/DC02/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
        <Node NodeName="Node05" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType05" FaultDomain="fd:/DC02/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
        <Node NodeName="Node06" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType06" FaultDomain="fd:/DC02/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
        <Node NodeName="Node07" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType07" FaultDomain="fd:/DC03/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
        <Node NodeName="Node08" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType08" FaultDomain="fd:/DC03/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
        <Node NodeName="Node09" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType09" FaultDomain="fd:/DC03/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
      </NodeList>
    </WindowsServer>
  </Infrastructure>
```
> [!NOTE]
> 在 Azure 部署中，容錯網域和升級網域是由 Azure 指派。 因此，您適用的節點與 Azure 基礎結構選項中的角色定義不包含容錯網域或升級網域資訊。
> 
> 

## <a name="placement-constraints-and-node-properties"></a>放置條件約束和節點屬性
有時候 (事實上是大部分的情況下) 您會想要確保工作負載只在叢集中的特定節點或一組特定的節點上執行。 例如，某些工作負載可能需要 GPU 或 SSD，而有些則不用。 幾乎每個多層式架構都是此情況的絕佳範例，其中特定機器會做為應用程式的前端/介面服務端 (因此通常會公開至網際網路)，而另一個不同的集合 (通常是使用不同的硬體資源) 會處理計算或儲存體層級的工作 (因此通常不會公開至網際網路)。 Service Fabric 預期甚至在微服務的世界中有特定工作負載，需要在特定硬體組態上執行，例如︰

* 現有的多層式架構應用程式已「提升並移轉」到 Service Fabric 環境
* 針對效能、級別或安全性隔離原因而想要在特定硬體上執行的工作負載
* 針對原則或資源耗用量原因而需要從其他工作負載隔離的工作負載

為了支援這種組態，Service Fabric 有我們稱為放置條件約束的第一級概念。 放置條件約束可以用於指出特定服務應在何處執行。 條件約束的集合可由使用者延伸，表示人員可以使用自訂屬性標記節點，然後也針對這些項目進行選取。

![叢集配置不同的工作負載][Image5]

節點上的不同索引鍵/值標記即為節點放置「屬性」(或僅稱為節點屬性)，而服務的陳述式則稱為放置「條件約束」。 節點屬性中指定的值可以是字串、bool，或帶正負號長值。 條件約束可以是任何在叢集中於不同節點上運作的布林值陳述式。 這些布林值陳述式 (也就是字串) 中的有效選取器是：

* 建立特定陳述式的條件式檢查
  * 「等於」==
  * 「大於」>
  * 「小於」<
  * 「不等於」!=
  * 「大於或等於」>=
  * 「小於或等於」<=
* 群組和否定的布林值陳述式
  * 「和」&&
  * 「或」||
  * 「非」!
* 群組作業的括號
  
  * ()
  
  以下是一些使用上述部分符號的基本條件約束陳述式範例。 請注意，節點屬性可以是字串、布林值或數值。   
  
  * "Foo >= 5"
  * "NodeColor != green"
  * "((OneProperty < 100) || ((AnotherProperty == false) && (OneProperty >= 100)))"

服務只能放置在整體陳述式評估為 "True" 的節點上。 未定義屬性的節點不符合包含該屬性的任何放置條件約束。

Service Fabric 也會定義一些預設屬性，可以自動使用，不需要使用者加以定義。 有鑑於此，在每個節點撰寫定義的預設屬性是 NodeType 和 NodeName。 因此舉例來說，您可以將放置條件約束撰寫成 "(NodeType == NodeType03)"。 通常我們發現 NodeType 是其中一個最常使用的屬性，因為它通常與機器的類型對應為 1:1，這會對應至傳統的多層式架構應用程式架構的工作負載類型。

![放置條件約束和節點屬性][Image6]

假設下列節點屬性是針對指定節點類型定義︰ClusterManifest.xml

```xml
    <NodeType Name="NodeType01">
      <PlacementProperties>
        <Property Name="HasSSD" Value="true"/>
        <Property Name="NodeColor" Value="green"/>
        <Property Name="SomeProperty" Value="5"/>
      </PlacementProperties>
    </NodeType>
```

您可以針對服務建立服務放置條件約束，如下所示︰

C#

```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
serviceDescription.PlacementConstraints = "(HasSSD == true && SomeProperty >= 4)";
// add other required servicedescription fields
//...
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceType -Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementConstraint "HasSSD == true && SomeProperty >= 4"
```

如果您確定 NodeType01 的所有節點都有效，您也可以只選取該節點類型，使用如上圖所示的放置條件約束。

服務放置條件約束其中很棒的一點是它們可以在執行階段動態更新。 所以如果您需要，可以在叢集中移動服務、加入和移除需求等等。Service Fabric 會負責確保服務保持執行且可用，即使這類變更持續進行。

C#：

```csharp
StatefulServiceUpdateDescription updateDescription = new StatefulServiceUpdateDescription();
updateDescription.PlacementConstraints = "NodeType == NodeType01";
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/app/service"), updateDescription);
```

PowerShell：

```posh
Update-ServiceFabricService -Stateful -ServiceName $serviceName -PlacementConstraints "NodeType == NodeType01"
```

放置條件約束 (以及我們即將討論的許多其他 Orchestrator 控制項) 是針對每個不同的已命名服務執行個體指定。 更新一律會取代 (覆寫) 先前指定的項目。

另外值得注意的是，目前節點上的屬性是透過叢集定義而定義的，因此無法在未升級叢集的情況下更新，且需要先讓每個節點關閉並再啟動以重新整理其屬性。

## <a name="capacity"></a>容量
任何 Orchestrator 的其中一個最重要的作業是協助管理叢集中的資源耗用量。 如果您想要有效率地執行服務，最不想遇到的情形是許多節點是熱的 (導致資源爭用和效能不佳)，而其他是冷的 (浪費資源)。 但是試著想想比平衡還要基本的事情 (在一分鐘內) – 只要確保節點不會在第一時間執行用完資源？

Service Fabric 以「度量」來代表資源。 度量是您想要向 Service Fabric 描述的任何邏輯或實體資源。 度量的範例是例如「WorkQueueDepth」或「MemoryInMb」的項目。 度量不同於放置條件約束和節點屬性，節點屬性通常是節點本身的靜態描述項，而度量則是有關節點所擁有的資源，以及服務在節點上執行時所使用的資源。 因此屬性看起來像是 HasSSD，可能會設定為 true 或 false，但是該 SSD 上的可用空間數量 (和服務的使用量) 就是類似「DriveSpaceInMb」的度量。 節點上的容量會將「DriveSpaceInMb」設定為磁碟機的非保留總空間量的數量，服務會報告在執行階段使用多少度量。

如果您關閉所有資源「平衡」 ，Service Fabric 的叢集資源管理員仍能確保不會有節點超出其容量 (除非叢集整體已經太滿)。 容量是另一個「條件約束」  ，叢集資源管理員會使用此條件約束來了解節點所擁有的資源量。 服務層級的容量和耗用量均以度量來表示。 因此舉例來說，度量可能是 "MemoryInMb"，且指定節點可能擁有 2048 單位的 MemoryInMb 容量，而指定服務可以說它目前正在使用 64 單位的 MemoryInMb。

在執行階段，叢集資源管理員會追蹤每個節點上的每個資源有多少 (依據其容量定義)，以及剩餘多少 (藉由減去每個服務所宣告的使用量)。 利用此資訊，Service Fabric 資源管理員可找出要放置或移動複本的位置，讓節點不會超過容量。

C#：

```csharp
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
ServiceLoadMetricDescription metric = new ServiceLoadMetricDescription();
metric.Name = "MemoryInMb";
metric.PrimaryDefaultLoad = 64;
metric.SecondaryDefaultLoad = 64;
metric.Weight = ServiceLoadMetricWeight.High;
serviceDescription.Metrics.Add(metric);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("Memory,High,64,64)
```

![叢集節點和容量][Image7]

您可以在叢集資訊清單中看到這些項目︰

ClusterManifest.xml

```xml
    <NodeType Name="NodeType02">
      <Capacities>
        <Capacity Name="MemoryInMb" Value="2048"/>
        <Capacity Name="Disk" Value="10000"/>
      </Capacities>
    </NodeType>
```

此外，服務的負載也會動態變更。 假設複本的負載從 64 變更成 1024，但是當時的正在執行該複本的節點上只剩下 512 (個單位的 "MemoryInMb" 度量)。 因為這樣，目前放置複本或執行個體的位置會變成無效，因為該節點上所有複本和執行個體的使用量總和超出該節點的容量。 我們稍後會更詳細討論負載可以動態變更的案例，但單就容量而言，系統都會以相同的方式處理 - 叢集資源管理會自動執行，並透過將該節點上的一或多個複本或執行個體移至不同的節點，來使節點不會超過容量。 執行這項操作時，叢集資源管理員會嘗試將所有移動的成本降到最低 (我們稍後再回頭討論「成本」的概念)。

## <a name="cluster-capacity"></a>叢集容量
那麼，我們要如何防止整體叢集太滿？ 使用動態負載時，實際上我們並沒有太多可以做的事 (因為服務本身也可能會發生獨立於叢集資源管理員所執行動作的負載尖峰。就算今天您的叢集具有許多空餘空間，但如果您在明天成名了，空間就會可能會變得不足)，但是有一些內建控制項可以防止基本問題。 我們可以做的第一件事是防止建立新的工作負載，該工作負載會導致叢集空間變滿。

假設您要建立簡單的無狀態服務，而且它具有某些與其相關聯的負載 (比預設值還多且動態負載稍後才報告)。 對於此服務，讓我們假設它關心某些資源 (例如磁碟空間)，依預設它會針對服務的每個執行個體使用 5 個單位的磁碟空間。 您想要建立服務的 3 個執行個體。 太棒了！ 因此表示我們需要叢集中有 15 個單位的磁碟空間，才能建立這些服務執行個體。 Service Fabric 持續計算整體容量和每個度量的耗用量，因此我們可以輕鬆地進行決定並且在空間不足時拒絕建立服務呼叫。

請注意，由於需求僅是 15 個可用單位，此空間可以使用不同的方式配置；例如，可能是在 15 個不同節點上各一個剩餘單位，或是在 5 個不同節點上各三個剩餘單位等等。如果三個不同節點上沒有足夠的容量，Service Fabric 將會重新組織已在叢集中的服務，以便在三個必要的節點上挪出空間。 這類的重新排列幾乎一律可行，除非整個叢集幾乎已滿。

## <a name="buffered-capacity"></a>緩衝處理的容量
另一個可以協助使用者管理整體叢集容量的概念，是對每個節點的指定容量加入一些保留緩衝區。 此設定是選擇性的，但是可以讓使用者保留整體節點容量的某些部分，讓它只會用來在升級和失敗期間放置服務 - 叢集的容量降低的情況。 現在緩衝區是透過 ClusterManifest 針對所有節點全域指定。 您針對保留容量挑選的值會是您的服務更受到條件約束的資源的函數，以及您在叢集中具有的容錯和升級網域的數目。 通常較多的容錯和升級網域表示您可以挑選較少數目的緩衝處理容量，因為您會預期在升級和失敗期間有較少量的叢集無法使用。 請注意，指定緩衝區百分比只有在您也指定了度量的節點容量時才具有意義。

以下是指定緩衝處理容量的方式：

ClusterManifest.xml

```xml
        <Section Name="NodeBufferPercentage">
            <Parameter Name="DiskSpace" Value="0.10" />
            <Parameter Name="Memory" Value="0.15" />
            <Parameter Name="SomeOtherMetric" Value="0.20" />
        </Section>
```

如果叢集已耗盡緩衝處理容量，將會導致建立新服務失敗，請確保叢集保留足夠的備用額外負荷，讓升級和失敗不會造成節點實際超過容量。 叢集資源管理員已透過 PowerShell 和查詢 API 公開許多此類資訊，供您查看緩衝處理容量設定、總容量，以及叢集中每個使用中度量的目前耗用量。 我們可以看到該輸出的範例如下︰

```posh
PS C:\Users\user> Get-ServiceFabricClusterLoadInformation
LastBalancingStartTimeUtc : 9/1/2015 12:54:59 AM
LastBalancingEndTimeUtc   : 9/1/2015 12:54:59 AM
LoadMetricInformation     :
                            LoadMetricName        : Metric1
                            IsBalancedBefore      : False
                            IsBalancedAfter       : False
                            DeviationBefore       : 0.192450089729875
                            DeviationAfter        : 0.192450089729875
                            BalancingThreshold    : 1
                            Action                : NoActionNeeded
                            ActivityThreshold     : 0
                            ClusterCapacity       : 189
                            ClusterLoad           : 45
                            ClusterRemainingCapacity : 144
                            NodeBufferPercentage  : 10
                            ClusterBufferedCapacity : 170
                            ClusterRemainingBufferedCapacity : 125
                            ClusterCapacityViolation : False
                            MinNodeLoadValue      : 0
                            MinNodeLoadNodeId     : 3ea71e8e01f4b0999b121abcbf27d74d
                            MaxNodeLoadValue      : 15
                            MaxNodeLoadNodeId     : 2cc648b6770be1bc9824fa995d5b68b1
```

## <a name="next-steps"></a>後續步驟
* 如需叢集資源管理員內的架構和資訊流程的相關資訊，請查看 [這篇文章 ](service-fabric-cluster-resource-manager-architecture.md)
* 定義重組度量是合併 (而不是擴增) 節點上負載的一種方式。 若要了解如何設定重組，請參閱 [這篇文章](service-fabric-cluster-resource-manager-defragmentation-metrics.md)
* 從頭開始，並 [取得 Service Fabric 叢集資源管理員的簡介](service-fabric-cluster-resource-manager-introduction.md)
* 若要了解叢集資源管理員如何管理並平衡叢集中的負載，請查看關於 [平衡負載](service-fabric-cluster-resource-manager-balancing.md)

[Image1]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-domains.png
[Image2]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-uneven-fault-domain-layout.png
[Image3]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-and-upgrade-domains-with-placement.png
[Image4]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-and-upgrade-domain-layout-strategies.png
[Image5]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-layout-different-workloads.png
[Image6]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-placement-constraints-node-properties.png
[Image7]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-nodes-and-capacity.png



<!--HONumber=Nov16_HO3-->


