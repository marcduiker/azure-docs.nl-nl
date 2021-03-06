---
title: 'Zelfstudie: Azure Active Directory-integratie met Bpm''online | Microsoft Docs'
description: Informatie over het configureren van eenmalige aanmelding tussen Azure Active Directory en Bpm'online.
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 052db91d-ccff-4098-8ae3-2f76eca90539
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/16/2017
ms.author: jeedes
ms.openlocfilehash: 52518cd76738f93162b3c1c8b76abff95f34fc2f
ms.sourcegitcommit: a7c01dbb03870adcb04ca34745ef256414dfc0b3
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/17/2017
---
# <a name="tutorial-azure-active-directory-integration-with-bpmonline"></a>Zelfstudie: Azure Active Directory-integratie met Bpm'online

In deze zelfstudie leert u hoe Bpm'online integreren met Azure Active Directory (Azure AD).

Bpm'online integreren met Azure AD biedt de volgende voordelen:

- U kunt beheren in Azure AD die toegang tot Bpm'online heeft.
- U kunt uw gebruikers automatisch ophalen aangemeld bij Bpm'online (Single Sign-On) met hun Azure AD-accounts kunt inschakelen.
- U kunt uw accounts op één centrale locatie - en de Azure-portal beheren.

Als u weten van meer informatie over de integratie van de SaaS-app met Azure AD wilt, Zie [wat is er toegang tot toepassingen en eenmalige aanmelding bij Azure Active Directory](active-directory-appssoaccess-whatis.md).

## <a name="prerequisites"></a>Vereisten

Voor het configureren van Azure AD-integratie met Bpm'online, moet u de volgende items:

- Een Azure AD-abonnement
- Een Bpm'online eenmalige aanmelding ingeschakeld abonnement

> [!NOTE]
> Test de stappen in deze zelfstudie, raden we niet met behulp van een productieomgeving.

Test de stappen in deze zelfstudie, moet u deze aanbevelingen volgen:

- Gebruik niet uw productieomgeving, tenzij het noodzakelijk is.
- Als u geen een proefabonnement Azure AD-omgeving hebt, kunt u [ophalen van een proefversie van één maand](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Scenariobeschrijving
In deze zelfstudie test u Azure AD eenmalige aanmelding in een testomgeving. Het scenario in deze zelfstudie bestaat uit twee belangrijkste bouwstenen:

1. Bpm'online uit de galerie toevoegen
2. Configureren en testen van Azure AD eenmalige aanmelding

## <a name="adding-bpmonline-from-the-gallery"></a>Bpm'online uit de galerie toevoegen
Voor het configureren van de integratie van Bpm'online in Azure AD, moet u Bpm'online uit de galerie toevoegen aan de lijst met beheerde SaaS-apps.

**Als u wilt toevoegen Bpm'online uit de galerie, moet u de volgende stappen uitvoeren:**

1. In de  **[Azure-portal](https://portal.azure.com)**, klik in het linkernavigatievenster op **Azure Active Directory** pictogram. 

    ![De Azure Active Directory-knop][1]

2. Navigeer naar **bedrijfstoepassingen**. Ga vervolgens naar **alle toepassingen**.

    ![De blade Enterprise-toepassingen][2]
    
3. Om de nieuwe toepassing toevoegen, klikt u op **nieuwe toepassing** knop boven aan het dialoogvenster.

    ![De knop Nieuw toepassing][3]

4. Typ in het zoekvak **Bpm'online**, selecteer **Bpm'online** van resultaat deelvenster klik vervolgens op **toevoegen** om toe te voegen van de toepassing.

    ![Bpm'online in de lijst met resultaten](./media/active-directory-saas-bpmonline-tutorial/tutorial_bpmonline_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configureren en testen eenmalige aanmelding Azure AD

In deze sectie maakt u configureren en Azure AD test eenmalige aanmelding met Bpm'online op basis van een testgebruiker genaamd "Britta Simon."

Voor eenmalige aanmelding werkt, moet Azure AD weten wat de gebruiker equivalent in Bpm'online is voor een gebruiker in Azure AD. Met andere woorden, moet een koppeling relatie tussen een Azure AD-gebruiker en de betreffende gebruiker in Bpm'online tot stand worden gebracht.

Wijs in Bpm'online, de waarde van de **gebruikersnaam** in Azure AD als de waarde van de **gebruikersnaam** de relatie van de koppeling tot stand brengen.

Om te configureren en testen van Azure AD eenmalige aanmelding met Bpm'online, moet u de volgende bouwstenen voltooien:

1. **[Azure AD eenmalige aanmelding configureren](#configure-azure-ad-single-sign-on)**  : als u wilt dat uw gebruikers kunnen deze functie gebruiken.
2. **[Maken van een Azure AD-testgebruiker](#create-an-azure-ad-test-user)**  - voor het testen van Azure AD eenmalige aanmelding met Britta Simon.
3. **[Maak een testgebruiker Bpm'online](#create-a-bpmonline-test-user)**  - Bpm'online die is gekoppeld aan de Azure AD-weergave van de gebruiker van een exemplaar van Britta Simon bevatten.
4. **[Toewijzen van de Azure AD-testgebruiker](#assign-the-azure-ad-test-user)**  - Britta Simon gebruik van Azure AD eenmalige aanmelding inschakelen.
5. **[Test eenmalige aanmelding](#test-single-sign-on)**  : om te controleren of de configuratie werkt.

### <a name="configure-azure-ad-single-sign-on"></a>Eenmalige aanmelding Azure AD configureren

In dit gedeelte Azure AD eenmalige aanmelding inschakelen in de Azure portal en eenmalige aanmelding in uw toepassing Bpm'online configureren.

**Voor het configureren van Azure AD eenmalige aanmelding met Bpm'online, moet u de volgende stappen uitvoeren:**

1. In de Azure-portal op de **Bpm'online** toepassing Integratiepagina, klikt u op **eenmalige aanmelding**.

    ![Koppeling voor eenmalige aanmelding configureren][4]

2. Op de **eenmalige aanmelding** dialoogvenster Selecteer **modus** als **op basis van SAML aanmelding** voor eenmalige aanmelding inschakelen.
 
    ![Dialoogvenster voor eenmalige aanmelding](./media/active-directory-saas-bpmonline-tutorial/tutorial_bpmonline_samlbase.png)

3. Op de **Bpm'online domein en de URL's** sectie, voert u de volgende stappen uit als u wilt configureren, de toepassing in **IDP** modus gestart:

    ![URL's en Bpm'online domein eenmalige aanmelding informatie](./media/active-directory-saas-bpmonline-tutorial/tutorial_bpmonline_url.png)

    a. In de **id** textbox, typ een URL met het volgende patroon volgen:`https://<client site name>.bpmonline.com/`

    b. In de **antwoord-URL** textbox, typ een URL met het volgende patroon volgen:`https://<client site name>.bpmonline.com/ServiceModel/AuthService.svc/SsoLogin`

4. Controleer **weergeven geavanceerde instellingen voor URL** en voer de volgende stap als u wilt configureren van de toepassing in **SP** modus gestart:

    ![URL's en Bpm'online domein eenmalige aanmelding informatie](./media/active-directory-saas-bpmonline-tutorial/tutorial_bpmonline_url1.png)

    In de **aanmeldings-URL** textbox, typ een URL met het volgende patroon volgen:`https://<client site name>.bpmonline.com/`
     
    > [!NOTE] 
    > Deze waarden zijn niet echt. Deze waarden bijwerken met de werkelijke id, antwoord-URL en aanmeldings-URL. Neem contact op met [Bpm'online Client ondersteuningsteam](mailto:support@bpmonline.com) ophalen van deze waarden. 

5. Voor het genereren van de **metagegevens** -url, de volgende stappen uitvoeren:

    a. Klik op **App registraties**.
    
    ![Eenmalige aanmelding configureren](./media/active-directory-saas-bpmonline-tutorial/tutorial_bpmonline_appregistrations.png)
   
    b. Klik op **eindpunten** openen **eindpunten** in het dialoogvenster.  
    
    ![Eenmalige aanmelding configureren](./media/active-directory-saas-bpmonline-tutorial/tutorial_bpmonline_endpointicon.png)

    c. Klik op de knop kopiëren om te kopiëren **DOCUMENT met federatieve metagegevens** url en plak deze in Kladblok.
    
    ![Eenmalige aanmelding configureren](./media/active-directory-saas-bpmonline-tutorial/tutorial_bpmonline_endpoint.png)
     
    d. Nu gaat u naar de eigenschappenpagina van **Bpm'online** en kopieer de **toepassings-ID** met **kopie** knop en plak deze in Kladblok.
 
    ![Eenmalige aanmelding configureren](./media/active-directory-saas-bpmonline-tutorial/tutorial_bpmonline_appid.png)

     e. Genereren van de **metagegevens-URL** met het volgende patroon volgen:`<FEDERATION METADATA DOCUMENT url>?appid=<application id>` 

6. Klik op **opslaan** knop.

    ![Knop Single Sign-On opslaan configureren](./media/active-directory-saas-bpmonline-tutorial/tutorial_general_400.png)
    
8. Eenmalige aanmelding configureren op **Bpm'online** kant die u wilt verzenden de **metagegevens-URL** naar [Bpm'online ondersteuningsteam](mailto:support@bpmonline.com). Ze deze instelling zodat de SAML SSO-verbinding juist is ingesteld op beide zijden ingesteld.

> [!TIP]
> U kunt nu een beknopte versie van deze instructies binnen lezen de [Azure-portal](https://portal.azure.com), terwijl u de app instelt!  Na het toevoegen van deze app uit de **Active Directory > bedrijfstoepassingen** sectie, klikt u op de **Single Sign-On** tabblad en toegang tot de ingesloten documentatie via de **configuratie** sectie onderaan. U kunt meer lezen over de ingesloten documentatie-functie: [embedded-documentatie voor Azure AD]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="create-an-azure-ad-test-user"></a>Een Azure AD-testgebruiker maken

Het doel van deze sectie is het een testgebruiker maken in de Azure portal Britta Simon aangeroepen.

   ![Een Azure AD-testgebruiker maken][100]

**Als u wilt een testgebruiker maken in Azure AD, moet u de volgende stappen uitvoeren:**

1. Klik in de Azure-portal in het linkerdeelvenster op het **Azure Active Directory** knop.

    ![De Azure Active Directory-knop](./media/active-directory-saas-bpmonline-tutorial/create_aaduser_01.png)

2. Als u wilt weergeven in de lijst met gebruikers, gaat u naar **gebruikers en groepen**, en klik vervolgens op **alle gebruikers**.

    !['Gebruikers en groepen' en 'Alle gebruikers' koppelingen](./media/active-directory-saas-bpmonline-tutorial/create_aaduser_02.png)

3. Openen van de **gebruiker** in het dialoogvenster klikt u op **toevoegen** boven aan de **alle gebruikers** in het dialoogvenster.

    ![De knop toevoegen](./media/active-directory-saas-bpmonline-tutorial/create_aaduser_03.png)

4. In de **gebruiker** dialoogvenster vak, voert u de volgende stappen uit:

    ![Het dialoogvenster gebruiker](./media/active-directory-saas-bpmonline-tutorial/create_aaduser_04.png)

    a. In de **naam** in het vak **BrittaSimon**.

    b. In de **gebruikersnaam** typt u het e-mailadres van gebruiker Britta Simon.

    c. Selecteer de **wachtwoord weergeven** selectievakje, en noteer de waarde die wordt weergegeven in de **wachtwoord** vak.

    d. Klik op **Create**.
 
### <a name="create-a-bpmonline-test-user"></a>Een testgebruiker Bpm'online maken

In deze sectie kunt u een gebruiker Britta Simon aangeroepen in Bpm'online maken. Werken met [Bpm'online ondersteuningsteam](mailto:support@bpmonline.com) de gebruikers van het platform Bpm'online toevoegen. Gebruikers moeten worden gemaakt en worden geactiveerd voordat u eenmalige aanmelding gebruiken. 

### <a name="assign-the-azure-ad-test-user"></a>De Azure AD-testgebruiker toewijzen

In deze sectie schakelt u Britta Simon gebruikt Azure eenmalige aanmelding toegang verlenen aan Bpm'online.

![Toewijzen van de gebruikersrol][200] 

**Britta Simon om aan te wijzen Bpm'online, moet u de volgende stappen uitvoeren:**

1. Open de weergave toepassingen in de Azure-portal en gaat u naar de directoryweergave en gaat u naar **bedrijfstoepassingen** klikt u vervolgens op **alle toepassingen**.

    ![Gebruiker toewijzen][201] 

2. Selecteer in de lijst met toepassingen **Bpm'online**.

    ![De koppeling Bpm'online in de lijst met toepassingen](./media/active-directory-saas-bpmonline-tutorial/tutorial_bpmonline_app.png)  

3. Klik in het menu aan de linkerkant op **gebruikers en groepen**.

    ![De koppeling 'Gebruikers en groepen'][202]

4. Klik op **toevoegen** knop. Selecteer vervolgens **gebruikers en groepen** op **toevoegen toewijzing** dialoogvenster.

    ![Het deelvenster toewijzing toevoegen][203]

5. Op **gebruikers en groepen** dialoogvenster Selecteer **Britta Simon** in de lijst gebruikers.

6. Klik op **Selecteer** knop op **gebruikers en groepen** dialoogvenster.

7. Klik op **toewijzen** knop op **toevoegen toewijzing** dialoogvenster.
    
### <a name="test-single-sign-on"></a>Test eenmalige aanmelding

In deze sectie kunt u uw Azure AD eenmalige aanmelding configuratie met behulp van het toegangsvenster testen.

Als u op de tegel Bpm'online in het deelvenster toegang, u moet ophalen automatisch aangemeld bij uw toepassing Bpm'online.
Zie voor meer informatie over het toegangsvenster [Inleiding tot het toegangsvenster](active-directory-saas-access-panel-introduction.md). 

## <a name="additional-resources"></a>Aanvullende bronnen

* [Lijst met zelfstudies over het integreren van SaaS-Apps met Azure Active Directory](active-directory-saas-tutorial-list.md)
* [Wat is de toegang tot toepassingen en eenmalige aanmelding bij Azure Active Directory?](active-directory-appssoaccess-whatis.md)



<!--Image references-->

[1]: ./media/active-directory-saas-bpmonline-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-bpmonline-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-bpmonline-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-bpmonline-tutorial/tutorial_general_04.png

[100]: ./media/active-directory-saas-bpmonline-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-bpmonline-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-bpmonline-tutorial/tutorial_general_201.png
[202]: ./media/active-directory-saas-bpmonline-tutorial/tutorial_general_202.png
[203]: ./media/active-directory-saas-bpmonline-tutorial/tutorial_general_203.png

