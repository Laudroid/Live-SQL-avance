# Gestion des cycles et limites dans les CTE récursifs  

Les Common Table Expressions récursifs permettent d’explorer des structures hiérarchiques en SQL. Toutefois, les graphes explorés peuvent contenir des cycles, ce qui cause des boucles infinies. Par ailleurs, les bases de données imposent des limites sur la profondeur de récursion. Ce guide détaille comment gérer ces deux aspects pour garantir la robustesse et la fiabilité des requêtes SQL récursives.

---

## 1. Problème des cycles dans les données hiérarchiques  

Un **cycle** se produit lorsqu’un nœud apparait plusieurs fois sur un même chemin de la hiérarchie, provoquant une boucle infinie dans la récursion SQL. Par exemple, dans une table d’employés, un manager pourrait référencer indirectement un de ses ancêtres via une chaîne de dépendances erronée.

---

## 2. Techniques pour éviter les cycles  

### 2.1. Suivi des chemins visités  

La méthode la plus courante consiste à accumuler dans la récursion l’historique des identifiants déjà rencontrés et à filtrer les nœuds déjà visités.

### Exemple : détection et élimination des cycles dans une hiérarchie `employees`  

```sql
WITH RECURSIVE employee_path AS (
    -- Cas de base
    SELECT employee_id, manager_id, ARRAY[employee_id] AS path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Requêtes récursive
    SELECT e.employee_id, e.manager_id, path || e.employee_id
    FROM employees e
    INNER JOIN employee_path ep ON e.manager_id = ep.employee_id
    WHERE NOT e.employee_id = ANY(path) -- évite la répétition dans le chemin
)
SELECT * FROM employee_path;
```

Ici, `path` est un tableau PostgreSQL qui stocke la séquence des IDs visités. La condition `WHERE NOT e.employee_id = ANY(path)` empêche le réexamen des nœuds déjà sur le chemin, supprimant ainsi les cycles.

---

## 3. Limites imposées par les SGBD  

### 3.1. Limite de profondeur de récursion  

Pour éviter des boucles infinies ou des descentes trop profondes, les systèmes comme PostgreSQL ou SQL Server limitent par défaut la récursion (exemple : 100 niveaux par défaut dans PostgreSQL).

- **PostgreSQL** :  
  La variable `max_recursion_depth` contrôle cette limite. Elle peut être augmentée ou diminuée par la commande :  

  ```sql
  SET max_recursion_depth = 200;
  ```

- **SQL Server** :  
  Max recursion est fixé dans la clause `OPTION (MAXRECURSION n)` avec `0` pour illimité (à utiliser avec précaution).  

### Exemple avec limite dans SQL Server

```sql
OPTION (MAXRECURSION 100)
```

---

## 4. Diagramme Mermaid illustratif  

Représentation simplifiée d’un parcours récursif avec suivi de chemin pour éviter les cycles :

```mermaid
flowchart TD
    A[Début] --> B[Cas de base (racines)]
    B --> C[Requête récursive]
    C --> D{Le nœud est-il déjà dans path ?}
    D -- Oui --> E[Ignorer pour éviter cycle]
    D -- Non --> F[Ajouter au path et continuer]
    F --> C
```

---

## 5. Bonnes pratiques  

- Toujours prévoir un suivi des nœuds visités pour les graphes potentiellement cycliques.  
- Tester les requêtes avec des jeux de données comportant des cycles.  
- Configurer une limite de récursion adaptée pour protéger la base.  
- En cas de risque élevé, privilégier la résolution côté application ou avec des algorithmes externes.  

---

## 6. Sources documentaires  

- [PostgreSQL Documentation - WITH Queries](https://www.postgresql.org/docs/current/queries-with.html)  
- [PostgreSQL Wiki - Common Table Expressions](https://wiki.postgresql.org/wiki/CTE)  
- [SQL Server Recursive CTE Documentation](https://docs.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql#recursive-ctes)  
- [Mode Analytics - Recursive CTE Best Practices](https://mode.com/sql-tutorial/sql-recursive-cte/#cycle-detection)  

---

## 7. En résumé  

La gestion des cycles dans les CTE récursifs est impérative pour garantir que la requête termine correctement. Le suivi explicite du chemin parcouru dans un tableau évite les répétitions et les boucles infinies. Complétée par une limite de récursion adéquate, cette approche assure robustesse et performance dans la manipulation des structures hiérarchiques complexes.