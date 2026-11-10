# Tableau de bord Power BI – Production & qualité (mini-ERP industriel)

Cas pratique **Power BI** réalisé à partir de données de production **fictives**, proches de ce qu'on exporte
d'un ERP industriel (articles, ateliers, ordres de fabrication, non-conformités), avec des **erreurs volontaires**
à détecter et à corriger.

Objectif : construire un rapport fiable de bout en bout, sur trois sujets :
1. un **modèle de données** propre (plusieurs tables, clés primaires, relations) ;
2. un **nettoyage sans risque**, en particulier la gestion des doublons ;
3. un **chargement automatisé** : ajouter un nouveau mois de données se fait sans rien refaire.

## Le rapport

Une page de pilotage avec :
- 3 indicateurs clés : **quantité produite**, **taux de réalisation** (produit / prévu) et **taux de non-conformité** ;
- la quantité produite et la quantité non conforme **par article** ;
- un filtre par **date de fin** des ordres de fabrication.

## Les données (`donnees/`, séparateur « ; »)

| Fichier | Contenu | Clé | Type |
|---|---|---|---|
| `articles.csv` | Pièces fabriquées (désignation, famille, prix) | `id_article` | Dimension |
| `ateliers.csv` | Les 4 ateliers (Chaudronnerie, Tuyauterie, Tôlerie, Soudure) | `id_atelier` | Dimension |
| `OF_mensuels/of_2026-08.csv`, `-09`, `-10` | Ordres de fabrication : un export par mois | `num_of` | Faits |
| `non_conformites.csv` | Défauts constatés sur les OF | `id_nc` | Faits |
| `a_ajouter_plus_tard/of_2026-11.csv` | Le mois suivant, pour tester l'automatisation | | |

## 1. Modèle de données : schéma en étoile

```
Articles (1) ──► (*) OF (*) ◄── (1) Ateliers
                     OF (1) ──► (*) Non-conformités
```

- Vérification que chaque clé primaire est unique avant de créer les relations.
- Relations **un à plusieurs**, filtre dans un seul sens (des dimensions vers les faits).

## 2. Nettoyage dans Power Query : « comprendre avant de supprimer »

La règle suivie : **comprendre → définir le doublon sur la clé → valider avec le métier → marquer plutôt que supprimer.**

- Types corrigés (dates, nombres), espaces et majuscules uniformisés sur les clés (« art-004 » redevient « ART-004 »).
- Doublons définis **sur la clé**, pas sur la ressemblance : les exports chargés deux fois sont retirés, mais
  deux constats de non-conformité qui se ressemblent et ont deux `id_nc` différents sont **conservés**,
  car ce sont deux vrais défauts. Les supprimer fausserait le taux de non-conformité.
- Valeurs anormales (quantité vide ou négative) et article inexistant repérés avant le calcul des indicateurs.

## 3. Automatisation

- Le connecteur **Dossier** lit et empile automatiquement tous les fichiers mensuels d'OF.
- **Test** : en copiant `of_2026-11.csv` dans `OF_mensuels` puis en cliquant sur *Actualiser*, novembre apparaît
  déjà nettoyé, sans rien refaire.
- Rapport **publié sur Power BI Service**.

## Ouvrir le projet

1. Ouvrir `Qualite.pbix` avec **Power BI Desktop** (gratuit).
2. Si Power BI ne trouve pas les fichiers : *Transformer les données* → modifier le chemin du dossier `donnees`.

## Pistes d'amélioration

- Une table **Calendrier** pour analyser par mois et par semaine.
- Une table **Anomalies** et une page « Contrôle qualité » qui affichent les lignes écartées à chaque actualisation.
- En entreprise : connexion directe à l'ERP, passerelle et actualisation planifiée sur Power BI Service.

Projet associé : [agent-completion-excel](https://github.com/Dwayne1976/agent-completion-excel), un agent IA qui
complète les cases vides de fichiers Excel en s'appuyant sur des fichiers existants.

Auteur : Dwayne Enam, étudiant ingénieur à l'EFREI Paris (Big Data & IA).
