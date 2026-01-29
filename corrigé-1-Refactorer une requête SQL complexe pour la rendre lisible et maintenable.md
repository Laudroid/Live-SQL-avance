Voici deux solutions refactorisées pour la requête SQL complexe, présentées sous forme de chapitres distincts. L'objectif est d'améliorer la lisibilité et la maintenabilité en utilisant les Common Table Expressions (CTEs).

---

### **Chapitre 1 : Solution avec CTEs Directes et `LEFT JOIN ... IS NULL`**

Cette solution décompose la requête initiale en deux CTEs principales, chacune gérant une condition de filtrage majeure. La première CTE identifie les clients ayant des achats importants en électronique, et la seconde ceux ayant acheté des livres récemment. La jointure finale combine ces résultats en excluant les clients de la deuxième catégorie via un `LEFT JOIN` et une condition `IS NULL`.


```sql
-- Solution 1: Refactorisation avec CTEs pour chaque condition principale et exclusion via LEFT JOIN
WITH
    -- CTE 1: Identifie les clients ayant acheté des produits 'Electronics' pour un montant total supérieur à 500€.
    -- Cette CTE agrège les montants des articles par client pour la catégorie 'Electronics'.
    ElectronicsHighValueBuyers AS (
        SELECT
            o.customer_id
        FROM
            Orders o
        JOIN
            OrderItems oi ON o.order_id = oi.order_id
        JOIN
            Products p ON oi.product_id = p.product_id
        JOIN
            Categories cat ON p.category_id = cat.category_id
        WHERE
            cat.category_name = 'Electronics'
        GROUP BY
            o.customer_id
        HAVING
            SUM(oi.quantity * oi.unit_price) > 500
    ),
    -- CTE 2: Identifie les clients ayant acheté des produits de la catégorie 'Books' au cours des 12 derniers mois.
    -- La fonction de date doit être adaptée au SGBD utilisé (ex: CURRENT_DATE - INTERVAL '1 year' pour PostgreSQL,
    -- CURDATE() - INTERVAL 1 YEAR pour MySQL, DATEADD(year, -1, GETDATE()) pour SQL Server).
    RecentBookBuyers AS (
        SELECT DISTINCT -- DISTINCT est utilisé pour s'assurer que chaque client n'apparaît qu'une seule fois
            o.customer_id
        FROM
            Orders o
        JOIN
            OrderItems oi ON o.order_id = oi.order_id
        JOIN
            Products p ON oi.product_id = p.product_id
        JOIN
            Categories cat ON p.category_id = cat.category_id
        WHERE
            cat.category_name = 'Books'
            AND o.order_date >= DATE('now', '-12 months') -- Exemple pour SQLite
    )
-- Requête finale: Sélectionne les clients qui sont des 'ElectronicsHighValueBuyers'
-- et qui NE SONT PAS des 'RecentBookBuyers'.
-- L'exclusion est gérée par un LEFT JOIN vers RecentBookBuyers et une condition IS NULL.
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    c.email
FROM
    Customers c
JOIN
    ElectronicsHighValueBuyers ehvb ON c.customer_id = ehvb.customer_id
LEFT JOIN
    RecentBookBuyers rbb ON c.customer_id = rbb.customer_id
WHERE
    rbb.customer_id IS NULL; -- Cette condition exclut les clients trouvés dans RecentBookBuyers
```


---

### **Chapitre 2 : Solution avec CTEs Granulaires et `NOT EXISTS`**

Cette deuxième approche affine la première condition en la décomposant en deux CTEs pour une granularité accrue. La première CTE calcule le total des achats électroniques par article, la seconde agrège ces totaux par client. L'exclusion des acheteurs de livres est ensuite gérée par la clause `NOT EXISTS`, qui peut parfois offrir une meilleure lisibilité pour les conditions d'exclusion.


```sql
-- Solution 2: Refactorisation avec CTEs plus granulaires et exclusion via NOT EXISTS
WITH
    -- CTE 1: Calcule le montant total pour chaque article acheté dans la catégorie 'Electronics'.
    -- Cette étape prépare les données pour l'agrégation ultérieure.
    ElectronicsItemTotals AS (
        SELECT
            o.customer_id,
            (oi.quantity * oi.unit_price) AS item_total_amount
        FROM
            Orders o
        JOIN
            OrderItems oi ON o.order_id = oi.order_id
        JOIN
            Products p ON oi.product_id = p.product_id
        JOIN
            Categories cat ON p.category_id = cat.category_id
        WHERE
            cat.category_name = 'Electronics'
    ),
    -- CTE 2: Identifie les clients ayant dépensé plus de 500€ en produits 'Electronics'.
    -- Elle s'appuie sur la CTE précédente pour agréger les montants par client.
    HighValueElectronicsCustomers AS (
        SELECT
            eits.customer_id
        FROM
            ElectronicsItemTotals eits
        GROUP BY
            eits.customer_id
        HAVING
            SUM(eits.item_total_amount) > 500
    ),
    -- CTE 3: Identifie les clients ayant acheté des produits de la catégorie 'Books' au cours des 12 derniers mois.
    -- Similaire à la solution précédente, mais renommée pour la cohérence de cette solution.
    RecentBookPurchasers AS (
        SELECT DISTINCT
            o.customer_id
        FROM
            Orders o
        JOIN
            OrderItems oi ON o.order_id = oi.order_id
        JOIN
            Products p ON oi.product_id = p.product_id
        JOIN
            Categories cat ON p.category_id = cat.category_id
        WHERE
            cat.category_name = 'Books'
            AND o.order_date >= DATE('now', '-12 months') -- Adaptez la fonction de date à votre SGBD
    )
-- Requête finale: Sélectionne les clients qui sont des 'HighValueElectronicsCustomers'
-- et qui NE SONT PAS des 'RecentBookPurchasers'.
-- L'exclusion est gérée par la clause NOT EXISTS, qui vérifie l'absence d'un enregistrement correspondant.
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    c.email
FROM
    Customers c
JOIN
    HighValueElectronicsCustomers hvec ON c.customer_id = hvec.customer_id
WHERE NOT EXISTS (
    SELECT 1
    FROM RecentBookPurchasers rbp
    WHERE rbp.customer_id = c.customer_id
);
```