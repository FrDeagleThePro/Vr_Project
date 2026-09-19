# TCVR — Cyberpunk 2077 VR Mods

Dépôt public dédié aux modules **TCVR** développés autour de **Cyberpunk 2077 VR / CyberpunkVRPort (CPVR)**.

Ce dépôt est séparé de ToolsCityVR afin de garder les expérimentations et correctifs Cyberpunk VR indépendants du projet principal.

## Module actuel

### TCVR Analog Locomotion

**Version :** v0.1.1  
**Cible :** CyberpunkVRPort 0.1.6  
**But :** restaurer une locomotion au stick gauche réellement analogique et progressive, comme dans CPVR 0.1.1.

Depuis CPVR 0.1.2, la locomotion sur stick est passée à une vitesse discrète : une fois la deadzone dépassée, le stick sert principalement à donner la direction, tandis que la magnitude est normalisée vers une vitesse fixe.

L'analyse de la branche CPVR montre que ce comportement est contrôlé par l'export natif :

```cpp
CyberpunkVR_MoveTiers
```

Dans CPVR 0.1.6, sa valeur par défaut est toujours :

```cpp
CyberpunkVR_MoveTiers = 1
```

- `1` : déplacement discret après la deadzone.
- `0` : la magnitude analogique du stick est conservée.

TCVR Analog Locomotion est donc volontairement minimal : il ne réimplémente pas tout XInput et ne remplace pas le système de locomotion CPVR. Il charge comme **plugin RED4ext séparé**, retrouve l'export `CyberpunkVR_MoveTiers` du module CPVR déjà chargé, et le maintient à `0`.

## Ce que cela change

- faible inclinaison du stick gauche → déplacement lent ;
- inclinaison moyenne → vitesse intermédiaire ;
- forte inclinaison → déplacement plus rapide ;
- la progression redevient continue au lieu d'être immédiatement normalisée.

## Ce que cela ne change PAS

- le sprint de CPVR reste géré par CPVR ;
- les véhicules restent gérés par CPVR ;
- le clic physique L3 n'est pas remappé ;
- aucun correctif de joystick matériel n'est appliqué ;
- les mappings TCVR existants, notamment Y + stick droit, ne sont pas touchés ;
- aucun fichier CPVR n'est remplacé.

## Installation

Télécharger le ZIP depuis le dossier `releases/` et l'extraire directement dans la racine de Cyberpunk 2077.

Le ZIP ajoute seulement :

```text
red4ext/
└── plugins/
    └── TCVR_AnalogLocomotion/
        ├── TCVR_AnalogLocomotion.dll
        └── README_TCVR_ANALOG.txt
```

Aucun fichier existant de CyberpunkVRPort n'est écrasé.

## Test recommandé

À pied, sans véhicule :

1. pousser le stick gauche à environ 25 % ;
2. passer à environ 50 % ;
3. passer à environ 75 % ;
4. pousser presque à 100 %.

La vitesse doit augmenter progressivement avec la course du stick.

## Désinstallation

Supprimer uniquement :

```text
red4ext/plugins/TCVR_AnalogLocomotion/
```

## Analyse technique

Voir [docs/ANALOG_LOCOMOTION_ANALYSIS.md](docs/ANALOG_LOCOMOTION_ANALYSIS.md).

## Référence upstream

CyberpunkVRPort :
https://github.com/dariulone/cyberpunk-vr-port

Le changement de philosophie de locomotion apparaît à partir de CPVR 0.1.2. Le code de CPVR 0.1.6 conserve encore l'export `CyberpunkVR_MoveTiers`, ce qui permet à ce module de restaurer proprement la magnitude analogique sans patcher la DLL CPVR sur disque.

---

TCVR est un projet indépendant. Cyberpunk 2077 et les marques associées appartiennent à leurs ayants droit respectifs.
