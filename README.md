# 💰 Personal Finance Dashboard — POC Architecture

> Un clone simplifié de YNAB / Bankin — conçu comme terrain d'expérimentation architectural.

---

## 🎯 Pourquoi ce projet ?

Les données sont **relationnelles**, les calculs doivent être **temps réel**, et l'affichage doit être **réactif**. C'est l'environnement idéal pour valider des choix d'architecture front-end concrets, proches de ce qu'on fait en production.

---

## 📋 Le Pitch

Une **Single Page Application (SPA)** où l'utilisateur peut :

- ➕ Ajouter des **Comptes** (Courant, Livret A…)
- 💸 Ajouter des **Transactions** (catégories, dates, montants)
- 📊 Consulter un **Dashboard** avec graphiques et solde total mis à jour en temps réel
- 🔍 **Filtrer, trier** et **annuler** la dernière action

---

## 📁 Structure

```
src/
  pages/            # Entrypoints de l'app — un fichier par route
  ui/
    components/     # Boutons, inputs — vraiment génériques
    accounts/       # Domains métiers de l'application
    transactions/
    dashboard/
  domain/           # Types et fonctions pures — zéro React, zéro import externe
  infrastructure/   # Accès à IndexedDB via Dexie
```

### La règle qui ne change pas

`domain/` n'importe rien d'autre. Tout le reste peut dépendre de `domain/`, jamais l'inverse.

---

## 🏗️ Choix Architecturaux

### Persistance — Dexie.js (IndexedDB)

Dexie est la source de vérité unique. Il n'y a pas de store global, pas de Context — **la base de données est le store**.

Le super-pouvoir de Dexie ici : `useLiveQuery`. N'importe quelle écriture en base notifie automatiquement tous les hooks qui observent ces données. Pas d'invalidation manuelle de cache.

```ts
// hooks/useAccounts.ts
export function useAccounts() {
  const accounts = useLiveQuery(() => db.accounts.toArray())
  const transactions = useLiveQuery(() => db.transactions.toArray())

  return accounts?.map(a => ({
    ...a,
    balance: getAccountBalance(a, transactions ?? []) // ← domain/
  }))
}
```

### Flux de données

Chaque hook est autonome et lit directement depuis `infrastructure/`. Le Dashboard ne réutilise pas l'état d'un autre composant — il lit ce dont il a besoin lui-même.

```
infrastructure/ (Dexie)
       ↑
   hooks/              useAccounts()  useTransactions()  useDashboard()
       ↑
   ui/components       reçoit des props — aucune logique
```

Pourquoi pas un Context global ? Un Context qui agrège tout devient un **God Context** : il sait trop de choses, crée du couplage, et force des re-renders inutiles. Chaque hook est responsable de ses données.

### Logique métier — Domain layer

Tout ce qui calcule vit dans `domain/` : fonctions pures, zéro React, 100% testables sans navigateur.

```ts
// domain/calculations.ts
getAccountBalance(account, transactions[]) → number
getTotalBalance(accounts[], transactions[]) → number
getTransactionsByMonth(transactions[])     → Record<string, Transaction[]>
filterTransactions(transactions[], filters) → Transaction[]
```

---

## ❓ Questions tranchées

| Question | Décision |
|---|---|
| State management | Aucun store global — Dexie est la source de vérité |
| Synchro entre hooks | `useLiveQuery` — automatique à chaque écriture |
| Logique de calcul | Fonctions pures dans `domain/`, appelées depuis les hooks |
| Context API | Réservé si besoin d'un état vraiment global (thème, user) |
| Testing | Les fonctions `domain/` se testent en pur Node — sans DOM |

---

## 🚀 Plan d'Attaque

- [ ] Figer les 3 écrans et lister les données affichées (avant d'écrire une ligne)
- [ ] Modéliser les types dans `domain/types.ts`
- [ ] Écrire les fonctions de calcul dans `domain/calculations.ts` + tests
- [ ] Mettre en place Dexie dans `infrastructure/`
- [ ] Générer un jeu de données seed réaliste
- [ ] Implémenter les hooks dans `hooks/`
- [ ] Brancher les composants UI
- [ ] Filtres, tri, annulation de la dernière action

---

## 💡 Pourquoi ce POC durera longtemps

> Simple à définir, **infini à perfectionner**.

La structure est conçue pour accueillir de nouveaux patterns sans réécriture :
- Remplacer Dexie par une vraie API (sans toucher aux hooks ni aux composants)
- Ajouter React Query si un backend apparaît
- Tester les React Signals comme alternative aux hooks de lecture
- Introduire du Windowing sur la liste des transactions

---

*Ce repo est un **sandbox d'architecture**. La feature complète est un prétexte — l'objectif est les décisions d'ingénierie.*