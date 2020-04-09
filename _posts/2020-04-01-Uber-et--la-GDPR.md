---
layout: post
title: "Uber et la GDPR"
date: 2020-04-01
author: "guiral Lacotte"
language: french 
tags: GDPR Python leaflet 
---
![Smartphone](/assets/img/kon-karampelas-9BbkkdurAnU-unsplash.jpg)
En ces temps de confinement Covid-19, j'ai enfin prit le temps d'effectuer quelques projet que j'avais dans mes calpins depuis un certain temps. L'un d'eux etait d'effectuer un maximum de demandes d'accès à mes données personnels via la [GDPR](https://fr.wikipedia.org/wiki/R%C3%A8glement_g%C3%A9n%C3%A9ral_sur_la_protection_des_donn%C3%A9es) et d'étudier les données qui m'étaient envoyés. Cela afin d'évaluer la mise en oeuvre de la GDPR par les différentes entreprises et plateformes. De détailler les modalités d'accès, car j'en connais quelque uns d'entre vous pour qui il est plus de facilités a écrire une requête vers une [API](https://en.wikipedia.org/wiki/Application_programming_interface) que d'écrire une lettre manuscrite avec photocopie de pièce d'identité vers la boite postale du service juridique (regard accusateur en direction du système bancaire et les opérateurs télécoms français). Mais aussi de mieux comprendre, le fonctionnement et le modèle économique de ces plateformes. Et enfin, avoir des données réels afin de m'exercer à la visualisation de données de géolocalisation avec [Python](https://www.python.org/), [Folium](https://python-visualization.github.io/folium/) et [Leaflet](https://leafletjs.com/).

Le but est de faciliter le travail pour des néophytes et d'encourager les utilisateurs a exercer leurs droits d'accès (un droit que l'on utilise pas c'est un droit qui meure). Il n'est pas prévue de se lancer dans des analyses complexes (reverse d'application mobile ou d'API, dés-anonymisation). Je n'ai malheureusement ni le temps, ni les compétence requise pour le faire.

Dans le premier article, je vais m'attaquer à la plateforme Uber. Ce n'est pas la première analyse que j'ai effectué mais c'est l'une des plus intéressantes et celle qui touche la population la plus importante.

## La demande d'accès
On peut demander ses donnée directement depuis le site de Uber via cette page: [help.uber.com/riders/..](https://help.uber.com/riders/article/que-contiennent-vos-donn%C3%A9es-t%C3%A9l%C3%A9charg%C3%A9es%C2%A0?nodeId=3d476006-87a4-4404-ac1e-216825414e05). Pour effectuer la demande il faut avoir conservé son compte Uber et le numéro de téléphone utilisé lors de l'enregistrement de l'application mobile. L'authentification sur le site web depuis un nouvel appareil et pour ce type de demande nécessite de l'authentification deux facteurs qui s'effectue par l'envoie d'un code par SMS. Comparativement a mes autres demandes qui nécessite souvent de passer par des échanges de mails avec le responsable [GDPR](https://fr.wikipedia.org/wiki/R%C3%A8glement_g%C3%A9n%C3%A9ral_sur_la_protection_des_donn%C3%A9es) de la société, cela reste plutôt simple. Ma demande a été traité en une trentaine d'heure, ce qui est là aussi plutôt bien. On reçoit un lien par mail qui pointe vers une page de téléchargement. Le lien est valide que 7 jours et la page nécessite une authentifier deux facteurs. Les données sont envoyé sous forme d'une archive [ZIP](https://fr.wikipedia.org/wiki/ZIP_(format_de_fichier)) contenant une arborescence et plusieurs fichiers [CVS](https://fr.wikipedia.org/wiki/Comma-separated_values). 

En ce qui concerne la demande d'accès, la transmission et sécurité, Uber fait partie des bons élèves.

## Les données collectées et transmises
Les données collectées par Uber et transmises dans le cadre des demandes GDPR sont listées là : [help.uber.com/riders/..](https://help.uber.com/riders/article/que-contiennent-vos-donn%C3%A9es-t%C3%A9l%C3%A9charg%C3%A9es%C2%A0?nodeId=3d476006-87a4-4404-ac1e-216825414e05)
Ce qui va particulièrement m’intéresse ce sont les données de géolocalisation. 

> DONNÉES DU PASSAGER   Vos données de passager contiennent les informations utilisées pour vous amener à destination, notamment :   
>  - Les détails des courses tels que **les heures et lieux de demande**, de    début et de fin, ainsi que les distances parcourues ;
>  - le prix des courses et la devise utilisée ;
>  - 30 jours de données d'événements mobiles, telles que le système d'exploitation de    l'appareil, le modèle de l'appareil, la langue de l'appareil, la version de l'application, ainsi que **l'heure et le lieu de la collecte des données**.

L'arborescence de l'archive est comme suit:

    Uber Data/
    ├── Account and Profile
    │   ├── payment_methods.csv
    │   └── profile_data.csv
    ├── readme.html
    ├── Regional information
    │   └── For_California_users.txt
    └── Rider
        ├── rider_app_analytics-0.csv
        └── trips_data.csv

 - **payment_methods.csv** : La liste des cartes et moyen de payment actuels ou passés.
 - **profile_data.csv** : Correspond au profile utilisateur, on y trouve nos première donnée de géolocation via Signup Lat et Signup Long qui sont les coordonnées lors de l'enregistrement sur l'application. Le champ `Referred to Uber?` correspond au code de vote éventuel parrain, le champ `Referral Code` correspond au code de parrainage. 
 - **readme.html** : Une copie de la page listant les données collectés.
 - **For_California_users.txt** : Un utilisateur peu attentif pourrait ne passer ce fichier texte, mais celui-ci est en réalité très intéressant. Il liste les catégories de partenaires qui peuvent potentiellement avoir accès aux données collectés par Uber et les partenaires avec les quels Uber a échangé vos donnés lors des douze derniers moi. Par exemple dans mon cas:`In the past 12 months, Uber has sold your personal data to the following categories of third parties: -Advertising network providers`. Uber est plutôt avare de détail au sujet de ses partenaires commerciaux. 
 - **rider_app_analytics-0.csv** : On attaque les choses sérieuses, ce fichier contient les "*30 jours de données d'événements mobiles"*. C'est une mine d'information et particulièrement d'information de géolocalisation. Le volume d'information est particulièrement étonnant . Fin Mars 2020 après deux semaines de confinement et une utilisation quasi nul de Uber exception faite des manipulation nécessaire à la demande d'accès, le fichier compte plus de 2000 lignes.
 - **trips_data.csv** :  Deuxième fichier particulièrement intéressant, celui-ci contient l'ensemble des  "*détails des courses tels que les heures et lieux de demande, de    début et de fin, ainsi que les distances parcourues*", depuis l'enregistrement sur la plateforme. 

### rider_app_analytics-0.csv
Comme nous l'avons vue ce fichier comporte toutes les données d'événements mobiles.
A titre d'exemple, voici un extrait d'une interaction avec l'application. Lors d'une course à l'autre bout de Paris, suite à une panne sur l'une des ligne de métro j'ai lancé l'application afin d'évaluer l'opportunité de prendre un Uber ou les transports en commun. J'ai finalement pris les transports en commun sans effectuer de réservations. D'après les données d'horodatage, l'opération aura prit 2,966 secondes et aura générée 332 lignes d'événements. 

 - Event Time (UTC),Session Start Time (UTC),GPS Time (UTC),Horizontal Accuracy,Latitude,Longitude,Speed (GPS),City,Cellular Carrier,Carrier MCC,Carrier MNC,Google Play Services Status,Device ID,IMEI number,IP Address,Device Language,Device Model,Device OS,Device OS Version,Region,Analytics Event Type 
- 2020-03-07 **13:07:53.469**	2020-03-07 13:07:49.979	2020-03-07 13:07:53.763	9.5	48.82764	2.33204	0.0	paris	Orange F	208	1	success	e80b48a6-08c6-4b4a-b457-XXX		10.35.167.XXX 100.69.178.XXX	fr	SM-A600FN	android	9	fr_FR	custom

- **... 330 lignes ...**

- 2020-03-07 **13:07:56.435**	2020-03-07 13:07:49.979	2020-03-07 13:07:53.763	9.5	48.82764	2.33204	0.0	paris	Orange F	208	1	success	e80b48a6-08c6-4b4a-b457-XXX		10.35.167.XXX 100.69.178.XXX	fr	SM-A600FN	android	9	fr_FR	custom

Quand l'on dépouille le fichier, il apparaît que chaque ligne correspond à une interaction avec l'appli ou à une réquête de l'API d'Uber. On retrouve trois type d' "Analytics Event Type", custom, tap et impression. Chaque déplacement de la carte, zoom, dé-zoom, ou clic est enregistré. 
le champ `GPS Time (UTC)` correspond à la date de la dernière position GPS connue.
Le champ `Horizontal Accuracy` correspond à l'écart type sigma de position horizontale. Il est particulièrement utile pour savoir si l'utilisateur est à l'intérieur ou à l'extérieur d'un bâtiment.
les champs `Latitude,Longitude` sont codés en degrés décimaux sur 5 décimales.
les champs `Carrier MCC, Carrier MNC`, correspondent au code [Mobile_country_code](https://en.wikipedia.org/wiki/Mobile_country_code) et [Mobile_Network_Codes](https://en.wikipedia.org/wiki/Mobile_Network_Codes_in_ITU_region_2xx_(Europe)#France_-_FR), 208 01 correspond a Orange France.
le champ `Device ID` correspond au Google Play Services ID for Android
le champ `IMEI number` est vide. C'est peu être un résidu de donnée qui ne sont plus collectés, ou de manière conditionnel (en l'absence de Device ID, lors de l'enregistrement).

En résumé ce fichier nous permet de tracer tout les lieux et dates d'interaction de l'utilisateur avec l'application, même si celui-ci n'effectue aucune réservation. Il nous permet aussi de savoir si il était en intérieur ou à l'extérieur, en déplacement ou statique. Si l'utilisateur utilisent plusieurs terminaux il permet aussi d'identifier les terminaux, les réseaux mobiles utilisés, leurs modèles et la version du système d'exploitation.

Je n'ai malheureusement pas de donnée pour un trajet réel (reservation, prise en charge, trajet, dépose) sur cette période confinement oblique. Je ne suis pas en mesure d'évaluer la fenêtre temporel d'enregistrement de l'application lors d'un trajet. Je n'ai pas observé d'activité dans les journaux lors que l'application n'est pas en premier plan sur le téléphone.

### trips_data.csv
C'est le fichier qui contient les journaux de"*détails des courses tels que les heures et lieux de demande, de    début et de fin, ainsi que les distances parcourues*". Il est de structure assez simple. Voici un exemple d'un aller retour, afin de préserver ma vie privé j'ai choisit un trajet *légèrement* éloigné de mes lieux de travail et d'habitation. 

 - City,	Product Type,	Trip or Order Status,	Request Time,	Begin Trip Time,	Begin Trip Lat,	Begin Trip Lng,	Begin Trip Address,	Dropoff Time,	Dropoff Lat,	Dropoff Lng	Dropoff Address,	Distance (miles),	Fare Amount,	Fare Currency
- Cape Town	UberX	COMPLETED	2019-02-15 19:42:32 +0000 UTC	2019-02-15 19:44:27 +0000 UTC	-33.92129	18.41808	105 Bree St, Cape Town City Centre, Cape Town, 8001, Afrique du Sud	2019-02-15 19:51:24 +0000 UTC	-33.90604	18.41783	3 Portswood Rd, V & A Waterfront, Cape Town, 8002, Afrique du Sud	1.66	31	ZAR
- Cape Town	UberX	COMPLETED	2019-02-15 17:28:51 +0000 UTC	2019-02-15 17:30:03 +0000 UTC	-33.9060588	18.417833	3 Portswood Rd, V & A Waterfront, Cape Town, 8002, Afrique du Sud	2019-02-15 17:39:16 +0000 UTC	-33.9244897	18.4195529	5 Wale St, Cape Town, 8001, ZA	1.94	40	ZAR

Chaque ligne correspond à un trajet, on retrouve les coordonnées de prise en charge `Begin Trip Lat,	Begin Trip Lng` et de dépose `Dropoff Lat,	Dropoff Lng`. Elles sont codées codés en degrés décimaux sur 5 décimales.
Ce fichier est moins verbeux que le précédents, mais nous permet de retracer l'ensemble de l'historique de réservation de l'utilisateur. On peut noter que dans une volonté de nous faciliter l'analyse Uber nous donne les adresses postales des lieux de prise en charge et de dépose, en français. Dans cet exemple on peut facilement identifier que j'effectuais un aller retour d'un hôtel vers un restaurant.  

## Visualisation des données de géolocalisation 
Le code pour visualiser les points d'activité utilisateur et les trajets est disponible là:
[https://github.com/berettavexee/Uber_map](https://github.com/berettavexee/Uber_map)

![Capture d'écran 00](/assets/img/ubermap-00.png)
![Capture d'écran 01](/assets/img/ubermap-01.png)

C'est assez simple d'utilisation, il vous suffit de déposer vos fichiers `rider_app_analytics-0.csv` et `trips_data.csv` dans le sous repertoire uber et de lancer le script via `python3 uber_map.py` .

## Conclusions

 - Il est plutôt facile d'effectuer la demande d'accès GDPR.
 - Les données fournies sont facilement exploitables.
 - Uber possède un graph des parrainage
 - Uber possède l'historique de toutes les interactions, qui inclue le dernier point de géolocalisation connue sur les 30 derniers jours.
 - Uber conserve les données techniques en mesure d'identifier tous les appareils, réseau téléphonique et réseau IP utilisés sur les 30 derniers jours.
 - Uber conserve un historique complete des trajets effectués.
 - Certaines données fournit par l'utilisateurs comme la notation des chauffeurs ou des trajets n'ont pas été transmises. 

# remerciements 
- [Raphael Rigo](https://syscall.eu/blog/) pour ses conseils et sa relecture.