# Dictionnaire des données — Marketplace des vendeuses de rue

Ce document décrit les **4 tables** de la base de données PostgreSQL et leurs colonnes, à destination des équipes **Base de données (Dylan, Henri)**, **Backend**, **Frontend** et **Carte**.

> **Règle d'équipe :** Les noms exacts des tables et colonnes ci-dessous sont figés pour que le Backend, le schéma SQL (`schema.sql`) et les données de test (`seed.sql`) utilisent exactement les mêmes identifiants.

---

## 1. Vue d'ensemble des livrables de modélisation (`docs/`)

- **MCD (Modèle Conceptuel de Données)** : [mcd.png](./mcd.png)
- **MLD (Modèle Logique de Données)** : [mld.png](./mld.png)
- **UML — Cas d'utilisation** : [uml-cas-utilisation.png](./uml-cas-utilisation.png)
- **UML — Diagramme de classes** : [uml-classes.png](./uml-classes.png)

---

## 2. Description des tables et colonnes

### 2.1 `categories` (Catégories de produits)

Regroupe les types de produits vendus sur la marketplace. Doit être créée **avant** `products`.

| Colonne | Type PostgreSQL | Contraintes | Description |
| :--- | :--- | :--- | :--- |
| `id` | `SERIAL` | `PRIMARY KEY` | Identifiant unique auto-incrémenté de la catégorie. |
| `nom` | `VARCHAR(100)` | `NOT NULL` | Nom lisible de la catégorie (ex. : `Plats cuisinés`, `Artisanat`). |
| `type` | `VARCHAR(20)` | `NOT NULL`, `CHECK (type IN ('nourriture', 'objet'))` | Grand type de catégorie : uniquement `nourriture` ou `objet`. |

---

### 2.2 `vendors` (Vendeuses de rue)

Contient les fiches des vendeuses, leur localisation GPS pour la carte et leur statut de validation. Doit être créée **avant** `products` et `field_notes`.

| Colonne | Type PostgreSQL | Contraintes | Description |
| :--- | :--- | :--- | :--- |
| `id` | `SERIAL` | `PRIMARY KEY` | Identifiant unique auto-incrémenté de la vendeuse. |
| `nom_commercial` | `VARCHAR(150)` | `NOT NULL` | Nom d'activité ou d'étal affiché aux clients (ex. : `Chez Maman Akossiwa`). |
| `quartier` | `VARCHAR(100)` | — | Quartier ou zone géographique où la vendeuse exerce. |
| `latitude` | `NUMERIC(9,6)` | — | Coordonnée GPS de latitude (précision à 6 décimales pour la carte). |
| `longitude` | `NUMERIC(9,6)` | — | Coordonnée GPS de longitude (précision à 6 décimales pour la carte). |
| `horaires` | `VARCHAR(100)` | — | Plage horaire habituelle d'ouverture (ex. : `7h-14h`). |
| `telephone` | `VARCHAR(30)` | — | Numéro de contact de la vendeuse. |
| `statut_validation` | `VARCHAR(20)` | `NOT NULL`, `DEFAULT 'en_attente'` | État de modération de la fiche (`en_attente` par défaut, `valide` après validation par l'administrateur). |

---

### 2.3 `products` (Produits proposés)

Liste les produits associés à une vendeuse et à une catégorie. Dépend de `vendors` et `categories`.

| Colonne | Type PostgreSQL | Contraintes | Description |
| :--- | :--- | :--- | :--- |
| `id` | `SERIAL` | `PRIMARY KEY` | Identifiant unique auto-incrémenté du produit. |
| `vendor_id` | `INTEGER` | `NOT NULL`, `REFERENCES vendors(id)` | Clé étrangère vers `vendors(id)` : vendeuse qui propose ce produit. |
| `category_id` | `INTEGER` | `NOT NULL`, `REFERENCES categories(id)` | Clé étrangère vers `categories(id)` : catégorie du produit. |
| `nom` | `VARCHAR(150)` | `NOT NULL` | Nom du produit (ex. : `Akassa + sauce poisson`, `Panier tressé`). |
| `prix_indicatif` | `NUMERIC(10,0)` | — | Prix indicatif affiché (entier, ex. en FCFA). |

---

### 2.4 `field_notes` (Notes de terrain)

Trace les visites de l'équipe terrain auprès des vendeuses (consentement, date, remarques). Dépend de `vendors`.

| Colonne | Type PostgreSQL | Contraintes | Description |
| :--- | :--- | :--- | :--- |
| `id` | `SERIAL` | `PRIMARY KEY` | Identifiant unique auto-incrémenté de la note de terrain. |
| `vendor_id` | `INTEGER` | `NOT NULL`, `REFERENCES vendors(id)` | Clé étrangère vers `vendors(id)` : vendeuse concernée par la visite. |
| `date_collecte` | `DATE` | — | Date de la collecte d'information sur le terrain. |
| `collecteur` | `VARCHAR(100)` | — | Nom ou identifiant du membre de l'équipe terrain. |
| `consentement` | `BOOLEAN` | `NOT NULL` | Indique si la vendeuse a donné son accord (`TRUE` / `FALSE`) pour figurer sur la plateforme. |
| `remarques` | `TEXT` | — | Observations libres relevées lors de la visite. |

---

## 3. Relations et cardinalités résumées

| Relation | Côté « 1 » | Côté « N » | Clé étrangère |
| :--- | :--- | :--- | :--- |
| **proposer** | `vendors` (`0,n`) | `products` (`1,1`) | `products.vendor_id → vendors.id` |
| **appartenir** | `categories` (`0,n`) | `products` (`1,1`) | `products.category_id → categories.id` |
| **concerner** | `vendors` (`0,n`) | `field_notes` (`1,1`) | `field_notes.vendor_id → vendors.id` |
