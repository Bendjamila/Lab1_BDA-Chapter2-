# Mémento — Algèbre Relationnelle Distribuée
## Aide-mémoire pour le TP 2.1

---

## Opérateurs de base

| Opérateur | Symbole | Description | Exemple |
|-----------|---------|-------------|---------|
| Sélection | σ | Filtre les tuples selon une condition | `σ_{region='EU'}(CLIENT)` |
| Projection | π | Sélectionne des colonnes | `π_{nom, email}(CLIENT)` |
| Jointure naturelle | ⋈ | Joint deux relations sur attributs communs | `CLIENT ⋈ COMMANDE` |
| Semi-jointure | ⋉ | Tuples de R qui ont correspondance dans S | `COMMANDE ⋉ CLIENT_EU` |
| Union | ∪ | Réunion de deux relations | `F1 ∪ F2 ∪ F3` |
| Différence | − | Tuples dans R mais pas dans S | `R − S` |
| Intersection | ∩ | Tuples communs à R et S | `R ∩ S` |
| Produit cartésien | × | Toutes les combinaisons de tuples | `R × S` |

---

## Fragmentation Horizontale

### Primaire
```
F_i = σ_{prédicat_i}(R)
```

**Propriétés à vérifier :**
- **Complétude :** `∀t ∈ R, ∃i tel que t ∈ F_i`
- **Disjonction :** `∀i ≠ j, F_i ∩ F_j = ∅`
- **Reconstruction :** `R = F_1 ∪ F_2 ∪ ... ∪ F_n`

**Exemple :**
```
CLIENT_EU   = σ_{region='EU'}(CLIENT)
CLIENT_NA   = σ_{region='NA'}(CLIENT)
CLIENT_APAC = σ_{region='APAC'}(CLIENT)

CLIENT = CLIENT_EU ∪ CLIENT_NA ∪ CLIENT_APAC
```

### Dérivée (par semi-jointure)
```
F_i^dérivé = R ⋉ F_i^primaire
```

**Exemple :**
```
CMD_EU   = COMMANDE ⋉_{client_id} CLIENT_EU
CMD_NA   = COMMANDE ⋉_{client_id} CLIENT_NA
CMD_APAC = COMMANDE ⋉_{client_id} CLIENT_APAC
```

---

## Fragmentation Verticale

### Définition
```
F_i = π_{A_i}(R)    où A_i ⊆ attributs(R)
```

**Règle indispensable :** La clé primaire (PK) doit apparaître dans **tous** les fragments.

**Reconstruction :**
```
R = F_1 ⋈_{PK} F_2 ⋈_{PK} ... ⋈_{PK} F_n
```

**Exemple :**
```
CLIENT_OLTP = π_{client_id, nom, email, pays, region}(CLIENT)
CLIENT_IA   = π_{client_id, date_naissance, pays, region, segment_IA, date_inscription}(CLIENT)

CLIENT = CLIENT_OLTP ⋈_{client_id} CLIENT_IA
```

---

## Fragmentation Hybride

### Horizontale puis Verticale (H→V)
```
H_i = σ_{p_i}(R)                 ← Fragmentation horizontale
V_{i,j} = π_{A_j}(H_i)           ← Fragmentation verticale de chaque fragment
```

### Verticale puis Horizontale (V→H)
```
V_j = π_{A_j}(R)                 ← Fragmentation verticale
H_{j,i} = σ_{p_i}(V_j)           ← Fragmentation horizontale de chaque fragment
```

---

## Mintermes

Avec n prédicats élémentaires `{p_1, p_2, ..., p_n}`, on peut former `2^n` mintermes :

```
m_k = p_1^{x_1} ∧ p_2^{x_2} ∧ ... ∧ p_n^{x_n}
      où x_i ∈ {0=vrai, 1=faux}
```

Un minterme est **inutile** si son cardinal est 0 (aucun tuple ne satisfait la condition).

---

## Coût d'allocation

```
Coût_total = Σ_{requêtes q_i} freq(q_i) × coût_transfert(q_i)
```

- Si le fragment est **local** au site demandeur : `coût_transfert = 0`
- Si le fragment est **distant** : `coût_transfert = nb_tuples × coût_unitaire`

---

## Rappel : Théorème CAP

Un système distribué ne peut garantir simultanément que **2 des 3** :

| Propriété | Description |
|-----------|-------------|
| **C** onsistency | Toutes les lectures retournent la donnée la plus récente |
| **A** vailability | Toutes les requêtes reçoivent une réponse (pas de timeout) |
| **P** artition tolerance | Le système fonctionne malgré des pannes réseau |

- **CP** : Cohérence + Tolérance (ex: HBase, Zookeeper) — Sacrifie la disponibilité
- **AP** : Disponibilité + Tolérance (ex: Cassandra, DynamoDB) — Sacrifie la cohérence
- **CA** : Cohérence + Disponibilité — Impossible dans un réseau réel (partitions inévitables)
