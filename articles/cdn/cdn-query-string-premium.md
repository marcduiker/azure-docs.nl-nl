---
title: Beheren van Azure CDN cachegedrag met queryreeksen - Premium | Microsoft Docs
description: Azure CDN-queryreeks opslaan in cache bepaalt hoe bestanden worden in de cache worden opgeslagen wanneer ze queryreeksen bevatten.
services: cdn
documentationcenter: 
author: zhangmanling
manager: erikre
editor: 
ms.assetid: 99db4a85-4f5f-431f-ac3a-69e05518c997
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/09/2017
ms.author: mazha
ms.openlocfilehash: ba9c28f0e6df25b101b45edf836d0b95056cbc6f
ms.sourcegitcommit: 6a22af82b88674cd029387f6cedf0fb9f8830afd
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 11/11/2017
---
# <a name="control-azure-content-delivery-network-caching-behavior-with-query-strings---premium"></a>Besturingselement Azure Content Delivery Network cachegedrag met queryreeksen - Premium
> [!div class="op_single_selector"]
> * [Standard](cdn-query-string.md)
> * [Azure CDN Premium van Verizon](cdn-query-string-premium.md)
> 
> 

## <a name="overview"></a>Overzicht
Met Azure Content Delivery Network (CDN), kunt u bepalen hoe bestanden in cache zijn opgeslagen voor een webaanvraag die een queryreeks bevat. In een webaanvraag met een queryreeks bevat de queryreeks is het gedeelte van de aanvraag die deze gebeurtenis treedt op nadat de `?` teken. Een queryreeks kunt bevatten een of meer parameters die worden gescheiden door een `&` teken. Bijvoorbeeld `http://www.domain.com/content.mov?data1=true&data2=false`. Als er meer dan een queryreeksparameter opgeven in een aanvraag, is de volgorde van de parameters niet van belang. 

> [!IMPORTANT]
> De standard en premium-CDN-producten dezelfde querytekenreeks cache-functionaliteit bieden, maar de gebruikersinterface is.  In dit artikel beschrijft de interface voor **Azure CDN Premium van Verizon**. Voor deze query opslaan in cache met **Azure CDN Standard van Akamai** en **Azure CDN Standard van Verizon**, Zie [beheren cachegedrag van CDN aanvragen met querytekenreeksen](cdn-query-string.md).
>

Drie beschikbare modi voor query-tekenreeks zijn beschikbaar:

- **Standard-cache**: standaardmodus. In deze modus wordt het CDN edge-knooppunt geeft de queryreeksen van de aanvrager naar de oorsprong van de eerste aanvraag en plaatst de asset. Alle volgende aanvragen voor de asset die worden aangeboden via het edge-knooppunt negeert de queryreeksen totdat de activa in de cache verloopt.
- **Er is geen cache**: In deze modus kan aanvragen met queryreeksen zijn niet in cache opgeslagen op de edge-knooppunt van het CDN. Het edge-knooppunt ophalen van de asset rechtstreeks vanuit de oorsprong en wordt doorgegeven aan de aanvrager aan elke aanvraag.
- **unieke cache**: In deze modus kan elke aanvraag met een unieke URL, met inbegrip van de queryreeks wordt behandeld als een unieke activum met een eigen cache. Bijvoorbeeld: de reactie van de oorsprong van een aanvraag voor `example.ashx?q=test1` is opgeslagen in de cache op de edge-knooppunt en voor daaropvolgende caches met dezelfde querytekenreeks geretourneerd. Een aanvraag voor `example.ashx?q=test2` in cache wordt opgeslagen als een afzonderlijk actief door een eigen time-to-live-instelling.

## <a name="changing-query-string-caching-settings-for-premium-cdn-profiles"></a>Het wijzigen van de queryreeks opslaan in cache-instellingen voor premium-CDN-profielen
1. Een CDN-profiel te openen en klik vervolgens op **beheren**.
   
    ![Knop voor CDN-profiel beheren](./media/cdn-query-string-premium/cdn-manage-btn.png)
   
    Hiermee opent u de CDN-beheerportal.
2. Beweeg de muisaanwijzer over de **HTTP grote** tabblad en klik vervolgens Beweeg de muisaanwijzer over de **Cache-instellingen** doel. Klik op **queryreeks Caching**.
   
    Queryreeks cache-opties worden weergegeven.
   
    ![CDN-queryreeks cacheopties](./media/cdn-query-string-premium/cdn-query-string.png)
3. Selecteer een modus voor query-tekenreeks en klik vervolgens op **Update**.

> [!IMPORTANT]
> Omdat het duurt om de registratie te geven in CDN, wijzigingen in de cache tekenreeks mogelijk niet meteen zichtbaar. Voor **Azure CDN Premium van Verizon** profielen, doorgeven meestal is binnen 90 minuten worden voltooid, maar in sommige gevallen kan langer duren.
 

