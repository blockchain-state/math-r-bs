# PROTOCOLE BS — FONDEMENT v0.1

> Couche opérationnelle du fondement. Ni ontologie (elle vit dans `BS_SEED_DOCUMENT_v0.3.md`), ni spécification technique (elle vit dans `bcs_core_agent_spec.md`). Il s'agit du **contour minimal** que tout premier participant, investisseur ou expert subventionnaire doit comprendre de manière identique.

La version canonique est l'anglaise (`BS_PROTOCOL_FOUNDATION_v0.1.md`). Le présent texte en est un récit littéraire en français. En cas de divergence, l'anglais prévaut.

---

## 1. Définition opératoire

Le **Protocole BS / ARQ MVP** est un protocole minimal de collaboration dans lequel les participants produisent eux-mêmes les questions, les décisions, les traces et les règles d'interaction, tandis que le système les aide à acheminer le chemin qui mène d'un problème encore flou jusqu'à un résultat consigné.

Dit autrement, à destination d'un lecteur extérieur : un protocole guidé par intelligence artificielle pour la collaboration adaptative. Il aide de petits groupes à formuler des problèmes, à répartir leur attention, à consigner leurs décisions et à adapter leurs propres règles de travail commun à mesure que la complexité s'accroît.

Cette formulation est la langue extérieure du projet. Elle est destinée à Innosuisse, à la page publique, à toute première rencontre. Tout discours plus élaboré relève de l'annexe.

## 2. L'itinéraire pratique

Un cycle complet de collaboration se déploie en huit moments :

```
Problème → Question → Trace → Signal → Coordination → Action → Résultat → Mise à jour de la règle
```

| Étape | Ce qui se passe | Qui fait avancer | Où cela vit |
|---|---|---|---|
| **Problème** | Une obscurité intérieure | Tout humain | Avant le système |
| **Question** | L'obscurité devient un Q formulé | ARQ assiste | Bot ARQ |
| **Trace** | Le Q et ses premiers mouvements sont consignés | ARQ et l'auteur | Stockage persistant |
| **Signal** | La Trace devient un signal pour d'autres | Le système route | ARQ + canal de la cellule |
| **Coordination** | Plusieurs participants reprennent, répartissent leur attention | Cellule (3 à 5 personnes) | ARQ + kChat |
| **Action** | Exécution de ce qui a été décidé | Cellule | Monde réel |
| **Résultat** | Un changement consigné | Cellule | Mise à jour de la Trace |
| **Mise à jour de la règle** | Si une correction procédurale est apparue nécessaire, elle est introduite | Cellule via fork BS | Fondement v.SUIVANTE |

Le MVP doit démontrer **au moins un cycle complet** sur un groupe de trois à cinq.

## 3. Les trois invariants structurels

Ces trois règles ne peuvent être forkées. En leur absence, le protocole s'effondre.

**1. Question ≠ tâche.** Un Q est une tension formulée, non une demande de travail. Sans cela, le Q se dégrade en ticket Jira et le protocole devient un gestionnaire de tâches.

**2. L'étincelle (Spark) doit porter un coût-d'erreur.** Toute proposition de direction de résolution est tenue d'énoncer explicitement « ce qui deviendra visible si cette hypothèse est fausse ». Sans cela, la Trace accumule des opinions au lieu de mouvements vérifiables.

**3. L'initiateur n'est pas l'assembleur.** La personne qui a formulé un Q ne peut être celle qui clôt sa Trace. Il s'agit d'un invariant structurel d'anti-autovalidation. Il empêche le cycle de s'effondrer sur une seule personne, quelle que soit la taille de la cellule.

Tout le reste — fenêtres temporelles, formats, rôles, échéances, organisation du stockage — est déterminé par la cellule elle-même et évolue par l'étape **Mise à jour de la règle** (voir §2).

## 4. Le principe d'auto-amendement

Une cellule applique le triptyque Q/S/T à sa propre procédure de collaboration. Lorsqu'une chose se brise ou fonctionne mal, cela devient un Q, traverse le cycle, et la Mise à jour de la règle modifie la procédure pour les cycles suivants.

**Ce qui évolue :** les fenêtres de collecte, les formats de Trace, la composition des rôles, les méthodes de routage, les critères de clôture.

**Ce qui n'évolue pas :** les trois invariants du §3. Ils protègent le fait même que l'évolution demeure protocolaire et ne se dissolve pas dans l'arbitraire.

## 5. Où chaque chose réside

| Couche | Artefact | Fonction |
|---|---|---|
| **Constitutionnelle** | ce document + `BS_SEED_DOCUMENT_v0.3.md` | Invariants, philosophie |
| **Exécution** | Bot ARQ Live (voir `bcs_core_agent_spec.md` et son addendum) | Algorithme de routage, exécution du protocole |
| **Surface** | mathr.ch | Explication de ce qu'est le projet, pour qui, comment y entrer |
| **Conventions de la cellule** | Chaque cellule tient son propre `CELL_LOG.md` | Règles locales, évoluant par Mise à jour |

Le bot ARQ Live n'est pas « une interface conversationnelle ». Il est **le conducteur qui mène de l'obscurité intérieure à l'action coordonnée.**

## 6. Vocabulaire : intérieur ↔ extérieur

Les mots dans lesquels nous parlons à l'intérieur ne fonctionnent pas toujours à l'extérieur. Cette table est contraignante.

| Intérieur | Extérieur (Innosuisse, page publique, premier contact) |
|---|---|
| Стая (essaim) | Cellule de protocole / groupe de collaboration guidée |
| Нода (nœud) | Participant / contributeur |
| Trace | Cycle de décision consigné |
| Spark | Direction proposée |
| Élévation guidée du protocole | Intégration au protocole guidée par IA pour collaboration adaptative |
| Protocole BS | Protocole de collaboration adaptative (dans les contextes techniques) |
| Forker la règle | Mettre à jour les conventions de la cellule |
| Свадхарма / vocation | Alignement de vocation (dans les textes destinés aux milieux académiques) |

Règle : préserver le vocabulaire vivant à l'intérieur, traduire en termes architecturaux et méthodologiques à l'extérieur. Ne jamais réduire à « encore une DAO » ou à « un outil IA ».

## 7. Phase 0 — Ce qui doit être prouvé

Non pas « que le protocole fonctionne à l'échelle ». Seulement trois choses :

1. **Itinéraire mono-utilisateur.** Une personne traverse ARQ depuis le flou jusqu'à un Q précis et une Trace close.
2. **Flux du petit groupe.** Un groupe de trois à cinq personnes accomplit un cycle complet du §2 sur une tâche réelle.
3. **Mise à jour de règle observable.** Une cellule détecte une lacune procédurale, applique Q/S/T à la procédure elle-même, met à jour son `CELL_LOG.md`.

Lorsque ces trois démonstrations existent, nous pouvons parler avec Innosuisse, les premiers participants et les auditoires sérieux non comme d'une utopie, mais comme d'un mécanisme.

## 8. Hors champ pour v0.1

- La mathématique du Trace Score
- Le routage inter-cellules
- L'ancrage blockchain, les jetons, les mécanismes anti-Sybil
- La gouvernance entre cellules
- L'architecture technique d'ARQ (elle vit dans `bcs_core_agent_spec.md` et son addendum)

Ces couches s'ouvriront après que la Phase 0 §7 sera close. Pas avant.

---

*v0.1 — 26 mai 2026. Ouvert au fork selon les règles du §2 étape 8 et §4.*
