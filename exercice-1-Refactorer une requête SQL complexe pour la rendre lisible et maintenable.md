
# TP SQL – Requêtes avancées, agrégations et Common Table Expressions (CTE)

## Contexte pédagogique

Dans ce TP, vous travaillez sur une base de données relationnelle représentant un **système de vente en ligne**.
L’objectif est d’écrire puis de **refactoriser une requête SQL complexe** afin d’améliorer sa **lisibilité**, sa **maintenabilité** et sa **robustesse**, en utilisant des mécanismes avancés du langage SQL, notamment les **Common Table Expressions (CTE)**.

Ce TP s’inscrit dans une démarche de **génie logiciel appliqué aux bases de données** : il ne s’agit pas uniquement d’obtenir un résultat correct, mais de produire une requête **compréhensible, modulaire et évolutive**.

---

## Schéma de données (rappel)

La base de données est composée des tables suivantes :

* **Customers** (`customer_id`, `first_name`, `last_name`, `email`, …)
* **Orders** (`order_id`, `customer_id`, `order_date`, …)
* **OrderItems** (`order_item_id`, `order_id`, `product_id`, `quantity`, `unit_price`)
* **Products** (`product_id`, `product_name`, `category_id`, …)
* **Categories** (`category_id`, `category_name`)

Relations principales :

* Un client peut passer plusieurs commandes
* Une commande contient plusieurs articles
* Un article référence un produit
* Un produit appartient à une catégorie

---

## Objectif fonctionnel

Vous devez écrire une requête SQL permettant de **lister les clients** répondant **simultanément** aux critères suivants :

1. Le client a effectué des achats dans la catégorie **« Electronics »**.
2. Le **montant total cumulé** de ses achats dans cette catégorie est **strictement supérieur à 500 €**.
3. Le client **n’a pas acheté de produits** de la catégorie **« Books »** au cours des **12 derniers mois**.

Les informations attendues pour chaque client sont :

* `customer_id`
* `first_name`
* `last_name`
* `email`

---

## Travail demandé

### Partie 1 – Requête SQL initiale (monolithique)

1. Écrivez une **requête SQL complète** permettant de répondre à l’objectif fonctionnel ci-dessus.
2. Vous pouvez utiliser :

   * des `JOIN`
   * des sous-requêtes
   * des fonctions d’agrégation (`SUM`, `GROUP BY`, `HAVING`)
3. À ce stade, la lisibilité n’est pas la priorité : l’objectif est d’obtenir un **résultat fonctionnel correct**.

Cette requête servira de point de départ pour la suite du TP.

---

### Partie 2 – Refactorisation avec Common Table Expressions (CTE)

Vous allez maintenant **refactoriser** votre requête afin d’en améliorer la structure.

#### 2.1 Décomposition logique avec CTE

1. Identifiez les **sous-problèmes logiques** de la requête, par exemple :

   * Identifier les clients avec des achats importants en électronique
   * Identifier les clients ayant acheté des livres récemment
2. Créez une ou plusieurs **CTE** pour représenter ces sous-ensembles de données.

Objectif : chaque CTE doit avoir une **responsabilité claire** et un **nom explicite**.

---

#### 2.2 Exclusion des clients via `LEFT JOIN … IS NULL`

1. Implémentez une solution dans laquelle :

   * Une CTE identifie les clients éligibles (électronique > 500 €)
   * Une autre CTE identifie les clients à exclure (acheteurs récents de livres)
2. Utilisez un `LEFT JOIN` combiné à une condition `IS NULL` pour exclure les clients non désirés.

Vous veillerez à commenter votre requête afin d’expliquer chaque étape.

---

### Partie 3 – Variante avec `NOT EXISTS` et CTEs granulaires

Dans cette partie, vous proposerez une **seconde solution alternative**, toujours basée sur les CTE.

1. Décomposez le calcul du montant des achats électroniques en plusieurs étapes (par exemple :

   * calcul par ligne de commande
   * agrégation par client)
2. Utilisez la clause `NOT EXISTS` pour gérer l’exclusion des clients ayant acheté des livres récemment.
3. Comparez cette approche avec celle utilisant `LEFT JOIN … IS NULL`.

---

## Contraintes techniques

* Vous pouvez adapter la fonction de gestion des dates selon le SGBD utilisé
  (SQLite, PostgreSQL, MySQL, SQL Server).
* Le code SQL doit être :

  * correctement indenté
  * commenté
  * lisible par un autre développeur

---

## Livrables attendus

* Un fichier `.sql` contenant :

  * la requête initiale
  * les deux versions refactorisées avec CTE
* Un court document (ou commentaires dans le SQL) répondant aux questions de réflexion

---

## Compétences évaluées

* Maîtrise des jointures SQL et des agrégations
* Utilisation correcte des Common Table Expressions (CTE)
* Capacité à structurer une requête complexe
* Approche génie logiciel appliquée aux bases de données
* Qualité, lisibilité et maintenabilité du code SQL

