# ENSTA — Parcours Ingénieur AI & SS — 3ème Année
## Bases de Données Avancées — Chapitre 2 : Bases de Données Distribuées

---

# TP 2.1 – Architectures Distribuées et Fragmentation des Données

> **Durée estimée :** 2h30 à 3h
> **Mode :** Binôme ou individuel
> **Rendu :** Rapport écrit + schémas annotés
> **Date limite :** _(à compléter par l'enseignant)_

---

## 1. Objectifs Pédagogiques

À l'issue de ce TP, l'étudiant sera capable de :

1. **Analyser et comparer** les architectures Shared-Memory, Shared-Disk et Shared-Nothing en justifiant le choix architectural selon les contraintes métier.
2. **Concevoir** un schéma de fragmentation horizontal, vertical et hybride adapté à une application distribuée réelle.
3. **Évaluer** les stratégies de réplication (complète, partielle, sans réplication) et leurs impacts sur la disponibilité et les performances.
4. **Appliquer** les critères d'allocation des fragments sur des nœuds physiques en minimisant les coûts de communication.
5. **Identifier** les risques de redondance, d'incohérence et les pièges de conception dans un système de base de données distribuée.

---

## 2. Prérequis et Matériel

### 2.1 Prérequis

- Modèle Relationnel (clés primaires, étrangères, normalisation 3NF)
- Algèbre relationnelle (σ, π, ⋈, ∪, -)
- Notions de réseau (latence, bande passante, topologie)
- Notions de base sur les systèmes distribués (nœud, message, défaillance)

### 2.2 Contexte du TP — Plateforme IA de Recommandation E-Commerce Mondiale : *ShopMind*

*ShopMind* est une plateforme e-commerce mondiale qui intègre un moteur de recommandation basé sur l'IA. La base de données doit supporter :
- **50 millions d'utilisateurs** répartis sur 3 régions géographiques : Europe (EU), Amérique du Nord (NA) et Asie-Pacifique (APAC).
- **10 millions de produits** catalogués globalement.
- **200 millions de commandes** annuelles.
- Des requêtes analytiques temps-réel pour l'entraînement des modèles IA.

### 2.3 Schéma de la Base de Données

```
CLIENT (client_id, nom, email, date_naissance, pays, region, segment_IA, date_inscription)
    PK : client_id

PRODUIT (produit_id, nom, categorie, sous_categorie, prix, stock_global, fournisseur_id, note_moyenne)
    PK : produit_id

COMMANDE (commande_id, client_id, produit_id, quantite, montant, date_commande, statut, entrepot_id)
    PK : commande_id
    FK : client_id → CLIENT, produit_id → PRODUIT, entrepot_id → ENTREPOT

ENTREPOT (entrepot_id, ville, pays, region, capacite_max, latitude, longitude)
    PK : entrepot_id

INTERACTION (interaction_id, client_id, produit_id, type_action, timestamp, score_pertinence)
    PK : interaction_id
    FK : client_id → CLIENT, produit_id → PRODUIT
    -- types d'action : 'vue', 'clic', 'achat', 'note', 'partage'
```

### 2.4 Topologie des Sites

| Site | Région | Rôle | Serveurs |
|------|--------|------|----------|
| S1 | Europe (Paris) | Coordinateur + Données EU | 8 nœuds |
| S2 | Amérique du Nord (New York) | Données NA | 6 nœuds |
| S3 | Asie-Pacifique (Singapour) | Données APAC | 6 nœuds |
| S4 | Europe (Francfort) | Data Warehouse + IA | 12 nœuds |

**Latences inter-sites (ms) :**

|  | S1 | S2 | S3 | S4 |
|--|----|----|----|----|
| **S1** | 0 | 80 | 170 | 15 |
| **S2** | 80 | 0 | 200 | 90 |
| **S3** | 170 | 200 | 0 | 180 |
| **S4** | 15 | 90 | 180 | 0 |

---

## 3. Énoncé — Exercices

---

### Exercice 1 — Analyse et Choix de l'Architecture Distribuée (30 min)

#### 1.1 — Comparaison des Architectures

Complétez le tableau comparatif suivant en remplissant chaque cellule avec une évaluation argumentée (Excellent / Bon / Moyen / Faible) :

| Critère | Shared-Memory | Shared-Disk | Shared-Nothing |
|---------|--------------|-------------|----------------|
| Scalabilité horizontale | | | |
| Tolérance aux pannes | | | |
| Complexité de gestion | | | |
| Cohérence des données | | | |
| Coût matériel | | | |
| Performances OLTP | | | |
| Performances OLAP | | | |
| Adéquation au cloud | | | |

#### 1.2 — Analyse de l'architecture Shared-Nothing

L'architecture Shared-Nothing repose sur le principe que chaque nœud possède ses propres ressources (CPU, mémoire, stockage) et communique exclusivement par passage de messages.

a) Expliquez pourquoi l'architecture Shared-Nothing est la plus adoptée pour les systèmes distribués à grande échelle (type Google Spanner, Apache Cassandra, CockroachDB). Donnez au moins **trois arguments techniques**.

b) Quelles sont les **deux contraintes fondamentales** imposées par cette architecture que le concepteur doit absolument gérer ?

c) Dans le contexte de *ShopMind*, quelle architecture recommandez-vous et pourquoi ? Justifiez en vous appuyant sur les caractéristiques du système (volume de données, distribution géographique, exigences IA).

#### 1.3 — Cas critique : panne de nœud

Dans une architecture Shared-Nothing avec 20 nœuds :

a) Si un nœud tombe en panne, quel pourcentage des données devient potentiellement inaccessible (sans réplication) ?

b) Proposez une stratégie simple pour réduire ce risque. Quel est le prix à payer en termes de stockage ?

---

### Exercice 2 — Fragmentation Horizontale (35 min)

La table `CLIENT` contient **50 millions de tuples**. La répartition géographique est la suivante :

| Région | Nombre de clients | % |
|--------|------------------|---|
| Europe (EU) | 20 000 000 | 40% |
| Amérique du Nord (NA) | 18 000 000 | 36% |
| Asie-Pacifique (APAC) | 12 000 000 | 24% |

#### 2.1 — Fragmentation horizontale primaire

a) Définissez une **fragmentation horizontale primaire** de la table `CLIENT` en 3 fragments selon la région géographique. Donnez pour chaque fragment :
- Le nom du fragment
- La **condition de sélection** (prédicat en algèbre relationnelle)
- Le site d'allocation
- Le nombre estimé de tuples

b) Vérifiez que votre fragmentation respecte les **3 propriétés fondamentales** :
- **Complétude** : tout tuple appartient à au moins un fragment
- **Disjonction** : aucun tuple n'apparaît dans deux fragments (pour la fragmentation horizontale)
- **Reconstruction** : l'union des fragments reconstitue la relation originale

Rédigez les opérations de reconstruction en algèbre relationnelle.

#### 2.2 — Fragmentation horizontale dérivée

La table `COMMANDE` est liée à `CLIENT` par `client_id`. On souhaite fragmenter `COMMANDE` de façon cohérente avec la fragmentation de `CLIENT`.

a) Définissez la **fragmentation horizontale dérivée** de `COMMANDE` en utilisant la semi-jointure (⋉). Écrivez la définition formelle de chaque fragment.

b) Quel est l'avantage principal de cette approche pour les requêtes de type :
```sql
SELECT C.nom, CMD.montant
FROM CLIENT C JOIN COMMANDE CMD ON C.client_id = CMD.client_id
WHERE C.region = 'EU';
```
Expliquez en termes de **localité des données**.

c) Estimez le nombre de tuples dans chaque fragment de `COMMANDE` sachant que la table contient 200 millions de commandes réparties proportionnellement aux clients.

#### 2.3 — Analyse des prédicats simples

Pour la table `INTERACTION` (5 milliards de tuples), on dispose des informations suivantes :

- 60% des interactions proviennent de clients EU
- 25% des interactions concernent des produits de catégorie 'Électronique'
- Les deux prédicats sont **indépendants** (pas de corrélation)

On envisage une fragmentation selon deux prédicats :
- p1 : `client_region = 'EU'`
- p2 : `categorie_produit = 'Électronique'`

a) Listez tous les **prédicats élémentaires** et leurs compléments. Combien de **mintermes** peut-on former ?

b) Calculez le nombre estimé de tuples dans chaque minterm.

c) Identifiez les mintermes **vides** ou **inutiles** et proposez une fragmentation réduite.

---

### Exercice 3 — Fragmentation Verticale (25 min)

La table `CLIENT` possède les attributs suivants avec leurs fréquences d'accès estimées :

| Attribut | Taille (octets) | Accès OLTP (req/s) | Accès IA/Analytics (req/s) |
|----------|----------------|---------------------|---------------------------|
| client_id | 8 | 1 200 | 800 |
| nom | 50 | 900 | 100 |
| email | 80 | 850 | 50 |
| date_naissance | 4 | 200 | 600 |
| pays | 20 | 800 | 700 |
| region | 10 | 1 000 | 750 |
| segment_IA | 30 | 50 | 950 |
| date_inscription | 4 | 100 | 400 |

#### 3.1 — Analyse d'affinité

a) Construisez une **matrice d'affinité des attributs** simplifiée (4×4) en regroupant les attributs par usage dominant :
- Groupe OLTP : {client_id, nom, email, pays, region}
- Groupe IA : {client_id, date_naissance, pays, region, segment_IA, date_inscription}

b) Quels attributs appartiennent aux **deux groupes** ? Quel problème cela pose-t-il ? Comment le résoudre ?

#### 3.2 — Définition des fragments verticaux

a) Proposez une **fragmentation verticale** de la table `CLIENT` en **deux fragments** adaptés aux usages. Pour chaque fragment, donnez :
- La liste des attributs inclus
- La **projection** en algèbre relationnelle
- Le site d'allocation préférentiel
- La justification

b) Quelle est la règle **indispensable** à respecter pour garantir la reconstruction de la relation originale à partir de fragments verticaux ?

c) Montrez la reconstruction en algèbre relationnelle (opération de jointure).

#### 3.3 — Comparaison avec une approche colonne

Les bases de données colonnes (ex: Apache Parquet, Google BigQuery) utilisent une forme de fragmentation verticale automatique.

a) Expliquez en quoi une BDD colonne est une forme de fragmentation verticale totale.

b) Donnez **deux avantages** et **deux inconvénients** par rapport à un stockage ligne classique pour les requêtes analytiques IA.

---

### Exercice 4 — Fragmentation Hybride et Stratégie d'Allocation (30 min)

#### 4.1 — Fragmentation hybride de COMMANDE

La table `COMMANDE` (200 millions de tuples) subit des accès très différenciés :
- Les équipes **opérationnelles** accèdent aux champs : `commande_id, client_id, statut, date_commande`
- Les équipes **financières** accèdent aux champs : `commande_id, client_id, montant, date_commande`
- Les équipes **logistiques** accèdent aux champs : `commande_id, produit_id, quantite, entrepot_id, statut`

a) Proposez une **fragmentation hybride** (horizontale puis verticale, ou l'inverse) de la table `COMMANDE`. Justifiez l'ordre des fragmentations.

b) Nommez chacun des fragments finaux et donnez leur définition formelle (algèbre relationnelle). Combien de fragments obtenez-vous au total ?

c) Représentez l'arbre de fragmentation sous forme de diagramme textuel :

```
COMMANDE
├── [Fragmentation horizontale par région]
│   ├── CMD_EU → [Fragmentation verticale]
│   │   ├── CMD_EU_OPS
│   │   ├── CMD_EU_FIN
│   │   └── CMD_EU_LOG
│   ├── ...
```

#### 4.2 — Stratégie d'allocation

Pour la fragmentation horizontale primaire de `CLIENT`, on dispose des données de requêtes suivantes :

| Requête | Fragment accédé | Site demandeur | Fréquence (req/h) |
|---------|----------------|----------------|-------------------|
| Q1 | CLIENT_EU | S1 | 5 000 |
| Q2 | CLIENT_EU | S2 | 800 |
| Q3 | CLIENT_NA | S2 | 4 500 |
| Q4 | CLIENT_NA | S1 | 600 |
| Q5 | CLIENT_APAC | S3 | 3 800 |
| Q6 | CLIENT_APAC | S1 | 400 |

Coût de transfert inter-sites : **1 unité de coût par tuple transféré par requête**.

Taille des fragments : CLIENT_EU = 20M tuples, CLIENT_NA = 18M tuples, CLIENT_APAC = 12M tuples.

a) Sans réplication, quel est le **plan d'allocation optimal** pour minimiser le coût total de communication ? Montrez le calcul.

b) Supposons une **réplication partielle** de CLIENT_EU sur S2 (pour les requêtes Q2). Calculez le nouveau coût et comparez-le au coût de stockage supplémentaire. Est-ce rentable si la bande passante coûte 0,01€/Go/mois et que chaque Go de stockage coûte 0,05€/mois ?

---

### Exercice 5 — Réplication : Stratégies et Compromis (20 min)

#### 5.1 — Taxonomie de la réplication

a) Complétez le tableau suivant en cochant (✓) les caractéristiques de chaque stratégie :

| Caractéristique | Réplication complète | Réplication partielle | Sans réplication |
|-----------------|---------------------|----------------------|-----------------|
| Disponibilité maximale | | | |
| Risque d'incohérence élevé | | | |
| Coût de stockage minimal | | | |
| Requêtes locales toujours possibles | | | |
| Mise à jour propagée à tous les sites | | | |
| Adapté aux données très volatiles | | | |

b) Pour la table `PRODUIT` (10 millions de tuples, peu modifiée, très consultée), quelle stratégie recommandez-vous ? Justifiez.

c) Pour la table `INTERACTION` (5 milliards de tuples, en écriture intensive), quelle stratégie recommandez-vous ? Justifiez.

#### 5.2 — Problème de la réplication asynchrone

Dans un système de réplication maître-esclave asynchrone, le nœud maître est à S1 (EU) et les réplicas sont à S2 (NA) et S3 (APAC).

a) Un client EU achète le dernier exemplaire d'un produit à t=0. La propagation vers S3 prend 250ms. Un client APAC commande le même produit à t=100ms. Quel problème survient ?

b) Comment le protocole **Read-Your-Writes** permet-il d'atténuer ce problème ?

c) Citez deux systèmes industriels ayant opté pour une **cohérence éventuelle** et expliquez pourquoi c'est acceptable dans leur contexte.

---

### Exercice 6 — Cas d'Étude Intégrateur : Conception Complète (30 min)

*ShopMind* souhaite déployer une nouvelle fonctionnalité : le **tableau de bord IA en temps réel** pour ses équipes marketing. Ce tableau de bord exécute les requêtes suivantes en continu :

```sql
-- Q_RECO : Recommandations personnalisées (exécutée 10 000 fois/heure par région)
SELECT P.nom, P.categorie, I.score_pertinence
FROM INTERACTION I
JOIN PRODUIT P ON I.produit_id = P.produit_id
WHERE I.client_id = ? AND I.type_action IN ('vue', 'clic')
  AND I.timestamp > NOW() - INTERVAL '7 days'
ORDER BY I.score_pertinence DESC
LIMIT 10;

-- Q_TENDANCE : Tendances par région (exécutée 500 fois/heure globalement)
SELECT P.categorie, COUNT(*) as nb_interactions, AVG(I.score_pertinence) as score_moyen
FROM INTERACTION I
JOIN PRODUIT P ON I.produit_id = P.produit_id
JOIN CLIENT C ON I.client_id = C.client_id
WHERE C.region = ? AND I.timestamp > NOW() - INTERVAL '24h'
GROUP BY P.categorie
ORDER BY nb_interactions DESC;
```

#### 6.1 — Analyse des besoins

a) Pour Q_RECO, identifiez les tables impliquées et les fragments qui seraient consultés si la fragmentation est bien conçue. Quel est l'intérêt de la localité des données ici ?

b) Pour Q_TENDANCE, la requête porte sur une seule région. Quelles tables doivent être fragmentées horizontalement pour éviter les transferts inter-sites ?

#### 6.2 — Proposition d'architecture complète

Proposez un schéma d'architecture distribuée complet pour ShopMind incluant :

a) Le type d'architecture choisi (Shared-Nothing) et la justification

b) Le plan de fragmentation pour chacune des 5 tables (au moins le type de fragmentation et l'attribut de partitionnement)

c) La stratégie de réplication pour chaque table

d) L'allocation des fragments sur les 4 sites

Synthétisez votre proposition dans un tableau récapitulatif :

| Table | Type de Fragmentation | Attribut de partition | Réplication | Site principal |
|-------|----------------------|----------------------|-------------|----------------|
| CLIENT | | | | |
| PRODUIT | | | | |
| COMMANDE | | | | |
| ENTREPOT | | | | |
| INTERACTION | | | | |

---

## 4. Questions Bonus — Pour aller plus loin

**B1.** Le théorème CAP (Brewer, 2000) affirme qu'un système distribué ne peut garantir simultanément que deux des trois propriétés suivantes : **C**ohérence, **D**isponibilité (Availability), **T**olérance au partitionnement (Partition tolerance). Dans quel cas ShopMind devrait-il sacrifier la **disponibilité** ? Dans quel cas sacrifierait-il la **cohérence** ?

**B2.** Comparez la fragmentation verticale avec le concept de **vues matérialisées** en base de données distribuée. Quand préfère-t-on l'une à l'autre ?

**B3.** Certains systèmes modernes comme **Google Spanner** prétendent offrir à la fois les garanties ACID et une distribution mondiale. Comment contournent-ils les limites du théorème CAP ?

---

## 5. Modalités de Rendu

Déposez vos fichiers dans le dossier `rendu/` de votre dépôt GitHub Classroom **avant la date limite** :

| Fichier | Contenu |
|---------|---------|
| `rendu/reponses_TP1.md` | Vos réponses rédigées en Markdown |
| `rendu/schemas/` | Schémas et diagrammes annotés (PNG, PDF ou Draw.io) |
| `rendu/rapport_TP1.pdf` | Rapport final compilé |

> Utilisez le fichier `rendu/reponses_TP1.md` fourni comme modèle de départ.

---

*Document préparé par le Département Informatique — ENSTA*
*Bases de Données Avancées — Chapitre 2 : Bases de Données Distribuées*
