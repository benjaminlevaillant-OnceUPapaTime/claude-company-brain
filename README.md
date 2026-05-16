---
title: "Donner une mémoire permanente à Claude — Architecture & Guide pratique"
description: "Un système complet pour construire une mémoire persistante pour Claude AI dans votre entreprise : Journal (court terme), Company Brain (long terme) et gouvernance. Basé sur Notion, MCP et Git."
lang: fr
author: Benjamin Levaillant
tags: [claude, anthropic, llm, notion, knowledge-management, ai-memory, company-brain, mcp, memory-architecture]
license: CC-BY-4.0
---

# 🧠 Donner une mémoire permanente à Claude — Architecture & Guide pratique

> **Problème** : Claude oublie tout entre deux conversations.
> **Solution** : un système de mémoire en trois couches, basé sur Notion, géré par des humains, portable vers n'importe quel LLM.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Made for Claude](https://img.shields.io/badge/Made%20for-Claude%20AI-orange)](https://claude.ai)
[![Built with Notion](https://img.shields.io/badge/Built%20with-Notion-black)](https://notion.so)

---

## Table des matières

- [En deux mots](#en-deux-mots)
- [1. Pourquoi faire ça](#1-pourquoi-faire-ça)
- [2. Ce qu'on construit](#2-ce-quon-construit)
  - [Les trois couches de mémoire](#les-trois-couches-de-mémoire)
  - [Couche 1 — Le Journal](#couche-1--le-journal-mémoire-court-terme)
  - [Couche 2 — Le Company Brain](#couche-2--le-company-brain-mémoire-long-terme)
  - [Couche 3 — Le Data Lake](#couche-3--le-data-lake-roadmap)
  - [Comment les couches interagissent](#comment-les-trois-couches-interagissent)
- [3. Gouvernance du Company Brain](#3-gouvernance-du-company-brain)
  - [3.1 Les Champions](#31-les-champions)
  - [3.2 Workflow de soumission](#32-workflow-de-soumission-et-validation)
  - [3.3 Changelog](#33-changelog)
  - [3.4 Brain Check](#34-brain-check)
  - [3.5 Niveaux de sensibilité](#35-niveaux-de-sensibilité)
- [4. Mise en place](#4-comment-on-le-met-en-place)
  - [Pourquoi Notion](#loutil-choisi--notion)
  - [Philosophie de construction](#philosophie-de-construction-du-brain)
  - [Ce qu'on crée dans Notion](#ce-quon-crée-dans-notion)
  - [Template d'instructions Claude](#ce-quon-configure-dans-claude)
  - [Score de fraîcheur](#score-de-fraîcheur)
  - [Conventions de session](#les-conventions-de-session)
  - [Le miroir Git](#le-miroir-git--sauvegarde-et-accès-développeur)
  - [Vue d'ensemble technique](#vue-densemble-technique)
- [5. Limites](#5-les-limites-à-connaître)
- [6. Conformité & Sécurité](#6-conformité--sécurité-des-données)
- [7. Architecture model-agnostic](#7-une-infrastructure-powered-by-claude-pas-claude-dependent)
- [8. Lexique](#8-lexique)
- [Contribuer](#contribuer)
- [Licence](#licence)

---

## En deux mots

Claude oublie tout entre deux conversations — c'est son défaut structurel. Ce système lui construit une mémoire en trois couches :

- Le **Journal** — ce qui se passe dans les projets (court terme)
- Le **Company Brain** — ce que votre entreprise sait (long terme)
- Le **Data Lake** *(roadmap)* — la donnée opérationnelle en temps réel

Le Journal se met à jour via `/sync` en fin de session et `/feed` en cours de session. Le Brain est géré par des Champions désignés, avec un Brain Check mensuel pour garantir sa cohérence.

> **Principe directeur** : toute la valeur est dans Notion — pas dans Claude. Git en est le miroir. Si le modèle change demain, la donnée reste.

---

## 1. Pourquoi faire ça

### Le problème qu'on résout

Claude est un outil puissant — mais par défaut, il souffre d'un défaut structurel : **il oublie tout entre deux conversations**. Chaque personne qui ouvre une nouvelle conversation repart de zéro. Claude ne sait pas ce que son collègue a décidé hier. Il ne connaît pas votre produit, vos clients, vos concurrents, vos conventions internes.

Résultat : on perd du temps à ré-expliquer le contexte à chaque session. On obtient des réponses génériques. Et quand plusieurs personnes utilisent Claude sur un même projet, leurs conversations ne se parlent pas.

### Ce qu'on veut atteindre

**Pour chaque individu** : ne plus jamais avoir à réexpliquer qui on est, ce qu'on fait, comment on travaille.

**Pour chaque équipe** : que le travail produit dans une conversation bénéficie aux suivantes, peu importe qui les conduit.

**Pour l'entreprise** : que les décisions stratégiques, la connaissance produit, la veille concurrentielle — tout ce qui a de la valeur — soit accessible à Claude dans n'importe quelle conversation, par n'importe qui.

---

## 2. Ce qu'on construit

### Les trois couches de mémoire

On ne construit pas un seul système, mais trois, qui servent des besoins différents.

#### Analogie médicale

| Type | Cerveau humain | Notre système |
|---|---|---|
| Court terme | Mémoire de travail — ce qui se passe maintenant | **Journal** — conversations, décisions du jour |
| Long terme | Mémoire consolidée — les savoirs durables | **Company Brain** — ce que l'entreprise sait |
| Temps réel | Les sens — observation directe du monde | **Data Lake** — données live des outils |

#### Analogie informatique

| Type | Ordinateur | Notre système |
|---|---|---|
| Court terme | RAM — actif, rapide, s'efface au redémarrage | **Journal** — par projet, mis à jour par `/sync` |
| Long terme | Disque dur — persistant, stable, toujours là | **Company Brain** — stable, curatée, validée |
| Temps réel | Connexion réseau — interrogeable à la demande | **Data Lake** — CRM, monitoring, produit |

```
┌─────────────────────────────────────────────────────────┐
│              COUCHE 3 — Data Lake (roadmap)             │
│   La donnée vivante de l'entreprise. Temps réel.        │
│   CRM, outils de monitoring, produit SaaS.              │
└─────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────┐
│              COUCHE 2 — Company Brain                   │
│   La mémoire de l'entreprise. Stable, curatée.          │
│   Accessible depuis tous les projets Claude.            │
└─────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────┐
│              COUCHE 1 — Le Journal                      │
│   La mémoire court terme du projet.                     │
│   Vivante, collaborative, spécifique à chaque équipe.   │
└─────────────────────────────────────────────────────────┘
```

### Data froide et data chaude

**La data froide** (Couches 1 et 2) : informations stables, éditées manuellement par des humains. Le Journal et le Company Brain sont de la data froide.

**La data chaude** (Couche 3) : données opérationnelles en temps réel, générées automatiquement par les systèmes. Chiffres, événements, états qui changent en permanence.

---

### Couche 1 — Le Journal *(mémoire court terme)*

La mémoire court terme d'une équipe. Il enregistre ce qui se passe dans les conversations Claude — décisions prises, actions en cours, blocages, sujets qui mûrissent.

**Chaque projet a son propre Journal.** Le Journal Sales ne pollue pas le Journal CSM.

**La prochaine personne qui ouvre Claude dans ce projet hérite de tout.** Elle repart du contexte laissé par ses collègues, pas de zéro.

#### Structure du Journal

| Section | Contenu |
|---|---|
| **TODO** | Prochaines actions à mener |
| **EN COURS** | Ce sur quoi l'équipe travaille activement |
| **EN ATTENTE** | Éléments bloqués en attente d'un déblocage externe |
| **SESSIONS** | Résumés horodatés des conversations passées |
| **SUJETS STABLES** | Sujets matures, décisions consolidées, prêts à migrer vers le Company Brain |

**Attribution par personne** : chaque item est préfixé par `@Prénom`. Quand quelqu'un demande *"c'est quoi ma todo ?"*, Claude filtre ses items puis mentionne les autres — conscience collective sans noyer l'individuel.

Exemple :

```
TODO
- @Alice — Relancer le client X avant 2026-05-20
- @Bob — Préparer la démo Y avant 2026-05-23

EN COURS
- @Bob — Rédaction du proposal Z
- @Alice — Qualification compte W
```

---

### Couche 2 — Le Company Brain *(mémoire long terme)*

La mémoire long terme de votre entreprise. Ce qui ne change pas souvent mais que tout le monde doit savoir.

#### Pages thématiques (exemples)

| Page | Contenu |
|---|---|
| 🏢 L'entreprise | Mission, histoire, valeurs, organisation |
| 📦 Le produit | Fonctionnalités, roadmap, pricing, cas d'usage |
| 🎯 Concurrents | Analyse, positionnement, différenciateurs |
| 👥 Clients & personas | ICP, profils, verbatims, segments |
| 💬 Messaging | Ton de voix, arguments clés, copy validé |
| 📊 Métriques | KPIs, OKRs en cours, benchmarks |

**Chaque projet accède au Brain via une vue filtrée dédiée** dans la base de données Notion. Claude la lit via MCP Notion au démarrage.

**Deux modes de chargement**, gérés par le TAG en tête du titre :
- `Contexte` → chargé **systématiquement** à chaque ouverture de conversation
- `Référence` → chargé **conditionnellement** selon le sujet de la demande

---

### Couche 3 — Le Data Lake *(roadmap)*

La donnée opérationnelle en temps réel. Agrège des données issues des outils de l'entreprise — CRM (HubSpot, Salesforce…), monitoring (Datadog…), produit SaaS.

Claude pourra répondre à des questions comme *"combien d'utilisateurs actifs ce mois-ci ?"* en puisant dans la donnée réelle, pas dans un document rédigé il y a trois mois.

**Cette couche est en cours de construction.**

---

### Comment les trois couches interagissent

Au démarrage de chaque conversation Claude :

1. Claude lit la **vue filtrée du projet** dans la BDD Company Brain via MCP Notion
2. Il charge systématiquement toutes les pages taguées `Contexte`
3. Il charge les pages taguées `Référence` si leur titre indique une pertinence
4. Il charge le **Journal du projet** pour récupérer le contexte récent
5. *(Roadmap)* Il interroge le **Data Lake** pour des données en temps réel

À la fin de la conversation :
- `/sync` → Claude résume et met à jour le Journal
- Le Company Brain n'est touché que par ses Champions
- Le Data Lake est alimenté automatiquement par les systèmes

> **⚠️ Règle de priorité** : en cas de contradiction entre le Journal et le Company Brain sur un sujet opérationnel récent, **le Journal prime**. Il est plus récent. Cette règle est incluse dans le template standard d'instructions (voir section 4).

---

## 3. Gouvernance du Company Brain

### 3.1 Les Champions

Chaque page du Company Brain a un ou plusieurs **Champions** désignés. Ce sont les seuls — avec les admins Notion — à avoir le droit de modifier le contenu d'une page.

Chaque Champion s'engage à :
- Maintenir sa page à jour quand la réalité évolue
- Traiter les demandes de modification soumises par l'équipe
- Documenter chaque modification dans le Changelog

💡 **Automatisations recommandées** (Make, Zapier, N8N…) :
- Notification Slack dès qu'une page dont on est Champion est modifiée dans Notion
- Notification Slack dès qu'une nouvelle demande de modification arrive

---

### 3.2 Workflow de soumission et validation

N'importe quel collaborateur peut proposer une modification. Il soumet une demande dans la database **"Demandes de modification du Brain"**, que le Champion concerné traite.

**Chaque demande contient :**
- La page concernée
- La modification proposée
- La justification (source, contexte, événement déclencheur)

**Escalade automatique** : si la demande reste sans réponse après **3 jours ouvrés**, un scénario automatisé la bascule en statut *Escaladée* et notifie les Architectes.

#### Statuts d'une demande

```
À traiter → À discuter → Acceptée
                      → Refusée
                      → Modifiée (acceptée avec ajustements)
          → [3j ouvrés] → Escaladée → idem
```

| Statut | Signification |
|---|---|
| **À traiter** | Demande soumise, en attente du Champion |
| **À discuter** | Le Champion souhaite échanger avant de trancher |
| **Acceptée** | Modification intégrée telle quelle |
| **Refusée** | Modification non retenue |
| **Modifiée** | Acceptée avec ajustements du Champion |
| **Escaladée** | Non traitée sous 3 jours — transmise aux Architectes |

---

### 3.3 Changelog

Chaque modification est tracée dans un **Changelog central**, accessible à tous en lecture.

| Champ | Contenu |
|---|---|
| Date | Date et heure de la modification |
| Page modifiée | Quelle page du Brain a été touchée |
| Champion | Qui a effectué ou validé la modification |
| Nature | Ajout / Modification / Suppression |
| Description | Résumé court de ce qui a changé |
| Demande liée | Lien vers la demande d'origine |

**Règles :**
- Tout est loggué, y compris les petites corrections
- Seuls les admins Notion peuvent modifier le Changelog
- Tout le monde peut le consulter

---

### 3.4 Brain Check

Vérification périodique de la cohérence et de la fraîcheur du Brain. Déclenché par un **Architecte** via `/brain-check`.

#### Ce que Claude fait pendant un Brain Check

1. **Détection des contradictions** entre pages
2. **Identification des informations potentiellement obsolètes** selon le score de fraîcheur
3. **Recommandations de mise à jour** — Claude propose, l'Architecte décide
4. **Vérification des Fiches Projet** — alignement avec l'état actuel du Brain
5. **Identification des SUJETS STABLES à migrer** depuis les Journaux
6. **Détection des signaux Journal non intégrés** dans le Brain

**Ce que Claude ne fait pas** : il ne tranche pas, ne modifie pas, n'interpelle pas d'autres personnes.

#### Fréquence recommandée

| Phase | Fréquence | Déclencheur additionnel |
|---|---|---|
| **Build** *(mise en place)* | 1 fois par semaine | |
| **Run** *(en production)* | 1 fois par mois | Nouveau produit, changement de pricing, réorganisation… |

---

### 3.5 Niveaux de sensibilité

| Niveau | Emoji | Définition | Comportement de Claude |
|---|---|---|---|
| **Public** | 🟢 | Accessible depuis tous les projets | Chargé normalement selon le TAG |
| **Restreint** | 🟡 | Accessible uniquement depuis les projets listés | Ignoré pour les projets hors liste blanche |
| **Confidentiel** | 🔴 | Jamais chargé automatiquement | Nécessite une demande explicite de l'Architecte |

> Les Fiches Projet sont **toujours 🔴 Confidentiel** par défaut.

---

## 4. Comment on le met en place

### L'outil choisi : Notion

Toutes les mémoires vivent dans un seul workspace Notion partagé avec toute l'équipe.

| Critère | Notion | Google Drive | GitLab |
|---|---|---|---|
| API lecture par Claude | ✅ Simple | ✅ Native | ✅ Simple |
| API écriture par Claude | ✅ Simple | ⚠️ Complexe | ✅ Simple |
| Lisible par l'équipe | ✅ UI soignée | ✅ | ⚠️ Orienté dev |
| Structure en database | ✅ | ❌ | ❌ |
| Scalable sans effort technique | ✅ | ⚠️ | ⚠️ |
| Setup accessible aux non-devs | ✅ ~30 min | ❌ | ❌ |

💡 **Note** : d'autres outils de knowledge management peuvent faire l'affaire — Confluence, Coda, Slab ou Obsidian pour les équipes plus techniques. Le critère décisif est l'existence d'une API permettant à Claude de lire et d'écrire, et d'une structure en base de données pour gérer les métadonnées. L'architecture décrite ici reste transposable à tout outil qui coche ces deux cases.

---

### Philosophie de construction du Brain

> **Un Brain vide dit « je ne sais pas ». Un Brain mal rempli dit quelque chose de faux avec confiance.**

**On ne migre pas. On construit.** Le passif documentaire de la plupart des entreprises est pollué — versions contradictoires, informations périmées. Chaque page est rédigée from scratch par son Champion, à partir de ce qu'il sait être vrai aujourd'hui.

**Ce qu'on ne met pas dans le Brain :**
- ❌ Documents bruts (PDF, decks, exports) — le Brain est rédigé, pas archivé
- ❌ Informations dont on n'est pas sûr
- ❌ Données personnelles ou confidentielles non tagées
- ❌ Passif non relu
- ❌ Informations opérationnelles à très court terme — elles appartiennent au Journal

> **Mieux vaut un Brain petit et fiable qu'un Brain grand et bruité.**

#### Pourquoi pas le RAG ?

| Critère | Notre approche (Notion) | RAG vectoriel |
|---|---|---|
| Lisibilité humaine | ✅ Tout le monde peut corriger | ❌ Vecteurs illisibles |
| Infrastructure requise | ✅ Aucune | ❌ Base vectorielle, pipeline, maintenance |
| Pertinence à petite échelle | ✅ Suffisant jusqu'à ~400 pages | ⚠️ Sous-utilisé |
| Transparence des sources | ✅ Page Notion identifiable | ❌ Boîte noire |

Le RAG reste une évolution envisageable au-delà de **~400 pages actives** dans le Brain, en mode **hybride** : Notion en priorité (stable, curatée), RAG en fallback pour les archives volumineuses.

---

### Ce qu'on crée dans Notion

#### Les colonnes de la base de données Company Brain

| Colonne | Rôle | Impact sur Claude |
|---|---|---|
| **Titre** | TAG + intention explicite | Détermine le comportement de chargement |
| **Projet** | Périmètre de la page | Filtre des vues |
| **Type** | Nature du contenu | Tri Notion uniquement |
| **Statut** | `Actif` / `WIP` / `Archivé` | Seules les pages `Actif` sont chargées |
| **Dernière mise à jour** | Date de dernière modification manuelle | Alimente le score de fraîcheur |
| **Owner** | Champion responsable | Gouvernance |
| **Sensibilité** | 🟢 / 🟡 / 🔴 | Claude adapte la discrétion de son traitement |
| **Freshness tier** | Délai de recertification attendu | Seuil d'alerte par page |

#### Filtre de la vue par projet

```
Projet contient "Général" OU Projet contient "[Nom du projet]"
ET Statut = "Actif"
```

#### Qualité des titres de pages Référence

Claude décide de lire une page `Référence` **sur la base du titre seul**. Les titres doivent être des intentions explicites :

| ✅ Bon | ❌ Mauvais |
|---|---|
| `Référence - Analyse concurrentielle vs. Salesforce, HubSpot` | `Référence - Concurrents` |
| `Référence - Pricing et conditions commerciales détaillées` | `Référence - Commercial` |

---

### Ce qu'on configure dans Claude

**Template d'instructions — à coller dans les paramètres d'un nouveau projet Claude.ai :**

```
Tu es [rôle et expertise — ex. "un expert RevOps pour équipes SaaS B2B"].

Consulte le contenu de toutes les pages de cette vue filtrée :
[URL de la vue filtrée du projet dans la BDD Company Brain]

La vue est déjà filtrée avec ce dont tu as besoin — ne cherche pas 
d'autre chose dans Notion.

Les pages dont le titre commence par "Contexte" doivent être lues 
systématiquement.
Les pages dont le titre commence par "Référence" doivent être lues 
uniquement si leur titre indique qu'elles sont pertinentes pour la 
demande en cours.

Charge également le Journal du projet :
[URL ou ID du Journal du projet]

En fin de conversation, propose un /sync pour mettre à jour le Journal.
En cas de contradiction entre le Journal et le Company Brain sur un sujet
opérationnel récent, le Journal prime.

Une fois le contexte chargé, réponds directement à la demande.
```

---

### Score de fraîcheur

| Type de contenu | Freshness tier suggéré |
|---|---|
| Concurrents | 30 jours |
| Métriques, OKRs | 30 jours |
| Produit, Pricing | 60 jours |
| Clients & Personas | 60 jours |
| Messaging, Ton de voix | 90 jours |
| Entreprise, Valeurs | 180 jours |

**Alertes automatiques :**
- ⚠️ Au-delà du Freshness tier → *"Attention, mes connaissances sur ce sujet datent de plus de [N] jours."*
- 🔴 Au-delà du double → *"Cette information est probablement obsolète."*

---

### Les conventions de session

| Commande | Ce que ça fait |
|---|---|
| `/sync` | Résume la session, met à jour les sections actives, enregistre une entrée horodatée, convertit toutes les dates relatives en dates absolues |
| `/feed [texte]` | Ajoute un item dans la bonne section du Journal, avec routing automatique |
| `/brain-check` | Déclenche un Brain Check (Architectes uniquement) |

#### Ce que fait Claude lors d'un `/sync`

1. Enregistre une entrée dans la base de sessions (date absolue YYYY-MM-DD, auteur, résumé, décisions)
2. Met à jour les sections actives (`@Prénom — description`)
3. Convertit toutes les références temporelles — *"demain"* → *"2026-05-15"*
4. Tous les 10 `/sync` : propose un nettoyage complet du Journal

#### Routing automatique du `/feed`

| Ce que tu dis | Section ciblée |
|---|---|
| "rappelle-moi de X" / "note X pour plus tard" | **TODO** |
| "je vais travailler sur X" | **EN COURS** |
| "je bloque sur X en attendant Y" | **EN ATTENTE** |
| "on a décidé X, c'est stable" | **SUJETS STABLES** |

---

### Le miroir Git — Sauvegarde et accès développeur

Le Company Brain vit dans Notion. Il est synchronisé en continu vers un **repo GitHub privé**, en lecture seule.

> **Règle fondamentale : Notion est le seul endroit où on écrit.** La sync est unidirectionnelle : Notion → Git.

| Contenu | Synchronisé | Raison |
|---|---|---|
| **Company Brain** (pages Actif) | ✅ | Asset stratégique, utile aux devs, stable |
| **Journaux de projet** | ❌ | Trop volatils, données opérationnelles |
| **Fiches Projet — config** | ✅ | Pièce de migration. Tokens → gestionnaire de mots de passe. |

#### Structure du repo

```
company-brain/
├── README.md
├── general/
│   ├── contexte-entreprise.md
│   └── contexte-produit.md
├── sales/
│   └── reference-concurrents.md
└── marketing/
    └── reference-messaging.md
```

Chaque fichier commence par un frontmatter YAML :

```yaml
---
notion_id: 343ec517-...
title: "Contexte — Qu'est-ce que notre produit ?"
tag: Contexte
projet: Général
owner: Alice
last_verified: 2026-03-15
freshness_tier: 180j
sensitivity: Public
status: Actif
---
```

#### Options de synchronisation

**Option A — GitHub Action (recommandée)**
- Cron horaire conditionnel : si une page a été modifiée dans Notion → sync et commit
- Cron nocturne inconditionnel : sync complète à 2h du matin (backup quotidien garanti)

**Option B — Polling court via automatisation**
- Outil d'automatisation (Make, Zapier, N8N…) interrogeant l'API Notion toutes les 5-15 min
- Même mécanisme que l'Option A, mais à fréquence plus élevée

---

### Vue d'ensemble technique

```
Claude (conversation)
        │
        ├── Au démarrage
        │     ├── Lit la vue filtrée du projet (BDD Company Brain via MCP Notion)
        │     ├── Charge toutes les pages taguées "Contexte" (systématique)
        │     ├── Charge les pages taguées "Référence" si pertinent (conditionnel)
        │     └── Charge le Journal du projet (Notion DB)
        │
        ├── Pendant la conversation
        │     └── Travail normal avec tout le contexte chargé
        │
        ├── À la fin (/sync)
        │     ├── Enregistre la session (dates absolues)
        │     ├── Met à jour les sections actives
        │     └── Tous les 10 /sync : propose un nettoyage du Journal
        │
        └── Sur demande (/brain-check — Architectes uniquement)
              └── Analyse le Brain et engage une conversation interactive

[Toutes les heures — GitHub Action]
        │
        ├── Vérifie si une page Brain a été modifiée dans Notion
        │     ├── Oui → exporte en markdown + commit dans le repo git
        │     └── Non → arrêt, rien ne se passe
        └── [2h du matin] Sync forcée inconditionnelle (backup quotidien garanti)
```

---

## 5. Les limites à connaître

### Limites techniques

- **Claude lit, il n'observe pas en temps réel.** Le contexte est chargé au démarrage. Un `/sync` fait par un collègue pendant votre session ne sera visible qu'à la prochaine.
- **La qualité dépend des `/sync`.** C'est une discipline d'équipe, pas une automatisation complète.
- **Notion a une limite de taille par requête.** Si le Journal devient très dense, archiver les entrées anciennes.
- **Le Company Brain ne se met pas à jour tout seul.** Il reflète ce que ses Champions y mettent.

### Limites organisationnelles

- **Ça demande un minimum de rigueur.** Désigner des Champions, lancer les Brain Checks.
- **Le Company Brain peut diverger de la réalité.** Le Brain Check est là pour le détecter.
- **Le système de sensibilité n'est pas infaillible.** Ne pas mettre dans le Brain des informations qu'on ne peut pas assumer de voir circuler même en cas de mauvais tagging.

### Ce que ce système ne remplace pas

- Une base de documentation généraliste pour les specs techniques et archives longue forme
- Un CRM pour l'historique client
- Un outil de gestion de projet pour le suivi des tâches
- La communication synchrone entre collègues

---

## 6. Conformité & Sécurité des données

### RGPD

- **Pas d'entraînement des modèles sur vos données** — sur les offres Claude Teams et Enterprise
- **Les données peuvent rester en Europe** — Anthropic propose une résidence des données en UE sur offres Enterprise
- **Le Brain ne contient pas de données personnelles** — c'est une base de connaissance métier
- **Droit à l'effacement** — toute suppression dans Notion déclenche la suppression du fichier git correspondant

### Référentiels de gouvernance IA

#### NIST AI Risk Management Framework

| Fonction | Ce que l'architecture y apporte |
|---|---|
| **Govern** | Champions, Architecte, Admin Notion — rôles formalisés. Changelog. |
| **Map** | Niveaux de sensibilité, data froide / chaude, section Limites. |
| **Measure** | Score de fraîcheur, Brain Check périodique. |
| **Manage** | Workflow de modification validé, Brain Check, plan de migration model-agnostic. |

#### ISO/IEC 42001

| Exigence | Notre réponse |
|---|---|
| Politique IA documentée | Ce document |
| Responsabilités définies | Champions, Architecte, Admin Notion |
| Traçabilité des décisions | Changelog central |
| Revue périodique | Brain Check mensuel |
| Gestion des risques | Niveaux de sensibilité, limites documentées |

#### EU AI Act

Usage de Claude classé **limited risk** — outil d'assistance interne, sans prise de décision autonome affectant des personnes. Toutes les décisions restent humaines.

---

## 7. Une infrastructure « powered by Claude », pas « Claude-dependent »

**La valeur est dans la donnée, pas dans le modèle.**

Tout ce qu'on construit vit dans Notion. Si Claude disparaissait demain, la connaissance accumulée resterait intégralement dans Notion, lisible et utilisable par l'équipe.

**L'architecture est model-agnostic.**

Les instructions de démarrage sont en langage naturel. Elles pourraient demain être adressées à GPT, Gemini, Mistral ou tout futur LLM avec des adaptations mineures.

**Plan de migration en 4 étapes :**

1. Le **Company Brain est déjà exporté** dans le repo git — aucune action nécessaire
2. Exporter les **Fiches Projet** depuis Notion
3. Adapter les instructions au nouveau modèle
4. Reconnecter au nouveau fournisseur via MCP ou API

La donnée ne bouge pas. La gouvernance ne change pas. Seul le moteur est remplacé.

---

## 8. Lexique

| Terme | Définition |
|---|---|
| **Mémoire** | Terme générique désignant toute source de contexte mise à disposition de Claude au démarrage d'une conversation. Trois types : Journal, Company Brain, Data Lake. |
| **Company Brain** | La mémoire long terme de l'entreprise. Pages thématiques stables dans Notion. Équivalent du disque dur. |
| **Journal** | La mémoire court terme d'un projet. Sections actives (TODO, EN COURS, EN ATTENTE, SUJETS STABLES) + base de sessions. Équivalent de la RAM. |
| **Data Lake** | *(Roadmap)* Troisième couche. Données opérationnelles en temps réel issues des outils de l'entreprise. |
| **Data froide** | Données stables, rédigées manuellement. Couches 1 et 2. |
| **Data chaude** | Données opérationnelles en temps réel. Couche 3. |
| **Champion** | Responsable d'une page du Company Brain. Seul le Champion (et les admins) peut modifier sa page. |
| **Architecte** | Garant de la cohérence globale du Brain. Déclenche les `/brain-check`. |
| **Admin Notion** | Administrateur technique du workspace. Accès en écriture à toutes les pages, y compris le Changelog. |
| **Index** | Vue filtrée dédiée dans la BDD Company Brain, préconfigurée pour chaque projet Claude. |
| **Chargement systématique** | Pages taguées `Contexte`. Chargées à chaque ouverture de conversation. |
| **Chargement conditionnel** | Pages taguées `Référence`. Chargées uniquement si leur titre indique une pertinence. |
| **Changelog** | Journal central de toutes les modifications apportées au Company Brain. |
| **Fiche Projet** | Page Notion 🔴 Confidentiel dédiée à un projet Claude.ai. Contient les instructions et la configuration complète. |
| **RAG** | Retrieval-Augmented Generation. Alternative vectorielle, pertinente au-delà de ~400 pages actives. |
| **Score de fraîcheur** | Indicateur calculé depuis `Dernière mise à jour` et `Freshness tier`. |
| **Sensibilité** | Niveau de confidentialité d'une page : 🟢 Public, 🟡 Restreint, 🔴 Confidentiel. |
| **Brain Check** | Vérification périodique déclenchée par `/brain-check`. |
| `/sync` | Commande de fin de session. Enregistre la session, met à jour les sections actives, convertit les dates en absolu. |
| `/feed` | Commande pour ajouter un item au Journal en cours de session avec routing automatique. |
| **SUJETS STABLES** | Section du Journal réservée aux sujets matures prêts à migrer vers le Company Brain. |
| **@Prénom** | Convention d'attribution dans les sections actives. Permet les réponses individuelles tout en conservant la vue collective. |
| **Miroir Git** | Repo GitHub privé synchronisé depuis Notion (sens unique : Notion → Git). |

---

## Contribuer

Ce guide est un document vivant. Si vous avez mis en place ce système dans votre entreprise et souhaitez partager des retours, des variantes ou des améliorations :

1. Ouvrez une **Issue** pour signaler une erreur ou proposer une amélioration
2. Soumettez une **Pull Request** pour contribuer directement au contenu
3. Partagez vos retours d'expérience en commentaire

---

## Licence

Ce document est publié sous licence **[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)**.

Vous êtes libre de partager, adapter et utiliser ce contenu, y compris commercialement, à condition de créditer l'auteur.

---

*Document vivant — dernière mise à jour : mai 2026*
