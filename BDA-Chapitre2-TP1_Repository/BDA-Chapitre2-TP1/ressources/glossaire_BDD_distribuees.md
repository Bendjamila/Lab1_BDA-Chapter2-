# Glossaire — Bases de Données Distribuées
## TP 2.1 — Architectures et Fragmentation

---

## A

**Allocation** : Processus d'assignation des fragments de données aux sites physiques d'un système distribué. L'objectif est de minimiser les transferts réseau tout en maximisant la disponibilité.

**Architecture Shared-Disk** : Architecture distribuée où tous les nœuds partagent un stockage commun (SAN) mais ont leur propre CPU et mémoire. Exemple : Oracle RAC.

**Architecture Shared-Memory** : Architecture où plusieurs processeurs partagent une mémoire physique commune. Limite naturelle autour de 256 processeurs (bus mémoire saturé).

**Architecture Shared-Nothing** : Architecture où chaque nœud est totalement autonome (CPU, mémoire, stockage propres). Communication uniquement par messages réseau. Exemple : Google Spanner, Cassandra.

---

## C

**CAP (Théorème)** : Théorème de Brewer (2000) : un système distribué ne peut garantir simultanément la Consistance, la Disponibilité (Availability) et la Tolérance au partitionnement (Partition tolerance).

**Cohérence causale** : Garantie que si l'opération A précède l'opération B dans la causalité, tous les nœuds les verront dans cet ordre.

**Cohérence éventuelle (Eventual Consistency)** : Garantie faible indiquant que, si aucune nouvelle mise à jour n'est faite, tous les réplicas convergeront vers la même valeur (mais pas immédiatement).

**Complétude** : Propriété d'une fragmentation garantissant que tout tuple de la relation originale appartient à au moins un fragment.

---

## D

**Disjonction** : Propriété d'une fragmentation horizontale garantissant qu'aucun tuple n'apparaît dans deux fragments différents (pas de redondance).

---

## F

**Fragment** : Sous-ensemble d'une relation obtenu par sélection (fragmentation horizontale) ou projection (fragmentation verticale).

**Fragmentation dérivée** : Fragmentation d'une table obtenue par semi-jointure avec la fragmentation primaire d'une autre table. Préserve la localité des jointures.

**Fragmentation horizontale** : Division d'une relation en sous-ensembles de tuples selon une condition de sélection (σ).

**Fragmentation hybride** : Combinaison de fragmentation horizontale et verticale, appliquées successivement.

**Fragmentation primaire** : Fragmentation d'une table maître, base pour la fragmentation dérivée des tables liées.

**Fragmentation verticale** : Division d'une relation en sous-ensembles d'attributs selon une projection (π). La clé primaire doit être incluse dans tous les fragments.

---

## L

**Latence** : Délai de transmission d'un message entre deux nœuds. Exprimée en millisecondes (ms). Impact direct sur les performances des requêtes distribuées.

**Localité des données** : Propriété d'un système distribué permettant d'exécuter une requête sur le nœud où résident les données concernées, sans transfert réseau.

---

## M

**Minterme** : Conjonction de prédicats élémentaires ou de leurs négations. Avec n prédicats, on peut former 2^n mintermes. Base de la théorie de la fragmentation.

---

## N

**Nœud coordinateur** : Site chargé de distribuer et coordonner l'exécution des requêtes dans un système distribué. Dans ShopMind : S1 (Paris).

---

## O

**OLAP (Online Analytical Processing)** : Requêtes analytiques complexes sur de grands volumes de données. Bénéficie des architectures Shared-Nothing (parallélisme massif).

**OLTP (Online Transaction Processing)** : Requêtes transactionnelles courtes et fréquentes. Bénéficie des architectures à mémoire partagée ou d'une bonne localité des données.

---

## P

**Partition** : Dans le théorème CAP, scission du réseau en deux parties qui ne peuvent plus communiquer temporairement.

**Prédicat** : Condition booléenne sur les attributs d'une relation (ex: `region = 'EU'`).

---

## R

**Read-Your-Writes** : Garantie de cohérence assurant qu'un utilisateur voit toujours ses propres écritures, même sur un réplica.

**Reconstruction** : Propriété d'une fragmentation garantissant que l'union (fragmentation horizontale) ou la jointure (fragmentation verticale) des fragments reconstitue la relation originale.

**Réplication** : Duplication des données sur plusieurs sites pour améliorer la disponibilité et les performances en lecture.

**Réplication asynchrone** : Mode de réplication où les mises à jour sont propagées aux réplicas avec un délai. Moins coûteuse mais risque d'incohérence temporaire.

**Réplication synchrone** : Mode de réplication où les mises à jour sont propagées simultanément à tous les réplicas avant confirmation. Forte cohérence mais latence accrue.

---

## S

**Semi-jointure (⋉)** : Opération `R ⋉ S` qui retourne les tuples de R qui ont au moins un tuple correspondant dans S. Utilisée pour la fragmentation dérivée.

**SPOF (Single Point of Failure)** : Point unique de défaillance. Son élimination est un objectif des architectures distribuées.

---

## T

**TrueTime (Google)** : API d'horloge atomique/GPS utilisée par Google Spanner pour synchroniser les transactions globalement avec une incertitude de quelques millisecondes.

---

## W

**WAL (Write-Ahead Log)** : Journal des modifications écrit avant l'application des changements. Utilisé pour la récupération après panne et la réplication.

---

*Département Informatique — ENSTA Alger — BDA 2024/2025*
