---
title: Load Balancer aangepaste tests en de status controleren | Microsoft Docs
description: Informatie over het gebruik van aangepaste tests voor de Azure Load Balancer instanties achter de Load Balancer bewaken
services: load-balancer
documentationcenter: na
author: KumudD
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 46b152c5-6a27-4bfc-bea3-05de9ce06a57
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: kumud
ms.openlocfilehash: 102c07ff0994b3b411f2a13d7a43c5398d5dfd42
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/11/2017
---
# <a name="understand-load-balancer-probes"></a>Load Balancer-testen

[!INCLUDE [load-balancer-basic-sku-include.md](../../includes/load-balancer-basic-sku-include.md)]

Azure Load Balancer biedt de mogelijkheid om te controleren van de status van exemplaren van de server met behulp van de tests. Wanneer een test niet reageert, stopt Load Balancer nieuwe verbindingen verzenden naar het exemplaar niet in orde. De bestaande verbindingen worden niet beïnvloed en nieuwe verbindingen worden verzonden naar orde exemplaren.

Guest agent cloud service-rollen (werkrollen en webrollen) gebruiken voor het bewaken van de test. Aangepaste TCP- of HTTP-tests moeten worden geconfigureerd wanneer u virtuele machines achter de Load Balancer gebruiken.

## <a name="understand-probe-count-and-timeout"></a>Aantal van de test- en time begrijpen

Afhankelijk van het gedrag van de test:

* Het aantal geslaagde tests waarmee een exemplaar moet het worden gelabeld als bedrijfs.
* Het aantal mislukte tests die ertoe leiden een exemplaar dat moet het worden gelabeld als omlaag.

De waarde voor time-outs en frequentie in SuccessFailCount bepalen of een exemplaar moet worden uitgevoerd of niet actief is bevestigd. In de Azure portal, is de time-out ingesteld op tweemaal de waarde van de frequentie.

De configuratie van de test van alle exemplaren met gelijke taakverdeling voor een eindpunt (dat wil zeggen, een set taakverdeling) moet hetzelfde zijn. Dit betekent dat er geen een andere test-configuratie voor elke rolinstantie of de virtuele machine in dezelfde gehoste service voor de combinatie van een bepaald eindpunt. Elk exemplaar moet bijvoorbeeld identieke lokale poorten en time-outs.

> [!IMPORTANT]
> Een Load Balancer-test maakt gebruik van het IP-adres 168.63.129.16. Dit openbare IP-adres vergemakkelijkt de communicatie met interne platform bronnen voor de bring-your-eigenaar-IP-scenario in Azure Virtual Network. De virtuele openbare IP-adres 168.63.129.16 wordt gebruikt in alle regio's en wordt niet gewijzigd. We raden u aan dit IP-adres in een lokale firewall-beleid. Deze mogen niet worden beschouwd als een beveiligingsrisico met zich mee omdat alleen de interne Azure-platform kan een bericht van dat adres bron. Als u dit doet, zullen er onverwacht gedrag in verschillende scenario's zoals het configureren van hetzelfde IP-adresbereik van 168.63.129.16 en IP-adressen die worden gedupliceerd.

## <a name="learn-about-the-types-of-probes"></a>Meer informatie over de typen tests

### <a name="guest-agent-probe"></a>Test voor gast-agent

Deze test is alleen beschikbaar voor Azure Cloud Services. Load Balancer gebruikmaakt van de guest-agent op de virtuele machine, en vervolgens luistert en reageert met een HTTP 200 OK antwoord wanneer het exemplaar in de status gereed (dat wil zeggen, niet in een andere status zoals bezet, Recycling of stoppen).

Zie voor meer informatie [het servicedefinitiebestand (csdef) configureren voor statuscontroles](https://msdn.microsoft.com/library/azure/ee758710.aspx) of [maken van een Internet gerichte load balancer voor cloudservices aan de slag](load-balancer-get-started-internet-classic-cloud.md#check-load-balancer-health-status-for-cloud-services).

### <a name="what-makes-a-guest-agent-probe-mark-an-instance-as-unhealthy"></a>Een test voor gast-agent is een exemplaar als beschadigd gemarkeerd is?

Als de guest agent niet reageert met HTTP 200 OK, wordt de load balancer markeert de instantie als niet-reagerende en stopt verkeer kunnen verzenden naar dit exemplaar. De load balancer, blijft het pingen van het exemplaar. Als u de guest-agent reageert met een HTTP 200, verzendt de load balancer verkeer opnieuw naar dit exemplaar.

Wanneer u een Webrol, de websitecode doorgaans wordt uitgevoerd in w3wp.exe die niet wordt bewaakt door de Azure fabric of Gast-agent. Dit betekent dat fouten in w3wp.exe (bijvoorbeeld HTTP 500-antwoorden) wordt niet gerapporteerd aan de guest-agent en de load balancer niet in werking te dat exemplaar buiten de draaihoek laten.

### <a name="http-custom-probe"></a>Aangepaste HTTP-test

De aangepaste HTTP-Load Balancer-test heeft voorrang op de standaard guest agent test, wat betekent dat u uw eigen aangepaste regels om te bepalen van de status van de rolinstantie kunt maken. De load balancer-tests uw eindpunt elke 15 seconden standaard. Het exemplaar wordt beschouwd als in de load balancer rotatie als deze met een HTTP 200 binnen de time-outperiode (31 seconden standaard reageert).

Dit is handig als u implementeren van uw eigen logica wilt voor het verwijderen van exemplaren van load balancer worden gedraaid. U kan bijvoorbeeld besluiten om te verwijderen van een exemplaar als het hoger dan 90% CPU is en de status van een niet-200 retourneert. Als u web-functies die er gebruik van w3wp.exe hebt, betekent dit ook dat krijgt u automatische bewaking van uw website, omdat er fouten in de websitecode van uw de status van een niet-200 wordt geretourneerd naar de load balancer-test.

> [!NOTE]
> De aangepaste HTTP-test ondersteunt relatieve paden en HTTP-protocol. HTTPS wordt niet ondersteund.

### <a name="what-makes-an-http-custom-probe-mark-an-instance-as-unhealthy"></a>Een aangepaste HTTP-test een exemplaar als beschadigd gemarkeerd is?

* De HTTP-toepassing retourneert een HTTP-antwoordcode dan 200 (bijvoorbeeld 403, 404 of 500). Dit is een positieve bevestiging dat exemplaar van de toepassing moet worden uitgevoerd buiten de service meteen.
* De HTTP-server reageert niet op alle na de time-outperiode. Afhankelijk van de time-outwaarde die is ingesteld, kan dit betekenen dat meerdere test Ga voordat de test wordt gemarkeerd als niet wordt uitgevoerd met openstaande aanvragen (dat wil zeggen, voordat SuccessFailCount tests worden verzonden).
* De server sluit de verbinding via een TCP-reset.

### <a name="tcp-custom-probe"></a>Aangepaste TCP-test

Een verbinding starten TCP tests door het uitvoeren van een handshake drie richtingen met de opgegeven poort.

### <a name="what-makes-a-tcp-custom-probe-mark-an-instance-as-unhealthy"></a>Een aangepaste TCP-test een exemplaar als beschadigd gemarkeerd is?

* De TCP-server reageert niet op alle na de time-outperiode. Wanneer de test is gemarkeerd als niet wordt uitgevoerd, is afhankelijk van het aantal mislukte test aanvragen die zijn geconfigureerd om te gaan beantwoorde voordat u markeert de test als niet wordt uitgevoerd.
* De test ontvangt een opnieuw instellen van de rolinstantie TCP.

Zie voor meer informatie over het configureren van de statuscontrole van een HTTP- of een TCP-controle [een internetgerichte load balancer maken in Resource Manager, met behulp van PowerShell](load-balancer-get-started-internet-arm-ps.md).

## <a name="add-healthy-instances-back-into-load-balancer-rotation"></a>In orde exemplaren weer in de load balancer rotatie toevoegen

TCP- en HTTP-tests worden beschouwd als in orde en markeert de rolinstantie als in orde wanneer:

* De load balancer opgehaald uit een positief test de eerste keer dat de virtuele machine wordt opgestart.
* Het aantal SuccessFailCount (zoals eerder beschreven) definieert de waarde van geslaagde tests die nodig zijn voor de rolinstantie als goed markeren. Als een rolinstantie werd verwijderd, moet het aantal geslaagde, opeenvolgende tests gelijk zijn aan of groter zijn dan de waarde van SuccessFailCount markeren van de rolinstantie als die wordt uitgevoerd.

> [!NOTE]
> Als u de status van een rolinstantie is veranderen, wordt de load balancer meer wacht voordat u de rolinstantie terug in de status in orde. Dit wordt gedaan via beleid voor het beveiligen van de gebruiker en de infrastructuur.

## <a name="use-log-analytics-for-load-balancer"></a>Log analytics gebruiken voor de Load Balancer

U kunt [analytics aanmelden voor de Load Balancer](load-balancer-monitor-log.md) bij het controleren van de status van de test en het aantal van de test. Logboekregistratie kan worden gebruikt met Power BI of Azure Operational Insights om statistieken over de status van de Load Balancer.
