---
title: Een Node.js en MongoDB web-app in Azure bouwen | Microsoft Docs
description: Informatie over het ophalen van een Node.js-app in Azure, werken met verbinding met een Cosmos-DB-database met een verbindingsreeks voor MongoDB.
services: app-service\web
documentationcenter: nodejs
author: cephalin
manager: erikre
editor: 
ms.assetid: 0b4d7d0e-e984-49a1-a57a-3c0caa955f0e
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 05/04/2017
ms.author: cephalin
ms.custom: mvc
ms.openlocfilehash: 9fc11352a031ac1c1abcc6c6bd173bd9b0e8a222
ms.sourcegitcommit: 38c9176c0c967dd641d3a87d1f9ae53636cf8260
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 11/06/2017
---
# <a name="build-a-nodejs-and-mongodb-web-app-in-azure"></a>Een Node.js en MongoDB web-app in Azure bouwen

Azure Web Apps biedt een zeer schaalbaar, zelf patch webhosting-service. Deze zelfstudie laat zien hoe een Node.js-web-app in Azure maken en te verbinden met een MongoDB-database. Wanneer u bent klaar, hebt u een gemiddelde-toepassing (MongoDB, snelle AngularJS en Node.js) wordt uitgevoerd in [Azure App Service](app-service-web-overview.md). Voor de eenvoud, de voorbeeldtoepassing gebruikt de [MEAN.js webframework](http://meanjs.org/).

![MEAN.js-app uitgevoerd in Azure App Service](./media/app-service-web-tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

Wat u leert het volgende:

> [!div class="checklist"]
> * Maak een MongoDB-database in Azure
> * Een Node.js-app verbinden met MongoDB
> * De app implementeren in Azure
> * Bijwerken van het gegevensmodel en de app implementeren
> * Diagnostische logboeken van de stroom van Azure
> * De app in de Azure portal beheren

## <a name="prerequisites"></a>Vereisten

Vereisten voor het voltooien van deze zelfstudie:

1. [Git installeren](https://git-scm.com/)
1. [Node.js en NPM installeren](https://nodejs.org/)
1. [Installeer Gulp.js](http://gulpjs.com/) (vereist door de [MEAN.js](http://meanjs.org/docs/0.5.x/#getting-started))
1. [Installeren en uitvoeren van MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/) 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="test-local-mongodb"></a>Test lokale MongoDB

Open de terminalvenster en `cd` naar de `bin` map van de MongoDB-installatie. Alle opdrachten kunt uitvoeren in deze zelfstudie kunt u deze terminalvenster.

Voer `mongo` in de terminal verbinding maken met uw lokale MongoDB-server.

```bash
mongo
```

Als de verbinding geslaagd is, wordt klikt u vervolgens de MongoDB-database al uitgevoerd. Als dit niet het geval is, zorg ervoor dat de lokale MongoDB-database is gestart door de stappen op [Installeer MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/). Vaak MongoDB is geïnstalleerd, maar u moet nog steeds start met `mongod`. 

Wanneer u klaar bent test de MongoDB-database, typ `Ctrl+C` in de terminal. 

## <a name="create-local-nodejs-app"></a>Lokale Node.js-app maken

In deze stap moet u het lokale Node.js-project instellen.

### <a name="clone-the-sample-application"></a>De voorbeeldtoepassing klonen

In het terminalvenster `cd` in een werkmap.  

Voer de volgende opdracht uit om de voorbeeldopslagplaats te klonen. 

```bash
git clone https://github.com/Azure-Samples/meanjs.git
```

Deze repository voorbeeld bevat een kopie van de [MEAN.js opslagplaats](https://github.com/meanjs/mean). Het is gewijzigd om te worden uitgevoerd op App Service (voor meer informatie raadpleegt u de opslagplaats MEAN.js [Leesmij-bestand](https://github.com/Azure-Samples/meanjs/blob/master/README.md)).

### <a name="run-the-application"></a>De toepassing uitvoeren

Voer de volgende opdrachten voor het installeren van de vereiste pakketten en de toepassing niet starten.

```bash
cd meanjs
npm install
npm start
```

Wanneer de app volledig wordt geladen is, ziet u iets soortgelijks als in het volgende bericht:

```console
--
MEAN.JS - Development Environment

Environment:     development
Server:          http://0.0.0.0:3000
Database:        mongodb://localhost/mean-dev
App version:     0.5.0
MEAN.JS version: 0.5.0
--
```

Ga naar `http://localhost:3000` in een browser. Klik op **aanmelden** in het bovenste menu en een testgebruiker maken. 

De MEAN.js-voorbeeldtoepassing slaat gebruikersgegevens op in de database. Als u geslaagde in het maken van een gebruiker en aanmelden, kunnen uw app is gegevens schrijven naar de lokale MongoDB-database.

![MEAN.js maakt een geslaagde verbinding met MongoDB](./media/app-service-web-tutorial-nodejs-mongodb-app/mongodb-connect-success.png)

Selecteer **Admin > artikelen beheren** sommige artikelen toevoegen.

Als u wilt Node.js op elk gewenst moment stoppen, drukt u op `Ctrl+C` in de terminal. 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-production-mongodb"></a>Productie MongoDB maken

In deze stap maakt u een MongoDB-database in Azure. Wanneer uw app wordt geïmplementeerd naar Azure, wordt deze cloud-database gebruikt.

Voor MongoDB, het gebruik van deze zelfstudie [Azure Cosmos DB](/azure/documentdb/). MongoDB-clientverbindingen biedt ondersteuning voor cosmos DB.

### <a name="create-a-resource-group"></a>Een resourcegroep maken

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)] 

### <a name="create-a-cosmos-db-account"></a>Een Cosmos-DB-account maken

In de Cloud-Shell, maakt u een Cosmos-DB-account met de [az cosmosdb maken](/cli/azure/cosmosdb#create) opdracht.

In de volgende opdracht te vervangen door een unieke naam van de Cosmos-database voor de  *\<cosmosdb_name >* tijdelijke aanduiding. Deze naam wordt gebruikt als het onderdeel van het eindpunt Cosmos DB `https://<cosmosdb_name>.documents.azure.com/`, zodat de naam moet uniek zijn in alle Cosmos-DB-accounts in Azure. De naam mag alleen kleine letters, cijfers en het koppelteken (-) en moet tussen 3 en 50 tekens bevatten.

```azurecli-interactive
az cosmosdb create --name <cosmosdb_name> --resource-group myResourceGroup --kind MongoDB
```

De *--kind MongoDB* parameter zorgt ervoor dat MongoDB-clientverbindingen.

Wanneer de Cosmos-DB-account is gemaakt, toont de Azure CLI informatie vergelijkbaar met het volgende voorbeeld:

```json
{
  "consistencyPolicy":
  {
    "defaultConsistencyLevel": "Session",
    "maxIntervalInSeconds": 5,
    "maxStalenessPrefix": 100
  },
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://<cosmosdb_name>.documents.azure.com:443/",
  "failoverPolicies": 
  ...
  < Output truncated for readability >
}
```

## <a name="connect-app-to-production-mongodb"></a>App verbinden met productie MongoDB

In deze stap maakt u verbinding maken uw voorbeeldtoepassing MEAN.js naar de Cosmos-DB-database die u zojuist hebt gemaakt, met een verbindingsreeks voor MongoDB. 

### <a name="retrieve-the-database-key"></a>De databasesleutel ophalen

Voor verbinding met de database van de Cosmos-database, moet u de databasesleutel. In de Cloud-Shell gebruiken de [az cosmosdb lijst-sleutels](/cli/azure/cosmosdb#list-keys) opdracht voor het ophalen van de primaire sleutel.

```azurecli-interactive
az cosmosdb list-keys --name <cosmosdb_name> --resource-group myResourceGroup
```

De Azure CLI toont informatie vergelijkbaar met het volgende voorbeeld:

```json
{
  "primaryMasterKey": "RS4CmUwzGRASJPMoc0kiEvdnKmxyRILC9BWisAYh3Hq4zBYKr0XQiSE4pqx3UchBeO4QRCzUt1i7w0rOkitoJw==",
  "primaryReadonlyMasterKey": "HvitsjIYz8TwRmIuPEUAALRwqgKOzJUjW22wPL2U8zoMVhGvregBkBk9LdMTxqBgDETSq7obbwZtdeFY7hElTg==",
  "secondaryMasterKey": "Lu9aeZTiXU4PjuuyGBbvS1N9IRG3oegIrIh95U6VOstf9bJiiIpw3IfwSUgQWSEYM3VeEyrhHJ4rn3Ci0vuFqA==",
  "secondaryReadonlyMasterKey": "LpsCicpVZqHRy7qbMgrzbRKjbYCwCKPQRl0QpgReAOxMcggTvxJFA94fTi0oQ7xtxpftTJcXkjTirQ0pT7QFrQ=="
}
```

Kopieer de waarde van `primaryMasterKey`. U hebt deze informatie nodig voor de volgende stap.

<a name="devconfig"></a>
### <a name="configure-the-connection-string-in-your-nodejs-application"></a>Configureer de verbindingsreeks in uw Node.js-toepassing

In de opslagplaats van uw lokale MEAN.js, in de _config/env/_ map, maakt u een bestand met de naam _lokale production.js_. Standaard _.gitignore_ is geconfigureerd voor dit bestand uit de opslagplaats houden. 

Kopieer de volgende code in de App. Zorg ervoor dat u de twee  *\<cosmosdb_name >* tijdelijke aanduidingen door uw Cosmos-DB databasenaam en vervang de  *\<primary_master_key >* aanduiding voor items met de sleutel u in de vorige stap hebt gekopieerd.

```javascript
module.exports = {
  db: {
    uri: 'mongodb://<cosmosdb_name>:<primary_master_key>@<cosmosdb_name>.documents.azure.com:10250/mean?ssl=true&sslverifycertificate=false'
  }
};
```

De `ssl=true` optie is vereist omdat [Cosmos DB SSL vereist](../cosmos-db/connect-mongodb-account.md#connection-string-requirements). 

Sla uw wijzigingen op.

### <a name="test-the-application-in-production-mode"></a>Test de toepassing in productiemodus 

Voer de volgende opdracht te minify en scripts voor de productie-omgeving te bundelen. Dit proces wordt gegenereerd voor de bestanden die nodig is voor de productie-omgeving.

```bash
gulp prod
```

Voer de volgende opdracht te gebruiken van de verbindingsreeks die u hebt geconfigureerd in _config/env/local-production.js_.

```bash
# Bash
NODE_ENV=production node server.js

# Windows PowerShell
$env:NODE_ENV = "production" 
node server.js
```

`NODE_ENV=production`Hiermee stelt u de variabele voor de omgeving waarin wordt uitgelegd Node.js om uit te voeren in de productieomgeving.  `node server.js`Hiermee start u de Node.js-server met `server.js` in de hoofdmap van uw opslagplaats. Dit is hoe uw Node.js-toepassing wordt geladen in Azure. 

Wanneer de app wordt geladen, moet u controleren om ervoor te zorgen dat deze wordt uitgevoerd in de productieomgeving:

```console
--
MEAN.JS

Environment:     production
Server:          http://0.0.0.0:8443
Database:        mongodb://<cosmosdb_name>:<primary_master_key>@<cosmosdb_name>.documents.azure.com:10250/mean?ssl=true&sslverifycertificate=false
App version:     0.5.0
MEAN.JS version: 0.5.0
```

Ga naar `http://localhost:8443` in een browser. Klik op **aanmelden** in het bovenste menu en een testgebruiker maken. Als u geslaagde maken van een gebruiker en aanmelden, kunnen uw app is gegevens schrijven naar de database van de Cosmos-database in Azure. 

In de terminal in stoppen Node.js `Ctrl+C`. 

## <a name="deploy-app-to-azure"></a>App implementeren in Azure

In deze stap maakt implementeren u uw MongoDB verbonden Node.js-toepassing in Azure App Service.

### <a name="configure-a-deployment-user"></a>Een implementatiegebruiker configureren

[!INCLUDE [Configure deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-an-app-service-plan"></a>Een App Service-plan maken

[!INCLUDE [Create app service plan no h](../../includes/app-service-web-create-app-service-plan-no-h.md)]

### <a name="create-a-web-app"></a>Een webtoepassing maken

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-nodejs-no-h.md)] 

### <a name="configure-an-environment-variable"></a>Configureren van een omgevingsvariabele

Standaard het project MEAN.js houdt _config/env/local-production.js_ buiten de Git-opslagplaats. Dus voor uw Azure-web-app gebruikt u app-instellingen voor het definiëren van de MongoDB-verbindingsreeks.

U kunt app-instellingen instellen met de [az webapp config appsettings bijwerken](/cli/azure/webapp/config/appsettings#update) opdracht in de Cloud-Shell. 

Het volgende voorbeeld wordt een `MONGODB_URI` app-instelling in uw Azure-web-app. Vervang de  *\<app_naam >*,  *\<cosmosdb_name >*, en  *\<primary_master_key >* tijdelijke aanduidingen.

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group myResourceGroup --settings MONGODB_URI="mongodb://<cosmosdb_name>:<primary_master_key>@<cosmosdb_name>.documents.azure.com:10250/mean?ssl=true"
```

In een Node.js-code, opent u deze app-instelling met `process.env.MONGODB_URI`, net zoals u toegang krijgen een omgevingsvariabele tot zou. 

Open in uw lokale MEAN.js-opslagplaats _config/env/production.js_ (geen _config/env/local-production.js_), die een specifieke configuratie van productie-omgeving heeft. De standaard MEAN.js app is al geconfigureerd voor het gebruik van de `MONGODB_URI` omgevingsvariabele die u hebt gemaakt.

```javascript
db: {
  uri: ... || process.env.MONGODB_URI || ...,
  ...
},
```

### <a name="push-to-azure-from-git"></a>Pushen naar Azure vanaf Git

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

```bash
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 489 bytes | 0 bytes/s, done.
Total 5 (delta 3), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '6c7c716eee'.
remote: Running custom deployment command...
remote: Running deployment command...
remote: Handling node.js deployment.
.
.
.
remote: Deployment successful.
To https://<app_name>.scm.azurewebsites.net/<app_name>.git
 * [new branch]      master -> master
``` 

U merkt dat het implementatieproces wordt uitgevoerd [Gulp](http://gulpjs.com/) nadat `npm install`. App Service wordt niet Gulp of knorvis taken uitgevoerd tijdens de implementatie, zodat deze opslagplaats voorbeeld heeft twee extra bestanden in de hoofdmap in te schakelen: 

- _.Deployment_ -dit bestand vertelt App Service om uit te voeren `bash deploy.sh` als het script voor aangepaste implementatie.
- _Deploy.sh_ -het script voor aangepaste implementatie. Als u het bestand bekijkt, ziet u dat deze wordt uitgevoerd `gulp prod` nadat `npm install` en `bower install`. 

Deze aanpak kunt u een stap toevoegen aan uw implementatie op basis van Git. Als u uw Azure-web-app op elk moment opnieuw opstart, wordt niet deze automatiseringstaken opnieuw uitvoeren App Service.

### <a name="browse-to-the-azure-web-app"></a>Blader naar de Azure-web-app 

Blader naar de geïmplementeerde web-app met behulp van uw webbrowser. 

```bash 
http://<app_name>.azurewebsites.net 
``` 

Klik op **aanmelden** in het bovenste menu en maak een dummy-gebruiker. 

Als u met succes voltooid zijn en de app automatisch zich bij de gebruiker aanmeldt, heeft uw MEAN.js-app in Azure verbinding met de database MongoDB (Cosmos DB). 

![MEAN.js-app uitgevoerd in Azure App Service](./media/app-service-web-tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

Selecteer **Admin > artikelen beheren** sommige artikelen toevoegen. 

**Gefeliciteerd!** U uitvoert een gegevensgestuurd Node.js-app in Azure App Service.

## <a name="update-data-model-and-redeploy"></a>Update-gegevensmodel en de implementatie opnieuw uit

In deze stap maakt u wijzigt de `article` data model en de wijziging van uw publiceren naar Azure.

### <a name="update-the-data-model"></a>Bijwerken van het gegevensmodel

Open _modules/articles/server/models/article.server.model.js_.

In `ArticleSchema`, Voeg een `String` type aangeroepen `comment`. Wanneer u bent klaar, ziet uw schema-code als volgt:

```javascript
var ArticleSchema = new Schema({
  ...,
  user: {
    type: Schema.ObjectId,
    ref: 'User'
  },
  comment: {
    type: String,
    default: '',
    trim: true
  }
});
```

### <a name="update-the-articles-code"></a>De code artikelen bijwerken

Bijwerken van de rest van uw `articles` code voor het gebruik `comment`.

Er zijn vijf bestanden die u wilt wijzigen: de server-controller en de weergaven vier client. 

Open _modules/articles/server/controllers/articles.server.controller.js_.

In de `update` werkt, het toevoegen van een toewijzing voor `article.comment`. De volgende code toont de voltooide `update` functie:

```javascript
exports.update = function (req, res) {
  var article = req.article;

  article.title = req.body.title;
  article.content = req.body.content;
  article.comment = req.body.comment;

  ...
};
```

Open _modules/articles/client/views/view-article.client.view.html_.

Net boven de afsluitende `</section>` labelen, voeg de volgende regel om weer te geven `comment` samen met de rest van de artikelgegevens:

```HTML
<p class="lead" ng-bind="vm.article.comment"></p>
```

Open _modules/articles/client/views/list-articles.client.view.html_.

Net boven de afsluitende `</a>` labelen, voeg de volgende regel om weer te geven `comment` samen met de rest van de artikelgegevens:

```HTML
<p class="list-group-item-text" ng-bind="article.comment"></p>
```

Open _modules/articles/client/views/admin/list-articles.client.view.html_.

In de `<div class="list-group">` element en net boven de afsluitende `</a>` labelen, voeg de volgende regel om weer te geven `comment` samen met de rest van de artikelgegevens:

```HTML
<p class="list-group-item-text" data-ng-bind="article.comment"></p>
```

Open _modules/articles/client/views/admin/form-article.client.view.html_.

Zoek de `<div class="form-group">` element met de verzendknop, welke ziet er als volgt:

```HTML
<div class="form-group">
  <button type="submit" class="btn btn-default">{{vm.article._id ? 'Update' : 'Create'}}</button>
</div>
```

Net boven deze tag toevoegen `<div class="form-group">` element waarmee mensen bewerken de `comment` veld. Uw nieuwe element moet er als volgt uitzien:

```HTML
<div class="form-group">
  <label class="control-label" for="comment">Comment</label>
  <textarea name="comment" data-ng-model="vm.article.comment" id="comment" class="form-control" cols="30" rows="10" placeholder="Comment"></textarea>
</div>
```

### <a name="test-your-changes-locally"></a>Uw wijzigingen lokaal testen

Sla al uw wijzigingen op.

Test uw wijzigingen in de productiemodus opnieuw in het lokale terminalvenster.

```bash
# Bash
gulp prod
NODE_ENV=production node server.js

# Windows PowerShell
gulp prod
$env:NODE_ENV = "production" 
node server.js
```

Navigeer naar `http://localhost:8443` in een browser en zorg ervoor dat u bent aangemeld.

Selecteer **Admin > artikelen beheren**, klikt u vervolgens een artikel toevoegen door te selecteren de  **+**  knop.

U ziet de nieuwe `Comment` textbox nu.

![Veld toegevoegde opmerking aan artikelen](./media/app-service-web-tutorial-nodejs-mongodb-app/added-comment-field.png)

In de terminal in stoppen Node.js `Ctrl+C`. 

### <a name="publish-changes-to-azure"></a>Wijzigingen publiceren naar Azure

Uw wijzigingen in Git in het lokale terminalvenster en de codewijzigingen pushen naar Azure.

```bash
git commit -am "added article comment"
git push azure master
```

Eenmaal de `git push` is voltooid, gaat u naar uw Azure-web-app en de nieuwe functionaliteit uitproberen.

![Model en de database-wijzigingen die zijn gepubliceerd naar Azure](media/app-service-web-tutorial-nodejs-mongodb-app/added-comment-field-published.png)

Als u alle artikelen die eerder is toegevoegd, kunt u nog steeds zien. Bestaande gegevens in uw Cosmos-database is niet verloren gaan. Ook de updates aan het gegevensschema en uw bestaande gegevens intact blijft.

## <a name="stream-diagnostic-logs"></a>Diagnostische logboeken 

Als uw Node.js-toepassing in Azure App Service wordt uitgevoerd, kunt u de logboeken van de console doorgesluisd naar uw terminal opvragen. Op die manier kunt u de dezelfde diagnostische berichten op te sporen toepassingsfouten ophalen.

Gebruik voor het starten van de streaming-logboek de [az webapp logboek tail](/cli/azure/webapp/log#tail) opdracht in de Cloud-Shell.

```azurecli-interactive
az webapp log tail --name <app_name> --resource-group myResourceGroup
``` 

Uw Azure-web-app in de browser om op te halen van sommige webverkeer eenmaal streaming-logboek is gestart, worden vernieuwd. U ziet nu de logboeken van de console is doorgegeven naar de terminal.

Stop log streaming op elk gewenst moment door typen `Ctrl+C`. 

## <a name="manage-your-azure-web-app"></a>Uw Azure-web-app beheren

Ga naar de [Azure-portal](https://portal.azure.com) om te zien van de web-app die u hebt gemaakt.

Klik vanuit het linkermenu op **App Services** en klik op de naam van uw Azure-web-app.

![Navigatie in de portal naar de Azure-web-app](./media/app-service-web-tutorial-nodejs-mongodb-app/access-portal.png)

Standaard ziet u de portal voor uw web-app **overzicht** pagina. Deze pagina geeft u een overzicht van hoe uw app presteert. Hier kunt u ook algemene beheertaken uitvoeren, zoals bladeren, stoppen, starten, opnieuw opstarten en verwijderen. De tabbladen aan de linkerkant van de pagina's worden weergegeven de andere configuratie dat kunt u openen.

![App Service-pagina in Azure Portal](./media/app-service-web-tutorial-nodejs-mongodb-app/web-app-blade.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>
## <a name="next-steps"></a>Volgende stappen

Wat u hebt geleerd:

> [!div class="checklist"]
> * Maak een MongoDB-database in Azure
> * Een Node.js-app verbinden met MongoDB
> * De app implementeren in Azure
> * Bijwerken van het gegevensmodel en de app implementeren
> * Logboeken van de stroom van Azure naar uw terminal
> * De app in de Azure portal beheren

Ga naar de volgende zelfstudie voor informatie over het toewijzen van een aangepaste DNS-naam aan uw web-app.

> [!div class="nextstepaction"] 
> [Een bestaande aangepaste DNS-naam toewijzen aan Azure Web Apps](app-service-web-tutorial-custom-domain.md)
