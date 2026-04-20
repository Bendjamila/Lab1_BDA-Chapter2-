# 🗄️ BDA — Chapitre 2 : Bases de Données Distribuées
## TP 2.1 — Architectures Distribuées et Fragmentation des Données

**ENSTA Alger — 3ème Année Ingénieur — Parcours AI & SS**
Module : Bases de Données Avancées | Année universitaire 2024–2025

---

## 📋 Informations du TP

| | |
|---|---|
| **Chapitre** | 2 — Bases de Données Distribuées |
| **TP** | 2.1 — Architectures et Fragmentation |
| **Durée** | 2h30 à 3h |
| **Mode** | Binôme ou individuel |
| **Date limite** | _(à préciser par l'enseignant)_ |

---

## 🎯 Objectifs

À l'issue de ce TP, vous serez capable de :

1. **Analyser** les architectures Shared-Memory, Shared-Disk et Shared-Nothing
2. **Concevoir** des schémas de fragmentation horizontale, verticale et hybride
3. **Évaluer** les stratégies de réplication et leurs impacts
4. **Optimiser** l'allocation des fragments pour minimiser les coûts réseau
5. **Appliquer** ces concepts au cas réel *ShopMind* (plateforme e-commerce IA)

---

## 📁 Structure du Dépôt

```
BDA-Chapitre2-TP1/
│
├── 📄 README.md                          ← Ce fichier
│
├── 📂 enonce/
│   └── TP1_Enonce.md                     ← Énoncé complet du TP (6 exercices)
│
├── 📂 rendu/                             ← 📤 DÉPOSEZ VOS FICHIERS ICI
│   ├── reponses_TP1.md                   ← Modèle à compléter (vos réponses)
│   ├── rapport_TP1.pdf                   ← Rapport final compilé (à ajouter)
│   └── schemas/                          ← Vos schémas et diagrammes annotés
│       └── .gitkeep
│
└── 📂 ressources/
    ├── algebre_relationnelle_memento.md  ← Rappel algèbre relationnelle
    ├── glossaire_BDD_distribuees.md      ← Glossaire des termes clés
    └── references.md                     ← Références bibliographiques
```

---

## 🚀 Comment démarrer

### Étape 1 — Cloner le dépôt

```bash
git clone https://github.com/ENSTA-BDA/BDA-Chapitre2-TP1-VOTRE-PSEUDO.git
cd BDA-Chapitre2-TP1-VOTRE-PSEUDO
```

### Étape 2 — Lire l'énoncé

Ouvrez `enonce/TP1_Enonce.md` dans VS Code ou directement sur GitHub.

### Étape 3 — Compléter vos réponses

```bash
# Ouvrir le modèle de réponses dans VS Code
code rendu/reponses_TP1.md
```

Remplissez toutes les sections marquées `_(Votre réponse)_` dans `rendu/reponses_TP1.md`.

### Étape 4 — Ajouter vos schémas

Placez vos diagrammes annotés (PNG, PDF, SVG ou fichiers Draw.io) dans `rendu/schemas/`.

### Étape 5 — Soumettre

```bash
# Ajouter vos fichiers
git add rendu/

# Committer avec un message clair
git commit -m "feat(TP1): complétion exercices 1 à 6 + rapport"

# Pousser vers GitHub Classroom
git push origin main
```

---

## 📝 Contenu de l'Énoncé

Le TP contient **6 exercices** couvrant tout le chapitre 2 :

| Exercice | Thème | Durée |
|----------|-------|-------|
| **Ex. 1** | Analyse et choix de l'architecture distribuée | 30 min |
| **Ex. 2** | Fragmentation horizontale (primaire, dérivée, prédicats) | 35 min |
| **Ex. 3** | Fragmentation verticale et affinité des attributs | 25 min |
| **Ex. 4** | Fragmentation hybride et stratégie d'allocation | 30 min |
| **Ex. 5** | Réplication : stratégies et compromis | 20 min |
| **Ex. 6** | Cas d'étude intégrateur — Architecture complète ShopMind | 30 min |
| **Bonus** | Théorème CAP, Vues matérialisées, Google Spanner | — |

---

## 📤 Modalités de Rendu

Déposez **obligatoirement** les fichiers suivants dans `rendu/` :

| Fichier | Description | Obligatoire |
|---------|-------------|:-----------:|
| `rendu/reponses_TP1.md` | Réponses rédigées (modèle fourni) | ✅ |
| `rendu/rapport_TP1.pdf` | Rapport final mis en forme | ✅ |
| `rendu/schemas/` | Schémas annotés (PNG/PDF/SVG) | ✅ |

> **⚠️ Important :** Vérifiez que votre `README.md` contient votre nom et matricule avant de soumettre.

---

## 🛠️ Ressources utiles

| Ressource | Lien |
|-----------|------|
| Rappel algèbre relationnelle | [`ressources/algebre_relationnelle_memento.md`](ressources/algebre_relationnelle_memento.md) |
| Glossaire BDD distribuées | [`ressources/glossaire_BDD_distribuees.md`](ressources/glossaire_BDD_distribuees.md) |
| Références bibliographiques | [`ressources/references.md`](ressources/references.md) |
| Draw.io (schémas gratuits) | [app.diagrams.net](https://app.diagrams.net) |
| Cours Chapitre 2 (ENT) | _(lien ENT ENSTA)_ |

---

## 👤 Informations Étudiant

> **Complétez cette section avant votre premier commit :**

| | |
|---|---|
| **Nom complet** | _(Prénom NOM)_ |
| **Matricule** | _(ex: 20231234)_ |
| **Binôme** | _(Prénom NOM du partenaire ou "Individuel")_ |
| **Promotion** | 3A Informatique 2024–2025 |

---

## 📊 Barème indicatif

| Exercice | Points |
|----------|--------|
| Ex. 1 — Architecture | 20 pts |
| Ex. 2 — Fragmentation horizontale | 25 pts |
| Ex. 3 — Fragmentation verticale | 20 pts |
| Ex. 4 — Fragmentation hybride + allocation | 20 pts |
| Ex. 5 — Réplication | 15 pts |
| Ex. 6 — Cas intégrateur | 20 pts |
| **TOTAL** | **120 pts** (ramené à 20) |
| Bonus | +10 pts max |

---

*Département Informatique — ENSTA Alger*
*Bases de Données Avancées — 2024/2025*
