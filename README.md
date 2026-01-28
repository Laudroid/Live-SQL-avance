# Cours SQL Avance

## Objectifs de la formation

- Optimisation avancée des accès aux données
- Architecture SQL robuste et sécurisée
- Maîtrise des mécanismes internes du SGBD
- Préparation aux systèmes critiques (finance, SI, ERP, SaaS)
## Séance 1 : SQL avancé : lisibilité, maintenabilité & bonnes pratiques

**Durée :** 2h30

### Objectifs pédagogiques

- Appliquer les bonnes pratiques pour écrire des requêtes SQL claires et maintenables
- Améliorer la lisibilité des requêtes SQL complexes
- Adopter des normes et conventions pour les projets professionnels
### Contenus

#### Normes de codage SQL

- Indentation et organisation logique des requêtes
- Nomination explicite des alias et objets
- Utilisation des commentaires pour la documentation
#### Techniques pour requêtes maintenables

- Modularisation avec des sous-requêtes et CTE
- Éviter les redondances
- Gestion des jointures explicites
#### Optimisation de la lisibilité

- Formatage standardisé
- Clarté dans l'ordre des clauses (SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY)
- Exemples pratiques sur PostgreSQL
## Séance 2 : CTE (Common Table Expressions)

**Durée :** 2h30

### Objectifs pédagogiques

- Comprendre et utiliser les CTE pour simplifier les requêtes complexes
- Exploiter les CTE récursifs pour traiter des structures hiérarchiques
- Améliorer la modularité et la réutilisabilité des requêtes
### Contenus

#### Introduction aux CTE

- Syntaxe et structure d'un CTE
- Usage classique vs récursif
#### CTE récursifs

- Mise en œuvre d'exemples sur arbres hiérarchiques
- Gestion des cycles et limites
#### Avantages et bonnes pratiques

- Lisibilité et décomposition des requêtes
- Comparaison avec les sous-requêtes classiques
- Impact sur les performances dans PostgreSQL
## Séance 3 : Views & Materialized Views

**Durée :** 2h30

### Objectifs pédagogiques

- Comprendre les concepts de vues et vues matérialisées
- Savoir créer, utiliser et gérer les vues dans PostgreSQL
- Évaluer les bénéfices des vues matérialisées pour la performance
### Contenus

#### Différences entre Views et Materialized Views

- Définition et usages
- Comportement au niveau des données
- Actualisation des vues matérialisées
#### Création et gestion dans PostgreSQL

- Syntaxe de création des vues
- Options d'actualisation des vues matérialisées
- Maintenance et optimisation
#### Cas d'utilisation avancés

- Amélioration des performances de requêtes lourdes
- Sécurisation des données via les vues
- Exemples pratiques en contexte ERP ou finance
## Séance 4 : Transactions & concurrence

**Durée :** 3h

### Objectifs pédagogiques

- Maîtriser les concepts de transactions et leur isolation
- Gérer les problèmes de concurrence et blocages
- Utiliser les mécanismes de verrouillage dans PostgreSQL
### Contenus

#### Principes des transactions

- Définition, ACID, et niveaux d'isolation
- Exemples concrets d'utilisation
#### Concurrence et problèmes associés

- Phénomènes de concurrence : dirty read, non-repeatable read, phantom read
- Deadlocks et stratégies de résolution
#### Gestion des verrous dans PostgreSQL

- Types de verroux
- Commandes et vues pour monitorer les blocages
- Bonnes pratiques pour éviter les conflits
## Séance 5 : Fonctions & procédures stockées

**Durée :** 2h30

### Objectifs pédagogiques

- Écrire et utiliser des fonctions et procédures stockées en PL/pgSQL
- Comprendre les avantages en termes de performance et maintenance
- Intégrer la programmation SQL avancée dans les bases de données
### Contenus

#### Création de fonctions et procédures

- Syntaxe et structure en PL/pgSQL
- Gestion des paramètres et types de retour
#### Fonctions avancées

- Utilisation des variables et contrôle de flux
- Gestion des exceptions
#### Cas pratiques

- Automatisation de tâches récurrentes
- Encapsulation de logique métier dans la base
## Séance 6 : Sécurité, permissions & rôles

**Durée :** 2h30

### Objectifs pédagogiques

- Maîtriser la gestion des permissions dans PostgreSQL
- Configurer des rôles pour une architecture sécurisée
- Garantir la confidentialité et l'intégrité des données
### Contenus

#### Modèle de sécurité PostgreSQL

- Concepts de rôles, utilisateurs, et groupes
- Hiérarchie et héritage des permissions
#### Gestion des permissions

- Attribution et révocation de privilèges
- Sécurité au niveau des objets (tables, vues, fonctions)
#### Bonnes pratiques et audits

- Principes du moindre privilège
- Utilisation des audits et logs
- Stratégies pour environnements critiques