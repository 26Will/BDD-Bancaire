# 🏦 Conception et implémentation d'une base de données bancaire (SGBD)

SAE « SGBD » — Série 154 — BUT Science des Données, IUT de Paris (2025–2026)

Conception, alimentation et fiabilisation d'une base de données relationnelle sous **Microsoft Access**, à partir de fichiers sources bruts d'un établissement bancaire (effectifs et volumes d'opérations), avec une analyse métier visant un objectif de gain de productivité de 3 %.

**Auteurs :** William Houblon-Tchagam, Florent Danlos, Nathan Sokol

---

## 🎯 Objectifs

- Concevoir un **Modèle Conceptuel des Données (MCD)** puis un **Modèle Logique (MLD)** à partir de fichiers sources bancaires (`EFF 154`, `VOL 154`)
- Implémenter le schéma relationnel sous Access (tables, clés primaires/étrangères, types de données)
- Importer et **nettoyer ~120 000 lignes** de données réelles, en détectant et corrigeant les anomalies
- Répondre à une problématique métier : proposer une **affectation d'effectifs** en visant un gain de productivité de 3 %

## 🗂️ Modèle de données

Le schéma s'articule autour de :

- **ETABLISSEMENT** — chaque agence bancaire (CC, CCCP ou EFERM), avec code, libellé, type d'entité et statut de fermeture
- **GROUPEMENT / ZONE / REGION** — hiérarchie géographique (rattachement à une Direction Commerciale)
- **CATEGORIE_OPERATION / ACTIVITE / FAMILLE_PRODUIT / GAMME_PRODUIT** — hiérarchie métier des opérations bancaires
- **TEMPS** — référentiel temporel (Période, Mois), distinguant année en cours (`REAL_A`) et année précédente (`REAL_A-1`)
- **CONCERNE_EFF / CONCERNE_VOL** — tables d'association portant respectivement les effectifs et les volumes d'opérations mensuels

Plus de **20 tables**, reliées par clés primaires/étrangères (ex. `ETABLISSEMENT.Code_Groupement → GROUPEMENT.Code_Groupement`).

## 🧹 Import et validation des données

- Import des référentiels puis des tables de faits depuis les fichiers sources
- **5 requêtes de contrôle** (Ano1 à Ano5) pour détecter les anomalies : établissements fermés (EFERM) avec volumes/effectifs résiduels, établissements sans effectif, codes hors organisation, incohérences effectif/volume
- Corrections notables :
  - Restitution du **zéro de tête manquant** sur `Code_Etablissement` (`Format([CODE ETABLISSEMENT], "000000")`)
  - Ajout de **47 établissements EFERM** manquants, entraînant un rechargement complet des tables de faits
  - Contournement des limitations d'Access sur les JOIN avec expression via des **requêtes Ajout avec sous-requête**
- Résultat final : `CONCERNE_VOL` = 57 208 lignes, `CONCERNE_EFF` = 62 292 lignes — écart nul avec les fichiers sources après validation

## 📊 Analyse métier — objectif de productivité 3 %

Méthode :

1. Moyenne mensuelle des volumes et des effectifs par Zone et Catégorie d'opération (année courante vs année précédente)
2. Évolution de l'activité : `Evol_Activite = Vol_Moyen_A / Vol_Moyen_A-1`
3. Effectif théorique attendu : `Eff_Theorique = (Eff_A-1 × Evol_Activite) / 1,03`
4. Écart = Effectif actuel − Effectif théorique

**Résultat clé :** sur 24 combinaisons Zone/Catégorie présentant un écart significatif, 19 révèlent un **déficit d'effectifs** (besoin de −58,88 ETP) contre seulement 5 excédents (+9,51 ETP) — soit un sous-effectif global de ~49 ETP, contredisant l'hypothèse initiale de marges de réduction.

## 🛠️ Technologies

- **Microsoft Access** (conception de schéma, requêtes SQL, formulaires)
- SQL avancé sous Access : sous-requêtes, `INNER JOIN`, requêtes Ajout, fonctions de formatage

## 📁 Structure du dépôt

```
├── data/                 # Fichiers sources (EFF 154, VOL 154)
├── bdd/                   # Base de données Access (.accdb)
├── rapport/               # Rapport de la SAE (.docx / .pdf)
└── README.md
```

## 📄 Livrables

- Rapport détaillé (MCD, MLD, dictionnaire des données, démarche de validation, analyse métier)
- Base de données Access fonctionnelle

---

*Projet réalisé dans le cadre de la SAE « SGBD » (Série 154) — BUT Science des Données, IUT de Paris – Rives de Seine, année 2025–2026.*
