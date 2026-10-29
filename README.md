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
- Lire les informations disponibles sur la liaison série.
- Exposer les informations pertinentes sous forme de capteurs.
- Permettre, si le protocole le confirme, de modifier les réglages depuis
  Home Assistant.
- Préserver la commande physique d'origine.
- Documenter le matériel, le protocole et les adaptations nécessaires.

Les fonctions réellement disponibles dépendront des commandes prises en
charge par le contrôleur de la VMC.

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

| Connecteur | Sérigraphie | Informations disponibles |
| --- | --- | --- |
| X6 | `RS232 - KFB` | Bornes `12V`, `TX`, `RX`, `GND` |
| X8 | `RS232 - PC` | Connecteur DB9 |

Le connecteur X8 est une première piste pour communiquer avec la VMC.
Le connecteur X6 constitue une autre piste, potentiellement liée à la
commande d'origine.

Le choix définitif sera fait après vérification du brochage, des niveaux
électriques et des fonctions réellement accessibles sur chaque interface.

### Préserver la commande d'origine

La priorité est de ne pas perturber le clavier à membrane.

Les premiers essais devront privilégier l'observation passive des échanges,
sans émission de commandes depuis l'ESP32.

Un raccordement en parallèle ne sera envisagé qu'après avoir confirmé
l'absence de risque électrique et les conditions de partage de la liaison.
Deux équipements qui émettent simultanément sur la même ligne TX peuvent
entrer en conflit.

## Protocole série

La documentation de protocole ComfoAir identifiée comme piste de travail
indique les paramètres suivants :

- Débit : `9600 bauds`
- Bits de données : `8`
- Parité : aucune
- Bits d'arrêt : `1`

Soit **9600 bauds, 8N1**.

Ces paramètres constituent une hypothèse de départ pour les essais. Ils ne
sont pas encore confirmés pour chaque interface du Paul Santos 570 DC.

Documentation consultée :

- [Description du protocole ComfoAir (PDF)](http://www.see-solutions.de/sonstiges/Protokollbeschreibung_ComfoAir.pdf)

La compatibilité de ce protocole avec le modèle Santos devra être vérifiée
par l'analyse des échanges et des réponses réelles de la VMC.

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

## Fonctionnalités envisagées

Les fonctionnalités suivantes sont des objectifs, pas des capacités déjà
validées sur le Paul Santos 570 DC.

### Supervision

Selon les informations accessibles dans le protocole :
- lecture de l'état de fonctionnement ;
- lecture du mode ou du niveau de ventilation ;
- lecture éventuelle des vitesses de ventilateurs ;
- lecture des alarmes ou informations de maintenance disponibles.

### Commande

Si les commandes correspondantes sont identifiées et validées :
- modification du niveau de ventilation ;
- changement de mode ;
- accès aux autres réglages explicitement pris en charge par la VMC.

Aucune commande ne sera publiée dans Home Assistant avant d'avoir été
identifiée et testée de manière fiable.

## Plan de développement

1. Étudier le brochage des connecteurs X6 et X8.
2. Confirmer les niveaux électriques et le rôle de chaque interface.
3. Réaliser un raccordement série adapté, avec MAX3232 si l'interface est
   réellement en RS-232.
4. Vérifier l'alimentation de l'ESP32-C3 et du convertisseur DC-DC.
5. Observer les échanges série sans perturber la commande d'origine.
6. Identifier les trames et les réponses du contrôleur.
7. Adapter le composant ESPHome existant au modèle Santos.
8. Exposer progressivement les capteurs dans Home Assistant.
9. Tester les commandes, si le protocole le permet.
10. Documenter les résultats, les limitations et le câblage validé.

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

- [x] Identification du modèle de VMC.
- [x] Choix de l'ESP32-C3 Super Mini.
- [x] Identification des connecteurs X6 et X8.
- [x] Identification d'un composant ESPHome existant à étudier.
- [x] Choix d'un convertisseur abaisseur DC-DC.
- [ ] Confirmation du brochage et des niveaux électriques.
- [ ] Première capture série passive.
- [ ] Validation du protocole sur le Santos 570 DC.
- [ ] Première intégration ESPHome fonctionnelle.
- [ ] Validation des commandes depuis Home Assistant.

---

Projet personnel de rétro-ingénierie et d'intégration domotique.
La compatibilité, les commandes et les limites de fonctionnement restent
à confirmer sur le matériel réel.
