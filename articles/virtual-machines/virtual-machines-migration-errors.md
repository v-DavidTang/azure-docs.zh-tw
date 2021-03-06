---
title: "從傳統到 Azure Resource Manager 移轉的常見錯誤 |Microsoft Docs"
description: "本文收錄將 IaaS 資源從 Azure 服務管理移轉至 Azure Resource Manager 堆疊的常見錯誤和緩和措施。"
services: virtual-machines-windows
documentationcenter: 
author: singhkays
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 5bc03a1b-eb1c-438c-83d9-f0e9d61f1b6a
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 10/13/2016
ms.author: singhkay
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 345db9b2e45937ea45329acf780601e4e0fbd504


---
# <a name="common-errors-during-classic-to-azure-resource-manager-migration"></a>從傳統到 Azure Resource Manager 移轉的常見錯誤
本文收錄將 IaaS 資源從 Azure 傳統部署模型移轉至 Azure Resource Manager 堆疊的常見錯誤和緩和措施。

## <a name="list-of-errors"></a>錯誤清單
| 錯誤字串 | 緩和 |
| --- | --- |
| 內部伺服器錯誤 |在某些情況下，這是隨著重試消失的暫時性錯誤。 如果持續發生，請[連絡 Azure 支援](../azure-supportability/how-to-create-azure-support-request.md)，因為需要平台記錄檔的調查。 <br><br> **注意︰**一旦支援小組追蹤事件，請不要嘗試任何自我緩和，這可能會使您的環境中產生非預期的結果。 |
| 移轉不支援 HostedService {hosted-service-name} 中的部署 {deployment-name}，因為它是 PaaS 部署 (Web/背景工作角色)。 |這會在部署包含 Web/背景工作角色時發生。 因為移轉僅支援虛擬機器，請從部署移除 Web/背景工作角色，然後再試一次移轉。 |
| 範本 {template-name} 部署失敗。 相互關聯識別碼 = {guid} |在移轉服務後端，我們可以使用 Azure Resource Manager 範本來建立 Azure Resource Manager 堆疊中的資源。 因為範本具有等冪性，通常您可以安全地重試移轉作業，以通過這項錯誤。 如果此錯誤持續存在，請[連絡 Azure 支援](../azure-supportability/how-to-create-azure-support-request.md)，並給予他們相互關聯識別碼。 <br><br> **注意︰**一旦支援小組追蹤事件，請不要嘗試任何自我緩和，這可能會使您的環境中產生非預期的結果。 |
| 虛擬網路 {virtual-network-name} 不存在。 |如果您在新的 Azure 入口網站中建立虛擬網路，會發生這種情形。 實際虛擬網路名稱遵循 "Group * <VNET name>" 模式 |
| HostedService {hosted-service-name} 中的 VM {vm-name} 包含擴充 {extension-name}，在 Azure Resource Manager 中不支援。 建議您先從 VM 中將它解除安裝，再繼續進行移轉。 |XML 擴充，例如 BGInfo 1.*，在 Azure Resource Manager 中不支援。 因此，這些擴充功能不能移轉。 如果這些擴充功能保持安裝在虛擬機器上，它們會在完成移轉之前自動解除安裝。 |
| HostedService {hosted-service-name} 中的 VM {vm-name} 包含擴充 VMSnapshot/VMSnapshotLinux，目前不支援移轉。 從 VM 解除安裝，並且在移轉完成之後使用 Azure Resource Manager 將它新增回來 |這是虛擬機器針對 Azure 備份進行設定的案例。 由於這是目前不支援的案例，請遵循 https://aka.ms/vmbackupmigration 的因應措施 |
| HostedService {hosted-service-name} 中的 VM {vm-name} 包含擴充 {extension-name}，其狀態未從 VM 報告。 因此，無法移轉此 VM。 請確定擴充的狀態已報告，或從 VM 解除安裝擴充，然後重試移轉。 <br><br> HostedService {hosted-service-name} 中的 VM {vm-name} 包含擴充 {extension-name}，報告處理常式狀態：{handler-status}。 因此，無法移轉 VM。 請確定報告的擴充處理常式狀態是 {handler-status}，或從 VM 解除安裝，然後重試移轉。 <br><br> HostedService {hosted-service-name} 中 VM {vm-name} 的 VM 代理程式將整體代理程式狀態報告為「未就緒」。 因此，VM 可能不會移轉，如果它有可移轉的擴充。 請確定 VM 代理程式將整體代理程式狀態報告為「就緒」。 請參閱 https://aka.ms/classiciaasmigrationfaqs。 |Azure 客體代理程式與 VM 擴充需要 VM 儲存體帳戶的輸出網際網路存取，來填入其狀態。 狀態失敗的常見原因包括 <li> 封鎖輸出網際網路存取的網路安全性群組 <li> 如果 VNET 有內部部署 DNS 伺服器且 DNS 連線遺失 <br><br> 如果您持續看到不支援的狀態，您可以解除安裝擴充以略過這項檢查，然後繼續進行移轉。 |
| 移轉不支援 HostedService {hosted-service-name} 中的部署 {deployment-name}，因為它有多個可用性設定組。 |目前，只能移轉具有 1 或更少可用性設定組的託管服務。 若要解決這個問題，請將額外可用性設定組和這些可用性設定組中的虛擬機器移至不同的託管服務。 |
| 移轉不支援 HostedService {hosted-service-name} 中的部署 {deployment-name}，因為它有不屬於可用性設定組的 VM，即使 HostedService 包含一個 VM。 |此案例的因應措施是將所有虛擬機器都移到單一可用性設定組，或在託管服務中從可用性設定組移除所有虛擬機器。 |
| 儲存體帳戶/HostedService/虛擬網路 {virtual-network-name} 正在移轉，因此無法變更 |在資源上已完成「準備」移轉作業，並且觸發會對資源進行變更的作業時，就會發生這個錯誤。 因為在「準備」作業之後會鎖定管理平面，所以對資源的任何變更都會遭到封鎖。 若要解除鎖定管理平面，您可以執行「認可」移轉作業以完成移轉，或「中止」移轉作業以回復「準備」作業。 |
| HostedService {hosted-service-name} 不允許移轉，因為它有 VM {vm-name} 處於下列「狀態」：RoleStateUnknown。 只有在 VM 處於下列其中一種狀態時才允許移轉 - 執行中、已停止、已停止解除配置。 |VM 可能會在轉換狀態時進行，通常發生在 HostedService 上的更新作業期間，例如重新啟動、擴充安裝等。建議在嘗試移轉之前在 HostedService 上完成更新作業。 |

## <a name="next-steps"></a>後續步驟
以下是說明處理的移轉文章清單。

* [平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager)](virtual-machines-windows-migration-classic-resource-manager.md)
* [使用 PowerShell 將 IaaS 資源從傳統移轉至 Azure Resource Manager](virtual-machines-windows-ps-migration-classic-resource-manager.md)
* [使用 CLI 將 IaaS 資源從傳統移轉至 Azure Resource Manager](virtual-machines-linux-cli-migration-classic-resource-manager.md)




<!--HONumber=Nov16_HO3-->


