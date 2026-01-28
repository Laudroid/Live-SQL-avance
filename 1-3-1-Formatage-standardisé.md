# Optimisation de la lisibilité : Formatage standardisé en SQL  

Le formatage standardisé du code SQL est un levier essentiel pour améliorer la lisibilité, accélérer la compréhension et faciliter la collaboration entre équipes. Un style uniforme permet de reproduire des habitudes solides, évitant les erreurs dues à une lecture erratique.

---

## 1. Pourquoi standardiser le formatage ?

- Facilite la lecture et la maintenance du code  
- Simplifie la détection des erreurs et incohérences  
- Permet aux équipes d’adopter un style commun, réduisant les divergences  
- Améliore la réutilisation et l’extensibilité des requêtes

---

## 2. Principes clés de formatage standardisé

### a) Indentation cohérente

Chaque nouvelle clause ou bloc important est indenté, typiquement avec 2 à 4 espaces, pour marquer la structure.

```sql
SELECT customer_id, customer_name
FROM customers
WHERE status = 'active';
```

Pour les jointures :

```sql
SELECT c.customer_name, o.order_id
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= '2024-01-01';
```

### b) Utilisation de majuscules pour les mots-clés SQL

Pour différencier clairement les mots-clés des noms d’objets, écrire `SELECT`, `FROM`, `WHERE` en majuscules.

### c) Une clause par ligne

Placer chaque clause SQL principale (`SELECT`, `FROM`, `JOIN`, `WHERE`, `GROUP BY`, `ORDER BY`) sur sa propre ligne.

### d) Séparation lisible des éléments listés

Dans `SELECT` ou `INSERT`, séparer les colonnes par une virgule en début ou fin de ligne, selon la convention choisie.

```sql
SELECT
    customer_id,
    customer_name,
    email
FROM customers;
```

### e) Espacer les opérateurs et parenthèses

Utiliser des espaces autour des opérateurs (`=`, `>`, `<`, `+`, etc.) pour clarifier la lecture.

---

## 3. Exemple complet mis en forme

```sql
WITH recent_orders AS (
    SELECT
        order_id,
        customer_id,
        order_date
    FROM orders
    WHERE order_date >= '2024-01-01'
)

SELECT
    c.customer_name,
    r.order_id,
    r.order_date
FROM customers c
INNER JOIN recent_orders r ON c.customer_id = r.customer_id
WHERE c.status = 'active'
ORDER BY r.order_date DESC;
```

---

## 4. Diagramme Mermaid : Structure d’une requête SQL formatée

```mermaid
flowchart TB
    A[WITH clause (CTE)] --> B[SELECT clause]
    B --> C[FROM clause]
    C --> D[JOIN clauses]
    D --> E[WHERE clause]
    E --> F[GROUP BY / HAVING clause]
    F --> G[ORDER BY clause]
```

---

## 5. Outils pour un formatage automatisé

- **SQL Formatter** : services en ligne comme [https://sqlformat.org](https://sqlformat.org)  
- **Extensions IDE** : SQL Prompt (Redgate), SQL Formatter (VSCode)  
- **Linting SQL** : analyse de style avec des outils intégrés dans pipelines CI/CD  

Ces outils permettent d’appliquer des normes automatiquement, garantissant une uniformité constante.

---

## 6. Références pédagogiques et guides à jour

- [SQL Style Guide - Simon Holywell](https://www.sqlstyle.guide/)  
- [Redgate - SQL Formatter and Best Practices](https://www.red-gate.com/simple-talk/sql/t-sql-programming/sql-code-formatting-tools-and-best-practices/)  
- [PostgreSQL Documentation - Formatting Queries](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-WHITESPACE)  
- [Mode Analytics - SQL Style Guide](https://mode.com/sql-tutorial/sql-style-guide/)  

---

## 7. Synthèse  

Un formatage rigoureux et standardisé du SQL maximise la lisibilité et réduit les erreurs. L'adoption d'une convention partagée garantie une communication fluide dans les équipes et une meilleure réutilisation des requêtes.  

Une requête bien formatée est un premier pas vers un SQL maintenable, performant et agréable à manipuler.