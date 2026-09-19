# TCVR Analog Locomotion — analyse technique

## Objectif

Restaurer dans **CyberpunkVRPort 0.1.6** la locomotion analogique progressive du stick gauche observée dans **CPVR 0.1.1**, sans modifier, remplacer ou repacker les fichiers de CPVR.

Le module final doit rester un addon TCVR autonome.

---

## 1. Symptôme observé

Sur CPVR 0.1.1, la vitesse de déplacement variait avec la magnitude du stick gauche :

- petite déflexion : marche lente ;
- déflexion moyenne : vitesse intermédiaire ;
- grande déflexion : course.

À partir de la génération 0.1.2/0.1.3, la sensation devient essentiellement discrète : une fois la deadzone franchie, la vitesse n'est plus proportionnelle de la même manière à la course du stick.

Ce comportement est distinct d'un éventuel problème matériel de clic L3. Le module Analog Locomotion ne traite volontairement **que la magnitude de locomotion**.

---

## 2. Comparaison des versions

L'enquête a utilisé :

- CPVR 0.1.1 comme référence historique du comportement analogique ;
- CPVR 0.1.5 pour examiner le comportement plus récent ;
- le code de release CPVR 0.1.6 pour vérifier que le mécanisme existe encore dans la version actuelle.

Le changement de philosophie apparaît dans l'historique upstream de CPVR 0.1.2 : la locomotion a été volontairement transformée en vitesse discrète.

Références upstream :

- repository : https://github.com/dariulone/cyberpunk-vr-port
- release/commit CPVR 0.1.6 : https://github.com/dariulone/cyberpunk-vr-port/commit/b4a74461b8f7964ad5231113075da44890117523
- fichier concerné : `src/Hooks/XInput.cpp`

---

## 3. Le point de contrôle trouvé

Le code XInput de CPVR exporte une variable globale :

```cpp
extern "C" __declspec(dllexport) int32_t CyberpunkVR_MoveTiers = 1;
```

La logique du hook calcule d'abord la magnitude réelle du stick gauche.

Lorsque `CyberpunkVR_MoveTiers != 0`, CPVR remplace ensuite cette magnitude par une magnitude de sortie choisie par son système discret.

Lorsque la valeur vaut `0`, ce bloc n'est pas appliqué : la magnitude analogique calculée à partir du stick reste disponible pour la sortie XInput.

Cette variable est toujours présente dans le code de release **CPVR 0.1.6**.

---

## 4. Pourquoi ne pas patcher CPVR directement

Une première solution possible aurait été de modifier la DLL CPVR ou de compiler une branche custom.

Cette solution a été volontairement rejetée.

Objectifs TCVR :

1. **zéro fichier CPVR remplacé** ;
2. conserver la possibilité de mettre à jour CPVR indépendamment ;
3. limiter le risque de régression sur les armes, le sprint, les véhicules, VRIK et les autres contrôles ;
4. pouvoir désinstaller le correctif en supprimant un seul dossier ;
5. éviter d'ajouter un deuxième système complet de locomotion ou un deuxième hook XInput.

---

## 5. Architecture retenue

`TCVR_AnalogLocomotion.dll` est un **plugin RED4ext séparé**.

Il ne remplace pas :

- la DLL principale de CyberpunkVRPort ;
- les scripts CPVR ;
- les fichiers XInput de CPVR ;
- le fichier de configuration CPVR.

Son travail est volontairement limité.

### Au chargement

Le plugin recherche le module CyberpunkVRPort déjà chargé dans le processus.

Il localise l'export :

```text
CyberpunkVR_MoveTiers
```

puis écrit :

```text
0
```

dans cette variable.

Le plugin conserve ensuite cette valeur à `0` afin que le mode discret ne soit pas réactivé par un état ultérieur.

### Pourquoi utiliser l'export

C'est plus propre qu'un patch par offset :

- aucun RVA CPVR n'est codé en dur ;
- aucune signature machine spécifique à un build n'est nécessaire ;
- on utilise une interface que CPVR expose déjà dans sa table d'exports ;
- le changement est minuscule et réversible ;
- l'addon n'a pas besoin de dupliquer le hook XInput CPVR.

---

## 6. Ce que CPVR continue de contrôler

Le module TCVR ne remplace pas le pipeline CPVR.

CPVR continue de gérer :

- la récupération OpenXR du stick ;
- la deadzone existante ;
- la direction de locomotion ;
- les modes de direction HMD/main/game ;
- le sprint ;
- le crouch et les gestes de stick ;
- les véhicules ;
- tous les autres boutons et mappings.

TCVR ne change que l'activation du bloc **MoveTiers**.

---

## 7. Compatibilité CPVR 0.1.6

La release 0.1.6 conserve :

```cpp
CyberpunkVR_MoveTiers = 1
```

et le même principe de normalisation de magnitude dans `XInput.cpp`.

Cela rend l'override compatible avec la 0.1.6 sans devoir revenir à la DLL 0.1.1 et sans supprimer les améliorations ajoutées entre 0.1.1 et 0.1.6.

---

## 8. Ce que le module ne fait volontairement pas

La v0.1.1 n'ajoute pas :

- de deadzone custom ;
- de courbe exponentielle ;
- de seuil de sprint custom ;
- de remapping L3 ;
- de correction hardware de joystick ;
- de nouveau menu ;
- de second hook XInput.

Le but du premier build est de tester **une seule hypothèse à la fois** : restaurer la magnitude analogique.

---

## 9. Package

Package :

`TCVR_CPVR_0.1.6_ANALOG_LOCOMOTION_V0_1_1_GAME_ROOT.zip`

SHA-256 :

```text
0330c52449870563279b196da01e71bb22195e86e5578ed1192bb1bf04e101fb
```

Contenu :

```text
red4ext/
└── plugins/
    └── TCVR_AnalogLocomotion/
        ├── TCVR_AnalogLocomotion.dll
        └── README_TCVR_ANALOG.txt
```

---

## 10. Procédure de validation

Test de base à pied :

1. stick ~25 % ;
2. stick ~50 % ;
3. stick ~75 % ;
4. stick presque 100 %.

Résultat recherché :

la vitesse de V doit monter progressivement avec la déflexion, au lieu de passer immédiatement au niveau de déplacement discret dès que le stick dépasse la deadzone.

Le sprint et les autres actions CPVR doivent rester inchangés.

---

## 11. Philosophie pour les prochaines versions

Cette branche doit rester minimaliste.

Si des réglages supplémentaires deviennent nécessaires, ils devront être ajoutés indépendamment, par exemple :

- deadzone analogique TCVR ;
- courbe de réponse ;
- multiplicateur de magnitude ;
- seuil/hystérésis optionnels.

Mais aucune de ces fonctions ne doit être confondue avec la restauration analogique de base.
