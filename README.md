# Paul Santos 570 DC — ESPHome / Home Assistant

Projet d'intégration d'une VMC double flux **Paul Santos 570 DC** dans
[Home Assistant](https://www.home-assistant.io/) à l'aide d'un
[ESP32-C3 Super Mini](https://www.espressif.com/).

L'objectif est de superviser et de piloter la VMC depuis Home Assistant,
tout en conservant autant que possible le fonctionnement de sa commande
d'origine.

> **État du projet : expérimental**
>
> Le matériel retenu et les premières pistes logicielles sont identifiés.
> Le protocole, le câblage définitif, les commandes disponibles et les
> fonctions exposées dans Home Assistant restent à confirmer par des essais
> sur la VMC.

## Objectifs

- Intégrer la VMC à Home Assistant grâce à ESPHome.
- Superviser les températures, les ventilateurs, les volets et les états disponibles.
- Piloter les réglages accessibles par le protocole : température de confort,
  niveau de ventilation et répartition soufflage / extraction.
- Étudier la commande du bypass en fonctionnement normal.
- Conserver la possibilité d'utiliser le clavier d'origine.

Le dialogue série avec la VMC est désormais fonctionnel. La couche physique,
la réception des trames et plusieurs réponses du protocole ont été validées
sur le matériel réel. Les commandes avancées restent à tester individuellement.

## Matériel

### VMC

- Modèle : Paul Santos 570 DC.
- Commande d'origine : clavier à membrane.
- Connecteur X6 : sérigraphié `RS232 - KFB`, avec les bornes `12V`, `TX`,
  `RX` et `GND`.
- Connecteur X8 : connecteur DB9 sérigraphié `RS232 - PC`.

Le rôle exact des deux interfaces et leurs niveaux électriques doivent être
confirmés avant le raccordement de l'ESP32.

### Microcontrôleur

- ESP32-C3 Super Mini.
- Connexion Wi-Fi au réseau local.
- Firmware ESPHome.
- Intégration à Home Assistant.

L'ESP32-C3 est utilisé pour sa compacité et sa compatibilité avec ESPHome.
Le Wi-Fi suffit pour cette intégration ; aucune fonction Zigbee n'est
nécessaire au projet actuel.

### Adaptation série : MAX3232

Un **MAX3232** est prévu pour assurer l'adaptation électrique entre une
véritable liaison RS-232 et l'UART TTL de l'ESP32-C3.

Le MAX3232 ne doit être utilisé que si les niveaux électriques de
l'interface choisie sont bien ceux d'une liaison RS-232.

Attention :
- Ne jamais raccorder directement une liaison RS-232 aux GPIO de l'ESP32.
- Ne pas supposer que les bornes TX et RX du connecteur X6 sont en niveaux
  TTL ou RS-232 sur la seule base de leur nom.
- Vérifier le brochage du DB9 X8 avant tout raccordement.
- Vérifier les tensions et la masse avec la documentation et des mesures
  adaptées avant de connecter le matériel.

Le MAX3232 assure une adaptation de niveaux électriques ; il ne convertit
pas un protocole propriétaire en un autre protocole.

### Alimentation

Un convertisseur abaisseur DC-DC a été retenu :

- Produit acheté : [convertisseur abaisseur DC-DC sur Amazon.fr](https://www.amazon.fr/dp/B0HFJYQ8WL)
- Référence Amazon : `B0HFJYQ8WL`

Il est destiné à fournir une alimentation adaptée à l'ESP32-C3 à partir
d'une source de tension supérieure, sous réserve de confirmer les
caractéristiques du module et la tension disponible sur la VMC.

Avant le montage définitif :
- vérifier la plage de tension d'entrée du convertisseur ;
- régler et mesurer la tension de sortie avant de raccorder l'ESP32 ;
- vérifier que le courant disponible est suffisant ;
- respecter les polarités ;
- vérifier que la sortie choisie correspond à l'entrée d'alimentation
  autorisée par la carte ESP32-C3 utilisée.

Ne pas considérer la borne `12V` du connecteur X6 comme une alimentation
disponible pour le projet avant d'avoir confirmé sa fonction, sa capacité
en courant et ses conditions d'utilisation.

## Interfaces de communication

Deux connecteurs sont présents sur la carte de la VMC.

| Connecteur | Sérigraphie | Fonction connue |
| --- | --- | --- |
| X6 | `RS232 - KFB` | Interface du clavier d'origine, utilisée pour les essais |
| X8 | `RS232 - PC` | Connecteur DB9, non utilisé dans le montage actuel |

**Le montage actuel utilise X6 à la place du clavier** : l'ESP32 est raccordé
sur cette interface et le clavier est débranché. Le fonctionnement du clavier
en parallèle n'est donc pas validé.

### Câblage validé

- ESP32-C3 Super Mini.
- UART ESPHome : GPIO0 en TX et GPIO1 en RX.
- Adaptation électrique par MAX3232 alimenté en 3,3 V.
- Liaison série : 9600 bauds, 8N1.

Les essais de boucle locale ont confirmé le bon fonctionnement de l'UART
et du MAX3232, y compris côté RS-232. La communication avec la VMC sur X6
a ensuite été obtenue après correction du raccordement.

Ne pas transposer ce câblage au connecteur X8 sans vérifier son brochage.
Ne jamais raccorder directement une liaison RS-232 aux GPIO de l'ESP32.

### UART proxy

Le composant comporte également un UART proxy destiné au relais des échanges
avec le clavier. La configuration actuelle fournit un second UART pour
satisfaire cette interface logicielle, mais celui-ci n'est pas raccordé
au clavier dans le montage où l'ESP32 remplace celui-ci.

## Protocole série

Paramètres validés sur l'interface X6 :

- Débit : `9600 bauds`
- Bits de données : `8`
- Parité : aucune
- Bits d'arrêt : `1`

Soit **9600 bauds, 8N1**.

Les trames de requête et les réponses de la VMC sont maintenant reçues et
décodées. Les commandes et réponses sont décrites dans
`components/comfoair/registers.h`.

Documentation et références :

- [Description du protocole ComfoAir (PDF)](http://www.see-solutions.de/sonstiges/Protokollbeschreibung_ComfoAir.pdf)
- [Module FHEM 98_ComfoAir.pm](https://github.com/mhop/fhem-mirror/blob/master/fhem/FHEM/98_ComfoAir.pm)

La compatibilité a été vérifiée par des échanges réels avec la Paul Santos
570 DC, et non uniquement déduite de la documentation générique ComfoAir.

### Commandes identifiées

- `0x99` — réglage du niveau de ventilation.
- `0xCF` — réglage des consignes de ventilation (soufflage et extraction).
- `0xD3` — réglage de la température de confort.

Le composant possède déjà le service ESPHome
`climate_set_operation_mode(exhaust_fan, supply_fan)`, qui permet de
configurer indépendamment les deux ventilateurs. Les combinaisons prévues
sont :

| Mode | Extraction | Soufflage |
| --- | --- | --- |
| Double flux | Activée | Activé |
| Soufflage seul | Désactivée | Activé |
| Extraction seule | Activée | Désactivé |

Ces commandes sont présentes dans le code, mais les trois modes doivent
encore être testés sur la VMC réelle.

### Bypass

La lecture de l'état du bypass est disponible. La commande documentée
`0x09` (`CMD_SET_FLAPS`) appartient au mode test du contrôleur.
Elle ne doit pas être utilisée comme commande normale de bypass sans
validation complémentaire. La méthode de pilotage du bypass en fonctionnement
normal reste donc à identifier.

### Correction du décodage du préchauffage

La réponse `0x0E` (`RES_GET_VALVE_STATUS`) fournit notamment la position
du bypass et l'état du volet de préchauffage. La réponse `0xE2`
(`RES_GET_PREHEATING_STATUS`) fournit séparément l'état du préchauffage.

Le composant publiait auparavant ces deux informations dans le même capteur
`preheating_state`, ce qui provoquait une alternance artificielle entre
ON et OFF à chaque cycle de lecture.

Le décodage a été corrigé :
- `preheating_valve` est alimenté par l'état du volet reçu dans `0x0E`
  et interprète les valeurs 0 (Closed), 1 (Open) et 2 (Unknown).
- `preheating_state` est alimenté uniquement par la réponse `0xE2`.
- L'état de protection antigel reste décodé séparément.

Sur la configuration actuellement testée, le préchauffage est éteint et
la réponse indique bien `preheating_state = OFF`.

## Sources logicielles et crédits

Ce projet s'appuie sur des travaux existants autour du protocole série
ComfoAir et de son intégration à ESPHome.

### Composant ESPHome de départ

- Dépôt utilisé comme point de départ :
  [julianpas/esphome-comfoair](https://github.com/julianpas/esphome-comfoair)
- Projet d'origine :
  [wichers/esphome-comfoair](https://github.com/wichers/esphome-comfoair)

Le projet de Julian Pas adapte le composant d'origine pour permettre
notamment le partage de la liaison série avec une commande ComfoSense.
Le code existant sert de base d'étude et d'adaptation ; la compatibilité
avec le Paul Santos 570 DC n'est pas présumée.

### Référence complémentaire

- [Module FHEM 98_ComfoAir.pm](https://github.com/mhop/fhem-mirror/blob/master/fhem/FHEM/98_ComfoAir.pm)

Ce module constitue une source complémentaire pour étudier le protocole,
les commandes et les réponses possibles. Il ne s'agit pas d'un composant
ESPHome directement réutilisable sans adaptation.

### Attribution

Les auteurs des projets d'origine conservent leurs droits sur leurs
contributions respectives. Les adaptations propres à ce dépôt devront
être identifiées clairement, sans supprimer les mentions de copyright
ni les avis de licence existants.

## Licence

Le composant ESPHome de départ est publié sous **GNU General Public License
v3.0 (GPL-3.0)**.

Le code dérivé de ce composant doit respecter les obligations de cette
licence. Le dépôt doit notamment conserver les mentions de droits d'auteur
et de licence, identifier les modifications apportées et fournir le texte
de la licence.

Le fichier `LICENSE` du dépôt doit contenir le texte intégral de la GPL-3.0,
en conservant les mentions pertinentes du projet d'origine.

Le module FHEM cité en référence annonce une licence GPL-2.0 ou, au choix,
toute version ultérieure. Toute reprise effective de code de ce module
devra être documentée, avec conservation des attributions et vérification
des conditions applicables aux portions reprises.

Les fichiers de code nouveaux ou modifiés distribués dans le cadre de ce
projet dérivé seront publiés conformément à la GPL-3.0, sous réserve de
vérifier les licences des éventuelles autres dépendances.

Le document de protocole cité ci-dessus constitue une source distincte ;
ses conditions de réutilisation restent à vérifier.

## Architecture logicielle envisagée

L'intégration sera réalisée sous la forme d'un composant externe ESPHome,
avec une configuration YAML propre à la VMC.

Organisation envisagée :

    components/
    └── comfoair/
        ├── __init__.py
        ├── comfoair.h
        └── comfoair.cpp

    example/
    └── santos570dc.yaml

    README.md
    LICENSE

Cette structure est indicative et pourra évoluer en fonction du composant
de départ effectivement conservé et des fonctionnalités ajoutées.

Le code du composant sera chargé de :
- gérer la liaison UART ;
- envoyer les requêtes nécessaires ;
- analyser les réponses reçues ;
- exposer les informations et commandes prises en charge par le protocole.

La configuration YAML définira les broches, les paramètres de liaison,
les capteurs et les commandes disponibles.

## Fonctionnalités

### Supervision déjà opérationnelle

Les informations suivantes sont lues depuis la VMC et exposées dans ESPHome
(selon les capteurs activés dans la configuration) :

- Températures extérieure, soufflage, reprise et extraction.
- Pourcentages et vitesses de rotation des ventilateurs de soufflage et
  d'extraction.
- Niveau de ventilation.
- Position du bypass et informations de commande du bypass.
- État du volet de préchauffage et état du préchauffage.
- État de la protection antigel.
- État du filtre.
- Informations complémentaires : heures de fonctionnement, paramètres
  du bypass et caractéristiques de la machine.

La réception des trames et le décodage de plusieurs réponses ont été validés
sur la Santos 570 DC.

### Commandes disponibles dans le composant

- Réglage de la température de confort.
- Réglage du niveau de ventilation.
- Réglage des consignes de soufflage et d'extraction.
- Sélection indépendante des ventilateurs de soufflage et d'extraction
  via le service `climate_set_operation_mode`.

La présence d'une commande dans le composant ne signifie pas encore que
tous ses modes ont été validés sur la machine. Les essais de pilotage restent
à effectuer progressivement.

### Fonctions encore à étudier

- Commande du bypass en fonctionnement normal.
- Validation sur la machine des modes soufflage seul, extraction seule
  et double flux.
- Fonctionnement du clavier d'origine en parallèle, ou mise en place
  d'un relais série si cela est techniquement possible.

## Plan de développement

1. [x] Identifier le modèle et les interfaces de la VMC.
2. [x] Valider l'UART de l'ESP32-C3 et le MAX3232 par boucle locale.
3. [x] Établir la communication série avec la VMC sur X6.
4. [x] Valider la réception et le décodage de plusieurs trames.
5. [x] Exposer les premières informations de supervision dans ESPHome.
6. [x] Corriger le décodage de l'état du préchauffage.
7. [ ] Tester les trois modes de ventilation depuis Home Assistant.
8. [ ] Identifier une commande normale et sûre du bypass.
9. [ ] Étudier la possibilité de conserver le clavier d'origine.
10. [ ] Compléter la documentation du câblage et des entités Home Assistant.

## Sécurité et précautions

- Ne pas raccorder directement du RS-232 aux GPIO de l'ESP32-C3.
- Ne pas supposer les niveaux électriques à partir de la sérigraphie seule.
- Ne pas injecter de tension sur TX ou RX.
- Ne pas émettre de commandes tant que les trames et leurs effets ne sont
  pas compris.
- Ne pas perturber la commande d'origine.
- Vérifier le brochage avant tout branchement.
- Ne pas effectuer de modifications susceptibles de compromettre la
  sécurité ou le fonctionnement normal de la VMC.

## État actuel

<badge>Communication série opérationnelle</badge>

- [x] ESP32-C3 Super Mini et MAX3232.
- [x] UART GPIO0 (TX) / GPIO1 (RX), 9600 bauds, 8N1.
- [x] Boucle locale UART et boucle côté RS-232 validées.
- [x] Communication avec la VMC sur X6.
- [x] Réception et décodage des réponses du contrôleur.
- [x] Premiers capteurs de supervision opérationnels.
- [x] Correction du conflit de publication de `preheating_state`.
- [x] Service logiciel de sélection soufflage / extraction présent.
- [ ] Validation pratique des trois modes de ventilation.
- [ ] Commande du bypass en fonctionnement normal.
- [ ] Clavier d'origine utilisable en parallèle.

Le montage actuel remplace le clavier sur X6. Le fonctionnement simultané
du clavier et de l'ESP32 n'est pas encore établi.

---

Projet personnel de rétro-ingénierie et d'intégration domotique.
La compatibilité, les commandes et les limites de fonctionnement restent
à confirmer sur le matériel réel.
