**ACID** : Acronyme pour Atomicité, Consistance, Isolation, Durabilité, propriétés fondamentales garanties par un SGBD pour les transactions.

**Actualisation des vues matérialisées** : Processus de mise à jour des données stockées dans une vue matérialisée pour refléter les changements des tables sous-jacentes.

**Alias** : Nom temporaire et souvent plus court donné à une table, une colonne ou une expression dans une requête SQL pour en améliorer la lisibilité.

**Architecture SQL** : La conception et l'organisation des composants d'une base de données relationnelle, incluant les structures de données et les mécanismes d'accès.

**Audits** : Examen systématique des activités et des journaux d'une base de données pour vérifier la conformité, la sécurité et l'intégrité des données.

**Blocages** : Situation où l'accès d'une transaction à une ressource (ex: ligne, table) est empêché car elle est déjà verrouillée par une autre transaction.

**Clauses SQL** : Éléments constitutifs d'une requête SQL (comme SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY) qui définissent son comportement et sa logique.

**Commentaires** : Texte explicatif inséré dans le code SQL, ignoré par le SGBD, utilisé pour la documentation et la compréhension des requêtes.

**Concurrence** : Situation où plusieurs transactions ou utilisateurs accèdent et/ou modifient simultanément les mêmes données dans la base de données.

**Contrôle de flux (PL/pgSQL)** : Structures de programmation (comme IF, LOOP) utilisées dans les langages procéduraux SQL (tel que PL/pgSQL) pour diriger l'exécution du code selon des conditions.

**CTE (Common Table Expressions)** : Requêtes nommées temporaires définies dans l'instruction d'une seule requête, utilisées pour simplifier les requêtes complexes et améliorer la lisibilité et la modularité.

**CTE récursifs** : Type de CTE capable de se référer à lui-même, utilisé pour traiter des structures de données hiérarchiques ou arborescentes.

**Cycles (CTE récursifs)** : Condition où un CTE récursif pourrait entrer dans une boucle infinie en suivant des relations circulaires dans les données hiérarchiques.

**Deadlocks** : Impasse où deux (ou plus) transactions se bloquent mutuellement en attendant des ressources que l'autre transaction détient déjà et ne libère pas.

**Dirty read** : Problème d'isolation des transactions où une transaction lit des données modifiées par une autre transaction qui n'a pas encore validé (commit) ses changements.

**ERP (Enterprise Resource Planning)** : Progiciel de gestion intégré qui couvre l'ensemble des fonctions opérationnelles d'une entreprise.

**Finance** : Domaine d'application mentionné pour les systèmes critiques, nécessitant une architecture SQL robuste et sécurisée avec des transactions fiables.

**Fonctions (PL/pgSQL)** : Bloc de code PL/pgSQL nommé qui effectue une tâche spécifique, peut prendre des paramètres et retourne une seule valeur.

**Formatage standardisé** : Application de règles uniformes (indentation, capitalisation, espacement) pour rendre le code SQL cohérent et facile à lire.

**Gestion des exceptions (PL/pgSQL)** : Mécanisme dans PL/pgSQL permettant de capturer et de traiter les erreurs qui surviennent pendant l'exécution d'une fonction ou procédure.

**Groupes (PostgreSQL)** : Collection de rôles ou d'utilisateurs à laquelle des privilèges peuvent être attribués collectivement (terme parfois utilisé pour les rôles avec des membres).

**Hiérarchie et héritage des permissions** : Système où les permissions peuvent être structurées en niveaux et transmises (héritées) des rôles parents aux rôles enfants.

**Indentation** : Utilisation d'espaces ou de tabulations pour décaler le code et mettre en évidence sa structure logique, améliorant la lisibilité.

**Isolation (transactions)** : Propriété ACID garantissant que l'exécution concurrente de transactions n'entraîne pas d'interférences entre elles.

**Jointures explicites** : Syntaxe SQL qui utilise des clauses comme INNER JOIN, LEFT JOIN, RIGHT JOIN pour spécifier clairement comment les tables sont liées.

**Lisibilité (requêtes SQL)** : Caractéristique d'une requête SQL qui est facile à comprendre, même pour quelqu'un qui ne l'a pas écrite.

**Logs (PostgreSQL)** : Fichiers journaux enregistrant les événements, erreurs et activités du SGBD, essentiels pour l'audit et le dépannage.

**Logique métier** : Ensemble des règles et processus spécifiques à un domaine d'activité ou à une entreprise, souvent encapsulés dans des fonctions ou procédures stockées.

**Maintenabilité (requêtes SQL)** : Capacité d'une requête SQL à être facilement modifiée, corrigée ou mise à jour sans introduire de nouveaux problèmes.

**Materialized Views (Vues matérialisées)** : Objets de base de données qui stockent physiquement le résultat d'une requête, offrant des performances améliorées au détriment de la fraîcheur des données qui doivent être actualisées.

**Monitorer les blocages** : Activité de surveillance des sessions et des verrous pour identifier et analyser les situations de blocage dans la base de données.

**Non-repeatable read** : Problème d'isolation des transactions où une transaction lit la même ligne de données deux fois et obtient des valeurs différentes, car une autre transaction a modifié et validé cette ligne entre-temps.

**Normes de codage SQL** : Ensemble de conventions et de directives établies pour écrire du code SQL de manière cohérente, lisible et maintenable.

**Objets de base de données** : Éléments d'une base de données tels que les tables, vues, fonctions, procédures stockées, sur lesquels des permissions peuvent être définies.

**Optimisation des performances** : Amélioration de la vitesse et de l'efficacité d'exécution des requêtes SQL ou du SGBD.

**Paramètres (fonctions/procédures)** : Variables passées à une fonction ou procédure stockée lors de son appel, servant d'entrées ou de sorties.

**Permissions** : Droits spécifiques accordés à un rôle ou un utilisateur pour effectuer des opérations (ex: SELECT, INSERT, UPDATE, DELETE) sur des objets de base de données.

**Phantom read** : Problème d'isolation des transactions où une requête exécutée deux fois dans la même transaction retourne un ensemble de lignes différent, car une autre transaction a inséré ou supprimé des lignes entre-temps.

**PL/pgSQL** : Langage procédural intégré à PostgreSQL, utilisé pour écrire des fonctions, procédures stockées et déclencheurs.

**PostgreSQL** : Système de Gestion de Base de Données Relationnel (SGBDR) open source, robuste et riche en fonctionnalités.

**Principes du moindre privilège** : Règle de sécurité qui stipule qu'un utilisateur, rôle ou programme ne devrait avoir que le minimum de permissions nécessaires pour accomplir sa tâche.

**Privilèges** : Droits accordés sur des objets de base de données, permettant des actions spécifiques (synonyme de permissions).

**Procédures stockées (PL/pgSQL)** : Bloc de code PL/pgSQL nommé qui effectue une tâche spécifique, peut prendre des paramètres mais ne retourne pas nécessairement une valeur.

**Requêtes SQL** : Instructions écrites en langage SQL (Structured Query Language) pour interagir avec une base de données.

**Robustesse (architecture SQL)** : Caractéristique d'une architecture SQL qui assure sa stabilité, sa fiabilité et sa capacité à fonctionner correctement même sous contraintes.

**Rôles (PostgreSQL)** : Entités dans PostgreSQL qui peuvent représenter un utilisateur ou un groupe d'utilisateurs, et à qui on peut attribuer des privilèges ou la propriété d'objets.

**Sécurité (SQL)** : Ensemble des mesures visant à protéger les données et le SGBD contre les accès non autorisés, les modifications malveillantes ou les pertes.

**Sous-requêtes** : Requête SQL imbriquée (ou 'nested') à l'intérieur d'une autre requête SQL.

**Structures hiérarchiques** : Données organisées de manière arborescente, où les éléments ont des relations parent-enfant (ex: organigrammes, catégories).

**Transactions** : Séquence d'une ou plusieurs opérations sur la base de données qui sont traitées comme une seule unité logique de travail, garantissant les propriétés ACID.

**Types de retour (fonctions PL/pgSQL)** : Définition du type de données (par exemple, INTEGER, TEXT, RECORD) qu'une fonction PL/pgSQL renverra après son exécution.

**Utilisateurs (PostgreSQL)** : Comptes individuels dans PostgreSQL qui permettent de se connecter à la base de données et d'effectuer des opérations selon les rôles qui leur sont attribués.

**Variables (PL/pgSQL)** : Conteneurs nommés utilisés dans les fonctions et procédures PL/pgSQL pour stocker temporairement des données.

**Verrouillage** : Mécanisme utilisé par les SGBD pour gérer l'accès concurrent aux données, empêchant les conflits et assurant l'intégrité des données.

**Views (Vues)** : Table virtuelle basée sur le résultat d'une requête SQL, qui ne stocke pas les données physiquement mais agit comme une fenêtre sur les données sous-jacentes.

