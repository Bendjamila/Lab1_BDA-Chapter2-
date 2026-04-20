# TP 2.1 — Mes Réponses
## BDA — Architectures Distribuées et Fragmentation des Données

---

> **Étudiant(e) :** _(Prénom Nom)_
> **Matricule :** _(ex: 20231234)_
> **Binôme :** _(Prénom Nom du partenaire, ou "Individuel")_
> **Date de rendu :** _(JJ/MM/AAAA)_

---

## Exercice 1 — Analyse et Choix de l'Architecture Distribuée

### 1.1 — Tableau comparatif des architectures

| Critère | Shared-Memory | Shared-Disk | Shared-Nothing |
|---------|:---:|:---:|:---:|
| Scalabilité horizontale | | | |
| Tolérance aux pannes | | | |
| Complexité de gestion | | | |
| Cohérence des données | | | |
| Coût matériel | | | |
| Performances OLTP | | | |
| Performances OLAP | | | |
| Adéquation au cloud | | | |

### 1.2 — Analyse de l'architecture Shared-Nothing

**a) Trois arguments techniques justifiant l'adoption du Shared-Nothing à grande échelle :**

1. _(Votre argument 1)_
2. _(Votre argument 2)_
3. _(Votre argument 3)_

**b) Deux contraintes fondamentales du Shared-Nothing :**

1. _(Contrainte 1)_
2. _(Contrainte 2)_

**c) Architecture recommandée pour ShopMind :**

_(Votre réponse et justification)_

### 1.3 — Cas critique : panne de nœud

**a) Pourcentage de données inaccessibles lors d'une panne :**

_(Calcul et réponse)_

**b) Stratégie pour réduire le risque :**

_(Stratégie proposée et coût en stockage)_

---

## Exercice 2 — Fragmentation Horizontale

### 2.1 — Fragmentation horizontale primaire

**a) Définition des 3 fragments de CLIENT :**

| Fragment | Prédicat (algèbre relationnelle) | Site d'allocation | Nb tuples estimé |
|----------|----------------------------------|-------------------|-----------------|
| CLIENT_EU | | | |
| CLIENT_NA | | | |
| CLIENT_APAC | | | |

**b) Vérification des 3 propriétés fondamentales :**

- **Complétude :** _(Démonstration)_
- **Disjonction :** _(Démonstration)_
- **Reconstruction :** _(Opération algébrique)_

```
CLIENT = CLIENT_EU ∪ CLIENT_NA ∪ CLIENT_APAC
```

### 2.2 — Fragmentation horizontale dérivée

**a) Définition formelle des fragments de COMMANDE (semi-jointure ⋉) :**

```
CMD_EU   = COMMANDE ⋉ CLIENT_EU
CMD_NA   = COMMANDE ⋉ CLIENT_NA
CMD_APAC = COMMANDE ⋉ CLIENT_APAC
```

_(Expliquer la définition)_

**b) Avantage en termes de localité :**

_(Votre explication)_

**c) Estimation du nombre de tuples dans chaque fragment de COMMANDE :**

| Fragment | Calcul | Nb tuples estimé |
|----------|--------|-----------------|
| CMD_EU | 200M × 40% | |
| CMD_NA | 200M × 36% | |
| CMD_APAC | 200M × 24% | |

### 2.3 — Analyse des prédicats simples (INTERACTION)

**a) Prédicats élémentaires et mintermes :**

- p1 : `client_region = 'EU'` / ¬p1 : `client_region ≠ 'EU'`
- p2 : `categorie_produit = 'Électronique'` / ¬p2 : `categorie_produit ≠ 'Électronique'`

Nombre de mintermes : _(calcul : 2² = ?)_

**b) Estimation des tuples par minterme :**

| Minterme | Condition | % | Nb tuples |
|----------|-----------|---|-----------|
| m1 | p1 ∧ p2 | | |
| m2 | p1 ∧ ¬p2 | | |
| m3 | ¬p1 ∧ p2 | | |
| m4 | ¬p1 ∧ ¬p2 | | |

**c) Mintermes inutiles et fragmentation réduite :**

_(Votre analyse)_

---

## Exercice 3 — Fragmentation Verticale

### 3.1 — Analyse d'affinité

**a) Matrice d'affinité simplifiée :**

_(Construire et commenter la matrice)_

**b) Attributs partagés et problème posé :**

Attributs communs : _{client_id, pays, region}_

Problème : _(Expliquer)_

Solution : _(Proposer)_

### 3.2 — Définition des fragments verticaux

**a) Deux fragments verticaux de CLIENT :**

**Fragment CLIENT_OLTP :**
- Attributs : `{client_id, nom, email, pays, region}`
- Projection : `π_{client_id, nom, email, pays, region}(CLIENT)`
- Site préférentiel : _(S1/S2/S3)_
- Justification : _(Explication)_

**Fragment CLIENT_IA :**
- Attributs : `{client_id, date_naissance, pays, region, segment_IA, date_inscription}`
- Projection : `π_{client_id, date_naissance, pays, region, segment_IA, date_inscription}(CLIENT)`
- Site préférentiel : _(S4)_
- Justification : _(Explication)_

**b) Règle indispensable :**

_(La clé primaire doit apparaître dans tous les fragments verticaux)_

**c) Reconstruction en algèbre relationnelle :**

```
CLIENT = CLIENT_OLTP ⋈_{client_id} CLIENT_IA
```

### 3.3 — Comparaison avec l'approche colonne

**a) BDD colonne = fragmentation verticale totale :**

_(Explication)_

**b) Avantages et inconvénients :**

| | Avantages | Inconvénients |
|-|-----------|---------------|
| BDD colonne | 1. | 1. |
| vs ligne | 2. | 2. |

---

## Exercice 4 — Fragmentation Hybride et Allocation

### 4.1 — Fragmentation hybride de COMMANDE

**a) Type de fragmentation choisie et justification :**

_(Horizontale d'abord, puis verticale — justification)_

**b) Fragments finaux (définitions formelles) :**

```
CMD_EU_OPS  = π_{commande_id, client_id, statut, date_commande}(COMMANDE ⋉ CLIENT_EU)
CMD_EU_FIN  = π_{commande_id, client_id, montant, date_commande}(COMMANDE ⋉ CLIENT_EU)
CMD_EU_LOG  = π_{commande_id, produit_id, quantite, entrepot_id, statut}(COMMANDE ⋉ CLIENT_EU)
...
```

Nombre total de fragments : _(3 régions × 3 usages = ?)_

**c) Arbre de fragmentation :**

```
COMMANDE
├── [Fragmentation horizontale par région]
│   ├── CMD_EU → [Fragmentation verticale]
│   │   ├── CMD_EU_OPS   (Opérationnel)
│   │   ├── CMD_EU_FIN   (Financier)
│   │   └── CMD_EU_LOG   (Logistique)
│   ├── CMD_NA → [Fragmentation verticale]
│   │   ├── CMD_NA_OPS
│   │   ├── CMD_NA_FIN
│   │   └── CMD_NA_LOG
│   └── CMD_APAC → [Fragmentation verticale]
│       ├── CMD_APAC_OPS
│       ├── CMD_APAC_FIN
│       └── CMD_APAC_LOG
```

### 4.2 — Stratégie d'allocation

**a) Plan d'allocation optimal (sans réplication) :**

| Fragment | Site optimal | Justification (fréquence dominante) |
|----------|-------------|--------------------------------------|
| CLIENT_EU | S1 | Q1 (5000 req/h) > Q2 (800 req/h) |
| CLIENT_NA | S2 | Q3 (4500 req/h) > Q4 (600 req/h) |
| CLIENT_APAC | S3 | Q5 (3800 req/h) > Q6 (400 req/h) |

Calcul du coût total :

| Requête | Local? | Coût unitaire | Fréquence | Coût total |
|---------|--------|--------------|-----------|-----------|
| Q1 | ✓ OUI | 0 | 5 000 | 0 |
| Q2 | ✗ NON | 1 | 800 | 800 |
| Q3 | ✓ OUI | 0 | 4 500 | 0 |
| Q4 | ✗ NON | 1 | 600 | 600 |
| Q5 | ✓ OUI | 0 | 3 800 | 0 |
| Q6 | ✗ NON | 1 | 400 | 400 |
| **TOTAL** | | | | **1 800 unités/h** |

**b) Analyse de la réplication partielle de CLIENT_EU sur S2 :**

_(Calcul économie vs coût stockage — votre conclusion)_

---

## Exercice 5 — Réplication

### 5.1 — Taxonomie de la réplication

**a) Tableau complété :**

| Caractéristique | Réplication complète | Réplication partielle | Sans réplication |
|-----------------|:-------------------:|:--------------------:|:---------------:|
| Disponibilité maximale | | | |
| Risque d'incohérence élevé | | | |
| Coût de stockage minimal | | | |
| Requêtes locales toujours possibles | | | |
| Mise à jour propagée à tous les sites | | | |
| Adapté aux données très volatiles | | | |

**b) Stratégie pour PRODUIT :**

_(Votre recommandation et justification)_

**c) Stratégie pour INTERACTION :**

_(Votre recommandation et justification)_

### 5.2 — Problème de la réplication asynchrone

**a) Problème identifié :**

_(Décrire le scénario de double-vente / incohérence)_

**b) Protocole Read-Your-Writes :**

_(Explication du mécanisme)_

**c) Deux systèmes en cohérence éventuelle :**

1. **_(Système 1)_** : _(Pourquoi acceptable)_
2. **_(Système 2)_** : _(Pourquoi acceptable)_

---

## Exercice 6 — Cas d'Étude Intégrateur

### 6.1 — Analyse des besoins

**a) Q_RECO — Tables et fragments consultés :**

_(Analyse)_

**b) Q_TENDANCE — Tables à fragmenter horizontalement :**

_(Analyse)_

### 6.2 — Architecture complète ShopMind

**Tableau récapitulatif :**

| Table | Type de Fragmentation | Attribut de partition | Réplication | Site principal |
|-------|----------------------|----------------------|-------------|----------------|
| CLIENT | | | | |
| PRODUIT | | | | |
| COMMANDE | | | | |
| ENTREPOT | | | | |
| INTERACTION | | | | |

**Justifications :**

_(Expliquer les choix pour chaque table)_

---

## Questions Bonus

**B1 — Théorème CAP pour ShopMind :**

_(Votre analyse)_

**B2 — Fragmentation verticale vs Vues matérialisées :**

_(Votre comparaison)_

**B3 — Google Spanner et le théorème CAP :**

_(Votre explication)_

---

*Fin du rendu — TP 2.1 BDA Distribuées — ENSTA*
