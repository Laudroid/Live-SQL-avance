# TP SQL – Vues matérialisées, performance et actualisation concurrente

## Contexte pédagogique

Dans les systèmes d’information décisionnels (reporting, BI, tableaux de bord), les requêtes portent souvent sur :

* de **grands volumes de données**,
* des **agrégations complexes**,
* des données qui évoluent régulièrement mais **pas en temps réel**.

Dans ce contexte, l’utilisation de **vues classiques** devient rapidement insuffisante en termes de performance.
Les **vues matérialisées** apportent une solution en stockant physiquement les résultats d’une requête, au prix d’un mécanisme d’actualisation à maîtriser.

Ce TP vise à vous placer dans une **situation proche de la production**, en abordant :

* la création de vues matérialisées de reporting,
* leur optimisation,
* leur actualisation concurrente,
* et les problématiques d’exploitation (automatisation, surveillance).

---

## Schéma de données et domaine fonctionnel

On considère une base de données de **vente e-commerce**, avec les tables suivantes :

### **products**

* `product_id`
* `product_name`
* `category`

### **regions**

* `region_id`
* `region_name`

### **sales**

* `sale_id`
* `product_id`
* `region_id`
* `sale_date`
* `quantity`
* `price_per_unit`
* `total_amount` (calculé)

Les ventes sont associées :

* à un produit (et donc une catégorie),
* à une région,
* à une date.

---

## Objectif fonctionnel

Vous devez mettre en place un **reporting mensuel agrégé** permettant d’analyser les ventes selon :

* le **mois de vente**,
* la **catégorie de produit**,
* la **région**.

Pour chaque combinaison, les indicateurs suivants doivent être calculés :

* nombre total de ventes,
* quantité totale vendue,
* chiffre d’affaires total.

Ce reporting doit :

* être **rapide à interroger**,
* rester **disponible pendant les actualisations**,
* être **exploitable dans un contexte multi-utilisateurs**.

---

## Travail demandé

### Partie 1 – Mise en place de l’environnement de travail

1. Créez une base de données dédiée au reporting.
2. Créez les tables `products`, `regions` et `sales`.
3. Ajoutez des **contraintes d’intégrité** (clés primaires, clés étrangères, contraintes de validité).
4. Ajoutez des **index pertinents** sur les tables sources afin d’optimiser :

   * les jointures,
   * les filtres temporels.
5. Insérez des données d’exemple.

   * Vous êtes encouragés à générer un **volume significatif de données** afin de rendre les problématiques de performance visibles.

Objectif pédagogique :
comprendre que les vues matérialisées reposent sur des tables sources correctement modélisées et indexées.

---

### Partie 2 – Création d’une vue matérialisée de reporting

1. Écrivez la requête SQL permettant de produire le reporting mensuel agrégé :

   * groupement par mois, catégorie et région,
   * calcul des indicateurs demandés.
2. Transformez cette requête en **vue matérialisée**.
3. Vérifiez que la vue contient les résultats attendus.

---

### Partie 3 – Optimisation de la vue matérialisée

1. Analysez les colonnes utilisées pour le groupement.
2. Créez un **index unique** sur la vue matérialisée :

   * couvrant les colonnes identifiant de manière unique chaque ligne du reporting.
3. Expliquez :

   * pourquoi cet index est important pour les performances,
   * et pourquoi il est indispensable pour l’actualisation concurrente.

Objectif pédagogique :
lier **structure logique**, **indexation** et **mécanismes internes du SGBD**.

---

### Partie 4 – Actualisation de la vue matérialisée et concurrence

1. Simulez l’arrivée de nouvelles ventes dans la table `sales`.
2. Testez l’actualisation classique de la vue matérialisée (`REFRESH MATERIALIZED VIEW`).
3. Mettez ensuite en œuvre une **actualisation concurrente**.
4. À l’aide de **deux sessions SQL distinctes**, observez :

   * le comportement des lectures pendant l’actualisation,
   * la visibilité des anciennes et nouvelles données selon les transactions.

Objectif pédagogique :
comprendre les notions de **verrouillage**, **isolation transactionnelle** et **disponibilité des données**.

---

### Partie 5 – Automatisation de l’actualisation

1. Proposez une solution d’automatisation de l’actualisation :

   * via un script externe (ex : `cron`),
   * ou via un planificateur intégré au SGBD (si disponible).
2. Expliquez :

   * à quelle fréquence l’actualisation doit être exécutée,
   * quels sont les risques d’une actualisation trop fréquente ou trop rare.

---

### Partie 6 – Surveillance et exploitation

1. Identifiez les outils permettant de :

   * surveiller les requêtes d’actualisation,
   * analyser leur durée d’exécution,
   * détecter d’éventuels blocages.
2. Proposez des indicateurs simples à surveiller en production :

   * durée moyenne d’actualisation,
   * fréquence d’échec,
   * impact sur les autres requêtes.

---

### Partie 7 – Approfondissement : vues matérialisées `WITH NO DATA`

1. Créez une vue matérialisée avec l’option `WITH NO DATA`.
2. Expliquez dans quels cas cette option est pertinente :

   * déploiement initial,
   * très grands volumes,
   * dépendances complexes.
3. Déclenchez manuellement le premier peuplement de la vue.

Objectif pédagogique :
comprendre que la création et le peuplement d’une vue matérialisée peuvent être **décorrélés**.

---

## Contraintes techniques

* Le TP est basé sur **PostgreSQL**.
* Les scripts doivent être :

  * lisibles,
  * commentés,
  * reproductibles.
* Toute hypothèse (volume, fréquence, usage) doit être explicitée.

---

## Livrables attendus

* Un ou plusieurs fichiers `.sql` contenant :

  * la création des tables,
  * la vue matérialisée,
  * les index,
  * les commandes d’actualisation.
* Un court document de synthèse répondant aux questions de réflexion.

---

## Compétences évaluées

* Compréhension des mécanismes internes du SGBD
* Maîtrise des vues matérialisées
* Raisonnement orienté performance et exploitation
* Gestion de la concurrence et des transactions
* Approche “production ready” des bases de données