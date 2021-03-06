---
title: Een Docker-installatiekopie maken voor Model Management in Azure Machine Learning | Microsoft Docs
description: In dit artikel beschrijft de stappen voor het maken van een Docker-afbeelding voor uw webservice.
services: machine-learning
author: chhavib
ms.author: chhavib
manager: neerajkh
editor: jasonwhowell
ms.service: machine-learning
ms.workload: data-services
ms.devlang: na
ms.topic: article
ms.date: 09/20/2017
ms.openlocfilehash: 1939a18fbaf0aed0f62ac9e7641b9901ec4762ea
ms.sourcegitcommit: e6029b2994fa5ba82d0ac72b264879c3484e3dd0
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/24/2017
---
# <a name="azure-machine-learning-model-management-account-api-reference"></a>Naslaginformatie over Azure Machine Learning-Model Management Account API

Zie voor meer informatie over het instellen van de implementatieomgeving [Model-Management-accountconfiguratie](model-management-configuration.md).

Azure Machine Learning-Model Management Account API implementeert de volgende bewerkingen:

- Modelregistratie
- Maken van het manifest
- Maken van de docker-installatiekopie
- Maken van de Web service

U kunt deze installatiekopie gebruiken voor het maken van een webservice lokaal of op een externe Azure Container Service-cluster of een andere Docker-ondersteunde omgeving van uw keuze.

## <a name="prerequisites"></a>Vereisten
Zorg ervoor dat u hebt doorlopen tot de installatiestappen in de [installeren en maak Quick Start](quickstart-installation.md) document.

Het volgende is vereist voordat u verdergaat:
1. Model account inrichting
2. Het maken van de omgeving voor het implementeren en beheren van modellen
3. Een Machine Learning-model

### <a name="azure-ad-token"></a>Azure AD-token
Als u gebruikmaakt van Azure CLI aanmelden met behulp van `az login`. De CLI maakt gebruik van uw Azure Active Directory (Azure AD)-token uit het bestand .azure. Als u de API's gebruiken wilt, hebt u de volgende opties.

##### <a name="acquire-the-azure-ad-token-manually"></a>Handmatig de Azure AD-token ophalen
U kunt `az login` en de meest recente-token ophalen uit het bestand .azure op uw basismap.

##### <a name="acquire-the-azure-ad-token-programmatically"></a>Programmatisch de Azure AD-token ophalen
```
az ad sp create-for-rbac --scopes /subscriptions/<SubscriptionId>/resourcegroups/<ResourceGroupName> --role Contributor --years <length of time> --name <MyservicePrincipalContributor>
```
Nadat u de service-principal gemaakt, kunt u de uitvoer opslaan. Nu kunt u die een token ophalen uit Azure AD:

```cs
 private static async Task<string> AcquireTokenAsync(string clientId, string password, string authority, string resource)
{
        var creds = new ClientCredential(clientId, password);
        var context = new AuthenticationContext(authority);
        var token = await context.AcquireTokenAsync(resource, creds).ConfigureAwait(false);
        return token.AccessToken;
}
```
Het token wordt geplaatst in een autorisatie-header voor API-aanroepen.


## <a name="register-a-model"></a>Registreren van een model

De model-registratiestap registreert uw Machine Learning-model met de Azure-Model-Management-account dat u hebt gemaakt. Deze registratie kunnen bijhouden van de modellen en hun versies die zijn toegewezen op het moment van inschrijving. De gebruiker bevat de naam van het model. Volgende registratie van modellen onder dezelfde naam genereert een nieuwe versie en -ID.

### <a name="request"></a>Aanvraag

| Methode | Aanvraag-URI |
|------------|------------|
| VERZENDEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} / modellen 
 |
### <a name="description"></a>Beschrijving
Registreert een model.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| Model | Hoofdtekst | De nettolading die wordt gebruikt voor het registreren van een model. | Ja | [Model](#model) |


### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | OK. De modelregistratie is voltooid. | [Model](#model) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-models-in-an-account"></a>Query uitvoeren op de lijst met modellen in een account
### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} / modellen 
 |
### <a name="description"></a>Beschrijving
De lijst met modellen in een account een query. U kunt de lijst met resultaten op label en naam filteren. Als geen filter wordt doorgegeven, bevat de query alle modellen in het account. De geretourneerde lijst is gepagineerd en het aantal items in elke pagina is een optionele parameter.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| naam | query | Objectnaam. | Nee | Tekenreeks |
| Label | query | Model label. | Nee | Tekenreeks |
| Aantal | query | Aantal items in een pagina ophalen. | Nee | Tekenreeks |
| $skipToken | query | Het vervolgtoken voor het ophalen van de volgende pagina. | Nee | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [PaginatedModelList](#paginatedmodellist) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |

## <a name="get-model-details"></a>Modeldetails van het ophalen
### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} /models/ {id}  
 |

### <a name="description"></a>Beschrijving
Een model met id opgehaald

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| id | Pad | Object-ID. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [Model](#model) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |

## <a name="register-a-manifest-with-the-registered-model-and-all-dependencies"></a>Een manifest te registreren met het geregistreerde model en alle afhankelijkheden

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| VERZENDEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} / manifesten | 

### <a name="description"></a>Beschrijving
Registreert een manifest met het geregistreerde model en de afhankelijkheden ervan.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| manifestRequest | Hoofdtekst | De nettolading die wordt gebruikt voor het registreren van een manifest. | Ja | [Manifest](#manifest) |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Manifest registratie is gelukt. | [Manifest](#manifest) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-manifests-in-an-account"></a>Query uitvoeren op de lijst van de manifesten in een account

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} / manifesten | 

### <a name="description"></a>Beschrijving
De lijst van de manifesten in een account met een query. U kunt het resultatenlijst filteren op model-ID en naam manifest. Als geen filter wordt doorgegeven, bevat de query de manifesten in het account. De geretourneerde lijst is gepagineerd en het aantal items in elke pagina is een optionele parameter.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| Model | query | Model-ID. | Nee | Tekenreeks |
| ManifestName | query | Naam van het manifest. | Nee | Tekenreeks |
| Aantal | query | Aantal items in een pagina ophalen. | Nee | Tekenreeks |
| $skipToken | query | Het vervolgtoken voor het ophalen van de volgende pagina. | Nee | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [PaginatedManifestList](#paginatedmanifestlist) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |

## <a name="get-manifest-details"></a>Details van het manifest ophalen

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} /manifests/ {id} | 

### <a name="description"></a>Beschrijving
Haalt het manifest van-ID.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| id | Pad | Object-ID. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [Manifest](#manifest) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |

## <a name="create-an-image"></a>Een installatiekopie maken

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| VERZENDEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} / installatiekopieën | 

### <a name="description"></a>Beschrijving
Hiermee maakt u een installatiekopie van het als een Docker-installatiekopie in Azure Container register.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| imageRequest | Hoofdtekst | De nettolading die wordt gebruikt om een installatiekopie te maken. | Ja | [ImageRequest](#imagerequest) |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Headers | Schema |
|--------------------|--------------------|--------------------|--------------------|
| 202 | Asynchrone bewerking locatie-URL. Een GET-aanroep ziet u de status van de installatiekopie van het maken van taak. | Locatie van de bewerking |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-images-in-an-account"></a>Query uitvoeren op de lijst met afbeeldingen in een account

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} / installatiekopieën | 

### <a name="description"></a>Beschrijving
De lijst met afbeeldingen in een account een query. U kunt de lijst met resultaten door de manifest-ID en naam filteren. Als geen filter wordt doorgegeven, bevat de query de afbeeldingen in het account. De geretourneerde lijst is gepagineerd en het aantal items in elke pagina is een optionele parameter.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| manifest-id | query | Manifest-ID. | Nee | Tekenreeks |
| ManifestName | query | Naam van het manifest. | Nee | Tekenreeks |
| Aantal | query | Aantal items in een pagina ophalen. | Nee | Tekenreeks |
| $skipToken | query | Het vervolgtoken voor het ophalen van de volgende pagina. | Nee | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [PaginatedImageList](#paginatedimagelist) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |

## <a name="get-image-details"></a>Details van de installatiekopie ophalen

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} /images/ {id} | 

### <a name="description"></a>Beschrijving
Een installatiekopie met id opgehaald

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| id | Pad | Installatiekopie-ID. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [Afbeelding](#image) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |


## <a name="create-a-service"></a>Een service maken

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| VERZENDEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} / services | 

### <a name="description"></a>Beschrijving
Maakt een service van een installatiekopie.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| Serviceaanvraag | Hoofdtekst | De nettolading die wordt gebruikt voor het maken van een service. | Ja | [ServiceCreateRequest](#servicecreaterequest) |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Headers | Schema |
|--------------------|--------------------|--------------------|--------------------|
| 202 | Asynchrone bewerking locatie-URL. Een GET-aanroep ziet u de status van de taak service maken. | Locatie van de bewerking |
| 409 | Er bestaat al een service met de opgegeven naam. |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-services-in-an-account"></a>Query uitvoeren op de lijst met services in een account

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} / services | 

### <a name="description"></a>Beschrijving
De lijst met services in een account een query. U kunt het resultatenlijst filteren op naam-/ ID model, manifest naam-ID, afbeeldings-ID, servicenaam of Machine Learning compute resource-ID. Als geen filter wordt doorgegeven, bevat de query alle services in het account. De geretourneerde lijst is gepagineerd en het aantal items in elke pagina is een optionele parameter.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| Servicenaam | query | De naam van de service. | Nee | Tekenreeks |
| Model | query | Naam van het model. | Nee | Tekenreeks |
| modelName | query | Model-ID. | Nee | Tekenreeks |
| manifest-id | query | Manifest-ID. | Nee | Tekenreeks |
| ManifestName | query | Naam van het manifest. | Nee | Tekenreeks |
| imageId | query | Installatiekopie-ID. | Nee | Tekenreeks |
| computeResourceId | query | Machine Learning compute resource-ID. | Nee | Tekenreeks |
| Aantal | query | Aantal items in een pagina ophalen. | Nee | Tekenreeks |
| $skipToken | query | Het vervolgtoken voor het ophalen van de volgende pagina. | Nee | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [PaginatedServiceList](#paginatedservicelist) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse) |

## <a name="get-service-details"></a>Servicedetails van ophalen

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} /services/ {id} | 

### <a name="description"></a>Beschrijving
Een service met id opgehaald

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| id | Pad | Object-ID. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [ServiceResponse](#serviceresponse) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse)

## <a name="update-a-service"></a>Een service bijwerken

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| PLAATSEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} /services/ {id} | 

### <a name="description"></a>Beschrijving
Een bestaande service-updates.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| id | Pad | Object-ID. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| serviceUpdateRequest | Hoofdtekst | De nettolading die wordt gebruikt voor het bijwerken van een bestaande service. | Ja |  [ServiceUpdateRequest](#serviceupdaterequest) |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Headers | Schema |
|--------------------|--------------------|--------------------|--------------------|
| 202 | Asynchrone bewerking locatie-URL. Een GET-aanroep ziet u de status van de taak van de service-update. | Locatie van de bewerking |
| 404 | De service met de opgegeven ID bestaat niet. |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse)

## <a name="delete-a-service"></a>Een service verwijderen

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| VERWIJDEREN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} /services/ {id} | 

### <a name="description"></a>Beschrijving
Hiermee verwijdert u een service.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| id | Pad | Object-ID. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. |  |
| 204 | De service met de opgegeven ID bestaat niet. |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse)

## <a name="get-service-keys"></a>Service-sleutels ophalen

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} /services/ {id} / sleutels | 

### <a name="description"></a>Beschrijving
Sleutels service opgehaald.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| id | Pad | Service-ID. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [AuthKeys](#authkeys)
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse)

## <a name="regenerate-service-keys"></a>Opnieuw genereren van sleutels service

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| VERZENDEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} /services/ {id} / sleutels | 

### <a name="description"></a>Beschrijving
Service sleutels genereert en vrijgegeven, gaan ze.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| id | Pad | Service-ID. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| regenerateKeyRequest | Hoofdtekst | De nettolading die wordt gebruikt voor het bijwerken van een bestaande service. | Ja | [ServiceRegenerateKeyRequest](#serviceregeneratekeyrequest) |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [AuthKeys](#authkeys)
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse)

## <a name="query-the-list-of-deployments-in-an-account"></a>Query uitvoeren op de lijst met implementaties in een account

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} / implementaties | 

### <a name="description"></a>Beschrijving
De lijst met implementaties in een account een query. U kunt de lijst met resultaten door de service-ID, die alleen de implementaties die zijn gemaakt voor de desbetreffende service retourneert filteren. Als geen filter wordt doorgegeven, bevat de query alle implementaties in het account.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |
| serviceId | query | Service-ID. | Nee | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [DeploymentList](#deploymentlist) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse)

## <a name="get-deployment-details"></a>Implementatiedetails ophalen

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} /deployments/ {id} | 

### <a name="description"></a>Beschrijving
Opgehaald van de implementatie van-ID.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| id | Pad | Implementatie-ID. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [Implementatie](#deployment) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse)

## <a name="get-operation-details"></a>Bewerkingsdetails van de ophalen

### <a name="request"></a>Aanvraag
| Methode | Aanvraag-URI |
|------------|------------|
| TOEVOEGEN |  /API/subscriptions / {subscriptionId} /resourceGroups/ {resourceGroupName} /accounts/ {accountName} /operations/ {id} | 

### <a name="description"></a>Beschrijving
Met deze eigenschap wordt de status van de asynchrone bewerking door de bewerking-ID.

### <a name="parameters"></a>Parameters
| Naam | Zich in | Beschrijving | Vereist | Schema
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | Pad | Azure-abonnement-ID. | Ja | Tekenreeks |
| resourceGroupName | Pad | Naam van de resourcegroep waarin de Model-Management-account zich bevindt. | Ja | Tekenreeks |
| Accountnaam | Pad | De naam van het Model-Management-account. | Ja | Tekenreeks |
| id | Pad | Bewerking-ID. | Ja | Tekenreeks |
| API-versie | query | Versie van de resourceprovider Microsoft.Machine.Learning API te gebruiken. | Ja | Tekenreeks |
| Autorisatie | koptekst | Verificatietoken. Deze moet ongeveer "Bearer XXXXXX." | Ja | Tekenreeks |

### <a name="responses"></a>Antwoorden
| Code | Beschrijving | Schema |
|--------------------|--------------------|--------------------|
| 200 | Geslaagd. | [OperationStatus](#asyncoperationstatus) |
| Standaard | Foutbericht waarin wordt beschreven waarom de bewerking is mislukt. | [ErrorResponse](#errorresponse)



<a name="definitions"></a>
## <a name="definitions"></a>Definities

<a name="asset"></a>
### <a name="asset"></a>Asset
Het object van het asset die tijdens het maken van de afbeelding Docker nodig.


|Naam|Beschrijving|Schema|
|---|---|---|
|**ID**  <br>*optioneel*|Asset-ID.|Tekenreeks|
|**mimeType**  <br>*optioneel*|MIME-type van de modelinhoud. Zie voor meer informatie over MIME-type, de [lijst met mediatypen IANA](https://www.iana.org/assignments/media-types/media-types.xhtml).|Tekenreeks|
|**uitpakken**  <br>*optioneel*|Hiermee wordt aangegeven waar moeten we het uitpakken van de inhoud tijdens het maken van de Docker-installatiekopie.|Booleaanse waarde|
|**URL**  <br>*optioneel*|Asset locatie-URL.|Tekenreeks|


<a name="asyncoperationstate"></a>
### <a name="asyncoperationstate"></a>AsyncOperationState
De status van de asynchrone bewerking.

*Type*: enum (NotStarted, die wordt uitgevoerd, geannuleerd, geslaagd, mislukt)


<a name="asyncoperationstatus"></a>
### <a name="asyncoperationstatus"></a>AsyncOperationStatus
De status van de bewerking.


|Naam|Beschrijving|Schema|
|---|---|---|
|**createdTime**  <br>*optioneel*  <br>*alleen-lezen*|Asynchrone bewerking maken tijd (UTC).|tekenreeks (datum-tijd)|
|**Eindtijd**  <br>*optioneel*  <br>*alleen-lezen*|Eindtijd van asynchrone bewerking (UTC).|tekenreeks (datum-tijd)|
|**Fout**  <br>*optioneel*||[ErrorResponse](#errorresponse)|
|**ID**  <br>*optioneel*|Asynchrone bewerking-ID.|Tekenreeks|
|**operationType**  <br>*optioneel*|Type van asynchrone bewerking.|Enum (Image, Service)|
|**resourceLocation**  <br>*optioneel*|Resource gemaakt of bijgewerkt door de asynchrone bewerking.|Tekenreeks|
|**status**  <br>*optioneel*||[AsyncOperationState](#asyncoperationstate)|


<a name="authkeys"></a>
### <a name="authkeys"></a>AuthKeys
De verificatiesleutels voor een service.


|Naam|Beschrijving|Schema|
|---|---|---|
|**primaryKey**  <br>*optioneel*|Primaire sleutel.|Tekenreeks|
|**secundaire sleutel**  <br>*optioneel*|Secundaire sleutel.|Tekenreeks|


<a name="autoscaler"></a>
### <a name="autoscaler"></a>AutoScaler
Instellingen voor de autoscaler.


|Naam|Beschrijving|Schema|
|---|---|---|
|**autoscaleEnabled**  <br>*optioneel*|In- of uitschakelen van de autoscaler.|Booleaanse waarde|
|**maxReplicas**  <br>*optioneel*|Maximum aantal replica's schil maximaal schalen.  <br>**Minimumwaarde**:`1`|geheel getal|
|**minReplicas**  <br>*optioneel*|Minimum aantal replica's schil omlaag te schalen.  <br>**Minimumwaarde**:`0`|geheel getal|
|**refreshPeriodInSeconds**  <br>*optioneel*|Voor automatisch schalen trigger te vernieuwen.  <br>**Minimumwaarde**:`1`|geheel getal|
|**targetUtilization**  <br>*optioneel*|Gebruikspercentage waarmee automatisch schalen wordt geactiveerd.  <br>**Minimumwaarde**:`0`  <br>**Maximumwaarde**:`100`|geheel getal|


<a name="computeresource"></a>
### <a name="computeresource"></a>ComputeResource
De Machine Learning-rekenbron.


|Naam|Beschrijving|Schema|
|---|---|---|
|**ID**  <br>*optioneel*|Bron-ID.|Tekenreeks|
|**type**  <br>*optioneel*|Type resource.|Enum (Cluster)|


<a name="containerresourcereservation"></a>
### <a name="containerresourcereservation"></a>ContainerResourceReservation
De configuratie te reserveren van resources voor een container in het cluster.


|Naam|Beschrijving|Schema|
|---|---|---|
|**CPU**  <br>*optioneel*|Hiermee geeft u CPU-reservering. Indeling voor Kubernetes: Zie [betekenis van CPU](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-cpu).|Tekenreeks|
|**geheugen**  <br>*optioneel*|Hiermee geeft u de reservering geheugen. Indeling voor Kubernetes: Zie [betekenis van het geheugen](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-memory).|Tekenreeks|


<a name="deployment"></a>
### <a name="deployment"></a>Implementatie
Een exemplaar van de implementatie van een Azure Machine Learning.


|Naam|Beschrijving|Schema|
|---|---|---|
|**createdAt**  <br>*optioneel*  <br>*alleen-lezen*|Aanmaaktijd van implementatie (UTC).|tekenreeks (datum-tijd)|
|**expiredAt**  <br>*optioneel*  <br>*alleen-lezen*|Implementatie verstreken tijd (UTC).|tekenreeks (datum-tijd)|
|**ID**  <br>*optioneel*|Implementatie-ID.|Tekenreeks|
|**imageId**  <br>*optioneel*|Afbeeldings-ID die is gekoppeld aan deze implementatie.|Tekenreeks|
|**Servicenaam**  <br>*optioneel*|De naam van de service.|Tekenreeks|
|**status**  <br>*optioneel*|Huidige Implementatiestatus.|Tekenreeks|


<a name="deploymentlist"></a>
### <a name="deploymentlist"></a>DeploymentList
Een matrix van implementatie-objecten.

*Type*: <[implementatie](#deployment)> matrix


<a name="errordetail"></a>
### <a name="errordetail"></a>ErrorDetail
Model Management-service foutgegevens.


|Naam|Beschrijving|Schema|
|---|---|---|
|**code**  <br>*Vereist*|Foutcode.|Tekenreeks|
|**Bericht**  <br>*Vereist*|Foutbericht.|Tekenreeks|


<a name="errorresponse"></a>
### <a name="errorresponse"></a>ErrorResponse
Een object van de fout in de Model-Management-service.


|Naam|Beschrijving|Schema|
|---|---|---|
|**code**  <br>*Vereist*|Foutcode.|Tekenreeks|
|**meer informatie**  <br>*optioneel*|Matrix van error detail-objecten.|<[ErrorDetail](#errordetail)> matrix|
|**Bericht**  <br>*Vereist*|Foutbericht.|Tekenreeks|
|**statusCode**  <br>*optioneel*|HTTP-statuscode.|geheel getal|


<a name="image"></a>
### <a name="image"></a>Installatiekopie
De installatiekopie van het Azure Machine Learning.


|Naam|Beschrijving|Schema|
|---|---|---|
|**computeResourceId**  <br>*optioneel*|ID van de omgeving gemaakt in de Machine Learning-rekenbron.|Tekenreeks|
|**createdTime**  <br>*optioneel*|De installatiekopie van het maken van tijd (UTC).|tekenreeks (datum-tijd)|
|**creationState**  <br>*optioneel*||[AsyncOperationState](#asyncoperationstate)|
|**Beschrijving**  <br>*optioneel*|Beschrijving van de installatiekopie.|Tekenreeks|
|**Fout**  <br>*optioneel*||[ErrorResponse](#errorresponse)|
|**ID**  <br>*optioneel*|Installatiekopie-ID.|Tekenreeks|
|**imageBuildLogUri**  <br>*optioneel*|De URI van de geüploade logboeken van de image build.|Tekenreeks|
|**imageLocation**  <br>*optioneel*|Azure Container register locatietekenreeks voor de installatiekopie gemaakt.|Tekenreeks|
|**imageType**  <br>*optioneel*||[ImageType](#imagetype)|
|**manifest**  <br>*optioneel*||[Manifest](#manifest)|
|**naam**  <br>*optioneel*|De naam van de installatiekopie.|Tekenreeks|
|**Versie**  <br>*optioneel*|Installatiekopie-versie is ingesteld door de Model-Management-service.|geheel getal|


<a name="imagerequest"></a>
### <a name="imagerequest"></a>imageRequest
Een aanvraag voor het maken van een installatiekopie van een Azure Machine Learning.


|Naam|Beschrijving|Schema|
|---|---|---|
|**computeResourceId**  <br>*Vereist*|ID van de omgeving gemaakt in de Machine Learning-rekenbron.|Tekenreeks|
|**Beschrijving**  <br>*optioneel*|Beschrijving van de installatiekopie.|Tekenreeks|
|**imageType**  <br>*Vereist*||[ImageType](#imagetype)|
|**manifest-id**  <br>*Vereist*|ID van het manifest van waaruit u de installatiekopie wordt gemaakt.|Tekenreeks|
|**naam**  <br>*Vereist*|De naam van de installatiekopie.|Tekenreeks|


<a name="imagetype"></a>
### <a name="imagetype"></a>ImageType
Geeft het type van de afbeelding.

*Type*: enum (Docker)


<a name="manifest"></a>
### <a name="manifest"></a>Manifest
Het Azure Machine Learning-manifest.


|Naam|Beschrijving|Schema|
|---|---|---|
|**activa**  <br>*Vereist*|Lijst met activa.|<[Asset](#asset)> matrix|
|**createdTime**  <br>*optioneel*  <br>*alleen-lezen*|Maken van het manifest tijd (UTC).|tekenreeks (datum-tijd)|
|**Beschrijving**  <br>*optioneel*|Beschrijvende tekst manifest.|Tekenreeks|
|**driverProgram**  <br>*Vereist*|Stuurprogramma-programma van het manifest.|Tekenreeks|
|**ID**  <br>*optioneel*|Manifest-ID.|Tekenreeks|
|**modelIds**  <br>*optioneel*|Lijst met model-id's van de geregistreerde modellen. De aanvraag mislukt als een van de opgenomen modellen niet zijn geregistreerd.|<string>matrix|
|**modelType**  <br>*optioneel*|Hiermee geeft u op dat de modellen al zijn geregistreerd bij de Model-Management-service.|Enum (geregistreerd)|
|**naam**  <br>*Vereist*|Naam van het manifest.|Tekenreeks|
|**targetRuntime**  <br>*Vereist*||[TargetRuntime](#targetruntime)|
|**Versie**  <br>*optioneel*  <br>*alleen-lezen*|Versie van het clustermanifest toegewezen door de Model-Management-service.|geheel getal|
|**webserviceType**  <br>*optioneel*|Hiermee geeft u het gewenste type van de webservice die wordt gemaakt van het manifest.|Enum (in real-time)|


<a name="model"></a>
### <a name="model"></a>Model
Een exemplaar van een Azure Machine Learning-model.


|Naam|Beschrijving|Schema|
|---|---|---|
|**createdAt**  <br>*optioneel*  <br>*alleen-lezen*|Aanmaaktijd model (UTC).|tekenreeks (datum-tijd)|
|**Beschrijving**  <br>*optioneel*|Beschrijving van het model.|Tekenreeks|
|**ID**  <br>*optioneel*  <br>*alleen-lezen*|Model-ID.|Tekenreeks|
|**mimeType**  <br>*Vereist*|MIME-type van de modelinhoud. Zie voor meer informatie over MIME-type, de [lijst met mediatypen IANA](https://www.iana.org/assignments/media-types/media-types.xhtml).|Tekenreeks|
|**naam**  <br>*Vereist*|Naam van het model.|Tekenreeks|
|**tags**  <br>*optioneel*|Lijst van de tag model.|<string>matrix|
|**uitpakken**  <br>*optioneel*|Hiermee wordt aangegeven of we wilt uitpakken van het model tijdens het maken van de Docker-installatiekopie.|Booleaanse waarde|
|**URL**  <br>*Vereist*|URL van het model. Meestal wordt een shared access signature-URL hier plaatst.|Tekenreeks|
|**Versie**  <br>*optioneel*  <br>*alleen-lezen*|Versie van het model die door de Model-Management-service is toegewezen.|geheel getal|


<a name="modeldatacollection"></a>
### <a name="modeldatacollection"></a>ModelDataCollection
Gegevens verzamelen informatie over het model.


|Naam|Beschrijving|Schema|
|---|---|---|
|**eventHubEnabled**  <br>*optioneel*|Schakel een event hub voor een service.|Booleaanse waarde|
|**storageEnabled**  <br>*optioneel*|Schakel de opslag voor een service.|Booleaanse waarde|


<a name="paginatedimagelist"></a>
### <a name="paginatedimagelist"></a>PaginatedImageList
Een gepagineerde lijst van afbeeldingen.


|Naam|Beschrijving|Schema|
|---|---|---|
|**nextLink**  <br>*optioneel*|Voortzetting-koppeling (absolute URI) naar de volgende pagina van de resultaten in de lijst.|Tekenreeks|
|**waarde**  <br>*optioneel*|Matrix van modelobjecten.|<[Afbeelding](#image)> matrix|


<a name="paginatedmanifestlist"></a>
### <a name="paginatedmanifestlist"></a>PaginatedManifestList
Een gepagineerde lijst van de manifesten.


|Naam|Beschrijving|Schema|
|---|---|---|
|**nextLink**  <br>*optioneel*|Voortzetting-koppeling (absolute URI) naar de volgende pagina van de resultaten in de lijst.|Tekenreeks|
|**waarde**  <br>*optioneel*|Matrix van de manifest-objecten.|<[Manifest](#manifest)> matrix|


<a name="paginatedmodellist"></a>
### <a name="paginatedmodellist"></a>PaginatedModelList
Een gepagineerde lijst met modellen.


|Naam|Beschrijving|Schema|
|---|---|---|
|**nextLink**  <br>*optioneel*|Voortzetting-koppeling (absolute URI) naar de volgende pagina van de resultaten in de lijst.|Tekenreeks|
|**waarde**  <br>*optioneel*|Matrix van modelobjecten.|<[Model](#model)> matrix|


<a name="paginatedservicelist"></a>
### <a name="paginatedservicelist"></a>PaginatedServiceList
Een gepagineerde lijst met services.


|Naam|Beschrijving|Schema|
|---|---|---|
|**nextLink**  <br>*optioneel*|Voortzetting-koppeling (absolute URI) naar de volgende pagina van de resultaten in de lijst.|Tekenreeks|
|**waarde**  <br>*optioneel*|Matrix van service-objecten.|<[ServiceResponse](#serviceresponse)> matrix|


<a name="servicecreaterequest"></a>
### <a name="servicecreaterequest"></a>ServiceCreateRequest
Een aanvraag voor het maken van een service.


|Naam|Beschrijving|Schema|
|---|---|---|
|**appInsightsEnabled**  <br>*optioneel*|Application insights voor een service inschakelen.|Booleaanse waarde|
|**autoScaler**  <br>*optioneel*||[AutoScaler](#autoscaler)|
|**computeResource**  <br>*Vereist*||[ComputeResource](#computeresource)|
|**containerResourceReservation**  <br>*optioneel*||[ContainerResourceReservation](#containerresourcereservation)|
|**dataCollection**  <br>*optioneel*||[ModelDataCollection](#modeldatacollection)|
|**imageId**  <br>*Vereist*|Installatiekopie van de service te maken.|Tekenreeks|
|**maxConcurrentRequestsPerContainer**  <br>*optioneel*|Maximum aantal gelijktijdige aanvragen.  <br>**Minimumwaarde**:`1`|geheel getal|
|**naam**  <br>*Vereist*|De naam van de service.|Tekenreeks|
|**numReplicas**  <br>*optioneel*|Het aantal schil replica's op elk gewenst moment worden uitgevoerd. Kan niet worden opgegeven als Autoscaler is ingeschakeld.  <br>**Minimumwaarde**:`0`|geheel getal|


<a name="serviceregeneratekeyrequest"></a>
### <a name="serviceregeneratekeyrequest"></a>ServiceRegenerateKeyRequest
Een aanvraag voor opnieuw genereren van een sleutel voor een service.


|Naam|Beschrijving|Schema|
|---|---|---|
|**keyType**  <br>*optioneel*|Bepaalt welke sleutel opnieuw genereren.|Enum (primair, secundair)|


<a name="serviceresponse"></a>
### <a name="serviceresponse"></a>ServiceResponse
De gedetailleerde status van de service.


|Naam|Beschrijving|Schema|
|---|---|---|
|**createdAt**  <br>*optioneel*|Het maken van service-tijd (UTC).|tekenreeks (datum-tijd)|
|**ID**  <br>*optioneel*|Service-ID.|Tekenreeks|
|**afbeelding**  <br>*optioneel*||[Afbeelding](#image)|
|**manifest**  <br>*optioneel*||[Manifest](#manifest)|
|**modellen**  <br>*optioneel*|Lijst met modellen.|<[Model](#model)> matrix|
|**naam**  <br>*optioneel*|De naam van de service.|Tekenreeks|
|**scoringUri**  <br>*optioneel*|De URI voor score berekenen voor de service.|Tekenreeks|
|**status**  <br>*optioneel*||[AsyncOperationState](#asyncoperationstate)|
|**updatedAt**  <br>*optioneel*|Laatste bijwerking van de tijd (UTC).|tekenreeks (datum-tijd)|
|**appInsightsEnabled**  <br>*optioneel*|Application insights voor een service inschakelen.|Booleaanse waarde|
|**autoScaler**  <br>*optioneel*||[AutoScaler](#autoscaler)|
|**computeResource**  <br>*Vereist*||[ComputeResource](#computeresource)|
|**containerResourceReservation**  <br>*optioneel*||[ContainerResourceReservation](#containerresourcereservation)|
|**dataCollection**  <br>*optioneel*||[ModelDataCollection](#modeldatacollection)|
|**maxConcurrentRequestsPerContainer**  <br>*optioneel*|Maximum aantal gelijktijdige aanvragen.  <br>**Minimumwaarde**:`1`|geheel getal|
|**numReplicas**  <br>*optioneel*|Het aantal schil replica's op elk gewenst moment worden uitgevoerd. Kan niet worden opgegeven als Autoscaler is ingeschakeld.  <br>**Minimumwaarde**:`0`|geheel getal|
|**Fout**  <br>*optioneel*||[ErrorResponse](#errorresponse)|


<a name="serviceupdaterequest"></a>
### <a name="serviceupdaterequest"></a>serviceUpdateRequest
Een aanvraag voor het bijwerken van een service.


|Naam|Beschrijving|Schema|
|---|---|---|
|**appInsightsEnabled**  <br>*optioneel*|Application insights voor een service inschakelen.|Booleaanse waarde|
|**autoScaler**  <br>*optioneel*||[AutoScaler](#autoscaler)|
|**containerResourceReservation**  <br>*optioneel*||[ContainerResourceReservation](#containerresourcereservation)|
|**dataCollection**  <br>*optioneel*||[ModelDataCollection](#modeldatacollection)|
|**imageId**  <br>*optioneel*|Installatiekopie van de service te maken.|Tekenreeks|
|**maxConcurrentRequestsPerContainer**  <br>*optioneel*|Maximum aantal gelijktijdige aanvragen.  <br>**Minimumwaarde**:`1`|geheel getal|
|**numReplicas**  <br>*optioneel*|Het aantal schil replica's op elk gewenst moment worden uitgevoerd. Kan niet worden opgegeven als Autoscaler is ingeschakeld.  <br>**Minimumwaarde**:`0`|geheel getal|


<a name="targetruntime"></a>
### <a name="targetruntime"></a>TargetRuntime
Het type van de doelruntime.


|Naam|Beschrijving|Schema|
|---|---|---|
|**Eigenschappen**  <br>*Vereist*||< tekenreeks, tekenreeks > kaart|
|**runtimeType**  <br>*Vereist*|Hiermee geeft u de runtime.|Enum (SparkPython, Python)|

