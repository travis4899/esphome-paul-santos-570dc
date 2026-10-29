
# ESPHome – Paul Santos 570 DC

Intégration d'une VMC double flux **Paul Santos 570 DC** dans Home Assistant à l'aide d'un ESP32-C3 et d'ESPHome, via le protocole série ComfoAir.

> **Statut : développement**
>
> Le projet est en cours de développement et de validation sur une Paul Santos 570 DC.
> Les commandes disponibles, les états remontés et la compatibilité matérielle doivent être confirmés sur l'installation réelle.

## Objectif du projet

L'objectif est de piloter et de superviser la VMC Paul Santos 570 DC depuis Home Assistant, sans remplacer son électronique ni compromettre ses fonctions de régulation et de sécurité d'origine.

Le projet vise notamment à :

- récupérer les états et les mesures disponibles ;
- consulter les températures et les paramètres de fonctionnement ;
- commander les niveaux de ventilation ;
- gérer les modes et les consignes lorsque le protocole le permet ;
- intégrer les fonctions disponibles dans Home Assistant ;
- conserver, autant que possible, le clavier de commande d'origine opérationnel.

L'intégration repose sur un ESP32-C3 exécutant ESPHome et communiquant avec la VMC par une liaison série.

## Origine et références

Ce projet s'appuie sur des travaux et documentations existants autour du protocole ComfoAir.

### Projets existants

- [julianpas/esphome-comfoair](https://github.com/julianpas/esphome-comfoair)  
  Composant ESPHome servant de base au développement de l'intégration.

- [mhop/fhem-mirror – 98_ComfoAir.pm](https://github.com/mhop/fhem-mirror/blob/master/fhem/FHEM/98_ComfoAir.pm)  
  Implémentation du protocole ComfoAir pour FHEM. Cette source constitue une référence pour les commandes, les réponses et le décodage des données.

### Documentation du protocole

- [Protokollbeschreibung ComfoAir (PDF)](http://www.see-solutions.de/sonstiges/Protokollbeschreibung_ComfoAir.pdf)

Cette documentation décrit le protocole série ComfoAir et sert de référence pour comprendre les échanges avec la VMC.

Les commandes et les fonctions documentées dans ces projets ne sont pas nécessairement toutes compatibles avec la Santos 570 DC. Elles doivent être vérifiées sur l'équipement réel.

Les licences des projets sources doivent être consultées avant toute réutilisation ou redistribution de leur code.

## Matériel

### Matériel principal

- VMC double flux Paul Santos 570 DC.
- ESP32-C3 Super Mini.
- Convertisseur abaisseur DC-DC 12 V vers 5 V.
- Interface électrique adaptée à la liaison série utilisée.
- Câblage et connecteurs adaptés au montage définitif.

### Interface de communication

Deux interfaces sont présentes sur la carte de la VMC :

- **X6 – RS232-KFB** : liaison avec le clavier à membrane d'origine.
- **X8 – RS232-PC (DB9)** : interface série destinée à la communication avec un ordinateur.

Le port DB9 est la première interface envisagée pour le développement, afin de limiter les modifications du câblage d'origine.

Le raccordement au clavier via X6 reste une piste alternative, notamment si certaines fonctions ne sont pas accessibles par le port PC.

Le brochage, les niveaux électriques et les possibilités de commande de chaque interface doivent être confirmés avant raccordement.

### Alimentation

L'alimentation du montage est prévue à partir d'une source 12 V de la VMC, sous réserve de vérifier sa disponibilité et son courant maximal.

Un convertisseur DC-DC abaisseur fournit le 5 V nécessaire à la carte ESP32-C3.

**Ne jamais raccorder directement une liaison RS232 aux GPIO de l'ESP32.** Une interface électrique adaptée est indispensable. Le brochage et les tensions doivent être vérifiés avant la mise sous tension du montage.

## Communication série

Les paramètres série relevés dans la documentation du protocole sont :

| Paramètre | Valeur |
|---|---|
| Débit | 9600 bauds |
| Bits de données | 8 |
| Parité | Aucune |
| Bits de stop | 1 |
| Format | 9600 8N1 |

Ces paramètres constituent la configuration de départ. Ils doivent être confirmés sur l'interface effectivement utilisée.

Le protocole ComfoAir définit les commandes, les réponses et le format des données échangées. Le composant ESPHome doit gérer l'émission des requêtes, la réception des réponses, leur validation et le décodage des valeurs.

## Architecture logicielle

Le projet utilise un composant externe ESPHome, écrit en Python et en C++.

- **Python** : déclaration et validation de la configuration ESPHome.
- **C++** : gestion de la communication série, des trames du protocole et des données.
- **YAML** : configuration de l'ESP32-C3 et des entités exposées à Home Assistant.

Organisation envisagée :

```text
components/
└── comfoair/
    ├── __init__.py
    ├── comfoair.h
    ├── comfoair.cpp
    ├── sensor.py
    ├── switch.py
    └── select.py

example/
└── santos570dc.yaml

README.md
LICENSE
```

Cette structure pourra évoluer selon l'organisation réelle du composant repris et les besoins de l'intégration.

## Fonctions envisagées

Les fonctions suivantes sont des objectifs du projet. Leur disponibilité dépend du protocole, du firmware de la VMC et de leur validation sur une Santos 570 DC.

### Supervision

- Lecture des températures disponibles.
- Lecture des niveaux et états de ventilation.
- Lecture des paramètres de fonctionnement.
- Lecture des informations de maintenance et des heures de fonctionnement, si disponibles.
- Lecture des états du bypass et de la protection antigel, si exposés par le protocole.
- Remontée des erreurs et des défauts, si disponibles.

### Commande

- Modification du niveau de ventilation.
- Activation d'un mode boost ou d'une vitesse temporaire, si pris en charge.
- Modification de la consigne de confort, si prise en charge.
- Accès aux autres paramètres documentés et compatibles.

Aucune commande ne sera considérée comme fonctionnelle avant d'avoir été testée sur la VMC.

## Installation

Les instructions d'installation seront complétées après validation du composant et de sa configuration.

Le principe prévu est d'ajouter le dépôt comme composant externe dans ESPHome, puis de configurer l'UART de l'ESP32-C3 et les entités souhaitées.

La configuration définitive devra notamment préciser :

- les GPIO RX et TX retenus ;
- l'interface électrique utilisée ;
- le mode de communication ;
- les options et les entités prises en charge.

## Feuille de route

- [ ] Étudier le composant ESPHome existant.
- [ ] Vérifier les licences et les conditions de réutilisation.
- [ ] Confirmer le brochage et les niveaux électriques de l'interface série.
- [ ] Valider l'alimentation 12 V vers 5 V.
- [ ] Compiler le composant sur ESP32-C3.
- [ ] Établir une communication série fiable.
- [ ] Valider une première commande de lecture.
- [ ] Vérifier les températures et les états remontés.
- [ ] Ajouter les capteurs Home Assistant.
- [ ] Ajouter les commandes de ventilation.
- [ ] Tester la coexistence avec le clavier d'origine.
- [ ] Documenter le câblage définitif et les fonctions validées.

## Sécurité et précautions

Ce projet intervient sur un équipement de ventilation disposant de sa propre régulation.

- Ne pas modifier les sécurités internes de la VMC.
- Ne pas perturber les protections antigel ou les régulations d'origine.
- Ne pas raccorder directement les GPIO de l'ESP32 à une interface RS232.
- Vérifier les tensions et le brochage avant tout raccordement.
- Commencer par les lectures non destructives avant d'essayer des commandes.
- Conserver le fonctionnement du clavier d'origine autant que possible.

Le montage doit être réalisé de manière à ne pas empêcher le fonctionnement normal de la ventilation en cas de panne de l'ESP32 ou de perte de communication.

## Contributions et retours

Les retours d'expérience, les corrections et les tests sur d'autres équipements compatibles avec le protocole ComfoAir sont les bienvenus.

Les informations sur le modèle exact de VMC, le firmware, le câblage et les commandes effectivement testées sont particulièrement utiles.

## Licence

À définir après vérification des licences des projets sources et des conditions de réutilisation du code.
