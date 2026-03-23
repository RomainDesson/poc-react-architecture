# 💰 Personal Finance Dashboard — POC Architecture

> Un clone simplifié de YNAB / Bankin — conçu comme terrain d'expérimentation architectural.

---

## 🎯 Pourquoi ce projet ?

Les données sont **relationnelles**, les calculs doivent être **temps réel**, et l'affichage doit être **réactif**. C'est l'environnement idéal pour tester et valider des choix d'architecture front-end non triviaux.

---

## 📋 Le Pitch

Une **Single Page Application (SPA)** où l'utilisateur peut :

- ➕ Ajouter des **Comptes** (Courant, Livret A…)
- 💸 Ajouter des **Transactions** (catégories, dates, montants)
- 📊 Consulter un **Dashboard** avec graphiques et solde total mis à jour en temps réel
- 🔍 **Filtrer, trier** et **annuler** la dernière action

---

## 🏗️ Les Défis Architecturaux

> C'est là que le projet devient "profond". Coder ça "comme un junior" (tout dans un gros state React, des `useEffect` imbriqués) donne une app qui lag dès 100 transactions. Voici les choix à trancher.

### Défi A — Gestion d'état (State Management)

**Le problème :** Quand une transaction est modifiée, il faut mettre à jour simultanément le solde du compte, le budget de la catégorie, et le graphique du dashboard — trois endroits distincts.

**Le choix architectural :**
| Option | Approche |
|---|---|
| `Redux` | Classique, verbeux, éprouvé |
| `Zustand` | Moderne, léger, pragmatique |
| `RxJS` | Event-Driven, puissant, complexe |
| `useReducer` (React pur) | Minimaliste, sans dépendance |

---

### Défi B — Séparation logique métier / UI (Separation of Concerns)

**Le problème :** Le calcul du solde ne doit **pas** vivre dans le composant `<BankCard />`.

**Le choix architectural :** Créer une couche **Domain Layer** explicite.
- Les composants UI n'affichent que des données (dumb components)
- Un `AccountService` ou des hooks personnalisés avancés portent la logique
- Objectif : **tester la logique de calcul sans lancer le navigateur**

---

### Défi C — Performance et Listes Virtuelles

**Le problème :** 5 000 transactions sur 3 ans = DOM explosé si on affiche tout.

**Le choix architectural :**
- Implémenter du **Windowing** (n'afficher que les ~20 éléments visibles)
- Ou de la **pagination intelligente côté client**
- Force à penser l'architecture des données dès le départ

---

### Défi D — Persistance et Mode Offline

**Le problème :** C'est un POC, mais il faut que ça fonctionne.

**Le choix architectural :**
| Option | Trade-off |
|---|---|
| `localStorage` (brut) | Simple, limité (~5MB), synchrone |
| `IndexedDB` via `Dexie.js` | Robuste, asynchrone, scalable |

---

## 📁 Structure de Dossiers (Clean Architecture)

```
/src
  /domain           # Types, interfaces, règles de calcul — pur TS, zéro React
  /infrastructure   # Couche stockage (LocalStorage, API mock, IndexedDB)
  /application      # Services d'orchestration, State management
  /ui               # Composants React "bêtes" — reçoivent des props, c'est tout
```

> **Règle d'or :** `/domain` et `/application` ne doivent jamais importer quoi que ce soit de `/ui`.

---

## ❓ Questions Architecturales à Trancher

Ces décisions constituent le vrai **cahier des charges** du POC :

1. **Unidirectional Data Flow**
   Comment garantir un état prévisible ? → Pattern Flux, Redux DevTools, immer…

2. **Dependency Injection**
   Comment injecter le service de stockage sans coupler les composants ? → Context API vs hooks personnalisés

3. **Testing Strategy**
   Comment tester que `+50€ sur une transaction` → `+50€ sur le solde du compte`, sans monter le DOM ?

---

## 🚀 Plan d'Attaque

> **Ne pas chercher à faire joli.** CSS minimal. L'objectif est la **structure des dossiers** et la **clarté des flux de données**.

- [ ] Modéliser le domaine (`Account`, `Transaction`, `Category`) en TypeScript pur
- [ ] Implémenter la couche `infrastructure` (persistence)  
- [ ] Choisir et câbler la solution de State Management
- [ ] Brancher les composants UI sur l'état applicatif
- [ ] Ajouter les cas limites : annulation, filtres, tri
- [ ] Mesurer les perfs et introduire le Windowing si nécessaire

---

## 💡 Pourquoi ce POC durera longtemps

> Simple à définir, **infini à perfectionner**.

Ce projet peut évoluer pendant des années pour tester de nouveaux patterns :
- React Signals
- Server Components
- Micro-frontends
- Observable stores
- …

---

*Ce repo est un **sandbox d'architecture**. La feature complète est un prétexte — l'objectif est les décisions d'ingénierie.*