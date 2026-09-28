# Documentation du projet — Formations 2024

**Auteure :** Sanaba Kanté · **Cours :** DECI1021 – Nettoyage et préparation des données (CCNB Bathurst) · **Date :** avril 2026

Jeu de données : `DECI1021 - Projet Final - Formations.xlsx`, 3 tables (Inscriptions, Employes, Formations).
Objectif : analyser les inscriptions des employés aux activités de formation pour l'année 2024.

Les versions Word originales se trouvent dans [`original-word/`](original-word/).

---

## 1. Audit de données

| Colonne (table Inscriptions) | Dimension | Rôle | Critique | Problème observé |
|---|---|---|---|---|
| Departement | Cohérence | Catégorie | Oui | Informatique et Finance écrits de deux façons différentes |
| NomFormation / FormationID | Intégrité | Catégorie | Oui | Formation `F10` absente de la table Formations |
| DateDebut / DateFin | Cohérence | Date | Oui | Certaines dates de début sont postérieures aux dates de fin |
| DureeHeures | Complétude | Mesure | Oui | Une valeur manquante |
| Statut | Cohérence | Catégorie | Oui | « Complété » écrit sans accent (« complete ») |

**Remarques générales**
- **Inscriptions :** la ligne 2 est dupliquée en ligne 9, et `F10` n'existe pas dans la table Formations.
- **Employes :** les en-têtes ne sont pas promus.
- **Formations :** la colonne NomFormation est absente (récupérable par jointure).

**Conclusion :** les tables de dimension (Employes, Formations) sont globalement fiables. La table de faits (Inscriptions) concentre les problèmes de cohérence, de complétude et d'intégrité référentielle. Après nettoyage et restructuration en modèle en étoile, le jeu de données est exploitable.

---

## 2. Plan de nettoyage

| Ordre | Problème | Colonne | Priorité | Dimension | Action | Justification |
|---|---|---|---|---|---|---|
| 1 | Doublon exact (ligne 9) | Toute la ligne | Haute | Exactitude | Supprimer le doublon | Évite la double comptabilisation |
| 2 | Valeurs écrites différemment (« IT » / « Informatique », « Finances » / « Finance ») | Departement | Haute | Uniformité | Standardiser | Permet une agrégation correcte |
| 3 | DateDebut > DateFin (ligne 3) | DateDebut, DateFin | Haute | Cohérence | Inverser les deux dates | Il s'agit d'une inversion : les deux dates sont présentes |
| 4 | Valeur manquante | DureeHeures | Moyenne | Complétude | Calculer à partir des dates | Évite de perdre la ligne |
| 5 | `F10` absent de la table Formations | FormationID | Haute | Intégrité | Créer l'enregistrement manquant s'il est réel, sinon corriger la ligne | Chaque clé étrangère doit référencer une clé primaire valide |
| 6 | Colonne NomFormation absente | Formations | Moyenne | Complétude | Ajouter par jointure avec Inscriptions | Enrichit la dimension |
| 7 | En-têtes non promus | Employes | Haute | Cohérence | Promouvoir les en-têtes et typer les colonnes | Lecture correcte des colonnes |

**Modèle cible :** modèle en étoile, avec Inscriptions comme table de faits et Employes, Formations (et Temps) comme dimensions.

---

## 3. Transformations appliquées (Power Query)

| Ordre | Transformation | Justification |
|---|---|---|
| 1 | Suppression des doublons dans Inscriptions | Évite la double comptabilisation |
| 2 | Standardisation des valeurs de Departement | Catégories uniformes pour l'agrégation |
| 3 | Standardisation du Statut | Cohérence sémantique des statuts |
| 4 | Colonnes de validation des dates (DateDebutValide, DateFinv) | Corrige les incohérences temporelles |
| 5 | Calcul de la durée (DureeHeures_finale) | Complète la mesure manquante |
| 6 | Suppression des colonnes de dates et de durée d'origine | Évite la redondance |
| 7 | Renommage des colonnes corrigées | Structure finale lisible |
| 8 | Définition des types (Formations) | Exactitude des données |
| 9 | Fusion Formations + Inscriptions (jointure externe droite) | Conserve toutes les inscriptions |
| 10 | Expansion de NomFormation depuis Inscriptions | Complète la dimension Formations |
| 11 | Suppression des lignes trop incomplètes (Cloud partiel) | Qualité des données |
| 12 | Suppression des doublons après fusion | Évite les duplications dues à la jointure |

---

## 4. Journal des problèmes et décisions

| Problème | Impact | Solution | Justification |
|---|---|---|---|
| Doublon (ligne 9 identique à la ligne 2) | Double comptabilisation | Suppression du doublon | Exactitude des analyses |
| Departement écrit de plusieurs façons | Catégories fragmentées | Standardisation en « Informatique » et « Finance » | Uniformité |
| Variantes du Statut | Mauvaise catégorisation | Uniformisation en « Complété » | Agrégation fiable |
| DateDebut > DateFin, avec risque de réapparition | Données invalides, biais | Colonnes corrigées avec validation | Cohérence temporelle et contrôle durable de la qualité |
| DureeHeures manquante | Durée non analysable | Calcul à partir des dates corrigées | Complétude |
| `F10` (« Cloud ») absent de Formations | Intégrité référentielle | Ajout de F10 dans Formations à partir des Inscriptions | Conserve toutes les inscriptions |
| NomFormation absente de Formations | Dimension incomplète | Récupération par jointure | Compréhension des données |
| En-têtes non promus (Employes) | Colonnes mal interprétées | Promotion des en-têtes | Bonne structure |
| Perte de données lors de la jointure Formations–Inscriptions | Inscriptions exclues | Jointure externe droite | Conservation complète de la table de faits |

---

## 5. Catalogue et dictionnaire de données

### Catalogue

| Table | Description | Clé primaire | Type |
|---|---|---|---|
| Inscriptions | Inscriptions des employés aux formations | InscriptionID | Table de faits |
| Employes | Informations sur les employés | EmployeID | Dimension |
| Formations | Informations sur les formations | FormationID | Dimension |

### Table Inscriptions

| Variable | Description | Type | Nature | Règle | Exemple |
|---|---|---|---|---|---|
| InscriptionID | Identifiant unique | Numérique | Clé primaire | Généré automatiquement | 1 |
| EmployeID | Identifiant de l'employé | Texte | Clé étrangère | Doit exister dans Employes | E1 |
| NomEmploye | Nom de l'employé | Texte | Attribut | — | Alice Martin |
| Departement | Département | Texte | Catégorie | Valeurs standardisées | Data |
| FormationID | Identifiant de la formation | Texte | Clé étrangère | Doit exister dans Formations | F4 |
| NomFormation | Nom de la formation | Texte | Attribut | — | SQL |
| DateDebut | Date de début | Date | Date | ≤ DateFin | 2024-01-10 |
| DateFin | Date de fin | Date | Date | ≥ DateDebut | 2024-01-12 |
| DureeHeures | Durée en heures | Numérique | Mesure | Calculée si vide | 16 |
| Statut | Statut de la formation | Texte | Catégorie | Valeurs standardisées | Complété |

### Table Formations

| Variable | Description | Type | Nature | Règle | Exemple |
|---|---|---|---|---|---|
| FormationID | Identifiant unique de la formation | Texte | Clé primaire | Unique | F1 |
| NomFormation | Nom de la formation | Texte | Attribut | Ajouté par jointure | Python |
| Categorie | Catégorie de la formation | Texte | Catégorie | — | BI |
| Niveau | Niveau de la formation | Texte | Catégorie | — | Débutant |
| DureePrevue | Durée prévue de la formation | Numérique | Mesure | — | — |

### Table Employes

| Variable | Description | Type | Nature | Règle | Exemple |
|---|---|---|---|---|---|
| EmployeID | Identifiant de l'employé | Texte | Clé primaire | Unique | E1 |
| NomEmploye | Nom complet | Texte | Attribut | — | Alice Martin |
| Niveau | Niveau de l'employé | Texte | Attribut | — | Débutant |
| Departement | Département | Texte | Catégorie | Valeurs standardisées | Informatique |
| Localisation | Lieu de travail | Texte | Catégorie | — | Montréal |
